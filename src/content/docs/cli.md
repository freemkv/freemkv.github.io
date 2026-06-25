---
title: CLI Reference
description: Every freemkv subcommand, flag, and stream URL.
---

The `freemkv` binary has two forms:

```bash
# Rip / remux: a source and a destination. No command word — the action is
# implied by passing two URLs.
freemkv <source> <destination> [flags]

# Subcommand: the first argument is the command.
freemkv <subcommand> [args]              # info, verify, update-keys, version, help
```

A bare invocation prints usage and exits `2`.

## Stream URLs

Every source and destination is a `scheme://` URL.

| URL | Source | Dest | Notes |
|---|---|---|---|
| `disc://` | ✓ | — | Optical drive (auto-detected; `disc:///dev/sg4` or `disc://D:` to target one) |
| `iso://path.iso` | ✓ | ✓ | Disc image |
| `mkv://path.mkv` | ✓ | ✓ | Matroska movie |
| `m2ts://path.m2ts` | ✓ | ✓ | Blu-ray transport stream |
| `network://host:port` | ✓ | ✓ | TCP (listen or connect) |
| `stdio://` | ✓ | ✓ | Stdin / stdout |
| `null://` | — | ✓ | Discard (read-speed benchmark) |
| `dir://path/` | — | ✓ | **Planned** — decrypted file tree (VIDEO\_TS / BDMV) |

`disk://` is an alias for `disc://`. Everything is **decrypted by default**; `--raw` (`iso://` only) is the sole encrypted output. BD/UHD discs need an AACS key — see [Decryption Keys](/decryption-keys/); DVDs need none.

## Scheme details

Most schemes are self-explanatory; two have behavior worth calling out.

### `disc://`

Rips the **main title** by default (`-t 1`). Pick others with `-t N` (repeatable).

### `iso://`

As a **source**, rips **all titles** by default — the destination must be a directory. As a **destination**, writes a decrypted sector image, plus two flags that work **only with `iso://`**:

- **`--multipass`** — sweep, then retry the bad sectors, with a resumable **mapfile** sidecar (sector state only — never keys). Re-run until clean. Damaged-disc workflow: `disc:// iso:// --multipass`, then `iso:// mkv://`.
- **`--raw`** — write the sectors **encrypted**, a faithful image. You can't mux or benchmark ciphertext, so both flags error on any other destination.

A plain `disc:// iso://` auto-resumes if interrupted.

### `dir://` *(planned)*

Extracts the decrypted on-disc file tree (`VIDEO_TS/` or `BDMV/`) straight into the folder. Not yet available.

## Examples

```bash
freemkv disc:// mkv://Movie.mkv             # disc → MKV (main title)
freemkv disc:// -t 1 -t 3 mkv://Movies/     # specific titles → directory
freemkv iso://Disc.iso mkv://Movie.mkv      # ISO → MKV
freemkv disc:// iso://Disc.iso              # disc → decrypted ISO
freemkv disc:// iso://Disc.iso --multipass  # recover a damaged disc (re-run until clean)
freemkv disc:// network://10.0.0.5:9000     # stream to a receiver
freemkv disc:// null://                     # benchmark read speed
```

Multiple titles write one file each (`<disc>_t<N>.<ext>`), so the destination must be a directory. Run `freemkv info disc://` first to list titles.

## Subcommands

### `info` — inspect a disc, image, or file

Lists titles, durations, sizes, and stream details. A lone URL with no destination is the same as `freemkv info <url>`. Needs no key, even on AACS discs.

```bash
freemkv info disc://
freemkv info iso://Disc.iso
```

| Flag | Description |
|---|---|
| `-d, --device PATH` | Target a specific device. |
| `-f, --full` | All titles (default: first five + "+N more"). |
| `-b, --basic` | Title rows only, no per-stream detail. |
| `-v, --verbose` | Wider AACS / drive detail. |
| `--share` | Capture the drive profile to a zip and print a ready-to-paste compatibility issue (`--mask` hides serials). Nothing is sent automatically. |

### `verify` — check disc health

Scans the main title and reports good / slow / recovered / bad sectors, with chapter + timestamp per damaged region. Writes nothing. **Exits `1` if any sector is unrecoverable** — scriptable as a pass/fail gate. Defaults to `disc://`.

```bash
freemkv verify
```

### `update-keys` — refresh the AACS key database

Downloads, verifies, and installs an AACS keydb (`.txt` / `.zip` / `.gz`). `--url` is required.

```bash
freemkv update-keys --url http://example.org/keydb_eng.zip
```

### `version` / `help`

```bash
freemkv version    # also --version / -V
freemkv help       # also --help / -h (exit 0)
```

## Flags

Rip / remux flags (the `<source> <destination>` form):

| Flag | Description |
|---|---|
| `-t, --title N` | Select title N (1-based, repeatable). Default: main for `disc://`, all for `iso://`. |
| `-k, --keydb PATH` | Path to a `keydb.cfg`. |
| `--key-url URL` | Online key service (`https://…`); local keydb tried first if both given. |
| `--key-auth TOKEN` | Bearer token for `--key-url`. |
| `--raw` | `iso://` only — write encrypted sectors. |
| `--multipass` | `iso://` only — multi-pass recovery with a resumable mapfile. |

Global (any invocation):

| Flag | Description |
|---|---|
| `--language CODE` | UI language (alias `--lang`). |
| `--log-level N` | Diagnostic **log file**, 1 = warn … 4 = trace (terminal stays clean; default `./log.txt`). For bug reports use `--log-level 3`. |
| `--log-file PATH` | Write the log to PATH. |
| `-q, --quiet` | Suppress stdout. |
| `RUST_LOG` | Power-user filter; enables file logging and wins over `--log-level`. |

Keys are never written to logs. One Ctrl-C halts a rip cleanly (tray unlocked, mapfile preserved); a second forces exit `130`.

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Success. |
| `1` | Failed (rip / mux / scan error, bad flag, missing key) or `verify` found unrecoverable sectors. |
| `2` | No subcommand or URL (usage printed). |
| `130` | Second Ctrl-C. |
