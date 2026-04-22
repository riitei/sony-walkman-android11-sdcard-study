# FAQ

This FAQ is written to answer the most common, searchable problems around **Android 11+ DAP SD card storage behavior**.

[Back to main README](../README.md) · [Traditional Chinese](README.zh-TW.md) · [Simplified Chinese](README.zh-CN.md) · [English article](README.en.md) · [Compatibility Matrix](compatibility.md)

---

## 1. The SD card is recognized, but the app still uses internal storage. Why?

Because these are not the same layer.

On Android 11+, at least three things can behave differently:

- the **system** can see the SD card,
- the **UI** can allow app migration,
- but the **app's own offline data** may still write to shared internal storage.

So “SD card recognized” does **not** automatically mean “offline songs now save to SD card.”

---

## 2. The app moved successfully. Why did the data not follow?

Because **app package location** and **app data location** are not guaranteed to be the same.

A package can move to an external-side volume, while offline songs, cache, or downloads still remain in:

```text
/storage/emulated/0
```

This is one of the main findings of this repository.

---

## 3. Does moving an app to SD card also move offline music data?

Not necessarily.

That is exactly the problem this repository studies.

Verified Sony Walkman + QQ Music result:

- app package moved: **yes**
- offline song data moved with it: **no**

So the correct answer is:

> **App migration success does not guarantee offline-data migration success.**

---

## 4. Why does Windows / MTP still show only about 103GB instead of the full 1TB?

Because Windows MTP usually exposes the **shared storage layer**, not every lower-level expanded volume.

In this repository's tested case:

- the device may report total capacity around 1.13TB,
- but Windows still shows only the internal shared-storage side.

So:

> Windows not showing 1TB does not mean the SD card is useless. It means the expanded volume is not exposed in the same way as a normal portable disk.

---

## 5. What is `/storage/emulated/0`?

It is the familiar **shared internal storage layer** used by many Android apps and shown by many file managers.

Apps often present this layer as the visible download destination.

But it is not the same thing as every lower-level storage volume available in the system.

---

## 6. What is `/mnt/expand/<UUID>`?

It is a lower-level **expanded / external volume path** seen by the system.

In some cases, the package manager may place an app package there.

However, that still does **not** guarantee that the app's offline songs or cache also move there.

---

## 7. What does `disk:179,192` mean?

It is a **StorageManager disk identifier** returned by:

```bash
adb shell sm list-disks
```

It is **not** a mount path.
It is **not** the internal storage volume itself.

It is simply an identifier used by Android's storage management layer.

---

## 8. Is root required for this study?

No.

The main workflow validated in this repository is **non-root**:

1. enable USB debugging,
2. inspect with ADB,
3. install and initialize the app,
4. manually move the app in system UI,
5. verify package and data behavior with ADB.

---

## 9. Are these commands mandatory?

```bash
adb shell sm set-force-adoptable true
adb shell sm partition disk:179,192 private
adb reboot
adb shell sm list-volumes all
```

No, not as the default reader workflow.

They belong to the **research trail** of the study.
They helped clarify low-level storage behavior, but they are **not** treated in this repository as the default instruction set for ordinary readers.

Why not?

- they can involve storage reconstruction,
- they can imply formatting risk,
- they are not the final stable workflow validated in this repo.

---

## 10. Is the main workflow UI-based or command-based?

For ordinary readers, the main workflow is:

- **UI-based app migration**
- plus **ADB-based verification**

That means:

1. install the app,
2. run it once,
3. close it,
4. move it through:

```text
Settings → Storage → Apps → Storage location
```

5. verify what actually changed with ADB.

So the correct answer is:

> **The migration step is UI-based; the verification step is ADB-based.**

---

## 11. Why install the app first, then launch it once, then close it?

Because without that sequence:

- there may be no meaningful initialized package state,
- no real app data behavior to inspect,
- and no meaningful comparison point for later ADB checks.

This repository treats that order as part of the valid method.

---

## 12. Why can QQ Music move, but offline songs still stay in internal storage?

Because the app package and offline data are controlled by different layers.

The system may allow the **package** to move.
The app may still choose to write **offline songs** into shared internal storage.

This is not necessarily a contradiction. It is a separation of responsibilities.

---

## 13. Does the "Modify system settings" permission matter?

QQ Music may expose a permission path like:

```text
App info → Advanced → Modify system settings
```

At present, there is **no sufficient evidence** in this repository that this permission directly determines whether offline songs can move to external storage.

So it is recorded as an observation item, not as a confirmed cause.

---

## 14. Can AI solve this problem?

AI can help with:

- separating the problem into layers,
- designing a reproducible verification workflow,
- interpreting ADB outputs,
- distinguishing UI appearance from actual storage behavior,
- writing clear documentation and compatibility records.

But AI cannot magically override Android storage rules or app-specific implementation limits.

So the honest answer is:

> **AI can help you understand, test, document, and narrow the problem. It cannot guarantee that every app will obey the storage behavior you want.**

---

## 15. What is the practical working solution right now?

In the verified Sony Walkman + QQ Music case, the practical solution is:

- treat internal storage as the **system/app disk**,
- treat the 1TB SD card as the **music-library disk**,
- accept that download and storage may be separated,
- and manually relocate music data when needed.

In short:

```text
Download -> internal storage
Store -> SD card
Play -> read from SD card
```

---

## 16. Is this repository a universal answer for all DAP devices and all streaming apps?

No.

It is currently:

- a **verified Sony Walkman case**,
- a **repeatable method**,
- and a **research entry point** for future compatibility reports.

For other devices and apps, see:

- [Compatibility Matrix](compatibility.md)

and treat unverified entries as open questions, not final conclusions.
