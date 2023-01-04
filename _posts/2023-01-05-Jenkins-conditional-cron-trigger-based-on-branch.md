---
title: "Jenkins: Conditional scheduled CRON trigger based on branch"
date: 2023-01-05
---

**Situation:** Multibranch or just a single pipeline branch jenkins job should be in some specific situations set with a `cron` or even a `parametrizedcron` settings but left with no schedules in other case.
To give a very specific and real example: when on a production `main` (or `master`) branch, the scheduled CRON should be active but not on any other (PR, dev, feature etc.) branches

**Solution:** This kind of syntax is not natively supported at this time by Jenkins `triggers { ... }` or `cron {}` statements but can be achieved with a quite simple check and variable initialization within Jenkinsfile.
Important part is with the `CRON_STRING` variable initialization and use. Other simple parts are added just to describe

```
String CRON_STRING = (scm.branches[0].name == "main") ? 'H */5 * * *' : ''

pipeline {  
  agent any
  triggers {
      cron(CRON_STRING)
  }
  
  // whatever other code, options, stages etc. is in your pipeline ...
  stages {
    stage("Test") {
      steps {
        echo "test"
      }
    }
  }
}
```

or a very similar example with a `parametrizedCron` that allows mulptiple different Cron configs with different parameters for a production branch (but not any other).

```
String CRON_STRING = (scm.branches[0].name == 'master' || scm.branches[0].name == 'main') ? '''
            0 7 * * 1-5 %SYNCHRONIZE_ALL=true;OPERATION_TYPE=UPDATE
            0 9 * * 1   %SYNCHRONIZE_ALL=true;OPERATION_TYPE=CREATE''': ''
pipeline {  
  agent any
  triggers {
      parameterizedCron(CRON_STRING)
  }

  stages {
    stage("Test") {
      steps {
        echo "test"
      }
    }
  }
}
```
