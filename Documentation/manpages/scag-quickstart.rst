.. program:: scag-quickstart
.. _scag-quickstart:

***********************************************************************
:program:`scag-quickstart` -- Bootstrap Gramine Scaffolding application
***********************************************************************

Synopsis
========

| :command:`scag quickstart`
| :command:`scag-quickstart`

Description
===========

This tool will guide interactively user for each step of application
Scaffolding:

- setup (``scag-setup``),
- build (``scag-build``),
- run (``docker run``).

The goal is to speed up the Scaffolding process for user, and get their
application up and running as fast as possible. Later on, users might want to do
more advanced configuration using :doc:`scag-setup`. After changing the
application, there is no need to run quickstart again; users can simply rebuild
the project using :doc:`scag-build`.

Usage for Human Interaction
===========================

:command:`scag-quickstart` is designed to be interactive and is intended for use
by humans who want a guided and hands-on experience while setting up their
Gramine Scaffolding application. It offers step-by-step instructions, making the
scaffolding process quick and easy.

Please note that the interface for :command:`scag-quickstart` may be unstable
and subject to variations between different versions of the tool.

For Automation and Advanced Configuration
=========================================

If developers wish to automate the scaffolding process or perform more advanced
configuration, it is recommended to use the :doc:`scag-setup`, and
:doc:`scag-build` tools. These tools provide programmatic interfaces and more
extensive options for customization.

Example
=======

::

    % scag-quickstart
    Which framework you want to use?
    1 - dotnet
    2 - expressjs
    3 - flask
    4 - java_gradle
    5 - java_jar
    6 - koajs
    7 - nodejs_plain
    8 - python_plain
    Please provide a number or name: 3
    Your's app directory is [/home/woju/tmp/flask-app]:
    The project directory seems to be empty. Do you want to bootstrap the framework example? [y/N]: y
    Do you want to build it now? [y/N]: y
    I: chroot architecture amd64 is equal to the host's architecture
    I: finding correct signed-by value...
    done
    I: automatically chosen format: tar
    I: using /tmp/mmdebstrap.GDka0a_ioG as tempdir
    I: running --setup-hook in shell: sh -c 'sh /home/woju/tmp/flask-app/.scag/mmdebstrap-hooks/setup.sh "$@"' exec /tmp/mmdebstrap.GDka0a_ioG
    I: running apt-get update...
    done
    I: downloading packages with apt...
    done
    I: extracting archives...
    done
    I: installing essential packages...
    done
    I: installing remaining packages inside the chroot...
    done
    done
    I: running --customize-hook in shell: sh -c 'sh /home/woju/tmp/flask-app/.scag/mmdebstrap-hooks/customize.sh "$@"' exec /tmp/mmdebstrap.GDka0a_ioG
    I: cleaning package lists and apt cache...
    done
    done
    I: creating tarball...
    I: done
    I: removing tempdir /tmp/mmdebstrap.GDka0a_ioG...
    I: success in 58.0602 seconds
    Attributes (required for enclave measurement):
        size:        0x10000000
        edmm:        False
        max_threads: 4
    SGX remote attestation:
        DCAP/ECDSA
    Memory:
        000000000ff73000-0000000010000000 [REG:R--] (manifest) measured
        000000000ff53000-000000000ff73000 [REG:RW-] (ssa) measured
        000000000ff4f000-000000000ff53000 [TCS:---] (tcs) measured
        000000000ff4b000-000000000ff4f000 [REG:RW-] (tls) measured
        000000000ff0b000-000000000ff4b000 [REG:RW-] (stack) measured
        000000000fecb000-000000000ff0b000 [REG:RW-] (stack) measured
        000000000fe8b000-000000000fecb000 [REG:RW-] (stack) measured
        000000000fe4b000-000000000fe8b000 [REG:RW-] (stack) measured
        000000000fe3b000-000000000fe4b000 [REG:RW-] (sig_stack) measured
        000000000fe2b000-000000000fe3b000 [REG:RW-] (sig_stack) measured
        000000000fe1b000-000000000fe2b000 [REG:RW-] (sig_stack) measured
        000000000fe0b000-000000000fe1b000 [REG:RW-] (sig_stack) measured
        000000000fdb1000-000000000fe00000 [REG:R-X] (code) measured
        000000000fe01000-000000000fe0b000 [REG:RW-] (data) measured
        0000000000010000-000000000fdb1000 [REG:RWX] (free)
    Measurement:
        e5591ce086b619f4d24ed5b716a0cb7c341ba2fbbe67caafca41717cbb86cd07
    Your new docker image sha256:88a9bf887e1e5e8e4a9bfeb09d450e06fe051a08b1a61d4816941d31efcb9d0b
    You can run it using command:
    docker run --device /dev/sgx_enclave --volume /var/run/aesmd/aesm.socket:/var/run/aesmd/aesm.socket sha256:88a9bf887e1e5e8e4a9bfeb09d450e06fe051a08b1a61d4816941d31efcb9d0b
    Do you want to run it now? [y/N]: y
    Gramine is starting. Parsing TOML manifest file, this may take some time...
    -----------------------------------------------------------------------------------------------------------------------
    Gramine detected the following insecure configurations:

    - sys.insecure__allow_eventfd = true         (host-based eventfd is enabled)

    Gramine will continue application execution, but this configuration must not be used in production!
    -----------------------------------------------------------------------------------------------------------------------
