# Compatibility Matrix

This file tracks verified and community-reported storage behavior on Android 11+ DAP devices.

[Back to main README](../README.md) · [Traditional Chinese article](README.zh-TW.md) · [Simplified Chinese article](README.zh-CN.md) · [English article](README.en.md) · [FAQ](faq.md)

---

## Status legend

- ✅ Verified working
- ❌ Verified not working
- ⚠️ Partial / mixed behavior
- ⏳ Community report needed

---

## What This Matrix Separates

Storage behavior is not a single yes/no result. A device can show the SD card, the Settings UI can allow an app move, and the app's offline data can still stay under shared internal storage.

This matrix separates:

- UI app migration availability
- actual package location
- offline-data behavior
- whether ADB verification was used
- Windows / MTP behavior
- firmware or region differences
- app version, when available

The main point is simple: an app can look movable in the UI while its offline data still does not move.

---

## App / Device Matrix

| Streaming App | App Version | Tested Device | Android Version | Firmware / Region | UI app migration available | Package moved | Offline data follows | ADB verified | Windows / MTP behavior | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| QQ Music | Not recorded in current verified case | Sony Walkman | Android 11 | Not explicitly separated in current verified case | ✅ Yes | ✅ Yes | ❌ No | ✅ Yes | Windows / MTP still mainly showed internal shared storage (~103/104GB) | App package can move; offline song data still mainly writes to shared storage |
| NetEase Cloud Music | ⏳ Community report needed | ⏳ Community report needed |  |  |  |  |  |  |  |  |
| Soda Music | ⏳ Community report needed | ⏳ Community report needed |  |  |  |  |  |  |  |  |
| Apple Music | ⏳ Community report needed | ⏳ Community report needed |  |  |  |  |  |  |  |  |
| Spotify | ⏳ Community report needed | ⏳ Community report needed |  |  |  |  |  |  |  |  |
| Tidal | ⏳ Community report needed | ⏳ Community report needed |  |  |  |  |  |  |  |  |
| KKBOX | ⏳ Community report needed | ⏳ Community report needed |  |  |  |  |  |  |  |  |

---

## Device Matrix

| Device | Android Version | Firmware / Region | Verified Scope | ADB-verified case available | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Sony Walkman | Android 11 | Not explicitly separated in current verified case | Verified in this repo | ✅ Yes | Main tested case |
| iBasso | Android 11+ | ⏳ Community report needed | Not yet verified here |  | Not yet verified here |
| HiBy | Android 11+ | ⏳ Community report needed | Not yet verified here |  | Not yet verified here |
| FiiO | Android 11+ | ⏳ Community report needed | Not yet verified here |  | Not yet verified here |

---

## How to Read the Matrix

**UI app migration available** only means the Settings UI exposes an app-migration entry. It does not prove that offline music data moved.

**Package moved** should ideally be checked with ADB:

```bash
adb shell pm path <package>
adb shell dumpsys package <package> | findstr volumeUuid
```

**Offline data follows** should be checked by comparing storage usage before and after downloading offline songs:

```bash
adb shell df -h /storage/emulated
adb shell df -h /mnt/expand/<UUID>
```

**Windows / MTP behavior** is useful, but incomplete. Many users first notice the problem because Windows still shows only the shared-storage side. That observation should be recorded, but it should not replace Android-side verification.

**App version** matters because streaming apps can change storage behavior between releases. If the version is unknown, leave it as not recorded rather than guessing.

---

## Reporting Suggestion

A useful report should include the device model, Android version, firmware / region, app name and version, whether UI migration is available, whether the package actually moved, whether offline data followed, whether ADB verification was used, what Windows / MTP showed, and a short result summary.

For structured reports, use:

- [CONTRIBUTING.md](../CONTRIBUTING.md)
- [Issue templates](../.github/ISSUE_TEMPLATE)
