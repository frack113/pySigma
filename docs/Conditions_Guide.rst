Conditions Guide
################

Conditions allow you to apply transformations selectively based on rule attributes, field names, or detection items.

Condition Types
***************

There are three levels of conditions in pySigma:

1. **Rule conditions** — applied to the whole rule
2. **Detection item conditions** — applied to each detection item
3. **Field name conditions** — applied to field names in detection items

Rule Conditions
***************

Rule conditions are evaluated once per rule. They are defined in the `rule_conditions` attribute.

Available Rule Conditions
=========================

| Identifier                  | Class                              | Purpose                                      |
|-----------------------------|------------------------------------|----------------------------------------------|
| `logsource`                 | `LogsourceCondition`               | Match by log source (product, service, etc.) |
| `contains_detection_item`   | `RuleContainsDetectionItemCondition` | Check if rule contains specific detection item |
| `processing_item_applied`   | `RuleProcessingItemAppliedCondition` | Check if a transformation was already applied |
| `processing_state`          | `RuleProcessingStateCondition`     | Check processing state set by `set_state`    |
| `is_sigma_rule`             | `IsSigmaRuleCondition`             | Check if rule is a Sigma rule (not correlation) |
| `is_sigma_correlation_rule` | `IsSigmaCorrelationRuleCondition`  | Check if rule is a correlation rule          |
| `rule_attribute`            | `RuleAttributeCondition`           | Check rule attributes (title, id, etc.)      |
| `tag`                       | `RuleTagCondition`                 | Match by Sigma tags                            |

Rule Condition Examples
=======================

Example 1: Match by log source
------------------------------

.. code-block:: yaml

   transformations:
     - type: field_name_mapping
       mapping:
         EventID: event_id
       rule_conditions:
         - type: logsource
           product: windows
           service: sysmon

Example 2: Match by tag
-----------------------

.. code-block:: yaml

   transformations:
     - type: add_field
       field: processed
       value: true
       rule_conditions:
         - type: tag
           tag: "attack.t1003"

Example 3: Check processing state
----------------------------------

.. code-block:: yaml

   transformations:
     - type: set_state
       state: mapped
       rule_conditions:
         - type: processing_state
           state: "not_mapped"

Example 4: Multiple conditions (AND)
-------------------------------------

.. code-block:: yaml

   transformations:
     - type: field_name_mapping
       mapping:
         Image: process.executable
       rule_conditions:
         - type: logsource
           service: sysmon
         - type: tag
           tag: "attack.execution"

Detection Item Conditions
*************************

Detection item conditions are evaluated for each detection item in the rule.

Available Detection Item Conditions
====================================

| Identifier              | Class                                  | Purpose                              |
|-------------------------|----------------------------------------|--------------------------------------|
| `match_string`          | `MatchStringCondition`                 | Match detection item string values   |
| `is_null`               | `IsNullCondition`                      | Check if detection item is null      |
| `processing_item_applied` | `DetectionItemProcessingItemAppliedCondition` | Check if transformation was applied |
| `processing_state`      | `DetectionItemProcessingStateCondition` | Check processing state               |

Detection Item Condition Examples
==================================

Example 1: Match specific string values
----------------------------------------

.. code-block:: yaml

   transformations:
     - type: set_value
       value: "known_bad"
       detection_item_conditions:
         - type: match_string
           match: "malware.exe"

Example 2: Check for null values
---------------------------------

.. code-block:: yaml

   transformations:
     - type: drop_detection_item
       detection_item_conditions:
         - type: is_null

Field Name Conditions
*********************

Field name conditions are evaluated for field names in detection items.

Available Field Name Conditions
================================

| Identifier              | Class                                  | Purpose                              |
|-------------------------|----------------------------------------|--------------------------------------|
| `include_fields`        | `IncludeFieldCondition`                | Match specific field names           |
| `exclude_fields`        | `ExcludeFieldCondition`                | Exclude specific field names         |
| `processing_item_applied` | `FieldNameProcessingItemAppliedCondition` | Check if transformation was applied |
| `processing_state`      | `FieldNameProcessingStateCondition`     | Check processing state               |

Field Name Condition Examples
==============================

Example 1: Apply to specific fields only
-----------------------------------------

.. code-block:: yaml

   transformations:
     - type: field_name_prefix
       prefix: "sysmon."
       field_name_conditions:
         - type: include_fields
           fields: ["Image", "CommandLine", "ParentImage"]

Example 2: Exclude certain fields
-----------------------------------

.. code-block:: yaml

   transformations:
     - type: field_name_suffix
       suffix: "_raw"
       field_name_conditions:
         - type: exclude_fields
           fields: ["EventID", "Timestamp"]

Condition Operators
*******************

You can combine multiple conditions using operators.

Using `*_cond_op` (list mode)
==============================

.. code-block:: yaml

   rule_conditions:
     - type: logsource
       service: sysmon
     - type: tag
       tag: "attack.execution"
   rule_cond_op: "and"  # both must match

Using `*_cond_expr` (expression mode)
======================================

.. code-block:: yaml

   rule_conditions:
     cond1:
       type: logsource
       service: sysmon
     cond2:
       type: tag
       tag: "attack.execution"
   rule_cond_expr: "cond1 and cond2"

Supported operators in expressions:
- `and` — logical AND
- `or` — logical OR
- `not` — logical NOT (prefix)

Condition Negation
******************

Negate a condition result by setting `*_cond_not: true`.

.. code-block:: yaml

   transformations:
     - type: field_name_mapping
       mapping:
         EventID: event_id
       rule_conditions:
         - type: logsource
           service: sysmon
       rule_cond_not: true  # apply to rules WITHOUT Sysmon logsource

Best Practices
**************

1. **Use `id` for tracking** — Add identifiers to your conditions for debugging:

   .. code-block:: yaml

      rule_conditions:
        - id: is_sysmon
          type: logsource
          service: sysmon

2. **Combine conditions carefully** — AND conditions are more restrictive, OR are broader.

3. **Test with simple rules first** — Verify your conditions work before applying to large rule sets.

4. **Use `set_state` for chaining** — Mark rules/items as processed to avoid double-transformation:

   .. code-block:: yaml

      transformations:
        - type: set_state
          state: mapped
        - type: field_name_mapping
          mapping: {EventID: event_id}
          rule_conditions:
            - type: processing_state
              state: "not_mapped"

5. **Document your conditions** — Add comments in YAML explaining why certain conditions are used.
