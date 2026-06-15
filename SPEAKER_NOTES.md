# Speaker Notes — The Upcoming Apache Spark™ 4.2
### DAIS 2026 keynote · two speakers · 76 slides

**Speakers:** XIAO = Xiao Li (gatorsmile) · DB = DB Tsai (dbtsai)

**Split:** Xiao presents slides 1–38; DB presents 39–76. One handoff, at the end of the Spark SQL section.

**Through-line (say it often):** *Spark 4.2 moves more of the modern data and AI stack into the engine itself.* Five benefits — (1) Define truth once · (2) Reach Spark from everywhere · (3) Run AI-native analytics in SQL · (4) Move changing data safely · (5) Operate & evolve predictably.

**PySpark hierarchy:** No code change (Arrow) → Small change (Arrow UDFs) → New capability (Python Data Sources) → Dev experience. On-screen perf claims are qualitative; say numbers verbally with "in our benchmarks."

**Style:** Short, simple sentences; read slowly. *(parentheses)* are stage directions. Section dividers carry a **Today → Spark 4.2 → Takeaway** triad.

---

### [1] The Upcoming Apache Spark™ 4.2 · XIAO · ~0:40
Good morning. Thank you for coming. I am Xiao Li. I work on Apache Spark at Databricks.
(DB:) And I am DB Tsai, also on the Spark team at Databricks.
(XIAO:) Today we will show you what is new in Apache Spark 4.2. I will present the first half. DB will present the second half. The theme this year is: native analytics for data and AI. The common thread is that Spark is absorbing more of the analytics stack into the engine. Let's start.

### [2] About us · XIAO · ~0:15
A quick note first. Spark 4.2 is built by the whole Apache Spark community. Hundreds of people. We are here to share their work. Everything we show today is open source.

### [3] Apache Spark™ 4.2 — Overview · XIAO · ~1:15
Here is Spark 4.2 in one sentence. Spark 4.2 brings more semantic, AI, Python, CDC, and operational capabilities into the engine. The numbers: more than 1,600 resolved JIRAs, from more than 260 contributors. But one goal ties it together — make analytics and AI workloads more native to Spark. Everything we show today serves that goal.

### [4] What we'll cover today · XIAO · ~0:45
We organized the talk around five benefits — five things 4.2 does for you. One: define truth once — metric views. Two: reach Spark from everywhere — Spark Connect, PySpark, Arrow, and Python Data Sources. Three: run AI-native analytics in SQL — vector search, sketches, geospatial, and SQL improvements. Four: move changing data safely — Auto CDC, CHANGES, Data Source V2, and streaming. Five: operate and evolve predictably — the Web UI, Project Feather, and a faster release cadence. Every section maps to one of these. I will cover the first benefits. DB will cover the rest.

### [5] The features at a glance · XIAO · ~0:40
This is the whole release on one slide. Six groups. About 24 features. We will not cover all of them. Please just see how much is here. Keep this slide as a map. Let's start with the foundation.


## Section 01 · Metrics — Metrics & Semantic Modeling

### [6] Section divider — Metrics & Semantic Modeling · XIAO · ~0:10
Benefit one: define truth once. Let's start with one number that quietly breaks. And the takeaway: move your top dashboard metrics into a metric view.

### [7] Some metrics don’t add up · XIAO · ~0:45
Let me start with one number that quietly breaks. Two countries. Revenue and active users add up fine — 200 and 15. But revenue per user is a ratio. Average the two rows and you get 15. That is wrong. The correct answer is total revenue over total users — 200 over 15 — which is 13.33. So the real question is: where should this metric be defined, so it comes out right every time? That is a semantic-layer problem.

### [8] Why this is a semantic-layer problem · XIAO · ~1:05
Every company already has a semantic layer. But it is scattered. Words like "revenue," "active users," and "churn" are defined again and again — in BI tools, in dbt, in notebooks, in copy-pasted SQL.
The cost is real. Dashboards do not agree. Teams waste time checking numbers. No one owns the real definition.
This is not about style. It is about correctness. Ratios and COUNT(DISTINCT) break when you group or roll up the data. The number looks fine, but it is wrong.
Client-side tools sit outside the engine. So they cannot enforce correctness, and permissions are not consistent.
And now AI needs this too. An agent must answer "revenue per customer in the EU last quarter" the same way every time. A governed metric view gives both BI and AI one source of truth.

### [9] Metric views: define metrics once · XIAO · ~1:05
So 4.2 adds metric views — SPIP SPARK-54119. You define a business metric once. Then you query it by any breakdown.
There are two parts. Dimensions — the ways to slice. And measures — a new column type that aggregates without a fixed GROUP BY.
This stops metric drift. Ratios and distinct counts are correct for every grouping. And measures can build on other measures.
It lives in the engine and the catalog. So the optimizer enforces correctness, and permissions stay the same everywhere.
One source of truth — for SQL, for BI, and for AI. Let me show an example.

### [10] Metric views in action · XIAO · ~0:40
You define the view once — the dimensions and the measures. Then look at the queries. The same metric, by region, by month, and the total. The numbers are correct at every level, because the engine knows how to aggregate each measure. You write it once. Everyone else just asks questions. Now — how do you connect to Spark?


## Section 02 · Connect — Spark Connect

### [11] Section divider — Spark Connect · XIAO · ~0:12
Benefit two: reach Spark from everywhere. First, Spark Connect — how more and more applications talk to Spark. Takeaway: call Spark with Connect from your service's own language.

### [12] Architecture: gRPC in, Arrow out · XIAO · ~0:45
The idea is simple. Your client builds a plan. It opens a gRPC stream to the server. The server resolves the plan, optimizes it, runs it on the executors, and sends Arrow batches back.
The client is thin. It does not need a full Spark runtime. It does not need a JVM next to your app. So the client and the cluster can change on their own. The protocol connects them. The point: your app does not need to be a Spark app.

### [13] Clients in any language · XIAO · ~0:35
Because the client only speaks the Connect protocol over gRPC, your app can use Spark from many languages — Python, Scala, Java, and also Go, Rust, Swift, TypeScript, .NET, and more. For example, a Go service can call Spark without embedding a JVM. You do not rewrite your stack. Here is the through-line: Connect makes Spark reachable from any app. The next frontier is making your custom logic — your UDFs — portable too. DB comes back to that in Looking Ahead.

### [14] Closing the gap with Spark Classic · XIAO · ~0:40
In 4.2 we keep closing the gap with the classic driver. Old RDD helpers come to Connect — zipWithIndex, toJSON, emptyDataFrame. read.json, csv, and xml can now take a DataFrame. There is a new GetStatus API. Errors are clearer — for example, when you use a column from the wrong DataFrame. And client file names and line numbers are saved, so debugging is easier.


## Section 03 · Python — PySpark, Faster by Default

### [15] Section divider — PySpark, Faster by Default · XIAO · ~0:12
Still benefit two — reach Spark from everywhere. Now Python, the language most of you use with Spark. The headline is in the title: faster by default. Takeaway: upgrade for free speedups, and use @arrow_udf on hot paths.

### [16] Arrow on by default · XIAO · ~0:40
The important part is the default. In 4.2, Arrow Python UDFs are the default — your old UDFs just run faster. Arrow IPC between the JVM and Python is on, too. And Arrow input skips an extra conversion step. This is the upgrade story: get this speed with no code change.

### [17] Python UDFs: same code, now faster · XIAO · ~0:25
(Code slide.) Same UDF you always write. No new decorator. No rewrite. But the engine now moves the data as Arrow. Same code — faster.

### [18] Native Arrow UDFs: write on columns, skip the copies · XIAO · ~0:50
Two new decorators — @arrow_udf and @arrow_udtf. PyArrow array in, PyArrow array out. The data stays columnar.
No pandas conversion cost. In our benchmarks, about 10 percent faster and about 40 percent less memory than a pandas UDF.
You get scalar, aggregate, and table functions. Iterator mode lets you set up once — for example, load a model once per batch.
There is also mapInArrow and applyInArrow at the DataFrame level. And if you keep your pandas UDFs, the new Arrow backend makes them about 2 times faster in our benchmarks — with no code change.

### [19] Arrow UDFs in action · XIAO · ~0:25
(Code slide.) Here is the new decorator. Arrays in, arrays out. No to_pandas. No round trip. Columnar from start to end.

### [20] Python Data Sources: connectors in pure Python · XIAO · ~0:50
You can now build readers and writers fully in Python. Batch and streaming. No JVM. No Scala.
Register it once. Then read and write with spark.read.format — just like a built-in source.
The API keeps growing — pushdown, Arrow writers, streaming, partitioning.
And there is a real ecosystem. The community package pyspark-data-sources has many ready connectors — Hugging Face, Google Sheets, Kaggle, Salesforce, and more.
One more thing for the AI era: an LLM can scaffold the long-tail connector — and you still test it, profile it, and harden it like normal code.

### [21] A growing Python Data Source ecosystem · XIAO · ~0:30
(Gesture at the logos.) This is the ecosystem today. Connectors the community already built, in pure Python. Before, you had to learn the JVM API. Now you just write Python.

### [22] Python Data Sources in action · XIAO · ~0:25
(Code slide.) Here is a full custom source. A reader class. Register it. Read it like any format. That is the whole thing.

### [23] Profiling Python Data Sources · XIAO · ~0:35
New in 4.2: you can profile these Python data sources, like UDFs — for time and memory. Turn it on with one config. Then see where time and memory go in your read and write code. Your connector is no longer a black box.

### [24] Better interop & developer experience · XIAO · ~0:40
This is small individually, but big in daily debugging. PyCapsule support: zero-copy sharing with Polars, DuckDB, and Arrow tools. You can profile Python data sources. builder.create() makes a new session without changing other configs. An optional strict mode catches unclear column names early. And pandas-on-Spark supports more axis=1 functions. Each one is minor alone — together, they save you time every day.

### [25] PyCapsule interop in action · XIAO · ~0:30
(Code slide.) Here a Spark DataFrame goes straight to Polars or DuckDB. No serialization between them. They share the same Arrow memory. Zero copy.

### [26] Debug UDFs where they run · XIAO · ~0:45
Now debugging. Your UDF runs in a Python worker on an executor — far from your notebook. When it hangs, before, you saw nothing.
In 4.2, hangs are visible. The worker dumps its stack trace. Idle workers can be dumped and killed. There is one profiler for Python, pandas, and Arrow UDFs — CPU time, and memory by line. It works on Spark Connect too. And you can use Python logging inside the UDF, then read the logs back as a table.

### [27] UDF debugging in action · XIAO · ~0:25
(Code slide.) Here is a stuck worker — and its stack trace, shown automatically. And here are UDF logs, read back as a normal table.


## Section 04 · New SQL Analytics — Spark SQL

### [28] Section divider — Spark SQL · XIAO · ~0:12
Benefit three: run AI-native analytics in SQL. This release added the most features here. We split it into four themes. Takeaway: replace a cross-join plus ROW_NUMBER top-K with NEAREST BY.

### [29] Query embeddings where your data lives · XIAO · ~0:45
Embeddings are everywhere now — search, recommendations, dedup, RAG. Usually you run a separate vector system. In 4.2, Spark adds built-in vector functions. So you can score similarity where your data already is — with no extra system. Distance and similarity functions. Norm and normalize. And avg and sum for vectors. These are the building blocks for the new NEAREST BY join.

### [30] Vector functions in action · XIAO · ~0:25
(Code slide.) Cosine similarity, L2 distance — called directly in SQL, on a column of embeddings, at Spark scale.

### [31] NEAREST BY: top-K ranking join · XIAO · ~0:50
This is a new join clause — SPIP SPARK-56395. For each query row, it returns the top-K closest rows.
Before: a cross join, a window, a rank, and a filter — slow, and the optimizer cannot understand it. Now: NEAREST 10 BY SIMILARITY — one clause the optimizer plans for you.
It can rank by any expression — vector similarity, distance, geospatial, even BM25. INNER drops rows with no match. LEFT OUTER keeps them.
EXACT is brute force today. APPROX lets a future index help — without changing your query. And an index never changes your results silently.

### [32] NEAREST BY in action · XIAO · ~0:25
(Code slide.) One query: for each row, give me the five nearest. That is it. No window. No cross join.

### [33] SQL gets more natural · XIAO · ~0:45
Before and now, four times — each one removes a pattern many of us have written for years. QUALIFY: top-N without a subquery. FILTER: conditional aggregates without a CASE. PIVOT with aliases: readable pivot columns. And cursors: row-by-row logic, now in pure SQL — no need to leave SQL for a loop. Less boilerplate, more intent. Takeaway: reach for QUALIFY and FILTER first — they remove the most code.

### [34] SQL compatibility improvements · XIAO · ~0:40
A quick compatibility round. SET PATH resolves names across schemas without the full path, and it is saved in views, so results stay reproducible — this helps PostgreSQL migrations. (If you demo it: it is off by default — first SET spark.sql.path.enabled=true, then SET PATH sales, shared, system.builtin, and check with SELECT current_path().) The type system is more complete: the TIME type works across more formats — but call it a preview: it is off by default in production builds (spark.sql.timeType.enabled), so do not live-demo TIME on a vanilla 4.2 build without enabling the flag. TIMESTAMP WITH LOCAL TIME ZONE is in SQL. Plus top-K max_by/min_by, time_bucket, and reverse on binary. And IGNORE/RESPECT NULLS is now honored by array_agg, collect_list, and collect_set — the clause itself is not new in 4.2; these aggregates just gained support. Takeaway: if you are porting SQL from another database, these close common gaps.

### [35] Data sketches: approximate, mergeable analytics · XIAO · ~0:55
Sketches are small, probabilistic summaries. One pass. Small memory. About 1 to 2% error — bounded-error answers, decision-grade when the error budget fits.
Much of this is already in Spark, as SQL aggregates. KLL for quantiles — P50, P90, P99 — since 4.1. Theta for distinct counts with set operations — since 4.1. Approx Top-K for frequent items — since 4.1. HLL for cardinality — since 3.5.
New in 4.2: tuple sketches — distinct count plus a metric, like revenue, in one pass.
The best part: they merge. Store sketches as Delta columns. Then combine them for any time range in milliseconds — with no rescan of raw data. And they work with the open Apache DataSketches project, so they merge across engines.

### [36] Tuple sketches in action · XIAO · ~0:30
(Code slide.) A tuple sketch — distinct users and total revenue, in one pass. Then we merge daily sketches into a month — instantly, with no rescan.

### [37] Native geospatial types · XIAO · ~0:45
Location data is everywhere — delivery, IoT, maps, risk. In 4.2, Spark adds GEOMETRY and GEOGRAPHY as native SQL types. On by default. No extra package. GEOMETRY is flat. GEOGRAPHY is on the globe. There is full SRID support. You read and write in Parquet, and the type and SRID are kept. It works in SQL, DataFrames, and PySpark. And it follows the open Parquet and Iceberg geospatial spec, so it works across engines.

### [38] Geospatial in action · XIAO · ~0:30
(Code slide. Only claim the shipped functions on screen.) Here are the functions that ship in 4.2 — ST_GeomFromWKB, ST_GeogFromWKB, ST_AsBinary, and the SRID helpers. Build a geometry, write it to Parquet, read it back — the type and SRID stay.
▶ HANDOFF — XIAO → DB. (XIAO:) So far, we saw how Spark 4.2 brings more logic into the engine — metrics, Python, and SQL. But analytics is not only about querying data. The data is changing underneath you. DB will now show how 4.2 makes change itself safer — CDC, streaming, Data Source V2, and operations. DB.


## Section 05 · Pipelines & Auto CDC — Spark Declarative Pipelines & Auto CDC

### [39] Section divider — Declarative Pipelines & Auto CDC · DB · ~0:15
Thank you, Xiao. Benefit four: move changing data safely. Let's start with a hard problem — keeping a table in sync with a stream of changes. Takeaway: replace custom foreachBatch plus MERGE with Auto CDC where it fits.

### [40] Applying a change feed is the hard part · DB · ~0:50
The common need: keep a copy of an operational table up to date. Rows are inserted, updated, deleted.
A CDC event has four parts: the key, the operation, the order, and the data.
This sounds easy, but it is not. Events arrive out of order, and across batches. Some events are partial. And retries must be safe.
The hand-written version — foreachBatch plus MERGE plus tombstones — is often hundreds of lines. Everyone writes it again, and everyone makes small mistakes.

### [41] Auto CDC: declare it, Spark reconciles it · DB · ~0:55
So 4.2 lets you declare it instead. You give the keys, the order, and the delete rule. Spark reconciles each batch against the target, in order, and merges it for you.
It is safe with out-of-order data. Inserts and updates both become upserts. Deletes come from the feed.
In 4.2 you get a Python API — create_auto_cdc_flow() — and Spark Connect support. It stores SCD Type 1. SQL syntax and SCD Type 2 are coming.
It runs inside Spark Declarative Pipelines. So checkpoints, retries, and idempotency are handled for you.

### [42] Auto CDC in action · DB · ~0:30
(Code slide.) Here is the whole thing. Declare the flow. Point it at the feed. Name the keys and the order column. These few lines replace the hundreds we just saw. Now — streaming.


## Section 06 · Streaming — Structured Streaming

### [43] Section divider — Structured Streaming · DB · ~0:12
Still benefit four — move changing data safely. Now streaming. It is about two things: changing a running query safely, and a state store you can trust. Takeaway: name your streaming sources so you can evolve them safely.

### [44] Evolve running pipelines safely · DB · ~0:40
A long-time problem: streaming sources were identified by position. So you could not add, remove, or reorder them without breaking the checkpoint.
Now you name them — IDENTIFIED BY in SQL, DataStreamReader.name(...) in PySpark (Classic and Connect; Scala too). Identity is no longer the position. So you can change the sources in a running query, and keep the checkpoint. (Sink naming exists internally in Scala, but it is not a public PySpark API — so do not demo df.writeStream.name().)

### [45] IDENTIFIED BY in action · DB · ~0:20
(Code slide.) Named sources. Then a query adds a new one, restarts, and keeps its checkpoint. No full reprocess.

### [46] Richer joins, lower latency · DB · ~0:35
Streaming joins get richer. Inner and LeftSemi stream-stream joins now run in Update mode. LeftSemi joins use less state-store space. And the real-time mode trigger is now in PySpark. (Remember real-time mode — we come back to it later.)

### [47] A state store you can trust · DB · ~0:35
The state store is the heart of stateful streaming. In 4.2 it is stronger. It can repair a corrupted snapshot automatically. Checksums find corruption early. And slow snapshot uploads are handled automatically — now on by default. Fewer pages at night.


## Section 07 · Data Source V2 — Data Source V2

### [48] Section divider — Data Source V2 · DB · ~0:15
Still benefit four — move changing data safely. Now Data Source V2 — how Spark connects to data. It improved a lot this release. Takeaway: read changes with one CHANGES query, not per-engine functions.

### [49] One integration API for every data source · DB · ~0:40
DSv2 is the standard API for data sources in Spark — Delta, Iceberg, and more. Users get the same SQL and behavior across sources. Connectors get DML, streaming, and Spark's optimizations.
It is growing fast. In 4.1 and 4.2: row-level DML, CDC, schema evolution, transactions, and better pushdown — all here.
There is a full talk at this Summit: "Spark DSV2: Growing Up Fast," by Szehon Ho and Anton Okolnychyi.

### [50] Ask the table: "what changed?" · DB · ~0:45
CDC is the row-level history of a table — which rows were added, updated, or deleted.
It is a typed log, in commit order. Each row is tagged: insert, update before, update after, or delete. Each row has the values, the operation, and the order.
CDC is important infrastructure. It powers incremental ETL, audit, and replication. And it refreshes vector indexes, materialized views, and streaming tables on commit.

### [51] Two problems with CDC today · DB · ~0:50
CDC today has two problems.
One: every engine built its own. Delta has table_changes(). Iceberg has create_changelog_view. Hudi has incremental reads. Same idea, three syntaxes.
Two: write-time CDC costs every writer. Change files are written on every update — about 1.2× the write time. You pay this even if no one reads them.
Some numbers: in 60 days, 206 million CDC queries. And 68% of them needed no stored change files at all.
So 4.2 makes read-time CDC first-class. Only the reader pays. And any engine can write the table.

### [52] One CHANGES API for any DSv2 source · DB · ~0:45
The answer is one API. One SQL CHANGES clause — batch by version or time, or streaming with STREAM ... CHANGES. The same shape in DataFrames — read.changes() and readStream.changes(). The same syntax for Delta, Iceberg, and Hudi.
Spark does the hard part — removing carry-overs, detecting updates, collapsing changes. Connectors only ship a small contract — one interface, three flags, and a row id and row version.

### [53] Reading changes in action · DB · ~0:25
(Code slide.) SELECT ... FROM t CHANGES. A version range in, typed change rows out. Same query for Delta or Iceberg.

### [54] Delta goes read-time — by default · DB · ~0:50
For a Delta table with row tracking, the result is great: read-time CDC with no Change Data Feed property to enable. No CDF table property. No ALTER TABLE. No ticket to the table owner.
And nothing to pay at write time. No change files. Changes are computed only when a reader asks.
It uses two columns Delta already has — a stable row id, and a row commit version. In one commit, removed and added rows with the same row id become an update. A lone add is an insert. A lone remove is a delete.
It needs Row Tracking — on by default in Databricks, opt-in in Delta OSS. Write-time CDC still works for special cases.

### [55] Smarter writes: INSERT & MERGE · DB · ~0:40
Writes are smarter and safer. INSERT INTO ... WITH SCHEMA EVOLUTION lets the target schema grow with the data. A source with fewer columns is accepted — missing fields are filled.
There is new INSERT ... REPLACE syntax, and BY NAME with REPLACE WHERE. MERGE gets fixes — schema evolution with DELETE, and nested fields kept. And TABLESAMPLE SYSTEM, pushed down to DSv2 and JDBC.

### [56] Schema evolution in action · DB · ~0:20
(Code slide.) A new column appears in the source. With SCHEMA EVOLUTION, the target takes it. No manual ALTER.

### [57] Transactions: atomic, isolated DML · DB · ~0:40
DML is read, transform, write back. Without isolation, a writer can overwrite changes it never saw.
So 4.2 starts a transaction for every DML, and tracks all reads and writes in it. Connectors check concurrent commits at commit time. A stale read fails cleanly — instead of corrupting data.
This is for single-statement DML today. Multi-statement and cross-catalog transactions are future work.

### [58] Observable, evolvable DML · DB · ~0:35
DML is observable now. MERGE, UPDATE, and DELETE report metrics — rows inserted, updated, deleted, copied. You see them where you already look — Delta DESCRIBE HISTORY, Iceberg snapshots. MERGE and INSERT support schema evolution, and the connector decides what is valid. And table refresh is consistent — so views over external tables stay correct.

### [59] Smarter pushdown: PartitionPredicate · DB · ~0:35
This is a real gap, and connectors hit it every day. Before, partition filters with UDFs or unusual expressions were lost between Catalyst and the DSv2 connectors. File sources could use them. Connectors could not.
The new PartitionPredicate API fixes this. Any Catalyst partition filter can be checked against partition values. It works for scans, DELETE WHERE, and runtime filters.
For example: WHERE udf(month(ts)) = 'JAN' now prunes partitions in Iceberg and Delta. Less data read. Faster queries.


## Section 08 · Others — Performance, UI & Operations

### [60] Section divider — Performance, UI & Operations · DB · ~0:12
Benefit five: operate and evolve predictably. First, the things that make Spark faster, easier to see, and easier to run. Takeaway: upgrade to 4.2 and pick up speed and stability with no code change.

### [61] A modern Spark Web UI · DB · ~0:30
The Web UI has a new look. Dark mode, and a faster interface. Interactive SQL plans — pan, zoom, search, and compare the first and final AQE plan side by side. And the environment page shows your non-default configs, with one-click export.

### [62] Faster & leaner · DB · ~0:35
On performance, four points. Faster scans — better vectorized Parquet reading. Smarter plans — pre-aggregation for many COUNT(DISTINCT), and more codegen. Less memory — bounded merge and early memory release cut OOMs. And faster I/O — zero-copy transfers, and less JVM-to-Python overhead. All with no code change.

### [63] Runtime & operations · DB · ~0:35
On operations: we now run on Java 25. Kubernetes — in-place executor and PVC resizing, smaller images, NetworkPolicy isolation. The History Server scales — multiple log folders, and on-demand loading. And consistent results — query-level retry for indeterminate shuffles, with correct SQL metrics under AQE. That is 4.2. Now — what is next.


## Section 09 · Ongoing Work — Looking Ahead

### [64] Section divider — Looking Ahead · DB · ~0:15
Still benefit five — operate and evolve predictably. This last part is ongoing work, already in progress for the next releases. We want to show you where Spark is going. Takeaway: follow the SPIPs, try the previews, and plan around the quarterly cadence.

### [65] Roadmap (five topics) · DB · ~0:12
(Gesture across the five.) Five things ahead: Project Feather, a language-agnostic UDF protocol, nanosecond timestamps, real-time mode for stateful streaming, and a faster release cadence. We will touch each one.

### [66] Project Feather: fast local queries · DB · ~0:45
First — Project Feather. The goal: make small queries fast on a laptop.
Spark uses one API for big and small jobs. But for small jobs, the fixed costs are too high. A query on less than 100 MB can still take seconds — because planning, scheduling, serialization, and shuffle were all built for big clusters.
Feather works on three areas: planning and scheduling, the cache format, and shuffle.

### [67] Project Feather: a three-part plan · DB · ~0:55
Feather has three parts. One — faster planning and scheduling: a single-pass analyzer, and running a small one-file query as a single task with no shuffle. Two — an Arrow columnar cache to replace the row-based df.cache, so re-reads are faster. Three — shuffle-free local execution: threads pass data through channels instead of shuffle files. Together, Spark works well on a laptop, and still scales to the cluster.

### [68] Language-agnostic UDF protocol · DB · ~0:45
Remember the Connect slide — clients in Go, Rust, Swift, and more. UDFs are the one thing Connect cannot give most of them. SPIP SPARK-55278 fixes this.
Today, each language must rebuild the whole UDF stack. The planning rules are tied to Python. Serialized UDFs are tied to a runtime — a server upgrade can break them. And UDF execution is locked inside the cluster — so a heavy or GPU UDF makes you over-size the cluster.

### [69] Three pillars · DB · ~0:40
The design has three parts. One: plan UDFs by their shape — scalar, map, grouped map, table, aggregate — not by language. Two: one execution protocol — init, data, finish — over gRPC and Arrow, versioned, with back-pressure. Three: a worker spec — the client says how to start, connect, and clean up a worker: local process, container, or remote GPU. Each part can change without touching the others.

### [70] Status & what it unlocks · DB · ~0:30
Status: the SPIP vote passed. First code is landing in apache/spark, under /udf/worker. PySpark behavior does not change — one shared core, with pluggable transport. Socket stays the default. gRPC is opt-in.
It unlocks UDFs in any language, runtimes that upgrade on their own, and heavy or GPU UDFs on separate workers.

### [71] Nanosecond-precision timestamps · DB · ~0:40
SPIP SPARK-56822. Today, Spark timestamps stop at microseconds. So nanosecond Parquet either fails, or falls back to a plain long — and loses the timestamp meaning.
This adds parameterized types — TIMESTAMP(n), with n from 0 to 9. 6 is micros, 9 is nanos. The value model is compact, and keeps today's date range.
It is fully backward compatible. The micro types stay the default. Nanosecond behavior only appears when you ask for it.

### [72] Real-time mode for stateful streaming · DB · ~0:40
Earlier I mentioned real-time mode. SPARK-54699 extends it to stateful queries — with about 100 milliseconds latency. It builds on stateless real-time mode, which shipped in 4.1 — now with the same low latency for transformWithState and aggregations.
Two parts make it work. A streaming shuffle — it sends data straight to the next task, instead of waiting for the batch. And concurrent stage scheduling — many stages run at the same time. The last part of the roadmap is how we ship all of this.

### [73] Faster, predictable releases · DB · ~0:50
And how we ship it. SPARK-54633 — a two-layer model: quarterly minor releases, and an annual major. Minors add features and APIs but freeze dependencies and defaults, so upgrades stay safe. Majors carry the breaking changes. Long-lived branches cut maintenance work, and the final minor of each major is an 18-month LTS. The quarterly cadence begins with 4.3 — so 4.2 is the bridge release, and 4.4 will be the LTS. Takeaway: plan your upgrades around the new quarterly cadence.

### [74] Join the community today! · DB + XIAO · ~0:20
(DB:) All of this is built by the Apache Spark community. And you can join. Get the release at spark.apache.org, the source on GitHub, and join the mailing lists. We welcome your contributions and your bug reports. (XIAO:) Thank you, DAIS. Enjoy the rest of the Summit.

