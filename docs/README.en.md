# Android 11+ DAP Storage Behavior Study

## Problem

On Android 11+, SD cards are no longer treated as simple external storage.

Storage is split into:

- `/storage/emulated/0` (shared layer)
- `/mnt/expand/<UUID>` (expanded volume)

Key question:

> Does moving an app also move its data?

---

## Method

- ADB inspection (`df -h`, `dumpsys`, `pm path`)
- System UI app migration
- Before/after comparison

---

## Key Findings

- Apps can be moved via system UI
- App data does NOT necessarily follow
- Behavior depends on app implementation

---

## Example (QQ Music)

- APK moved successfully
- Offline data remained in shared storage

Conclusion:

> App installation location != App data location

---

## Scope

Applies to most Android 11+ DAP devices:

- Sony
- iBasso
- HiBy
- FiiO
