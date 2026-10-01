# VeneerOS package repository

This repository is a small HTTPS package index for `ferm`. Application source
code and built release assets stay in their upstream projects; this repository
stores only the catalog and versioned package manifests.

## Repository layout

```text
repo/
├── index.json
└── packages/
    └── <package-name>/
        └── <version>.json
```

`repo/index.json` is the catalog consumed by `ferm`. It contains schema version
`1` and a `packages` object. Each catalog entry records the current version,
description, upstream repository URL, and HTTPS URL of that exact version's
manifest. The catalog currently has no entries because no release assets and
verified digests have been supplied.

Each manifest records its schema, package name, version, upstream repository,
and files to install. Every file entry must have an absolute destination under
`/programs`, `/data`, `/drivers`, or `/skins`, an HTTPS URL to an immutable,
versioned release asset, and the lowercase SHA-256 digest of that exact asset.
Do not add application payloads, mutable branch archives, `latest` URLs, or
placeholder digests here. Keep old versioned manifests when publishing updates.
Update the catalog's version and manifest URL together.

## `ferm`

The Python 3 `ferm` script in this repository is a developer-side utility. It
uses only the Python standard library, downloads over HTTPS, validates
manifests and SHA-256 digests, and supports a test or mounted tree with
`--root DIR`:

```sh
./ferm -I example-app --root /tmp/veneer-root
./ferm -U example-app --root /tmp/veneer-root
./ferm -R example-app --root /tmp/veneer-root
./ferm -AU --root /tmp/veneer-root
```

Without `--root`, paths are installed under `/` and the installed-package
database is `/sys/settings/ferm/installed.json`. Removal deletes only files
recorded for the package whose current contents still match the recorded
digest. Modified files are left in place and reported.

The VeneerOS HTTP library supports plain HTTP only, so it cannot safely fetch
this repository's HTTPS index or upstream GitHub assets. This Python utility
must be ported to an HTTPS-capable VeneerOS runtime before it can ship as an
in-OS shell command.