# Privacy Policy — os.memory

`os.memory` is a **system app** that ships with the device and acts as a
cross-app memory hub. This release is local-first and offline-safe.

## Data this app stores

- `memories.json` in `.local-state/os.memory/` — the user's pinned and
  focused memory entries plus an `export.txt` snapshot.
- All storage stays on-device in the system app jail allocated by
  `card-host --system` (64 MiB cap, governed by `HostLimits::system()`).

## Data this app requests

- `storage` capability — used only for the on-device jail above.
- Four `octos.*` host services (read-only host APIs provided by the device
  shell): `octos.session.open`, `octos.session.history`,
  `octos.turn.start`, `octos.turn.interrupt`. These are used to read the
  assistant lane that lane over other apps' own agents; they do not transmit
  user data off-device.

## What this app does NOT do

- It does not request `network.hosts` and does not open any HTTP/HTTPS
  connection.
- It does not collect, transmit, or store any data off-device.
- It does not declare a personal-data category; the on-device jail is the
  sole destination for any memory entry.

## Agent profile

`read-only` — the app never asks the host to write, sign, or publish.

## Contact

https://github.com/Thneoly/os-memory/issues

## Source

https://github.com/Thneoly/os-memory