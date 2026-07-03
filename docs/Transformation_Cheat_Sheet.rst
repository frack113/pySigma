Transformation Cheat Sheet
##########################

This cheat sheet helps you find the right transformation for common use cases.

Field Operations
****************

.. list-table::
   :header-rows: 1
   :widths: 35 25 40

   * - Need
     - Transformation
     - Example YAML
   * - Rename a field
     - ``field_name_mapping``
     - ``mapping: {EventID: event_id}``
   * - Add prefix to all fields
     - ``field_name_prefix``
     - ``prefix: "sysmon."``
   * - Add suffix to all fields
     - ``field_name_suffix``
     - ``suffix: "_raw"``
   * - Map field A to B, C, D (OR)
     - ``field_name_mapping``
     - ``mapping: {OldField: [FieldB, FieldC, FieldD]}``
   * - Apply function to field name
     - ``field_name_transform``
     - ``function: "lower"``

Value Operations
****************

.. list-table::
   :header-rows: 1
   :widths: 30 25 45

   * - Need
     - Transformation
     - Example YAML
   * - Replace exact values
     - ``map_string``
     - ``mapping: {True: "enabled", False: "disabled"}``
   * - Replace with regex
     - ``replace_string``
     - ``regex: "foo" replacement: "bar"``
   * - Convert type (str→int, etc.)
     - ``convert_type``
     - ``target_type: "int"``
   * - Set specific value
     - ``set_value``
     - ``value: "custom_value"``
   * - Hash a field (MD5, SHA256, etc.)
     - ``hashes_fields``
     - ``hash_algos: ["md5", "sha256"]``
   * - Apply case transformation
     - ``case``
     - ``case: "lower"``

Condition Operations
********************

.. list-table::
   :header-rows: 1
   :widths: 30 25 45

   * - Need
     - Transformation
     - Example YAML
   * - Add detection condition
     - ``add_condition``
     - ``conditions: ["selection"]``
   * - Change log source
     - ``change_logsource``
     - ``logsource: {product: "windows", service: "sysmon"}``
   * - Add new field to rule
     - ``add_field``
     - ``field: "custom_field", value: "value"``
   * - Remove field from rule
     - ``remove_field``
     - ``field: "unwanted_field"``
   * - Set field value
     - ``set_field``
     - ``field: "event_category", value: "process"``

State & Tracking
****************

.. list-table::
   :header-rows: 1
   :widths: 30 25 45

   * - Need
     - Transformation
     - Example YAML
   * - Mark rule as processed
     - ``set_state``
     - ``state: "mapped"``
   * - Track applied transformations
     - ``set_state``
     - Use with ``rule_conditions`` to check state

Advanced Operations
*******************

.. list-table::
   :header-rows: 1
   :widths: 30 25 45

   * - Need
     - Transformation
     - Example YAML
   * - Group multiple transformations
     - ``nest``
     - See :doc:`YAML_Pipeline_Tutorial` for details
   * - Apply regex to field
     - ``regex``
     - ``regex: "pattern" field: "target_field"``
   * - Drop detection item
     - ``drop_detection_item``
     - ``condition: {type: "field_name", field: "old_field"}``
   * - Convert hash fields
     - ``hashes_fields``
     - ``hash_algos: ["md5"], field_prefix: "file."``
   * - Add wildcard placeholders
     - ``wildcard_placeholders``
     - ``field: "command_line"``
   * - Add value list placeholders
     - ``value_placeholders``
     - ``field: "status_code"``

Post-Processing Operations
**************************

.. list-table::
   :header-rows: 1
   :widths: 30 25 45

   * - Need
     - Transformation
     - Example YAML
   * - Wrap query in template
     - ``simple_template``
     - ``template: "search {query}"``
   * - Custom query template
     - ``template``
     - See :doc:`YAML_Pipeline_Tutorial` for details
   * - Embed query in JSON
     - ``json``
     - ``template: '{"query": "{query}"}'``
   * - Replace parts of query
     - ``replace``
     - ``regex: "old" replacement: "new"``
   * - Embed raw query
     - ``embed``
     - ``prefix: "[", suffix: "]"``

Output Finalization
*******************

.. list-table::
   :header-rows: 1
   :widths: 30 25 45

   * - Need
     - Transformation
     - Example YAML
   * - Join multiple queries
     - ``concat``
     - ``separator: " OR "``
   * - Template final output
     - ``template``
     - ``template: "header\n{queries}\nfooter"``
   * - Export as JSON
     - ``json``
     - ``template: '{"queries": {queries}}'``
   * - Export as YAML
     - ``yaml``
     - (no extra params needed)

Common Recipes
**************

Recipe 1: Sysmon → Elastic
==========================

.. code-block:: yaml

   name: Sysmon to Elastic
   priority: 10
   transformations:
     - type: field_name_mapping
       mapping:
         EventID: event_id
         Image: process.executable
         CommandLine: process.command_line
     - type: add_field
       field: event.category
       value: process
   postprocessing:
     - type: simple_template
       template: |
         {
           "query": {
             "bool": {
               "must": [{"query_string": {"query": "{query}"}}],
               "filter": [{"term": {"event.category": "process"}}]
             }
           }
         }

Recipe 2: Add Timestamp Standardization
========================================

.. code-block:: yaml

   name: Standardize timestamps
   priority: 10
   transformations:
     - type: field_name_mapping
       mapping:
         TimeGenerated: "@timestamp"
         Created: "@timestamp"
         Date: "@timestamp"

Recipe 3: Field Prefix for Multi-Source
========================================

.. code-block:: yaml

   name: Add source prefix
   priority: 20
   transformations:
     - type: field_name_prefix
       prefix: "sysmon."
       field_name_conditions:
         - type: include_fields
           fields: ["Image", "CommandLine", "ParentImage"]

Recipe 4: Value Mapping with OR
================================

.. code-block:: yaml

   name: Map status codes
   priority: 10
   transformations:
     - type: map_string
       mapping:
         "0": "success"
         "1": "partial_success"
         "2": "failure"
         "3":
           - "error"
           - "failed"

Recipe 5: Conditional Transformation
=====================================

.. code-block:: yaml

   name: Conditional field mapping
   priority: 10
   transformations:
     - type: field_name_mapping
       mapping:
         EventID: event_id
       rule_conditions:
         - type: logsource
           service: sysmon
     - type: field_name_mapping
       mapping:
         EventCode: event_id
       rule_conditions:
         - type: logsource
           service: windows
