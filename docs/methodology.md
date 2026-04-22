# Methodology

This document explains the method used in this repository to analyze **Android 11+ DAP SD card storage behavior**.

[Back to main README](../README.md) · [Traditional Chinese article](README.zh-TW.md) · [Simplified Chinese article](README.zh-CN.md) · [English article](README.en.md) · [Compatibility Matrix](compatibility.md)

---

## Why a Separate Methodology Document Exists

Most storage discussions fail because they mix several different questions together:

- Did the system detect the SD card?
- Did the UI allow the app to move?
- Did the app package really move?
- Did offline songs actually follow?
- Did Windows / MTP reflect the real storage layout?

If these are not separated, the result is usually a false conclusion.

This repository therefore treats the problem as a **verification problem**, not just a settings problem.

---

## Core Principle

The method used here is based on one rule:

> **Do not treat system visibility, UI behavior, package location, offline-data location, and Windows / MTP view as the same thing.**

They may overlap, but they are not guaranteed to be identical.

---

## The Five Layers This Repository Separates

### 1. Device-detection layer
Question:

- Can Android see the external SD card at all?

Typical checks:

```bash
adb shell sm list-disks
adb shell df -h
```

### 2. UI migration layer
Question:

- Does the system UI expose an app-migration entry?

Typical path:

```text
Settings → Storage → Apps → Storage location
```

### 3. Package-location layer
Question:

- Did the app package really move to the external-side volume?

Typical checks:

```bash
adb shell pm path <package>
adb shell dumpsys package <package> | findstr volumeUuid
```

### 4. Offline-data layer
Question:

- Did offline songs, cache, or downloads actually follow?

Typical checks:

```bash
adb shell df -h /storage/emulated
adb shell df -h /mnt/expand/<UUID>
```

### 5. Host-visibility layer
Question:

- What does Windows / MTP actually show?

Typical observation:

- Windows may still show only the shared-storage side, even when the device internally uses an expanded volume.

---

## Why the Workflow Uses "Install → Launch Once → Close → Move → Verify"

This order is intentional.

### Step 1: Install the app
Without installation, there is no package state to inspect.

### Step 2: Launch the app once
Without one real launch, the app may not finish initialization.
That means later package or data observations may be incomplete.

### Step 3: Close the app
This reduces noise during later storage checks and keeps the next comparison cleaner.

### Step 4: Move the app through UI
This is the actual migration operation for ordinary non-root use.

### Step 5: Verify with ADB
Only after the move should the result be checked.
Otherwise, the article becomes a UI note instead of a verification record.

---

## Why the Mainline Workflow Is UI-Based but Verification Is ADB-Based

This repository separates:

- **the action that changes state**, and
- **the action that proves what changed**.

### State-changing step
For ordinary users, the actual migration step is the UI path:

```text
Settings → Storage → Apps → Storage location
```

### Proof step
The proof step is ADB-based:

```bash
adb shell pm path <package>
adb shell dumpsys package <package> | findstr volumeUuid
adb shell df -h /storage/emulated
adb shell df -h /mnt/expand/<UUID>
```

This separation matters because UI alone can be misleading.

---

## Why Windows / MTP Is Not Used as the Only Judge

Many users naturally check storage by plugging the device into a PC.
That is useful, but it is not enough.

Why?

Because Windows / MTP often mainly reflects the shared-storage namespace, not every lower-level expanded volume.

So the method in this repository is:

- treat Windows / MTP as **one observation source**,
- but do not use it as the **only truth source**.

---

## Why Exploration Commands Are Kept but Not Promoted as Default Steps

The following commands may help during research:

```bash
adb shell sm set-force-adoptable true
adb shell sm partition disk:179,192 private
adb reboot
adb shell sm list-volumes all
```

They are kept because they help reveal low-level storage structure.

They are **not** promoted as the default reader workflow because:

1. they may involve storage reconstruction,
2. they may imply formatting risk,
3. they are not required for the verified non-root UI-based method.

So the methodology here is:

> Keep exploration commands as research evidence, but keep the mainline user workflow conservative.

---

## What Counts as a Strong Result

A strong result in this repository usually includes all of the following:

1. the device model,
2. Android version,
3. app name and version,
4. whether UI migration exists,
5. whether the package really moved,
6. whether offline data followed,
7. whether ADB verification was used,
8. what Windows / MTP showed,
9. a short factual summary.

Example:

> App package moved through system UI. `pm path` and `volumeUuid` pointed to the external-side volume. Offline-song data still grew under shared internal storage. Windows MTP still mainly showed the internal shared-storage side.

This is strong because it separates the layers instead of compressing them into one sentence.

---

## What Counts as a Weak Result

Weak results are usually things like:

- “It worked”
- “It did not work”
- “I think it moved”
- screenshots without explanation
- Windows view only, without system-side verification
- UI observation only, without package/data verification

These are not useless, but they are not strong enough for compatibility conclusions.

---

## Practical Goal of the Method

The goal of this method is not to force Android into a preferred design.
It is to answer three practical questions clearly:

1. What changed?
2. What did not change?
3. What can still be used as a practical workaround?

That is why the final output of this repository often becomes:

```text
Package moved -> yes
Offline data followed -> no
Practical workaround -> yes
```

---

## Final Method Summary

This repository uses the following method:

1. separate storage behavior into multiple layers,
2. use UI for the ordinary migration step,
3. use ADB for proof,
4. treat Windows / MTP as partial evidence only,
5. keep exploration commands as research history,
6. prefer reproducible, comparable, factual reporting.

In short:

> **The method is not “change one setting and hope.” The method is “separate the layers, change one state, and verify what really moved.”**
