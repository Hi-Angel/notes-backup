# Basics

Docker is integrated with dockerhub, a site where people are dropping containers. So, many commands, such as `docker run foo`, may try to fetch the container from the hub.

After installation start the daemon as `systemctl start docker` *(at least on Arch)*, then `docker run hello-world` to check it's working.

## Running containers

* `docker run -it fedora:latest` to make docker download *(if needed)* the `fedora:latest` container, and run it interactively.
* `docker run -it --storage-opt size=1M fedora:latest`: same as above, but limits disk space allocated for container.
