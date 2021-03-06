# Whole-Stage Java Code Generation

*Whole-Stage Java Code Generation* (aka _Whole-Stage CodeGen_) is a physical query optimization in Spark SQL that fuses multiple physical operators (as a subtree of plans that link:spark-sql-CodegenSupport.adoc[support code generation]) together into a single Java function.

Whole-Stage Java Code Generation improves the execution performance of a query by collapsing a query tree into a single optimized function that eliminates virtual function calls and leverages CPU registers for intermediate data.

[[spark.sql.codegen.wholeStage]]
[NOTE]
====
Whole-Stage Code Generation is controlled by link:spark-sql-properties.adoc#spark.sql.codegen.wholeStage[spark.sql.codegen.wholeStage] Spark internal property.

Whole-Stage Code Generation is enabled by default.

[source, scala]
----
import org.apache.spark.sql.internal.SQLConf.WHOLESTAGE_CODEGEN_ENABLED
scala> spark.conf.get(WHOLESTAGE_CODEGEN_ENABLED)
res0: String = true
----

Use [SQLConf.wholeStageEnabled](SQLConf.md#wholeStageEnabled) method to access the current value.

```text
scala> spark.sessionState.conf.wholeStageEnabled
res1: Boolean = true
```
====

[NOTE]
====
Whole-Stage Code Generation is used by some modern massively parallel processing (MPP) databases to achieve a better query execution performance.

See http://www.vldb.org/pvldb/vol4/p539-neumann.pdf[Efficiently Compiling Efficient Query Plans for Modern Hardware (PDF)].
====

NOTE: https://janino-compiler.github.io/janino/[Janino] is used to compile a Java source code into a Java class at runtime.

[[CollapseCodegenStages]]
Before a query is executed, link:spark-sql-CollapseCodegenStages.adoc[CollapseCodegenStages] physical preparation rule finds the physical query plans that support codegen and collapses them together as `WholeStageCodegen` (possibly with link:spark-sql-SparkPlan-InputAdapter.adoc[InputAdapter] in-between for physical operators with no support for Java code generation).

NOTE: `CollapseCodegenStages` is part of the sequence of physical preparation rules link:spark-sql-QueryExecution.adoc#preparations[QueryExecution.preparations] that will be applied in order to the physical plan before execution.

There are the following code generation paths (as coined in https://github.com/apache/spark/commit/70221903f54eaa0514d5d189dfb6f175a62228a8[this commit]):

1. Non-whole-stage-codegen path

[[produce-path]]
1. Whole-stage-codegen "produce" path

[[consume-path]]
1. Whole-stage-codegen "consume" path

TIP: Review https://issues.apache.org/jira/browse/SPARK-12795[SPARK-12795 Whole stage codegen] to learn about the work to support it.

=== [[BenchmarkWholeStageCodegen]] BenchmarkWholeStageCodegen -- Performance Benchmark

`BenchmarkWholeStageCodegen` class provides a benchmark to measure whole stage codegen performance.

You can execute it using the command:

```
build/sbt 'sql/testOnly *BenchmarkWholeStageCodegen'
```

NOTE: You need to un-ignore tests in `BenchmarkWholeStageCodegen` by replacing `ignore` with `test`.

```
$ build/sbt 'sql/testOnly *BenchmarkWholeStageCodegen'
...
Running benchmark: range/limit/sum
  Running case: range/limit/sum codegen=false
22:55:23.028 WARN org.apache.hadoop.util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
  Running case: range/limit/sum codegen=true

Java HotSpot(TM) 64-Bit Server VM 1.8.0_77-b03 on Mac OS X 10.10.5
Intel(R) Core(TM) i7-4870HQ CPU @ 2.50GHz

range/limit/sum:                    Best/Avg Time(ms)    Rate(M/s)   Per Row(ns)   Relative
-------------------------------------------------------------------------------------------
range/limit/sum codegen=false             376 /  433       1394.5           0.7       1.0X
range/limit/sum codegen=true              332 /  388       1581.3           0.6       1.1X

[info] - range/limit/sum (10 seconds, 74 milliseconds)
```
