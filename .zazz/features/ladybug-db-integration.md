# Feature: LadybugDB Integration

**Status:** Discovery — proposal in progress
**Related proposal:** [LadybugDB graph store integration](../proposals/ladybug-db-integration.md)

## Feature Summary

Evaluate a durable embedded graph-store capability for the Zgraphify fork using
LadybugDB, while preserving Graphify’s graph semantics, portable artifacts, and
current user-facing behavior.

## Problem and Value

Graphify currently persists its graph as a node-link JSON document and rehydrates it
into NetworkX for queries and analysis. A suitable embedded graph database may improve
durability, graph-native querying, and scalability for larger graphs, but only if it
preserves the current extraction, incremental-update, and query contracts.

The feature matters to future fork development because it could establish a more
capable persistence boundary without requiring a server-managed graph database.

## Current State

No LadybugDB integration exists. `graphify-out/graph.json` is the current persisted
graph artifact; NetworkX remains the in-memory graph model for graph construction,
clustering, analysis, and much of the query surface.

## Desired Future State

Zgraphify can use an embedded graph-store backend selected through a documented,
testable storage boundary. The backend preserves Graphify node and edge identity,
direction, metadata, incremental source ownership, and derived outputs. Any change
to the canonical store is made only after compatibility and operational evidence are
reviewed.

The intended product shape is optional and project-persistent: a project deliberately
uses either the existing JSON/NetworkX store or LadybugDB. The selection should come
from a committed configuration source, not only an environment variable, so a graph
does not change storage formats unexpectedly across CLI, watch, CI, and MCP sessions.

## Core Concepts

- **Canonical graph state:** The authoritative persisted representation of a project
  graph.
- **Compatibility artifact:** A generated representation, initially `graph.json`,
  retained for current consumers, portability, or inspection.
- **Storage adapter:** The boundary that maps Graphify extraction and graph records to
  a persistence backend.
- **Source ownership:** The `source_file` basis for replacing changed-file records and
  pruning deleted-file records during incremental updates.
- **Optional backend selection:** A durable per-project setting that selects JSON or
  LadybugDB and makes the resulting canonical artifact unambiguous.

## System Flow

```text
extractors -> canonical graph records -> storage adapter -> persisted graph state
                                              |                    |
                                              v                    v
                                      compatibility export   query/read adapter
```

The first discovery work must show how existing Graphify construction and read paths
can use this boundary without changing their observable behavior.

## Feature-Level Success Criteria

- A proposed backend can represent the full Graphify graph contract, including
  directional edges, confidence, provenance, communities, and hyperedges.
- Incremental updates safely replace and prune source-owned data without corrupting
  unrelated graph content.
- Existing query and analysis behaviors have documented compatibility evidence before
  any default backend changes.
- Portable backup and interchange artifacts have an explicit policy.
- The dependency remains embedded and local-first, with no new required service.
- Any claimed performance benefit is demonstrated with comparable JSON and Ladybug
  workloads; it is not inferred solely from the use of a database.
- Ladybug query mode does not retain an unbounded NetworkX copy of the persisted graph
  during normal served queries; any temporary NetworkX algorithm use is explicit and
  measured.

## Discovery Status

The feature is in proposal discovery. The proposal is the current decision surface;
this document records the long-lived feature intent while that investigation proceeds.
No implementation specification, delivery schedule, or backend selection is defined.

## Constraints and Non-Goals

- Do not remove NetworkX, `graph.json`, or existing CLI/MCP behavior during discovery.
- Do not introduce a required external graph server.
- Do not assume the embedded database safely supports Graphify's current independent
  writer and reader processes without an explicit concurrency design.
- Do not claim lower build memory while construction still holds complete NetworkX and
  Ladybug representations of the same graph.
- Do not create an implementation specification until the proposal selects a bounded
  first experiment.

## Open Questions

- Should LadybugDB first be a read-optimized projection or eventually become the
  canonical store?
- Which graph query path offers the best first validation target?
- What schema and migration strategy can preserve Graphify hyperedges and metadata?
- Which portable formats should be retained for backup, sharing, and test fixtures?

## Future Handoff

When the proposal reaches a decision, update this document with the selected backend
role and the first meaningful capability outcome. Only then create a bounded
specification for the implementation spike.
