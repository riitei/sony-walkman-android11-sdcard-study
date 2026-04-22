# Android 11+ DAP SD Card Storage Study

Research on Android 11+ digital audio players (DAP) storage behavior, with Sony Walkman used as the primary verified case.

This repository focuses on a practical question:

> On Android 11+ DAP devices, can an external SD card really become usable storage for apps and offline music data?

The answer is more nuanced than it first appears:

- an app package may be movable,
- but its offline data may still remain in shared internal storage,
- and UI-reported capacity does not always match actual app data behavior.

---

## Documentation

- [Traditional Chinese / 繁體中文](docs/README.zh-TW.md)
- [Simplified Chinese / 简体中文](docs/README.zh-CN.md)
- [English](docs/README.en.md)
- [Compatibility Matrix](docs/compatibility.md)

---

## Key Findings

- Apps can be moved.
- App data may **not** follow.
- System storage layer is **not** the same as app data layer.
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

---

## Case Summary: QQ Music on Sony Walkman

Verified result:

- the app package can be moved,
- but offline songs do **not necessarily** move with it.

This leads to the practical strategy used in this repo:

- internal storage as the **system/app disk**,
- SD card as the **music library disk**.

---

## Notes

This repository is not intended as a universal final answer for every Android DAP or every streaming service.
It is a growing research record, starting from a verified Sony Walkman case and expanding through later testing and community feedback.
