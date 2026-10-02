# Memory (os.memory) — brief

The shared-memory hub: one system app where the person sees, manages and
exports everything the device remembers across apps — and where apps'
agents sync their knowledge to. Works fully without AI; AI parts degrade
gracefully.

## Architecture decision this brief encodes

Two kinds of memory, two stores, both shown in one app:

1. **Pinned memory (the base, no AI needed)** — entries the person added
   or chose to keep, stored in the app's own jail (`memories.json`):
   `{id, source, text, created}`. Source is `"self"` or a syncing app's
   id. Survives everything, exportable, the gate-visible core.
2. **The agent's memory (AI, degradable)** — what this app's own agent
   has been told through the system agent (`octos.session.history`).
   Conversational, fuzzy, shown as a separate lane; the person can pin
   any entry into the pinned store with one tap.

## Screens

1. **Main** — header with app title and an "assistant: on/off/unavailable"
   chip; two sections:
   - **Pinned** (the list from `memories.json`): grouped by source
     ("Mine", "From memory-notes", …), each row is the text with a date;
     tap to open a row menu (pin/unpin n/a here, delete); "Add" entry +
     button at the top like the template.
   - **From the assistant** (only when `octos.session.history` answers):
     the agent's recent memory lines, each with a "Pin" button that
     copies it into the pinned store; when the service is unavailable the
     section shows the honest one-liner ("No assistant on this device —
     pinned memory still works").
2. **Export** — a button that writes `export.txt` (all pinned entries,
   one per line with source and date) into the jail and confirms.

## Actions

- Add a pinned entry (text input + Add; empty ignored).
- Delete a pinned entry (tap).
- Pin an agent-memory line into the pinned store (button; base store
  only grows by user action).
- Export pinned memory to `export.txt`.
- Restart persistence for pinned memory.

## Data

- `memories.json` in the jail: array of `{id, source, text, created}`
  (id: increasing integer as string; created: `time_now()` seconds).
- `export.txt` written on demand.

## States

- **Empty**: "Nothing pinned yet. Add something, or pin a line from the
  assistant below."
- **No assistant** (card-host and any host without `octos.*`): pinned
  memory fully usable; assistant section shows the honest one-liner.
- **Restart**: pinned entries survive; the assistant section re-reads
  whatever the peer still holds.
- **Corrupt json**: fall back to empty (documented `parse_json` caveat:
  check fields before use).

## Hosts

None. No network requests.

## Capabilities and why

- `storage` — `memories.json` and `export.txt` in the jail.
- `octos.session.open`, `octos.turn.start`, `octos.session.history`,
  `octos.turn.interrupt` — talk to this app's own agent only (read its
  memory lane; never another app's).
- `agent` block (`"tools": ["ask_user_question"]`) — declares the app's
  agent so the system agent can sync other apps' knowledge into it; the
  app itself is the surface where the person allows and reviews that.
- `glance` — **deferred to a later version** (runtime pin predates
  `sys.chat`); not requested in 0.1.0 to keep every granted capability
  map to something a screen does today.

## Not in this app (by architecture)

- Reading any other app's jail or agent memory directly (impossible by
  isolation; syncing happens via the system agent, user-approved).
- Any new `memory.*` host service (native code; out of the race's rules).
