# Center Seat Console

Center Seat Console is a fork of Zed, distributed under GPL-3.0-or-later. GPUI remains under its own
Apache-2.0 license. Zed's own `README.md` is left as upstream wrote it, so it never conflicts on sync.

## Pin

Upstream Zed **v1.22.0**, commit `76659a55a8c10ed355a070f8764a0b1733e3c115`. `UPSTREAM_VERSION` holds the
current tag.

## Branches

- `main` mirrors Zed's `main`. It is updated only by GitHub's "Sync fork" and is never committed to.
- `centerseat` is the default, product branch. It was cut from the pin above, and all Center Seat work lands
  on it or on a branch off it.
- Upstream updates are merges of stable Zed tags into `centerseat`, never rebases. Each arrives as a
  `sync/<tag>` branch for a human to review and merge.

## Upstream edits

Center Seat code lives in `crates/centerseat_*`. Every edit to a file that exists upstream is listed in
`docs/upstream-patches.md` with its reason and a replayable diff.

## Governance

This fork is governed by `../Project-Dry-Dock/constitution.md` and the specs beside it. ADR 0011 records the
fork strategy, and Spec 002 the fork and shell swap.
