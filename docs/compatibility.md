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

## What This Matrix Tries to Separate

This matrix does **not** treat storage behavior as a single yes/no question.
It tries to separate several different layers:

1. **UI app migration available**
2. **App package actually moved**
3. **Offline data actually followed**
4. **ADB verification used or not**
5. **Windows / MTP behavior**
6. **Firmware / region differences**

This matters because:

> An app can appear movable in UI, while its offline data still remains in shared internal storage.

---

## App / Device Matrix

| Streaming App | Tested Device | Android Version | Firmware / Region | UI app migration available | Package moved | Offline data follows | ADB verified | Windows / MTP behavior | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| QQ Music | Sony Walkman | Android 11 | Not explicitly separated in current verified case | ✅ Yes | ✅ Yes | ❌ No | ✅ Yes | Windows / MTP still mainly showed internal shared storage (~103/104GB) | App package can move; offline song data still mainly writes to shared storage |
| NetEase Cloud Music | ⏳ Community report needed |  |  |  |  |  |  |  |  |
| Soda Music | ⏳ Community report needed |  |  |  |  |  |  |  |  |
| Apple Music | ⏳ Community report needed |  |  |  |  |  |  |  |  |
| Spotify | ⏳ Community report needed |  |  |  |  |  |  |  |  |
| Tidal | ⏳ Community report needed |  |  |  |  |  |  |  |  |
| KKBOX | ⏳ Community report needed |  |  |  |  |  |  |  |  |

---

## Device Matrix

| Device | Android Version | Firmware / Region | Verified Scope | ADB-verified case available | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Sony Walkman | Android 11 | Not explicitly separated in current verified case | Verified in this repo | ✅ Yes | Main tested case |
| iBasso | Android 11+ | ⏳ Community report needed | Not yet verified here |  | Not yet verified here |
| HiBy | Android 11+ | ⏳ Community report needed | Not yet verified here |  | Not yet verified here |
| FiiO | Android 11+ | ⏳ Community report needed | Not yet verified here |  | Not yet verified here |

---

## Interpretation Notes

### 1. UI app migration available
This only means the system UI exposes an app-migration entry.
It does **not** guarantee offline-data migration.

### 2. Package moved
This should ideally be confirmed with ADB, for example:

```bash
adb shell pm path <package>
adb shell dumpsys package <package> | findstr volumeUuid
```

### 3. Offline data follows
This should ideally be checked using storage comparison, for example:

```bash
adb shell df -h /storage/emulated
adb shell df -h /mnt/expand/<UUID>
```

### 4. Windows / MTP behavior
This is useful because many users incorrectly assume that what Windows shows is the full storage truth.
In practice, MTP often still reflects mainly the shared-storage layer.

---

## Reporting Suggestion

If you want to contribute a result, report at least:

1. device model,
2. Android version,
3. firmware / region,
4. app name and version,
5. whether UI app migration is available,
6. whether the package actually moved,
7. whether offline data followed,
8. whether ADB verification was used,
9. what Windows / MTP showed,
10. a short factual result summary.

For structured reports, please use the repository's issue template and contribution guide:

- [CONTRIBUTING.md](../CONTRIBUTING.md)
- [Issue templates](../.github/ISSUE_TEMPLATE)
