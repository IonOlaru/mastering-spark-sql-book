title: ExternalAppendOnlyUnsafeRowArray

# ExternalAppendOnlyUnsafeRowArray -- Append-Only Array for UnsafeRows (with Disk Spill Threshold)

`ExternalAppendOnlyUnsafeRowArray` is an append-only array for link:spark-sql-UnsafeRow.adoc[UnsafeRows] that spills content to disk when a <<numRowsSpillThreshold, predefined spill threshold of rows>> is reached.

NOTE: Choosing a proper *spill threshold of rows* is a performance optimization.

`ExternalAppendOnlyUnsafeRowArray` is created when:

* `WindowExec` physical operator is link:spark-sql-SparkPlan-WindowExec.adoc#doExecute[executed] (and creates an internal buffer for window frames)

* `WindowFunctionFrame` is link:spark-sql-WindowFunctionFrame.adoc#prepare[prepared]

* `SortMergeJoinExec` physical operator is link:spark-sql-SparkPlan-SortMergeJoinExec.adoc#doExecute[executed] (and creates a `RowIterator` for INNER and CROSS joins) and for `getBufferedMatches`

* `SortMergeJoinScanner` creates an internal `bufferedMatches`

* `UnsafeCartesianRDD` is computed

[[internal-registries]]
.ExternalAppendOnlyUnsafeRowArray's Internal Registries and Counters
[cols="1,2",options="header",width="100%"]
|===
| Name
| Description

| [[initialSizeOfInMemoryBuffer]] `initialSizeOfInMemoryBuffer`
| FIXME

Used when...FIXME

| [[inMemoryBuffer]] `inMemoryBuffer`
| FIXME

Can grow up to <<numRowsSpillThreshold, numRowsSpillThreshold>> rows (i.e. new `UnsafeRows` are <<add, added>>)

Used when...FIXME

| [[spillableArray]] `spillableArray`
| `UnsafeExternalSorter`

Used when...FIXME

| [[numRows]] `numRows`
|

Used when...FIXME

| [[modificationsCount]] `modificationsCount`
|

Used when...FIXME

| [[numFieldsPerRow]] `numFieldsPerRow`
|

Used when...FIXME
|===

[TIP]
====
Enable `INFO` logging level for `org.apache.spark.sql.execution.ExternalAppendOnlyUnsafeRowArray` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.sql.execution.ExternalAppendOnlyUnsafeRowArray=INFO
```

Refer to link:spark-logging.adoc[Logging].
====

=== [[generateIterator]] `generateIterator` Method

[source, scala]
----
generateIterator(): Iterator[UnsafeRow]
generateIterator(startIndex: Int): Iterator[UnsafeRow]
----

CAUTION: FIXME

=== [[add]] `add` Method

[source, scala]
----
add(unsafeRow: UnsafeRow): Unit
----

CAUTION: FIXME

[NOTE]
====
`add` is used when:

* `WindowExec` is executed (and link:spark-sql-SparkPlan-WindowExec.adoc#fetchNextPartition[fetches all rows in a partition for a group].

* `SortMergeJoinScanner` buffers matching rows

* `UnsafeCartesianRDD` is computed
====

=== [[clear]] `clear` Method

[source, scala]
----
clear(): Unit
----

CAUTION: FIXME

=== [[creating-instance]] Creating ExternalAppendOnlyUnsafeRowArray Instance

`ExternalAppendOnlyUnsafeRowArray` takes the following when created:

* [[taskMemoryManager]] link:spark-taskscheduler-taskmemorymanager.adoc[TaskMemoryManager]
* [[blockManager]] link:spark-blockmanager.adoc[BlockManager]
* [[serializerManager]] link:spark-SerializerManager.adoc[SerializerManager]
* [[taskContext]] link:spark-taskscheduler-taskcontext.adoc[TaskContext]
* [[initialSize]] Initial size
* [[pageSizeBytes]] Page size (in bytes)
* [[numRowsSpillThreshold]] Number of rows to hold before spilling them to disk

`ExternalAppendOnlyUnsafeRowArray` initializes the <<internal-registries, internal registries and counters>>.
