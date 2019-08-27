# Basics

Docker is integrated with dockerhub, a site where people are dropping containers. So, many commands, such as `docker run foo`, may try to fetch the container from the hub.

## Initial setup

After installation start the daemon as `systemctl start docker` *(at least on Arch)*, then `docker run hello-world` to check it's working.

To make sure terminal keys work properly, add this to `~/.docker/config.json`

    {
        "detachKeys": "ctrl-g,g"
    }

## Running containers

* `docker run -it fedora:latest` to make docker download *(if needed)* the `fedora:latest` container, and run it interactively.
* `docker run -it --storage-opt size=1M fedora:latest`: same as above, but limits disk space allocated for container.

## Modifying containers

Whatever you do inside container does not get saved after you quit. While container is running, you can do `docker ps -l` to get its id, and then do `docker commit ID new-image-name` to save it. If you already quit, you still can restore it by using `start` and `attach`, see [this answer](https://stackoverflow.com/a/19616598/2388257)
