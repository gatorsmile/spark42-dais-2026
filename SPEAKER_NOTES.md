# Speaker Notes — *The Upcoming Apache Spark™ 4.2*
### DAIS 2026 keynote · ~40 minutes · two speakers

**Speakers:** **XIAO** = Xiao Li (gatorsmile) · **DB** = DB Tsai (dbtsai)

**Split:** **Xiao presents slides 1–51.** **DB presents slides 52–100.** One handoff, at the end of slide 51.

**Through-line (say it often):** *Spark 4.2 moves more of the modern data and AI stack into the engine itself.* The talk is organized around five audience benefits — and every section divider names its benefit, so keep repeating it: **(1) Define truth once · (2) Reach Spark from everywhere · (3) Run AI-native analytics in SQL · (4) Move changing data safely · (5) Operate & evolve predictably.**

**Style:** Short, simple sentences — written to be easy to say aloud. Read them as written, slow and clear. Pause at each period. Lines in *(parentheses italics)* are stage directions, not spoken.

**Pacing:** ~40 minutes. The code slides and the small roadmap slides should go fast — one or two short lines. If you run long, shorten the code slides first.

| Section | Slides | Speaker |
|---|---|---|
| Open | 1–5 | XIAO |
| 01 · Metrics & Semantic Modeling | 6–9 | XIAO |
| 02 · Spark Connect | 10–13 | XIAO |
| 03 · PySpark | 14–30 | XIAO |
| 04 · Spark SQL | 31–51 | XIAO |
| 05 · Pipelines & Auto CDC | 52–55 | DB |
| 06 · Structured Streaming | 56–60 | DB |
| 07 · Data Source V2 | 61–75 | DB |
| 08 · Performance, UI & Ops | 76–79 | DB |
| 09 · Looking Ahead | 80–97 | DB |
| Close (Thank you) | 98 | DB |

---

## OPEN — Slides 1–5 · XIAO

### [1] The Upcoming Apache Spark™ 4.2 · **XIAO** · ~0:40
Good morning. Thank you for coming. I am Xiao Li. I work on Apache Spark at Databricks.
*(DB:)* And I am DB Tsai, also on the Spark team at Databricks.
*(XIAO:)* Today we will show you what is new in Apache Spark 4.2. I will present the first half. DB will present the second half. The theme this year is: native analytics for data and AI. The common thread is that Spark is absorbing more of the analytics stack into the engine. Let's start.

### [2] About us · **XIAO** · ~0:15
A quick note first. Spark 4.2 is built by the whole Apache Spark community. Hundreds of people. We are here to share their work. Everything we show today is open source.

### [3] Apache Spark™ 4.2 — Overview · **XIAO** · ~1:15
Here is Spark 4.2 in one sentence. Spark 4.2 brings more semantic, AI, Python, CDC, and operational capabilities into the engine. The numbers: more than 1,600 resolved JIRAs, from more than 260 contributors. But one goal ties it together — make analytics and AI workloads more native to Spark. Everything we show today serves that goal.

### [4] What we'll cover today · **XIAO** · ~0:45
We organized the talk around five benefits — five things 4.2 does for you. One: define truth once — metric views. Two: reach Spark from everywhere — Spark Connect, PySpark, Arrow, and Python Data Sources. Three: run AI-native analytics in SQL — vector search, sketches, geospatial, and SQL improvements. Four: move changing data safely — Auto CDC, CHANGES, Data Source V2, and streaming. Five: operate and evolve predictably — the Web UI, Project Feather, and a faster release cadence. Every section maps to one of these. I will cover the first benefits. DB will cover the rest.

### [5] The features at a glance · **XIAO** · ~0:40
This is the whole release on one slide. Six groups. About 24 features. We will not cover all of them. Please just see how much is here. Keep this slide as a map. Let's start with the foundation.

---

## 01 · METRICS & SEMANTIC MODELING — Slides 6–9 · XIAO

### [6] Section divider — Metrics & Semantic Modeling · **XIAO** · ~0:10
Benefit one: define truth once. Let's start with the semantic layer inside Spark.

### [7] Why a semantic layer is critical · **XIAO** · ~1:05
Every company already has a semantic layer. But it is scattered. Words like "revenue," "active users," and "churn" are defined again and again — in BI tools, in dbt, in notebooks, in copy-pasted SQL.
The cost is real. Dashboards do not agree. Teams waste time checking numbers. No one owns the real definition.
This is not about style. It is about correctness. Ratios and `COUNT(DISTINCT)` break when you group or roll up the data. The number looks fine, but it is wrong.
Client-side tools sit outside the engine. So they cannot enforce correctness, and permissions are not consistent.
And now AI needs this too. An agent must answer "revenue per customer in the EU last quarter" the same way every time. A governed metric view gives both BI and AI one source of truth.

### [8] Metric views: define metrics once · **XIAO** · ~1:05
So 4.2 adds metric views — SPIP SPARK-54119. You define a business metric once. Then you query it by any breakdown.
There are two parts. Dimensions — the ways to slice. And measures — a new column type that aggregates without a fixed `GROUP BY`.
This stops metric drift. Ratios and distinct counts are correct for every grouping. And measures can build on other measures.
It lives in the engine and the catalog. So the optimizer enforces correctness, and permissions stay the same everywhere.
One source of truth — for SQL, for BI, and for AI. Let me show an example.

### [9] Metric views in action · **XIAO** · ~0:40
You define the view once — the dimensions and the measures. Then look at the queries. The same metric, by region, by month, and the total. The numbers are correct at every level, because the engine knows how to aggregate each measure. You write it once. Everyone else just asks questions. Now — how do you connect to Spark?

---

## 02 · SPARK CONNECT — Slides 10–13 · XIAO

### [10] Section divider — Spark Connect · **XIAO** · ~0:12
Benefit two: reach Spark from everywhere. First, Spark Connect — how more and more applications talk to Spark.

### [11] Architecture: gRPC in, Arrow out · **XIAO** · ~0:45
The idea is simple. Your client builds a plan. It opens a gRPC stream to the server. The server resolves the plan, optimizes it, runs it on the executors, and sends Arrow batches back.
The client is thin. It does not need a full Spark runtime. It does not need a JVM next to your app. So the client and the cluster can change on their own. The protocol connects them.

### [12] Clients in any language · **XIAO** · ~0:35
Because the client only speaks the Connect protocol over gRPC, your app can use Spark from many languages. Python, Scala, Java — and also Go, Rust, Swift, TypeScript, .NET, and more. You do not rewrite your stack. You call Spark from where you already work. *(Note: UDFs are still the gap — DB will talk about that later.)*

### [13] Closing the gap with Spark Classic · **XIAO** · ~0:40
In 4.2 we keep closing the gap with the classic driver. Old RDD helpers come to Connect — `zipWithIndex`, `toJSON`, `emptyDataFrame`. `read.json`, `csv`, and `xml` can now take a DataFrame. There is a new `GetStatus` API. Errors are clearer — for example, when you use a column from the wrong DataFrame. And client file names and line numbers are saved, so debugging is easier.

---

## 03 · PYSPARK — Slides 14–30 · XIAO

### [14] Section divider — PySpark, Faster by Default · **XIAO** · ~0:12
Still benefit two — reach Spark from everywhere. Now Python, the language most of you use with Spark. The headline is in the title: faster by default.

### [15] Roadmap (Arrow by default) · **XIAO** · ~0:08
*(One breath.)* Four things in the Python story. First — Arrow, on by default.

### [16] Arrow on by default · **XIAO** · ~0:40
This slide is simple: upgrade, and it is faster. In 4.2, Arrow Python UDFs are the default. Your old UDFs just run faster. Arrow IPC between the JVM and Python is on by default. And Arrow input skips an extra conversion step. Upgrade to 4.2 and get this speed — with no code change.

### [17] Python UDFs: same code, now faster · **XIAO** · ~0:25
*(Code slide.)* Same UDF you always write. No new decorator. No rewrite. But the engine now moves the data as Arrow. Same code — faster.

### [18] Roadmap (Native Arrow UDFs) · **XIAO** · ~0:06
*(One breath.)* If you change your code a little, you can go faster — native Arrow UDFs.

### [19] Native Arrow UDFs: write on columns, skip the copies · **XIAO** · ~0:50
Two new decorators — `@arrow_udf` and `@arrow_udtf`. PyArrow array in, PyArrow array out. The data stays columnar.
No pandas conversion cost. So it is about 10% faster, and uses about 40% less memory, than a pandas UDF.
You get scalar, aggregate, and table functions. Iterator mode lets you set up once — for example, load a model once per batch.
There is also `mapInArrow` and `applyInArrow` at the DataFrame level. And if you keep your pandas UDFs, the new Arrow backend makes them about 2× faster — with no code change.

### [20] Arrow UDFs in action · **XIAO** · ~0:25
*(Code slide.)* Here is the new decorator. Arrays in, arrays out. No `to_pandas`. No round trip. Columnar from start to end.

### [21] Roadmap (Interop & debugging) · **XIAO** · ~0:06
*(One breath.)* Third — interop and developer experience.

### [22] Better interop & developer experience · **XIAO** · ~0:40
A group of small but useful wins. PyCapsule support: zero-copy sharing with Polars, DuckDB, and Arrow tools. You can profile Python data sources. `builder.create()` makes a new session without changing other configs. An optional strict mode catches unclear column names early. And pandas-on-Spark supports more `axis=1` functions. Small things — but things you hit every day.

### [23] PyCapsule interop in action · **XIAO** · ~0:30
*(Code slide.)* Here a Spark DataFrame goes straight to Polars or DuckDB. No serialization between them. They share the same Arrow memory. Zero copy.

### [24] Debug UDFs where they run · **XIAO** · ~0:45
Now debugging. Your UDF runs in a Python worker on an executor — far from your notebook. When it hangs, before, you saw nothing.
In 4.2, hangs are visible. The worker dumps its stack trace. Idle workers can be dumped and killed. There is one profiler for Python, pandas, and Arrow UDFs — CPU time, and memory by line. It works on Spark Connect too. And you can use Python `logging` inside the UDF, then read the logs back as a table.

### [25] UDF debugging in action · **XIAO** · ~0:25
*(Code slide.)* Here is a stuck worker — and its stack trace, shown automatically. And here are UDF logs, read back as a normal table.

### [26] Roadmap (Python Data Sources) · **XIAO** · ~0:06
*(One breath.)* And fourth — my favorite — Python Data Sources.

### [27] Python Data Sources: connectors in pure Python · **XIAO** · ~0:50
You can now build readers and writers fully in Python. Batch and streaming. No JVM. No Scala.
Register it once. Then read and write with `spark.read.format` — just like a built-in source.
The API keeps growing — pushdown, Arrow writers, streaming, partitioning.
And there is a real ecosystem. The community package `pyspark-data-sources` has many ready connectors — Hugging Face, Google Sheets, Kaggle, Salesforce, and more.
One more thing for the AI era: give an LLM an API spec, and it can write a working data source for you.

### [28] A growing Python Data Source ecosystem · **XIAO** · ~0:30
*(Gesture at the logos.)* This is the ecosystem today. Connectors the community already built, in pure Python. Before, you had to learn the JVM API. Now you just write Python.

### [29] Python Data Sources in action · **XIAO** · ~0:25
*(Code slide.)* Here is a full custom source. A reader class. Register it. Read it like any format. That is the whole thing.

### [30] Profiling Python Data Sources · **XIAO** · ~0:35
New in 4.2: you can profile these Python data sources, like UDFs — for time and memory. Turn it on with one config. Then see where time and memory go in your read and write code. Your connector is no longer a black box.

---

## 04 · SPARK SQL — Slides 31–51 · XIAO

### [31] Section divider — Spark SQL · **XIAO** · ~0:12
Benefit three: run AI-native analytics in SQL. This release added the most features here. We split it into four themes.

### [32] Roadmap (Vector search) · **XIAO** · ~0:08
*(Point at item 1.)* We start with vector search.

### [33] Query embeddings where your data lives · **XIAO** · ~0:45
Embeddings are everywhere now — search, recommendations, dedup, RAG. Usually you run a separate vector system. In 4.2, Spark adds built-in vector functions. So you can score similarity where your data already is — with no extra system. Distance and similarity functions. Norm and normalize. And `avg` and `sum` for vectors. These are the building blocks for the new `NEAREST BY` join.

### [34] Vector functions in action · **XIAO** · ~0:25
*(Code slide.)* Cosine similarity, L2 distance — called directly in SQL, on a column of embeddings, at Spark scale.

### [35] NEAREST BY: top-K ranking join · **XIAO** · ~0:50
This is a new join clause — SPIP SPARK-56395. For each query row, it returns the top-K closest rows.
Today you write this as a cross join plus a `ROW_NUMBER()` window. It is slow, and the optimizer cannot understand it. `NEAREST BY` replaces all of that with one clause.
It can rank by any expression — vector similarity, distance, geospatial, even BM25. `INNER` drops rows with no match. `LEFT OUTER` keeps them.
`EXACT` is brute force today. `APPROX` lets a future index help — without changing your query. And an index never changes your results silently.

### [36] NEAREST BY in action · **XIAO** · ~0:25
*(Code slide.)* One query: for each row, give me the five nearest. That is it. No window. No cross join.

### [37] Roadmap (SQL language) · **XIAO** · ~0:06
*(One breath.)* Next — the SQL language itself. Cleaner, and easier to write.

### [38] Write window logic naturally · **XIAO** · ~0:40
A few SQL features people asked for a long time. `QUALIFY` — filter on a window result, with no subquery. `FILTER` now works inside window aggregates. `PIVOT` supports aliases. And SQL scripting now has cursors, for row-by-row work. Let me show a few.

### [39] QUALIFY in action · **XIAO** · ~0:18
*(Code slide.)* Top-N per group. No subquery. Just `QUALIFY`.

### [40] FILTER in window aggregates · **XIAO** · ~0:18
*(Code slide.)* A conditional running total. The `FILTER` is on the window aggregate.

### [41] PIVOT aliases & the | pipe · **XIAO** · ~0:18
*(Code slide.)* Aliased pivot columns, and the pipe syntax.

### [42] SQL scripting: cursors · **XIAO** · ~0:18
*(Code slide.)* A cursor — row-by-row logic, in pure SQL.

### [43] SQL search path: SET PATH · **XIAO** · ~0:35
`SET PATH` lets you find tables, functions, and variables across schemas — without writing the full name. `SET PATH` sets the order. `CURRENT_PATH()` shows what is active. The path is saved in views and SQL UDFs, and shown in `DESCRIBE` — so results stay the same. This helps if you migrate from PostgreSQL.

### [44] SET PATH in action · **XIAO** · ~0:18
*(Code slide.)* Set the path once. Then use short names. `DESCRIBE` shows the path the view saved.

### [45] A more complete type system · **XIAO** · ~0:35
A few type improvements. The `TIME` type now works in JSON, CSV, XML, ORC, and Avro, with casts from string. `TIMESTAMP WITH LOCAL TIME ZONE` is in SQL syntax. `IGNORE NULLS` and `RESPECT NULLS` for `array_agg`, `collect_list`, and `collect_set`. Plus top-K `max_by` and `min_by`, and `time_bucket` for time series.

### [46] Roadmap (Data sketches) · **XIAO** · ~0:06
*(One breath.)* Third theme — data sketches.

### [47] Data sketches: approximate, mergeable analytics · **XIAO** · ~0:55
Sketches are small, probabilistic summaries. One pass. Small memory. About 1 to 2% error. Approximate answers — but exact decisions.
Much of this is already in Spark, as SQL aggregates. `KLL` for quantiles — P50, P90, P99 — since 4.1. `Theta` for distinct counts with set operations — since 4.1. Approx Top-K for frequent items — since 4.1. `HLL` for cardinality — since 3.5.
New in 4.2: tuple sketches — distinct count plus a metric, like revenue, in one pass.
The best part: they merge. Store sketches as Delta columns. Then combine them for any time range in milliseconds — with no rescan of raw data. And they work with the open Apache DataSketches project, so they merge across engines.

### [48] Tuple sketches in action · **XIAO** · ~0:30
*(Code slide.)* A tuple sketch — distinct users and total revenue, in one pass. Then we merge daily sketches into a month — instantly, with no rescan.

### [49] Roadmap (Geospatial) · **XIAO** · ~0:06
*(One breath.)* And the fourth theme — geospatial.

### [50] Native geospatial types · **XIAO** · ~0:45
Location data is everywhere — delivery, IoT, maps, risk. In 4.2, Spark adds `GEOMETRY` and `GEOGRAPHY` as native SQL types. On by default. No extra package. `GEOMETRY` is flat. `GEOGRAPHY` is on the globe. There is full SRID support. You read and write in Parquet, and the type and SRID are kept. It works in SQL, DataFrames, PySpark, and the Thrift server. And it follows the open Parquet and Iceberg geospatial spec, so it works across engines.

### [51] Geospatial in action · **XIAO** · ~0:30
*(Code slide. Only claim the shipped functions on screen.)* Here are the functions that ship in 4.2 — `ST_GeomFromWKB`, `ST_GeogFromWKB`, `ST_AsBinary`, and the SRID helpers. Build a geometry, write it to Parquet, read it back — the type and SRID stay.
**▶ HANDOFF — XIAO → DB.** *(XIAO:)* That is my half — SQL and Python. Now DB will show how data moves, and where Spark is going. DB.

---

## 05 · PIPELINES & AUTO CDC — Slides 52–55 · DB

### [52] Section divider — Declarative Pipelines & Auto CDC · **DB** · ~0:15
Thank you, Xiao. Benefit four: move changing data safely. Let's start with a hard problem — keeping a table in sync with a stream of changes.

### [53] Applying a change feed is the hard part · **DB** · ~0:50
The common need: keep a copy of an operational table up to date. Rows are inserted, updated, deleted.
A CDC event has four parts: the key, the operation, the order, and the data.
This sounds easy, but it is not. Events arrive out of order, and across batches. Some events are partial. And retries must be safe.
The hand-written version — `foreachBatch` plus `MERGE` plus tombstones — is often hundreds of lines. Everyone writes it again, and everyone makes small mistakes.

### [54] Auto CDC: declare it, Spark reconciles it · **DB** · ~0:55
So 4.2 lets you declare it instead. You give the keys, the order, and the delete rule. Spark reconciles each batch against the target, in order, and merges it for you.
It is safe with out-of-order data. Inserts and updates both become upserts. Deletes come from the feed.
In 4.2 you get a Python API — `create_auto_cdc_flow()` — and Spark Connect support. It stores SCD Type 1. SQL syntax and SCD Type 2 are coming.
It runs inside Spark Declarative Pipelines. So checkpoints, retries, and idempotency are handled for you.

### [55] Auto CDC in action · **DB** · ~0:30
*(Code slide.)* Here is the whole thing. Declare the flow. Point it at the feed. Name the keys and the order column. These few lines replace the hundreds we just saw. Now — streaming.

---

## 06 · STRUCTURED STREAMING — Slides 56–60 · DB

### [56] Section divider — Structured Streaming · **DB** · ~0:12
Still benefit four — move changing data safely. Now streaming. It is about two things: changing a running query safely, and a state store you can trust.

### [57] Evolve running pipelines safely · **DB** · ~0:40
A long-time problem: streaming sources were identified by position. So you could not add, remove, or reorder them without breaking the checkpoint.
Now you name them, with `IDENTIFIED BY`. Identity is no longer the position. So you can change the sources in a running query, and keep the checkpoint. `name()` does the same for sinks. It works in SQL functions, Scala, and PySpark — classic and Connect.

### [58] IDENTIFIED BY in action · **DB** · ~0:20
*(Code slide.)* Named sources. Then a query adds a new one, restarts, and keeps its checkpoint. No full reprocess.

### [59] Richer joins, lower latency · **DB** · ~0:35
Streaming joins get richer. Inner and LeftSemi stream-stream joins now run in Update mode. LeftSemi joins use less state-store space. And the real-time mode trigger is now in PySpark. *(Remember real-time mode — we come back to it later.)*

### [60] A state store you can trust · **DB** · ~0:35
The state store is the heart of stateful streaming. In 4.2 it is stronger. It can repair a corrupted snapshot automatically. Checksums find corruption early. And slow snapshot uploads are handled automatically — now on by default. Fewer pages at night.

---

## 07 · DATA SOURCE V2 — Slides 61–75 · DB

### [61] Section divider — Data Source V2 · **DB** · ~0:15
Still benefit four — move changing data safely. Now Data Source V2 — how Spark connects to data. It improved a lot this release.

### [62] One integration API for every data source · **DB** · ~0:40
DSv2 is the standard API for data sources in Spark — Delta, Iceberg, and more. Users get the same SQL and behavior across sources. Connectors get DML, streaming, and Spark's optimizations.
It is growing fast. In 4.1 and 4.2: row-level DML, CDC, schema evolution, transactions, and better pushdown — all here.
There is a full talk at this Summit: "Spark DSV2: Growing Up Fast," by Szehon Ho and Anton Okolnychyi.

### [63] Roadmap (Reading changes) · **DB** · ~0:06
*(Point at item 1.)* Three themes. First — reading changes.

### [64] Ask the table: "what changed?" · **DB** · ~0:45
CDC is the row-level history of a table — which rows were added, updated, or deleted.
It is a typed log, in commit order. Each row is tagged: insert, update before, update after, or delete. Each row has the values, the operation, and the order.
CDC is important infrastructure. It powers incremental ETL, audit, and replication. And it refreshes vector indexes, materialized views, and streaming tables on commit.

### [65] Two problems with CDC today · **DB** · ~0:50
CDC today has two problems.
One: every engine built its own. Delta has `table_changes()`. Iceberg has `create_changelog_view`. Hudi has incremental reads. Same idea, three syntaxes.
Two: write-time CDC costs every writer. Change files are written on every update — about 1.2× the write time. You pay this even if no one reads them.
Some numbers: in 60 days, 206 million CDC queries. And 68% of them needed no stored change files at all.
So 4.2 makes read-time CDC first-class. Only the reader pays. And any engine can write the table.

### [66] One CHANGES API for any DSv2 source · **DB** · ~0:45
The answer is one API. One SQL `CHANGES` clause — batch by version or time, or streaming with `STREAM ... CHANGES`. The same shape in DataFrames — `read.changes()` and `readStream.changes()`. The same syntax for Delta, Iceberg, and Hudi.
Spark does the hard part — removing carry-overs, detecting updates, collapsing changes. Connectors only ship a small contract — one interface, three flags, and a row id and row version.

### [67] Reading changes in action · **DB** · ~0:25
*(Code slide.)* `SELECT ... FROM t CHANGES`. A version range in, typed change rows out. Same query for Delta or Iceberg.

### [68] Delta goes read-time — by default · **DB** · ~0:50
For Delta, the result is great: CDC on every table, with nothing to enable. No table property. No `ALTER TABLE`. No ticket to the table owner.
And nothing to pay at write time. No change files. Changes are computed only when a reader asks.
It uses two columns Delta already has — a stable row id, and a row commit version. In one commit, removed and added rows with the same row id become an update. A lone add is an insert. A lone remove is a delete.
It needs Row Tracking — on by default in Databricks, opt-in in Delta OSS. Write-time CDC still works for special cases.

### [69] Roadmap (Smarter writes) · **DB** · ~0:06
*(One breath.)* Second theme — smarter writes.

### [70] Smarter writes: INSERT & MERGE · **DB** · ~0:40
Writes are smarter and safer. `INSERT INTO ... WITH SCHEMA EVOLUTION` lets the target schema grow with the data. A source with fewer columns is accepted — missing fields are filled.
There is new `INSERT ... REPLACE` syntax, and `BY NAME` with `REPLACE WHERE`. `MERGE` gets fixes — schema evolution with `DELETE`, and nested fields kept. And `TABLESAMPLE SYSTEM`, pushed down to DSv2 and JDBC.

### [71] Schema evolution in action · **DB** · ~0:20
*(Code slide.)* A new column appears in the source. With `SCHEMA EVOLUTION`, the target takes it. No manual `ALTER`.

### [72] Transactions: atomic, isolated DML · **DB** · ~0:40
DML is read, transform, write back. Without isolation, a writer can overwrite changes it never saw.
So 4.2 starts a transaction for every DML, and tracks all reads and writes in it. Connectors check concurrent commits at commit time. A stale read fails cleanly — instead of corrupting data.
This is for single-statement DML today. Multi-statement and cross-catalog transactions are future work.

### [73] Observable, evolvable DML · **DB** · ~0:35
DML is observable now. `MERGE`, `UPDATE`, and `DELETE` report metrics — rows inserted, updated, deleted, copied. You see them where you already look — Delta `DESCRIBE HISTORY`, Iceberg snapshots. `MERGE` and `INSERT` support schema evolution, and the connector decides what is valid. And table refresh is consistent — so views over external tables stay correct.

### [74] Roadmap (Smarter pushdown) · **DB** · ~0:06
*(One breath.)* And the third theme — smarter pushdown.

### [75] Smarter pushdown: PartitionPredicate · **DB** · ~0:35
A small but real gap. Before, partition filters with UDFs or unusual expressions were lost between Catalyst and the DSv2 connectors. File sources could use them. Connectors could not.
The new `PartitionPredicate` API fixes this. Any Catalyst partition filter can be checked against partition values. It works for scans, `DELETE WHERE`, and runtime filters.
For example: `WHERE udf(month(ts)) = 'JAN'` now prunes partitions in Iceberg and Delta. Less data read. Faster queries.

---

## 08 · PERFORMANCE, UI & OPERATIONS — Slides 76–79 · DB

### [76] Section divider — Performance, UI & Operations · **DB** · ~0:12
Benefit five: operate and evolve predictably. First, the things that make Spark faster, easier to see, and easier to run.

### [77] A modern Spark Web UI · **DB** · ~0:30
The Web UI has a new look. Dark mode, and a faster interface. Interactive SQL plans — pan, zoom, search, and compare the first and final AQE plan side by side. And the environment page shows your non-default configs, with one-click export.

### [78] Faster & leaner · **DB** · ~0:35
On performance, four points. Faster scans — better vectorized Parquet reading. Smarter plans — pre-aggregation for many `COUNT(DISTINCT)`, and more codegen. Less memory — bounded merge and early memory release cut OOMs. And faster I/O — zero-copy transfers, and less JVM-to-Python overhead. All with no code change.

### [79] Runtime & operations · **DB** · ~0:35
On operations: we now run on Java 25. Kubernetes — in-place executor and PVC resizing, smaller images, NetworkPolicy isolation. The History Server scales — multiple log folders, and on-demand loading. And consistent results — query-level retry for indeterminate shuffles, with correct SQL metrics under AQE. That is 4.2. Now — what is next.

---

## 09 · LOOKING AHEAD — Slides 80–97 · DB

### [80] Section divider — Looking Ahead · **DB** · ~0:15
Still benefit five — operate and evolve predictably. This last part is ongoing work, already in progress for the next releases. We want to show you where Spark is going.

### [81] Roadmap (five topics) · **DB** · ~0:12
*(Gesture across the five.)* Five things ahead: Project Feather, a language-agnostic UDF protocol, nanosecond timestamps, real-time mode for stateful streaming, and a faster release cadence. We will touch each one.

### [82] Project Feather: fast local queries · **DB** · ~0:45
First — Project Feather. The goal: make small queries fast on a laptop.
Spark uses one API for big and small jobs. But for small jobs, the fixed costs are too high. A query on less than 100 MB can still take seconds — because planning, scheduling, serialization, and shuffle were all built for big clusters.
Feather works on three areas: planning and scheduling, the cache format, and shuffle.

### [83] Faster planning & scheduling · **DB** · ~0:35
On planning: a single-pass analyzer — more predictable, and near-linear in query size. When the input is one small file, run the whole query as one task — with no shuffle. AQE gets better stage hashing. And better join and broadcast choices. Less overhead before your answer.

### [84] Arrow-based DataFrame cache · **DB** · ~0:30
On caching: local analysis means load once, cache, and repeat. `df.cache` is a core feature.
Today the cache is row-based. So every reread costs extra CPU and memory. We are moving to an Arrow columnar cache, with compression. Smaller, and faster on the second and third query. This work is in progress.

### [85] Shuffle-free local execution · **DB** · ~0:35
On shuffle: disk shuffle is right for clusters — but too much for one JVM. There is a new multithreaded local mode. Threads pass data through channels, instead of shuffle files. With back-pressure and pipelining. It runs only when the query fits safely in one JVM.
Together — faster planning, Arrow cache, and no shuffle — Spark works well on a laptop, and still scales to the cluster.

### [86] Roadmap (UDF protocol) · **DB** · ~0:05
*(One breath.)* Second — and this closes a real gap.

### [87] Language-agnostic UDF protocol · **DB** · ~0:45
Remember the Connect slide — clients in Go, Rust, Swift, and more. UDFs are the one thing Connect cannot give most of them. SPIP SPARK-55278 fixes this.
Today, each language must rebuild the whole UDF stack. The planning rules are tied to Python. Serialized UDFs are tied to a runtime — a server upgrade can break them. And UDF execution is locked inside the cluster — so a heavy or GPU UDF makes you over-size the cluster.

### [88] Three pillars · **DB** · ~0:40
The design has three parts. One: plan UDFs by their shape — scalar, map, grouped map, table, aggregate — not by language. Two: one execution protocol — init, data, finish — over gRPC and Arrow, versioned, with back-pressure. Three: a worker spec — the client says how to start, connect, and clean up a worker: local process, container, or remote GPU. Each part can change without touching the others.

### [89] Status & what it unlocks · **DB** · ~0:30
Status: the SPIP vote passed. First code is landing in apache/spark, under `/udf/worker`. PySpark behavior does not change — one shared core, with pluggable transport. Socket stays the default. gRPC is opt-in.
It unlocks UDFs in any language, runtimes that upgrade on their own, and heavy or GPU UDFs on separate workers.

### [90] Roadmap (Nanosecond timestamps) · **DB** · ~0:05
*(One breath.)* Third — a precision fix for interop.

### [91] Nanosecond-precision timestamps · **DB** · ~0:40
SPIP SPARK-56822. Today, Spark timestamps stop at microseconds. So nanosecond Parquet either fails, or falls back to a plain long — and loses the timestamp meaning.
This adds parameterized types — `TIMESTAMP(n)`, with `n` from 0 to 9. 6 is micros, 9 is nanos. The value model is compact, and keeps today's date range.
It is fully backward compatible. The micro types stay the default. Nanosecond behavior only appears when you ask for it.

### [92] Declaring nanosecond timestamps · **DB** · ~0:20
*(Code slide.)* New Catalyst types, wired end to end — parser, codegen, Parquet, casts. This gives clean interop with PyArrow, Trino, ClickHouse, and DuckDB.

### [93] Roadmap (Real-time stateful streaming) · **DB** · ~0:05
*(One breath.)* Fourth — and this connects back to streaming.

### [94] Real-time mode for stateful streaming · **DB** · ~0:40
Earlier I mentioned real-time mode. SPARK-54699 extends it to stateful queries — with about 100 milliseconds latency. It builds on stateless real-time mode, which shipped in 4.1 — now with the same low latency for `transformWithState` and aggregations.
Two parts make it work. A streaming shuffle — it sends data straight to the next task, instead of waiting for the batch. And concurrent stage scheduling — many stages run at the same time. The last part of the roadmap is how we ship all of this.

### [95] Roadmap (Faster releases) · **DB** · ~0:05
*(One breath.)* The last one is about cadence.

### [96] Faster, predictable releases · **DB** · ~0:50
SPIP SPARK-54633 — a two-layer release model. Minor releases every quarter. Major releases every year.
Minors add features, performance, and APIs. But they freeze dependencies, behavior, and config defaults. So upgrades are safe.
Majors are once a year — for dependency upgrades and breaking changes.
Long-lived branches cut maintenance branches from 6-plus to 2 or 3. And the last minor of each major is an 18-month LTS.
The quarterly cadence starts with 4.3. So 4.2 is the bridge release. The dev list also voted to stop the monthly preview releases.

### [97] An example release calendar · **DB** · ~0:35
*(Walk the four cards.)* An example calendar. 4.2 this month — the bridge. 4.3 around September — first on the quarterly cadence. 4.4 around December — the last 4.x minor, and the 18-month LTS. And 5.0 in January — the yearly major. The dates are examples. The rhythm is the promise.

---

## CLOSE — Slide 98 · DB

### [98] Join the community today! · **DB + XIAO** · ~0:20
*(DB:)* All of this is built by the Apache Spark community. And you can join. Get the release at spark.apache.org, the source on GitHub, and join the mailing lists. We welcome your contributions and your bug reports. *(XIAO:)* Thank you, DAIS. Enjoy the rest of the Summit.

---

*Total ~38–40 minutes, spoken slowly and clearly. If you run long, shorten the code slides (17, 20, 23, 25, 29, 34, 36, 39–42, 44, 48, 58, 67, 71, 92) to one short line each.*
