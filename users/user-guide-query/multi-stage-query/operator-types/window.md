---
description: >-
  Describes the window relational operator in the multi-stage query engine.
---

# Window relational operator

The window operator is used to define a window over which to perform calculations. 

This page describes the window operator defined in the relational algebra used by multi-stage queries.
This operator is generated by the multi-stage query engine when you _window functions_ in a query.
You can read more about window functions in the [windows functions](../../query-syntax/windows-functions.md) reference
documentation.

Unlike the [aggregate operator](aggregate.md), which will output one row per group, the window operator will output as
many rows as input rows.

## Implementation details

Window operators take a single input relation and apply window functions to it.
For each input row, a window of rows is calculated and one or many aggregations are applied to it.

In general window operator are expensive in terms of CPU and memory usage, but they open the door to a wide range of
analytical queries.


### Blocking nature
The window operator is a blocking operator. It needs to consume all the input data before emitting the result.

## Hints
Window hints are configured with the `windowOptions` hint, which accepts as argument a map of options and values.

For example:

```sql
SELECT
/*+  windowOptions(option1='value1', option2='value2') */
    col1, SUM(intCol) OVER() as sum FROM table
```

### max_rows_in_window
Type: Integer

Default: 1048576

Max rows allowed to cache the rows in window for further processing.

### window_overflow_mode
Type: THROW or BREAK

Default: 'THROW'

Mode when window overflow happens, supported values:
* `THROW`: Break window cache build process, and throw exception, no further WINDOW operation performed.
* `BREAK`: Break window cache build process, continue to perform WINDOW operation, results might be partial.

## Stats
### executionTimeMs
Type: Long

The summation of time spent by all threads executing the operator.
This means that the wall time spent in the operation may be smaller that this value if the parallelism is larger than 1.
This number is affected by the number of received rows and the complexity of the window function.

### emittedRows
Type: Long

The number of groups emitted by the operator.
A large number of emitted rows can indicate that the query is not well optimized.

Unlike the [aggregate operator](aggregate.md), which will output one row per group, the window operator will output as 
many rows as input rows.

### maxRowsInWindowReached
Type: Boolean

This attribute is set to `true` if the maximum number of rows in the window has been reached.

## Explain attributes

The window operator is represented in the explain plan as a `LogicalWindow` explain node.

### window#
Type: Expression

The window expressions used by the operator.
There may be more than one of these attributes depending on the number of window functions used in the query,
although sometimes multiple window function clauses in SQL can be combined into a single window operator.

The expression may use indexed columns (`$0`, `$1`, etc) that represent the columns of the virtual row generated by the 
upstream.

## Tips and tricks
None
