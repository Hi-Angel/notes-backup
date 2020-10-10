# misc

* a project URL has a owner's name before it. The owner can be either a user or a group.

Apparently, it's possible to point to a docker image with syntax `image: fedora:latest`.

# CI

When on the main project page, on the left panel there should be an entry called "CI/CD", for me it's just below "Merge Requests". If it's not there, it needs to be enabled in `Settings → General → Visibility → ✓ Pipelines`.

Actions are defined at `.gitlab-ci.yml` at the root of the project.
* *runner* is the entity that runs actions from `.gitlab-ci.yml`. It needs to be configured, and can be lots of different things. *Shared runner* is a runner that serves multiple projects.

gitlab runners are separate from the repository executables, that needs to run on a separate server, and then to register them inside a gitlab interface. Once a `gitlab-runner` installed, execute `gitlab-runner register`. Most of the stuff it asks for is from `Settings → CI/CD → Runners` tab. As the executor type use `shell` if unsure.

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

## Tricks

* *run a job as another user*: gitlab-runner creates a user `gitlab-runner`, which is used for building by default. You can add this user to `sudo` group, and execute `visudo` to add `gitlab-runner ALL=(ALL) NOPASSWD: ALL` line. This gonna allow to runner to run specific commands with `sudo` without a password.
* to see runner stats *(besides using a command line option)* you can put to `config.toml` a `listen_address = "[::]:9252"` *(it's a global option)*, then go to `http://ip-of-gitlab-runner/metrics`. Things to note: `https` don't seem to work, and make sure to type the `/metrics` subdir. The top-level page for some reason is 404.
  However trying to use them while debugging a problem with runner never finishing *(which somehow turned out to be because some process left running; which is bizzare since it was on a ssh session closed many steps ago)*, I haven't found the stats to be useful at all. They're too general.
