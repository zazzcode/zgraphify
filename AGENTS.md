# Zgraphify Agent Guide

## Purpose

The objective for future Zgraphify fork development is to use the Zazz methodology
for durable project context, bounded delivery specifications, selective standards
loading, execution evidence, and reviewable integration. Use this file first to
discover the documentation root, project workflow, and Graphify-specific constraints.

## Graphify Knowledge Graph

This project has a graphify knowledge graph at `graphify-out/`.

- Before answering architecture or codebase questions, read
  `graphify-out/GRAPH_REPORT.md` for god nodes and community structure.
- If `graphify-out/wiki/index.md` exists, navigate it instead of reading raw files.
- After modifying code files in this session, run `graphify update .` to keep the
  graph current (AST-only, no API cost).

## Docs Root

`Methodology docs root: .zazz`

The docs root is a repository-relative path. All fork-owned Zazz durable documents
and local execution artifacts belong under `.zazz/`; do not create a competing
top-level project documentation root.

## Required Zazz Orientation

Before changing repository content, branches, or Git remotes, read both:

1. `.zazz/docs/zgraphify-agent-orientation.md` for this fork's branch, upstream, and
   checkout policy.
2. `.zazz/standards/index.yaml` to route the task to the applicable standards.

Then load only the standards selected by the index for the files and activity in
scope.

## Documentation Operating Model

Documentation operating model: GitHub-only committed Markdown, with local ignored
execution artifacts under `.zazz/ephemeral/`.

| Artifact | Source of truth | Agent update rule |
| --- | --- | --- |
| Project overview | `.zazz/project.md` | Update when fork purpose or branch model changes. |
| Architecture | `.zazz/architecture/` | Record approved fork-specific direction and decisions. |
| Feature requirements | `.zazz/features/` | Update durable capability context when shipped behavior changes. |
| Plans, roadmap, and milestones | `.zazz/project.md` until dedicated docs are created | Add a stable pointer before treating another surface as authoritative. |
| Proposals | `.zazz/proposals/` | Keep decision proposals as committed Markdown. |
| Specifications | `.zazz/specifications/` | Commit and review bounded deliverable specifications with their implementation work. |
| RUN_LOG, QA, handoff, and evidence | `.zazz/ephemeral/` | Keep local and ignored unless the owner explicitly requests durable promotion. |
| Final implemented specifications | `.zazz/specifications/` | Update the completed specification with implementation and review evidence. |
| Standards | `.zazz/standards/` | Keep the active routing index and adopted standards here. |
| General Zazz reference docs | `.zazz/docs/` | Treat as the selective top-level reference-guide library; active artifacts stay in their root-level Zazz directories. |

`.zazz/ephemeral/` is local scratch space for run logs, QA notes, handoffs,
evidence, recovery notes, and scratch analysis. Do not put durable architecture,
feature, proposal, or specification documents there.

## Standards Loading Rules

Authoritative standards index:

- `.zazz/standards/index.yaml`

Before creating, modifying, reviewing, or validating code:

1. Read `.zazz/standards/index.yaml`.
2. Match standards by `applies_to.paths` and `applies_to.activities`.
3. Load only the relevant standards and any relevant companion documents.
4. Treat every listed path as repository-relative.
5. If no standard applies, follow the most recent intentional project pattern; ask
   the owner when the intended pattern remains ambiguous.

## Feature Context Rules

Feature index:

- `.zazz/features/index.yaml`

Read the feature index whenever product behavior, capability direction, or roadmap
context may affect the task. Load only the relevant feature document. Feature
requirements are durable capability context, not a substitute for a bounded
deliverable specification.

## Tracking and Coordination Policy

- Tracking system: GitHub Issues and pull requests when the owner provides or
  creates an issue. No external tracker or Zazz Board integration is declared.
- Shared-file coordination: no external locking tool is declared. Use the active
  agent harness for coordination and serialize overlapping-file edits when safe
  isolation is not available.

## Branch and Checkout Policy

Read `.zazz/docs/zgraphify-agent-orientation.md` before changing branches,
synchronizing upstream, or creating a pull request.

- Current operating model: one regular Git checkout and ordinary Git branches.
  Do not create, convert, remove, or assume worktrees until the owner approves the
  planned migration.
- `main` is this fork's integration branch and normal pull-request target.
- `upstream/v8` is the read-only active upstream development line; synchronize
  upstream changes from it into `main` through a documented reviewable change.
- `v8` is this fork's baseline snapshot. Do not put fork-specific work or `.zazz/`
  on it.
- Fork feature branches start from `main`, use concise flat hyphenated names without
  `/`, and target `main` in their pull requests. The initial feature branch is
  `ladybug-integration-mvp`.
- The `upstream` remote is fetch-only. Push fork work only to `origin`.
- A potential upstream contribution starts from `upstream/v8` in a separate branch;
  it MUST NOT include `.zazz/` or unrelated fork work and targets upstream `v8`.

## Agent Execution Discipline

Default discipline:

- `.zazz/docs/agent-execution-discipline.md`

Repo-specific overrides:

- Integration branch: `main`.
- Execution records: local ignored files under `.zazz/ephemeral/` unless the owner
  requests a committed artifact.
- Stack: Python with `uv`; project code lives primarily under `graphify/`, tests
  under `tests/`, and development tooling under `tools/`.
- Known health exception: macOS case-insensitive filesystems cannot run the two
  Fortran fixture variants simultaneously; use Linux or Docker when both variants
  must be tested.

## Development and Contribution Notes

- Use conventional commit subjects such as `fix: description`, `feat: description`,
  or `docs: description`.
- Before opening a PR, run `uv run pytest tests/ -q` and record the result.
- For a language extractor, add a fixture under `tests/fixtures/` and coverage in
  `tests/test_languages.py`.
- Useful upstream contributions include worked examples under `worked/<slug>/` with
  an honest `review.md`, and extraction-bug reports with the input file and relevant
  `graphify-out/cache/` entry.

## Quick Links

- Agent orientation: `.zazz/docs/zgraphify-agent-orientation.md`
- Zazz methodology: `.zazz/zazz-methodology.md`
- Agent execution discipline: `.zazz/docs/agent-execution-discipline.md`
- Standards index: `.zazz/standards/index.yaml`
- Feature index: `.zazz/features/index.yaml`
- Local execution surface: `.zazz/ephemeral/`
