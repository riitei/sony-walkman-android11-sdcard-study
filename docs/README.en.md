# Android 11+ DAP SD Card Storage Behavior Study: Sony Walkman as a Verified Case

[Back to main README](../README.md) · [Traditional Chinese](README.zh-TW.md) · [Simplified Chinese](README.zh-CN.md) · [FAQ](faq.md) · [Compatibility Matrix](compatibility.md)

---

## Introduction

This article records a real test with **Sony Walkman + Android 11 + QQ Music**. I first wanted to check whether QQ Music could use a 1TB SD card. The actual issue turned out to be more specific: on Android 11, system storage, Settings UI behavior, and app data behavior are not always the same thing.

In this test, the Walkman could see the SD card, and the QQ Music app package could be moved through Sony's Settings UI. But offline songs did not necessarily move with the app. Windows / MTP also continued to show mainly the shared internal storage view.

So this is not only a “move one app to SD card” note. It is a storage-behavior record: what moved, what only looked moved, and what still wrote to internal shared storage.

---

## Problem Definition

The practical questions are simple: why does an app still consume internal storage after the SD card is detected; why can the app package move while offline songs stay behind; and why does Windows / MTP still show about 103GB instead of the full 1TB?

The key is to separate three layers:

- system layer: whether Android detects the SD card,
- UI layer: whether Settings allows app migration,
- app-data layer: whether songs, cache, and downloads actually write to the SD card.

This article does not treat a successful UI move as proof that app data moved.

---

## Current Verified Scope

The verified case is Sony Walkman, Android 11, QQ Music, tested with ADB and Sony's native Settings UI. The confirmed result is: **the QQ Music app package can move, but offline-song data does not necessarily follow.**

Other Android 11+ DAPs, China ROM / global ROM differences, and other streaming apps such as NetEase Cloud Music, Soda Music, Apple Music, Spotify, Tidal, and KKBOX are left for later reports. They are not treated as proven conclusions here.

In short, this is a Sony Walkman verification record, not a universal conclusion for every Android DAP and every streaming app.

---

## Conclusions First

The current findings are:

1. A Sony Walkman can recognize an external 1TB SD card and expose it as an app-selectable storage target.
2. QQ Music's app package can be manually moved through the system UI.
3. QQ Music's offline-song data does not necessarily move with the package.
4. The stable verification path is USB debugging, ADB checks, Sony UI migration, and another ADB check.
5. Root and full SD-card reformatting are not used as the default reader workflow.

The main point is: **successful app-package migration does not guarantee successful app-data migration.**

---

## Research Goals

My first goal was to solve the 128GB internal-storage limit on the Walkman. High-resolution music, offline streaming files, and long-term cache can quickly fill internal storage, and repeated downloads or cache rebuilds also keep writing to the same internal disk.

Intuitively, a 1TB SD card should become the main music storage. In practice, Android 11 storage separation and Sony's OEM behavior make “visible capacity” and “usable app-data capacity” different things. This article records that gap.

---

## Test Environment

| Item | Value |
| :--- | :--- |
| Device | Sony Walkman |
| OS | Android 11 |
| Internal storage | about 128GB |
| External storage | microSD 1TB |
| Privilege model | non-root |
| Tooling | ADB (Windows PowerShell) |
| Verified app | QQ Music |

---

## Prerequisites

Enable Developer Options and USB debugging on the Walkman, connect it to a computer by USB, and approve the USB debugging prompt on the device. Then check the connection:

```bash
adb devices
```

A normal result shows `device`. If it shows `unauthorized`, the Walkman has not approved the ADB prompt yet.

---

## Low-Level Model: Three Layers Must Be Separated

Android 11 storage cannot be judged from one path only. In this test, the confusing parts were these three layers.

The first layer is shared storage:

```text
/storage/emulated/0
```

This is the internal shared storage most users see in file managers. Many apps also show this kind of path in download settings, and Windows / MTP usually exposes this layer most clearly.

The second layer is the expanded or external volume:

```text
/mnt/expand/<UUID>
```

After the SD card is mounted by Android, Package Manager may place an app package on this external-side volume. But this volume is not always exposed like a normal USB drive, and users may not be able to manage it directly from a normal file manager.

The third layer is app-storage management in Settings:

```text
Settings → Storage → Apps → Storage location
```

This UI entry may change where the app package is placed. It does not guarantee that the app's own downloads, offline songs, cache, or custom folders will also move.

So the working assumption is: app installation location, app-reported download location, and actual offline-data write location may be three different things.

---

## Main Workflow

The final workflow is not root-based and does not start by reformatting the SD card. It uses Sony's normal UI to move the app, then uses ADB to verify the result.

The order is: enable USB debugging, inspect the current storage state with ADB, install QQ Music, launch it once, close it, move it through Sony Settings, and then compare package location and data-write behavior with ADB.

### Step 1: Confirm that Android sees the SD card

```bash
adb shell sm list-disks
adb shell df -h
```

This step only establishes the baseline. It confirms that the system sees the external SD card and shows the current internal / external storage structure. The `disk:179,192` value from `sm list-disks` is a StorageManager disk identifier, not a mount path and not the internal storage volume itself.

### Step 2: Install and launch QQ Music once

In this case, QQ Music was first installed on the Walkman itself. After installation, launch it once so it can finish basic initialization, then close it. This gives the later package and data checks a meaningful state to compare.

### Step 3: Move the app through Sony UI

Navigate to:

```text
Settings → Storage → Apps → Storage location
```

If another storage target is available, move QQ Music toward the SD-card-backed location. This only suggests that the app package may move. It does not prove that offline songs move.

### Step 4: Verify app-package location with ADB

Check the volume UUID:

```bash
adb shell dumpsys package com.tencent.qqmusic | findstr volumeUuid
```

Then check the APK path:

```bash
adb shell pm path com.tencent.qqmusic
```

If `volumeUuid` points to the external-side volume and `pm path` points to the corresponding external-side path, the app package has moved.

### Step 5: Verify whether offline data followed

Before and after downloading songs, run:

```bash
adb shell df -h /storage/emulated
adb shell df -h /mnt/expand/<UUID>
```

If `/storage/emulated` changes while `/mnt/expand/<UUID>` barely changes, offline data is still mainly written to shared internal storage. If the expanded volume changes clearly, then there is stronger evidence that offline data entered the SD-card-backed volume.

---

## QQ Music Case: Verified Result

The actual test order was: install QQ Music, open it once, close it, move it through Sony UI, then check package state and data writes with ADB.

For the app package, `dumpsys package` and `pm path` can confirm whether the package moved to the external-side volume.

For offline data, the important check is the before/after `df -h` comparison. In this test, the key observation was that app-package migration succeeded, but offline-song data did not reliably follow.

---

## Practical Working Solution

The system did not automatically redirect QQ Music's offline downloads to the 1TB card, but a practical workflow still exists: let QQ Music download first, then move the music-data folder into an SD-card music area such as:

```text
SD/Music/qqmusic/
```

The Android 11 limit here behaves more like a write-location problem than a read-location problem. The usable pattern becomes:

```text
Download -> 128GB
Store -> 1TB
Play -> read from 1TB
```

This is not full system-level storage integration. For a dedicated music player, though, it is still usable.

---

## Extension: Treating the Walkman as a Dedicated Music Device

If the Walkman is treated as a music player rather than a general Android device, the storage roles become clearer. The 128GB internal storage should be reserved for Android, app packages, required cache, and small temporary files. The 1TB SD card should be used for the local music library, moved music data, and long-term collections.

In short: **use 128GB as the system / app disk, and 1TB as the music-library disk.**

This approach works only if the download layer and the storage layer are treated as separate.

---

## Final Conclusion

This test currently leads to five conclusions: Sony Walkman on Android 11 allows manual app migration through the UI in a non-root setup; ADB is useful for verifying app-package location and data-write behavior; QQ Music's app package can move, but offline-song data does not necessarily follow; the practical solution is not a full storage merge, but a split model; and this repository is a verified case study plus a repeatable method, not a universal answer for all Android 11+ DAPs and all streaming apps.

Put more simply: on a Sony Walkman, a 1TB SD card can become a real music-library disk, but it does not automatically become the primary storage layer for every app's offline data. In the QQ Music case, package migration succeeded while song-data migration failed, so practical use depends on data relocation and clear storage roles.

---

## Method Notes

The testing sequence was not wrong. Installing the app, launching it once, closing it, moving it through UI, and checking it with ADB is a reasonable order.

The part that needs careful wording is the exploration path. Commands such as `sm set-force-adoptable true` and `sm partition ... private` should not be presented as mandatory steps for all readers. `disk:179,192` should not be described as the internal-storage volume itself. And UI-based app migration should not be treated as proof that offline data also moved.

The main workflow should stay centered on **UI app migration + ADB verification**.

---

## Exploration Commands

During the broader test, these commands were also used:

```bash
adb shell sm set-force-adoptable true
adb shell sm partition disk:179,192 private
adb reboot
adb shell sm list-volumes all
```

They helped reveal adoptable-storage and private-volume behavior. They are still part of the research trail, but they are not the recommended first steps for ordinary readers because they may involve storage reconstruction or formatting risk.

---

## About the “Modify system settings” Permission

QQ Music may expose an advanced permission such as:

```text
App info → Advanced → Modify system settings
```

For this storage question, there is currently no sufficient evidence that this permission directly determines whether QQ Music offline songs can move to external storage. It can be recorded as an observation, but it should not be treated as the main cause.

---

## Device Compatibility and Community Reporting

At present, this repository has directly tested only Sony Walkman, Android 11, and QQ Music. Later reports can add other Android 11+ DAPs, ROM differences, and other streaming apps using the same fields.

| Streaming App | Tested Device (OS) | UI app migration available | Offline data follows | Notes |
| :--- | :--- | :--- | :--- | :--- |
| QQ Music | Sony Walkman (Android 11) | ✅ Yes | ❌ No | App package can move; song data still mainly writes to shared storage |
| NetEase Cloud Music | Community report needed |  |  |  |
| Soda Music | Community report needed |  |  |  |
| Apple Music | Community report needed |  |  |  |
| Spotify | Community report needed |  |  |  |
| Tidal | Community report needed |  |  |  |
| KKBOX | Community report needed |  |  |  |

This section is only a reporting entry point, not a universal compatibility claim.

---

## Why Windows / MTP Still Shows Only ~103GB

In this case, Sony Settings may report total capacity around 1.13TB, while Windows over MTP still shows only around 103/104GB. MTP mainly exposes the shared storage layer, and the expanded external volume does not necessarily appear as a normal portable disk on the computer.

So Windows not showing 1TB does not mean the SD card is ineffective. It means the expanded volume is not exposed through MTP in the same way as a normal external drive.
