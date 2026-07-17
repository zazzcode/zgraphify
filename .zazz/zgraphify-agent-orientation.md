# Zgraphify Agent Orientation

## Purpose

This document is the operational orientation for agents and contributors working in
the Zgraphify fork. Its objective is to apply the Zazz methodology to future fork
development. It explains how the fork relates to upstream Graphify, which branches
are safe to change, and how the Zazz methodology applies here.

Read this document before changing branches, synchronizing upstream changes, or
preparing a pull request. For all other repository rules, read `../AGENTS.md`.

## Current Git Operating Model

We currently use one regular Git checkout and ordinary Git branches. We are **not**
using the Zazz bare-repository plus sibling-worktree model yet.

- Do not create, convert, remove, or assume Git worktrees.
- Keep one focused change or feature per branch.
- Use concise, flat, hyphenated branch names; do not use `/` in branch names.
- A future owner-approved migration may adopt Zazz worktrees. Until then, this
  document and `AGENTS.md` take precedence over the vendored worktree guide.

## Branch Roles

```text
upstream/v8  ->  origin/main  ->  origin/<feature-branch>
upstream code    fork integration    focused fork work
```

| Branch or reference | Purpose | Rules |
| --- | --- | --- |
| `upstream/v8` | The upstream project's active development line. It is the source of future upstream updates. | Treat as read-only. Do not add Zazz or fork feature work here. |
| `v8` | This fork's local snapshot of the upstream baseline. | Keep it free of fork-specific work; use it only for comparison or upstream-compatible contribution branches. |
| `main` | Zgraphify's integration branch and normal pull-request target. It contains the Zazz foundation plus accepted fork work. | Keep it clean. Integrate reviewed work and upstream `v8` updates here. |
| `lady-bug-integration-mvp` | The initial fork feature branch, based on `main`. | Implement only the feature's approved scope; target `main` in its PR. |
| Future feature branches | One focused fork change each, created from current `main`. | Target `main` in PRs. |
| Upstream-contribution branches | A focused change that may be proposed to Graphify itself, created from `upstream/v8`. | Keep it independent from fork-only work and target `upstream/v8` in an upstream PR. |

The `upstream` remote is configured as `https://github.com/Graphify-Labs/graphify.git`.
It is a fetch-only source for the upstream project; fork work MUST be pushed only to
`origin`.

## Zazz Ownership Boundary

The `.zazz/` directory is fork-owned methodology and orientation content. It exists
on `main` and on fork feature branches created from `main`.

- `.zazz/` MUST NOT be added to `v8`, `upstream/v8`, or an upstream-contribution
  branch.
- An upstream-contribution branch begins from `upstream/v8`, so it does not inherit
  `.zazz/` or other fork-only process changes.
- A fork feature may inform an upstream proposal, but the upstream contribution
  MUST be re-created as a focused, independently reviewable change.

## Everyday Branch Workflow

1. Start from current `main`: fetch and update it from `origin/main`.
2. Create a focused branch from `main` using a flat descriptive name.
3. Make, test, and commit the bounded change on that feature branch.
4. Open a pull request from the feature branch to `main`.
5. Merge only reviewed changes into `main`.

Do not start ordinary fork feature work from `v8`, and do not merge a feature
branch directly into `v8`.

## Upstream Synchronization

Upstream maintains `v8` as its active development branch. `main` exists upstream,
but upstream's contributor guidance identifies `v8` as the current development
target. Therefore, synchronize this fork from `upstream/v8`, not upstream `main`.

After the `upstream` remote is configured:

1. Fetch `upstream`.
2. Review what changed between `main` and `upstream/v8`.
3. Integrate the selected upstream changes into `main` with a documented merge or
   rebase; resolve and test any conflicts there.
4. Update feature branches from the resulting `main` only when needed.

Never overwrite `main` with upstream history: it intentionally carries fork-owned
Zazz and feature commits. Preserve that boundary so upstream pulls remain
reviewable and recoverable.

## Potential Upstream Contributions

Some Zgraphify work may be valuable to Graphify itself. For example, support for
PostgreSQL SQL (`psql`) or Transact-SQL (`T-SQL`) may become an upstream proposal
when the implementation is generally useful and does not depend on fork-only
workflow or product decisions.

For an upstream contribution:

1. Fetch `upstream` and start a new flat branch from `upstream/v8`.
2. Implement only the upstream-ready change; do not copy `.zazz/`, fork feature
   code, or unrelated cleanup into the branch.
3. Follow upstream's contributor notes, including its test and extractor-fixture
   expectations.
4. Open a pull request targeting upstream `v8`.
5. If the upstream change is accepted, later synchronize `main` from
   `upstream/v8` rather than manually duplicating the merged patch.

## Contributor Notes Versus Fork Policy

Use the source of guidance that owns the decision:

| Decision | Authority |
| --- | --- |
| Graphify behavior, code architecture, test expectations, and upstream PR conventions | Upstream documentation and established code patterns |
| Zazz documents, this fork's branch topology, feature priorities, and integration decisions | This fork's `AGENTS.md` and `.zazz/` documentation |
| A patch intended for upstream | Start from `upstream/v8` in a separate branch and exclude fork-only Zazz changes unless upstream requests them |

For normal fork changes, continue to honor useful upstream developer rules: use
conventional commit subjects, run `uv run pytest tests/ -q` before a PR, and add
fixtures plus `tests/test_languages.py` coverage for new language extractors.

## Zazz Documentation

- `.zazz/project.md` is the fork project overview.
- `.zazz/standards/index.yaml` is the entry point for selectively loading relevant
  engineering standards before code changes or review; its paths are relative to
  the repository root.
- `.zazz/docs/` contains the complete vendored Zazz general-documentation library.
- `.zazz/features/` holds long-lived feature context.
- `.zazz/specifications/` holds bounded deliverable specifications.
- `.zazz/ephemeral/` is untracked local workspace for run logs, QA notes, handoffs,
  evidence, and scratch material.

When the worktree migration is approved, update this document, `AGENTS.md`, and the
branch workflow before asking agents to use worktrees.
