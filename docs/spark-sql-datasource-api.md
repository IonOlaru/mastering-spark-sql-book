# DataSource API -- Managing Datasets in External Data Sources

=== [[reading-datasets]] Reading Datasets

Spark SQL can read data from external storage systems like files, Hive tables and JDBC databases through [DataFrameReader](DataFrameReader.md) interface.

You use link:SparkSession.md[SparkSession] to access `DataFrameReader` using link:SparkSession.md#read[read] operation.

[source, scala]
----
import org.apache.spark.sql.SparkSession
val spark = SparkSession.builder.getOrCreate

val reader = spark.read
----

`DataFrameReader` is an interface to create link:spark-sql-DataFrame.adoc[DataFrames] (aka `Dataset[Row]`) from [files](DataFrameReader.md#creating-dataframes-from-files), [Hive tables](DataFrameReader.md#creating-dataframes-from-tables) or [tables using JDBC](DataFrameReader.md#jdbc).

[source, scala]
----
val people = reader.csv("people.csv")
val cities = reader.format("json").load("cities.json")
----

As of Spark 2.0, `DataFrameReader` can read text files using [textFile](DataFrameReader.md#textFile) methods that return `Dataset[String]` (not `DataFrames`).

[source, scala]
----
spark.read.textFile("README.md")
----

You can also link:spark-sql-datasource-custom-formats.adoc[define your own custom file formats].

[source, scala]
----
val countries = reader.format("customFormat").load("countries.cf")
----

There are two operation modes in Spark SQL, i.e. batch and link:spark-structured-streaming.adoc[streaming] (part of Spark Structured Streaming).

You can access link:spark-sql-streaming-DataStreamReader.adoc[DataStreamReader] for reading streaming datasets through link:SparkSession.md#readStream[SparkSession.readStream] method.

[source, scala]
----
import org.apache.spark.sql.streaming.DataStreamReader
val stream: DataStreamReader = spark.readStream
----

The available methods in `DataStreamReader` are similar to `DataFrameReader`.

=== [[saving-datasets]] Saving Datasets

Spark SQL can save data to external storage systems like files, Hive tables and JDBC databases through link:spark-sql-DataFrameWriter.adoc[DataFrameWriter] interface.

You use link:spark-sql-Dataset.adoc#write[write] method on a `Dataset` to access `DataFrameWriter`.

[source, scala]
----
import org.apache.spark.sql.{DataFrameWriter, Dataset}
val ints: Dataset[Int] = (0 to 5).toDS

val writer: DataFrameWriter[Int] = ints.write
----

`DataFrameWriter` is an interface to persist a link:spark-sql-Dataset.adoc[Datasets] to an external storage system in a batch fashion.

You can access link:spark-sql-streaming-DataStreamWriter.adoc[DataStreamWriter] for writing streaming datasets through link:spark-sql-Dataset.adoc#writeStream[Dataset.writeStream] method.

[source, scala]
----
val papers = spark.readStream.text("papers").as[String]

import org.apache.spark.sql.streaming.DataStreamWriter
val writer: DataStreamWriter[String] = papers.writeStream
----

The available methods in `DataStreamWriter` are similar to `DataFrameWriter`.
