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
Provide them one of two ways: an online key service or a local `keydb.cfg`. Both autorip and
the `freemkv` CLI support either source.

## Online key service

A third-party web service looks up keys for an inserted disc on demand, so you don't
maintain a local file.

In **autorip**, enable the online key service and set its URL under **Settings**.
See [autorip Service](/autorip/#keys).

In the **CLI**, point at the service with `--key-url` (and `--key-auth` if it requires a
bearer token):

```bash
# resolve keys from an online key service
freemkv disc:// mkv://Movie.mkv --key-url https://keys.example/keys

# with an authentication token
freemkv disc:// mkv://Movie.mkv --key-url https://keys.example/keys --key-auth <TOKEN>
```

If you supply **both** `--key-url` and `--keydb`, the local keydb is consulted first
(local-first) and the service is only queried when the keydb has no key for the disc.
The URL is validated before any request, and freemkv refuses to send disc-key material to a
loopback, private, or cloud-metadata address.

## Local keydb.cfg

A `keydb.cfg` file on disk. The CLI looks for it in a per-OS list of locations and uses the
first one that exists.

| OS | Searched in order |
| --- | --- |
| **Windows** | `%APPDATA%\freemkv\keydb.cfg` (e.g. `C:\Users\<you>\AppData\Roaming\freemkv\keydb.cfg`), then the legacy `%USERPROFILE%\.config\freemkv\keydb.cfg` |
| **Linux / macOS** | `$XDG_CONFIG_HOME/freemkv/keydb.cfg` (if `XDG_CONFIG_HOME` is set), then `~/.config/freemkv/keydb.cfg` |

On Windows, put the file at `%APPDATA%\freemkv\keydb.cfg` — type `%APPDATA%` into the
Explorer address bar to open the `Roaming` folder, then create a `freemkv` subfolder. The
older `.config` dotfolder under your user profile is still read for back-compat, but
`%APPDATA%` is the recommended location.

Download or refresh it from a URL with `update-keys` (the URL is required — there's no
built-in default). It writes to the first location above for your OS (`%APPDATA%\freemkv\`
on Windows, `~/.config/freemkv/` on Linux/macOS):

```bash
# download keydb.cfg to the default location
freemkv update-keys --url <KEYDB_URL>
```

To use a `keydb.cfg` elsewhere, point the CLI at it with `--keydb`:

```bash
# use a keydb.cfg from a custom path
freemkv disc:// mkv://Movie.mkv --keydb /path/to/keydb.cfg
```

**autorip** looks in the same per-OS default location as the CLI (above), and can also
download and refresh the file for you from **Settings**. For the Docker image, bind-mount a
host keys directory to `/root/.config/freemkv` (see [autorip → Deploy](/autorip/#deploy)) so
it persists across restarts.

## When keys are missing

If you rip an AACS-encrypted disc with no keys available, freemkv reports a key-load error
rather than writing corrupt output. autorip shows "no KEYDB.cfg found." DVDs are never
affected.
