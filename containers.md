# To build and run containers without Docker

    dnf install buildah podman -y

Then to build the container:

    buildah bud --tag <tag> -f <file> /path/to/cwd

To run the `Dockerfile`'s `CMD`:

    podman run <tag>

# Docker Registry

Docker hosts a [container image](https://hub.docker.com/_/registry) that can
be used to host a local [registry](https://docs.docker.com/registry). See
[documentation](https://docs.docker.com/registry/deploying/) on using this
image from Docker.

To run:

    podman pull docker.io/
    podman run -d -p 5000:5000 --restart=always --name registry registry:latest

Then to push and pull images, they need to be tagged into the registry:

    podman tag postgres:latest localhost:5000/postgres:latest
    podman push localhost:5000/postgres:latest

This registry can then be updated in the Podman config as follows:

    cat /etc/containers/registries.conf
    ...snip...
    unqualified-search-registries = ["localhost:5000", "registry.fedoraproject.org", "registry.access.redhat.com", "quay.io", "docker.io"]
    ...snip...
    [[registry]]
    insecure = true
    location = "localhost:5000"

# Default registry search in Podman

Set `short-name-mode` in the configuration file:

    cat /etc/containers/registries.conf
    ...snip...
    short-name-mode="permissive"

# Podman Daemon

To start a Docker-compatible daemon ("API/REST responder")

    podman system service --time 0 unix:///path/to/some/podman.sock

To use this with Docker:

    export DOCKER_HOST="unix:///path/to/some/podman.sock"
    docker images
    podman images
