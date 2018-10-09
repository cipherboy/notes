# To build and run containers without Docker

    dnf install buildah podman -y

Then to build the container:

    buildah bud --tag <tag> -f <file> /path/to/cwd

To run the `Dockerfile`'s `CMD`:

    podman run <tag>
