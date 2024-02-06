Developer HOWTO
===============

This document is intended for people who want to add support for new framework.

How it works
------------

There's a class called ``Builder`` (see :file:`graminescaffolding/builder.py`)
which directs the whole process of building the container. The build is composed
of steps:

1. Templates are rendered into ``.scag/`` directory.
2. Base system image is installed into temporary chroot directory and is
   archived into temporary tarball ``.scag/rootfs.tar``.
3. A base system docker is created from ``rootfs.tar`` (see
   ``Dockerfile-rootfs`` template, rendered to ``.scag/Dockerfile-rootfs``).
4. Main docker image is built (from ``Dockerfile`` template, rendered to
   ``.scag/Dockerfile``). This is the step that varies the most according to
   specific framework. ``gramine-manifest`` is run at the end of this stage.
5. Contents of this customised docker image is extracted into temporary
   directory and ``gramine-sgx-sign`` is ran *outside of the docker build
   process*.
4. Artifacts from ``gramine-sgx-sign`` (``app.manifest.sgx`` and ``app.sig``)
   are used to build third Docker container, based on the second (cusomised)
   one. The manifest and SIGSTRUCT are added to this container. This is the
   final container that can be shipped.

Templates are found in
:file:`graminescaffolding/templates/framework/{<framework>}` directory or
directly in :file:`graminescaffolding/templates`. They are rendered into files
in subdirectory :file:`.scag` placed next to :file:`scag.toml` in root directory
of the project. There are a few common templates that are rendered for every
build (like ``app.manifest.template`` and ``Dockerfile``), and frameworks may
render additional templates as required.

Step 2 is executed as single command, ``mmdebstrap``. ``mmdebstrap`` is a tool
for creating Debian-based system images. The process is customisable with
*hooks*. There are two hooks used by scaffolding:

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
:file:`/app/app.manifest{*}`    Gramine Manifest files (``.manifest.template``,
                                ``.manifest`` and ``.manifest.sgx``)
:file:`/usr/local/etc`          Templates rendered and files copied to
                                :file:`{<project_dir>}/.scag/etc`
=============================== ================================================

Builders
--------

To create a new framework, inherit from `Builder` class, then override:

- `framework` (str)
- `depends` (iterable of strings)
- `extra_files` (dict of str: iterable, str is file path relative to ``.scag/``
  magic directory, and iterable of template names, which are sequentially tried,
  until one is found)

After defining this class, you should add it to entrypoints in
:file:`pyproject.toml`.

Template variables
------------------

``scag.*``
    Dictionary with system-wide, readonly variables. Those can't be overridden
    by user-level variables, nor they should be, as they are e.g., system paths.

``scag.builder``
    Reference to the instance of `Builder`. `Builder` has useful attributes:
    `project_dir`, `scag_dir` (also `variables`, but those are
    primarily available as globals).

``scag.keys_path``
    Path to directory that ships Gramine and Intel release keys. Used in
    ``setup.sh`` hook.

``scag.magic_dir``
    Directory that contains all files generated during the build phase.
    This path is constant, so it can be safely used in a Dockerfile.

``sgx.*``
    Available as ``sgx.*`` global directory in templates. Used for
    ``sgx.sign_args``.

All values in ``[<framework>]`` section in :file:`scag.toml` are available as
global variables.

Template filters
----------------

``shquote``
    Quotes shell strings (see :py:func:`shlex.quote`). Useful in
    templates. For example, if you need a path passed to a shell command:

    .. code-block:: dockerfile

        RUN cp {{ source | shquote }} {{ destination | shquote }}

Template macros
---------------

``apt_install(package[, package2[, ...]])``
    Defined in ``Dockerfile`` template (available also if you
    ``{% extends 'Dockerfile' %}``). ``{{ apt.install('pkg1', 'pkg2', ...)``
    will emit ``RUN apt-get ...`` invocation that will correctly install the
    set of packages given as arguments.
