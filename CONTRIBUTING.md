# Contributing

Thanks for helping improve this repository.

This project tracks **Android 11+ DAP SD card storage behavior**. The current verified case is **Sony Walkman + Android 11 + QQ Music**. Reports are most useful when they are reproducible and easy to compare with the existing Walkman test.

---

## Useful Contributions

Useful reports include another DAP device, another Android version, another streaming app, a China ROM / global ROM comparison, or an ADB-verified check of whether offline data follows app migration.

Less useful reports are things like “it works,” “it failed,” or screenshots without device, app, version, and verification details. They can still be clues, but they are not enough to update the compatibility matrix.

---

## Before Opening an Issue

Please read the main notes first:

- [README](README.md)
- [Traditional Chinese article](docs/README.zh-TW.md)
- [Simplified Chinese article](docs/README.zh-CN.md)
- [English article](docs/README.en.md)
- [FAQ](docs/faq.md)
- [Compatibility Matrix](docs/compatibility.md)

This helps avoid duplicate reports and keeps the compatibility table easier to maintain.

---

## Preferred Report Format

Please use the repository issue template for DAP storage behavior reports.

A good report should include:

- device model
- Android version
- firmware / region
- app name and version
- whether UI app migration is available
- whether the package actually moved
- whether offline data followed
- whether ADB verification was used
- reproduction steps
- short result summary

The goal is not to make the report long. The goal is to make it comparable.

---

## Minimum Reproduction Quality

At minimum, include the device model, Android version, app name, clear steps, and a clear result summary.

A stronger report includes these ADB checks:

```bash
adb shell pm path <package>
adb shell dumpsys package <package> | findstr volumeUuid
adb shell df -h /storage/emulated
adb shell df -h /mnt/expand/<UUID>
```

---

## Recommended Verification Flow

For consistency, this repo prefers this order:

```text
Install → Launch once → Close → Move through UI → Verify with ADB
```

That order matters because UI appearance alone is not enough. Package migration and offline-data migration are not the same thing, and Windows / MTP does not always show the lower-level Android storage layout.

---

## Keep These Layers Separate

When reporting a result, please separate these points:

- whether Android sees the SD card
- whether the Settings UI allows app migration
- whether the app package moved
- whether offline songs or cache moved
- what Windows / MTP showed

The most common mistake is treating “the app moved in UI” as proof that offline music data also moved.

---

## About ADB and Exploration Commands

This repo separates the normal user workflow from research exploration.

The following commands may help when exploring Android storage behavior:

```bash
adb shell sm set-force-adoptable true
adb shell sm partition disk:179,192 private
adb reboot
adb shell sm list-volumes all
```

They are not the default workflow for ordinary readers because they may involve storage reconstruction or formatting risk. If you used them, mention that clearly in the report.

---

## Updating the Compatibility Matrix

Verified results may be added to:

- [Compatibility Matrix](docs/compatibility.md)

Good matrix entries state the device, Android version, app, whether UI migration exists, whether offline data follows, whether ADB verification was used, and any special notes such as MTP behavior or region differences.

---

## Writing Style for Reports

Use short factual statements. Mention exact device / app / version names. Separate observation from conclusion.

Example:

> App package moved successfully through system UI. `pm path` changed to external-side storage. Offline-song data still increased under shared internal storage. Windows MTP still showed only about 103GB.

This is much more useful than:

> It kind of worked but also did not work.

---

## Final Note

This repository is a verified case study plus a repeatable method. It is not a universal final answer for every Android 11+ DAP and every streaming app.

The best contributions make the next user's test easier to reproduce and compare.
