---
title: Decryption Keys
description: DVDs (CSS) decrypt with no setup. Blu-ray and 4K UHD (AACS) require keys you supply, from an online key service or a local keydb.cfg.
---

How freemkv decrypts a disc depends on its format: DVDs need nothing, while Blu-ray and 4K UHD need keys you supply.

## DVD

DVDs decrypt automatically. There are no keys to install and nothing to configure.

## Blu-ray and 4K UHD

| Format | Encryption |
|---|---|
| Blu-ray | AACS 1.0 |
| 4K UHD  | AACS 2.0 / 2.1 |

AACS-encrypted discs decrypt only when you supply AACS keys. No AACS keys are built in.
Provide them one of two ways: an online key service or a local `keydb.cfg`. Pick one.

## Online key service

A third-party web service looks up keys for an inserted disc on demand, so you don't
maintain a local file. autorip supports this: enable the online key service and set its URL
under **Settings**. See [autorip Service](/autorip/#keys).

## Local keydb.cfg

A `keydb.cfg` file on disk. The CLI reads it from `~/.config/freemkv/keydb.cfg` by default.
Download or refresh it from a URL with `update-keys` (the URL is required — there's no
built-in default):

```bash
# download keydb.cfg to the default location
freemkv update-keys --url <KEYDB_URL>
```

To use a `keydb.cfg` elsewhere, point the CLI at it with `--keydb`:

```bash
# use a keydb.cfg from a custom path
freemkv disc:// mkv://Movie.mkv --keydb /path/to/keydb.cfg
```

For autorip, bind-mount a host keys directory to `/root/.config/freemkv` (see the
[autorip Docker setup](/install/#docker)) so the file persists across restarts; autorip can
also download and refresh it for you.

## When keys are missing

If you rip an AACS-encrypted disc with no keys available, freemkv reports a key-load error
rather than writing corrupt output. autorip shows "no KEYDB.cfg found." DVDs are never
affected.
