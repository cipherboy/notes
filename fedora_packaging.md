# Fedora Packaging Notes

## FreeRADIUS

 - `master` and `f29` are the same branch (currently).
 - `f28` is different and won't build on `f29+`.

1. To add a new source, use: (optional)

    fedpkg new-sources /path/to/source.tar.gz

2. To build locally and lint:

    fedpkg local
    fedpkg lint

3. To generate a COPR build:

    fedpkg copr-build freeradius-server --nowait

4. Commit changes & push.

5. Generate a Koji build:

    fedpkg build --nowait

6. After the Koji build has succeeded, use Bodhi to ship:

    bodhi updates new \
        --request=testing \
        --notes="Update to FreeRADIUS Server $version" \
        --type=enhancement \
        "freeradius-$version.fc$fedora"
