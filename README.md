TL;DR
=====

1- spinoff a VM
 * n1-highcpu-4 (4 vCPUs, 3.6 GB memory)
 * 25 GB SSD
 * Ubuntu 14.04 LTS

2- `sudo apt-get install git make gcc g++ m4`

3- `git clone https://github.com/edgarcosta/binary-pkg.git`

4-  `cd binary-pkg`

5- edit sage.yaml line 4 accordingly

6- In a screen `cd binary-pkg && make clean && make bdist-sage-linux` and wait  `5+` hours

7 - Now on your local machine, copy the tar and then reupload to `grace.mit.edu` at `/home/lmfdb/sage-lmfdb/'

Original TL;DR
=====

To build Sage binaries, just clone this repository and run

    git clone https://github.com/edgarcosta/binary-pkg.git
    cd binary-pkg
    make bdist-sage-linux     # If you are on Linux
    make bdist-sage-osx       # If you are on OSX
    ls dist/                  # Built binaries will be in this directory

Edit `sage.yaml` if you want to build something else than the current
Sage `develop` git branch.


Binary Packaging
================

This utility helps with creating binaries for distribution. Similar to
conda, it builds the source tree in a long directory name which is
then patched at install time. It is written for Sage
(http://www.sagemath.org) but equally work for other software.


Usage
-----

Building and packaging of an application is configured with a yaml
file in the repository root. As an example, there is test.yaml and
sage.yaml. To create a binary tarball for Sage, for example, all you
have to do is run

    make package-sage

This creates a tarball with an added ``relocate-once.py`` file that
patches any hard-coded paths. If there are more than one packaging
configurations (see below), then the first one is used by default. Use
the ``PACKAGE`` variable to pick another one, for example

    make package-sage PACKAGE="OSX DMG image"

In particular, see below for the different Mac packaging possibilities.

Configuration Syntax
--------------------

Binary packaging information can be specified with a YAML
configuration file. As a simplified example, let us look at the test
application in the test.yaml file. It starts with the name:

    name: PackagingTest

which will be the name of the root directory in the binary
tarball. Our basic assumption is that your source code lives in a git
repository, which we specify next:

    repository: https://github.com/octol/minimal-gtest-autotools
    branch: master

Then we have a (bash) build script to build the application

    build: |
        autoreconf -vfi
        ./configure
        make

After the build is complete, we need to know the version. Typically
this can be gotten via a command line switch of the application,
though in this case we cheat. In any case, the output of this script
is the version:

    version: |
        echo 1.0

Finally, we define how to package the built source tree. There may be
more than one way to package the application, each of which is
distinguished by an internal name.

    package:
      - name: Full binary tarball
        command: |
            tar cjf {dist}/test-{version}-{osname}-{arch}.tar.bz2 {path}
        files:
          - include: '**'
        rewrite_path:
          - exclude: '**/*.a'
    
The `files` section is a list of include/exclude directives, to be
read from the bottom up. That is, later directives override earlier
ones. The `rewrite_path` section defines a subset of the files that
are to be ignored when rewriting hard-coded paths. In this example,
all files are included but paths in static archives are not patched.


Auxiliary Make Targets
----------------------

Any ``*.mk`` file in the root directory will be included in the
``Makefile`` for recurring build targets.

Mac Packaging Options
---------------------

There are three options for packaging Mac.  The default will simply
create a ``.tar.gz`` file, which is usually not what one wants on Mac.

To create a Mac disk image file of the normal Sage command line distribution,
use this package option:

    make package-sage PACKAGE="OSX DMG image"

To create a Mac disk image file of the Mac app bundle with menu items and
which automatically launches a notebook, use

    make package-sage PACKAGE="OSX mac app"
