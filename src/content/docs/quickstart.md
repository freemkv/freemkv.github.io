---
title: Quickstart
description: Inspect a disc, rip it to MKV with bad-sector recovery, and remux an existing file.
---

This page takes you from a disc in the drive to an MKV on disk in a few commands.

It assumes you've [installed the CLI](/install/). DVDs need no extra setup; for
Blu-ray or 4K UHD, set up your [decryption keys](/decryption-keys/) first.

Everything is addressed with `scheme://` [stream URLs](/cli/#stream-urls). The
full command surface is in the [CLI reference](/cli/).

## Inspect the disc

See what's on the disc — titles, streams, and encryption status. This works
without an AACS key.

```bash
freemkv info disc://
```

Useful options:

```text
disc:///dev/sg4     target a specific drive
--full              list every title, not just the main feature
--log-level 2       show extra AACS and drive detail
```

## Rip the disc to MKV

Rip the disc straight to an MKV, decrypting and muxing in one step:

```bash
freemkv disc:// mkv://Movie.mkv
```

To rip only the main feature, select its title with `-t`. Repeat `-t` to keep
several titles.

```bash
freemkv disc:// mkv://Movie.mkv -t 1
```

### Rip a scratched disc

For a disc with read errors, sweep it to an ISO with recovery enabled, re-run
the same command to patch the bad ranges, then mux the result.

```bash
# Sweep the disc to an ISO, then re-run the SAME command to patch the
# remaining bad ranges from the disc. Repeat until the mapfile is clean.
freemkv disc:// iso://Disc.iso --multipass

# Mux the recovered ISO to MKV
freemkv iso://Disc.iso mkv://Movie.mkv
```

Each pass auto-detects where it left off from the mapfile and re-reads only the
ranges still marked bad. See [How recovery works](/how-recovery-works/) for what
happens under the hood.

## Remux an existing file

Already have an ISO or an M2TS? Decrypt (if needed) and mux it to MKV without
touching a drive.

```bash
freemkv iso://Movie.iso mkv://Movie.mkv
freemkv m2ts://Movie.m2ts mkv://Movie.mkv
```

## Verify a disc

A read-only health check that reports good, slow, recovered, and bad sectors,
and exits non-zero if anything is unrecoverable.

```bash
freemkv verify disc://
```

## Update decryption keys

Blu-ray and UHD discs need decryption keys (see [Decryption Keys](/decryption-keys/)). To fetch them with the CLI:

```bash
freemkv update-keys --url <KEYDB_URL>
```

See [Decryption Keys](/decryption-keys/) for where keys live and why
AACS-encrypted discs require them.

## Going hands-off

To rip a stack of discs without running a command per disc, use the
[autorip service](/autorip/): insert a disc, get an MKV, repeat.
