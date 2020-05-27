# misc

* a project URL has a owner's name before it. The owner can be either a user or a group.

Apparently, it's possible to point to a docker image with syntax `image: fedora:latest`.

# CI

When on the main project page, on the left panel there should be an entry called "CI/CD", for me it's just below "Merge Requests". If it's not there, it needs to be enabled in `Settings → General → Visibility → ✓ Pipelines`.

Actions are defined at `.gitlab-ci.yml` at the root of the project.
* *runner* is the entity that runs actions from `.gitlab-ci.yml`. It needs to be configured, and can be lots of different things. *Shared runner* is a runner that serves multiple projects.

gitlab runners are separate from the repository executables, that needs to run on a separate server, and then to register them inside a gitlab interface.

In general, following gitlab docs for setting up CI worked for me. The only question I stumbled upon was after I got it up and running, which is "how do I make CI only run on merge_requests?". The answer is:

```
mybuild:
  only:
    - merge_requests
    - master
  script:
    do_something
```

The `master` I didn't test, but on MRs it does get run.
