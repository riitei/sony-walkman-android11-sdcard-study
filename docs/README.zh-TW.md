# Android 11+ DAP SD 卡儲存行為研究：以 Sony Walkman 為例

[返回主 README](../README.md) · [简体中文](README.zh-CN.md) · [English](README.en.md) · [FAQ](faq.md) · [相容性矩陣](compatibility.md)

---

## 前言

這篇文章整理 **Sony Walkman + Android 11 + QQ 音樂** 的實測結果。我原本只是想確認 QQ 音樂能不能放到 1TB SD 卡，後來才發現問題不在「能不能看到 SD 卡」，而是在 Android 11 之後，系統、UI、App 資料三層常常不是同一件事。

測試中看到的情況很直接：系統能辨識 SD 卡，QQ 音樂 App 本體也能透過 Sony 的設定介面搬移；但離線歌曲資料不一定跟著搬。Windows 透過 MTP 看到的容量，也不一定等於 Android 底層實際掛載的外接卷狀態。

所以這篇不是單純的搬 App 教學，而是一份儲存行為紀錄。重點是確認：App 本體、App 下載資料、MTP 顯示容量，到底是不是同一層。

---

## 問題定義

這次主要碰到幾個實際問題：SD 卡明明有被系統辨識，App 卻仍然吃內建儲存；App 明明可以搬移，離線歌曲或快取卻沒有跟著搬；Sony 設定頁看起來有更大的總容量，但 Windows / MTP 仍只看到約 103GB。

這些現象不能直接混成一句「SD 卡有沒有成功」。比較準確的拆法是：

- 系統層：Android 是否看得到外接 SD 卡
- UI 層：設定畫面是否允許搬移 App
- App 資料層：離線歌曲、快取、下載資料是否真的寫到 SD 卡

本文只把這三層拆開驗證，不把 UI 顯示成功直接當成資料搬移成功。

---

## 目前驗證範圍

目前已實測的範圍是 Sony Walkman、Android 11、QQ 音樂，工具是 ADB 加 Sony 原生設定介面。已確認的結果是：**QQ 音樂 App 本體可搬，但離線歌曲資料不保證跟著搬。**

其他 Android 11 以上 DAP、國行版 / 國際版差異，以及網易雲、汽水音樂、Apple Music、Spotify、Tidal、KKBOX 等 App，目前只列為後續回報項目，不在這篇直接下定論。

簡單說，這篇是以 Sony Walkman 為例的技術驗證紀錄，不是所有 Android DAP 與所有串流 App 的通用結論。

---

## 先看結論

目前確認的結果如下：

1. Sony Walkman 的外接 1TB SD 卡可以被系統辨識，也可以成為 App 可選擇的儲存位置。
2. QQ 音樂 App 本體可以透過系統 UI 手動搬移。
3. QQ 音樂的離線歌曲資料不一定跟著搬移。
4. 比較穩的驗證路線是：USB 偵錯、ADB 檢查、Sony UI 手動搬移，再用 ADB 確認資料變化。
5. 本文不把 root 當前提，也不把重新格式化 SD 卡列成一般使用者的主流程。

核心判斷是：**App 本體搬移成功，不代表 App 下載資料搬移成功。**

---

## 研究目的

我一開始的目標是解決 Walkman 內建 128GB 不夠用的問題。高音質音樂、串流離線資料和長期快取都放在內建儲存時，很快就會碰到容量限制，也會讓內建儲存長期承受寫入壓力。

直覺上，插入 1TB SD 卡後，應該能把主要音樂資料移出去。但 Android 11 的儲存分層加上 Sony 的 OEM 設計，使「系統看得到容量」和「App 真的把資料寫過去」變成兩件事。這篇文章就是把這個落差記錄下來。

---

## 測試環境

| 項目 | 內容 |
| :--- | :--- |
| 裝置 | Sony Walkman |
| 系統 | Android 11 |
| 內建儲存 | 約 128GB |
| 外接儲存 | microSD 1TB |
| 權限條件 | non-root |
| 工具 | ADB（Windows PowerShell） |
| 驗證 App | QQ 音樂 |

---

## 前置條件

先在 Sony Walkman 開啟開發人員選項與 USB 偵錯，用 USB 線接上電腦，並在裝置畫面允許 USB 偵錯授權。接著用以下指令確認連線：

```bash
adb devices
```

正常情況會看到 `device`。如果顯示 `unauthorized`，代表還沒在 Walkman 上允許授權，需要回到裝置端按下允許。

---

## 底層模型：先分清楚三層

Android 11 的儲存問題不能只看單一路徑。我這次測試時，最容易混淆的是下面三層。

第一層是共享儲存層，常見路徑是：

```text
/storage/emulated/0
```

這是使用者在檔案管理器中最常看到的「內部共用儲存空間」，也是很多 App 顯示下載路徑時會指向的位置。Windows 透過 MTP 通常也最容易看到這一層。

第二層是外接卷 / 擴充卷層，常見路徑是：

```text
/mnt/expand/<UUID>
```

外接 SD 卡被系統掛載後，Package Manager 可能把 App 本體導向這裡。但這一層不一定能被一般檔案總管直接操作，也不一定會像普通 USB 隨身碟一樣被 Windows 直接看到。

第三層是 App Storage 管理層，也就是 Sony / Android 設定裡的入口：

```text
設定 → 儲存空間 → 應用程式 → 儲存位置
```

這個入口可能改變 App 本體放在哪裡，但不保證 App 自己的下載資料、離線歌曲、快取和自建資料夾都會一起搬。

所以這次測試採用的基本判斷是：App 安裝位置、App 顯示的下載路徑、App 真正寫入的離線資料路徑，可能是三套不同邏輯。

---

## 主流程：目前確認的可用方法

最後比較穩的路線不是 root，也不是先把 SD 卡重新格式化成 private volume，而是先走 Sony 原生 UI，再用 ADB 驗證。

操作順序是：先開啟 USB 偵錯，用 ADB 看目前儲存狀態；接著安裝 QQ 音樂，開啟一次後關閉；再到 Sony 設定介面手動搬移 App；最後用 ADB 比對 package 位置與離線資料寫入位置。

### Step 1：確認系統是否看得到 SD 卡

```bash
adb shell sm list-disks
adb shell df -h
```

這一步只是建立基準狀態，確認系統有看到外接 SD 卡，也確認內建與 SD 卡目前的容量結構。`sm list-disks` 顯示的 `disk:179,192` 是 StorageManager 的磁碟裝置代號，不是掛載路徑，也不是內建儲存本體。

### Step 2：安裝並執行一次 QQ 音樂

本文案例中，QQ 音樂一開始先安裝在 Walkman 本體上。安裝後先開啟一次，讓它完成初始化，再關閉 App。這樣後續才有 package 狀態與資料行為可以觀察。

### Step 3：從 Sony UI 手動搬移 App

進入：

```text
設定 → 儲存空間 → 應用程式 → 儲存位置
```

如果畫面允許選擇儲存位置，就把 QQ 音樂搬到 SD 卡方向的儲存位置。這一步只能代表 App 本體可能搬移，還不能代表離線歌曲也搬移。

### Step 4：用 ADB 驗證 App 本體位置

先查 volume UUID：

```bash
adb shell dumpsys package com.tencent.qqmusic | findstr volumeUuid
```

再查 APK 路徑：

```bash
adb shell pm path com.tencent.qqmusic
```

如果 `volumeUuid` 指向外接卷，且 `pm path` 也指向對應外接儲存路徑，才可以說 App 本體搬移成功。

### Step 5：驗證離線資料是否跟著搬移

下載歌曲前後，各跑一次：

```bash
adb shell df -h /storage/emulated
adb shell df -h /mnt/expand/<UUID>
```

如果下載後 `/storage/emulated` 空間明顯減少，而 `/mnt/expand/<UUID>` 幾乎不變，代表離線資料仍主要寫在內建共享儲存層。反過來，如果外接卷容量有明顯變化，才比較能支持「離線資料進入 SD 卡」這個判斷。

---

## QQ 音樂案例：實測結果

這次的實際順序是：安裝 QQ 音樂、開啟一次、關閉 App、到 Sony UI 手動搬移，再用 ADB 檢查 package 狀態和資料寫入位置。

App 本體部分，如果 `dumpsys package` 的 `volumeUuid` 指向外接卷，且 `pm path` 指向外接路徑，可以判定 QQ 音樂 App 本體已搬移成功。

但離線資料部分要看下載前後的 `df -h`。我這次觀察到的重點是：App 本體搬移成功，不等於離線歌曲資料跟著搬。這就是本研究最重要的結論。

---

## 最終可用解法

系統層沒有把 QQ 音樂的離線下載資料自動導到 1TB，但實務上仍有可用方案：先讓 QQ 音樂下載，再把音樂資料夾搬到 SD 卡上的音樂區，例如：

```text
SD卡/Music/qqmusic/
```

這裡的限制比較像是 Android 11 限制「寫入位置」，但不一定完全限制「讀取位置」。最後形成的使用模式是：

```text
下載 -> 128GB
存放 -> 1TB
播放 -> 從 1TB 讀
```

這不是系統層完全整合成功，但對把 Walkman 當音樂機使用來說，已經是可用方案。

---

## 延伸：把 Walkman 當成純音樂機

如果把 Walkman 定位成音樂機，而不是一般 Android 手機，儲存策略會比較清楚：128GB 內建儲存保留給 Android 系統、App 本體、必要快取與少量臨時資料；1TB SD 卡則保留給本地音樂主庫、搬移後的音樂資料和長期收藏曲庫。

也就是：**128GB 當系統盤，1TB 當音樂盤。**

這個策略的前提是接受下載層與存放層分離，不再要求 QQ 音樂天生自動把所有離線歌曲直接寫進 1TB。

---

## 這次測試的結論

這次測試可以收斂成五點：Sony Walkman（Android 11）在 non-root 下可透過 UI 手動搬移 App；ADB 可以驗證 App 本體實際儲存位置與資料寫入位置；QQ 音樂 App 本體可搬，但離線歌曲資料不保證跟著搬；最後可行方案不是讓系統完全合併儲存，而是接受「系統層分離、使用層解決」；本文目前是已驗證案例與可重現方法的整理，不是所有 Android 11+ DAP 與所有串流 App 的最終定論。

整體來看，Sony Walkman 的 1TB SD 卡可以成為實際音樂盤，但不會自然成為所有 App 離線資料的主儲存層。至少在 QQ 音樂案例中，App 本體搬移成功，離線歌曲搬移失敗，最後要透過資料搬移與使用策略調整，才比較能實際利用 1TB。

---

## 方法整理

這次方法裡有幾個正確方向：先安裝 App，再執行、關閉、搬移；用 ADB 驗證 package 與儲存行為，而不是只看 UI；把 UI 搬移與底層驗證分開看。

容易寫偏的地方也要保留：`sm set-force-adoptable true` 和 `sm partition ... private` 不適合寫成所有人必做主流程；`disk:179,192` 不能寫成內建空間本體；UI 可搬移也不能直接等同於離線資料一定會搬。

因此，本文主流程應該是「UI 手動搬移 + ADB 驗證」，探索命令只留在研究歷程中。

---

## 研究歷程中的探索命令

測試過程中曾執行以下命令：

```bash
adb shell sm set-force-adoptable true
adb shell sm partition disk:179,192 private
adb reboot
adb shell sm list-volumes all
```

這些命令用來觀察 adoptable storage、private volume 和底層 volume 狀態。它們提供了研究線索，也幫助釐清系統層結構；但因為牽涉格式化與重建儲存，風險較高，而且不是最後確認的穩定路線，所以不列為一般讀者主流程。

比較合適的定位是：這些命令屬於研究歷程中的探索步驟，不是一般讀者必做操作。

---

## 關於「修改系統設定」權限

QQ 音樂在「應用程式資訊 → 進階 → 修改系統設定」的權限，主要影響它是否能改動某些系統層設定，例如顯示層或控制層行為。就這次主題而言，目前沒有足夠證據顯示這個權限會直接決定 QQ 音樂的離線歌曲是否能搬到外接儲存。

所以這個權限可以記錄為觀察項，但不適合直接當作主因。

---

## 設備相容性與社群回報

目前只實測 Sony Walkman、Android 11、QQ 音樂。後續如果要擴充，可以把其他 Android 11 以上 DAP、國行版 / 國際版差異，以及其他串流 App 的儲存行為放進同一套表格比較。

| 串流 App | 測試設備 (系統) | UI 可否手動搬移 App | 離線資料是否跟著搬移 | 備註 |
| :--- | :--- | :--- | :--- | :--- |
| QQ 音樂 | Sony Walkman (Android 11) | ✅ 可 | ❌ 否 | App 本體可搬，歌曲資料仍主要寫入共享儲存層 |
| 網易雲音樂 | 待社群回報 |  |  |  |
| 汽水音樂 | 待社群回報 |  |  |  |
| Apple Music | 待社群回報 |  |  |  |
| Spotify | 待社群回報 |  |  |  |
| Tidal | 待社群回報 |  |  |  |
| KKBOX | 待社群回報 |  |  |  |

這一節的目的不是現在就替所有設備下定論，而是留下可擴充、可共編、可回報的入口。

---

## Windows / MTP 為什麼還是只看到 103GB

本案例中，Sony 的設定頁可能顯示總容量約 1.13TB，但 Windows 透過 MTP 仍只看到約 103/104GB。原因是 MTP 主要 expose 的仍是共享儲存層；外接擴充卷不一定會像一般 portable SD 一樣，直接當普通磁碟暴露給電腦。

所以 Windows 看不到 1TB，不代表 1TB 沒生效。它只代表該外接卷不是以一般外接碟邏輯提供給 MTP。
