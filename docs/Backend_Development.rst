Backend Development
###################

This guide explains how to write a custom backend for pySigma.

Backend Overview
****************

A backend is a class that converts Sigma rules into backend-specific query formats.

The conversion pipeline:

1. **Rule** (Sigma rule) → **ParsedRule** (parsed rule object)
2. **ParsedRule** → **QueryExpression** (intermediate query representation)
3. **QueryExpression** → **BackendQuery** (backend-specific query string)

Key Classes
***********

.. list-table::
   :header-rows: 1
   :widths: 40 60

   * - Class
     - Purpose
   * - ``SigmaBackend``
     - Base class for all backends
   * - ``SigmaQueryExpression``
     - Base class for query expressions
   * - ``SigmaQueryExpressionTransformer``
     - Transforms query expressions (e.g., to string)
   * - ``SigmaConditionTranslator``
     - Translates Sigma conditions to query expressions
   * - ``SigmaRuleConverter``
     - Converts parsed rules to query expressions
   * - ``SigmaDetectionConverter``
     - Converts detection items to query expressions
   * - ``SigmaDetectionItemConverter``
     - Converts individual detection items
   * - ``SigmaFieldMapping``
     - Handles field name mappings
   * - ``SigmaValueConverter``
     - Converts values (strings, numbers, etc.)

Required Methods
****************

A minimal backend must implement:

1. `convert_rule()` — Convert a parsed rule to queries
2. `convert_condition()` — Convert a condition expression to a query expression
3. `convert_detection_item()` — Convert a detection item to a query expression

Example: Minimal Backend
************************

.. code-block:: python

   from sigma.processing.pipeline import ProcessingPipeline
   from sigma.conversion.backends.base import BaseSigmaBackend
   from sigma.conversion.state import QueryObject
   from sigma.types import SigmaString
   from sigma.rule import SigmaRule
   from sigma.parsing.parser import SigmaParser
   from sigma.collection import SigmaCollection

   class MyBackend(BaseSigmaBackend):
       """Minimal backend that outputs raw query strings."""

       def __init__(self, pipeline: ProcessingPipeline = None):
           super().__init__(pipeline)

       def convert_rule(self, parsed_rule: SigmaRule) -> QueryObject:
           """Convert a parsed rule to a backend query."""
           # Get the detection items from the rule
           detections = parsed_rule.detection
           # Build a simple query from the detection items
           conditions = []
           for condition in detections.condition:
               conditions.append(str(condition))
           return QueryObject(
               query="\nOR\n".join(conditions),
               pipeline=self.pipeline,
           )

       def finalize_query(self, pipeline: ProcessingPipeline, query: QueryObject) -> str:
           """Finalize a single query."""
           return str(query.query)

       def finalize_query_output(self, pipeline: ProcessingPipeline, query: QueryObject) -> dict:
           """Finalize query output format."""
           return {"query": str(query.query)}

       def finalize_output(self, pipeline: ProcessingPipeline, queries: list[QueryObject]) -> str:
           """Finalize all queries."""
           return "\n---\n".join(self.finalize_query(pipeline, q) for q in queries)

       def convert_condition(self, condition: str) -> str:
           """Convert a condition expression."""
           return condition

       def convert_detection_item(self, detection_item: dict) -> str:
           """Convert a detection item."""
           return str(detection_item)

Using the Backend
*****************

.. code-block:: python

   from sigma.collection import SigmaCollection
   from my_backend import MyBackend

   # Load rules
   rules = SigmaCollection.from_yaml("""
   title: Test Rule
   id: 12345678-1234-1234-1234-123456789012
   status: experimental
   logsource:
     product: windows
     service: sysmon
   detection:
     selection:
       EventID: 1
       Image: "*cmd.exe"
     condition: selection
   """)

   # Convert with backend
   backend = MyBackend()
   queries = backend.convert(rules)
   print(queries)

Query Object
************

The `QueryObject` class stores the converted query:

.. code-block:: python

   from sigma.conversion.state import QueryObject

   query = QueryObject(
       query="EventID=1 AND Image=cmd.exe",
       pipeline=pipeline,
   )

   # Access the query
   print(query.query)

   # Access the pipeline
   print(query.pipeline)

State Management
****************

Use `QueryObject` to pass state between conversion steps:

.. code-block:: python

   from sigma.conversion.state import QueryObject

   query = QueryObject(
       query="EventID=1",
       pipeline=pipeline,
       state={"processed": True},
   )

   # Check state
   if query.state.get("processed"):
       print("Already processed")

Error Handling
**************

Raise `SigmaError` for conversion errors:

.. code-block:: python

   from sigma.exceptions import SigmaError

   class MyBackend(BaseSigmaBackend):
       def convert_rule(self, parsed_rule: SigmaRule) -> QueryObject:
           if not parsed_rule.detection:
               raise SigmaError(f"Rule '{parsed_rule.title}' has no detection items")
           return QueryObject(query="...", pipeline=self.pipeline)

Testing Your Backend
********************

Use `pytest` with `SigmaCollection` fixtures:

.. code-block:: python

   from sigma.collection import SigmaCollection
   from sigma.testing import TestRule
   from my_backend import MyBackend

   def test_convert_rule():
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

       backend = MyBackend()
       queries = backend.convert(rules)
       assert len(queries) == 1
       assert "EventID=1" in queries[0].query

Best Practices
**************

1. **Inherit from `BaseSigmaBackend`** — Don't implement from scratch.

2. **Use `QueryObject` for state** — Pass state between conversion steps.

3. **Handle errors with `SigmaError`** — Provide clear error messages.

4. **Test with real rules** — Use `SigmaCollection.from_yaml()` for test data.

5. **Document your backend** — Add a README explaining how to use it.

6. **Support processing pipelines** — Allow custom pipelines for field mappings, etc.

7. **Use `finalize_query_output()`** — Control the output format (JSON, YAML, etc.).

8. **Handle edge cases** — Empty rules, missing detections, invalid conditions.

9. **Use `SigmaString` for values** — Handle type conversion correctly.

10. **Test with multiple backends** — Ensure compatibility with other backends.
