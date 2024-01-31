.. highlight:: toml

Configuration
=============

SCAG is configured in file:`scag.toml`, which should be placed in main directory
of the app and committed to the repository.

See https://toml.io/ for the description of the TOML format.

The following configuration knobs are available:

General options
---------------

``application.framework`` (string)
    Framework. One of:

    - ``dotnet``
    - ``expressjs``
    - ``flask``
    - ``java_gradle``
    - ``java_jar``
    - ``koajs``
    - ``nodejs_plain``
    - ``python_plain``

``application.templates`` (string)
    Path to directory with extra templates, relative to project directory. Any
    files in this directory will take precedence over templates in SCAG's global
    directory.

``sgx.sign_args`` (array of strings)
    Extra arguments to :command:`gramine-sgx-sign` command. Can be used to
    specify alternative RSA key, or use plugins.

    .. code-block::

        [sgx]
        sign_args = ['--key', 'example.pem']

``sgx.debug`` (bool, default false)
    INSECURE. Build debug enclave. Debug enclaves do not give any security
    guarantees and should fail attestation, but can be used for debugging
    running application.

    This option mirrors ``sgx.debug`` option in Gramine manifest.

``sgx.remote_attestation`` (string)
    Chooses remote attestation, or disables it. One of:

    - ``dcap`` (the default)
    - ``epid``
    - ``none``

    This option mirrors ``sgx.remote_attestation`` option in Gramine manifest.

    For EPID, you need to also add two options to Gramine manifest:
    ``sgx.ra_client_spid`` and ``sgx.ra_client_linkable``. Those options cannot
    be added in :file:`scag.toml`, you need to add them by overriding manifest
    template as described in :doc:`templates`.

    For MAA, leave the default of ``dcap``.

.. please keep this list sorted lexicographically

Options specific to ``dotnet`` framework
-------------------------------------------

``dotnet.build_config`` (string)
    Build configuration

``dotnet.project_file`` (string)
    Path to the application's main project file inside application's directory.

``dotnet.target`` (string)
    Path to the application's binary inside application's directory.

Options specific to ``expressjs`` framework
-------------------------------------------

``expressjs.application`` (string)
    Path to the main script inside application's directory.

Options specific to ``flask`` framework
----------------------------------------------

(none)

Options specific to ``java_gradle`` framework
------------------------------------------

``java_gradle.application`` (string)
    Path to the JAR file inside application's directory.

Options specific to ``java_jar`` framework
------------------------------------------

``java_jar.application`` (string)
    Path to the JAR file inside application's directory.

Options specific to ``koajs`` framework
---------------------------------------

``koajs.application`` (string)
    Path to the main script inside application's directory.

Options specific to ``nodejs_plain`` framework
----------------------------------------------

``nodejs_plain.application`` (string)
    Path to the main script inside application's directory.

Options specific to ``python_plain`` framework
----------------------------------------------

``python_plain.application`` (string)
    Path to the main script inside application's directory.
