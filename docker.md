# Basics

Docker is integrated with dockerhub, a site where people are dropping containers. So, many commands, such as `docker run foo`, may try to fetch the container from the hub.

## Initial setup

After installation start the daemon as `systemctl start docker` *(at least on Arch)*, then `docker run hello-world` to check it's working.

To make sure terminal keys work properly, add this to `~/.docker/config.json`

    {
        "detachKeys": "ctrl-g,g"
    }

## General configuration

It's tricky. It has various deprecated configs that may or may not work depending on the init system. The config that always works is `/etc/docker/daemon.json` *(create it if it didn't exist)*. Then, if you want to add an option `-X` but don't know how to do that via config file, just search [this page](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file) for keywords from `dockerd --help` description. Example: I want to add `-g` option. Help says "data-root". So I search the page, and find there's exact key with this name. So I create `daemon.json` file as:

```
{
    "data-root": "/my/path"
}
```

## Running containers

* `docker run -it fedora:latest` to make docker download *(if needed)* the `fedora:latest` container, and run it interactively.
* `docker run -it --storage-opt size=1M fedora:latest`: same as above, but limits disk space allocated for container.

## Modifying containers

Whatever you do inside container does not get saved after you quit. While container is running, you can do `docker ps -l` to get its id, and then do `docker commit ID new-image-name` to save it. If you already quit, you still can restore it by using `start` and `attach`, see [this answer](https://stackoverflow.com/a/19616598/2388257)

## Storing to remote

1. Login with `docker login`
2. Associate image with a remote:
    ```
    docker tag local_image_name docker.io/user_name/remote_repo_name
    ```
3. Then push it: `docker push user_name/remote_repo_name`

# Dockerfile

A list of options and commands used to build an image. An example:

```
FROM ubuntu:bionic

RUN echo hello
```

`FROM` specifies container to be used as a base. `RUN` just commands to run.

Worth noting that Docker caches each such step, so for example to re-run from the last successful one on fail. That implies, it's worth to group commands at each `RUN` in such way, so if one failed, rebuilding wouldn't include too many commands.
