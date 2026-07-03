Backends
########

Backends are responsible for conversion of Sigma rules into a target query languages. Mainly, they
have to convert the conditions of the Sigma rules with their reference to detection items into
equivalent query. Backends should not be used to handle log source types or data models, e.g. field
naming or differences in value representation. Use :doc:`Processing_Pipelines` instead.

To implement a conversion for a new query language derive an appropriate backend base class from
below and override properties or methods as required.

Use the `Cookiecutter template <https://github.com/SigmaHQ/cookiecutter-pySigma-backend>`_ to start a
new backend.

Example: Converting a Rule to Splunk
************************************

.. code-block:: python

   from sigma.collection import SigmaCollection
   from sigma.backends.splunk import SplunkQueryBackend
   from sigma.pipelines.sysmon import sysmon_pipeline

   # Load rules
   rules = SigmaCollection.from_yaml_file("rules.yml")

   # Create backend with pipeline
   pipeline = sysmon_pipeline()
   backend = SplunkQueryBackend(pipeline=pipeline)

   # Convert
   queries = backend.convert(rules)
   for query in queries:
       print(query)

Available Backends
******************

See the `pySigma backends repository <https://github.com/SigmaHQ/pySigma-backends>`_ for a
complete list of official backends.

Conventions
***********

* Always implement the default output format in a way that the user does get some directly
  actionable output if she/he doesn't explicitely chooses a format.
* Don't do any concatenation of simple queries in the basic default format, the CLI or other tools
  will take care of this.
* Use backend specific options to control behavior of the backend. Backend options are provided as
  additional keyword arguments to the `__init__` constructor of a backend. They can be passed to the
  CLI with the `-O`` option. Numbers are automatically converted to int type. If a backend option is
  specified multiple times in the CLI, all values are passed as arrays.
* Don't print any output to the console or create files from the backend. Return text output as
  string or file output as bytes. The tools using your backend will take care of the proper handling
  of the result.

Concepts
********

Conversion Methods
==================

Builtin Processing Pipeline
===========================

Output Formats
==============

Rule Finalization
-----------------

Output Finalization
-------------------

Classes
*******

Backend
=======

The backend base class is generic and can generate arbitrary output, e.g. Python data structures.

.. autoclass:: sigma.conversion.base.Backend
   :members:

TextQueryBackend
================

Backend base class for conversion to text based query languages. In many cases the methods doesn't
have to be overridden but string tokens have to be defined as class variable members (tbd).

.. autoclass:: sigma.conversion.base.TextQueryBackend
   :members: