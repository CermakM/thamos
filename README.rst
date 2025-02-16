Thamos
------

A CLI tool and library for communicating with Thoth backend.


Using Thamos as a CLI tool
==========================

Thamos is released on `PyPI <https://pypi.org/project/thamos>`_. See
installation instructions bellow to setup Thoth/Thamos for your repository:

.. code-block:: console

  # Install Thamos CLI tool:
  $ pip3 install thamos  # keep in mind: requires Python 3.6!!
  # Go to repository that should be managed by Thoth:
  $ cd ~/git/repo/
  # Setup Thamos configuration:
  $ thamos config
  # Ask Thoth for software stack recommendations:
  $ thamos advise


Adjusting configuration based on environment variables
======================================================

You can adjust content of configuration file each time Thamos CLI or Thamos
library loads it by expanding entries with environment variables. This can be
handy if you would like to parameterize some of the options at
runtime (e.g. in deployment).

This behaviour is (due to security reasons) explicitly turned off by default.
However you can turn it on by setting `THAMOS_CONFIG_EXPAND_ENV` environment
variable to `1` (`0` explicitly turns this behaviour off, default value):


.. code-block:: console

    THOTH_HOST=test.thoth-station.ninja THAMOS_CONFIG_EXPAND_ENV=1 thamos advise
    2019-03-13 11:22:59,562 [18639] INFO     thamos.config: Expanding configuration file based on environment variables

Entries which should be expanded have environment variables in curly braces
like the following example:

.. code-block:: yaml

   host: {THOTH_HOST}


Note the expansion is done by replacing these values directly with values of
environment variable, this means types need to be taken into account
(environment variable with value `"true"` is put into configuration file as
`true`).


Using custom configuration file template
========================================

You can use your own custom configuration file as a template. This is
especially useful if you want to have some configuration entries constant and
let expand only some of the configuration options. In other words, you can
parametrize configuration file.

An example of configuration file template can be:

.. code-block:: yaml

  host: {THOTH_SERVICE_HOST}
  tls_verify: true
  requirements_format: pipenv

  runtime_environments:
    - name: '{os_name}:{os_version}'
      operating_system:
        name: {os_name}
        version: '{os_version}'
      hardware:
        cpu_family: {cpu_family}
        cpu_model: {cpu_model}
      python_version: '{python_version}'
      cuda_version: {cuda_version}
      recommendation_type: stable
      limit_latest_versions: null

Then, you need to supply this configuration file to the following command:

.. code-block:: console

  thamos config --template template.yaml

Listing of automatically expanded configuration options which are supplied the
config sub-command (these options are optional and will be expanded based on HW
or SW discovery):

+------------------------+--------------------------------+----------+
| Configuration option   | Explanation                    | Example  |
+========================+================================+==========+
| `os_name`              | name of operating system       | fedora   |
+------------------------+--------------------------------+----------+
| `os_version`           | version of operating system    |  30      |
+------------------------+--------------------------------+----------+
| `cpu_family`           | CPU family identifier          |  6       |
+------------------------+--------------------------------+----------+
| `cpu_model`            | CPU model identifier           |  94      |
+------------------------+--------------------------------+----------+
| `python_version`       | Python version (major.minor)   |  3.6     |
+------------------------+--------------------------------+----------+
| `cuda_version`         | CUDA version (major.minor)     |  9.0     |
+------------------------+--------------------------------+----------+

These configuration options are optional and can be mixed with adjustment based
on environment variables (see `THOTH_SERVICE_HOST` example above). Note the
environment variables are not expanded on `thamos config` call but rather on
other sub-commands issued (e.g. `thamos advise` or others).

Using Thoth and thamos in OpenShift's s2i
=========================================

Using configuration templates is especially useful for OpenShift builds where
you can specify your template in an s2i repository (omit `Pipfile.lock` to
enable call to `thamos advise` as shown in `this repository
<https://github.com/thoth-station/s2i-thoth-example>`_).

Then, you need to provide following environment variables:

* `THAMOS_CONFIG_TEMPLATE` - holds path to template - use `/tmp/src` prefix to point to root of s2i repository (e.g. `/tmp/src/template.yaml` if `template.yaml` is the configuration template and is stored in root of your Git repository)
* `THAMOS_NO_INTERACTIVE` - set to `1` if you don't want to omit interactive thamos (suitable for automated s2i builds happening in the cluster)
* `THOTH_SERVICE_HOST` - set to host of Thoth backend you would like to talk to, applicable only you use expansion based on environment variables as shown in the example above
* `THAMOS_NO_PROGRESSBAR` - set to `1` to disable progressbar while waiting for response from Thoth backend - it can cause annoying too verbose output printed to OpenShift console during the build

Using Thamos as a library
=========================


.. code-block:: python

   from thamos.lib import image_analysis
   from thamos.config import config

   # Set global context.
   # Host to Thoth's User API. API discovery will be done
   # transparently and the most appropriate API version will be used.
   config.explicit_host = "thoth-user-api.redhat.com"
   # TLS verification when communicating with Thoth API.
   config.tls_verify = True

   image_analysis(
     image="registry.redhat.com/fedora:29",
     registry_user="fridex",
     registry_password="secret!",
     # TLS verification when communicating with registry.
     verify_tls=True,
     nowait=False
   )

Disabling TLS related warnings
==============================

If you communicate with Thoth's user API without TLS (you have set the
``tls_verify`` configuration option to ``false`` in the ``.thoth.yaml`` file),
Thamos CLI and Thamos library issue a warning each time there is done
communication with the API server. To suppress this warning, set the
``THAMOS_DISABLE_TLS_WARNING`` environment variable to a non-zero value:

.. code-block:: console

  $ export THAMOS_DISABLE_TLS_WARNING=1
  $ thamos advise

Autogenerated client from OpenAPI
=================================

Most parts of Thamos consist of automatic generated code. You can update Thamos
by running the following command:

.. code-block:: console

  $ ./swagger-codegen.sh

The command above will download and run automatic code generation tool against
the most recent OpenAPI specification of `User API
<https://github.com/thoth-station/user-api/>`_. Results of the tool are
automatically placed into this repository in `thamos/swagger_client/` and
`Documentation/`. They consist of automatically generated code as well as
`documentation on how to use the code
<https://github.com/thoth-station/thamos/tree/master/Documentation>`_.  Thamos
itself provides routines built on top of this automated generated code to
simplify usage in `thamos/lib`.

