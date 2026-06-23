---
title: Platforms
description: freemkv on Windows, macOS, and Linux — where each tool puts its files, how it reaches the optical drive, and per-OS quirks.
---

freemkv runs on Windows, macOS, and Linux. The same tools are available everywhere, with small per-OS differences in where files live and how each tool reaches the optical drive.

## Where does freemkv put its files?

It depends on the OS and which tool you're running:

- **`freemkv` CLI** — cross-platform, one static binary. It stores AACS keys (`keydb.cfg`) under a per-user config directory, and writes a diagnostic log only when you ask for one.
- **`autorip` service** — a self-contained app. On Windows it keeps all of its state next to the executable; on Linux it runs as a Docker container with config and staging bind-mounted in.

The exact paths differ per platform. Pick yours:

- **[Windows](/platforms-windows/)** — native SPTI drive access; autorip is self-contained next to the `.exe`.
- **[macOS](/platforms-macos/)** — IOKit exclusive drive access; `/dev/diskN` devices.
- **[Linux](/platforms-linux/)** — `/dev/sg*` SCSI generic; autorip as a Docker container.

## CLI vs. autorip

The **`freemkv` CLI** is cross-platform and behaves the same on all three systems. Differences are limited to file paths and how you name the drive device.

The **`autorip` service** runs as a native app on Windows and macOS, and as a Docker container on Linux. The web UI and rip behavior are identical across platforms; only the install model and file layout differ.
