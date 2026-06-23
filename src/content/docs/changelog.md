---
title: Changelog
description: Notable changes across the freemkv toolchain (CLI, library, and autorip service), newest first.
---

All notable changes across the freemkv toolchain — the `freemkv` CLI,
the `libfreemkv` core library, and the `autorip` service — are recorded
here, newest first. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.0.0/) and the project
follows semantic versioning.

The toolchain releases as a set: every component ships the same version
number on each release, even when a given component has no functional
change in that cycle.

## Unreleased — rc.4

In progress. Focused on a clean, predictable operator experience.

### Changed

- **Clean terminal by default.** The terminal now shows only curated
  progress, status, and a final result block — never raw diagnostic
  log lines. How you invoke the tools is unchanged.
- **Diagnostic logging is file-only.** Trace/debug output is written to
  a log file only when you ask for it (`--log-level` / `--log-file`),
  with timestamps on and terminal colour codes off. It never bleeds
  onto the terminal.
- **Overhauled error messages.** A fatal error now prints a single
  readable block — what failed, how to fix it, how to turn on a
  diagnostic log, and where that log is written — instead of a raw
  internal error. The first real cause is reported rather than a
  confusing downstream symptom.

### Fixed

- The reported failure is now the root cause (for example, an early
  key/authorization rejection), not the cascaded read error it used to
  surface further down the pipeline.

### Notes

- A faster, parallelized release pipeline is being folded in so future
  releases publish in minutes. No change to released artifacts.

## 1.0.0-rc.3.1 — 2026-06-22

Patch on rc.3. Closes silent-failure gaps so a rip can never report
success without actually producing output, and rounds out
cross-platform behavior.

### Added

- English rendering for every error code, so any failure prints a
  human-readable message rather than a bare code.
- Windows key-database path support (`%APPDATA%`) and an online key
  service option (`--key-url` / `--key-auth`) in the CLI.

### Fixed

- **Silent-failure guards.** Muxing now requires that real frames and
  bytes were produced before it reports "done" — an empty or
  zero-output result can no longer be mistaken for success, in both the
  CLI/library and the autorip service.
- Correct DVD/CSS handling for the no-key and per-title cases, with the
  scrambled-versus-unencrypted distinction made explicitly instead of
  inferred.

## 1.0.0-rc.3 — 2026-06-22

Third release candidate. Hardens the recovery path against
transport-level drive failures, fixes several Windows-specific issues,
and introduces a pluggable drive-unlock seam.

### Added

- A pluggable drive-unlock seam, so optional drive-side capabilities can
  be supplied without baking them into the core library.

### Changed

- The single-pass rip and the patch (Pass N) retry loop now abort
  cleanly on a transport-level failure (for example a USB bridge that
  drops off the bus) instead of looping on a dead device.
- Finer mux timeline resolution (0.1 ms timestamp scale) with
  video-only timeline epochs.

### Fixed

- Windows multi-drive selection, the `disc://` alias, and `READ`
  request chunking.
- Corrected a Windows storage-reset request code.
- Corrected AACS extent alignment in the patch path.

## 1.0.0-rc.2 — 2026-06-22

Second release candidate. Adds keyless DVD/CSS ripping and correct DVD
video output, on top of security and recovery hardening.

### Added

- **Keyless DVD/CSS ripping.** A CSS-protected DVD decrypts with no key
  database — the title key is recovered directly from the scrambled disc
  data and validated by descrambling a sector and confirming the known
  plaintext reappears, so a wrong key fails cleanly instead of producing
  silent garbage. A raw, still-scrambled CSS image (`iso://`) can be
  muxed without a pre-decryption step.
- **Correct DVD video** via an MPEG-2 Program-Stream access-unit
  reassembler: exactly one coded picture per MKV block, with
  presentation timestamps reconstructed from the stream, fixing
  previously corrupted DVD video. The reassembler is bounded so a
  malformed stream cannot exhaust memory.
- **Bare-run mode for autorip.** `autorip serve` runs the daemon
  directly with no container, storing config under `~/.config/autorip`
  — useful for the static binary on a bare Linux host.
- **Static-binary releases.** Each tagged release attaches a single
  static binary per platform (Linux x86_64/aarch64, macOS Intel/Silicon,
  Windows for the CLI) with a `.sha256` checksum.
- **Runtime debug toggle** in autorip (`POST /api/debug`) to switch the
  active log filter without a restart.
- A `.completed` restart guard so a container restart cannot re-mux a
  disc that already finished.
- CLI `--log-level` / `--log-file` for verbosity and a non-blocking file
  sink; logs go to stderr so stdout stays pipe-clean.

### Changed

- Self-contained keyframes: video param sets (HEVC, H.264, VC-1) are
  re-asserted at every keyframe and any mid-title change is emitted
  in-band, fixing whole-segment corruption.
- Block timestamps use presentation order per track type, so B-frame
  video keeps its true presentation timestamps.
- Mux unit alignment is scheme-aware (AACS vs CSS/none), so DVD extents
  are no longer rejected for unit misalignment.
- Subtitle `BlockDuration` is scaled by the segment timecode scale.
- MKV output records `freemkv <version>` in the Writing-application
  field, so every file is traceable to its build.
- autorip's staging-relocation check uses path existence rather than a
  write probe, so a transient storage hiccup at startup no longer
  orphans an in-progress image.

### Fixed

- A read returning GOOD status with a residual underrun is treated as a
  failed read (routed to retry) rather than committing stale buffer
  data, closing a silent-corruption hole in the sweep and patch paths.
- A read that returns sense data alongside a GOOD response is no longer
  misclassified as a transport error.
- Disc-capacity reads reject the over-32-bit sentinel instead of
  wrapping to zero and misreporting size.
- An operator stop during the mux phase no longer quarantines a
  resumable disc; stop-versus-failure is classified on typed error
  variants, not message strings.
- Settings POST validates enum fields and clamps numeric values before
  mutating config.

### Security

- Content keys (CSS and AACS) are redacted in all log output (shown as
  `<redacted>` with a 1-byte fingerprint); a test guards against any key
  being logged raw.
- The macOS device shim spawns helpers directly rather than through a
  shell, removing a command-injection vector on the device path.
- `settings.json` is written with owner-only (0600) permissions.

## 1.0.0-rc.1 — 2026-06-21

First release candidate for 1.0 — the first tagged 1.0 milestone across
the toolchain. Establishes the full feature set: the stream-URL CLI
surface, the core library (multipass recovery, content decryption, disc
parsing, and the threaded mux pipeline), and the autorip service.

### Added

- **Stream-URL CLI.** `freemkv <source> <dest>` over `disc://`,
  `iso://`, `mkv://`, `m2ts://`, `network://`, `stdio://`, and
  `null://`, with `info` for disc/file metadata and `update-keys` for
  fetching a key database.
- **Multipass recovery engine.** Pass 1 sweeps the whole disc
  sequentially, tolerating bad sectors with an adaptive damage-jump
  (mark the bad range, keep going). Pass N retries the bad ranges with
  per-sector recovery timeouts, reverse-direction reads, and range
  bisection. A mapfile tracks per-sector state across passes so a rip
  can resume.
- **Drive and SCSI layer.** Single-shot, synchronous transport on Linux
  (IOKit on macOS, SPTI on Windows), full sense decoding, and drive
  enumeration/presence probes. Single-shot reads by design — recovery
  lives in the multipass orchestration, not inline in the read path.
- **Content decryption.** CSS for DVDs and AACS 1.0/2.0 for Blu-ray and
  UHD, with AACS keys read from a key database. A single decrypting
  decorator wraps the sector source so decryption is one audited
  surface, and a resolved key is verified against disc content before it
  is applied.
- **Disc parsing.** UDF, MPLS/CLPI (Blu-ray), and IFO (DVD) parsing for
  title and extent assembly, with bounds checks on values derived from
  untrusted disc input, and canonical main-title selection.
- **Mux pipeline (the "highway").** A three-stage threaded pipeline —
  read+decrypt, demux, codec parse — with a recycled buffer pool, taking
  file-backed mux from tens of MB/s to several hundred MB/s warm-cache.
  Codec parsers for HEVC, H.264, VC-1, MPEG-2, TrueHD, DTS(-HD), and PGS
  feed a Matroska writer.
- **autorip service.** Detect a disc on insert, identify it, rip it, mux
  to MKV, and move the finished file to the library — hands-off. Web
  dashboard with live progress, per-device drive cards, settings,
  history, and webhooks. Single-pass and multi-pass rip modes with an
  abort-on-loss threshold and three-bucket Good/Maybe/Lost progress.
  Resumable staging across restarts, and a curated minimal Docker image
  alongside the static binary.

### Notes

- The library carries no user-facing English — all errors are numeric
  codes handled by the application layer. Release builds use thin LTO;
  the workspace is Rust 2024 edition.

## 0.x — pre-1.0 development (condensed)

The 0.x series was the iterative development run leading up to 1.0,
spanning roughly four dozen tagged releases. The feature set listed
under 1.0.0-rc.1 was built up and hardened across that series:

- The multipass recovery engine (adaptive sweep + targeted patch with
  bisection and resume) matured here, along with the single-shot
  drive/SCSI transport and full sense decoding.
- The mux "highway" — the three-stage threaded pipeline with a recycled
  buffer pool — was developed and tuned over this series, taking
  file-backed mux throughput up by an order of magnitude warm-cache.
- The bounded-cache writeback path (`sync_file_range` +
  `posix_fadvise`) and time-batched mapfile persistence were added to
  keep long, NFS-staged rips fast.
- The autorip service grew its parallel staged-worker pipeline (rip,
  mux, and move as independent stages so the drive frees as soon as
  sweep+patch finish), resilient `.ripped`/`.done`/`.completed`/`.failed`
  staging markers, the web dashboard, and a series of security/UX
  hardening fixes (escaped dashboard output, redacted secrets, validated
  settings, SSRF and cross-origin guards).

Per-release detail for the 0.x series is available from the git tags in
each repository.
