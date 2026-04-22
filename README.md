# Android 11+ DAP SD Card Storage Study

Research on Android 11+ digital audio players (DAP) storage behavior, with **Sony Walkman + Android 11 + QQ Music** used as the primary verified case.

This repository focuses on a practical question:

> On Android 11+ DAP devices, can an external SD card really become usable storage for apps and offline music data?

The answer is more nuanced than it first appears:

- an app package may be movable,
- but its offline data may still remain in shared internal storage,
- and UI-reported capacity does not always match actual app data behavior.

---

## Who This Repository Is For

This repository is useful if you are facing problems such as:

- the SD card is recognized, but the app still consumes internal storage,
- the app can be moved, but offline songs or cached media do not follow,
- Windows / MTP still shows only the internal shared storage,
- `/storage/emulated/0` and `/mnt/expand/<UUID>` appear to behave differently,
- you want a **non-root**, reproducible workflow to verify what really moved and what did not.

---

## Common Search Problems

This repository is also written around search-oriented problem statements such as:

- **Android 11 SD card recognized but app still uses internal storage**
- **App moved but data did not follow**
- **QQ Music SD card Android 11 DAP**
- **Sony Walkman SD card app migration**
- **Windows MTP only shows 103GB**
- **`/storage/emulated/0` vs `/mnt/expand/<UUID>`**
- **Android 11 offline music still stored in internal storage**

These phrases describe the actual user-facing symptoms that this repository tries to explain.

---

## Documentation

- [Traditional Chinese / 繁體中文](docs/README.zh-TW.md)
- [Simplified Chinese / 简体中文](docs/README.zh-CN.md)
- [English](docs/README.en.md)
- [FAQ (English)](docs/faq.md)
- [FAQ (繁體中文)](docs/faq.zh-TW.md)
- [FAQ (简体中文)](docs/faq.zh-CN.md)
- [Compatibility Matrix](docs/compatibility.md)

---

## Key Findings

- Apps can be moved.
- App data may **not** follow.
- System storage layer is **not** the same as app data layer.
- UI success does **not** automatically mean offline-data success.
- The validated workflow is **UI-based app migration + ADB verification**, not root.

---

## Scope

This repository currently treats **Sony Walkman + Android 11 + QQ Music** as the verified case.

Potentially relevant to other Android 11+ DAP devices, such as:

- Sony
- iBasso
- HiBy
- FiiO

However, other devices and apps are currently treated as **community-reported / to be verified**, not as proven conclusions.

---

## Verified Workflow

The repository currently emphasizes the following workflow:

1. Enable USB debugging
2. Use ADB for inspection and verification
3. Install, launch, and close the target app once
4. Use system UI to manually move the app
5. Verify package location and offline data behavior with ADB

This is intended to separate three different layers clearly:

- what the **system** sees,
- what the **UI** allows,
- what the **app data layer** actually does.

---

## Case Summary: QQ Music on Sony Walkman

Verified result:

- the app package can be moved,
- but offline songs do **not necessarily** move with it.

This leads to the practical strategy used in this repo:

- internal storage as the **system/app disk**,
- SD card as the **music library disk**.

---

## Research Position

This repository is not written as a generic “how to move one app” note.
It is written as a storage-behavior study for Android 11+ DAP devices.

The Sony Walkman case is used to build a repeatable method for answering questions such as:

- What actually moved?
- What only looked moved in UI?
- What still writes to shared internal storage?
- Which behaviors are device-specific, and which may be broader Android 11+ DAP behaviors?

---

## Notes

This repository is not intended as a universal final answer for every Android DAP or every streaming service.
It is a growing research record, starting from a verified Sony Walkman case and expanding through later testing and community feedback.
