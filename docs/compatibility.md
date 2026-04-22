# Compatibility Matrix

This file tracks verified and community-reported storage behavior on Android 11+ DAP devices.

## Status legend

- ✅ Verified working
- ❌ Verified not working
- ⚠️ Partial / mixed behavior
- ⏳ Community report needed

---

## App / Device Matrix

| Streaming App | Tested Device (OS) | UI app migration available | Offline data follows | Notes |
| :--- | :--- | :--- | :--- | :--- |
| QQ Music | Sony Walkman (Android 11) | ✅ Yes | ❌ No | App package can move; offline song data still mainly writes to shared storage |
| NetEase Cloud Music | ⏳ Community report needed |  |  |  |
| Soda Music | ⏳ Community report needed |  |  |  |
| Apple Music | ⏳ Community report needed |  |  |  |
| Spotify | ⏳ Community report needed |  |  |  |
| Tidal | ⏳ Community report needed |  |  |  |
| KKBOX | ⏳ Community report needed |  |  |  |

---

## Device Matrix

| Device | Android Version | Verified Scope | Notes |
| :--- | :--- | :--- | :--- |
| Sony Walkman | Android 11 | Verified in this repo | Main tested case |
| iBasso | Android 11+ | ⏳ Community report needed | Not yet verified here |
| HiBy | Android 11+ | ⏳ Community report needed | Not yet verified here |
| FiiO | Android 11+ | ⏳ Community report needed | Not yet verified here |

---

## Reporting suggestion

If you want to contribute results, report at least:

1. Device model
2. Android version
3. App name and version
4. Whether UI app migration is available
5. Whether offline data follows after migration
6. Whether ADB verification was used (`pm path`, `dumpsys package`, `df -h`)
