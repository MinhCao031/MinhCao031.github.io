# OPX Build: Build environment and scripts for OpenSwitch.

If you would like to download binaries instead, see
[Install OPX on Dell EMC ON Series platforms][install-docs].

# Quick start

```bash
# get source code
repo init -u https://github.com/open-switch/opx-manifest && repo sync

# build all open-source packages
opx-build/scripts/opx_run opx_build all

# assemble installer (use stretch)
opx-build/scripts/opx_run opx_rel_pkgasm.py --dist unstable \
  -b opx-onie-installer/release_bp/OPX_dell_base_stretch.xml
```

# Getting started with OpenSwitch

## Prerequisites

- 20-32GB free disk space
- [Docker](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/)
- [Repo](https://source.android.com/source/downloading)

## Get source code

```bash
repo init -u https://github.com/open-switch/opx-manifest && repo sync
```

The `repo` commands download all of the source files that are necessary to
build OpenSwitch. Binary libraries for the SAI are also required.
These binary libraries are currently *not* open source, as they are based on
the Broadcom SDK.

**Note:** These command may be run first to fix error:
```bash
# error: repo not installed yet
sudo apt install repo
```
```bash
# error: not configuration yet
git config --global user.email "you@example.com"
```


## Build packages

Build dependencies are pulled from the `unstable` distribution. To
change this, use `$OPX_RELEASE`.

```bash
# Build all repositories
opx-build/scripts/opx_run opx_build all

# Build a single repository
opx-build/scripts/opx_run opx_build opx-logging

# Build against the 2.2.1 release
OPX_RELEASE=2.2.1 opx-build/scripts/opx_run opx_build all
```
**Note:** 
- Ensure the Docker image is up to date by running docker pull opxhub/build.
- These command may be run first to fix error:
```bash
# error: deb.debian.org is outdated, so replace it with archive.debian.org
sudo find ./* -type f -exec sed -i 's/deb.debian.org/archive.debian.org/g' {} +
sudo find /var/lib/docker -type f -exec sed -i 's/deb.debian.org/archive.debian.org/g' {} +
```


# Installation

Creating an installer requires the
[opx-onie-installer](http://git.openswitch.net/cgit/opx/opx-onie-installer/)
repository. This repository is included if you cloned with `repo` and contains
the blueprints used to assemble an installer.

Any local packages you have built will be included in the installer. To exclude
them, remove the `deb` files from the repo root.

The `unstable` distribution is used to grab missing packages on
installer creation and fetch updates when running. To use a different
distribution, use the `--dist` flag.

Run `opx-build/scripts/opx_run opx_rel_pkgasm.py --help` to see the available
distributions.

```bash
opx-build/scripts/opx_run opx_rel_pkgasm.py --dist stable \
  -b opx-onie-installer/release_bp/OPX_dell_base_stretch.xml
```

# Creating the opx-build Docker image

```bash
./docker_build.sh
```

The default Docker image builds against the unstable OPX distribution. When
other distributions are requested, pbuilder chroots are created on the fly.
These chroots are lost when the container is removed, but only take 7.5sec to
create.

# Docker Image Architecture

Since `git-buildpackage` with `cowbuilder` is used to build our packages, a
pbuilder chroot is created in the image. Due to an issue with
docker/kernel/overlayfs/pbuilder, the pbuilder chroot is created by running a
privileged base container and committing it. To keep the image size small, only
one chroot is created. This chroot contains sources from the unstable OPX
release. To support building against multiple OPX releases, this chroot is
copied and modified as needed with new package sources at runtime (when the
`OPX_RELEASE` variable is used). When publishing our image, we use a tag with
the format `${sha}-${dist}`, where `${sha}` is the HEAD of this repository and
`${dist}` is the Debian distribution of the pbuilder chroot. The `latest` tag
always point to the most recently published image.

# Build Options

These environment variables enable different options.

* `OPX_GIT_TAG=yes`: after each build, tag the repository for publishing
* `OPX_RELEASE=2.2.1`: change which OPX release to build against

---

> [For older documentation, see b64c3be](https://github.com/open-switch/opx-build/blob/b64c3bedf6db0d5c5ed9fbe0e3ddcb5f4da3f525/README.md).

© 2018 Dell Inc. or its subsidiaries. All Rights Reserved.

[install-docs]: https://github.com/open-switch/opx-docs/wiki/Install-OPX-on-Dell-EMC-ON-series-platforms
