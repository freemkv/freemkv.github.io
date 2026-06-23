---
title: CLI Reference
description: Every freemkv subcommand, flag, and stream URL — the complete command-line reference.
---

The freemkv command-line tool drives a rip, remux, or disc inspection from a single
invocation. It is a thin front end over [libfreemkv](/components/) and takes two shapes:

```bash
# copy / rip / remux — source URL to dest URL
freemkv <source> <dest> [flags]

# info, verify, update-keys, version, help
freemkv <subcommand> [args] [flags]
```

Everything is addressed with **stream URLs** (`scheme://path`). A bare invocation prints
usage and exits non-zero; `freemkv help` exits zero.

## Stream URLs

Every source and destination is a URL with a `scheme://` prefix. The schemes the CLI
understands:

| URL | Meaning |
|---|---|
| `disc://` | Optical drive, auto-detected |
| `disc:///dev/sg4` | A specific optical drive (Linux — use `/dev/sg*`) |
| `disc://D:` | A specific optical drive (Windows) |
| `iso://image.iso` | Disc image file (read or write) |
| `mkv://path.mkv` | Matroska file |
| `m2ts://path.m2ts` | Blu-ray transport-stream file |
| `network://host:port` | TCP stream (sender or receiver) |
| `stdio://` | Stdin / stdout pipe |
| `null://` | Discard output (read-speed benchmarking) |

File paths follow the scheme directly: `mkv://./Movie.mkv`, `m2ts://./Movie.m2ts`,
`iso://Disc.iso`. A destination without a scheme is rejected — add one.

## Ripping and remuxing

When the first argument isn't a known subcommand, freemkv treats the positional URLs as a
source and a destination and runs the appropriate pipeline. The combination of schemes
determines what happens:

```bash
# rip disc → MKV (decrypt + mux)
freemkv disc:// mkv://Movie.mkv

# rip disc → m2ts
freemkv disc:// m2ts://Movie.m2ts

# rip a specific drive
freemkv disc:///dev/sg4 mkv://Movie.mkv

# full-disc raw sector copy → ISO (auto-resumes)
freemkv disc:// iso://Disc.iso

# remux an ISO → MKV
freemkv iso://Disc.iso mkv://Movie.mkv

# remux m2ts → MKV
freemkv m2ts://Movie.m2ts mkv://Movie.mkv

# stream a disc to a network receiver
freemkv disc:// network://192.0.2.10:9000

# receive a network stream → MKV
freemkv network://0.0.0.0:9000 mkv://Movie.mkv

# pipe to stdout
freemkv disc:// stdio://

# benchmark read speed (no output)
freemkv disc:// null://
```

A `disc://` → `iso://` (or `null://`) pair performs a **raw sector copy**
(`Disc::copy`), not a stream mux. Every other combination runs the title pipeline:
read → PES frames → write the destination container.

### Selecting titles

By default all titles are ripped. Use `-t` / `--title` (1-based) to select one or more.
Repeating the flag selects multiple titles; repeated values are de-duplicated:

```bash
# main feature only → one file
freemkv disc:// mkv://Movie.mkv -t 1

# titles 1 and 3 → directory dest; one MKV per title (Movie_t1.mkv, Movie_t3.mkv)
freemkv disc:// mkv://Movies/ -t 1 -t 3
```

Selecting more than one title writes **one separate file per title** (never a merged file),
so the destination must be a **directory** — freemkv names each `<name>_t<N>.<ext>`. A
single-title selection writes one file as named.

### Rip flags

These flags apply to the `<source> <dest>` form:

| Flag | Description |
|---|---|
| `-t, --title N` | Select title N (1-based, repeatable). Default: all titles. |
| `-k, --keydb PATH` | Path to a keydb.cfg (overrides the default location). |
| `--log-level N` | Write a diagnostic log file at this verbosity: 1 = warnings, 2 = info, 3 = debug, 4 = trace. Without it (and without `--log-file` / `RUST_LOG`) **no log file is written and the terminal stays clean** — diagnostics never print to the terminal. Default log path is `./log.txt`. |
| `--log-file PATH` | Write the diagnostic log to PATH (defaults to debug detail if `--log-level` is absent). For bug reports. |
| `-q, --quiet` | Suppress progress and informational output. |
| `--raw` | Skip decryption — raw encrypted bytes (`iso://` output only). |
| `--multipass` | Write/update a mapfile for multipass recovery (disc → ISO). |

### Multipass recovery with mapfiles

`--multipass` turns a disc → ISO copy into a recoverable, resumable sweep backed by a
mapfile (see **[How recovery works](/how-recovery-works/)**):

```bash
# Run once to sweep the disc to an ISO; re-run the SAME command to patch the
# remaining bad ranges from the disc. Repeat until the mapfile is clean.
freemkv disc:// iso://Disc.iso --multipass
```

It's a single command you re-run — it auto-detects the pass from the mapfile: the first run
sweeps the whole disc; each later run re-reads only the ranges still marked bad. A plain
`disc:// iso://Disc.iso` (without `--multipass`) also **auto-resumes** if interrupted.

By default the ISO is decrypted. Add `--raw` to write the disc's sectors **as-is**, without
decrypting — an exact encrypted image. It combines with the sweep:

```bash
freemkv disc:// iso://Disc.iso --raw --multipass
```

`--multipass` only applies to the disc → ISO path. A direct disc → MKV/M2TS mux is
single-pass; passing `--multipass` there prints a notice and is otherwise ignored.

### Interrupting a rip

A single Ctrl-C halts cleanly: the sweep stops, the drive tray is unlocked, and a partial
ISO's mapfile is preserved so a later run can resume. A mux interrupted mid-write is
**not** finalized — freemkv exits non-zero rather than presenting a truncated MKV as
complete. A second Ctrl-C forces an immediate exit.

## Inspecting a disc, image, or file

`freemkv info <url>` shows what's on a source: titles, durations, sizes, and stream
details (video / audio / subtitle). Listing titles from a disc or ISO needs **no AACS
key** — only the clear UDF navigation is read.

```bash
# disc in the auto-detected drive
freemkv info disc://

# a specific drive
freemkv info disc:///dev/sg4

# an ISO image (keyless title listing)
freemkv info iso://Disc.iso

# a transport-stream file
freemkv info m2ts://Movie.m2ts

# a Matroska file
freemkv info mkv://Movie.mkv
```

Running `freemkv <url>` with a single URL and no destination is equivalent to
`freemkv info <url>`.

`info` flags (for `disc://` / `iso://` sources):

| Flag | Description |
|---|---|
| `-d, --device PATH` | Target a specific device (alternative to embedding it in the URL). |
| `-f, --full` | Show every title (otherwise the first five, with a "+N more" footer). |
| `-b, --basic` | Title rows only — omit per-stream detail. |
| `--log-level N` | Write a diagnostic log file (1 = warnings … 4 = trace); without it nothing is logged. Level ≥ 2 also widens on-screen AACS / drive detail. |
| `-q, --quiet` | Suppress output. |

### Sharing a drive profile

`freemkv info disc:// --share` captures the drive's profile to a zip file on disk and
prints a ready-to-paste GitHub issue so you can contribute it to the community
drive-compatibility database. Submission is manual — it never sends anything over the
network. `--mask` (`-m`) masks serial numbers in the captured profile.

```bash
# capture this drive's profile and print a GitHub issue
freemkv info disc:// --share

# with serial numbers masked
freemkv info disc:// --share --mask
```

## Verifying disc health

`freemkv verify` scans the main title and reads it through, reporting good / slow /
recovered / bad sectors without writing any output. Works only with `disc://` URLs
(defaults to `disc://` if none is given):

```bash
# verify the disc in the auto-detected drive
freemkv verify

# verify a specific drive
freemkv verify disc:///dev/sg4
```

It prints a per-cluster breakdown (with the chapter and timestamp where damage falls) and
a final verdict. **Exit code is non-zero if any sector is unrecoverable (`bad > 0`)**,
making it scriptable as a pass/fail gate.

## Refreshing keydb.cfg

`freemkv update-keys` downloads an AACS key database, verifies it, and saves it to the
default location (`~/.config/freemkv/keydb.cfg`). The URL is **required** — there is no
built-in default:

```bash
freemkv update-keys --url http://example.org/keydb_eng.zip

freemkv update-keys -u http://example.org/keydb_eng.zip
```

The downloader handles plain text, `.zip`, and `.gz` payloads and follows redirects. See
**[Decryption Keys](/decryption-keys/)** for what needs keys and where they live.

## Version and help

```bash
# print the version
freemkv version

# print usage and exit 0
freemkv help
```

Both also accept `--version` / `-V` and `--help` / `-h`.

## Global options

| Flag | Description |
|---|---|
| `--language CODE` | UI language (e.g. `--language de`; alias `--lang`). Parsed first; never consumes a following URL or flag. |
| `--log-level N` | Write a diagnostic **log file** at verbosity 1 = warnings, 2 = info, 3 = debug, 4 = trace (applies to freemkv and libfreemkv). The terminal is always clean — diagnostics go to the file only, never to the terminal. Default file is `./log.txt`. |
| `--log-file PATH` | Write the diagnostic log to PATH (non-blocking; flushed on exit; defaults to debug detail if `--log-level` is absent). For bug reports. |
| `-q, --quiet` | Suppress normal stdout output. |
| `RUST_LOG` | Power-user override: if set, enables file logging and wins over `--log-level`. |

### Clean terminal, file-only diagnostics

The terminal shows only curated progress, status, and the final result block — **never** raw
trace/debug log lines. Diagnostic logging is a separate channel that only exists when you ask
for it, and it goes to a **file**, never to the terminal. With none of `--log-level`,
`--log-file`, or `RUST_LOG` set, no log file is written at all.

When an operation fails, freemkv prints a single readable block — what failed, the cause, and
how to turn on a diagnostic log:

```text
✗ rip failed: <plain-English cause>.
  For a diagnostic log, re-run with --log-level 3 (writes ./log.txt).
```

### Getting a debug log for bug reports

If something fails or hangs, re-run with `--log-level 3`. That writes a diagnostic log to
`./log.txt` (override the path with `--log-file`):

```bash
freemkv <source> <dest> --log-level 3              # writes ./log.txt
freemkv <source> <dest> --log-level 3 --log-file freemkv-debug.log
```

Level 3 (debug) is recommended for bug reports — comprehensive diagnostics at a
manageable size. The extra detail covers the failure path (CSS auth, retries, read
errors, mux-stage decisions, stalls) — exactly when you need it. If a maintainer asks for
maximum detail, use `--log-level 4` (trace), but note it's a per-sector firehose that can
be gigabytes on a long rip. Log files are written with timestamps on and terminal colour
codes off, so they paste cleanly into a bug report.

**Keys are never written to logs** — log output contains paths and disc metadata; CSS/AACS
key material is always redacted.

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Success (also `help` / `--help`). |
| `1` | Operation failed (rip/mux error, scan failure, bad flag value, out-of-range title) or `verify` found unrecoverable sectors. |
| `2` | Bare invocation with no subcommand or URL (usage printed). |
| `130` | Forced exit on a second Ctrl-C. |
