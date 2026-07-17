# Proposal: LadybugDB Graph Store Integration

**Status:** Discovery — no implementation decision approved
**Scope:** Technical-direction proposal and feature discovery
**Related feature:** [LadybugDB integration](../features/ladybug-db-integration.md)

## Context and Problem Statement

Graphify currently builds an in-memory NetworkX graph and serializes it as
`graphify-out/graph.json`. That JSON file is not a passive export: it is the
incremental-build baseline and the input for querying, serving, visualizations,
global-graph merging, PR analysis, and work-memory overlays. The pipeline and
current data shape are described in [ARCHITECTURE.md](../../ARCHITECTURE.md).

JSON makes the graph portable and inspectable, but it requires full-file loading and
NetworkX rehydration for graph operations. It also makes updates, query planning,
indexes, and transactional durability application-level responsibilities.

This proposal investigates whether [LadybugDB](https://github.com/LadybugDB/ladybug)
should become a graph persistence and query backend for fork-owned Graphify work.
LadybugDB is an embedded, on-disk property-graph database with Cypher, columnar
disk storage, adjacency indexes, ACID transactions, and a Python API. Its Python API
opens an on-disk database through `ladybug.Database(path)` and
`ladybug.Connection`; it also supports in-memory use. [Ladybug Python API](https://docs.ladybugdb.com/client-apis/python/)

## Scope

This proposal evaluates persistent storage and graph-query integration for Graphify
graphs. It includes the graph lifecycle, schema mapping, migration safety, query
compatibility, and the role of Parquet or another columnar interchange format.

It does not approve a dependency, remove NetworkX, define a deliverable
specification, change the Graphify public CLI, or decide a product release plan.

## LadybugDB Runtime and Packaging Deep Dive

### What an integration would install and run

For the normal embedded integration, Graphify would add the `ladybug` Python package
as an optional dependency and open the database from its own Python process. The
package exposes `ladybug.Database(path)` and `ladybug.Connection`; Graphify would not
need to operate a separate database server or require users to manage a standalone
database binary. [Python client API](https://docs.ladybugdb.com/client-apis/python/)

LadybugDB also has developer-facing tooling, but that is distinct from Graphify's
runtime requirement. The `lbug` CLI can aid local inspection and administration, and
Ladybug Explorer is an optional Docker-hosted visualization. Neither should be a
required dependency for a first Graphify backend. Optional Ladybug extensions should
also remain out of the initial dependency boundary unless a selected query feature
requires one. [Extensions](https://docs.ladybugdb.com/extensions/)

The spike must validate the exact Ladybug release, supported Python versions,
platform wheels, package footprint, license compatibility, and CI installation
experience before the optional dependency contract is approved. The intended shape is
an optional extra—not a new mandatory Graphify install—for example a future
`graphifyy[ladybug]` extra if the project keeps its existing distribution naming. That
would follow the repository's current optional-extra convention: development installs
could use `uv sync --extra ladybug`, while CLI users could use
`uv tool install "graphifyy[ladybug]"`. These are proposed commands, not a change to
the package contract yet.

### Live database files versus Parquet

LadybugDB persists an on-disk database as a single database file, conventionally with
an `.lbug` suffix. It creates transient sibling files such as write-ahead-log,
shadow, and temporary files while operating. Those runtime artifacts belong under
`graphify-out/`, never in source control; the proposed path to evaluate is
`graphify-out/graph.lbug`. Their lifecycle and backup behavior need explicit tests.
[Database files](https://docs.ladybugdb.com/developer-guide/files/)

The live store is therefore the `.lbug` database, not a Parquet file. Parquet remains
valuable for bulk loading, portable snapshots, reproducible fixtures, and database
export. LadybugDB supports `COPY FROM` for bulk import and database export that emits
schema plus Parquet data by default. [Parquet import](https://docs.ladybugdb.com/import/parquet/)
[Database migration](https://docs.ladybugdb.com/migrate/)

Graphify would treat the `.lbug` file as an opaque database artifact. LadybugDB owns
its internal columnar layout, compressed-sparse-row adjacency and join indexes, and
transactional state; Graphify supplies typed graph records and receives query results.
[Ladybug architecture overview](https://docs.ladybugdb.com/)

The proposed runtime path does not require generating Parquet first. Graphify can
create the typed node and relationship tables, then write the graph records it already
builds through Ladybug's Python/Cypher API. A small update can use parameterized
`CREATE` or `MERGE` statements; a large full rebuild can use the database's supported
bulk-load mechanism once a safe staging boundary is defined. [Import overview](https://docs.ladybugdb.com/import/)
Parquet should be evaluated only when its bulk throughput, portability, or recovery
value justifies the additional serialization step.

### Concurrency and process lifecycle

LadybugDB supports multiple connections from one read-write `Database` object, but
its documented model does not permit independent read-write or read-write plus
read-only database instances against the same file at the same time. This matters
because Graphify can currently update a graph in one process while a separate MCP
server or CLI process rereads `graph.json`. [Concurrency](https://docs.ladybugdb.com/concurrency/)

An eventual backend design must choose one of these operational models:

| Model | Shape | Assessment |
| --- | --- | --- |
| Single-owner service | One process owns the read-write database and serves queries. | Most natural for concurrent watch and MCP use, but changes local CLI lifecycle. |
| Short-lived exclusive sessions | Build/update and query commands open the database only while active. | Simpler initially; callers must handle lock conflicts and retry guidance. |
| Immutable published snapshots | A writer builds a new database and atomically publishes a completed snapshot for readers. | Strong isolation, but requires an explicit publication protocol and storage overhead. |

No model is selected yet. A first optional backend experiment should make
single-process use correct before it expands to watch or shared HTTP MCP operation.

## Existing Graph Lifecycle and Integration Touch Points

| Concern | Current implementation | LadybugDB implication |
| --- | --- | --- |
| Graph construction | `graphify.build.build_from_json()` builds a NetworkX graph from extraction records. | Preserve the extraction contract first; add a mapper from canonical records to a Ladybug schema. |
| Incremental rebuild | `graphify.build.build_merge()` reads prior `graph.json`, replaces source-owned records, and prunes deleted sources. | The replacement and prune semantics need transactional equivalents keyed by `source_file`. |
| Primary persistence | `graphify.export.to_json()` writes NetworkX node-link JSON atomically and applies a node-count shrink guard. | Define the database location, transaction boundary, backup policy, and an equivalent incomplete-build guard. |
| Query and MCP serving | `graphify.serve._load_graph()` loads JSON into NetworkX; query ranking and BFS traversal run in Python. | A database alone does not preserve ranking behavior. Decide which operations move to Cypher and which remain in Python. |
| CLI tools | `query`, `path`, `explain`, `tree`, merge operations, and diagnostics read the JSON graph. | Introduce a storage boundary before converting callers; preserve the current CLI contract during migration. |
| Derived artifacts | HTML, call-flow HTML, reports, labels, and the learning overlay use JSON or NetworkX. | Keep `graph.json` as a compatibility/export projection initially, or provide a common read model. |
| Global and PR analysis | `global_graph.py`, `prs.py`, and `affected.py` load node-link JSON. | Define whether these stay JSON-based, obtain a snapshot from Ladybug, or receive database-native adapters. |
| Hyperedges and provenance | The current graph carries graph-level `hyperedges`, directional edge metadata, source provenance, and confidence. | Model hyperedges explicitly and preserve all metadata before considering database-native queries. |

The direct JSON readers are concentrated in `build.py`, `export.py`, `serve.py`,
`watch.py`, `cli.py`, `global_graph.py`, `affected.py`, `prs.py`, `callflow_html.py`,
`tree_html.py`, and `reflect.py`. This makes a direct replacement of one file format
too broad for a first implementation.

## Target Backend Shape

The preferred direction for discovery is a real optional data-store backend, not a
Parquet conversion layer. With `storage_backend = "json"`, Graphify preserves today's
JSON/NetworkX path. With `storage_backend = "ladybug"`, Graphify would build its
normal canonical records, persist them to `graphify-out/graph.lbug` through a storage
adapter, and execute supported read operations through a Ladybug-backed query adapter.

During development, a test-only dual-write path can generate both representations for
parity comparison. It should not become a permanent production requirement. A
Ladybug-selected project may still generate `graph.json` as an explicit compatibility
export for tools not yet moved to the adapter, but the database must be the selected
backend's authoritative graph state. The implementation must record this choice in
the generated output so a stale JSON file cannot be mistaken for the active store.

## Data Format Clarification

LadybugDB supports bulk import from and export to Parquet, and recommends Parquet for
large bulk imports. It can also export an entire database into schema plus data files,
using Parquet by default. [Parquet import](https://docs.ladybugdb.com/import/parquet/)
[Database migration](https://docs.ladybugdb.com/migrate/)

That does **not** establish Parquet as the live Graphify data store. LadybugDB’s
on-disk database has its own persistent storage and transaction model. Parquet is a
strong candidate for reproducible snapshots, bulk bootstrap, interchange, and backup
artifacts; it should not be assumed to replace the active database without a separate
decision. Ladybug also documents query-in-place support for some external columnar
sources, which may be useful later but does not remove the need to model Graphify’s
incremental updates. [External-data scanning](https://docs.ladybugdb.com/get-started/scan/)

## Candidate Data Model

The starting point should be one stable Graphify entity identity: the existing node
`id`. A first schema experiment can use a generic `Entity` node table with `id` as
the primary key and typed common fields such as `label`, `file_type`, `source_file`,
`source_location`, `community`, and `community_name`.

A generic `RELATES_TO` relationship table can connect `Entity` to `Entity` and carry
the current `relation`, `confidence`, `confidence_score`, and direction-preserving
metadata. Hyperedges likely require a separate `Hyperedge` node table and membership
relationships. Less-stable or nested JSON fields need a deliberate mapping decision:
typed columns where queried, a serialized metadata field where not queried, or a
normalized side table.

This is a hypothesis for a spike, not an approved schema. Ladybug table typing,
property support, multi-edge behavior, indexing, and migration behavior must be
validated against the installed version before a specification is written.

## Alternatives Considered

| Option | Description | Advantages | Costs and risks |
| --- | --- | --- | --- |
| Retain JSON and NetworkX | Keep the present canonical `graph.json` model. | Lowest implementation risk; all existing tools stay unchanged; transparent Git artifact. | Full-file load and in-process traversal remain; no database indexes or transactions. |
| Ladybug projection beside JSON | Keep JSON canonical initially; write and validate a Ladybug projection for selected query paths. | Reversible, supports parity testing, isolates schema work, and preserves all existing tools. | Dual-write complexity; must detect divergence; benefits are delayed. |
| Optional Ladybug backend | Select Ladybug per project; write its database directly and query it through an adapter, while retaining JSON as the default backend. | Matches the intended user experience and enables meaningful end-to-end performance tests. | Requires clear backend metadata, compatibility exports while callers migrate, and a complete update/read contract. |
| Ladybug as canonical store with JSON compatibility export | Make Ladybug authoritative while regenerating JSON for legacy consumers and portable artifacts. | Stronger transactional model and database-native traversal potential; controlled migration path. | Requires a storage abstraction and updates across all direct JSON readers; snapshot/export contract must be designed. |
| Ladybug-only replacement | Remove the canonical JSON graph and make every read surface database-native. | Eliminates duplicate persisted graph state after migration. | Highest risk; broad compatibility break; difficult to stage and verify. |
| Parquet-first graph store | Persist nodes and edges as Parquet and query through an engine or Ladybug external tables. | Columnar, portable bulk snapshots. | Does not naturally cover graph traversal, incremental replacement, transactional writes, or the current query semantics. |

## Tradeoff Analysis

The key choice is not JSON versus a columnar file. It is whether Graphify’s canonical
graph state should stay a portable node-link document, become an embedded database, or
temporarily exist in both forms while compatibility is proven.

Ladybug is best aligned with structural graph traversal and analytical workloads. The
current query layer, however, includes Python-side fuzzy matching, token scoring,
trigram indexing, graph traversal budgets, and display overlays. Those behaviors do
not automatically map to Cypher. A migration must demonstrate semantic parity for
existing `query`, `path`, `explain`, MCP, and watch behavior—not merely faster storage.

### Expected Performance Profile

LadybugDB is a credible performance candidate, but not a blanket performance upgrade.
Its columnar storage, adjacency representation, and Cypher execution should be most
useful when Graphify avoids repeatedly loading a large JSON document and NetworkX
graph, executes selective property filters, or runs repeated graph traversals over a
persisted graph. The benefit should grow with graph size, query frequency, and the
share of a workflow that can execute in the database.

It is unlikely to speed up AST or semantic extraction, graph construction that remains
in NetworkX, Python-only community analysis, rendering, or the existing fuzzy and
trigram ranking algorithms unless those operations are deliberately redesigned. A
dual-write backend may initially be slower because it writes both stores. The proposal
must therefore treat performance as a measured hypothesis, not an assumed outcome.

The spike should compare the JSON/NetworkX and optional Ladybug paths using the same
representative corpora and cold/warm runs. Record at least graph-open latency,
incremental-update duration, peak resident memory where practical, and representative
`query`, `path`, and MCP request latency. Semantic parity and correctness remain gates;
no benchmark target is proposed yet.

## Optional Backend Selection

The intended product direction is an optional persistent-store backend: users can
retain the current JSON/NetworkX behavior or deliberately select LadybugDB for a
project. A project-persistent configuration is preferable to a lone environment
variable because it makes the chosen graph format stable across shells, CI, watch
processes, and MCP sessions.

The first design should introduce a committed, project-scoped configuration source
with an explicit value such as `storage_backend = "json"` or
`storage_backend = "ladybug"`. The file name, command-line override, and exact
default remain open decisions; the configuration should be read once per invocation,
record the backend used in generated artifacts, and fail clearly when the optional
Ladybug dependency is absent. An environment variable may later be useful as an
explicit temporary override for CI or benchmarking, but must not silently select a
different canonical store in routine development.

## Standards and Constraints

- [Code structure](../standards/code-structure.md) favors an explicit storage seam
  over broad rewrites or a parallel unstructured implementation.
- [Python testing](../standards/python-testing.md) requires behavior-focused tests;
  parity fixtures should prove that a known graph yields equivalent observable results.
- [PR process](../standards/pr-process.md) requires one logical change per review;
  schema experimentation, adapter introduction, and a canonical-store switch should
  not be bundled into one change.
- Current upstream contribution conventions require a fixture and language tests for
  extractor work, but this proposal concerns fork-owned persistence architecture.

## Risks and Mitigations

| Risk | Mitigation |
| --- | --- |
| Query behavior changes while storage changes | Establish fixture-based parity tests before changing any default backend. |
| Loss of incremental rebuild or shrink-guard safety | Model source ownership, replacement, pruning, backup, and incomplete-build refusal explicitly in the adapter design. |
| Ladybug package or platform constraints | Validate supported Python versions, wheel availability, disk layout, package size, and CI support in a disposable spike. |
| Concurrent writer and reader processes contend for one database file | Choose and test an ownership or publication model before enabling watch or MCP access. |
| Backend choice changes unexpectedly across shells or sessions | Make the project configuration authoritative; treat environment variables only as explicit, observable overrides. |
| Schema cannot represent Graphify metadata or hyperedges cleanly | Prototype representative AST, semantic, directional, and hyperedge fixtures before committing to a canonical schema. |
| Dual persisted representations diverge | Declare one authority per stage and add deterministic export/import validation. |
| Database files are unsuitable for source control | Keep database runtime artifacts ignored; retain a portable JSON or Parquet export policy for tests, fixtures, and sharing. |
| Upstream divergence | Keep this investigation fork-owned. Any generally useful, isolated integration can later be recreated from `upstream/v8` for upstream review. |

## Recommendation

Approve a bounded discovery spike before selecting a production backend. The spike
should implement a small storage adapter boundary and prove that one representative
graph can round-trip between current extraction records and a local Ladybug database
without losing identity, direction, confidence, source ownership, communities, or
hyperedges.

The initial experiment should retain `graph.json` as the canonical compatibility
artifact for the existing default backend and as a parity/export artifact during
development. For a Ladybug-selected project, the experiment should write the `.lbug`
database directly and run one bounded query path through it. If it demonstrates query
and update parity plus operational benefit, the next proposal revision can select the
first production-facing backend boundary. Do not select Parquet as the active store in
advance; assess it separately as a bulk/snapshot interchange format.

## Approval Questions

- Is the discovery spike approved as fork-owned work?
- Must the first experiment prove a measurable performance or memory threshold, or
  is semantic and operational parity the immediate gate?
- Which current read surface is the priority candidate for database-backed queries:
  MCP serving, CLI query/path, watch updates, or another workflow?
- Should `graph.json` remain a committed/exported artifact after a successful
  migration, or only a generated compatibility artifact?
- What portability, offline, and licensing constraints must the embedded dependency
  satisfy?
- Which project-level configuration location and override precedence preserve the
  existing CLI experience while making the selected backend durable?

## Open Questions

- How should hyperedges and graph-level metadata map to Ladybug’s typed schema?
- Which graph properties need indexed typed columns versus serialized metadata?
- Can the current fuzzy ranking remain in Python over database candidates without
  negating the intended performance benefits?
- What concurrency and file-locking behavior is required for watch mode and MCP
  serving against the same database path?
- What version upgrade/export/import policy protects existing user graphs?
- Which concurrency model is acceptable for simultaneous watch, CLI, and MCP use?
- Which Parquet or database export artifacts are appropriate for Git, cache, and
  user-facing backup workflows?

## Discussion Log

- The owner requested an open-ended investigation into replacing the existing graph
  data store with LadybugDB.
- The owner raised Parquet or another columnar format as a likely data-content store;
  this proposal records it as an interchange/snapshot hypothesis rather than an
  assumed live-store design.
- The owner prefers an optional, project-persistent data-store selection over a
  session-scoped environment variable, with performance as a required validation
  hypothesis and richer query capability as the motivating value.

## Sign-off and Next-Phase Handoff

**Outcome:** Not approved; discovery remains open.

If approved, update the related feature document with the selected direction and
create a bounded specification for the first spike. That specification must define
the fixture corpus, schema mapping, parity evidence, dependency policy, and explicit
stop conditions before implementation starts.
