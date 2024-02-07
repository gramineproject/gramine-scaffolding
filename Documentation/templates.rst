Templates
=========

Introduction
------------

SCAG works by rendering certain files from Jinja templates, then processing the
resulting directory (containing both user's application and mentioned
renderings) into system image, measuring and signing it, and last but not least,
packaging everything into Docker (OCI) container. For more complicated
applications (e.g. those that have or had their own Dockerfiles with significant
logic), there might be need to do some postprocessing on docker images after
copying in the build artifacts. In SCAG this is done by overriding templates
that get rendered into configuration files such as Dockerfile or Gramine
manifest. This guide will show how to override the Dockerfile for
``python-plain`` framework's helloworld example.

What templates are rendered for a particular app is defined by the framework,
but there are some templates common to all frameworks, like
:file:`scag-client.toml`. Some have common rendered path, but are almost always
rendered from framework-specific template (like
:file:`.scag/app.manifest.template` rendered from
:file:`frameworks/{<framework>}/app.manifest.template`) or have a common base
template, but frameworks inherit from base template and customise it in some way
(this is the case of :file:`.scag/Dockerfile` rendered from
:file:`frameworks/{<framework>}/Dockerfile`, all of which ``{% extends
'Dockerfile' %}`` ). If you need to override a template specific to a framework,
unfortunately you need to look into SCAG source to find the exact template name.

SCAG uses `Jinja templates <https://jinja.palletsprojects.com/>`__. Introduction
to this template language is outside of scope for this document, which will
only describe concepts needed to explain, how SCAG uses those templates.

Template names, paths and inheritance
-------------------------------------

In Jinja, templates have names, which usually translate to filenames under some
preconfigured directory (though not always and not directly). An example name
would be ``Dockerfile`` or ``framework/python-flask/app.manifest.template``. To
override a template, you need to know its name beforehand. In SCAG, default
templates are stored in :file:`{platlib}/graminescaffolding/templates` (on
Debian-derived system it might be
:file:`/usr/lib/python3/dist-packages/graminescaffolding/templates`). You should
not change the default templates, your changes will be lost when SCAG is
updated.

To define your own templates, you need to specify your own directory with
templates (usually :file:`templates/` subdirectory of your project directory)
and provide its name in ``application.templates`` setting in :file:`scag.toml`.
Example from bootstrap contains this line commented, so you probably need to
uncomment it and ``mkdir`` this directory.

Templates which are present in project's ``templates/`` override completely
templates that have the same name (i.e. are under the same subpath in default
directory). You need to either provide all the template content (even the parts
that you don't need to change), or you can *inherit* from one of the default
templates. If you need to inherit template that has the same name, in SCAG you
can add ``!`` character to the template name in ``{% extends %}`` statement:

.. code-block:: jinja

    {% extends '!Dockerfile %}

Just writing ``{% extends 'Dockerfile' %}`` (without ``!``) is an error, because
it causes recursion in inheritance resolution.

When you are inheriting from existing template, you can override any number of
blocks. Read the original template to see what you can change there. Previous
contents of the block are available as ``{{ super() }}``.

Choosing the template to override
---------------------------------

Here's a non-exhaustive list of files used by SCAG and templates from which they
are rendered (and which we will be overriding):

- File ``.scag/Dockerfile`` is responsible for creating initial (not signed)
  image. Rendered from ``framework/<framework>/Dockerfile`` or ``Dockerfile``
  template (in the root directory of respective templates directory).

- File ``.scag/Dockerfile-final`` is responsible for replacing
  ``app.manifest.sgx`` and ``app.sig`` (SIGSTRUCT) inside the container.
  Rendered from ``Dockerfile-final``. It is not advisable to override this
  template.

- File ``.scag/app.manifest.template``, which ends up in the container as
  ``/app/app.manifest.template``. Rendered from either
  ``frameworks/<framework>/app.manifest.template`` or (if that template is not
  provided by framework), then ``app.manifest.template``. This is the Gramine
  manifest template.

  .. important::

      You need to know if the framework uses its own manifest template, because
      if it does, you need to override the framework-specific template.
      Overriding global template won't work.

Example
-------

The following file can be placed in
:file:`{<project_dir>}/templates/frameworks/python_plain/Dockerfile`
to change the message printed by ``hello_world.py`` script in demo app from
``python_plain``:

.. code-block:: jinja

    {% extends '!frameworks/python_plain/Dockerfile' %}

    {% block build %}
    {{ super() }}

    RUN sed -i -e s/world/asdfg/ /app/hello_world.py
    {% endblock %}

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
    Defined in ``Dockerfile`` template (available if you
    ``{% extends 'Dockerfile' %}``). ``{{ apt_install('pkg1', 'pkg2', ...)``
    will emit ``RUN apt-get ...`` invocation that will correctly install the
    set of packages given as arguments.
