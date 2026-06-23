---
title: Windows
description: Running freemkv and autorip on Windows — file locations, SPTI drive access, and the self-contained autorip layout.
---

freemkv runs natively on Windows. The `freemkv` CLI is a single `.exe`, and `autorip.exe` is a self-contained service.

## Install / how to run

Download the Windows build from the **[Download](/download/)** page, then run it from a terminal:

```powershell
freemkv.exe --version
autorip.exe serve          # then open http://localhost:8080
```

No runtime or dependencies — each is a standalone executable.

## File locations

### freemkv CLI

| What | Path |
|---|---|
| AACS keys (`keydb.cfg`) | `%APPDATA%\freemkv\keydb.cfg` |
| Diagnostic log | off by default — see below |

The CLI keeps the terminal clean and never writes a log unless asked. Run with `--log-level 3` to write `.\log.txt` in the current directory, or `--log-file PATH` to choose the destination.

### autorip

`autorip.exe` is **self-contained**: it keeps all of its state in a `config\` folder **next to the executable**. Move the folder and its state moves with it.

| What | Path (relative to `autorip.exe`) |
|---|---|
| Settings | `config\settings.json` |
| Per-device logs | `config\logs\` |
| AACS keys | `config\keys\keydb.cfg` |
| Staging | `config\staging\` |
| Output | `config\output\` |

Override the base location with the `AUTORIP_DIR` environment variable. If the executable's own path can't be determined, autorip falls back to `%APPDATA%\autorip`.

## Device / drive access

autorip and the CLI reach the drive through native **SPTI** (SCSI pass-through). Pass a drive letter or device path; no special permissions are typically needed beyond running the program normally.

## Known quirks / troubleshooting

- Because autorip is self-contained, back it up or relocate it by copying the whole app folder (executable plus its `config\` folder).
- Earlier releases scattered autorip state at the drive root (`C:\config`); as of rc4 everything lives next to the executable. If you have leftover state at `C:\config`, you can delete it once the app folder's `config\` is in use.
- For capturing logs and other common fixes, see [Troubleshooting](/troubleshooting/).
