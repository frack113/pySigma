Transformation Development
##########################

This guide explains how to write a custom transformation for pySigma.

Transformation Overview
***********************

A transformation is a processing step that modifies Sigma rules before conversion.

Transformations are defined in YAML pipelines and applied in order.

Key Classes
***********

| Class                              | Purpose                                      |
|------------------------------------|----------------------------------------------|
| `SigmaTransformation`            | Base class for all transformations           |
| `SigmaRuleTransformation`        | Base class for rule-level transformations    |
| `SigmaDetectionItemTransformation` | Base class for detection item transformations |
| `SigmaFieldMappingTransformation`  | Base class for field mapping transformations |

Available Transformation Types
******************************

| Identifier                       | Class                              | Purpose                                      |
|----------------------------------|------------------------------------|----------------------------------------------|
| `field_name_mapping`             | `FieldNameMappingTransformation`   | Map field names                                |
| `field_name_prefix`              | `FieldNamePrefixTransformation`    | Add prefix to field names                      |
| `field_name_suffix`              | `FieldNameSuffixTransformation`    | Add suffix to field names                      |
| `field_name_transform`           | `FieldNameTransformTransformation` | Apply function to field names                  |
| `map_string`                     | `MapStringTransformation`          | Map string values                              |
| `replace_string`                 | `ReplaceStringTransformation`      | Replace strings with regex                     |
| `convert_type`                   | `ConvertTypeTransformation`        | Convert value types                            |
| `set_value`                      | `SetValueTransformation`           | Set specific value                             |
| `add_condition`                  | `AddConditionTransformation`       | Add detection condition                        |
| `change_logsource`               | `ChangeLogsourceTransformation`    | Change log source                              |
| `add_field`                      | `AddFieldTransformation`           | Add field to rule                              |
| `remove_field`                   | `RemoveFieldTransformation`        | Remove field from rule                         |
| `set_field`                      | `SetFieldTransformation`           | Set field value                                |
| `set_state`                      | `SetStateTransformation`           | Set processing state                           |
| `nest`                           | `NestTransformation`               | Group transformations                          |
| `regex`                          | `RegexTransformation`              | Apply regex to field                           |
| `drop_detection_item`            | `DropDetectionItemTransformation`  | Remove detection item                          |
| `hashes_fields`                  | `HashesFieldsTransformation`       | Convert hash fields                            |
| `wildcard_placeholders`          | `WildcardPlaceholdersTransformation` | Add wildcard placeholders                      |
| `value_placeholders`             | `ValuePlaceholdersTransformation`  | Add value list placeholders                    |
| `simple_template`                | `SimpleTemplateTransformation`     | Wrap query in template                         |
| `template`                       | `TemplateTransformation`           | Custom query template                          |
| `json`                           | `JsonTransformation`               | Export as JSON                                 |
| `yaml`                           | `YamlTransformation`               | Export as YAML                                 |

Writing a Custom Transformation
*******************************

Example 1: Rule-Level Transformation
=====================================

.. code-block:: python

   from sigma.processing.transformations import SigmaRuleTransformation
   from sigma.rule import SigmaRule
   from sigma.processing.pipeline import ProcessingItem

   class CustomRuleTransformation(SigmaRuleTransformation):
       """Add a custom field to all rules."""

       def apply(self, rule: SigmaRule, item: ProcessingItem) -> None:
           """Apply transformation to rule."""
           rule.custom_field = "custom_value"

Example 2: Detection Item Transformation
=========================================

.. code-block:: python

   from sigma.processing.transformations import SigmaDetectionItemTransformation
   from sigma.detection import SigmaDetectionItem
   from sigma.processing.pipeline import ProcessingItem

   class CustomDetectionItemTransformation(SigmaDetectionItemTransformation):
       """Convert specific values."""

       def apply(self, detection_item: SigmaDetectionItem, item: ProcessingItem) -> None:
           """Apply transformation to detection item."""
           if detection_item.value == "old_value":
               detection_item.value = "new_value"

Example 3: Field Mapping Transformation
========================================

.. code-block:: python

   from sigma.processing.transformations import SigmaFieldMappingTransformation
   from sigma.processing.pipeline import ProcessingItem

   class CustomFieldMappingTransformation(SigmaFieldMappingTransformation):
       """Custom field mapping logic."""

       def apply(self, field_mapping: dict, item: ProcessingItem) -> None:
           """Apply transformation to field mapping."""
           # Modify field mapping
           for old_field, new_field in field_mapping.items():
               if "old_prefix" in old_field:
                   field_mapping[old_field.replace("old_prefix", "new_prefix")] = new_field
                   del field_mapping[old_field]

Using Conditions
****************

You can apply transformations conditionally:

.. code-block:: python

   from sigma.processing.transformations import SigmaRuleTransformation
   from sigma.rule import SigmaRule
   from sigma.processing.pipeline import ProcessingItem
   from sigma.conditions import ConditionLogsource

   class ConditionalTransformation(SigmaRuleTransformation):
       """Apply transformation only to specific log sources."""

       def apply(self, rule: SigmaRule, item: ProcessingItem) -> None:
           """Apply transformation conditionally."""
           if rule.logsource and rule.logsource.service == "sysmon":
               # Apply transformation
               rule.custom_field = "sysmon_rule"

Best Practices
**************

1. **Inherit from base classes** — Use `SigmaRuleTransformation`, `SigmaDetectionItemTransformation`, etc.

2. **Use `apply()` method** — Implement the `apply()` method for your transformation logic.

3. **Handle conditions** — Check conditions before applying transformations.

4. **Modify in place** — Transformations modify the rule/item directly.

5. **Use `ProcessingItem` for config** — Access transformation configuration via `item.config`.

6. **Test with real rules** — Use `SigmaCollection.from_yaml()` for test data.

7. **Document your transformation** — Add docstrings explaining what it does.

8. **Handle edge cases** — Check for None values, empty fields, etc.

9. **Use `set_state` for chaining** — Mark rules/items as processed to avoid double-transformation.

10. **Keep transformations focused** — Each transformation should do one thing well.

Testing Transformations
***********************

Use `pytest` with `SigmaCollection` fixtures:

.. code-block:: python

   from sigma.collection import SigmaCollection
   from sigma.processing.pipeline import ProcessingPipeline
   from my_transformations import CustomRuleTransformation

   def test_custom_transformation():
       rules = SigmaCollection.from_yaml("""
       title: Test Rule
       id: 12345678-1234-1234-1234-123456789012
       status: experimental
       logsource:
         product: windows
       detection:
         selection:
           EventID: 1
         condition: selection
       """)

       pipeline = ProcessingPipeline(
           name="Custom Pipeline",
           priority=10,
           transformations=[CustomRuleTransformation()],
           postprocessing=[],
       )

       # Apply pipeline
       rules.apply_pipeline(pipeline)

       # Check transformation was applied
       for rule in rules.rules:
           assert hasattr(rule, "custom_field")
           assert rule.custom_field == "custom_value"

Integration with Backends
*************************

Transformations are applied before backend conversion:

.. code-block:: python

   from sigma.collection import SigmaCollection
   from sigma.processing.pipeline import ProcessingPipeline
   from my_backend import MyBackend
   from my_transformations import CustomRuleTransformation

   # Create pipeline
   pipeline = ProcessingPipeline(
       name="Custom Pipeline",
       priority=10,
       transformations=[CustomRuleTransformation()],
       postprocessing=[],
   )

   # Create backend with pipeline
   backend = MyBackend(pipeline=pipeline)

   # Convert rules
   rules = SigmaCollection.from_yaml("...")
   queries = backend.convert(rules)
