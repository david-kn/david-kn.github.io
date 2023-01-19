---
title: "Jenkins: GitHub event on a shared library triggers a job build"
date: 2023-01-03
---

**Issue description:** A Jenkins job gets triggered by a git push (event) to a Jenkins shared library but not a Jenkins job (pipeline) code itself. This behaviour might not be desirable when a job should be triggered only by a git push event strictly related and attached to a specific repo itself (when a setting `Git SCM polling` is enabled). Not any other _generic_ code like shared library defined globally for a whole Jenkins etc.

_Note:_ based on a Jenkins community, such a behaviour is actually in accordance to its design. Any change within any of the code related to the job (let's call it a _code dependency chain_) should be taken into account when working with `git scm polling` for changes and having git hooks set. This behaviour is also well logged within each impacted job - in section `GitHub Hook log` where it is transparent which event and/or change caused a job trigger.

**Desired state**: Git polling should monitor, and job get built only when a git push is related directly to the job repository.

**Solution:** 
A) Disable globally within Jenkins configuration an option for such shared library `Include @Library changes in job recent changes` (within Jenkins -> Manage Jenkins -> under specific Jenkins shared library section)
B) If such an option is not possible due to any other reasoning it is possible to disable such a behaviour _ad hoc_ and explicitly for individual pipelines by `changelog=false` parameter  within pipeline include statement, e.g.: 

```
Library("my-shared-library", changelog=false) _
```

Jenkins issue tracker: https://issues.jenkins.io/browse/JENKINS-47756 and https://issues.jenkins.io/browse/JENKINS-39615 
Solution found also here: https://devops.stackexchange.com/a/13167
