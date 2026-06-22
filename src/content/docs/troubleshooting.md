---
title: Troubleshooting
description: Fixes for the most common freemkv and autorip problems — no drives detected, missing keydb, bad sectors, interrupted rips, and capturing a debug log.
---

Fixes for the problems reported most often with freemkv and autorip. Find your symptom below; if none match, capture a debug log (next) and open an issue.

## Capturing a debug log

For any failure or hang, capture a debug log first — it's the fastest path to a diagnosis and the one thing to attach to a bug report.

On the CLI, re-run with debug logging to a file:

```bash
freemkv --log-level 3 --log-file freemkv-debug.log <source> <dest>
```

In autorip, enable the Debug toggle in the web UI (or `POST /api/debug`), reproduce the problem, then collect the container logs.

## No drives detected in autorip

The container is missing `privileged: true`. Without it the container starts normally but enumerates zero drives, and the UI shows "No drives detected" with no other error.

```yaml
services:
  autorip:
    # required for optical SCSI access
    privileged: true
    volumes:
      # required: exposes host devices to the container
      - /dev:/dev
```

Confirm both `privileged: true` and the `/dev:/dev` bind mount are present, then restart the container. See [Deploy](/autorip/#deploy).

## No drive found on the CLI

If `freemkv info disc://` reports no drive:

- On Linux, target the SCSI generic device explicitly — use `/dev/sg*`, not `/dev/sr*`:

  ```bash
  freemkv info disc:///dev/sg4
  ```

- List candidate devices with `-d` / `--device` and pick the correct one.
- Confirm your user has permission to access the device node.

## Missing keydb

You tried to read an AACS-encrypted disc (Blu-ray or 4K UHD) without keys available. The CLI reports "no KEYDB.cfg found"; DVDs are never affected.

Blu-ray and 4K UHD need decryption keys you provide. **[Decryption Keys](/decryption-keys/)** covers the ways to supply them (for the CLI and for autorip).

## Bad sectors on a disc

freemkv is built to recover damaged discs, but behavior depends on the mode:

- **Single-pass** (CLI direct disc → MKV, or autorip `max_retries = 0`) — no retries; a read error fails the rip.
- **Multipass** (CLI `--multipass`, or autorip `max_retries ≥ 1`) — the sweep records bad ranges and skips ahead; patch passes then re-read only those ranges from the disc. Use this mode for any disc you suspect is scratched. See [How recovery works](/how-recovery-works/).

Tips:

- Run `freemkv verify disc://` for a read-only health check. It reports good / slow / recovered / bad sectors and exits non-zero on anything truly unrecoverable.
- In autorip, set `abort_on_lost_secs` above `0` to tolerate a bounded amount of main-movie loss rather than failing on a disc that can't be read perfectly.

:::caution[Don't keep retrying a dying disc]
Repeatedly hammering the same bad sectors can push a drive into a fast-fail state where it stops attempting recovery entirely. If recovery stalls, let the drive cool down or eject and reload before trying again — don't launch pass after pass back-to-back.
:::

## An interrupted rip

- **CLI** — Ctrl-C halts cleanly and preserves the mapfile. Re-running the same `disc:// iso://` command resumes. A mux interrupted mid-write is not finalized (freemkv exits non-zero), so you never get a truncated MKV that looks complete.
- **autorip** — `/api/stop/{device}` preserves staging; the rip resumes on the next disc insert or container restart. See [Resume](/autorip/#resume).

## Raw flag rejected for MKV or M2TS output

`--raw` writes undecrypted bytes, which is only meaningful for an `iso://` destination. freemkv rejects `--raw` with an `mkv://` or `m2ts://` destination because ciphertext can't be muxed. Drop `--raw`, or change the destination to an ISO.

## Multiple titles to one file

Selecting more than one title (e.g. `-t 1 -t 3`) requires a directory destination — freemkv writes one file per title. Point the destination at a directory instead of a single file:

```bash
# directory destination, one file per title
freemkv disc:// mkv://out/ -t 1 -t 3
```

## Checking versions

```bash
# CLI
freemkv version
```

```bash
# autorip
curl -s http://<host>:8080/api/version
```
