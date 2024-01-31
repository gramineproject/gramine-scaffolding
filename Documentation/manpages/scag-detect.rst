.. program:: scag-detect
.. _scag-detect:

**********************************************************
:program:`scag-detect` -- Gramine Scaffolding autodetector
**********************************************************

Synopsis
========

| :command:`scag detect` [*OPTIONS*]
| :command:`scag-detect` [*OPTIONS*]

Description
===========

This program detects whether SGX is properly configured on the machine. If not,
will report 

Options
=======

.. option:: --quiet

    Suppress printing error message to standard error on SGX misconfiguration.
    The only result is the exit code. The default is to print diagnostics.

.. option:: --verbose

    Re-enable printing error messages, possibly disabled with :option:`--quiet`.
    This is the default.

Exit status
===========

0
    When SGX works correctly.

1
    When something is broken. Diagnostics are printed to standard error, unless
    :option:`--quiet` is in effect.


See also
========

`is-sgx-available
<https://gramine.readthedocs.io/en/stable/manpages/is-sgx-available.html>`__
