# Center Seat Console agents

Before anything else, read these, in order. They outrank everything else in this repository:

1. `../Project-Dry-Dock/constitution.md`
2. `../Project-Dry-Dock/docs/adr/0010-dry-dock-process-boundary.md`
3. `../Project-Dry-Dock/docs/adr/0011-fork-strategy-and-upstream-sync.md`
4. The active spec: `../Project-Dry-Dock/specs/002-fork-shell-swap/spec.md` and its `tasks.md`

Zed's own `CLAUDE.md` and `AGENTS.md` at the repository root describe upstream conventions only. Follow
them for code style in upstream crates, but never over the documents above.

- Never commit to `main`. It mirrors upstream Zed. Work goes on `centerseat` or a branch off it.
- Every edit to a file that exists upstream goes in `docs/upstream-patches.md` with its reason and a
  replayable diff.
- No crate in this fork depends on a `drydock_*` crate (constitution Article V.7).
