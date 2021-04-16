# Building packages

## With `debuild`

```
debuild -S -k<key-id>
```

Two helpful options:

 - `-sd`: exclude original tarball
 - `-sa`: explicitly include original tarball

Otherwise the results are contextual (usually, existing packages will lack
original tarball). See `dpkg-genchanges` for more information on these
options and other useful options.

Execute from within source directory (directory including `debian/`).


## Uploading packages

```
dput <ppa> /path/to/<package>_source.changes
```

Files listed in the signed `*_source.changes` will be uploaded. See above
about controlling what goes in that list.
