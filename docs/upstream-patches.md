# Upstream patch ledger

Every edit Center Seat Console makes to a file that exists in upstream Zed is listed here, with the reason
and a replayable diff (constitution Article V.5, ADR 0011). An upstream edit that is not listed here is a
constitution violation.

Files that do not exist upstream are not listed: `crates/centerseat_*`, `UPSTREAM_VERSION`,
`CENTERSEAT.md`, `.claude/CLAUDE.md` and this file.

## Pin

- Upstream: `zed-industries/zed`
- Tag: `v1.22.0`
- Commit: `76659a55a8c10ed355a070f8764a0b1733e3c115`
- Branch: `centerseat`, cut from the commit above

## Patches

None yet.

Each patch is a row in this form:

### `<path>`

- Reason: why the edit cannot live in a `crates/centerseat_*` crate.
- Task: the Spec task that made it.

```diff
<output of git diff <tag> centerseat -- <path>>
```
