# Speaker Notes — *The Upcoming Apache Spark™ 4.2*
### DAIS 2026 keynote · ~40 minutes · two speakers

**Speakers:** **XIAO** = Xiao Li (gatorsmile) · **DB** = DB Tsai (dbtsai)

**Split:** **Xiao presents slides 1–51** (open, Metrics, Spark Connect, PySpark, Spark SQL). **DB presents slides 52–101** (Pipelines & Auto CDC, Streaming, Data Source V2, Performance/Ops, Looking Ahead, close). There is exactly **one handoff**, at the end of slide 51.

**How to read this:** Each slide has a `[#] Title · SPEAKER · ~time`. The text below it is the word-by-word script — read it as written, or use it as a spine. The single **▶ HANDOFF** is marked at slide 51→52. Lines in *(parentheses italics)* are stage directions, not spoken.

**Pacing:** This runs ~40 minutes at a brisk keynote pace. The deck is dense, so the "in action" code slides and the roadmap divider slides are meant to go *fast* — one or two breaths each. If you're running long, compress the code walk-throughs and dividers first. Rough section budget:

| Section | Slides | Speaker | Budget |
|---|---|---|---|
| Open (title, overview, agenda, map) | 1–5 | XIAO | 4:00 |
| 01 · Metrics & Semantic Modeling | 6–9 | XIAO | 3:30 |
| 02 · Spark Connect | 10–13 | XIAO | 2:15 |
| 03 · PySpark | 14–30 | XIAO | 6:00 |
| 04 · Spark SQL | 31–51 | XIAO | 6:30 |
| 05 · Pipelines & Auto CDC | 52–55 | DB | 2:30 |
| 06 · Structured Streaming | 56–60 | DB | 2:15 |
| 07 · Data Source V2 | 61–75 | DB | 5:00 |
| 08 · Performance, UI & Ops | 76–79 | DB | 2:00 |
| 09 · Looking Ahead | 80–97 | DB | 5:00 |
| Recap & close | 98–99 | DB | 1:00 |
| **Total** | | | **~40:00** |

---

## OPEN — Slides 1–5 · XIAO · ~4:00

### [1] The Upcoming Apache Spark™ 4.2 · **XIAO** · ~0:40
Good morning, everyone — thank you for being here. I'm Xiao Li, I work on Apache Spark at Databricks.
*(DB:)* And I'm DB Tsai, also on the Spark team at Databricks.
*(XIAO:)* Between the two of us we've spent a lot of years living inside the Spark codebase — and today we get to do one of our favorite things: show you what's coming in **Apache Spark 4.2**. I'll take the first half; DB will take the second. The theme this year is right there on the slide — **native analytics for data and AI** — and you'll see that thread run through everything. Let's dig in.

### [2] About us · **XIAO** · ~0:15
*(Quick — don't dwell.)* A word on where this comes from: 4.2 is the work of the whole Apache Spark community — hundreds of contributors. We're here as messengers for that community. Everything we show today is in the open-source release.

### [3] Apache Spark™ 4.2 — Overview · **XIAO** · ~1:20
4.2 is the third release in the 4.x line — over **1,600 resolved JIRAs from more than 260 contributors**. Here's the shape of it.
First: **metric views** bring a governed semantic layer right into Spark SQL — define a business metric once, query it by any dimension.
Second: a wave of **new analytics built into SQL** — vector similarity search, data sketches, and native geospatial types.
Third: **Change Data Capture becomes first-class** — across SQL, DataFrames, Declarative Pipelines, and Data Source V2.
Fourth: **PySpark is faster by default** — Arrow-optimized UDFs and Arrow IPC out of the box, plus Python Data Sources.
And finally: Structured Streaming gets a **real-time mode**, there's a modern Web UI, and we now run on **Java 25**. Let's look at how it all fits together.

### [4] What we'll cover today · **XIAO** · ~0:50
Here's the plan. We've grouped 4.2 into four arcs. **Metrics and Spark SQL** — the semantic layer and the new analytics. **PySpark and Spark Connect** — making Python first-class and fast. **Pipelines, streaming, and Data Source V2** — the data-movement and CDC story. And we'll close with **Looking Ahead** — there's a lot of exciting work already in flight for the releases after this one. I'll take the first four sections; DB will pick up from pipelines onward.

### [5] The features at a glance · **XIAO** · ~0:45
*(Gesture across the grid — don't read every tile.)* This is the whole release on one slide — six buckets, two dozen headline features. We are not going to cover all of these. What I want you to take away is the *breadth*: foundation pieces like metric views and Spark Connect compatibility; new SQL features; the Python story; streaming and pipelines; Data Source V2; and platform. Keep this slide in the back of your mind as a map. Let's start at the top — the foundation.

---

## 01 · METRICS & SEMANTIC MODELING — Slides 6–9 · XIAO · ~3:30

### [6] Section divider — Metrics & Semantic Modeling · **XIAO** · ~0:10
Let's start with something we think is one of the most important additions in this whole release: a semantic layer, native to Spark.

### [7] Why a semantic layer is critical · **XIAO** · ~1:10
Here's the thing — **every organization already has a semantic layer. It's just scattered.** "Revenue," "active users," "churn" — those get redefined in your BI tool, in dbt, in notebooks, in copy-pasted SQL. And the cost is real: dashboards disagree, teams burn whole afternoons reconciling numbers, and nobody actually owns the official definition.
And this is not a style problem — it's a **correctness** problem. Ratios and `COUNT(DISTINCT)` silently break when you re-slice or roll them up. The number looks fine; it's just wrong.
Client-side semantic layers help, but they sit *outside* the engine — so there's no optimizer enforcing correctness, permissions get inconsistent, and your logic is locked into one vendor's tool.
And here's the new urgency: **AI needs a contract.** When an agent asks "revenue per customer in the EU last quarter," it has to resolve that *deterministically*. A governed, in-catalog metric view is the grounding that both your BI tools and your LLMs can trust.

### [8] Metric views: define metrics once · **XIAO** · ~1:10
So 4.2 introduces **metric views** — SPIP SPARK-54119. A first-class, governed metric view: you define a business metric **once**, and query it by **any** breakdown.
There are two ingredients. **Dimensions** — the ways you slice. And **measures** — a new kind of column that aggregates *without* a pre-set `GROUP BY`.
This is what kills metric drift: ratios and distinct counts are computed correctly **for each grouping** — never wrongly re-aggregated — and measures can compose from other measures.
And critically, it **lives in the engine and the catalog**. The optimizer enforces correctness; permissions stay consistent — unlike a client-side layer bolted on top.
The result: **one governed source of truth** — for SQL, for BI, and for AI agents. Let me show you what that looks like.

### [9] Metric views in action · **XIAO** · ~0:40
*(Walk the example briefly.)* Here you define the view once — the dimensions, the measures. And then look at the queries: same metric, sliced by region, by month, rolled up to a total — and the numbers are *consistent at every grain*, because the engine knows how to aggregate each measure. You write the definition once; everyone downstream just asks questions. Now — how do you actually connect to Spark to ask those questions?

---

## 02 · SPARK CONNECT — Slides 10–13 · XIAO · ~2:15

### [10] Section divider — Spark Connect · **XIAO** · ~0:15
That brings us to Spark Connect — increasingly, how applications talk to Spark.

### [11] Architecture: gRPC in, Arrow out · **XIAO** · ~0:50
The model is simple and powerful. Your client builds an **unresolved logical plan** and opens a **gRPC stream** to the server. The server **resolves and optimizes** that plan, **schedules it across the executors**, and **streams Arrow batches back** to you. The client is thin — it doesn't need a full Spark runtime, it doesn't need a JVM next to your app. That decoupling is the whole point: the client and the cluster evolve independently, and the protocol is the contract between them.

### [12] Clients in any language · **XIAO** · ~0:40
*(Gesture at the language logos.)* And because the client only has to speak the Connect protocol over gRPC, your app can reach Spark **from whatever language your team already works in** — Python, Scala, Java, but also Go, Rust, Swift, TypeScript, .NET, and more. You don't rewrite your stack to use Spark; you talk to it from where you already live. *(Note: UDFs-in-any-language is the one gap — and it's one of DB's Looking-Ahead topics later.)*

### [13] Closing the gap with Spark Classic · **XIAO** · ~0:40
In 4.2 we keep closing the gap between Connect and the classic driver. RDD-era conveniences arrive — `zipWithIndex`, `DataFrame.toJSON`, `emptyDataFrame`. `spark.read.json/csv/xml` now accept a DataFrame as input. There's a new `GetStatus` API for monitoring execution from any client. Errors are clearer — including a dedicated one for the classic mistake of referencing a column from the wrong DataFrame. And client file names and line numbers are captured, so server-side debugging is far less painful. The goal is simple: Connect should feel like no compromise.

---

## 03 · PYSPARK — Slides 14–30 · XIAO · ~6:00

### [14] Section divider — PySpark, Faster by Default · **XIAO** · ~0:12
Now to the language most of you actually write Spark in: Python. And the headline is right in the title — **faster by default**.

### [15] Roadmap (Arrow by default) · **XIAO** · ~0:08
*(One breath. Point at item 1.)* Four things in the PySpark story. First — Arrow, on by default.

### [16] Arrow on by default · **XIAO** · ~0:45
This is the upgrade-and-it's-just-faster slide. In 4.2, **Arrow-optimized Python UDFs are the default** — your existing UDFs simply get faster. Arrow-based IPC between the JVM and the Python workers is **on out of the box**. And Arrow-backed input now skips the `ColumnarToRow` conversion on the way into your UDFs. The punchline: **upgrade to 4.2 and pick up these speedups with zero code changes.**

### [17] Python UDFs: same code, now faster · **XIAO** · ~0:30
*(Code slide — point, don't read.)* Same UDF you've always written. No new decorator, no rewrite. The only difference is the engine underneath is now moving your data as Arrow. Same code — faster.

### [18] Roadmap (Native Arrow UDFs) · **XIAO** · ~0:06
*(One breath.)* But if you're willing to write a *little* differently, you can go further — native Arrow UDFs.

### [19] Native Arrow UDFs: write on columns, skip the copies · **XIAO** · ~0:55
Here are two new decorators — `@arrow_udf` and `@arrow_udtf`. PyArrow array in, PyArrow array out — your data **never leaves its columnar layout**. There's no pandas conversion tax, so you get roughly **10% faster and 40% less memory** than the equivalent pandas UDF. You get scalar, aggregate — `groupBy` and window — and table functions, and the iterator modes let you amortize one-time setup, like loading a model once per batch. There's `mapInArrow` and `applyInArrow` for whole-batch and per-group work at the DataFrame level. And if you'd rather not change anything — the Arrow-backed pandas backend makes your **existing** pandas UDFs about **2× faster with zero code changes**.

### [20] Arrow UDFs in action · **XIAO** · ~0:30
*(Code slide.)* Here's the new decorator in practice — arrays in, arrays out. Notice there's no `to_pandas`, no round-trip. The data stays columnar from end to end.

### [21] Roadmap (Interop & debugging) · **XIAO** · ~0:06
*(One breath.)* Third — interoperability and the developer experience.

### [22] Better interop & developer experience · **XIAO** · ~0:45
A cluster of quality-of-life wins. **PyCapsule** support means **zero-copy interchange** with Polars, DuckDB, and the rest of the Arrow ecosystem. You can profile Python data sources with the built-in profilers. `SparkSession.builder.create()` spins up a fresh session without mutating existing configs. There's an optional strict column-resolution mode that catches ambiguous references early. And pandas-on-Spark gets broader `axis=1` support. Small things — but they're the things you hit every day.

### [23] PyCapsule interop in action · **XIAO** · ~0:30
*(Code slide.)* Watch this — a Spark DataFrame handed straight to Polars or DuckDB with no serialization in between. That's the PyCapsule protocol: the two libraries share the same Arrow buffers. Zero copy.

### [24] Debug UDFs where they run · **XIAO** · ~0:45
Now, debugging. Your UDF runs in a Python worker on an executor — far from your notebook — and when it hangs, historically you got... nothing. In 4.2, **hangs become observable**: periodic worker stack-trace dumps, plus dump-and-kill for idle workers whose socket has gone silent. There's unified profiling — CPU calls, time, and line-by-line memory — for Python, pandas, and Arrow UDFs, and it works on Spark Connect too. And there's structured UDF logging: use Python's `logging` inside the UDF, then **query the logs back as a table.**

### [25] UDF debugging in action · **XIAO** · ~0:25
*(Code slide.)* Here's a stuck worker — and there's the stack trace, surfaced automatically. And here are UDF logs, queried back as a regular table. The black box is open.

### [26] Roadmap (Python Data Sources) · **XIAO** · ~0:06
*(One breath.)* And fourth — my favorite — Python Data Sources.

### [27] Python Data Sources: connectors in pure Python · **XIAO** · ~0:55
You can now build batch **and** streaming readers and writers **entirely in Python** — `DataSource`, `DataSourceReader`, the streaming variants — **no JVM, no Scala.** You register once with `spark.dataSource.register(...)`, and then read and write through `spark.read.format(...)` — first-class, right alongside the built-in sources. The API keeps growing: filter and column pushdown, Arrow-based writers, streaming, partitioning. And there's a **real ecosystem** — the community `pyspark-data-sources` package already ships ready-made connectors for Hugging Face, Google Sheets, Kaggle, Salesforce, and more. And here's the kicker for the AI era: hand an LLM an API spec, and it'll **scaffold a working data source for you** — the long tail of connectors, on demand.

### [28] A growing Python Data Source ecosystem · **XIAO** · ~0:30
*(Gesture at the logo wall.)* This is what that ecosystem looks like today — connectors the community has already built, in pure Python. The barrier to adding a new source to Spark used to be "learn the JVM connector API." Now it's "write some Python."

### [29] Python Data Sources in action · **XIAO** · ~0:25
*(Code slide.)* Here's a complete custom source — a reader class, register it, and read from it like any built-in format. That's the whole thing.

### [30] Profiling Python Data Sources · **XIAO** · ~0:35
And new in 4.2 — SPARK-55161 — you can now **profile** those Python data sources, just like UDFs, for both performance and memory. Flip on `spark.sql.pyspark.dataSource.profiler`, and you'll see exactly where time and allocations go inside your read and write code, surfaced through `spark.profile.show()`. Your custom connector goes from a black box to something you can measure and tune.

---

## 04 · SPARK SQL — Slides 31–51 · XIAO · ~6:30

### [31] Section divider — Spark SQL · **XIAO** · ~0:15
Now to Spark SQL — where the largest number of new features landed this release. We've split it into four themes.

### [32] Roadmap (Vector search) · **XIAO** · ~0:08
*(Point at item 1.)* We start with vector search.

### [33] Query embeddings where your data lives · **XIAO** · ~0:45
Embeddings are everywhere now — semantic search, recommendations, dedup, RAG. The usual story is "stand up a separate vector system." In 4.2, Spark adds **built-in vector functions** — so you can score similarity **right where your data already lives**, no extra system to operate. Distance and similarity functions, vector norm and normalize, and vector `avg` and `sum` aggregations. And these are the scoring building blocks for something bigger — the new `NEAREST BY` join.

### [34] Vector functions in action · **XIAO** · ~0:25
*(Code slide.)* Cosine similarity, L2 distance — called directly in SQL, over a column of embeddings, at Spark scale.

### [35] NEAREST BY: top-K ranking join · **XIAO** · ~0:55
This is a new SQL join clause — SPIP SPARK-56395: `{APPROX or EXACT} NEAREST k BY {DISTANCE or SIMILARITY}`. For each query row, it returns the **top-K closest base rows**. Today, you'd write that as a cross-join plus a `ROW_NUMBER()` window — slow, and the optimizer can't reason about it. `NEAREST BY` replaces that whole idiom with **one clause the optimizer understands.** It ranks by **any scalar expression** — vector similarity, L2 distance, geospatial, even BM25. `INNER` drops unmatched query rows; `LEFT OUTER` keeps them. And here's the design we're proud of: `EXACT` is brute-force KNN today, but `APPROX` lets a future ANN index kick in **without changing your query** — and the presence of an index never silently changes your results.

### [36] NEAREST BY in action · **XIAO** · ~0:25
*(Code slide.)* One query — "for each of these, give me the five nearest" — and that's it. No window, no cross join. Readable, and optimizable.

### [37] Roadmap (SQL language) · **XIAO** · ~0:06
*(One breath.)* Next — the SQL language itself gets cleaner and more expressive.

### [38] Write window logic naturally · **XIAO** · ~0:40
A set of long-requested ANSI SQL ergonomics. **`QUALIFY`** — filter on a window-function result without wrapping it in a subquery. **`FILTER`** predicates now work inside window aggregates. **`PIVOT`** supports aliases, and the pipe syntax accepts a single bar. And **SQL scripting** gains **cursor** support for row-by-row processing. Let me show a few.

### [39] QUALIFY in action · **XIAO** · ~0:20
*(Code slide — quick.)* Top-N per group — no subquery. Just `QUALIFY ROW_NUMBER() ... <= 3`.

### [40] FILTER in window aggregates · **XIAO** · ~0:20
*(Code slide — quick.)* A conditional running total — the `FILTER` lives right on the window aggregate.

### [41] PIVOT aliases & the | pipe · **XIAO** · ~0:20
*(Code slide — quick.)* Aliased pivot columns, and the pipe syntax for chaining transformations.

### [42] SQL scripting: cursors · **XIAO** · ~0:20
*(Code slide — quick.)* And a cursor — row-by-row procedural logic, in pure SQL, for the migrations that need it.

### [43] SQL search path: SET PATH · **XIAO** · ~0:35
`SET PATH` lets you resolve tables, functions, and variables across namespaces **without fully qualifying** every name. `SET PATH` controls lookup order; `CURRENT_PATH()` shows what's active. And crucially, the path is **persisted in views and SQL UDFs** and shown in `DESCRIBE` — so results stay reproducible. If you're migrating off PostgreSQL or another database with a schema search path, this one's for you.

### [44] SET PATH in action · **XIAO** · ~0:20
*(Code slide — quick.)* Set the path once, then reference objects by short name — and `DESCRIBE` shows you exactly what path the view captured.

### [45] A more complete type system · **XIAO** · ~0:35
A round of type-system completeness. The `TIME` type now works across JSON, CSV, XML, ORC, and Avro, with implicit casts from string. `TIMESTAMP WITH LOCAL TIME ZONE` is supported in SQL syntax. `IGNORE NULLS` / `RESPECT NULLS` for `array_agg`, `collect_list`, and `collect_set`. Plus top-K `max_by`/`min_by`, and `time_bucket` for time-series rollups.

### [46] Roadmap (Data sketches) · **XIAO** · ~0:06
*(One breath.)* Third theme — data sketches.

### [47] Data sketches: approximate, mergeable analytics · **XIAO** · ~0:55
Sketches are bounded-memory probabilistic summaries — **single pass, sublinear memory, one-to-two-percent error.** I like to say: *approximate answers, exact decisions.* And a lot of this is **already in Spark** — a toolkit you call as SQL aggregates. **KLL** for quantiles — P50, P90, P99 without a global sort — since 4.1. **Theta** for distinct counts with set union, intersection, difference — since 4.1. **Approx Top-K** for frequent items — since 4.1. **HLL** for cardinality — since 3.5. **New in 4.2: Tuple sketches** — distinct count **plus a metric**, like revenue, in one pass. The magic is they're **composable and mergeable**: store sketches as Delta columns, then merge precomputed sketches for any time range in **milliseconds**, with no re-scan of the raw data. And they're interoperable with the open Apache DataSketches ecosystem — so sketches built in Spark merge with other engines.

### [48] Tuple sketches in action · **XIAO** · ~0:30
*(Code slide.)* Here's a tuple sketch — distinct users **and** their total revenue, in a single pass. And then we merge daily sketches into a monthly answer — instantly, without touching the raw events.

### [49] Roadmap (Geospatial) · **XIAO** · ~0:06
*(One breath.)* And the fourth theme — geospatial.

### [50] Native geospatial types · **XIAO** · ~0:45
Location data is everywhere — deliveries, IoT, mobility, mapping, risk. In 4.2, Spark adds **`GEOMETRY` and `GEOGRAPHY` as native SQL types** — enabled by default, no extra packages. `GEOMETRY` is planar — flat-surface shapes; `GEOGRAPHY` is geodetic — on the ellipsoid. There's **full SRID support** backed by a complete spatial reference registry. You read and write natively in **Parquet** — types and SRID preserved — and it's available across SQL, DataFrames, PySpark, and the Thrift server. And it's aligned with the **open Parquet and Iceberg geospatial spec**, so it's interoperable across engines.

### [51] Geospatial in action · **XIAO** · ~0:25
*(Code slide. Note: only claim the shipped functions on screen.)* Here are the functions that ship in 4.2 — `ST_GeomFromWKB`, `ST_GeogFromWKB`, `ST_AsBinary`, and the SRID helpers. Construct geometries, round-trip them through Parquet, and the type and SRID survive.
**▶ HANDOFF — XIAO → DB.** *(XIAO:)* That's the first half — everything you can reach through SQL and Python. To take you through how data **moves** — pipelines, streaming, sources — and where Spark is headed, let me hand over to DB.

---

## 05 · PIPELINES & AUTO CDC — Slides 52–55 · DB · ~2:30

### [52] Section divider — Declarative Pipelines & Auto CDC · **DB** · ~0:15
Thanks, Xiao. Let's talk about one of the most painful things in data engineering — keeping a table in sync with a stream of changes.

### [53] Applying a change feed is the hard part · **DB** · ~0:50
The classic need: keep a lakehouse replica of an operational table in sync — rows get inserted, updated, deleted. A CDC event has four parts — the **key**, the **operation**, the **sequence**, and the **payload**. Sounds simple. It isn't. Events arrive **out of order** and across micro-batch boundaries; change events can be **partial**; and your retries have to stay **idempotent**. The hand-rolled version — `foreachBatch` plus `MERGE` plus tombstone tracking — easily runs to **hundreds of lines** of fragile code. Everyone rewrites it, and everyone gets it slightly wrong.

### [54] Auto CDC: declare it, Spark reconciles it · **DB** · ~0:55
So 4.2 lets you **declare it instead.** You specify the keys, the sequencing, and the delete condition — and Spark **reconciles each batch against the target, in sequence order, and merges it for you.** It's out-of-order safe: inserts and updates both reduce to upserts with identical semantics, and deletes are driven by the feed. In 4.2 you get a Python `create_auto_cdc_flow()` and Spark Connect support, storing **SCD Type 1** — with SQL syntax and SCD Type 2 history on the way. And it lives inside **Spark Declarative Pipelines**, so checkpoints, dependencies, retries, and idempotency are all handled for you.

### [55] Auto CDC in action · **DB** · ~0:30
*(Code slide.)* Here's the whole thing — declare the flow, point it at the change feed, name your keys and sequence column. Those few lines replace the hundreds we just talked about. And speaking of continuous data — let's talk streaming.

---

## 06 · STRUCTURED STREAMING — Slides 56–60 · DB · ~2:15

### [56] Section divider — Structured Streaming · **DB** · ~0:12
Streaming in 4.2 is about two things: evolving running queries safely, and a state store you can actually trust.

### [57] Evolve running pipelines safely · **DB** · ~0:40
A long-standing pain: a streaming query's sources were identified by **position** — so you couldn't safely add, remove, or reorder them without breaking the checkpoint. Now you name them with **`IDENTIFIED BY`** — identity is decoupled from position. So you can evolve the sources in a running query without losing your checkpoint. `DataStreamWriter.name()` gives sinks the same story. And it's available across SQL table-valued functions, Scala, and PySpark — Classic and Connect.

### [58] IDENTIFIED BY in action · **DB** · ~0:20
*(Code slide.)* Named sources — and then a query that adds a new one, restarts, and keeps its checkpoint. No full reprocess.

### [59] Richer joins, lower latency · **DB** · ~0:35
Streaming joins get richer: stream-stream **Inner and LeftSemi** joins now run in **Update** output mode, and LeftSemi joins use **less state-store space**. And the **real-time mode** trigger — that low-latency execution path — is now accessible from **PySpark**. *(Hold the thought on real-time mode — we'll come back to where it's going in Looking Ahead.)*

### [60] A state store you can trust · **DB** · ~0:35
The state store is the heart of any stateful streaming job, and in 4.2 it gets a lot more robust. **Automatic snapshot repair** recovers corrupted state snapshots — no manual intervention. **Row checksums and checkpoint verification** detect corruption early, before it spreads. And lagging snapshot uploads are now handled automatically — **on by default**. Less paging, more sleeping.

---

## 07 · DATA SOURCE V2 — Slides 61–75 · DB · ~5:00

### [61] Section divider — Data Source V2 · **DB** · ~0:15
Next — Data Source V2. This is the plumbing that connects Spark to the world's data, and it grew up a lot this release.

### [62] One integration API for every data source · **DB** · ~0:40
DSv2 is the de-facto API for plugging a data source into Spark — Delta, Iceberg, and beyond. Users get **consistent SQL and behavior** across sources; connectors get DML, streaming, and all of Spark's optimizations for free. And it's evolving fast: in 4.1 and 4.2, **row-level DML, CDC, schema evolution, transactions, and smarter pushdown** all live here. If you want to go deep, there's a whole talk at this Summit — *"Spark DSV2: Growing Up Fast"* by Szehon Ho and Anton Okolnychyi.

### [63] Roadmap (Reading changes) · **DB** · ~0:06
*(Point at item 1.)* Three themes. First — reading changes.

### [64] Ask the table: "what changed?" · **DB** · ~0:45
CDC, at its core, is the **row-level change history of a table** — which rows were added, updated, or deleted. It's a typed log in commit order: each row tagged insert, update preimage or postimage, or delete — carrying the values, the operation, and the ordering. And it's **load-bearing infrastructure** — incremental ETL, audit, replication. And increasingly: vector search indexes, materialized views, and streaming tables that all refresh **on commit.** When you can ask a table "what changed," a whole class of incremental systems gets simpler.

### [65] Two problems with CDC today · **DB** · ~0:50
But CDC today has two problems. **One — everyone reinvented it.** Delta has `table_changes()`, Iceberg has `create_changelog_view`, Hudi has incremental reads — same idea, three syntaxes, three sets of defaults. **Two — write-time CDC taxes every writer.** Change files get written on **every** update commit — roughly 1.2× the write time — paid up front, whether anyone ever reads them or not. And here's the data that motivated us: in a 60-day window, **206 million** batch CDC queries — and **68% of them needed no stored change files at all.** So 4.2 makes **read-time CDC** first-class: only the reader pays, and any engine can write the table.

### [66] One CHANGES API for any DSv2 source · **DB** · ~0:45
The answer is one unified API. A single SQL **`CHANGES`** clause — batch over a version or timestamp range, or streaming with `STREAM t CHANGES`. The same shape from DataFrames — `.changes()` on read and readStream. The **same syntax** for Delta, Iceberg, Hudi — anything that ships a Changelog implementation. **Spark owns the hard post-processing** — carry-over removal, update detection, net-change collapse. And connectors only ship a tiny contract: one interface, three capability booleans, and row id / row version. The complexity moves into Spark, where it's written once.

### [67] Reading changes in action · **DB** · ~0:25
*(Code slide.)* `SELECT ... FROM t CHANGES (...)` — version range in, typed change rows out. Same query whether the table underneath is Delta or Iceberg.

### [68] Delta goes read-time — by default · **DB** · ~0:50
And here's the payoff for Delta specifically: **CDC on every Delta table, with nothing to enable.** No table property, no `ALTER TABLE`, no ticket to the table owner. **Nothing to pay at write time** — no change files; changes are computed only when a reader asks. It's built on two columns Delta already tracks — a stable **row id** and a **row commit version**. Within a commit, removed-plus-added rows matched by row id become updates; lone adds are inserts, lone removes are deletes. It requires Row Tracking — on by default in Databricks, opt-in in Delta OSS — and write-time CDC stays available for the workloads that still want it.

### [69] Roadmap (Smarter writes) · **DB** · ~0:06
*(One breath.)* Second theme — smarter writes.

### [70] Smarter writes: INSERT & MERGE · **DB** · ~0:40
Writes get smarter and safer. `INSERT INTO ... WITH SCHEMA EVOLUTION` lets the target schema **evolve with the incoming data**, and sources with fewer columns are accepted — missing fields filled automatically. There's new `INSERT INTO ... REPLACE` syntax, and `BY NAME` with `REPLACE WHERE`. `MERGE` gets fixes — schema evolution with `WHEN MATCHED THEN DELETE`, and nested-field preservation. Plus `TABLESAMPLE SYSTEM` block sampling, pushed down to DSv2 and JDBC.

### [71] Schema evolution in action · **DB** · ~0:20
*(Code slide.)* New column shows up in the source — and with `WITH SCHEMA EVOLUTION`, the target just absorbs it. No manual `ALTER`.

### [72] Transactions: atomic, isolated DML · **DB** · ~0:40
DML is fundamentally read, transform, write back — and without isolation, a writer can silently overwrite concurrent changes it never saw. So in 4.2, Spark **begins a transaction for every DML operation** and tracks all the reads and writes inside it. Connectors validate concurrent commits **at commit time** — so a stale read **fails cleanly** instead of corrupting your data. This is engine-level today for single-statement DML; multi-statement and cross-catalog transactions are future work.

### [73] Observable, evolvable DML · **DB** · ~0:35
And your DML becomes observable. `MERGE`, `UPDATE`, `DELETE` now report **operation metrics** — rows inserted, updated, deleted, copied — surfaced right where you already look: Delta's `DESCRIBE HISTORY`, Iceberg snapshot summaries. `MERGE` and `INSERT` support `WITH SCHEMA EVOLUTION`, with the connector deciding which changes are valid. And table refresh is consistent now — Spark refreshes before writes and non-pinned reads, so views over external tables stay correct.

### [74] Roadmap (Smarter pushdown) · **DB** · ~0:06
*(One breath.)* And the third theme — smarter pushdown.

### [75] Smarter pushdown: PartitionPredicate · **DB** · ~0:35
Here's a subtle but real gap we closed. Before, **arbitrary partition filters** — ones with UDFs or non-standard expressions — got **lost** between Catalyst and the DSv2 connectors. Built-in file sources could prune on them; connectors couldn't. The new **`PartitionPredicate` API** lets **any** Catalyst partition filter be evaluated against partition values, while preserving the DSv2 abstraction. It works across scans, metadata-only `DELETE WHERE`, and runtime filters. The concrete win: `WHERE udf(month(ts)) = 'JAN'` now **prunes partitions** in Iceberg and Delta. Less data read, faster queries.

---

## 08 · PERFORMANCE, UI & OPERATIONS — Slides 76–79 · DB · ~2:00

### [76] Section divider — Performance, UI & Operations · **DB** · ~0:12
Last of the shipped features — the things that make Spark faster, easier to see into, and easier to run.

### [77] A modern Spark Web UI · **DB** · ~0:30
The Web UI gets a real refresh. **Dark mode** and a faster interface. **Interactive SQL plans** — pan, zoom, search, and compare the initial versus final AQE plan **side by side**, which is huge for understanding what the optimizer actually did. And the environment page now highlights your **non-default configs**, with one-click export.

### [78] Faster & leaner · **DB** · ~0:35
On raw performance, four headlines. **Faster scans** — optimized vectorized Parquet reading for bytes, booleans, binary, and null runs. **Smarter plans** — pre-aggregation for multiple `COUNT(DISTINCT)`, and more whole-stage codegen. **Leaner memory** — bounded external merge and eager memory release **cut OOMs.** And **faster I/O** — zero-copy transfers and lower JVM-to-Python overhead. These add up across every workload, with no code change.

### [79] Runtime & operations · **DB** · ~0:35
And on the operations side: we now build and run on **Java 25**. **Kubernetes** gets in-place executor and PVC resizing, smaller images, and NetworkPolicy isolation. The **History Server** scales — multiple log directories and on-demand metadata loading. And **consistent results** — query-level retry for indeterminate shuffles, with accurate SQL metrics under AQE. That's 4.2, shipped. Now — the part we're most excited about. What's next.

---

## 09 · LOOKING AHEAD — Slides 80–97 · DB · ~5:00

### [80] Section divider — Looking Ahead · **DB** · ~0:15
Everything up to here ships in 4.2. This last stretch is **ongoing work** — SPIPs already in flight for the releases after this. We want you to see where Spark is going.

### [81] Roadmap (five topics) · **DB** · ~0:12
*(Gesture across the five.)* Five things on the horizon: Project Feather, a language-agnostic UDF protocol, nanosecond timestamps, real-time mode for stateful streaming, and a faster release cadence. We'll touch each one. *(This roadmap reappears between topics so you always know where we are.)*

### [82] Project Feather: fast local queries · **DB** · ~0:45
First — **Project Feather.** The goal: make Spark's **small-data queries run fast on a laptop.** Spark gives you one API for big and small jobs — but for small queries, the **fixed costs dominate.** A query over less than 100 megabytes can still take *seconds* — because planning, scheduling, serialization, and the shuffle machinery were all built for distributed execution. Feather attacks three areas: query compilation and scheduling, the cache format, and shuffle overhead.

### [83] Faster planning & scheduling · **DB** · ~0:35
On planning: a **single-pass analyzer** — bottom-up resolution that's more predictable and near-linear in query size. When the scan is one small file, run the **whole query as a single task and stage** — skip the shuffle entirely. AQE gets identity-based stage hashing so the rule loop never blocks on unrelated work. And smarter join and broadcast choices. Less overhead between you and your answer.

### [84] Arrow-based DataFrame cache · **DB** · ~0:30
On caching: interactive local analysis is load-once, cache, iterate — `df.cache` is a **core experience**, not a storage detail. Today's cache is **row-oriented**, so every re-read pays avoidable CPU and memory. We're moving to an **Arrow columnar cache with IPC compression** — smaller footprint, faster second, third, fourth queries. Implementation is underway.

### [85] Shuffle-free local execution · **DB** · ~0:35
And on shuffle: disk-based blocking shuffles are right for clusters — and **overkill** when everything runs in one JVM. So there's a new **multithreaded local mode** — FIFO channels between threads replace shuffle files, with back-pressure and pipelined execution so downstream operators start as soon as data arrives. It only kicks in when the planner is confident the query fits safely in one JVM. Put it together — faster planning, the Arrow cache, shuffle-free local — and you get **Spark that works at laptop scale, with a clear path back to the cluster.**

### [86] Roadmap (UDF protocol) · **DB** · ~0:05
*(One breath.)* Second — and this one closes a real gap.

### [87] Language-agnostic UDF protocol · **DB** · ~0:45
Remember the Spark Connect slide — clients in Go, Rust, Swift, TypeScript, .NET? **UDFs are the one promise Connect can't keep for most of them.** SPIP SPARK-55278 fixes that. Today, every language has to re-invent the **entire** UDF stack — and the planning rules are hard-wired to Python. Serialized UDFs are tied to a specific runtime, so a server upgrade can quietly break them. And UDF execution is **caged inside the cluster** — so a heavy or GPU UDF forces you to over-size every executor. We can do better.

### [88] Three pillars · **DB** · ~0:40
The design rests on three pillars. **Language-agnostic planning** — plan UDFs by their **execution shape** — scalar, mapPartitions, grouped map, table function, aggregator — not by language. **One standardized execution protocol** — init, data, finish — over gRPC and Protobuf with Arrow batches, versioned, streaming, with back-pressure. And a **declarative worker spec** — the client declares how to start, connect, and clean up a worker: local process, container, or a remote GPU box. Each pillar makes one thing replaceable without touching the others.

### [89] Status & what it unlocks · **DB** · ~0:30
Where it stands: the **SPIP vote passed**, and the first pieces are landing in apache/spark under `/udf/worker`. PySpark's behavior is **unchanged** — one shared core, with the transport becoming pluggable; the socket stays the default, gRPC is opt-in. And what it unlocks is big: **UDFs in any language**, runtimes that **upgrade independently** of Spark, and heavy or GPU UDFs running on **dedicated workers.**

### [90] Roadmap (Nanosecond timestamps) · **DB** · ~0:05
*(One breath.)* Third — a precision fix that interop has been waiting for.

### [91] Nanosecond-precision timestamps · **DB** · ~0:40
SPIP SPARK-56822. Today, Spark timestamps stop at **microseconds** — so nanosecond Parquet either errors out or falls back to a raw `LongType`, losing all the time-zone and timestamp semantics. This adds **parameterized timestamp types** — `TIMESTAMP(n)` and the NTZ/LTZ variants — with `n` from 0 to 9, where 9 is nanoseconds. The value model is compact — epoch micros plus nanos-within-micro — which keeps today's calendar range and avoids the single-INT64 "range cliff." And it's **fully additive and backward compatible** — the existing micro types stay the default; nanosecond behavior only appears when you ask for `(n)`.

### [92] Declaring nanosecond timestamps · **DB** · ~0:20
*(Code slide.)* New Catalyst types, wired end-to-end — parser, codegen, Parquet, casts. The point: clean interop with PyArrow, Trino, ClickHouse, DuckDB — the rest of the data world that already does nanoseconds.

### [93] Roadmap (Real-time stateful streaming) · **DB** · ~0:05
*(One breath.)* Fourth — and this connects back to streaming.

### [94] Real-time mode for stateful streaming · **DB** · ~0:40
Earlier I mentioned real-time mode. SPARK-54699 **extends it to stateful queries** — targeting roughly **100-millisecond** end-to-end latency. It builds on real-time mode for **stateless** queries, which shipped in **4.1** — now bringing that same low latency to `transformWithState` and aggregations. Two pieces make it work: a **streaming shuffle** — a push-based exchange that streams task output straight downstream, instead of waiting on batch boundaries — and **concurrent stage scheduling**, so multiple plan stages run at once and stateful operators keep pace. Real-time stateful streaming, at Spark scale. And the last piece of the roadmap is about how we'll actually *ship* all of this.

### [95] Roadmap (Faster releases) · **DB** · ~0:05
*(One breath.)* The last one is about *cadence.*

### [96] Faster, predictable releases · **DB** · ~0:50
SPIP SPARK-54633 — a **two-layer release model.** **Quarterly minor** releases, every three months. **Annual major** releases. The minors ship new features, performance, and API additions — but they **freeze dependencies, behavior, and config defaults**, so upgrading stays drop-in safe. The majors are the once-a-year home for dependency bumps and breaking changes. Long-lived `branch-4.x` and `branch-5.x` cut active maintenance branches from 6-plus down to **two or three**, and the **final minor of each major becomes an 18-month LTS.** The quarterly cadence begins with **4.3** — so **4.2 is the bridge release** — and the dev list also voted to retire the monthly preview releases now that minors ship every quarter. Predictable for you, sustainable for the community.

### [97] An example release calendar · **DB** · ~0:35
*(Walk the four cards.)* Concretely: **4.2** this month — the bridge. **4.3** around September — first on the quarterly cadence. **4.4** around December — the final 4.x minor, and the **18-month LTS** the ecosystem can standardize on. And **5.0** in January — the annual major, where the big dependency upgrades and breaking changes land. Dates are illustrative — but the *rhythm* is the commitment.

---

## CLOSE — Slides 98–99 · DB · ~1:00

### [98] Now Available: Apache Spark™ 4.2 · **DB** · ~0:40
So — that's 4.2. If you take three things home: **one**, analytics is going native — metric views, vector search, sketches, geospatial, all in the engine. **Two**, Python and your data sources are faster and more open than ever. And **three**, the data-movement story — CDC, transactions, pipelines — grew up across SQL, DataFrames, and Data Source V2. It's available now. Go try it, and tell us what breaks.

### [99] Join the community today! · **DB + XIAO** · ~0:20
*(DB:)* All of this is the work of the Apache Spark community — and that community is open to you. Grab the release at spark.apache.org, the source on GitHub, and join the dev and user lists — we'd love your contributions, and your bug reports. *(XIAO:)* Thank you, DAIS — enjoy the rest of the Summit!

---

*Total target ~40:00. If over: trim the four SQL "in action" code slides (39–42) to a single sentence each, shorten the roadmap dividers to a pointed finger, and tighten the DSv2 section (62, 73) — that recovers ~2 minutes without losing a single feature.*
