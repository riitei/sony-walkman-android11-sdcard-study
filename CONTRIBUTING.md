# Contributing

Thanks for contributing to this repository.

This project is not just a note about one app. It is a small research repository for **Android 11+ DAP SD card storage behavior**, currently using **Sony Walkman + Android 11 + QQ Music** as the primary verified case.

The goal is to collect **reproducible**, **verifiable**, and **comparable** reports.

---

## What Kind of Contributions Are Useful

Useful contributions include:

- testing another DAP device,
- testing another Android version,
- testing another streaming app,
- confirming whether UI app migration is available,
- verifying whether offline data follows the app migration,
- comparing behavior across China ROM / global ROM,
- adding ADB-verified observations.

Less useful contributions are reports such as:

- “it works” without device/app/version details,
- “it failed” without reproduction steps,
- screenshots only, without explanation,
- conclusions that were not checked with either UI observation or ADB verification.

---

## Before You Open an Issue

Please read:

- [README](README.md)
- [Traditional Chinese article](docs/README.zh-TW.md)
- [Simplified Chinese article](docs/README.zh-CN.md)
- [English article](docs/README.en.md)
- [FAQ](docs/faq.md)
- [Compatibility Matrix](docs/compatibility.md)

This helps avoid duplicate reports and makes later compatibility tracking more reliable.

---

## Preferred Reporting Method

Please use the repository's issue template:

- **DAP storage behavior report**

That template asks for:

1. device model,
2. Android version,
3. firmware / region,
4. app name and version,
5. whether UI app migration is available,
6. whether offline data follows,
7. whether ADB verification was used,
8. reproduction steps,
9. result summary.

This structure exists so reports can later be compared side by side.

---

## Minimum Reproduction Quality

A useful report should include at least:

- **device model**
- **Android version**
- **app name**
- **clear reproduction steps**
- **clear result summary**

A stronger report also includes:

- `adb shell pm path <package>`
- `adb shell dumpsys package <package> | findstr volumeUuid`
- `adb shell df -h /storage/emulated`
- `adb shell df -h /mnt/expand/<UUID>`

---

## Recommended Verification Flow

For consistency, this repository prefers the following method:

1. Enable USB debugging
2. Install the target app
3. Launch the app once and then close it
4. Move the app through system UI
5. Verify the package location with ADB
6. Verify offline-data behavior with ADB

This matters because:

- UI appearance alone is not enough,
- package migration and offline-data migration are not the same thing,
- Windows / MTP view is not the same thing as the system's lower-level storage layout.

---

## Important Distinctions

When contributing, please keep these distinctions clear:

### 1. System sees the SD card
This does **not** automatically mean app offline data uses it.

### 2. App package moved
This does **not** automatically mean offline songs or cache moved.

### 3. Windows / MTP shows capacity
This does **not** automatically describe the real expanded-volume behavior.

Reports are most useful when they explicitly separate these three layers.

---

## About ADB and Exploration Commands

This repository distinguishes between:

- **mainline user workflow**, and
- **research exploration commands**.

The following commands may be useful for research:

```bash
adb shell sm set-force-adoptable true
adb shell sm partition disk:179,192 private
adb reboot
adb shell sm list-volumes all
```

But they are **not** the default workflow for ordinary readers because they may involve storage reconstruction and formatting risk.

If you use them in a report, please clearly say so.

---

## Updating the Compatibility Matrix

If you contribute a verified result, it may later be added to:

- [Compatibility Matrix](docs/compatibility.md)

Good entries usually state:

- device,
- Android version,
- app,
- whether UI migration exists,
- whether offline data follows,
- whether ADB verification was used,
- special notes such as MTP behavior or region differences.

---

## Writing Style for Reports

Please prefer:

- short factual statements,
- exact device / app / version names,
- explicit separation between observation and conclusion,
- direct mention of what moved and what did not.

Example:

> App package moved successfully through system UI. `pm path` changed to external-side storage. Offline-song data still increased under shared internal storage. Windows MTP still showed only about 103GB.

This is much more useful than:

> It kind of worked but also did not work.

---

## Final Note

This repository is currently a **verified case study plus a repeatable method**, not a universal final answer for all Android 11+ DAP devices and all streaming apps.

The most valuable contributions are the ones that make the repository more:

- reproducible,
- comparable,
- searchable,
- and useful to the next person facing the same storage problem.
