# Android 11+ DAP SD Card Storage Behavior Study: Sony Walkman as a Verified Case

[Back to main README](../README.md) · [Traditional Chinese](README.zh-TW.md) · [Simplified Chinese](README.zh-CN.md) · [FAQ](faq.md) · [Compatibility Matrix](compatibility.md)

---

## Introduction

This document is not just about how to move QQ Music to an SD card. It addresses a deeper question:

> On Android 11 and later, can an external SD card on a DAP (digital audio player) really become primary usable storage for apps and offline music data?

Devices such as Sony Walkman, iBasso, HiBy, and FiiO may all face the same class of problem:

- the system can see the SD card, but that does not mean an app will write data there;
- an app package may be movable, but offline songs may still remain in shared internal storage;
- UI-reported capacity may increase, but that does not mean the main shared-storage layer has truly been integrated.

So although this document uses **Sony Walkman + Android 11 + QQ Music** as the verified case, the real subject is broader:

> **storage-layer separation on Android 11+ DAP devices**.

---

## Problem Definition

This document is meant to answer several real-world questions:

1. **If the SD card is recognized by the system, why does the app still consume internal storage?**
2. **If the app can be moved, why do offline songs or cache fail to follow?**
3. **Why does Windows / MTP still show only about 103GB instead of the full 1TB?**
4. **Which steps belong to research exploration, and which steps form a reproducible end-user workflow?**
5. **Under a non-root constraint, is there a stable, verifiable, long-term workable path at all?**

The goal is not to compress every symptom into one sentence, but to separate the **system layer**, **UI layer**, and **app-data layer**.

---

## Audit Statement: What Is Verified and What Is Still Open

### Verified in this repository

The following scope has been directly tested:

- Device: Sony Walkman
- OS: Android 11
- App: QQ Music
- Tools: ADB + Sony system UI
- Verified conclusion: **the app package can move, but offline-song data does not necessarily follow**

### Still open / community-reported

The following are **not treated as proven conclusions yet**:

- whether other Android 11+ DAP devices behave the same way,
- whether China ROM / global ROM behavior is exactly identical,
- how NetEase Cloud Music, Soda Music, Apple Music, Spotify, Tidal, KKBOX, and other streaming apps behave.

In other words, this repository is currently:

> **a technical verification record using Sony Walkman as a case**, not a universal conclusion for every Android DAP and every streaming app.

---

## Conclusions First

To avoid confusion, here are the key conclusions upfront:

1. **A Sony Walkman can recognize an external 1TB SD card and expose it as an app-selectable storage target.**
2. **QQ Music's app package can be manually moved through the system UI.**
3. **QQ Music's offline-song data does not necessarily move with the package.**
4. **The validated practical workflow is USB debugging + ADB verification + Sony UI app migration.**
5. **This document does not treat root or full SD-card reformatting as the default workflow for ordinary readers.**

One-line summary:

> **Successful app-package migration does not guarantee successful app-data migration.**

---

## Research Goals

This study has three practical goals.

### 1. Solve the capacity problem

Sony Walkman internal storage is roughly 128GB, which is limited for high-resolution music libraries and offline streaming data. The goal is to make the 1TB SD card the real music-bearing storage.

### 2. Reduce write pressure on internal storage

If downloads, cache rebuilds, and repeated file churn always happen on internal storage, that storage bears both the capacity burden and the write burden. The goal is to shift most music-related data flow toward a replaceable 1TB SD card.

### 3. Verify the actual impact of Android 11 storage separation on DAP devices

Intuitively, inserting a 1TB SD card should let music apps use a much larger storage pool. In practice, Android 11 storage layering and OEM design may separate **visible capacity** from **actually usable app-data capacity**.

---

## Test Environment

- Device: Sony Walkman
- OS: Android 11
- Internal storage: about 128GB
- External storage: microSD 1TB
- Privilege model: non-root
- Tooling: ADB (Windows PowerShell)
- Verified app: QQ Music

---

## Prerequisites

Before running any ADB verification, prepare the following:

1. Enable **Developer Options** on the Walkman
2. Enable **USB debugging**
3. Connect the device to a computer via USB
4. Approve the **USB debugging authorization** prompt on the device

Verify with:

```bash
adb devices
```

Expected output:

```text
device
```

If it shows `unauthorized`, the device has not completed ADB authorization yet.

---

## Low-Level Model: Three Layers Must Be Separated

Before the workflow makes sense, Android 11 storage needs to be split into three layers.

### 1. Shared storage layer

Typical path:

```text
/storage/emulated/0
```

This is usually:

- the internal shared storage users know from file managers,
- the path many apps show in download settings,
- the layer most easily exposed to Windows through MTP.

Many users assume this is “all storage,” but it is only the most visible layer.

### 2. Expanded / external volume layer

Typical path:

```text
/mnt/expand/<UUID>
```

This is usually:

- the expanded external volume seen by the system,
- a place where Package Manager may direct an app package,
- a lower-level volume not always exposed in the same way as normal user-facing storage.

### 3. App-storage management layer

This is not a path by itself. It is a UI-level control point, for example:

```text
Settings → Storage → Apps → Storage location
```

This layer can affect:

- where the app package is placed,
- some app-managed storage behavior.

But it does **not** guarantee that the app's own:

- download directory,
- offline songs,
- cache,
- custom data folders

will follow.

One-line summary:

> App installation location, app-reported download location, and actual offline-data write location can be three different things.

---

## Main Workflow: The Method Ultimately Confirmed by This Study

The final usable workflow is **not** root and **not** “format the SD card first.” It is:

1. Enable USB debugging
2. Use ADB for observation and verification
3. Install, launch, and close the target app
4. Manually move the app through Sony UI
5. Use ADB again to verify app-package placement and offline-data behavior

### Step 1: Confirm the system sees the SD card

```bash
adb shell sm list-disks
adb shell df -h
```

The purpose here is not to rebuild storage. It is to confirm:

- the system sees the external SD card,
- the current capacity structure of internal storage and SD card,
- the baseline state before later checks.

Important note: `disk:179,192` from `sm list-disks` is a StorageManager disk identifier. It is **not** a mount path and **not** the internal storage volume itself.

### Step 2: Install QQ Music on the device first

In the verified case, QQ Music is first installed on the Walkman itself, not pre-targeted to the SD card.

This matters because the study later checks:

- the actual package location after installation,
- whether one successful launch creates observable state,
- whether manual migration changes package placement and data behavior together.

### Step 3: Launch the app once, then close it

After installation, launch QQ Music once so it can complete its required initialization, then close it.

This step should not be skipped. Without one successful run, there is no meaningful package state or data behavior to inspect.

### Step 4: Manually move the app through Sony UI

Navigate to:

```text
Settings → Storage → Apps → Storage location
```

If the UI allows selecting another storage location, manually move QQ Music toward the SD-card-backed target.

This is the core operational step of the study. The practical method is not the risky storage-rebuild route, but confirming that Sony Walkman already exposes a usable app-migration entry in the current system.

### Step 5: Verify app-package migration with ADB

First check the volume UUID:

```bash
adb shell dumpsys package com.tencent.qqmusic | findstr volumeUuid
```

Then check the APK path:

```bash
adb shell pm path com.tencent.qqmusic
```

If the result shows:

- a `volumeUuid` pointing to the external-side volume,
- and `pm path` pointing to the corresponding external storage-side location,

then app-package migration succeeded.

### Step 6: Verify whether offline data followed

Record before downloading:

```bash
adb shell df -h /storage/emulated
adb shell df -h /mnt/expand/<UUID>
```

Download a fixed amount of offline music, then run the same commands again.

Interpretation rules:

- if `/storage/emulated` increases while `/mnt/expand/<UUID>` remains almost unchanged, offline data is still mainly written to shared internal storage;
- if `/mnt/expand/<UUID>` increases, offline data has actually entered the expanded volume.

---

## QQ Music Case: Verified Result

The practical order in the QQ Music case was:

1. install QQ Music,
2. launch and run it once,
3. close it,
4. manually move it through Sony UI,
5. verify package state and data-write behavior through ADB.

### Package verification

If `dumpsys package` shows a `volumeUuid` pointing to the external volume, and `pm path` points to the external-side package path, then QQ Music's app package was successfully moved.

### Offline-data verification

When `df -h` is compared before and after downloading:

- if `/storage/emulated` usage changes,
- while `/mnt/expand/<UUID>` remains almost unchanged,

then the correct conclusion is:

> QQ Music's app package moved successfully, but offline-song data did not follow.

That is the central verified result of this repository.

---

## Practical Working Solution

Even though the system did not automatically redirect QQ Music's offline downloads to the 1TB card, a practical working strategy still emerged.

### Practical approach

- let QQ Music download first,
- then move the music-data folder into an SD-card music area such as:

```text
SD/Music/qqmusic/
```

### Structural meaning

The Android 11 limitation here behaves more like:

- it restricts **where writing happens**,
- but in many cases it does not fully restrict **where reading happens**.

So the resulting workable pattern becomes:

```text
Download -> 128GB
Store -> 1TB
Play -> read from 1TB
```

This is not full system-layer integration. But for a music-player use case, it is already practical and stable.

---

## Extension: Treating the Walkman as a Dedicated Music Device

If the device is ultimately used as a music player rather than a general Android device, the most rational storage strategy becomes:

### Internal 128GB

Reserve for:

- Android system,
- app packages,
- required cache,
- a small amount of temporary data.

### External 1TB SD card

Reserve for:

- the main local music library,
- moved music data,
- long-term archival collections.

In short:

> Use the 128GB internal storage as the **system/app disk**, and the 1TB SD card as the **music-library disk**.

This only works if you accept the separation between the download layer and the storage layer.

---

## Final Conclusion

This repository currently converges to five main conclusions:

1. Sony Walkman on Android 11 allows manual app migration through the UI in a non-root setup.
2. ADB is useful for verifying actual app-package location and data-write behavior.
3. QQ Music's app package can move, but offline-song data does not necessarily follow.
4. The final practical solution is not a full storage merge, but a split model: **system layer separated, usage layer solved pragmatically**.
5. This repository is currently a verified case study plus a repeatable method, not a final universal answer for all Android 11+ DAP devices and all streaming apps.

One-line conclusion:

> On a Sony Walkman, a 1TB SD card can become a real music-library disk, but it does not naturally become the primary storage layer for all app offline data. In the QQ Music case, package migration succeeded while song-data migration failed, so practical use depends on data relocation and storage-role separation.

---

## Was the Method Wrong? Audit Judgment

No. The method is not a hallucination, and it should not be dismissed as simply wrong.

A more precise audit judgment is:

### What was correct

- installing the app first, then running it, closing it, and moving it is a reasonable sequence;
- using ADB to verify package state and storage behavior is correct;
- separating UI-based migration from lower-level verification is the right way to think.

### What could be miswritten

- presenting `sm set-force-adoptable true` and `sm partition ... private` as mandatory steps for all readers would distort the article;
- treating `disk:179,192` as if it were the internal-storage volume itself would be inaccurate;
- assuming that UI-based app migration automatically implies offline-data migration would also be inaccurate.

So the correct final writing strategy is:

> Keep the exploratory commands as part of the research record, but keep the main workflow centered on **UI app migration + ADB verification**.

---

## Exploration Commands Retained as Research History

During the broader research process, the following commands were also used:

```bash
adb shell sm set-force-adoptable true
adb shell sm partition disk:179,192 private
adb reboot
adb shell sm list-volumes all
```

These commands were useful for:

- forcing adoptable capability,
- attempting to rebuild the external card as a private volume,
- inspecting low-level volume state.

They are **not wrong**, and they provided useful research insight. However, they are **not** the preferred workflow for ordinary readers because:

1. they involve storage reconstruction and possible formatting risk,
2. they are not the final stable workflow confirmed by this study,
3. this repository aims to preserve a reproducible non-root path for normal users.

The correct audit conclusion is:

> **These commands belong to the research trail, not to the default end-user workflow.**

---

## About the “Modify system settings” Permission

QQ Music may expose an advanced permission such as:

```text
App info → Advanced → Modify system settings
```

This permission typically affects whether the app can change certain system-level settings.
For the specific storage question discussed here, there is currently **no sufficient evidence** that this permission directly determines whether QQ Music offline songs can move to external storage.

So it may be recorded as an observation item, but it should **not** be treated as the primary cause.

---

## Device Compatibility and Community Reporting

At present, this repository has directly tested only:

- Sony Walkman
- Android 11
- QQ Music

However, this problem likely extends beyond Sony and beyond QQ Music.
The repository can later grow into an **Android 11+ DAP storage compatibility record**, supported by community reports.

### Suggested future coverage

- other Android 11+ DAP devices,
- China ROM / global ROM differences,
- storage behavior of other streaming apps.

### Suggested table

| Streaming App | Tested Device (OS) | UI app migration available | Offline data follows | Notes |
| :--- | :--- | :--- | :--- | :--- |
| QQ Music | Sony Walkman (Android 11) | ✅ Yes | ❌ No | App package can move; song data still mainly writes to shared storage |
| NetEase Cloud Music | Community report needed |  |  |  |
| Soda Music | Community report needed |  |  |  |
| Apple Music | Community report needed |  |  |  |
| Spotify | Community report needed |  |  |  |
| Tidal | Community report needed |  |  |  |
| KKBOX | Community report needed |  |  |  |

The purpose of this section is not to make universal claims now, but to turn the repository into a **community-expandable research entry point**.

---

## Why Windows / MTP Still Shows Only ~103GB

In this case:

- Sony settings may report total capacity around 1.13TB,
- but Windows over MTP still shows only around 103/104GB.

The reason is:

- MTP mainly exposes the shared storage layer,
- and the expanded external volume does not necessarily appear as an ordinary portable disk to the computer.

So:

> Windows not showing 1TB does not mean the 1TB SD card is ineffective. It means the expanded volume is not exposed through MTP in the same way as a normal external drive.
