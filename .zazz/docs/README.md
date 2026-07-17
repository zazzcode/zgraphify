# Zazz Reference Guides

This directory contains general Zazz reference material. It is not a second
documentation root and it does not contain active project artifacts.

Project-specific documents live once in their corresponding directories directly
under [`.zazz/`](../): [standards](../standards/),
[proposals](../proposals/), [specifications](../specifications/),
[features](../features/), and [architecture](../architecture/).

The `methodology/` directory is vendor-managed general reference material used by
the canonical root-level [Zazz methodology entry point](../zazz-methodology.md).
Do not edit or reorganize it locally; refresh it only by pulling from
`zazz-skills`.

The standards library and proposal templates are intentionally not copied here.
The authoritative standards library is [`.zazz/standards/`](../standards/), and
project proposals belong in [`.zazz/proposals/`](../proposals/).

## Top-Level Guides

- [zgraphify-agent-orientation.md](zgraphify-agent-orientation.md) — first-read
  branch, checkout, and upstream policy for this fork
- [agent-execution-discipline.md](agent-execution-discipline.md) — agent
  execution discipline
- [code-review-graph.md](code-review-graph.md) — code review graph guidance
- [human-in-loop-pr-review-strategy.md](human-in-loop-pr-review-strategy.md) —
  human review and merge strategy
- [using-gh-stack.md](using-gh-stack.md) — stacked pull-request guidance
- [worktree-setup.md](worktree-setup.md) and [wt-cheat-sheet.md](wt-cheat-sheet.md)
  — reference material only; this fork does not use worktrees yet

## Sync Boundary

Pull vendor-managed material from `zazz-skills`; do not locally rewrite it.
Local project documentation belongs under the root-level `.zazz/` directories.

