Quickstart
##########

This guide gets you up and running with pySigma in 5 minutes.

Prerequisites
*************

* Python 3.10 or higher
* pip or poetry

Installation
************

.. code-block:: bash

   pip install pysigma

Or with poetry:

.. code-block:: bash

   poetry add pysigma

Your First Sigma Rule
*********************

Load a Sigma rule from YAML:

.. code-block:: python

   from sigma.collection import SigmaCollection
   from sigma.backends.test import TextQueryTestBackend

   # Minimal Sigma rule
   sigma_rule = """
   title: Test Rule
   logsource:
       category: process_creation
       product: windows
   detection:
       selection:
           CommandLine|startswith: 'cmd.exe'
       condition: selection
   """

   # Load the rule
   rules = SigmaCollection.from_yaml(sigma_rule)

   # Convert to query (using test backend)
   backend = TextQueryTestBackend()
   queries = backend.convert(rules)

   print("\\n".join(queries))

Expected output:

.. code-block:: text

   CommandLine startswith 'cmd.exe'

With a Real Backend (e.g., Splunk)
***********************************

.. code-block:: python

   from sigma.collection import SigmaCollection
   from sigma.backends.splunk import SplunkQueryBackend

   # Load multiple rules from a file
   rules = SigmaCollection.from_yaml_file("rules.yml")

   # Splunk backend with Sysmon pipeline
   from sigma.pipelines.sysmon import sysmon_pipeline
   pipeline = sysmon_pipeline()
   backend = SplunkQueryBackend(pipeline=pipeline)

   # Convert
   queries = backend.convert(rules)

   for q in queries:
       print(q)

With a Custom YAML Pipeline
****************************

.. code-block:: python

   from sigma.processing.pipeline import ProcessingPipeline

   # Load a pipeline from a YAML file
   with open("my_pipeline.yml") as f:
       pipeline = ProcessingPipeline.from_yaml(f.read())

   # Use the pipeline with a backend
   backend = ElasticsearchQueryBackend(pipeline=pipeline)

Next Steps
**********

* :doc:`YAML_Pipeline_Tutorial` — Learn how to create your own YAML pipelines
* :doc:`Sigma_Rules` — Understand Sigma rule structure
* :doc:`Processing_Pipelines` — Complete reference for transformations
* :doc:`Backends` — List of available backends
