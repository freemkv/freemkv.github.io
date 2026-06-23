---
title: Windows
description: A complete step-by-step Windows guide for freemkv and autorip — downloading, getting past SmartScreen, running from PowerShell, where your files go, and fixing common problems.
---

freemkv runs natively on Windows. There are two programs, each a single file: **`freemkv.exe`** (the command-line tool) and **`autorip.exe`** (a self-contained service you open in your browser). This page walks through everything start to finish; if you've never run a program from a terminal before, you can still follow along.

## Step 1 — Download the right file

Go to the **[Download](/download/)** page. It detects Windows and gives you the correct file:

- **`freemkv.exe`** — the command-line tool (manual, one disc at a time).
- **`autorip.exe`** — the automatic service with a web page (insert a disc, it rips by itself). Most people want this one.

These are plain `.exe` files. There is **no installer** — the download *is* the program.

:::caution[Pick a real folder, not Downloads]
Don't run it straight out of your `Downloads` folder. Make a dedicated folder so autorip's files stay tidy in one place:

```
C:\Users\you\freemkv\
```

Move the downloaded `.exe` into that folder. (Replace `you` with your actual Windows username.)
:::

## Step 2 — Get past the blue "Windows protected your PC" screen

The first time you run a freshly downloaded program that isn't code-signed, Windows SmartScreen shows a blue box that says **"Windows protected your PC"**. This is expected for any small independent tool — it doesn't mean the file is broken or infected.

To run it anyway:

1. Click the **More info** link in the blue box.
2. A **Run anyway** button appears at the bottom. Click it.

You only have to do this once per download.

:::note[Antivirus flagged it?]
Some antivirus tools quarantine unknown `.exe` files on sight (a "false positive" — it's reacting to the file being new and unsigned, not to anything it found). If the file vanishes after download or won't start, check your antivirus quarantine and allow/restore it, then add an exclusion for your `C:\Users\you\freemkv\` folder.
:::

## Step 3 — Open a terminal *in the folder*

freemkv is a command-line program, so **double-clicking the `.exe` does nothing useful** — a black window may flash and vanish. You run it by typing a command in a terminal that's already pointed at the folder.

The easy way:

1. Open **File Explorer** and go to your `C:\Users\you\freemkv\` folder.
2. **Right-click an empty area** inside the folder.
3. Choose **Open in Terminal** (Windows 11) or **Open PowerShell window here** (Windows 10).

A terminal opens, already "inside" your folder. You'll know it worked because the prompt shows your folder path, e.g.:

```
PS C:\Users\you\freemkv>
```

## Step 4 — Run it

Type the command and press Enter. **On Windows you must type the `.\` in front** (it tells PowerShell "the program is right here in this folder"):

```powershell
# check it runs at all
.\autorip.exe --version

# start the service
.\autorip.exe serve
```

When `serve` is running, the terminal will say it's listening. **Leave that window open** — closing it stops autorip. Now open your browser to:

```
http://localhost:8080
```

That's the autorip control panel. Insert a disc and it takes over from there. For the `freemkv` CLI instead, the same `.\` rule applies:

```powershell
.\freemkv.exe --version
.\freemkv.exe disc-info
```

:::tip["...is not recognized" error?]
If you typed `autorip.exe` and got *"The term 'autorip.exe' is not recognized..."*, you left off the `.\`. Type `.\autorip.exe` — with the leading dot-backslash — and it'll work.
:::

## Step 5 — Where are my files? (the important part)

`autorip.exe` is **self-contained**. It keeps *everything* in a single `config\` folder that sits **right next to the executable** — it does **not** scatter files across your system, and it will **never** create a `C:\config` folder at the root of your drive.

So if your `autorip.exe` is at `C:\Users\you\freemkv\autorip.exe`, then:

| What | Where it lives |
|---|---|
| Settings | `C:\Users\you\freemkv\config\settings.json` |
| Logs (one file per drive) | `C:\Users\you\freemkv\config\logs\` |
| AACS keys (`keydb.cfg`) | `C:\Users\you\freemkv\config\keys\keydb.cfg` |
| Work-in-progress (staging) | `C:\Users\you\freemkv\config\staging\` |
| **Finished movies** | `C:\Users\you\freemkv\config\output\` |

**Your finished `.mkv` files are in `config\output\`** unless you change the output folder in **Settings** in the web UI.

The control panel always shows you the *real, full path* it's using for staging and output — so you're never guessing where things went. If you'd rather send finished movies straight to a NAS or another drive, set the **Output directory** in Settings to any path you like, e.g. `D:\Movies` or `\\NAS\media\movies`.

:::tip[Moving or backing up autorip]
Because it's self-contained, you back up or relocate the whole thing by copying **the folder** — the `.exe` and its `config\` folder together. Drop that folder on another PC and it picks up exactly where it left off.
:::

### Putting it somewhere else

If you want autorip's data in a specific spot regardless of where the `.exe` lives, set the **`AUTORIP_DIR`** environment variable to a full path before starting it:

```powershell
$env:AUTORIP_DIR = "D:\autorip-data"
.\autorip.exe serve
```

### The `freemkv` CLI's files

The CLI is lighter and stores only its keys under your user profile:

| What | Path |
|---|---|
| AACS keys (`keydb.cfg`) | `%APPDATA%\freemkv\keydb.cfg` |
| Diagnostic log | off by default (see [Logs](#step-7--turning-on-logs-for-bug-reports)) |

`%APPDATA%` expands to `C:\Users\you\AppData\Roaming`, so the full path is `C:\Users\you\AppData\Roaming\freemkv\keydb.cfg`.

## Step 6 — Decryption keys for Blu-ray and UHD

**DVDs work out of the box** — no key file needed. **Blu-ray and 4K UHD discs are AACS-encrypted** and need a `keydb.cfg` key database:

- **autorip** can fetch and refresh it automatically — set the **KEYDB Update URL** in **Settings** and it downloads to `config\keys\keydb.cfg` and keeps it current.
- Or drop a `keydb.cfg` you already have into `config\keys\` yourself.

If a Blu-ray/UHD rip stops with *"no KEYDB.cfg found,"* that's this — see **[Decryption Keys](/decryption-keys/)** for the full explanation.

## Step 7 — Turning on logs (for bug reports)

By design freemkv keeps the terminal **clean** and writes **no log file** unless you ask. If you hit a problem and want to file a report, turn logging on:

```powershell
# CLI: write a detailed log next to where you're running it
.\freemkv.exe --log-level 3 disc-info
# creates .\log.txt in the current folder
```

Use `--log-level 3` for bug reports (level 4 is even more verbose). To choose where the file goes, add `--log-file C:\Users\you\Desktop\freemkv-log.txt`. For autorip, enable debug logging from the web UI and the per-drive logs collect under `config\logs\`.

## Device / drive access

freemkv and autorip reach your optical drive through native **SPTI** (SCSI pass-through) — the standard Windows way to talk to a drive directly. No drivers and, in normal use, no special permissions: just run the program normally. You can refer to a drive by its letter (e.g. `D:`) where a device is requested.

:::caution[Run-as-admin]
You generally do **not** need to run as Administrator. If a drive isn't detected, first make sure the disc is fully seated and the drive shows up in File Explorer; only try an elevated terminal if normal access genuinely fails.
:::

## Common problems

| Symptom | What's going on / fix |
|---|---|
| Blue "Windows protected your PC" box | SmartScreen on an unsigned download. Click **More info → Run anyway** (Step 2). |
| `.exe` flashes a black window and closes | You double-clicked it. It's a terminal program — run it from a terminal (Step 3–4). |
| *"...is not recognized as the name of a cmdlet"* | You left off `.\`. Type `.\autorip.exe` (Step 4). |
| Downloaded file disappeared | Antivirus quarantine — restore it and add a folder exclusion (Step 2 note). |
| "No drives detected" | Confirm the drive appears in File Explorer and a disc is inserted; reseat USB drives. |
| Web page won't load at `localhost:8080` | The `serve` terminal window must stay open. If you closed it, run `.\autorip.exe serve` again. |
| Can't find my finished movie | It's in `config\output\` next to the `.exe`, unless you changed **Output directory** in Settings (Step 5). |
| Leftover `C:\config` from an old version | Pre-rc4 builds used `C:\config`; new builds don't. Once the app-folder `config\` is in use you can delete the old `C:\config`. |

For capturing logs and fixes that apply to every platform, see **[Troubleshooting](/troubleshooting/)**.
