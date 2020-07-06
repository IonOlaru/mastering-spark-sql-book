# SQL Tab -- Monitoring Structured Queries in web UI

*SQL* tab in link:spark-webui.adoc[web UI] shows link:spark-sql-SQLMetric.adoc[SQLMetrics] per link:spark-sql-SparkPlan.adoc[physical operator] in a structured query physical plan.

You can access the SQL tab under `/SQL` URL, e.g. http://localhost:4040/SQL/.

By default, it displays <<AllExecutionsPage, all SQL query executions>>. However, after a query has been selected, the SQL tab <<ExecutionPage, displays the details for the structured query execution>>.

=== [[AllExecutionsPage]] AllExecutionsPage

`AllExecutionsPage` displays all SQL query executions in a Spark application per state sorted by their submission time reversed.

.SQL Tab in web UI (AllExecutionsPage)
image::images/spark-webui-sql.png[align="center"]

Internally, the page requests link:spark-sql-SQLListener.adoc[SQLListener] for query executions in running, completed, and failed states (the states correspond to the respective tables on the page).

=== [[ExecutionPage]] ExecutionPage -- Details for Query

`ExecutionPage` shows details for structured query execution by `id`.

NOTE: The `id` request parameter is mandatory.

`ExecutionPage` displays a summary with *Submitted Time*, *Duration*, the clickable identifiers of the *Running Jobs*, *Succeeded Jobs*, and *Failed Jobs*.

It also display a visualization (using link:spark-sql-SQLListener.adoc#getExecutionMetrics[accumulator updates] and the `SparkPlanGraph` for the query) with the expandable *Details* section (that corresponds to `SQLExecutionUIData.physicalPlanDescription`).

.Details for Query in web UI
image::images/spark-webui-sql-execution-graph.png[align="center"]

If there is no information to display for a given query `id`, you should see the following page.

.No Details for SQL Query
image::images/spark-webui-sql-no-details-for-query.png[align="center"]

Internally, it uses link:spark-sql-SQLListener.adoc[SQLListener] exclusively to get the SQL query execution metrics. It requests link:spark-sql-SQLListener.adoc#getExecution[`SQLListener` for SQL execution data] to display for the `id` request parameter.

=== [[creating-instance]] Creating SQLTab Instance

`SQLTab` is created when link:spark-sql-SharedState.adoc[SharedState] is or at the first link:spark-sql-SQLListener.adoc#SparkListenerSQLExecutionStart[SparkListenerSQLExecutionStart] event when link:spark-history-server.adoc[Spark History Server] is used.

.Creating SQLTab Instance
image::images/spark-SQLTab-creating-instance.png[align="center"]

NOTE: link:spark-sql-SharedState.adoc[SharedState] represents the shared state across `SparkSessions`.