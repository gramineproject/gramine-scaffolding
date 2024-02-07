Developer HOWTO
===============

This document is intended for people who want to add support for new framework.

How it works
------------

There's a class called ``Builder`` (see :file:`graminescaffolding/builder.py`)
which directs the whole process of building the container. The build is composed
of steps:

1. Templates are rendered. Templates are found in
   :file:`graminescaffolding/templates/framework/{<framework>}` directory or
   directly in :file:`graminescaffolding/templates`. They are rendered into
   files in subdirectory :file:`.scag` placed next to :file:`scag.toml` in root
   directory of the project. There are a few common templates that are rendered
   for every build (like :file:`app.manifest.template` and :file:`Dockerfile`),
   and frameworks may render additional templates as required.

   Most of the documentation about templates is in :doc:`templates`. That
   document is aimed mostly at people who customise existing templates through
   application-specific overrides, but if you're developing new framework,
   you'll probably ship your templates in `graminescaffolding/templates`.

2. Base system image is installed into temporary chroot directory and is
   archived into temporary tarball :file:`.scag/rootfs.tar`. “Base system”
   consists of vanilla Debian image, on top of which Gramine package is
   installed. Gramine installed inside this chroot is required to be exactly the
   same version as the one installed. See ``create_chroot()`` method in
   ``builder.py`` file.

3. A base system Docker image is created from :file:`rootfs.tar` (see
   :file:`Dockerfile-rootfs` template, rendered to
   :file:`.scag/Dockerfile-rootfs`).

4. Main docker image is built (with :file:`framework/{<framework>}/Dockerfile`
   template, rendered to :file:`.scag/Dockerfile`). It's derived ``FROM`` the
   base system image created in step 3.

   This is the step that varies the most according to specific framework.
   ``gramine-manifest`` is run at the end of this stage.

5. Contents of this customised docker image are extracted into temporary
   directory and ``gramine-sgx-sign`` is ran *outside of the docker build
   process*. (This is because if user has signing plugins configured on the
   workstation, they can't easily be executed inside Docker).

6. Artifacts from ``gramine-sgx-sign`` (:file:`app.manifest.sgx` and
   :file:`app.sig`) are used to build third Docker image, based on the
   second (customised) one. The manifest and SIGSTRUCT are added to this
   image. This is the final image that can be shipped.

Step 2 is executed via ``mmdebstrap``. ``mmdebstrap`` is a tool for creating
Debian-based system images. The process is customisable with *hooks*. There are
two hooks used by scaffolding:

- ``setup`` (rendered as :file:`.scag/mmdebstrap-hooks/setup.sh`): Runs very
  early in the bootstrap process and prepares installation environment. Probably
  should not be touched unless you need to pin some packages for reproducible
  builds.

- ``customize`` (rendered as :file:`.scag/mmdebstrap-hooks/customize.sh`): Runs
  after all deb packages have been installed. Normally not needed, all
  customisation happens in ``Dockerfile``, but might be used for exotic setups
  which need to be cached in ``rootfs.tar`` instead of Docker layers.

All hooks are executed with the first argument being the path to temporary
chroot directory, so if you need to run something inside chroot, you should
prefix your command with ``chroot "$1"``, like ``chroot "$1" gramine-manifest
...``. If you don't want to do that (for example, to copy files from outside),
you obviously shouldn't (``cp /path/to/source "$1"/path/inside/chroot``). Please
use absolute paths.

Expected filesystem layout
--------------------------

=============================== ================================================
path                            contents
=============================== ================================================
:file:`/`                       Base system (Debian 12) with gramine installed
                                from packages.
:file:`/app`                    Full contents of the app repository. Also
                                Docker's ``WORKDIR``.
:file:`/app/app.manifest{*}`    Gramine Manifest files
                                (:file:`.manifest.template`, :file:`.manifest`
                                and :file:`.manifest.sgx`)
:file:`/usr/local/etc`          Templates rendered and files copied to
                                :file:`{<project_dir>}/.scag/etc`
=============================== ================================================

Builders
--------

To create a new framework, inherit from `Builder` class, then override:

- `framework` (str)
- `extra_files` (dict of str: iterable, str is file path relative to
  :file:`.scag/` magic directory, and iterable of template names, which are
  sequentially tried, until one is found)

After defining this class, you should add it to entrypoints in
:file:`pyproject.toml`.
