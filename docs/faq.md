# FAQ

This FAQ answers the common questions that came up while testing **Android 11+ DAP SD card storage behavior**.

[Back to main README](../README.md) · [Traditional Chinese](README.zh-TW.md) · [Simplified Chinese](README.zh-CN.md) · [English article](README.en.md) · [Compatibility Matrix](compatibility.md)

---

## 1. The SD card is recognized, but the app still uses internal storage. Why?

Because SD card detection and app data storage are not the same layer.

On Android 11+, the system may see the SD card, the Settings UI may allow an app move, and the app may still write its offline songs or cache to shared internal storage. In this repo, those layers are checked separately.

---

## 2. The app moved successfully. Why did the data not follow?

The app package and the app's offline data are controlled separately. A package can move to an external-side volume, while downloads or cache still remain under:

```text
/storage/emulated/0
```

That is the main behavior observed in the Sony Walkman + QQ Music case.

---

## 3. Does moving an app to SD card also move offline music data?

Not necessarily.

In the verified Walkman + QQ Music test:

```text
app package moved -> yes
offline song data followed -> no
```

So app migration success should not be treated as proof that offline music data also moved.

---

## 4. Why does Windows / MTP still show only about 103GB instead of the full 1TB?

Windows / MTP usually shows the shared-storage layer, not every lower-level Android volume.

In this test, the Walkman settings page could report a larger total capacity, while Windows still mainly showed the internal shared-storage side. That does not mean the SD card is useless. It means the expanded volume is not exposed to MTP like a normal portable drive.

---

## 5. What is `/storage/emulated/0`?

It is the shared internal storage layer most users see in file managers. Many apps also show it as the visible download location.

It is important, but it is not the whole storage model of the device.

---

## 6. What is `/mnt/expand/<UUID>`?

It is a lower-level expanded or external volume path seen by Android. In some cases, Package Manager may place the app package there.

That still does not guarantee that offline songs or cache data will follow.

---

## 7. What does `disk:179,192` mean?

It is a StorageManager disk identifier returned by:

```bash
adb shell sm list-disks
```

It is not a mount path, and it is not the internal storage volume itself. It is only an Android storage-management identifier.

---

## 8. Is root required?

No. The verified workflow in this repo is non-root.

The normal path is: enable USB debugging, inspect with ADB, install and launch the app once, move it through the system UI, then verify package and data behavior with ADB.

---

## 9. Are these commands mandatory?

```bash
adb shell sm set-force-adoptable true
adb shell sm partition disk:179,192 private
adb reboot
adb shell sm list-volumes all
```

No. They are part of the research trail, not the default reader workflow.

They helped reveal low-level storage behavior, but they may involve storage reconstruction or formatting risk. The stable main workflow in this repo is still UI app migration plus ADB verification.

---

## 10. Is the main workflow UI-based or command-based?

The migration step is UI-based. The verification step is ADB-based.

In practice, install the app, run it once, close it, move it through:

```text
Settings → Storage → Apps → Storage location
```

Then use ADB to check what actually changed.

---

## 11. Why install the app first, then launch it once, then close it?

That sequence gives the app a real initialized state before testing. Without it, package state and data behavior may be incomplete, and the later ADB comparison is less useful.

---

## 12. Why can QQ Music move, but offline songs still stay in internal storage?

Because the system can move the app package, while the app itself may still choose to write offline songs into shared internal storage.

That is not a contradiction. It is a separation between package placement and app data behavior.

---

## 13. Does the “Modify system settings” permission matter?

QQ Music may expose a permission path such as:

```text
App info → Advanced → Modify system settings
```

For the storage behavior tested here, there is currently no sufficient evidence that this permission decides whether offline songs can move to external storage. It is recorded as an observation, not as a confirmed cause.

---

## 14. Can AI solve this problem?

AI can help separate the layers, design the test flow, interpret ADB output, and write clearer notes. It cannot override Android storage rules or force every app to store offline data where the user wants.

For this problem, AI is useful as a testing and documentation aid, not as a replacement for device-side verification.

---

## 15. What is the practical working solution right now?

For the verified Sony Walkman + QQ Music case, the practical storage role is:

```text
Download -> internal storage
Store -> SD card
Play -> read from SD card
```

In other words, treat internal storage as the system / app disk, and treat the 1TB SD card as the music-library disk. Manual relocation may still be needed.

---

## 16. Is this repository a universal answer for all DAP devices and all streaming apps?

No. It is a verified Sony Walkman case, a repeatable method, and a place for later compatibility reports.

For other devices and apps, check the [Compatibility Matrix](compatibility.md). Unverified entries should be treated as open questions, not final conclusions.
