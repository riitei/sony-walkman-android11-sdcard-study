# Methodology

This document describes how this repository checks **Android 11+ DAP SD card storage behavior**.

[Back to main README](../README.md) · [Traditional Chinese article](README.zh-TW.md) · [Simplified Chinese article](README.zh-CN.md) · [English article](README.en.md) · [Compatibility Matrix](compatibility.md)

---

## Why This Method Is Needed

The storage problem is easy to misread. A device may detect the SD card, the Settings UI may allow an app move, and Windows may still show only the shared internal storage view. Those observations can all be true at the same time.

For this Walkman test, I treated the result as a verification problem: first change one thing, then check which layer actually changed. This avoids turning a UI result into a storage conclusion too quickly.

---

## Core Rule

Do not treat these as the same thing:

- SD card detection
- Settings UI behavior
- app package location
- offline-song / cache location
- Windows / MTP view

They may point in the same direction, but Android 11 does not guarantee that they are identical.

---

## Layers Checked in This Repository

### 1. Device detection

This checks whether Android can see the external SD card at all.

```bash
adb shell sm list-disks
adb shell df -h
```

### 2. UI migration

This checks whether the device exposes an app-migration entry in Settings.

```text
Settings → Storage → Apps → Storage location
```

### 3. Package location

This checks whether the app package actually moved to the external-side volume.

```bash
adb shell pm path <package>
adb shell dumpsys package <package> | findstr volumeUuid
```

### 4. Offline data

This checks whether offline songs, downloads, or cache data followed the package move.

```bash
adb shell df -h /storage/emulated
adb shell df -h /mnt/expand/<UUID>
```

### 5. Host visibility

This checks what Windows / MTP shows after the device is connected to a PC. In this case, Windows may still show mainly the shared-storage side even when Android has a lower-level expanded volume.

---

## Test Order

The working order used here is:

```text
Install → Launch once → Close → Move through UI → Verify with ADB
```

I use this order because each step leaves a clearer state for the next check. The app needs to be installed before there is any package state to inspect. It should be launched once so it can finish basic initialization. Closing it reduces noise before the storage comparison. The UI move is the ordinary non-root migration step. ADB is then used to check what actually changed.

Without the final ADB check, the result would only be a Settings UI note, not a verification record.

---

## UI Changes State; ADB Proves the State

The main workflow deliberately separates the operation from the proof.

For ordinary non-root use, the state-changing step is the Settings UI:

```text
Settings → Storage → Apps → Storage location
```

The proof step is ADB:

```bash
adb shell pm path <package>
adb shell dumpsys package <package> | findstr volumeUuid
adb shell df -h /storage/emulated
adb shell df -h /mnt/expand/<UUID>
```

This matters because the UI can show that an app was moved, while offline data may still grow under shared internal storage.

---

## Why Windows / MTP Is Only Partial Evidence

Plugging the player into a PC is useful, but it is not enough by itself. Windows / MTP often shows the shared-storage namespace, not every lower-level Android volume.

In this repo, Windows / MTP is treated as one observation source. It is not used as the only source of truth.

---

## Exploration Commands

During testing, the following commands were also used to understand Android storage behavior:

```bash
adb shell sm set-force-adoptable true
adb shell sm partition disk:179,192 private
adb reboot
adb shell sm list-volumes all
```

These commands are useful for exploring adoptable storage and private-volume behavior, but they are not the default reader workflow. They may involve storage reconstruction or formatting risk, and they are not required for the verified non-root UI-based method.

Their role in this repo is research history, not a recommended first step.

---

## Strong Report Format

A useful compatibility report should include the device model, Android version, app name and version, whether UI migration exists, whether the package really moved, whether offline data followed, whether ADB verification was used, what Windows / MTP showed, and a short factual summary.

Example:

> App package moved through system UI. `pm path` and `volumeUuid` pointed to the external-side volume. Offline-song data still grew under shared internal storage. Windows MTP still mainly showed the internal shared-storage side.

This style is useful because it keeps the layers separate instead of compressing everything into “worked” or “failed.”

---

## Weak Report Format

Reports such as “it worked,” “it did not work,” “I think it moved,” or screenshots without explanation are still useful as early clues, but they are not strong enough for compatibility conclusions.

A Windows-only result is also incomplete. So is a UI-only result. Both need package and data checks before they can support a storage-behavior conclusion.

---

## Practical Goal

The goal is not to force Android into a preferred design. The goal is to answer three practical questions:

1. What changed?
2. What did not change?
3. What workaround is still usable?

For the Walkman + QQ Music case, the result is:

```text
Package moved -> yes
Offline data followed -> no
Practical workaround -> yes
```

---

## Summary

The method used in this repo is simple: separate the storage layers, change one state through the normal UI path, then verify with ADB.

In short:

> Change one thing, then check what really moved.
