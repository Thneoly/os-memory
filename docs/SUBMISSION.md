# OctoSense Hackathon Submission — 3-App Packet

> Copy-paste blocks ready for the App Hub issue form (per
> [`docs/PUBLISHING.md` § 7](https://github.com/OctoSense-org/OctoSense-App-Hub/blob/main/docs/PUBLISHING.md))
>
> All three bundles were checked with `hub check --publisher-key` on
> 2026-10-03, signed under one publisher identity.

---

## Block 1 — publisher identity

```text
publisher id:        TheOne
publisher public:    0f1ea5274fe7dccc24880f5fc1601fef671f29719b6a86ffdf3bd841a6e16785
algorithm:           ed25519 (64-byte signature, 128 hex chars)
private key location: outside repo, ~/.octosense/publisher.key
```

## Block 2 — bundle pointer (3 apps, one identity)

| app            | kind       | repo / commit                          | hub check                                        |
|----------------|------------|----------------------------------------|--------------------------------------------------|
| os.memory      | system app | https://github.com/Thneoly/os-memory @ `9c76708` | `os.memory 0.1.0 — PASSED` (64 MiB, --system-app) |
| memory-notes   | store app  | https://github.com/Thneoly/memory-notes @ `84f9d6a` | `memory-notes 0.1.0 — PASSED` (16 MiB) |
| memory-digest  | store app  | https://github.com/Thneoly/memory-digest @ `3080bff` | `memory-digest 0.1.0 — PASSED` (16 MiB) |

## Block 3 — `hub check` output per bundle

> Re-run on your side if you want to re-verify; commands:
>
> ```bash
> export APP_PUBLISHER_ID=TheOne
> export APP_PUBLISHER_PUBLIC_KEY=0f1ea5274fe7dccc24880f5fc1601fef671f29719b6a86ffdf3bd841a6e16785
> # os-memory (system app):
> hub check <os-memory-repo>/bundle --publisher-key "$APP_PUBLISHER_ID=$APP_PUBLISHER_PUBLIC_KEY" --system-app
> # notes / digest (store apps):
> hub check <repo>/bundle --publisher-key "$APP_PUBLISHER_ID=$APP_PUBLISHER_PUBLIC_KEY"
> ```

Last observed output (2026-10-03):

```text
$ hub check os-memory/bundle --publisher-key TheOne=0f1ea527... --system-app
os.memory 0.1.0 — PASSED
  grants: capabilities {"octos.session.history", "storage"}, hosts {}, storage 67108864 bytes, agent read-only

$ hub check memory-notes/bundle --publisher-key TheOne=0f1ea527...
memory-notes 0.1.0 — PASSED
  grants: capabilities {"octos.turn.start", "storage"}, hosts {}, storage 16777216 bytes, agent read-only

$ hub check memory-digest/bundle --publisher-key TheOne=0f1ea527...
memory-digest 0.1.0 — PASSED
  grants: capabilities {"octos.turn.start", "storage"}, hosts {}, storage 16777216 bytes, agent read-only
```

## Block 4 — answers to `hub scan` questions

| app            | packet (review.json)                              | answers                                                |
|----------------|---------------------------------------------------|--------------------------------------------------------|
| os.memory      | `os-memory/build/review.json`                     | `os-memory/build/REVIEW-ANSWERS.md`                   |
| memory-notes   | `memory-notes/build/review.json`                  | `memory-notes/build/REVIEW-ANSWERS.md`                |
| memory-digest  | `memory-digest/build/review.json`                 | `memory-digest/build/REVIEW-ANSWERS.md`               |

---

## Honest disclosures (do not hide)

- **System-app submission path**: `hub publish` never accepts `os.*` ids — os.memory is admitted
  by digest into the device's built-in slot via `card-host --system`. The signed bundle is
  shipped with the build, not via the App Hub. The `hub check --system-app` output above is the
  gate-equivalent check for that bundle; the App Hub catalog entry is the two store apps.
- **Platforms**: all three `platforms: ["windows"]`. Other platforms are not claimed.
- **Manifest sign-off**: 2026-10-03 with key `TheOne`. No replacement possible; see
  `docs/PUBLISHING.md` § 5.
- **Capabilities** are trimmed to one screen's worth of use per app (no dead declarations).
- **F-21** (memory-notes ScrollYView repaint): closed 2026-10-03. Fix recorded in
  `memory-notes/bundle/main.splash` + `memory-notes/build/REVIEW-ANSWERS.md` § 7.

---

## Cross-app joint demo

The three apps form a shared-memory system: notes writes its own peer's memory through
`octos.turn.start`; digest reads its own peer's memory; os.memory aggregates across the
device. See `os-memory/docs/JOINT-DEMO.md` for the architecture and demo script.

---

*Generated 2026-10-03 · publisher TheOne · bundling os-memory + memory-notes + memory-digest.*
