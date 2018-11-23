# misc

* a project URL has a owner's name before it. The owner can be either a user or a group.

Apparently, it's possible to point to a docker image with syntax `image: fedora:latest`.

# CI

Actions are defined at `.gitlab-ci.yml` at the root of the project.
* *runner* is the entity that runs actions from `.gitlab-ci.yml`. It needs to be configured, and can be lots of different things.
*Shared runner* is a runner that serves multiple projects.

gitlab runners are separate from the repository executables, that needs to run on a separate server, and then to register them inside a gitlab interface.
