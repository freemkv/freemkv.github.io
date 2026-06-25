---
title: CLI Reference
description: Every freemkv subcommand, flag, and stream URL. The complete command-line reference.
---

The `freemkv` binary takes two forms:

```bash
freemkv <source> <dest> [flags]   # rip / remux
freemkv <subcommand> [args]       # info / verify / update-keys / version / help
```

A bare invocation prints usage and exits `2`. `freemkv help` exits `0`.

## Stream URLs

Every source and destination is a `scheme://` URL.

| URL | Source | Sink | Notes |
|---|---|---|---|
| `disc://` | ✓ | — | Auto-detected optical drive |
| `disc:///dev/sg4` | ✓ | — | Specific drive (Linux: `/dev/sg*`) |
| `disc://D:` | ✓ | — | Specific drive (Windows) |
| `iso://path.iso` | ✓ | ✓ | Disc image file |
| `mkv://path.mkv` | ✓ | ✓ | Matroska file |
| `m2ts://path.m2ts` | ✓ | ✓ | Blu-ray transport-stream file |
| `network://host:port` | ✓ | ✓ | TCP stream (listen or connect) |
| `stdio://` | ✓ | ✓ | Stdin / stdout |
| `null://` | — | ✓ | Discard (read-speed benchmark) |
| `dir://path/` | — | ✓ | **Planned** — decrypted file tree (VIDEO\_TS / BDMV) |

`disk://` is accepted as an alias for `disc://`. A destination without a recognized scheme is rejected. File paths follow the scheme directly: `mkv://./Movie.mkv`, `iso://Disc.iso`.

## Output formats

When a URL is the **destination**, the scheme determines what freemkv writes:

| Destination | Writes | Decrypted? | `--multipass` | `--raw` |
|---|---|---|---|---|
| `iso://PATH` | Sector image | **Yes** (default) | ✓ | ✓ → encrypted |
| `mkv://PATH` | Muxed movie | **Yes** | Error ¹ | Error ² |
| `m2ts://PATH` | Transport stream | **Yes** | Error ¹ | Error ² |
| `null://` | Discard | — | Error ¹ | Error ² |
| `dir://PATH/` | File tree | **Yes** | Error ¹ | Error ² |

> ¹ **`--multipass` is `iso://`-only.** For a damaged disc: rip to ISO first (`disc:// iso:// --multipass`), then mux (`iso:// mkv://`).
> ² **`--raw` is `iso://`-only.** Keeps sectors encrypted — a faithful disc image. You can't mux or benchmark ciphertext.

**Everything is decrypted by default.** The only encrypted output is `iso://` with `--raw`. See [Decryption and keys](#decryption-and-keys).

:::note[dir:// is planned]
`dir://` is not yet available. It is listed here as a roadmap item.
:::

## Ripping and remuxing

```bash
# disc → MKV (decrypt + mux, main title)
freemkv disc:// mkv://Movie.mkv

# disc → M2TS (main title)
freemkv disc:// m2ts://Movie.m2ts

# specific drive
freemkv disc:///dev/sg4 mkv://Movie.mkv

# disc → decrypted ISO (auto-resumes if interrupted)
freemkv disc:// iso://Disc.iso

# disc → encrypted ISO (faithful disc image)
freemkv disc:// iso://Disc.iso --raw

# multi-pass recovery: sweep + patch bad sectors; re-run same command until clean
freemkv disc:// iso://Disc.iso --multipass

# ISO → MKV
freemkv iso://Disc.iso mkv://Movie.mkv

# M2TS → MKV
freemkv m2ts://Movie.m2ts mkv://Movie.mkv

# stream to network receiver
freemkv disc:// network://192.0.2.10:9000

# receive network stream → MKV
freemkv network://0.0.0.0:9000 mkv://Movie.mkv

# pipe to stdout
freemkv disc:// stdio://

# benchmark read speed (no output)
freemkv disc:// null://
```

`disc:// → iso://` runs `Disc::copy` (raw sector copy), not a stream mux. Every other source/dest combination runs the title pipeline: read → PES frames → write the destination container.

### Which output should I use?

| Goal | Command |
|---|---|
| Clean disc, one movie file | `freemkv disc:// mkv://Movie.mkv` |
| Damaged / marginal disc | `freemkv disc:// iso://Disc.iso --multipass` (repeat until clean), then `freemkv iso://Disc.iso mkv://Movie.mkv` |
| Decrypted backup image | `freemkv disc:// iso://Disc.iso` |
| Encrypted (faithful) backup | `freemkv disc:// iso://Disc.iso --raw` |
| Benchmark drive speed | `freemkv disc:// null://` |

### Selecting titles

- **`disc://`** rips the **main title** by default. `-t 1` is equivalent.
- **`iso://`** rips **all titles** by default; the destination must be a directory.

Use `-t` / `--title N` (1-based, repeatable) to pick titles explicitly:

```bash
freemkv disc:// -t 3 mkv://Extras.mkv           # title 3 → one file
freemkv disc:// -t 1 -t 3 mkv://Movies/          # titles 1 and 3 → directory
```

Multiple titles write one file each; the destination must be a directory. freemkv names each `<disc>_t<N>.<ext>`. Run `freemkv info disc://` to list all titles with durations before ripping.

### Rip flags

| Flag | Description |
|---|---|
| `-t, --title N` | Select title N (1-based, repeatable). Default: main title for `disc://`, all titles for `iso://`. |
| `-k, --keydb PATH` | Path to a `keydb.cfg` (overrides the default search locations). |
| `--key-url URL` | Enable the online key service at URL (`https://…`). When combined with `-k`, the local keydb is tried first. |
| `--key-auth TOKEN` | Bearer token for the key service (`--key-url`). |
| `--raw` | **`iso://` only.** Write encrypted sectors (skip decryption). Error with any other destination. |
| `--multipass` | **`iso://` only.** Multi-pass recovery with a mapfile sidecar for resume. Error with any other destination. |
| `-q, --quiet` | Suppress progress output. |
| `--log-level N` | Write a diagnostic log file: 1 = warnings, 2 = info, 3 = debug, 4 = trace. Default path: `./log.txt`. |
| `--log-file PATH` | Write the log to PATH (implies debug detail if `--log-level` is absent). |

### Multipass and interrupted rips

`--multipass` turns a `disc:// → iso://` copy into a resumable sweep: the first run sweeps the whole disc; each re-run patches only the ranges still marked bad. Re-run the same command until the mapfile is clean.

A plain `disc:// → iso://` (without `--multipass`) also **auto-resumes** if interrupted — the mapfile is written and picked up on the next run.

**The mapfile holds sector-recovery state only — no keys.** Keys are resolved fresh each run from your key sources.

`--raw` can be combined with `--multipass` for an encrypted multi-pass image:

```bash
freemkv disc:// iso://Disc.iso --raw --multipass
```

### Interrupting a rip

One Ctrl-C halts cleanly: the sweep stops, the drive tray is unlocked, and the mapfile is preserved so the next run resumes. A mux interrupted mid-write is not finalized — freemkv exits non-zero rather than presenting a truncated container as complete. A second Ctrl-C forces an immediate exit (code `130`).

## Decryption and keys

**freemkv decrypts everything by default.** The only encrypted output is `iso://` with `--raw`.

| Format | Encryption | Keys needed |
|---|---|---|
| DVD | CSS | None — keyless recovery built in |
| Blu-ray | AACS 1.0 | `keydb.cfg` or online key service |
| 4K UHD | AACS 2.0 / 2.1 | `keydb.cfg` or online key service |

DVDs decrypt automatically using built-in CSS recovery. Blu-ray and UHD require AACS keys — see **[Decryption Keys](/decryption-keys/)**.

**A missing key fails loudly and early** — a clear error, non-zero exit, and no output file written. freemkv never writes a silently-encrypted or partially-decrypted file.

Listing titles (`freemkv info disc://`) reads only UDF navigation and needs no key, even on AACS discs.

## Inspecting a disc, image, or file

`freemkv info <url>` lists titles, durations, sizes, and stream details. A single URL with no destination is equivalent to `freemkv info <url>`.

```bash
freemkv info disc://              # auto-detected drive
freemkv info disc:///dev/sg4      # specific drive
freemkv info iso://Disc.iso       # ISO (no AACS key needed for title listing)
freemkv info m2ts://Movie.m2ts
freemkv info mkv://Movie.mkv
```

`info` flags (for `disc://` / `iso://` sources):

| Flag | Description |
|---|---|
| `-d, --device PATH` | Target a specific device (alternative to embedding it in the URL). |
| `-f, --full` | Show all titles (otherwise the first five, with a "+N more" footer). |
| `-b, --basic` | Title rows only; omit per-stream detail. |
| `-v, --verbose` | Widen on-screen AACS / drive detail. |
| `-q, --quiet` | Suppress output. |

### Sharing a drive profile

`freemkv info disc:// --share` captures the drive's profile to a zip file and prints a ready-to-paste GitHub issue to submit to the community drive-compatibility database. Nothing is sent over the network automatically; submission is manual (or interactive with Y/n prompt if a token is compiled in).

```bash
freemkv info disc:// --share           # capture and print issue
freemkv info disc:// --share --mask    # mask serial numbers
```

## Verifying disc health

`freemkv verify` scans the main title and reports good / slow / recovered / bad sectors without writing any output. Defaults to `disc://` if no URL is given.

```bash
freemkv verify                    # auto-detected drive
freemkv verify disc:///dev/sg4    # specific drive
```

Output includes a per-cluster breakdown with chapter and timestamp for each damaged region, and a final verdict. **Exit code is `1` if any sector is unrecoverable (`bad > 0`)**, making it scriptable as a pass/fail gate.

## Refreshing your key database

`freemkv update-keys` downloads an AACS key database, verifies it, and saves it to the default location. `--url` is required; there is no built-in default.

```bash
freemkv update-keys --url http://example.org/keydb_eng.zip
freemkv update-keys -u http://example.org/keydb_eng.zip
```

The downloader handles plain text, `.zip`, and `.gz` payloads and follows redirects. See **[Decryption Keys](/decryption-keys/)** for where the file is saved.

## Global options

| Flag | Description |
|---|---|
| `--language CODE` | UI language (e.g. `--language de`; alias `--lang`). Parsed before everything else; never consumes a following URL or flag. |
| `--log-level N` | Write a diagnostic **log file** at verbosity 1 = warnings, 2 = info, 3 = debug, 4 = trace. Terminal stays clean — diagnostics go to the file only. Default file: `./log.txt`. |
| `--log-file PATH` | Write the log to PATH (implies debug detail if `--log-level` is absent). |
| `-q, --quiet` | Suppress normal stdout output. |
| `RUST_LOG` | Power-user filter override: if set, enables file logging and wins over `--log-level`. |

The terminal shows only curated progress and results — never raw trace lines. With none of `--log-level`, `--log-file`, or `RUST_LOG` set, no log file is written at all.

For bug reports, re-run with `--log-level 3`:

```bash
freemkv <source> <dest> --log-level 3              # writes ./log.txt
freemkv <source> <dest> --log-level 3 --log-file freemkv-debug.log
```

**Keys are never written to logs.** Log output contains paths and disc metadata; CSS/AACS key material is always redacted.

## Version and help

```bash
freemkv version    # print version
freemkv help       # print usage (exit 0)
```

Both also accept `--version` / `-V` and `--help` / `-h`.

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Success (also `help` / `--help`). |
| `1` | Operation failed (rip/mux error, scan failure, bad flag value, out-of-range title, missing key) or `verify` found unrecoverable sectors. |
| `2` | Bare invocation with no subcommand or URL (usage printed). |
| `130` | Forced exit on a second Ctrl-C. |
