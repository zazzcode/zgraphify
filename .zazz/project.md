# Zgraphify Project Overview

## Purpose

Zgraphify is this fork's working integration of the upstream Graphify v8 codebase.
Graphify builds a local-first knowledge graph from source code and supported
documents so developers and coding agents can explore structure, dependencies, and
design rationale without repeatedly reading an entire repository.

## Development Objective

Future Zgraphify fork development uses the Zazz methodology. Zazz provides the
durable project context, standards routing, feature and proposal documents,
deliverable specifications, execution records, and review discipline that govern
fork-owned work from planning through integration.

## Branch Model

- `v8` is the upstream baseline retained for comparison and upstream synchronization.
- `main` is this fork's integration branch and pull-request target.
- `lady-bug-integration-mvp` is the first feature branch, based on `main`.

## Documentation Policy

The `.zazz/` directory is the repository's durable Zazz documentation root. Active
implementation artifacts belong in `.zazz/ephemeral/`, which is intentionally
untracked. See `../AGENTS.md` for the full operating policy.
