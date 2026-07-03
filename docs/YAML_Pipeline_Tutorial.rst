YAML Pipeline Tutorial
######################

This tutorial teaches you how to create YAML pipelines to adapt pySigma to your environment.

What is a YAML Pipeline?
************************

A YAML pipeline is a file that describes how to transform Sigma rules before conversion.
It is used to map fields, modify values, add conditions, etc.

Basic Structure
***************

A minimal pipeline:

.. code-block:: yaml

    name: My Pipeline
    priority: 10
    transformations:
      - type: field_name_mapping
        mapping:
          EventID: EventCode

Explanation:

* ``name`` : pipeline name (informational)
* ``priority`` : order of application (lowest first, default = 0)
* ``transformations`` : list of operations to apply

Loading a Pipeline
******************

From Python:

.. code-block:: python

    from sigma.processing.pipeline import ProcessingPipeline

    # From a YAML string
    pipeline = ProcessingPipeline.from_yaml(yaml_string)

    # From a file
    with open("pipeline.yml") as f:
        pipeline = ProcessingPipeline.from_yaml(f.read())

    # From a dictionary
    pipeline = ProcessingPipeline.from_dict({
        "name": "My Pipeline",
        "priority": 10,
        "transformations": [
            {
                "type": "field_name_mapping",
                "mapping": {"EventID": "EventCode"}
            }
        ]
    })

Using the Resolver (multi-pipeline):

.. code-block:: python

    from sigma.processing.resolver import ProcessingPipelineResolver

    resolver = ProcessingPipelineResolver()
    resolver.add_pipeline_class(my_pipeline)  # registered pipeline

    # Resolve a single pipeline
    pipeline = resolver.resolve_pipeline("my_pipeline")

    # Resolve and merge multiple pipelines
    consolidated = resolver.resolve([
        "pipeline1.yml",
        "pipeline2.yml",
        "my_pipeline"
    ])

Progressive Examples
********************

Example 1: Simple Mapping
==========================

Map fields from one log source to another:

.. code-block:: yaml

    name: Sysmon to Elastic
    priority: 10
    transformations:
      - type: field_name_mapping
        mapping:
          EventID: event_id
          CommandLine: command_line
          ParentImage: parent_process
          Image: process

Example 2: Mapping with Conditions
===================================

Apply mapping only for certain log sources:

.. code-block:: yaml

    name: Sysmon conditional mapping
    priority: 10
    transformations:
      - type: field_name_mapping
        mapping:
          EventID: event_id
        rule_conditions:
          - type: logsource
            product: windows
            service: sysmon

Example 3: Adding Suffixes/Prefixes
====================================

.. code-block:: yaml

    name: Add field prefixes
    priority: 20
    transformations:
      - type: field_name_prefix
        prefix: "sysmon."
      - type: field_name_suffix
        suffix: "_raw"

Example 4: Post-processing
============================

Transform the generated query for Splunk:

.. code-block:: yaml

    name: Splunk search command
    priority: 50
    postprocessing:
      - type: simple_template
        template: |
          search {query} | fields - _raw
        query_field: "_raw"

Example 5: Finalizers
=======================

Merge multiple queries into a single output:

.. code-block:: yaml

    name: Concatenate queries
    priority: 60
    finalizers:
      - type: concat
        separator: " OR "
        prefix: "("
        suffix: ")"

Example 6: Variables
=====================

Use variables in transformations:

.. code-block:: yaml

    name: Pipeline with variables
    priority: 10
    vars:
      target_field: "command_line"
      search_string: "powershell"
    transformations:
      - type: regex
        field: "{vars[target_field]}"
        regex: "(?i){vars[search_string]}"

Example 7: Nested Pipeline (nest)
===================================

Group multiple transformations:

.. code-block:: yaml

    name: Nested pipeline
    priority: 10
    transformations:
      - type: nest
        items:
          - type: field_name_mapping
            mapping:
              EventID: event_id
          - type: field_name_prefix
            prefix: "sysmon."
          - type: set_state
            state: mapped

Example 8: Real-world Sysmon to Splunk
========================================

Complete pipeline to adapt Sigma Sysmon rules to Splunk:

.. code-block:: yaml

    name: Sysmon to Splunk
    priority: 10
    allowed_backends:
      - splunk

    transformations:
      # Map Sysmon fields to Splunk field names
      - type: field_name_mapping
        mapping:
          EventID: EventCode
          Image: Process
          CommandLine: CommandLine
          ParentImage: ParentProcess
          ParentCommandLine: ParentCommandLine
          User: User
          LogonId: LogonId
          Hashes: Hash

      # Add Splunk-specific fields
      - type: add_field
        field: EventType
        value: ProcessCreate

      # Condition: apply only to Sysmon rules
      - type: set_state
        state: sysmon_mapped
        rule_conditions:
          - type: logsource
            service: sysmon

    postprocessing:
      # Format for Splunk search command
      - type: simple_template
        template: |
          index=os (EventCode="{query}") | fields Process, CommandLine, User
        query_field: "_raw"

Best Practices
**************

1. **Naming**: Use descriptive names for your pipeline
2. **Priority**:
   - 10: Log source pipelines (Sysmon, etc.)
   - 20: Pipelines provided by backends
   - 50: Pipelines integrated in backend
   - 60: Output format pipelines
3. **Tracking**: Use ``id`` in each transformation for debugging
4. **Conditions**: Use ``rule_conditions`` to apply transformations conditionally
5. **Testing**: Test your pipeline with a simple rule before using it in production

Debugging
*********

To see which transformations are applied:

.. code-block:: python

    from sigma.processing.pipeline import ProcessingPipeline
    from sigma.processing.tracking import ItemTrackingMode

    pipeline = ProcessingPipeline.from_yaml(yaml_string)

    # Enable tracking
    pipeline.set_tracking_mode(ItemTrackingMode.TRACK_ALL)

    # Apply to a rule
    processed_rule = pipeline.apply(rule)

    # View history
    print(processed_rule.tracking_data)

Similarities with Sigma
************************

Conditions in YAML pipelines use the same syntax as Sigma rules:

* ``logsource`` : filter by log source
* ``contains_detection_item`` : check for presence of an item
* ``processing_state`` : check processing state
* etc.

See :doc:`Processing_Pipelines` for the complete reference.
