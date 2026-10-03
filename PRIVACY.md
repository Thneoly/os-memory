# Privacy Policy — Memory

`Memory` is the **pinned view** in a Notes + Digest + Memory trio. Every pin
you add is sent straight to your device's assistant through the
`octos.turn.start` host service, and the list shown back to you is read
from the same assistant's `octos.session.history`. The device itself
keeps no copy of the pins.

## Data this app stores

None. No file is written under `.local-state/os-memory/` for the pins
themselves; the app does not call the `storage` capability.

## Data this app requests

- `octos.turn.start` host service — invoked on three explicit user
  actions, each sending exactly one short string to the assistant:
  - **Pin**: `Pin: <text>`
  - **Unpin** (per row): `Forget this pin: <text>`
  - **Export**: `Summarize what you remember as pinned entries, one bullet per pin.`
- `octos.session.history` host service — invoked when the screen opens
  and after every Pin / Unpin. The response is filtered locally for
  messages whose role is `user` and whose text starts with `Pin: `. The
  remaining text becomes one row. Nothing else from the conversation is
  read, rendered, stored, or forwarded.

## What this app does NOT do

- It does not request `network.hosts` and does not open any HTTP/HTTPS
  connection.
- It does not request the `storage` capability and does not write any
  local file.
- It does not collect telemetry, analytics, or device identifiers.
- It does not auto-pin: every `octos.turn.start` call is gated by an
  explicit button tap.
- It does not collect passwords, PINs, codes, or any account field.

## Agent profile

`read-only` — the app never asks the host to write, sign, or publish.

## On devices without an assistant

If the host has no assistant service (today: every OctoSense shell,
every `card-host`), the list shows **Assistant unavailable** and the
Pin / Unpin / Export buttons do nothing. There is no local cache to fall
back to.

## System-app note

This bundle declares id `os.memory` and is intended as a system app. On
Rinx it must be added to the built-in `system-apps.json` catalog; the
runtime import screen refuses `os.*` IDs by design.

## Contact

https://github.com/Thneoly/os-memory/issues

## Source

https://github.com/Thneoly/os-memory
