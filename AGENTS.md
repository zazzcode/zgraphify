# Graphify + Zazz Contributor Guide

## Graphify Knowledge Graph

This project has a graphify knowledge graph at `graphify-out/`.

Rules:

- Before answering architecture or codebase questions, read `graphify-out/GRAPH_REPORT.md` for god nodes and community structure.
- If `graphify-out/wiki/index.md` exists, navigate it instead of reading raw files.
- After modifying code files in this session, run `graphify update .` to keep the graph current (AST-only, no API cost).

## Zazz Methodology

This repository uses Zazz for durable product and engineering context, bounded
delivery specifications, and selective standards loading.

- Methodology docs root: `.zazz`.
- Documentation operating model: GitHub-only committed Markdown. Durable project,
  architecture, feature, proposal, specification, and standards docs are tracked
  under `.zazz/`. Active run logs, QA notes, handoffs, evidence, and scratch work
  are local-only under `.zazz/ephemeral/`.
- Tracking system: no external tracker is declared. Include an issue or work-item
  link in PR context when the owner provides one.
- Shared-file coordination: no external locking tool is declared. Use the active
  agent harness for coordination and serialize work that overlaps the same files.

### Documentation and Standards

- Start methodology work with `.zazz/zazz-methodology.md`, then load only the
  focused guide in `.zazz/docs/methodology/` for the current stage.
- Before creating, modifying, reviewing, or validating code, read
  `.zazz/standards/index.yaml`; load only standards matching the affected paths
  and activities.
- Read `.zazz/features/index.yaml` when product behavior or feature context matters.
- The project overview is `.zazz/project.md`; architecture decisions and direction
  live under `.zazz/architecture/`.
- Deliverable specifications live in `.zazz/specifications/`; use
  `.zazz/ephemeral/` only for non-durable execution artifacts.
- The default execution discipline is
  `.zazz/docs/agent-execution-discipline.md`.

### Branch and Worktree Policy

- Read `.zazz/zgraphify-agent-orientation.md` before changing branches, syncing
  upstream, or creating a pull request.
- The current operating model uses a regular Git checkout and ordinary branches;
  do not create or convert to worktrees unless the owner explicitly authorizes the
  planned migration.
- `main` is this fork's integration branch. Keep it clean and merge reviewed work
  into it through pull requests.
- `v8` is the upstream baseline branch; do not treat it as this fork's integration
  target or put fork-specific work directly on it.
- Use concise, flat, hyphenated feature branch names without `/`. The initial
  feature branch is `lady-bug-integration-mvp`, based on `main`.

### Development and Contribution Notes

- Use conventional commit subjects such as `fix: description`, `feat: description`,
  or `docs: description`.
- Before opening a PR, run `uv run pytest tests/ -q` and record the result.
- For a language extractor, add an appropriate fixture under `tests/fixtures/` and
  coverage in `tests/test_languages.py`.
- Useful upstream contributions include worked examples under `worked/<slug>/` with
  an honest `review.md`, and extraction-bug reports with the input file and relevant
  `graphify-out/cache/` entry.
