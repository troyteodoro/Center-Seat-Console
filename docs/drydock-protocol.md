# Dry Dock protocol v1

Status: **frozen**, 2026-10-04. Any change to a message's shape is a new `protocol_version`.

Center Seat Console draws the ship; a separate program, `drydock`, decides where everything goes. They
share no code. They speak this protocol over the child process's stdin and stdout, the way an editor speaks
LSP to a language server. This document, the JSON Schema and the golden messages beside it are the whole
contract. Each side writes its own types from them and tests against the same goldens.

| File | What it is |
|---|---|
| `docs/drydock-protocol.md` | This description |
| `tests/fixtures/drydock/protocol-v1.schema.json` | JSON Schema (draft 2020-12) for one message |
| `tests/fixtures/drydock/f1-ship.jsonl` | A `ship` for the F1 sample repository |
| `tests/fixtures/drydock/f1-transition.jsonl` | A `transition` for an edit to F1 |
| `tests/fixtures/drydock/messages.jsonl` | One of each remaining message |

**The goldens are contract examples, not layout claims.** Their geometry is hand-built to exercise every
field, including one collision. Goldens produced by a real `drydock` replace them later, under the same
schema.

## Transport

- The editor spawns `drydock serve` and writes to its stdin. `drydock` writes to its stdout.
- **JSON Lines.** Each message is one compact JSON object in UTF-8 on one line, ending in `\n`. A message
  never contains a raw newline.
- Every message has a `type`. Objects are closed: a key the schema does not name is an error.
- `drydock` writes diagnostics to stderr only, never to stdout.

## Session

```
editor                          drydock
  initialize        ─────────▶
                    ◀─────────  ship
  files_changed     ─────────▶
                    ◀─────────  transition
  room_state        ─────────▶                (no reply)
  shutdown          ─────────▶                drydock exits
```

- `initialize` is the first message, and `ship` answers it.
- Each `files_changed` is answered by one `transition`, in order.
- `room_state` has no reply and never moves geometry.
- After `shutdown`, or when its stdin closes, `drydock` exits cleanly.
- A message that cannot be accepted is answered with `error`, and the session continues. A crash is never
  the answer.

## Versioning

`initialize` and `ship` carry `protocol_version`, which is `1`. A side that receives a version it does not
speak refuses it with `error` code `unsupported_protocol_version`. The editor shows a notice and keeps
working without a ship.

## Geometry

- **Units.** Integer tiles in **unrotated world space**. `x` and `y` lie in the plane, and `z` is up.
  Rotation, projection, pan, zoom, painter's order, insets and gaps are the editor's. `drydock` never sends
  screen coordinates.
- **Boxes.** A `tile_box` is `{x0, y0, z0, x1, y1, z1}` and covers `[x0, x1) × [y0, y1) × [z0, z1)`. It is
  never empty.
- **Plates.** A plate is a floor slab. It is identified by `(section, deck)`: `section` is a directory
  relative to the repository root, and `deck` is the plate's index within that section. `layer` is its
  vertical layer. Plates spilled beside one another share a layer.
- **Rooms.** Each room is one file, and **its path is its identity.** No other id crosses the boundary.
  - `path` is relative to the repository root and `/`-separated.
  - Clicking a room opens `path`, and a changed file is reported by its path.
- **Frames.** Hull frames are boxes drawn around the plates. They never move, clip or occlude a room.

### Collisions: one box per room, chained

A room has **one bounding box**. Most rooms fill their box. A room that does not, such as an L-shaped run of
tiles, also lists `holes`: the boxes inside its box that it does not own.

A room owns a tile when the tile is inside its `box` and inside none of its `holes`.

**At most one room owns any tile.** Two rooms' boxes may overlap ("collide"), but then every tile in the
overlap lies in a hole of one of them. A room that fills its box has no `holes` key, and it can never
collide.

To find the room under a tile:
1. Test boxes first. This is the fast path.
2. On a hit, if the tile is in one of that room's holes, continue to the next candidate.

This is a hash table with chaining: the box is the bucket, and the holes resolve the rare collision.

`holes` are sorted, pairwise disjoint, inside the room's box, and non-empty. The key is left out rather than
sent as `[]`.

### Order

- Plates are sorted by `(section, deck)`.
- Rooms, and the entries of a `transition`, are sorted by `path`.
- Identical input gives byte-identical messages.

## Messages

| `type` | Direction | Fields |
|---|---|---|
| `initialize` | editor → drydock | `protocol_version` (1), `repo_path`: the repository's absolute path |
| `ship` | drydock → editor | `protocol_version` (1), `seed`: 16 lowercase hex digits, `plates`, `rooms`, `frames` |
| `files_changed` | editor → drydock | `paths`: the files that changed, relative to the repository root |
| `transition` | drydock → editor | `rooms`: the rooms the edit moved, each `{path, before, after}` |
| `room_state` | editor → drydock | `path`, `state: {git, diagnostics: {errors, warnings}, tests}` |
| `shutdown` | editor → drydock | none |
| `error` | drydock → editor | `code`: `unsupported_protocol_version` or `malformed_message`; `message`, for a human |

- **Plate:** `{section, deck, layer, box}`.
- **Room:** `{path, plate: {section, deck}, box, holes?}`.
- **Frame:** `{hull, boxes}`. `hull` is one of `block`, `cruiser`, `saucer`, `ring_station` or
  `spine_and_pods`.
- **Transition entries:**
  - `before` and `after` are each a placement, `{plate, box, holes?}`, or `null`.
  - `before` is `null` for a new room, and `after` is `null` for a removed one. Never both.
  - Rooms an edit did not move are not listed.
- **`room_state` values:**
  - `git` is one of `clean`, `modified`, `added`, `deleted`, `renamed`, `untracked`, `ignored` or
    `conflicted`.
  - `tests` is one of `unknown`, `passing` or `failing`.
  - The editor paints from its own data. It sends this so `drydock` may use it at the paint tier only.

The schema is authoritative wherever this table and the schema differ.

## Changing the contract

v1 is frozen. A change to any message's shape:
- takes a new `protocol_version`;
- regenerates the schema and the goldens from Dry Dock;
- updates this document in the same change.

Both sides' tests parse the same goldens, so a drift fails on whichever side has not caught up.
