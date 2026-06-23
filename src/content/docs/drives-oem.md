---
title: OEM (stock) drives
description: How freemkv works with a normal, off-the-shelf optical drive — DVDs, the legitimate AACS handshake for Blu-ray and 4K UHD, and why stock drives can be slow ("rip-lock").
---

An **OEM drive** is just a normal drive — the one that came in your computer, or any Blu-ray/DVD drive you buy off the shelf. It runs the manufacturer's original ("stock") firmware. **Most people have one of these, and that's completely fine.**

## DVDs just work

DVDs use older copy protection that freemkv handles entirely on its own. **Any drive, no setup, no key files** — put the disc in and rip.

## Blu-ray and 4K UHD: the legitimate AACS workflow

Blu-ray and 4K UHD discs use a newer protection system called **AACS**. freemkv reads them the official, by-the-book way — the same handshake the AACS system was designed for. Two things have to be in place:

1. **A security handshake with the drive.** Before it hands over the disc, the drive checks that the software is allowed to ask, using a *host certificate*. freemkv performs this standard AACS handshake — nothing is bypassed or faked.
2. **Keys and credentials, from your key sources.** freemkv reads its host certificate **and** the disc's decryption keys from the **[key sources](/decryption-keys/)** you set up — a local key database, or an online key service (which will also supply the host certificate in a future release). That's the piece that does the real work, and it's why a disc that "just works" in some other software needs this one setup step here: freemkv keeps these credentials in *your* key sources instead of baking them in.

DVDs need none of this; Blu-ray and 4K UHD do. See **[Decryption Keys](/decryption-keys/)** to set up your key sources.

## "Rip-lock" — why a stock drive can be slow

Many manufacturers **rip-lock** their drives: the firmware deliberately caps the read speed when it notices you're copying a whole disc (it keeps the drive quiet and cool for movie *playback*). Nothing is broken — it's slow *by design*, and a two-hour movie can take a while.

If you want full speed and fewer restrictions, that's exactly what an **[unlocked drive](/drives-unlocked/)** gives you.

## In short

| Disc | On a stock drive | Host cert + keys needed? |
|---|---|---|
| DVD | Works, any drive | No |
| Blu-ray | Works | Yes — from your key sources |
| 4K UHD | Works | Yes — from your key sources |

Want it faster, or is a particular drive giving you trouble? See **[Unlocked drives](/drives-unlocked/)**.
