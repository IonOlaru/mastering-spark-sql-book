# AggregateProcessor

`AggregateProcessor` is <<apply, created>> and used exclusively when `WindowExec` physical operator is executed.

`AggregateProcessor` supports link:spark-sql-Expression-DeclarativeAggregate.adoc[DeclarativeAggregate] and link:spark-sql-Expression-ImperativeAggregate.adoc[ImperativeAggregate] aggregate <<functions, functions>> only (which link:spark-sql-SparkPlan-WindowExec.adoc#windowFrameExpressionFactoryPairs[happen to] be link:spark-sql-Expression-AggregateFunction.adoc[AggregateFunction] in [AggregateExpression](../expressions/AggregateExpression.md) or [AggregateWindowFunction](../expressions/AggregateWindowFunction.md)).

[[properties]]
.AggregateProcessor's Properties
[cols="1,2",options="header",width="100%"]
|===
| Name
| Description

| [[buffer]] `buffer`
| `SpecificInternalRow` with data types given <<bufferSchema, bufferSchema>>
|===

NOTE: `AggregateProcessor` is <<creating-instance, created>> using `AggregateProcessor` factory object (using <<apply, apply>> method).

=== [[initialize]] `initialize` Method

[source, scala]
----
initialize(size: Int): Unit
----

CAUTION: FIXME

[NOTE]
====
`initialize` is used when:

* `SlidingWindowFunctionFrame` writes out to the target row
* `UnboundedWindowFunctionFrame` is prepared
* `UnboundedPrecedingWindowFunctionFrame` is prepared
* `UnboundedFollowingWindowFunctionFrame` writes out to the target row
====

=== [[evaluate]] `evaluate` Method

[source, scala]
----
evaluate(target: InternalRow): Unit
----

CAUTION: FIXME

NOTE: `evaluate` is used when...FIXME

=== [[apply]][[functions]] `apply` Factory Method

[source, scala]
----
apply(
  functions: Array[Expression],
  ordinal: Int,
  inputAttributes: Seq[Attribute],
  newMutableProjection: (Seq[Expression], Seq[Attribute]) => MutableProjection): AggregateProcessor
----

NOTE: `apply` is used exclusively when `WindowExec` is link:spark-sql-SparkPlan-WindowExec.adoc#doExecute[executed] (and creates link:spark-sql-WindowFunctionFrame.adoc[WindowFunctionFrame] per `AGGREGATE` window aggregate functions, i.e. [AggregateExpression](../expressions/AggregateExpression.md) or [AggregateWindowFunction](../expressions/AggregateWindowFunction.md))

=== [[update]] Executing update on ImperativeAggregates -- `update` Method

[source, scala]
----
update(input: InternalRow): Unit
----

`update` executes the link:spark-sql-Expression-ImperativeAggregate.adoc#update[update] method on every input <<imperatives, ImperativeAggregate>> sequentially (one by one).

Internally, `update` joins <<buffer, buffer>> with `input` link:spark-sql-InternalRow.adoc[internal binary row] and converts the joined `InternalRow` using the <<updateProjection, MutableProjection>> function.

`update` then requests every <<imperatives, ImperativeAggregate>> to  link:spark-sql-Expression-ImperativeAggregate.adoc#update[update] passing in the <<buffer, buffer>> and the input `input` rows.

NOTE: `MutableProjection` mutates the same underlying binary row object each time it is executed.

NOTE: `update` is used when `WindowFunctionFrame` link:spark-sql-WindowFunctionFrame.adoc#prepare[prepares] or link:spark-sql-WindowFunctionFrame.adoc#write[writes].

=== [[creating-instance]] Creating AggregateProcessor Instance

`AggregateProcessor` takes the following when created:

* [[bufferSchema]] Schema of the buffer (as a collection of `AttributeReferences`)
* [[initialProjection]] Initial `MutableProjection`
* [[updateProjection]] Update `MutableProjection`
* [[evaluateProjection]] Evaluate `MutableProjection`
* [[imperatives]] link:spark-sql-Expression-ImperativeAggregate.adoc[ImperativeAggregate] expressions for aggregate functions
* [[trackPartitionSize]] Flag whether to track partition size
