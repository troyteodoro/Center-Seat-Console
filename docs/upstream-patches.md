# Upstream patch ledger

Every edit Center Seat Console makes to a file that exists in upstream Zed is listed here, with the reason
and a replayable diff (constitution Article V.5, ADR 0011). An upstream edit that is not listed here is a
constitution violation. `script/check-ledger` enforces this.

Files that do not exist upstream are not listed: `crates/centerseat_*`, `UPSTREAM_VERSION`,
`CENTERSEAT.md`, `.claude/CLAUDE.md`, `script/check-ledger` and this file.

## Pin

- Upstream: `zed-industries/zed`
- Tag: `v1.22.0`
- Commit: `76659a55a8c10ed355a070f8764a0b1733e3c115`
- Branch: `centerseat`, cut from the commit above

## Row format

Each row under Patches is a `###` heading naming the path in backticks, then the reason, the task, and a
`diff` block holding the output of `git diff <tag> centerseat -- <path>`:

~~~markdown
### `crates/workspace/src/workspace.rs`

- Reason: why the edit cannot live in a `crates/centerseat_*` crate.
- Task: the spec task that made it.

```diff
<output of git diff <tag> centerseat -- crates/workspace/src/workspace.rs>
```
~~~

## Patches

None yet.
