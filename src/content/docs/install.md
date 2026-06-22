---
title: Install
description: Install the freemkv CLI, run the autorip service, or build from source.
---

How to get freemkv: install the CLI (a prebuilt binary, or build from source), or run the
autorip service.

## CLI

Go to the **[Download](/download/)** page — it detects your OS and hands you the right
binary (Linux, macOS, or Windows; Intel or ARM). Make it executable and run it:

```bash
# rename the downloaded binary to `freemkv`, make it executable, and run it
mv freemkv-* freemkv && chmod +x freemkv
./freemkv --version
```

One static binary, no runtime or dependencies. Then head to the
**[Quickstart](/quickstart/)** for your first rip.

Need a specific platform, an older version, or a `.sha256` checksum? Every build is on the
[releases page](https://github.com/freemkv/freemkv/releases).

## autorip

[autorip](/autorip/) is a web app: insert a disc and it rips automatically to MKV, with
progress, settings, and history all in the browser. It runs on Linux, macOS, or Windows, on a
machine with an optical drive (a home server or NAS works well). Run it as a single binary, or
— on Linux — via Docker.

### Binary

Get the autorip binary from the **[Download](/download/)** page, then run the service:

```bash
# rename to `autorip`, make it executable, and start the service
mv autorip-* autorip && chmod +x autorip
./autorip serve          # then open http://localhost:8080
```

One static binary, no container, no runtime. Drive access uses the `cdrom` group or a udev
rule; see **[autorip → Deploy](/autorip/#deploy)** for a systemd unit and configuration.

### Docker

Also published to GHCR at `ghcr.io/freemkv/autorip:latest`:

```yaml
# docker-compose.yml
services:
  autorip:
    image: ghcr.io/freemkv/autorip:latest
    # REQUIRED — without privileged, drive enumeration returns zero drives
    privileged: true
    volumes:
      # optical drive access
      - /dev:/dev
      # finished MKVs land here
      - ./output:/output
      # keydb.cfg for Blu-ray / UHD decryption
      - ./config/keys:/root/.config/freemkv
    ports:
      # web UI
      - "8080:8080"
```

```bash
docker compose up -d
```

Three things matter for that one-time setup:

- **`privileged: true` is required.** Without it the container starts but drive
  enumeration silently returns zero drives and the UI shows "No drives detected."
- **Bind-mount `/dev:/dev`** so autorip can see the optical drives.
- **Bind-mount a keys directory to `/root/.config/freemkv`** so your `keydb.cfg` persists
  across restarts. Required for AACS-encrypted discs (Blu-ray + UHD); DVDs (CSS) work
  without it. See **[Decryption Keys](/decryption-keys/)**.

Open `http://<host>:8080`, configure settings, and insert a disc — autorip takes it from
there. For the full compose file (staging volume, health check, log-level env var) and
every setting, see **[autorip → Deploy](/autorip/#deploy)**.

## Build from source

Pure Rust — clone and build with Cargo. Builds both the CLI and autorip:

```bash
git clone https://github.com/freemkv/freemkv
cd freemkv
cargo build --release
# binaries at target/release/freemkv and target/release/autorip
```

## Next steps

- **[CLI reference](/cli/)** — every subcommand, flag, and stream URL.
- **[Decryption Keys](/decryption-keys/)** — what Blu-ray and UHD need before they decrypt.
