# Speaker Notes — The Upcoming Apache Spark™ 4.2
### DAIS 2026 keynote · two speakers · 77 slides

**Speakers:** XIAO = Xiao Li (gatorsmile) · DB = DB Tsai (dbtsai)

**Split:** Xiao presents slides 1–39; DB presents 40–75. One handoff, at the end of the Spark SQL section.

**Through-line (say it often):** *Spark 4.2 moves more of the modern data and AI stack into the engine itself.* Five benefits — (1) Define truth once · (2) Reach Spark from everywhere · (3) Run AI-native analytics in SQL · (4) Move changing data safely · (5) Operate & evolve predictably.

**PySpark hierarchy:** No code change (Arrow) → Small change (Arrow UDFs) → New capability (Python Data Sources) → Dev experience. On-screen perf claims are qualitative; say numbers verbally with "in our benchmarks."

**Style:** Short, simple sentences; read slowly. *(parentheses)* are stage directions. Section dividers carry a **Today → Spark 4.2 → Takeaway** triad.

---

### [1] The Upcoming Apache Spark™ 4.2 · XIAO · ~0:35
Good morning. I am Xiao Li, I work on Apache Spark at Databricks. (DB:) And I am DB Tsai, also on the Spark team. (XIAO:) Today we show what is new in Apache Spark 4.2 — I take the first half, DB the second. The theme: native analytics for data and AI; Spark is bringing more of the stack into the engine. Let's start.

### [2] About us · XIAO · ~0:12
A quick note: Spark 4.2 is built by the whole Apache Spark community — hundreds of people. Everything we show is open source.

### [3] Apache Spark™ 4.2 — Overview · XIAO · ~0:35
Spark 4.2 in one sentence: more semantic, AI, Python, CDC, and operational power in the engine. The numbers — over 1,600 JIRAs from over 260 contributors. One goal ties it together: make analytics and AI more native to Spark.

### [4] What we'll cover today · XIAO · ~0:40
We organized the talk around five benefits. One: define truth once — metric views. Two: reach Spark from everywhere — Connect, PySpark, Arrow, Python Data Sources. Three: AI-native analytics in SQL — vectors, sketches, geospatial. Four: move changing data safely — Auto CDC, CHANGES, DSv2, streaming. Five: operate and evolve predictably — UI, Feather, faster releases. I cover the first; DB covers the rest.

### [5] The features at a glance · XIAO · ~0:25
This is the whole release on one slide — six groups, about 24 features. We won't cover all of them; just see how much is here. Keep it as a map. Let's start.

## Section 01 · Metrics — Metrics & Semantic Modeling

### [6] Section divider — Metrics & Semantic Modeling · XIAO · ~0:10
Benefit one: define truth once. Let's start with the problem. Takeaway: move your top dashboard metrics into a metric view.

### [7] The same metric should not change when the query changes · XIAO · ~0:30
The pain, in plain words. Revenue per user is a ratio — and ratios are not additive. You cannot average them across groups and get the right answer. So when every team writes the formula by hand, dashboards drift apart: same metric, different numbers. Let me show how it breaks.

### [8] Some metrics don't add up · XIAO · ~0:35
Here it is in one table. Two countries. Revenue and active users add up fine. But revenue per user is a ratio — average the two rows and you get 15, which is wrong. The right answer is total revenue over total users: 13.33. So where should this metric be defined, so it comes out right every time? That is a semantic-layer problem.

### [9] Why this is a semantic-layer problem · XIAO · ~0:45
Every company already has a semantic layer — but it is scattered. 'Revenue,' 'active users,' 'churn' get defined again and again across BI tools, dbt, and notebooks. The cost is real: dashboards disagree and no one owns the definition. And it is about correctness — ratios and COUNT(DISTINCT) break on roll-up. Client tools outside the engine can't enforce it. AI needs this too — an agent must answer the same way every time. A governed metric view gives BI and AI one source of truth.

### [10] Metric views: define metrics once · XIAO · ~0:40
So 4.2 adds metric views — SPIP SPARK-54119. Define a business metric once, then query it by any breakdown. Two parts: dimensions — the ways to slice — and measures, a new column type that aggregates without a fixed GROUP BY. This stops drift: ratios and distinct counts stay correct for every grouping, and measures compose. It lives in the engine and catalog, so the optimizer enforces correctness and permissions hold everywhere. One source of truth — for SQL, BI, and AI.

### [11] Metric views in action · XIAO · ~0:30
Three ideas, not syntax. A dimension is how people slice the metric. A measure is what the business cares about — here, revenue per user, a ratio. The query asks by any breakdown. The point is not the syntax — the metric is now an object Spark understands, computed correctly at every grain, for everyone. Now — how do you connect to Spark?

## Section 02 · Connect — Spark Connect

### [12] Section divider — Spark Connect · XIAO · ~0:12
Benefit two: reach Spark from everywhere. First, Spark Connect. Takeaway: call Spark from your service's own language.

### [13] Architecture: gRPC in, Arrow out · XIAO · ~0:35
The idea is simple. Your client builds a plan, opens a gRPC stream, and the server resolves, optimizes, runs it, and sends Arrow batches back. The client is thin — no full Spark runtime, no JVM next to your app — so client and cluster evolve on their own. Before Connect, using Spark from another runtime meant wrapping or embedding it. With Connect, Spark becomes a service API — your app does not need to be a Spark app.

### [14] Spark becomes a service API · XIAO · ~0:35
The point is the use case, not the language list. Before Connect, reaching Spark meant wrapping or embedding it. Now Spark is a service API: a Rust service submits work, a Python notebook connects remotely, a lightweight app uses Spark without packaging it. Python and Scala ship with Spark; Go, Swift, Rust, R, TypeScript, and .NET speak the same protocol at different maturity levels. The next frontier — portable UDFs — comes back in Looking Ahead.

### [15] Closing the gap with Spark Classic · XIAO · ~0:30
In 4.2 we keep closing the gap with the classic driver. Old RDD helpers come to Connect — zipWithIndex, toJSON, emptyDataFrame. read.json, csv, and xml can take a DataFrame. There is a new GetStatus API. Errors are clearer — like using a column from the wrong DataFrame — and client file names and line numbers are kept for debugging.

## Section 03 · Python — PySpark, Faster by Default

### [16] Section divider — PySpark, Faster by Default · XIAO · ~0:12
Still benefit two. Now Python — the language most of you use with Spark. The headline is in the title: faster by default. Takeaway: upgrade for free speedups, and use @arrow_udf on hot paths.

### [17] Arrow on by default · XIAO · ~0:28
The important part is the default. In 4.2, Arrow Python UDFs are the default — your old UDFs just run faster. Arrow IPC between JVM and Python is on, and Arrow input skips a conversion step. The upgrade story: this speed with no code change.

### [18] Python UDFs: same code, now faster · XIAO · ~0:20
(Code slide.) Same UDF you always write. No new decorator, no rewrite — but the engine moves the data as Arrow. Same code, faster.

### [19] Native Arrow UDFs: write on columns, skip the copies · XIAO · ~0:40
4.2 continues the Arrow-first direction. The decorators — @arrow_udf and @arrow_udtf — arrived in 4.1 and are the APIs for hot paths: PyArrow array in and out, data stays columnar, no pandas cost. In our benchmarks, about 10% faster and 40% less memory than a pandas UDF. You get scalar, aggregate, and table functions, plus iterator mode to set up once — like loading a model per batch. And the Arrow backend speeds up existing pandas UDFs about 2x, no code change.

### [20] Arrow UDFs in action · XIAO · ~0:20
(Code slide.) The @arrow_udf decorator. Arrays in, arrays out. No to_pandas, no round trip — columnar start to end.

### [21] Python Data Sources: connectors in pure Python · XIAO · ~0:40
You can now build readers and writers fully in Python — batch and streaming, no JVM, no Scala. Register once, then use spark.read.format like a built-in source. The API keeps growing — pushdown, Arrow writers, streaming, partitioning. And there is a real ecosystem: the community package pyspark-data-sources has many ready connectors — Hugging Face, Google Sheets, Kaggle, Salesforce. For the AI era, an LLM can scaffold a long-tail connector — and you still test and harden it.

### [22] A growing Python Data Source ecosystem · XIAO · ~0:20
(Gesture at the logos.) This is the ecosystem today — connectors the community already built, in pure Python. Before, you learned the JVM API. Now you just write Python.

### [23] Python Data Sources in action · XIAO · ~0:20
(Code slide.) A full custom source: a reader class, register it, read it like any format. That is the whole thing.

### [24] Profiling Python Data Sources · XIAO · ~0:28
New in 4.2: profile these Python data sources like UDFs — for time and memory. One config turns it on. Then you see where time and memory go in your read and write code. Your connector is no longer a black box.

### [25] Better interop & developer experience · XIAO · ~0:35
Small individually, big in daily work. PyCapsule: when both sides speak the Arrow C Data Interface, you exchange data zero-copy with Polars, DuckDB, and Arrow tools. You can profile Python data sources. builder.create() makes a new session without changing other configs. A strict mode catches ambiguous columns early. And pandas-on-Spark adds more axis=1 functions. Each is minor alone — together they save time every day.

### [26] PyCapsule interop in action · XIAO · ~0:25
(Code slide.) A Spark DataFrame goes straight to Polars or DuckDB through the Arrow C Data Interface. When both sides support it, they share the same Arrow memory — zero copy, no serialization.

### [27] Debug UDFs where they run · XIAO · ~0:35
Now debugging. Your UDF runs in a Python worker on an executor, far from your notebook — and when it hangs, before, you saw nothing. In 4.2 hangs are visible: the worker dumps its stack trace, and idle workers get dumped and killed. One profiler covers Python, pandas, and Arrow UDFs — CPU and memory by line — and works on Connect. And Python logging inside a UDF can be read back as a table.

### [28] UDF debugging in action · XIAO · ~0:20
(Code slide.) A stuck worker, with its stack trace shown automatically. And UDF logs, read back as a normal table.

## Section 04 · New SQL Analytics — Spark SQL

### [29] Section divider — Spark SQL · XIAO · ~0:12
Benefit three: AI-native analytics in SQL. This release added the most features here, in four themes. Takeaway: replace cross-join plus ROW_NUMBER top-K with NEAREST BY.

### [30] Query embeddings where your data lives · XIAO · ~0:32
Embeddings are everywhere — search, recommendations, dedup, RAG. Usually you run a separate vector system. In 4.2, Spark adds built-in vector functions, so you score similarity where your data already is. Distance and similarity, norm and normalize, vector avg and sum. These are the building blocks for the new NEAREST BY join.

### [31] Vector functions in action · XIAO · ~0:20
(Code slide.) Cosine similarity and L2 distance, called directly in SQL on a column of embeddings, at Spark scale.

### [32] NEAREST BY: top-K ranking join · XIAO · ~0:45
A new join clause — SPIP SPARK-56395. For each query row, it returns the top-K closest rows. Before: a cross join, a window, a rank, a filter — slow and opaque to the optimizer. Now: NEAREST 10 BY SIMILARITY, one clause the optimizer plans. It ranks by any expression — similarity, distance, geospatial, even BM25. INNER drops unmatched rows; LEFT OUTER keeps them. In 4.2, think exact first: EXACT is brute-force KNN, and that is what ships. ANN indexes are out of scope for now — APPROX is the door, so a future index helps without changing your query or results.

### [33] NEAREST BY in action · XIAO · ~0:20
(Code slide.) One query: for each row, the five nearest. That is it — no window, no cross join.

### [34] SQL gets more natural · XIAO · ~0:35
Before and now, four times — each removes a pattern we've written for years. QUALIFY: top-N without a subquery. FILTER: conditional aggregates now run over a window too — SUM(...) FILTER (WHERE ...) OVER (...), which was rejected before 4.2, that's SPARK-55702. PIVOT with aliases: readable columns. And cursors: row-by-row logic in pure SQL. Less boilerplate, more intent. Takeaway: reach for QUALIFY first, and FILTER over windows where you used a subquery.

### [35] SQL compatibility improvements · XIAO · ~0:40
A quick compatibility round. SET PATH resolves names across schemas and is saved in views, so results stay reproducible — good for PostgreSQL migration. It is off by default: enable spark.sql.path.enabled first. The type system is more complete: TIME works across more formats — but it is a preview, off by default in production via spark.sql.timeType.enabled, so don't live-demo it unflagged. TIMESTAMP WITH LOCAL TIME ZONE is in SQL. Plus max_by/min_by, time_bucket, and reverse on binary. And IGNORE/RESPECT NULLS is now honored by array_agg, collect_list, and collect_set — the clause isn't new, these aggregates just gained support. Takeaway: these close common gaps when porting SQL.

### [36] Data sketches: approximate, mergeable analytics · XIAO · ~0:45
Sketches are small probabilistic summaries — one pass, small memory, about 1 to 2% error, decision-grade when the budget fits. Much is already in Spark as SQL aggregates: KLL quantiles, Theta distinct plus set ops, and Approx Top-K — all since 4.1 — and HLL since 3.5. New in 4.2: tuple sketches — distinct count plus a metric, like revenue, in one pass. The best part is they merge. Store them as Delta columns, then combine any time range in milliseconds, no rescan. They interoperate with open Apache DataSketches, so they merge across engines.

### [37] Tuple sketches in action · XIAO · ~0:22
(Code slide.) A tuple sketch — distinct users and total revenue in one pass. Then merge daily sketches into a month, instantly, no rescan.

### [38] Native geospatial types · XIAO · ~0:35
Location data is everywhere — delivery, IoT, maps, risk. 4.2 adds GEOMETRY and GEOGRAPHY as native SQL types, on by default, no package. GEOMETRY is flat; GEOGRAPHY is on the globe; full SRID support. Read and write in Parquet with type and SRID kept. It works in SQL, DataFrames, and PySpark, and follows the open Parquet and Iceberg geospatial spec, so it crosses engines.

### [39] Geospatial in action · XIAO · ~0:35
(Code slide — claim only shipped functions.) The functions that ship in 4.2: ST_GeomFromWKB, ST_GeogFromWKB, ST_AsBinary, and SRID helpers. Build a geometry, write to Parquet, read it back — type and SRID stay.
▶ HANDOFF — XIAO → DB. (XIAO:) So far, 4.2 brings more logic into the engine — metrics, Python, SQL. But analytics isn't only querying; the data changes underneath you. DB will show how 4.2 makes change itself safer — CDC, streaming, DSv2, operations. DB.

## Section 05 · Pipelines & Auto CDC — Spark Declarative Pipelines & Auto CDC

### [40] Section divider — Declarative Pipelines & Auto CDC · DB · ~0:12
Thank you, Xiao. Benefit four: move changing data safely. Let's start with a hard problem — keeping a table in sync with a stream of changes. Takeaway: replace custom foreachBatch plus MERGE with Auto CDC.

### [41] Applying a change feed is the hard part · DB · ~0:35
The common need: keep a copy of an operational table up to date — rows inserted, updated, deleted. A CDC event has four parts: key, operation, sequence, and payload. It sounds easy, but isn't: events arrive out of order and across batches, some are partial, and retries must stay idempotent. The hand-written version — foreachBatch plus MERGE plus tombstones — is often hundreds of lines, and everyone makes small mistakes.

### [42] Auto CDC: declare it, Spark reconciles it · DB · ~0:40
Here is the contrast, in line count. By hand it is about a hundred lines — a foreachBatch loop, a MERGE, ordering, dedupe, late data, tombstone deletes, idempotent retries. With Auto CDC it is about eight: declare the keys, order, and delete rule, and Spark reconciles each batch in order. It is safe with out-of-order data; inserts and updates become upserts; deletes come from the feed. New in 4.2: a Python API, create_auto_cdc_flow(), plus Connect support; it stores SCD Type 1, with SQL and SCD Type 2 coming. It runs in Declarative Pipelines, so checkpoints and retries are handled.

### [43] Auto CDC in action · DB · ~0:25
(Code slide.) The whole thing, inside a Declarative Pipelines file — create_auto_cdc_flow registers a flow, not a standalone call. Declare the flow, point it at the feed, name the keys and order column. These few lines replace the hundreds we saw. Now — streaming.

## Section 06 · Streaming — Structured Streaming

### [44] Section divider — Structured Streaming · DB · ~0:12
Still benefit four. Now streaming — two things: changing a running query safely, and a state store you can trust. Takeaway: name your streaming sources so you can evolve them safely.

### [45] Evolve running pipelines safely · DB · ~0:32
A long-time problem: streaming sources were identified by position, so you couldn't add, remove, or reorder them without breaking the checkpoint. Now you name them — IDENTIFIED BY in SQL, DataStreamReader.name() in PySpark, Classic and Connect. Identity is the name, not the position, so you can change sources in a running query and keep the checkpoint. (Sink naming exists internally in Scala but isn't a public PySpark API — don't demo df.writeStream.name().)

### [46] IDENTIFIED BY in action · DB · ~0:18
(Code slide.) Named sources. A query adds a new one, restarts, and keeps its checkpoint — no full reprocess.

### [47] Richer joins, lower latency · DB · ~0:25
Streaming joins get richer. Inner and LeftSemi stream-stream joins now run in Update mode. LeftSemi uses less state-store space. And the real-time mode trigger is now in PySpark — remember it, we come back later.

### [48] A state store you can trust · DB · ~0:25
The state store is the heart of stateful streaming, and in 4.2 it is stronger: it repairs a corrupted snapshot automatically, checksums catch corruption early, and slow snapshot uploads are handled — on by default. Fewer pages at night.

## Section 07 · Data Source V2 — Data Source V2

### [49] Section divider — Data Source V2 · DB · ~0:12
Still benefit four. Now Data Source V2 — how Spark connects to data. It improved a lot this release. Takeaway: read changes with one CHANGES query, not per-engine functions.

### [50] One integration API for every data source · DB · ~0:33
DSv2 is the standard API for data sources — Delta, Iceberg, and more. Users get the same SQL and behavior across sources; connectors get DML, streaming, and Spark's optimizations. It is growing fast — in 4.1 and 4.2: row-level DML, CDC, schema evolution, transactions, better pushdown. There is a full talk at this Summit: 'Spark DSV2: Growing Up Fast,' by Szehon Ho and Anton Okolnychyi.

### [51] Ask the table: "what changed?" · DB · ~0:32
CDC is the row-level history of a table — which rows were added, updated, or deleted. It is a typed log in commit order: each row tagged insert, update before/after, or delete, with values, operation, and order. It is load-bearing infrastructure — incremental ETL, audit, replication — and it refreshes vector indexes, materialized views, and streaming tables on commit.

### [52] Two problems with CDC today · DB · ~0:40
CDC today has two problems. One: every engine built its own — Delta table_changes(), Iceberg create_changelog_view, Hudi incremental reads — same idea, three syntaxes. Two: write-time CDC taxes every writer — change files on every update, about 1.2x the write time, paid even if no one reads them. By the numbers: 206 million CDC queries in 60 days, and 68% needed no stored change files at all. So 4.2 makes read-time CDC first-class: only the reader pays, and any engine can write the table.

### [53] One CHANGES API for changelog-capable DSv2 sources · DB · ~0:35
The answer is one API. One SQL CHANGES clause — by version or time, or streaming with STREAM ... CHANGES. The same shape in DataFrames — read.changes() and readStream.changes(). The same syntax for any DSv2 connector that implements changelog support — Delta, Iceberg, Hudi today. Not every source has it; the connector must ship the interface. Spark does the hard part — carry-over removal, update detection, collapsing changes — connectors ship a tiny contract.

### [54] Reading changes in action · DB · ~0:20
(Code slide.) SELECT ... FROM t CHANGES — a version range in, typed change rows out. Same query for Delta or Iceberg.

### [55] Delta goes read-time — by default · DB · ~0:40
For a Delta table with row tracking, the result is great: read-time CDC with no Change Data Feed property — no CDF setting, no ALTER TABLE, no ticket to the owner. Nothing is paid at write time; changes are computed only when a reader asks. It uses two columns Delta already has — a stable row id and a row commit version: matched ids become an update, a lone add an insert, a lone remove a delete. It needs Row Tracking — default in Databricks, opt-in in Delta OSS. Write-time CDC still works for special cases.

### [56] Smarter writes: INSERT & MERGE · DB · ~0:30
Writes are smarter and safer. INSERT INTO ... WITH SCHEMA EVOLUTION lets the target grow with the data, and a source with fewer columns is accepted — missing fields filled. New INSERT ... REPLACE syntax, and BY NAME with REPLACE WHERE. MERGE gets fixes — schema evolution with DELETE, nested fields kept. And TABLESAMPLE SYSTEM, pushed down to DSv2 and JDBC.

### [57] Schema evolution in action · DB · ~0:18
(Code slide.) A new column appears in the source. With SCHEMA EVOLUTION the target takes it — no manual ALTER.

### [58] Transactions: atomic, isolated DML · DB · ~0:30
DML is read, transform, write back. Without isolation, a writer can overwrite changes it never saw. So 4.2 starts a transaction for every DML and tracks all reads and writes in it; connectors check concurrent commits at commit time, so a stale read fails cleanly instead of corrupting data. This is single-statement DML today; multi-statement and cross-catalog are future work.

### [59] Observable, evolvable DML · DB · ~0:28
DML is observable now. MERGE, UPDATE, and DELETE report metrics — rows inserted, updated, deleted, copied — where you already look: Delta DESCRIBE HISTORY, Iceberg snapshots. MERGE and INSERT support schema evolution, and the connector decides what is valid. And table refresh is consistent, so views over external tables stay correct.

### [60] Smarter pushdown: PartitionPredicate · DB · ~0:30
A real gap connectors hit daily. Before, partition filters with UDFs or unusual expressions were lost between Catalyst and DSv2 connectors — file sources could prune on them, connectors couldn't. The new PartitionPredicate API fixes it: any Catalyst partition filter is evaluated against partition values, for scans, DELETE WHERE, and runtime filters. So WHERE udf(month(ts)) = 'JAN' now prunes partitions in Iceberg and Delta — less data read, faster queries.

## Section 08 · Others — Performance, UI & Operations

### [61] Section divider — Performance, UI & Operations · DB · ~0:12
Benefit five: operate and evolve predictably. First, what makes Spark faster, easier to see, and easier to run. Takeaway: upgrade and pick up speed and stability with no code change.

### [62] A modern Spark Web UI · DB · ~0:25
The Web UI has a new look — dark mode and a faster interface. Interactive SQL plans you can pan, zoom, search, and compare initial vs final AQE plan side by side. And the environment page shows your non-default configs, with one-click export.

### [63] Faster & leaner · DB · ~0:28
Four performance points. Faster scans — better vectorized Parquet. Smarter plans — pre-aggregation for many COUNT(DISTINCT) and more codegen. Leaner memory — bounded merge and early release cut OOMs. And faster I/O — zero-copy transfers, less JVM-to-Python overhead. All with no code change.

### [64] Runtime & operations · DB · ~0:28
On operations: we run on Java 25. Kubernetes — in-place executor and PVC resizing, smaller images, NetworkPolicy isolation. The History Server scales — multiple log folders, on-demand loading. And consistent results — query-level retry for indeterminate shuffles, with correct SQL metrics under AQE. That is 4.2. Now — what is next.

## Section 09 · Ongoing Work — Looking Ahead

### [65] Section divider — Looking Ahead · DB · ~0:12
Still benefit five. This last part is ongoing work, already in progress for the next releases — where Spark is going. Takeaway: follow the SPIPs, try the previews, and plan around the quarterly cadence.

### [66] Roadmap (five topics) · DB · ~0:12
(Gesture across the five.) Five things ahead: Project Feather, a language-agnostic UDF protocol, nanosecond timestamps, real-time mode for stateful streaming, and a faster release cadence. We'll touch each.

### [67] Project Feather: fast local queries · DB · ~0:32
First, Project Feather — make small queries fast on a laptop. Spark uses one API for big and small jobs, but for small jobs the fixed costs are too high: a query over less than 100 MB can still take seconds, because planning, scheduling, serialization, and shuffle were built for big clusters. Feather works on three areas: planning and scheduling, the cache format, and shuffle.

### [68] Project Feather: a three-part plan · DB · ~0:32
Feather has three parts. One — faster planning and scheduling: a single-pass analyzer, and running a one-file query as a single task with no shuffle. Two — an Arrow columnar cache to replace the row-based df.cache, for faster re-reads. Three — shuffle-free local execution: threads pass data through channels instead of shuffle files. Together, Spark works on a laptop and still scales to the cluster.

### [69] Language-agnostic UDF protocol · DB · ~0:35
Remember the Connect clients — Go, Rust, Swift, and more. UDFs are the one thing Connect can't give most of them; SPIP SPARK-55278 fixes that. Today each language rebuilds the whole UDF stack: planning rules are tied to Python, serialized UDFs are tied to a runtime — a server upgrade can break them — and execution is locked in the cluster, so a heavy or GPU UDF forces an over-sized cluster.

### [70] Three pillars · DB · ~0:32
The design has three parts. One: plan UDFs by shape — scalar, map, grouped map, table, aggregate — not by language. Two: one execution protocol — init, data, finish — over gRPC and Arrow, versioned, with back-pressure. Three: a worker spec — the client says how to start, connect, and clean up a worker: local, container, or remote GPU. Each part changes without touching the others.

### [71] Status & what it unlocks · DB · ~0:25
Status: the SPIP vote passed, and first code is landing in apache/spark under /udf/worker. PySpark behavior doesn't change — one shared core with pluggable transport; socket stays the default, gRPC is opt-in. It unlocks UDFs in any language, runtimes that upgrade on their own, and heavy or GPU UDFs on separate workers.

### [72] Nanosecond-precision timestamps · DB · ~0:32
SPIP SPARK-56822. Today Spark timestamps stop at microseconds, so nanosecond Parquet either fails or falls back to a plain long, losing the timestamp meaning. This adds parameterized types — TIMESTAMP(n), n from 0 to 9; 6 is micros, 9 is nanos — with a compact value model that keeps today's date range. It is fully backward compatible: micro types stay the default, nanosecond behavior appears only when you ask.

### [73] Real-time mode for stateful streaming · DB · ~0:35
Earlier I mentioned real-time mode. SPARK-54699 extends it to stateful queries — about 100 milliseconds latency. It builds on stateless real-time mode from 4.1, now with the same low latency for transformWithState and aggregations. Two parts make it work: a streaming shuffle that sends data straight to the next task instead of waiting for the batch, and concurrent stage scheduling, so many stages run at once. Last, how we ship all this.

### [74] Faster, predictable releases · DB · ~0:40
And how we ship it. SPARK-54633 — a two-layer model: quarterly minors and an annual major. Minors freeze dependencies and defaults so upgrades stay safe; majors carry the breaking changes. The final minor of each major is an 18-month LTS. 4.2 is the bridge, and the quarterly train starts at 4.3. As a transition exception, the 4.x LTS is 4.5.0 — the last planned 4.x, around March 2027 — not 4.3. Then Spark 5.0 follows around June 2027. Takeaway: plan upgrades around the quarterly cadence, and target the 4.5 LTS.

### [75] Join the community today! · DB + XIAO · ~0:20
(DB:) All of this is built by the Apache Spark community — and you can join. Get the release at spark.apache.org, the source on GitHub, and join the mailing lists. We welcome your contributions and bug reports. (XIAO:) Thank you, DAIS. Enjoy the rest of the Summit.