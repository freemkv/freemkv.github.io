---
title: Changelog
description: Notable changes across the freemkv toolchain (CLI, library, and autorip service), newest first.
---

All notable changes across the freemkv toolchain (the `freemkv` CLI,
the `libfreemkv` core library, and the `autorip` service) are recorded
here, newest first. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.0.0/) and the project
follows semantic versioning.

The toolchain releases as a set: every component ships the same version
number on each release, even when a given component has no functional
change in that cycle.

## 1.0.0-rc.5.2 (2026-06-24)

A recovery and decryption-correctness round. Single-pass rips now recover
marginal sectors the way multipass already did, large AACS Blu-ray titles
that failed to decrypt now mux cleanly, the correct audio track is picked
on DVDs with unusual sub-stream ordering, interlaced SD-DVD reports its
full frame rate on Windows, and a 4K decode glitch on discs with
non-seamless clip joins is fixed.

### Fixed

- **Windows Explorer now reports the full frame rate for interlaced
  SD-DVD.** A field-duration hint added in the previous release backfired:
  on interlaced (576i/480i) discs it made Windows Explorer report half the
  frame rate and flipped tools such as MediaInfo to "variable". The hint is
  no longer written; the frame-rate signal players trust is the full-frame
  value, so Explorer now shows the correct rate and reports a constant frame
  rate. Interlace signalling (top-field-first) is retained.
- **Correct audio track selected on DVDs with non-standard sub-stream
  ordering.** freemkv assumed a disc's first declared audio stream lives on
  the first physical sub-stream. On discs where the 5.1 main mix sits
  elsewhere and the first sub-stream carries a 2.0 down-mix (for example
  *The Silence of the Lambs*), the 2.0 track was muxed under a "5.1" label.
  freemkv now probes each physical sub-stream's actual channel count from
  the disc and routes every declared stream onto the sub-stream that
  genuinely matches.
- **"Decryption failed" on large AACS Blu-ray titles.** The biggest
  titles on some AACS Blu-rays failed to decrypt and aborted the mux.
  The decryption unit alignment is now anchored to each clip, so these
  titles decrypt and mux correctly.
- **Single-pass rips now recover marginal and transient sectors.** A
  single-pass (direct disc → MKV) rip now gives the drive its full error
  recovery budget on a bad sector before reporting it unreadable, matching
  the behavior of multipass mode. Discs with a few recoverable read errors
  that previously failed (or lost data) in single-pass now complete.
- **4K playback glitches on discs with non-seamless clip joins.** Some
  UHD discs (for example the *Top Gun* class of titles) join clips
  non-seamlessly, which produced "Could not find ref" decode errors and
  visible glitches at the joins during playback. These joins are now
  handled correctly and play back cleanly.

### Changed

- **`freemkv-keysources` is now a pure key lookup.** The encrypted
  content-sample reader and the candidate-key resolution loop moved into
  `libfreemkv` (they read the disc and validate keys — decryption
  mechanism, not lookup). A key source now only looks a key up and hands it
  back.

### Added

- **`--log-level 3` is now self-sufficient for MKV and opening-frame
  diagnosis.** The diagnostic pass now dumps the actual MKV track-header
  elements written per track, and captures the first ~100 coded frames per
  track to an `<output>.opening.bin` side file with a per-frame summary, so
  container-metadata and opening-GOP issues are diagnosable from a log alone
  without the disc. Both are gated to log level 3; a normal run writes
  nothing extra.

### Verified

- **DVD opening-GOP / still-frame handling is correct (no change needed).**
  The theory that a title's opening pictures get the wrong sequence header
  or have their timestamps floored to zero was traced and ruled out: the
  codec setup is the first sequence header, DVD structure guarantees each
  title opens on a sequence header and I-frame, leading still-frames are
  anchored to the disc's real timeline, and the muxer bases its timestamps
  on the opening keyframe's real time. Regression tests pin all three.

## 1.0.0-rc.5.1 (2026-06-24)

Patch on rc.5. DVD decryption reliability improvements, audio and video
metadata fixes, cleaner CLI validation, broader UI localization, and a
minor autorip housekeeping fix.

### Fixed

- **DVDs now decrypt correctly on CSS-enforcing drives.** Some drives
  require a bus-level authentication handshake before they allow
  scrambled sectors to be read. Without it, freemkv produced an empty
  MKV (exit 0 — silently wrong) or hung indefinitely. The handshake is
  now performed automatically; if a title key still cannot be recovered,
  you get a clear error instead of an empty file.
- **DVD first-play menu is no longer prepended to the movie.** On discs
  that author a per-title menu (for example a studio "the parental level
  has been set, press yes" prompt), that menu was being included ahead of
  the feature, shifting the whole rip. The rip now opens on the movie's
  first frame.
- **DVD audio channel count is now read from the audio stream.** Some
  discs incorrectly label a stereo (2.0) audio track as 5.1 in their
  disc table of contents. freemkv now reads the channel count directly
  from the AC-3 audio data, so the MKV track reflects what the stream
  actually contains.
- **Interlaced DVDs now report the correct frame rate on Windows.**
  Interlaced (576i/480i) content now includes the metadata field Windows
  uses to determine the display frame rate. Previously, Windows Media
  Player and Explorer could show an incorrect or blank frame rate for
  interlaced discs.
- **Interlaced field order corrected.** Interlaced tracks now carry
  top-field-first (TFF) order in the MKV, matching the order in the
  source stream, so deinterlacers in players use the correct field
  parity.
- **Video and audio bitrates now appear in file properties.** Each track
  in the output MKV now carries a per-track bitrate tag, so players and
  Windows Explorer can display the per-stream bitrate without scanning
  the whole file.
- **autorip: no more spurious "stranded staging directory" warnings.**
  autorip's mover logged a false warning every 10 seconds for any staging
  directory that was actively being written to. The warning now appears
  only when a directory is genuinely left behind with no active rip (for
  example after a crash or restart).

### Changed

- **AACS handshake skipped on DVDs.** The AACS authentication sequence is
  no longer attempted on DVDs, where it never applied — it was a no-op at
  best and produced spurious errors at worst.

### Added

- **Disc diagnostics at log level 3.** Running with `--log-level 3` now
  emits a structured snapshot of the disc's layout before muxing — PGC
  and cell structure for DVDs; playlist, clip, and AACS metadata for
  Blu-ray and UHD. Useful when diagnosing a mux or authentication
  problem without needing a developer build.
- **CLI input validation.** `--log-level`, `--key-url`, and `--language`
  now validate their inputs up front and print a specific, actionable
  error if the value is invalid.
- **UI localization in seven languages.** The interface is now localized
  across seven locales.

## 1.0.0-rc.4.3 (2026-06-23)

Patch on rc.4.2. DVD playback-correctness fixes (PAL aspect, resolution,
and colour) plus broader optical-drive unlock support.

### Fixed

- **DVD: 16:9 widescreen no longer shows as 4:3.** Anamorphic DVDs now
  carry the correct display aspect ratio in the MKV, so a widescreen film
  fills the frame instead of appearing squeezed into 4:3.
- **DVD: PAL discs are detected as PAL.** A PAL disc (576-line, 25 fps)
  was being mis-read as NTSC (480-line, 29.97 fps) because the video
  standard was read from the wrong bits of the disc's video attributes.
  PAL discs now report the correct resolution and frame rate.
- **DVD: standard-definition colour is tagged correctly.** SD DVDs were
  stamped with HD (BT.709) colour. They now carry the correct SD
  colorimetry — BT.470BG for PAL, SMPTE-170M for NTSC — so colours render
  as intended.
- **More optical drives unlock correctly.** Per-drive unlock data is now
  applied accurately across the full range of supported drives, fixing
  drives that previously failed to unlock.

## 1.0.0-rc.4.2 (2026-06-23)

Patch on rc.4.1. Windows durability fixes — Windows users on rc.4 or
rc.4.1 should update.

### Fixed

- **Windows: autorip no longer re-muxes the same disc forever.** When a
  rip finished, autorip flushed the completed MKV to disk before marking
  it done — but on Windows that flush was rejected with "Access is
  denied", so the "done" marker was never written and the service
  re-muxed the same disc on an endless loop. The flush now opens the file
  with write access, so it succeeds and the rip completes normally.
- **Windows: the staging free-space check works.** The "is there room to
  rip?" pre-flight did nothing on Windows, so a too-small staging drive
  could fill up partway through a rip with no warning. autorip now reads
  the real free space on Windows before it starts.
- **Windows: log noise removed.** An internal directory-sync step that
  has no Windows equivalent logged a warning on every write (thousands
  per rip, including from the CLI). It is now skipped on Windows.

## 1.0.0-rc.4.1 (2026-06-23)

Patch on rc.4. A single, high-impact Windows fix.

### Fixed

- **Windows: drives are detected again.** rc.4 shipped a regression that
  made 64-bit Windows report "no drives detected" — drive enumeration
  came back empty because the SCSI pass-through request structure used a
  32-bit memory layout on a 64-bit host, so every low-level drive query
  failed. The structure now uses the correct 64-bit layout (matching the
  Windows SDK), and drive detection works as it did before rc.4. Windows
  users on rc.4 should update.

## 1.0.0-rc.4 (2026-06-23)

Focused on a clean, predictable operator experience, clearer error
messages, and a round of audit-driven correctness and durability fixes.

### Changed

- **Clean terminal by default.** The CLI terminal now shows only curated
  progress, status, and a final result block, never raw diagnostic
  log lines. How you invoke the tools is unchanged.
- **Diagnostic logging is file-only.** Trace/debug output is written to
  a log file only when you ask for it (`--log-level`, `--log-file`, or
  `RUST_LOG`), with timestamps on and terminal colour codes off. With
  none of those set, no log file is written and nothing bleeds onto the
  terminal. The default log path is `./log.txt`.
- **Overhauled error messages.** A fatal error now prints a single
  readable block (what failed, the plain-English cause, and how to turn
  on a diagnostic log, and where it's written) instead of a raw
  internal code. 47 previously unmapped error codes now have clear
  messages, jargon-heavy ones were rewritten, and the `verify` subcommand
  is fully localized.

### Fixed

- The reported failure is now the root cause (for example, an early
  key/authorization rejection at the drive handshake), not the cascaded
  read error it used to surface further down the pipeline.
- **autorip: no more re-mux loop.** A DVD that hit a post-mux loss abort
  could be re-muxed indefinitely; the failed marker is now terminal in
  the mux worker, so an aborted disc stays aborted.
- A disc → ISO rip that recovered zero readable bytes now fails instead
  of producing an empty image, and a CLI pipe that hits an early EOF no
  longer exits successfully with a structurally invalid MKV.
- Decrypt-time loss is now accounted for, so a partial AACS/CSS decryption
  failure can no longer pass as a perfect rip; partial per-title key
  coverage is rejected rather than producing partly-garbage output.
- Durability and correctness hardening across the library: the key
  database is written atomically (temp + fsync + rename) and the resume
  checkpoint fsyncs its directory; dropped connections and transport
  failures are preserved as their true cause instead of being relabeled;
  a failed capacity read warns instead of silently treating the disc as
  empty; and a leaked pipeline consumer can no longer finalize an
  abandoned output.
- Windows drive access fixes: correct SCSI pass-through struct packing
  and bus-type field width, surfaced storage-reset failures, and bounded
  read-batch sizing on non-sysfs (Windows) optical drives.

## 1.0.0-rc.3.1 (2026-06-22)

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
  bytes were produced before it reports "done"; an empty or
  zero-output result can no longer be mistaken for success, in both the
  CLI/library and the autorip service.
- Correct DVD/CSS handling for the no-key and per-title cases, with the
  scrambled-versus-unencrypted distinction made explicitly instead of
  inferred.

## 1.0.0-rc.3 (2026-06-22)

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

## 1.0.0-rc.2 (2026-06-22)

Second release candidate. Adds keyless DVD/CSS ripping and correct DVD
video output, on top of security and recovery hardening.

### Added

- **Keyless DVD/CSS ripping.** A CSS-protected DVD decrypts with no key
  database: the title key is recovered directly from the scrambled disc
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
  directly with no container, storing config under `~/.config/autorip`,
  useful for the static binary on a bare Linux host.
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

## 1.0.0-rc.1 (2026-06-21)

First release candidate for 1.0, the first tagged 1.0 milestone across
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
  enumeration/presence probes. Single-shot reads by design: recovery
  lives in the multipass orchestration, not inline in the read path.
- **Content decryption.** CSS for DVDs and AACS 1.0/2.0 for Blu-ray and
  UHD, with AACS keys read from a key database. A single decrypting
  decorator wraps the sector source so decryption is one audited
  surface, and a resolved key is verified against disc content before it
  is applied.
- **Disc parsing.** UDF, MPLS/CLPI (Blu-ray), and IFO (DVD) parsing for
  title and extent assembly, with bounds checks on values derived from
  untrusted disc input, and canonical main-title selection.
- **Mux pipeline (the "highway").** A three-stage threaded pipeline
  (read+decrypt, demux, codec parse) with a recycled buffer pool, taking
  file-backed mux from tens of MB/s to several hundred MB/s warm-cache.
  Codec parsers for HEVC, H.264, VC-1, MPEG-2, TrueHD, DTS(-HD), and PGS
  feed a Matroska writer.
- **autorip service.** Detect a disc on insert, identify it, rip it, mux
  to MKV, and move the finished file to the library, hands-off. Web
  dashboard with live progress, per-device drive cards, settings,
  history, and webhooks. Single-pass and multi-pass rip modes with an
  abort-on-loss threshold and three-bucket Good/Maybe/Lost progress.
  Resumable staging across restarts, and a curated minimal Docker image
  alongside the static binary.

### Notes

- The library carries no user-facing English; all errors are numeric
  codes handled by the application layer. Release builds use thin LTO;
  the workspace is Rust 2024 edition.

## 0.x: pre-1.0 development (condensed)

The 0.x series was the iterative development run leading up to 1.0,
spanning roughly four dozen tagged releases. The feature set listed
under 1.0.0-rc.1 was built up and hardened across that series:

- The multipass recovery engine (adaptive sweep + targeted patch with
  bisection and resume) matured here, along with the single-shot
  drive/SCSI transport and full sense decoding.
- The mux "highway" (the three-stage threaded pipeline with a recycled
  buffer pool) was developed and tuned over this series, taking
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
