---
title: Unlocked drives
description: What an unlocked-firmware optical drive is, why it helps, and which drive families freemkv supports today.
---

Some optical drives can run **modified ("unlocked") firmware** in place of the manufacturer's stock firmware. Unlocking lifts restrictions the manufacturer built in, most notably **[rip-lock](/drives-oem/#rip-lock-why-a-stock-drive-can-be-slow)** (the artificial speed cap), as well as the host-certificate requirement, so ripping is **faster and more reliable**. If you rip a lot of discs, an unlocked drive is the nicer experience.

You don't *need* an unlocked drive (**[stock drives work](/drives-oem/)**), but a supported unlocked drive is the smoothest path, especially for 4K UHD.

## Which drives does freemkv support?

:::caution[Nothing here is guaranteed]
This is a best-effort list, not a promise. Drives, firmware versions, and even individual discs all vary, so yours might be on a firmware version we haven't profiled yet. The only way to know for sure is to try it.
:::

### MediaTek-based drives

The large majority of consumer Blu-ray drives use a **MediaTek** chipset, and freemkv supports them. This covers **~200 drive + firmware combinations**, including:

- **LG**: most LG Blu-ray writers. In Windows your drive may show up as **"HL-DT-ST"**; that's LG (Hitachi-LG Data Storage), not a different brand. This is the most common family by far.
- **ASUS**: the BW-16 series (e.g. BW-16D1HT, BW-16D1X-U) and BC-12 series. These are often LG hardware under an ASUS badge.
- **HP**: a handful of rebadged BD-RE / BD-RW models.

If you own an LG, ASUS, or HP Blu-ray drive from roughly **2012-2020**, it's very likely supported.

### Work in Progress

**Pioneer** drives (the BDR-… series, built on a different chipset) are **not supported yet**; support is actively being worked on. Any unsupported drive falls back to the **[stock-drive AACS workflow](/drives-oem/)**.

## How do I tell what I have?

Check the drive's model number; it's on the label, or in Windows under **Device Manager → DVD/CD-ROM drives**:

- **LG / "HL-DT-ST" / ASUS BW-16 / HP BD** → MediaTek platform → **likely supported**.
- **Pioneer BDR-…** → **work in progress**.

## "It works in other software but not freemkv"

If a drive rips fine in some other tool but not here, it's almost always a **firmware-support difference**: different programs cover different drive families and firmware versions. freemkv's supported set is the list above, and it's **growing with every release**.

## Help us add your drive

**Even if your drive already works, please share it.** Every profile we collect helps us add features, handle edge cases, and support more drives. Whether yours works, doesn't, or you just want to help, send us its firmware profile; it takes one command:

```bash
freemkv info disc:// --share --mask
```

This captures your drive's identity and capabilities so we can add or verify support. `--mask` hides your serial number. freemkv can submit the profile **straight to GitHub** for you, or just print it so you can copy-paste it into a new issue. Every profile helps us extend support to more drives.

:::note[This list keeps changing]
Drive support is an **evolving list**. Each freemkv release can add new drive families and firmware versions (Pioneer is next on the roadmap), so support will change over time. If your drive isn't covered yet, it may well be in a later release; keep an eye on the **[Changelog](/changelog/)**.
:::
