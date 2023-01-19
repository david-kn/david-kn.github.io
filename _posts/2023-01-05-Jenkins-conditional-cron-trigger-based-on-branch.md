---
title: "Jenkins: Conditional scheduled CRON trigger based on branch"
date: 2023-01-05
---

**Description / Desired state:** Multibranch or just a pipeline jenkins job should be under certain condition(s) scheduled with a `cron` or even a `parametrizedCron` settings but left with no schedules in all other cases (for all other branches).
To give a very specific example: when on a production `main` (or `master`) branch, the scheduled CRON should be active for a pipeline job. But no scheduling should be applied for any other (PR, dev, feature etc.) branch builds.

**Solution:** This kind of condition syntax is not natively supported at this time by Jenkins `triggers { ... }` or `cron {}` statements but can be achieved with a quite simple piece of code within Jenkinsfile.
An important part in the following example is the `CRON_STRING` variable initialization and use. Other simple parts are added just to make Jenkinsfile feel more real:

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

Or a very similar example with a `parametrizedCron` that allows multiple different cron configs with different parameters for a production branch (but not any other).

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

__Note: The condition itself can be of course more complex but for a clarity of this example as well as real-life application (scheduling only pipeline using the production branch), a ternary conditional operator is used._
