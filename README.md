# Android 11+ DAP SD Card Storage Study

This repository records a practical storage test on Android 11+ digital audio players (DAPs). The verified case is **Sony Walkman + Android 11 + QQ Music**.

The problem I wanted to check was simple: after an Android DAP recognizes an external SD card, can that card really become useful storage for apps and offline music data?

In the Walkman test, the answer was mixed. The app package could be moved through the Android system UI, but offline songs did not necessarily follow the app package. Windows / MTP also continued to show the shared internal storage view, so the UI result alone was not enough to prove where the app data was actually written.

---

## What This Repository Covers

This repo is mainly for cases where the SD card is visible, but the app still appears to consume internal storage. It is also useful when an app can be moved, but downloaded songs, cache, or offline media remain under shared internal storage.

The notes focus on a **non-root** workflow: use the Android system UI to move the app, then use ADB to verify what changed. The goal is to separate three layers that are easy to mix up:

- what Android storage reports,
- what the Settings UI allows,
- where the app actually writes offline data.

---

## Common Search Problems

These are the symptoms this repo tries to explain:

- Android 11 SD card recognized but app still uses internal storage
- App moved but data did not follow
- QQ Music SD card Android 11 DAP
- Sony Walkman SD card app migration
- Windows MTP only shows internal storage capacity
- `/storage/emulated/0` vs `/mnt/expand/<UUID>`
- Android 11 offline music still stored in internal storage

---

## Documentation

- [Traditional Chinese / 繁體中文](docs/README.zh-TW.md)
- [Simplified Chinese / 简体中文](docs/README.zh-CN.md)
- [English](docs/README.en.md)
- [Methodology](docs/methodology.md)
- [FAQ (English)](docs/faq.md)
- [FAQ (繁體中文)](docs/faq.zh-TW.md)
- [FAQ (简体中文)](docs/faq.zh-CN.md)
- [Compatibility Matrix](docs/compatibility.md)

---

## Project Structure

`README.md` is the entry point. The longer articles are stored under `docs/`, with separate Traditional Chinese, Simplified Chinese, and English versions. Methodology, FAQ, compatibility notes, contribution rules, and issue templates are split out so device reports can stay comparable.

Important files:

- `docs/README.zh-TW.md` / `docs/README.zh-CN.md` / `docs/README.en.md`
- `docs/methodology.md`
- `docs/faq.md`, `docs/faq.zh-TW.md`, `docs/faq.zh-CN.md`
- `docs/compatibility.md`
- `CONTRIBUTING.md`
- `.github/ISSUE_TEMPLATE/`

---

## Key Findings

In the verified Walkman case, moving the app package did not automatically prove that offline music data moved with it. The system storage layer, the Settings UI, and the app data layer need to be checked separately.

The practical result is:

- internal storage remains the safer system / app disk,
- SD card works better as the music library disk,
- ADB verification is needed before treating an app migration as successful.

---

## Current Verified Scope

The confirmed test case is **Sony Walkman + Android 11 + QQ Music**.

Other Android 11+ DAP devices may show related behavior, including Sony, iBasso, HiBy, and FiiO models. They are not treated as proven conclusions in this repo unless there is a reproducible report with device model, firmware, app version, and verification method.

---

## Verified Workflow

The current workflow is UI migration first, ADB verification second.

First enable USB debugging and use ADB to inspect the device. Then install the target app, launch it once, close it, and move it through the Android system UI if that option is available. After the move, use ADB again to check package location and offline data behavior.

This avoids treating a successful Settings UI operation as proof that downloaded music or cache data also moved.

---

## Case Summary: QQ Music on Sony Walkman

My test result was that QQ Music could be moved at the app-package level, but offline songs did **not necessarily** follow the app package.

That is why this repo treats the SD card as a media-library solution rather than a complete replacement for internal app storage.

---

## Research Position

This is not a universal guide for every Android player or every streaming app. It is a storage-behavior record built from one verified Walkman case, then expanded only when later tests or community reports can be compared against the same method.

The main questions are:

- What actually moved?
- What only looked moved in the UI?
- What still writes to shared internal storage?
- Which behavior is device-specific, and which behavior may be common on Android 11+ DAPs?

---

## Contributing

Verified reports are welcome. Please include the device model, Android version, firmware / region, app name and version, whether UI app migration is available, whether offline data follows, and whether ADB verification was used.

Before opening a report, read:

- [CONTRIBUTING.md](CONTRIBUTING.md)
- [Compatibility Matrix](docs/compatibility.md)
- [Methodology](docs/methodology.md)

Reports with the same fields are easier to compare, especially when different devices show similar UI behavior but different app-data behavior.

---

## Notes

This repo starts from a verified Sony Walkman case. Broader Android 11+ DAP conclusions should be added only after comparable tests or clearly labeled community reports.
