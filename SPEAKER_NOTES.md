# Speaker Notes — The Upcoming Apache Spark™ 4.2
### DAIS 2026 keynote · two speakers · 78 slides

**Speakers:** XIAO = Xiao Li (gatorsmile) · DB = DB Tsai (dbtsai)

**Split:** Xiao presents slides 1–40; DB presents 41–76. One handoff, at the end of the Spark SQL section.

**Through-line (say it often):** *Spark 4.2 moves more of the modern data and AI stack into the engine itself.* Five benefits — (1) Define truth once · (2) Reach Spark from everywhere · (3) Run AI-native analytics in SQL · (4) Move changing data safely · (5) Operate & evolve predictably.

**PySpark hierarchy:** No code change (Arrow) → Small change (Arrow UDFs) → New capability (Python Data Sources) → Dev experience. On-screen perf claims are qualitative; say numbers verbally with "in our benchmarks."

**Style:** Short, simple sentences; read slowly. *(parentheses)* are stage directions. Section dividers carry a **Today → Spark 4.2 → Takeaway** triad.

---

### [1] The Upcoming Apache Spark™ 4.2 · XIAO · ~0:35
Good morning. I am Xiao Li. I work on Apache Spark at Databricks. (DB:) And I am DB Tsai. I am also on the Spark team. (XIAO:) Today we show what is new in Apache Spark 4.2. I take the first half. DB takes the second. The theme this year is native analytics for data and AI. Spark is moving more of the stack into the engine. Let's start.

### [2] About us · XIAO · ~0:12
One quick note. Spark 4.2 is built by the whole Apache Spark community — hundreds of people. Everything we show is open source.

### [3] Apache Spark™ 4.2 — Overview · XIAO · ~0:35
Spark 4.2 in one line: more semantic, AI, Python, CDC, and operations power, all in the engine. The numbers: over 1,600 JIRAs, from over 260 contributors. One goal ties it together. Make analytics and AI more native to Spark.

### [4] What we'll cover today · XIAO · ~0:40
We built the talk around five benefits. One: define truth once — metric views. Two: reach Spark from everywhere — Connect, PySpark, Arrow, Python Data Sources. Three: AI-native analytics in SQL — vectors, sketches, geospatial. Four: move changing data safely — Auto CDC, CHANGES, DSv2, streaming. Five: operate and evolve predictably — the UI, Feather, faster releases. I cover the first ones. DB covers the rest.

### [5] The features at a glance · XIAO · ~0:25
This is the whole release on one slide. Six groups. About 24 features. We will not cover them all. Just see how much is here. Keep it as a map. Let's start.

## Section 01 · Metrics — Metrics & Semantic Modeling

### [6] Section divider — Metrics & Semantic Modeling · XIAO · ~0:10
Benefit one: define truth once. Let's start with the problem. Takeaway: move your top dashboard metrics into a metric view.

### [7] The same metric should not change when the query changes · XIAO · ~0:30
The problem, in plain words. Revenue per user is a ratio. Ratios are not additive. You cannot average them across groups and get the right answer. So when every team writes the formula by hand, the dashboards drift apart. Same metric, different numbers. Let me show how it breaks.

### [8] Some metrics don't add up · XIAO · ~0:35
Here it is, in one table. Two countries. Revenue and active users add up fine. But revenue per user is a ratio. If you average the two rows, you get 15. That is wrong. The right answer is total revenue over total users. That is 13.33. So where should this metric live, so it is right every time? That is a semantic-layer problem.

### [9] Why this is a semantic-layer problem · XIAO · ~0:45
Every company already has a semantic layer. But it is scattered. 'Revenue', 'active users', and 'churn' are defined again and again — in BI tools, in dbt, in notebooks. The cost is real. Dashboards disagree. No one owns the definition. And it is about correctness. Ratios and COUNT(DISTINCT) break when you roll up. Client tools sit outside the engine, so they cannot enforce it. AI needs this too. An agent must answer the same way every time. A governed metric view gives BI and AI one source of truth.

### [10] Metric views: define metrics once · XIAO · ~0:40
So 4.2 adds metric views. That is SPIP SPARK-54119. You define a metric once. Then you query it by any breakdown. There are two parts. Dimensions are the ways to slice. Measures are a new column type that aggregates without a fixed GROUP BY. This stops drift. Ratios and distinct counts stay correct for every grouping. And measures can build on other measures. It lives in the engine and the catalog. So the optimizer keeps it correct, and permissions are the same everywhere. One source of truth — for SQL, BI, and AI.

### [11] Metric views in action · XIAO · ~0:30
Three ideas, not syntax. A dimension is how people slice the metric. A measure is what the business cares about. Here it is revenue per user — a ratio. The query asks by any breakdown. The point is not the syntax. The metric is now an object Spark understands. So it is correct at every grain, for everyone. Now — how do you connect to Spark?

## Section 02 · Connect — Spark Connect

### [12] Section divider — Spark Connect · XIAO · ~0:12
Benefit two: reach Spark from everywhere. First, Spark Connect. Takeaway: call Spark from your own service's language.

### [13] Architecture: gRPC in, Arrow out · XIAO · ~0:35
The idea is simple. Your client builds a plan. It opens a gRPC stream. The server resolves the plan, optimizes it, runs it, and sends Arrow batches back. The client is thin. It needs no full Spark runtime, and no JVM next to your app. So the client and the cluster can change on their own. Before Connect, using Spark from another runtime meant wrapping it or embedding it. With Connect, Spark is a service API. Your app does not need to be a Spark app.

### [14] Spark becomes a service API · XIAO · ~0:35
The point is the use case, not the language list. Before Connect, reaching Spark meant wrapping it or embedding it. Now Spark is a service API. A Rust service submits work. A Python notebook connects to a remote cluster. A small app uses Spark without packaging it. Python and Scala ship with Spark. Go, Swift, Rust, R, TypeScript, and .NET speak the same protocol, at different maturity levels. The next step is to make your UDFs portable too. DB comes back to that in Looking Ahead.

### [15] Closing the gap with Spark Classic · XIAO · ~0:30
In 4.2 we keep closing the gap with the classic driver. Old RDD helpers come to Connect — zipWithIndex, toJSON, emptyDataFrame. read.json, csv, and xml can take a DataFrame. There is a new GetStatus API. Errors are clearer — for example, when you use a column from the wrong DataFrame. And client file names and line numbers are kept, so debugging is easier.

## Section 03 · Python — PySpark, Faster by Default

### [16] Section divider — PySpark, Faster by Default · XIAO · ~0:30
PySpark in Spark 4.2 is moving to an Arrow-first path. Existing Python UDFs get faster by default, hot paths can use native @arrow_udf, and Python teams also get first-class data sources plus better profiling and debugging. The takeaway is simple: upgrade for the free speedup, then optimize only the Python code that matters.

### [17] Arrow on by default · XIAO · ~0:28
The important part is the default. In 4.2, Arrow Python UDFs are the default. Your old UDFs just run faster. Arrow IPC between the JVM and Python is on. And Arrow input skips one conversion step. The upgrade story: this speed, with no code change.

### [18] Python UDFs: same code, now faster · XIAO · ~0:20
(Code slide.) The same UDF you always write. No new decorator. No rewrite. But the engine moves the data as Arrow. Same code, faster.

### [19] Native Arrow UDFs: truer types, fewer copies · XIAO · ~0:42
Spark 4.2 continues the Arrow-first direction. The decorator is @arrow_udf, since 4.1: a PyArrow array in, a PyArrow array out. It removes the pandas round-trip — one less conversion — so in our benchmarks it is about 10% faster and 40% less memory than a pandas UDF. The bigger win is type fidelity. pandas mangles types — NA, NaN, NaT, and integers-with-nulls become floats — and nested types are awkward. Arrow keeps nested and struct types, uses Python-standard coercion, and stays SIMD-friendly even on nested data. At the DataFrame level you also get mapInArrow, applyInArrow, and toArrow.

### [20] Arrow UDFs in action · XIAO · ~0:20
(Code slide.) The @arrow_udf decorator. Arrays in, arrays out. No to_pandas. No round trip. Columnar from start to end.

### [21] Python Data Sources: connectors in pure Python · XIAO · ~0:40
You can now build readers and writers fully in Python — batch and streaming, no JVM, no Scala. Register it once. Then use spark.read.format, like a built-in source. The API keeps growing — pushdown, Arrow writers, streaming, partitioning. And there is a real ecosystem. The community package pyspark-data-sources has many ready connectors — Hugging Face, Google Sheets, Kaggle, Salesforce. For the AI era, an LLM can draft a long-tail connector. And you still test it and harden it.

### [22] A growing Python Data Source ecosystem · XIAO · ~0:20
(Gesture at the logos.) This is the ecosystem today. Connectors the community already built, in pure Python. Before, you learned the JVM API. Now you just write Python.

### [23] Python Data Sources in action · XIAO · ~0:20
(Code slide.) A full custom source. A reader class. Register it. Read it like any format. That is the whole thing.

### [24] Profiling Python Data Sources · XIAO · ~0:28
New in 4.2: you can profile these Python data sources, like UDFs — for time and memory. One config turns it on. Then you see where time and memory go, in your read and write code. Your connector is no longer a black box.

### [25] Better interop & developer experience · XIAO · ~0:35
Small on their own, but they help every day. PyCapsule: when both sides speak the Arrow C Data Interface, you pass data to Polars, DuckDB, and Arrow tools with no copy. You can profile Python data sources. builder.create() makes a new session without changing other configs. A strict mode catches unclear column names early. And pandas-on-Spark adds more axis=1 functions. Each one is small. Together they save you time.

### [26] PyCapsule interop in action · XIAO · ~0:25
(Code slide.) A Spark DataFrame goes straight to Polars or DuckDB, through the Arrow C Data Interface. When both sides support it, they share the same Arrow memory. No copy. No serialization.

### [27] Debug UDFs where they run · XIAO · ~0:35
Now debugging. Your UDF runs in a Python worker on an executor, far from your notebook. When it hangs, before, you saw nothing. In 4.2 you can see the hang. The worker dumps its stack trace. Idle workers are dumped and killed. One profiler covers Python, pandas, and Arrow UDFs — CPU and memory, by line. It works on Connect too. And you can use Python logging inside a UDF, then read the logs back as a table.

### [28] UDF debugging in action · XIAO · ~0:20
(Code slide.) A stuck worker, with its stack trace, shown for you. And UDF logs, read back as a normal table.

## Section 04 · New SQL Analytics — Spark SQL

### [29] Section divider — Spark SQL · XIAO · ~0:28
Benefit three: AI-native analytics in SQL. Today these workflows are scattered — a vector system here, verbose SQL there. Spark 4.2 brings them into the engine: vector functions, NEAREST BY, QUALIFY, sketches, geospatial. Takeaway: score embeddings with vector functions; use NEAREST BY when that top-K runs for every query row.

### [30] Vector functions: score embeddings in Spark SQL · XIAO · ~0:28
Embeddings are now ordinary table data. The first need is simple: compare two vectors where the data lives. Spark 4.2 adds vector functions for that — cosine similarity, inner product, L2 distance, normalize, and aggregations, all in SQL. This is the scoring primitive, not an index yet. Next: turn scoring into a top-K search.

### [31] Single-query vector search in SQL · XIAO · ~0:26
The simplest vector search: one query vector against a table. Score, sort, take the top 10 — and it is just SQL. But real workloads have many query rows, like top products for every user. By hand that means a cross join, a window, and a filter. NEAREST BY replaces that.

### [32] NEAREST BY: top-K per query row · XIAO · ~0:30
Now we generalize: from one query vector to many query rows. For each left row, find the K best right rows. The score is usually similarity, but it can be any expression. The old way — cross join, ROW_NUMBER, filter — hides the intent; NEAREST BY makes it explicit. In 4.2, EXACT is brute-force top-K, just with clearer SQL. APPROX keeps the same shape ready for future index-backed plans.

### [33] NEAREST BY in action: batch vector search · XIAO · ~0:26
The same search, now batched: for every user, the top 10 products by similarity. The point is the shape, not the function — no cross join, no window, no filter. Vector search is the common case, but NEAREST BY fits any top-K join. Demo EXACT; APPROX keeps the query stable for smarter execution later.

### [34] SQL gets more natural · XIAO · ~0:35
Before and now, four times. Each one removes a pattern we have written for years. QUALIFY: top-N without a subquery. FILTER: conditional aggregates now run over a window too — SUM(...) FILTER (WHERE ...) OVER (...). That was rejected before 4.2. It is SPARK-55702. PIVOT with aliases: readable columns. And cursors: row-by-row logic, in pure SQL. Less boilerplate, more intent. Takeaway: reach for QUALIFY first, and use FILTER over a window where you used a subquery.

### [35] SQL compatibility: shorter names, smoother migrations · XIAO · ~0:32
SET PATH is the SQL search path. Before, every name needed its full catalog and schema. Now you set the path once, and unqualified names resolve across schemas. It is saved in views, so resolution stays reproducible. It is opt-in — spark.sql.path.enabled — and it is not an import; objects stay where they are. Takeaway: porting PostgreSQL-style SQL? Start with SET PATH.

### [36] More compatibility: richer types & built-ins · XIAO · ~0:35
A few more compatibility wins. Richer types: the TIME type now works across more formats — but it is a preview, off by default in production behind spark.sql.timeType.enabled, so do not live-demo it unflagged. TIMESTAMP WITH LOCAL TIME ZONE is now in SQL. More built-ins: top-K max_by and min_by, time_bucket, and reverse on binary. And IGNORE/RESPECT NULLS is now honored by array_agg, collect_list, and collect_set — the clause is not new; these aggregates just gained support.

### [37] Data sketches: approximate, mergeable analytics · XIAO · ~0:45
Sketches are small, probabilistic summaries. One pass. Small memory. About 1 to 2% error — good enough when the error budget fits. Much of this is already in Spark, as SQL aggregates. KLL for quantiles, Theta for distinct counts and set ops, and Approx Top-K — all since 4.1. HLL since 3.5. New in 4.2: tuple sketches. A distinct count plus a metric, like revenue, in one pass. The best part is they merge. Store them as Delta columns. Then combine any time range in milliseconds, with no rescan. They work with open Apache DataSketches, so they merge across engines.

### [38] Tuple sketches in action · XIAO · ~0:22
(Code slide.) A tuple sketch. Distinct users and total revenue, in one pass. Then we merge daily sketches into a month — instantly, with no rescan.

### [39] Native geospatial types · XIAO · ~0:35
Location data is everywhere — delivery, IoT, maps, risk. 4.2 adds GEOMETRY and GEOGRAPHY as native SQL types. On by default. No extra package. GEOMETRY is flat. GEOGRAPHY is on the globe. There is full SRID support. You read and write in Parquet, and the type and SRID are kept. It works in SQL, DataFrames, and PySpark. And it follows the open Parquet and Iceberg geospatial spec, so it works across engines.

### [40] Geospatial in action · XIAO · ~0:35
(Code slide. Claim only the shipped functions.) These ship in 4.2: ST_GeomFromWKB, ST_GeogFromWKB, ST_AsBinary, and the SRID helpers. Build a geometry, write it to Parquet, read it back. The type and SRID stay.
▶ HANDOFF — XIAO → DB. (XIAO:) So far, 4.2 brings more logic into the engine — metrics, Python, and SQL. But analytics is not only querying. The data changes underneath you. DB will show how 4.2 makes change itself safer — CDC, streaming, DSv2, and operations. DB.

## Section 05 · Pipelines & Auto CDC — Spark Declarative Pipelines & Auto CDC

### [41] Section divider — Declarative Pipelines & Auto CDC · DB · ~0:12
Thank you, Xiao. Benefit four: move changing data safely. Let's start with a hard problem — keeping a table in sync with a stream of changes. Takeaway: move the reconciliation logic into a declared Auto CDC flow.

### [42] Applying a change feed is the hard part · DB · ~0:40
The common need: keep a lakehouse table in sync with an operational change feed. Here is one customer. Version 1 says SF. Version 3 says Seattle. Then version 2 arrives late and says LA. If you process by arrival order, LA can overwrite Seattle, even though LA is older. Then version 4 deletes the row. A CDC event has four parts: key, operation, sequence, and payload. The key is the row identity. The sequence is the business order. Hand-written foreachBatch plus MERGE turns that simple intent into ordering, dedupe, tombstone, and retry code. That is where small mistakes happen.

### [43] Auto CDC: key plus sequence is the contract · DB · ~0:45
This is the mental model. Spark Declarative Pipelines is the pipeline manager. Auto CDC is one special flow inside it. You give it the key — customer_id — the sequence column — version — and the storage mode, SCD Type 1 in Spark 4.2. That tells Spark: group changes by customer, and trust version as the real order, not arrival time. For customer 101, the logical order is version 1 SF, version 2 LA, version 3 Seattle, version 4 delete. So for SCD Type 1, the target keeps only the latest current value. With the delete, 101 is gone. Without the delete, 101 would be Seattle, not LA. New in 4.2: the Python API, create_auto_cdc_flow(), plus Connect support. It stores SCD Type 1; SQL and SCD Type 2 are coming.

### [44] Auto CDC in action · DB · ~0:35
(Code slide.) Same idea, now in code. This lives inside a Declarative Pipelines file. create_auto_cdc_flow registers a flow; it is not a standalone repair job. The magic is keys plus sequence_by. Keys group the changes. sequence_by gives the real order. For SCD Type 1, Spark keeps the latest current value. In this example, customer 101 is gone because version 4 is a delete, and customer 102 remains NY. If there were no delete, customer 101 would be Seattle, not LA. SCD Type 2 would keep the history — SF, LA, Seattle — and close the active row at delete time. Important caveat: in Spark 4.2, Auto CDC ships for SCD Type 1 only. SCD Type 2 is shown here as the model, but it is coming later. Now — streaming.

## Section 06 · Streaming — Structured Streaming

### [45] Section divider — Structured Streaming · DB · ~0:12
Still benefit four. Now streaming. Two things: change a running query safely, and a state store you can trust. Takeaway: name your streaming sources, so you can evolve them safely.

### [46] Evolve running pipelines safely · DB · ~0:32
A long-time problem: streaming sources were identified by position. So you could not add, remove, or reorder them without breaking the checkpoint. Now you name them — IDENTIFIED BY in SQL, and DataStreamReader.name() in PySpark, both Classic and Connect. Identity is the name, not the position. So you can change sources in a running query and keep the checkpoint. (Sink naming exists in Scala internally, but it is not a public PySpark API. So do not demo df.writeStream.name().)

### [47] IDENTIFIED BY in action · DB · ~0:18
(Code slide.) Named sources. A query adds a new one, restarts, and keeps its checkpoint. No full reprocess.

### [48] Richer joins, lower latency · DB · ~0:25
Streaming joins get richer. Inner and LeftSemi stream-stream joins now run in Update mode. LeftSemi uses less state-store space. And the real-time mode trigger is now in PySpark. Remember it — we come back to it later.

### [49] A state store you can trust · DB · ~0:25
The state store is the heart of stateful streaming. In 4.2 it is stronger. It repairs a corrupted snapshot by itself. Checksums catch corruption early. And slow snapshot uploads are handled — on by default. Fewer pages at night.

## Section 07 · Data Source V2 — Data Source V2

### [50] Section divider — Data Source V2 · DB · ~0:12
Still benefit four. Now Data Source V2 — how Spark connects to data. It improved a lot this release. Takeaway: read changes with one CHANGES query, not per-engine functions.

### [51] One integration API for every data source · DB · ~0:33
DSv2 is the standard API for data sources — Delta, Iceberg, and more. Users get the same SQL and behavior across sources. Connectors get DML, streaming, and Spark's optimizations. It is growing fast. In 4.1 and 4.2: row-level DML, CDC, schema evolution, transactions, and better pushdown. There is a full talk at this Summit: 'Spark DSV2: Growing Up Fast', by Szehon Ho and Anton Okolnychyi.

### [52] Ask the table: "what changed?" · DB · ~0:32
CDC is the row-level history of a table — which rows were added, updated, or deleted. It is a typed log, in commit order. Each row is tagged: insert, update before, update after, or delete. Each row has the values, the operation, and the order. CDC is used everywhere — incremental ETL, audit, replication. And it refreshes vector indexes, materialized views, and streaming tables on commit.

### [53] Two problems with CDC today · DB · ~0:40
CDC today has two problems. One: every engine built its own. Delta has table_changes(). Iceberg has create_changelog_view. Hudi has incremental reads. Same idea, three syntaxes. Two: write-time CDC costs every writer. Change files are written on every update — about 1.2x the write time. You pay this even if no one reads them. Some numbers: 206 million CDC queries in 60 days. And 68% of them needed no stored change files at all. So 4.2 makes read-time CDC built in. Only the reader pays. And any engine can write the table.

### [54] One CHANGES API for changelog-capable DSv2 sources · DB · ~0:35
The answer is one API. One SQL CHANGES clause — by version or by time, or streaming with STREAM ... CHANGES. The same shape in DataFrames — read.changes() and readStream.changes(). The same syntax for any DSv2 connector that supports a changelog — Delta, Iceberg, and Hudi today. Not every source has it; the connector must ship the interface. Spark does the hard part — removing carry-overs, finding updates, collapsing changes. Connectors ship only a small contract.

### [55] Reading changes in action · DB · ~0:20
(Code slide.) SELECT ... FROM t CHANGES. A version range in, typed change rows out. The same query for Delta or Iceberg.

### [56] Delta goes read-time — by default · DB · ~0:40
For a Delta table with row tracking, the result is great: read-time CDC, with no Change Data Feed property. No CDF setting. No ALTER TABLE. No ticket to the owner. And you pay nothing at write time. Changes are computed only when a reader asks. It uses two columns Delta already has — a stable row id and a row commit version. A matched id is an update. A lone add is an insert. A lone remove is a delete. It needs Row Tracking — default in Databricks, opt-in in Delta OSS. Write-time CDC still works for special cases.

### [57] Smarter writes: INSERT & MERGE · DB · ~0:30
Writes are smarter and safer. INSERT INTO ... WITH SCHEMA EVOLUTION lets the target grow with the data. A source with fewer columns is fine — missing fields are filled. There is new INSERT ... REPLACE syntax, and BY NAME with REPLACE WHERE. MERGE gets fixes — schema evolution with DELETE, and nested fields kept. And TABLESAMPLE SYSTEM, pushed down to DSv2 and JDBC.

### [58] Schema evolution in action · DB · ~0:18
(Code slide.) A new column appears in the source. With SCHEMA EVOLUTION, the target takes it. No manual ALTER.

### [59] Transactions: atomic, isolated DML · DB · ~0:30
DML is read, transform, write back. Without isolation, a writer can overwrite changes it never saw. So 4.2 starts a transaction for every DML. It tracks all reads and writes in it. Connectors check concurrent commits at commit time. So a stale read fails cleanly, instead of corrupting data. This is single-statement DML today. Multi-statement and cross-catalog are future work.

### [60] Observable, evolvable DML · DB · ~0:28
DML is observable now. MERGE, UPDATE, and DELETE report metrics — rows inserted, updated, deleted, copied. You see them where you already look — Delta DESCRIBE HISTORY, Iceberg snapshots. MERGE and INSERT support schema evolution, and the connector decides what is valid. And table refresh is consistent, so views over external tables stay correct.

### [61] Smarter pushdown: PartitionPredicate · DB · ~0:30
This is a real gap, and connectors hit it every day. Before, partition filters with UDFs or unusual expressions were lost between Catalyst and the connectors. File sources could use them. Connectors could not. The new PartitionPredicate API fixes this. Any Catalyst partition filter is checked against the partition values — for scans, DELETE WHERE, and runtime filters. So WHERE udf(month(ts)) = 'JAN' now prunes partitions in Iceberg and Delta. Less data read. Faster queries.

## Section 08 · Others — Performance, UI & Operations

### [62] Section divider — Performance, UI & Operations · DB · ~0:12
Benefit five: operate and evolve predictably. First, what makes Spark faster, easier to see, and easier to run. Takeaway: upgrade and get speed and stability, with no code change.

### [63] A modern Spark Web UI · DB · ~0:25
The Web UI has a new look. Dark mode, and a faster interface. Interactive SQL plans — you can pan, zoom, search, and compare the first and final AQE plans, side by side. And the environment page shows your non-default configs, with one-click export.

### [64] Faster & leaner · DB · ~0:28
Four performance points. Faster scans — better vectorized Parquet. Smarter plans — pre-aggregation for many COUNT(DISTINCT), and more codegen. Leaner memory — bounded merge and early release cut OOMs. And faster I/O — zero-copy transfers, and less JVM-to-Python overhead. All with no code change.

### [65] Runtime & operations · DB · ~0:28
On operations: we now run on Java 25. Kubernetes — in-place executor and PVC resizing, smaller images, NetworkPolicy isolation. The History Server scales — multiple log folders, and on-demand loading. And consistent results — query-level retry for indeterminate shuffles, with correct SQL metrics under AQE. That is 4.2. Now — what is next.

## Section 09 · Ongoing Work — Looking Ahead

### [66] Section divider — Looking Ahead · DB · ~0:12
Still benefit five. This last part is ongoing work, already in progress for the next releases. It shows where Spark is going. Takeaway: follow the SPIPs, try the previews, and plan around the quarterly cadence.

### [67] Roadmap (five topics) · DB · ~0:12
(Gesture across the five.) Five things ahead: Project Feather, a language-agnostic UDF protocol, nanosecond timestamps, real-time mode for stateful streaming, and a faster release cadence. We will touch each one.

### [68] Project Feather: fast local queries · DB · ~0:32
First, Project Feather. The goal: make small queries fast on a laptop. Spark uses one API for big and small jobs. But for small jobs, the fixed costs are too high. A query over less than 100 MB can still take seconds — because planning, scheduling, serialization, and shuffle were built for big clusters. Feather works on three areas: planning and scheduling, the cache format, and shuffle.
Here is the key difference. Unlike DuckDB or Polars, the value of Feather is this: many users want to start small on a laptop and scale to big data later — without switching APIs, engines, or mental models. If Spark can feel lightweight for local experimentation while still scaling to production workloads, that is a much smoother path from prototype to production.

### [69] Project Feather: a three-part plan · DB · ~0:32
Feather has three parts. One: faster planning and scheduling. A single-pass analyzer, and a one-file query that runs as a single task, with no shuffle. Two: an Arrow columnar cache, to replace the row-based df.cache, for faster re-reads. And UDFs now take Arrow as input too — so the data stays Arrow end to end, with no format change across the pipeline. Three: shuffle-free local execution. Threads pass data through channels, instead of shuffle files. Together, Spark works on a laptop, and still scales to the cluster.

### [70] Language-agnostic UDF protocol · DB · ~0:35
Remember the Connect clients — Go, Rust, Swift, and more. UDFs are the one thing Connect cannot give most of them. SPIP SPARK-55278 fixes that. Today, each language rebuilds the whole UDF stack. The planning rules are tied to Python. Serialized UDFs are tied to a runtime — a server upgrade can break them. And UDF execution is locked in the cluster. So a heavy or GPU UDF forces an over-sized cluster.

### [71] Three pillars · DB · ~0:32
The design has three parts. One: plan UDFs by their shape — scalar, map, grouped map, table, aggregate — not by language. Two: one execution protocol — init, data, finish — over gRPC and Arrow. It is versioned, and has back-pressure. Three: a worker spec. The client says how to start, connect, and clean up a worker — local, container, or remote GPU. Each part can change without touching the others.

### [72] Status & what it unlocks · DB · ~0:25
Status: the SPIP vote passed. First code is landing in apache/spark, under /udf/worker. PySpark behavior does not change — one shared core, with pluggable transport. Socket stays the default. gRPC is opt-in. It unlocks UDFs in any language, runtimes that upgrade on their own, and heavy or GPU UDFs on separate workers.

### [73] Nanosecond-precision timestamps · DB · ~0:32
SPIP SPARK-56822. Today, Spark timestamps stop at microseconds. So nanosecond Parquet either fails, or falls back to a plain long — and loses the timestamp meaning. This adds parameterized types — TIMESTAMP(n), with n from 0 to 9. 6 is micros, 9 is nanos. The value model is compact, and keeps today's date range. It is fully backward compatible. Micro types stay the default. Nanosecond behavior appears only when you ask.

### [74] Real-time mode for stateful streaming · DB · ~0:35
Earlier I mentioned real-time mode. SPARK-54699 extends it to stateful queries — about 100 milliseconds latency. It builds on stateless real-time mode from 4.1, now with the same low latency for transformWithState and aggregations. Two parts make it work. A streaming shuffle sends data straight to the next task, instead of waiting for the batch. And concurrent stage scheduling runs many stages at once. Last, how we ship all of this.

### [75] Faster, predictable releases · DB · ~0:40
And how we ship it. SPARK-54633 — a two-layer model: quarterly minor releases, and one major a year. Minors do not change dependencies or defaults, so upgrades stay safe. Majors carry the breaking changes. The last minor of each major is an 18-month LTS. 4.2 is the bridge. The quarterly train starts at 4.3. As a transition exception, the 4.x LTS is 4.5.0 — the last 4.x release, around March 2027 — not 4.3. Then Spark 5.0 follows, around June 2027. Takeaway: plan upgrades around the quarterly cadence, and target the 4.5 LTS.

### [76] Join the community today! · DB + XIAO · ~0:20
(DB:) All of this is built by the Apache Spark community. And you can join. Get the release at spark.apache.org. Get the source on GitHub. And join the mailing lists. We welcome your contributions and your bug reports. (XIAO:) Thank you, DAIS. Enjoy the rest of the Summit.