# Sony Walkman Android 11 SD 卡儲存行為研究

## 前言

在 Android 11 之後，外接 SD 卡不再是早期 Android 時代那種「App 想放什麼就放什麼」的自由擴充空間。  
系統把儲存分成多層：一層是使用者熟悉的共享儲存，例如 `/storage/emulated/0`；另一層是外接卡被 adopt 後形成的 private/internal volume，例如 `/mnt/expand/<UUID>`。對使用者來說，兩者都可能被 UI 統稱為「儲存空間」；但對 App 來說，這兩層的權限、可見性與用途並不相同。

也因此，Android 11 的真實問題不是「SD 卡還能不能用」，而是：

> 哪一類資料能搬、哪一類資料不能搬、誰在決定搬移規則。

本篇文章的目的，不是單純介紹 ADB 指令，也不是只討論 QQ 音樂。  
本篇要處理的問題是：

- Sony Walkman（Android 11）外接 1TB SD 卡後，系統究竟能不能把它當成 App 可用空間
- App 本體與 App 離線資料，是否會一起搬移
- 哪些方法只是 UI 看起來成功，哪些才是底層真的成功

本文最後會用 QQ 音樂做案例。  
原因不是因為它是唯一特殊的 App，而是它很適合拿來驗證 Android 11 儲存分層的邊界：**App 本體可以搬，不等於離線歌曲也會跟著搬。**

## 目的

本次研究的真正目的有三個：

### 1. 解決容量限制
Sony Walkman 內建儲存約 128GB，對高音質音樂、串流離線資料與長期使用來說偏小。  
希望讓 1TB SD 卡成為主要音樂承載空間。

### 2. 解決寫入壓力集中在內建儲存的問題
如果大量下載、刪除、重建快取都長期寫在內建儲存，內建空間會同時承受容量與寫入壓力。  
因此希望把主要音樂資料流量導向可替換的 1TB SD 卡。

### 3. 驗證 Sony Walkman 這類音樂設備在 Android 11 下的真實限制
直覺上，1TB SD 卡插進去之後，應該可以讓音樂 App 直接使用大容量儲存。  
但實際上，Android 11 的儲存分層與 OEM 設計，可能讓「可見容量」與「可實際使用容量」不同。

本篇的核心問題可以寫成一句話：

> 在 Sony Walkman（Android 11）上，1TB SD 卡究竟能不能被 App 真正當作主要可用儲存空間？

## 測試環境

- 裝置：Sony Walkman
- 系統：Android 11
- 內建儲存：約 128GB
- 外接儲存：microSD 1TB
- 權限條件：non-root
- 工具：ADB（Windows PowerShell）
- 驗證 App：QQ 音樂

## 前置條件

在執行 ADB 指令前，需先完成以下設定：

1. 在 Sony Walkman 開啟 **開發人員選項**
2. 開啟 **USB 偵錯**
3. 以 USB 線連接電腦
4. 在裝置畫面上允許 **USB 偵錯授權**

可先用以下指令確認：

```bash
adb devices
```

若正常，應顯示：

```text
device
```

若顯示 `unauthorized`，代表裝置尚未完成授權，需回到裝置端確認 USB 偵錯提示視窗並按下允許。

## 儲存架構：先分清楚三層

在正式操作前，必須先把 Android 11 的儲存結構拆開看。

### 1. 共享儲存層
典型路徑：

```text
/storage/emulated/0
```

這一層通常是：

- 使用者在檔案管理器中熟悉的「內部共用儲存空間」
- App 最常顯示給使用者看的下載路徑
- Windows 透過 MTP 最容易看到的那一層

很多人會誤以為這一層就是全部儲存。  
但它其實只是 App 和使用者最常接觸到的一層。

### 2. adopted/private volume 層
典型路徑：

```text
/mnt/expand/<UUID>
```

這一層通常是：

- 外接 SD 卡被 adopt 成 private/internal 後的真實掛載點
- Package Manager 可能把 App 本體搬到這裡
- 一般使用者不一定能直接透過檔案總管操作

這一層是「系統層」真正看到的 adopted storage。

### 3. App Storage 管理層
這一層不是單一路徑，而是系統 UI 提供的管理入口，例如：

```text
儲存空間 -> 內部共用儲存空間 -> 應用程式 -> 應用程式名稱 -> 已使用的空間儲存空間
```

這一層可以影響：

- App 本體放在哪個 volume
- 部分 app 管理資料的位置

但它不保證：

- App 自己的下載資料
- 離線歌曲
- 快取
- 自建資料夾

也會跟著搬。

一句話總結：

> App 安裝位置、App 顯示的下載路徑、App 真正寫入的離線資料路徑，可能是三套不同邏輯。

## 方法一：確認初始狀態

### Step 1：確認裝置可被 ADB 正常辨識

```bash
adb devices
```

預期輸出應為：

```text
device
```

如果看到 `unauthorized`，先處理 USB 偵錯授權，否則後續所有指令都無法執行。

### Step 2：確認 Sony 是否正式開放 adoptable

```bash
adb shell sm has-adoptable
```

本案例輸出為：

```text
false
```

這代表：

- Sony 原廠預設沒有正式打開 adoptable storage
- 但不代表底層能力完全不存在

這是一個很重要的訊號：  
**前台沒有正式支援，不等於底層不能測。**

### Step 3：確認系統是否看得到 SD 卡

```bash
adb shell sm list-disks
```

本案例得到：

```text
disk:179,192
```

表示系統有抓到這張 SD 卡。

### Step 4：確認原始掛載狀態

```bash
adb shell df -h
```

在原始狀態下，本案例可看到大致如下的結構：

- `/data` 對應約 104G
- portable SD 卡會掛在類似：
  - `/storage/6006-A885`
  - 容量約 922G

這代表：

- 內建儲存與 SD 卡仍是分離的
- SD 卡目前是 **portable external storage**
- 還沒有變成 adopted/private volume

## 方法二：把 SD 卡做成 adopted/private volume

### Step 1：強制開 adoptable 能力

```bash
adb shell sm set-force-adoptable true
```

### Step 2：把 SD 卡 partition 成 private

```bash
adb shell sm partition disk:179,192 private
```

請把 `disk:179,192` 換成你自己 `sm list-disks` 查到的值。

⚠️ 注意：  
這一步會清空 SD 卡資料，操作前必須先備份。

### Step 3：重開機

```bash
adb reboot
```

## 方法三：驗證 adopted/private 是否真的建立成功

### Step 1：查看 volume 狀態

```bash
adb shell sm list-volumes all
```

成功時應會出現類似：

```text
private:179,194 mounted 74f5042c-b29b-4784-aa1f-a836d699bca8
```

### Step 2：查看真實掛載點

```bash
adb shell df -h
```

成功後，本案例看到：

```text
/dev/block/dm-13  922G  ...  /mnt/expand/74f5042c-b29b-4784-aa1f-a836d699bca8
```

這表示：

- 1TB SD 卡已被系統 adopt 成 private/internal volume
- 真實掛載點位於：

```text
/mnt/expand/<UUID>
```

這一步只能證明一件事：

> 系統層成功。

它還不能證明 App 所有資料都會跟著搬過去。

## 方法四：從 Sony UI 搬移 App 本體

接著不要急著判定「整個任務成功」。  
這時候需要進入 Sony 的 UI 路徑：

```text
儲存空間 -> 內部共用儲存空間 -> 應用程式 -> 應用程式名稱 -> 已使用的空間儲存空間
```

如果在這裡看到搬移入口，代表：

- Sony 的 App Storage 管理層承認 adopted volume 可作為 app 儲存位置
- 至少在「App 本體」層級，系統願意把該 App 導向 1TB volume

這一步是 UI 層證據。  
它很重要，但仍然不等於下載資料會跟著搬。

## 方法五：用 ADB 驗證 App 本體是否真的搬過去

先檢查 volume UUID：

```bash
adb shell dumpsys package com.tencent.qqmusic | findstr volumeUuid
```

再檢查 APK 路徑：

```bash
adb shell pm path com.tencent.qqmusic
```

若成功，應看到兩件事：

1. `volumeUuid` 指向 adopted volume 的 UUID
2. `pm path` 指向：

```text
/mnt/expand/<UUID>/app/.../base.apk
```

這代表：

> QQ 音樂 App 本體確實已成功搬到 1TB adopted/private volume。

這一步是 Package Manager 層的硬證據。  
也就是說，UI 顯示不是假象，App 本體確實被搬過去了。

## 方法六：真正驗證「離線歌曲」到底寫去哪裡

這一步是全篇最關鍵的驗證。

### Step 1：下載前先記錄

```bash
adb shell df -h /storage/emulated
adb shell df -h /mnt/expand/74f5042c-b29b-4784-aa1f-a836d699bca8
```

### Step 2：下載固定量內容
例如下載約 500MB～1GB 的離線歌曲。

### Step 3：下載後再記錄

```bash
adb shell df -h /storage/emulated
adb shell df -h /mnt/expand/74f5042c-b29b-4784-aa1f-a836d699bca8
```

## 判讀規則

### 情況 A
如果只有：

```text
/storage/emulated
```

的已用空間增加，而：

```text
/mnt/expand/<UUID>
```

幾乎不變，代表：

> 離線歌曲仍主要寫在內建共享儲存層。

### 情況 B
如果只有 `/mnt/expand/<UUID>` 增加，代表：

> 離線歌曲確實進入 1TB adopted/private volume。

### 情況 C
如果兩邊都增加，代表：

> 混合寫入。

## 本案例實測結果

本案例前後對照結果如下：

### 下載前
```text
/storage/emulated   104G   50G   54G
/mnt/expand/...     922G  6.6G  915G
```

### 下載後
```text
/storage/emulated   104G   50G   53G
/mnt/expand/...     922G  6.6G  915G
```

### 結果判讀
這表示：

- `/storage/emulated` 的可用空間減少
- `/mnt/expand/<UUID>` 幾乎沒有變化

因此可下明確結論：

> QQ 音樂的 App 本體雖然搬到了 1TB adopted/private volume，但離線歌曲資料沒有跟著搬。

這就是本研究的核心結果。

## 為什麼會出現「App 搬了，歌沒搬」

因為這是兩套不同邏輯。

### 系統能控制的
- App 安裝位置
- Package Manager 對 APK 的管理
- adopted/private volume 的建立與掛載

### App 自己控制的
- 下載資料寫入位置
- 離線歌曲資料夾
- 快取與內部邏輯

所以即使：

- `pm path` 已經在 `/mnt/expand/...`
- `volumeUuid` 也已經指到 adopted volume

也不保證：

- 離線歌曲會進 `/mnt/expand/...`

這不是矛盾，而是：

> App 安裝位置與 App 離線資料位置，本來就可能是分離管理。

## Windows / MTP 為什麼還是只看到 103GB

這一點也很容易誤判。

本案例中：

- Sony 的設定頁可能會顯示總容量約 1.13TB
- 但 Windows 透過 MTP 仍只看到約 103/104GB

原因是：

- MTP 主要 expose 的仍是共享儲存層
- adopted/private volume 不會像一般 portable SD 一樣直接當普通磁碟暴露給電腦

所以：

> Windows 看不到 1TB，不代表 1TB 沒生效。  
> 它只是代表 adopted/private volume 不是以一般外接碟邏輯提供給 MTP。

## QQ 音樂案例：最後怎麼變成可用

雖然系統層沒有真正把 QQ 音樂的離線下載資料導到 1TB，  
但最後仍找到一個實用上的可行方案。

### 實務作法
- 先讓 QQ 音樂下載
- 再把音樂資料夾搬到 SD 卡上的音樂區，例如：

```text
SD卡/Music/qqmusic/
```

### 本質
Android 11 這裡的限制比較像是：

- 限制你「寫哪裡」
- 但在很多情況下，不一定完全限制你「從哪裡讀」

所以最後形成的可用模式是：

```text
下載 -> 128GB
存放 -> 1TB
播放 -> 從 1TB 讀
```

這不是系統層完全整合成功。  
但對音樂機用途來說，已經是可用且穩定的方案。

## 延伸：把 Walkman 當成純音樂機

既然最終定位是音樂機，而不是一般 Android 手機，  
那後續最合理的儲存策略應該是：

### 128GB 內建
保留給：

- Android 系統
- App 本體
- 必要快取
- 少量臨時資料

### 1TB SD 卡
保留給：

- 本地音樂主庫
- 搬移後的音樂資料
- 長期收藏曲庫

也就是：

> 128GB 當系統盤，1TB 當音樂盤。

這句話成立的前提是：

- 你接受下載層與存放層分離
- 不再要求 QQ 音樂天生自動把所有離線歌曲直接寫進 1TB

## 最終結論

本次研究可收斂成四點：

1. Sony Walkman（Android 11）可透過 ADB 建立 adopted/private volume
2. QQ 音樂 App 本體可搬到 1TB adopted volume
3. 但 QQ 音樂離線歌曲資料不保證跟著搬移
4. 最終可行方案不是讓系統完全合併儲存，而是接受「系統層分離、使用層解決」

一句話總結：

> Sony Walkman 的 1TB SD 卡可以成為實際音樂盤，但不會自然成為所有 App 離線資料的主儲存層；至少在 QQ 音樂案例中，App 本體搬移成功，離線歌曲搬移失敗，最後必須透過資料搬移與使用策略調整來實際利用 1TB。
