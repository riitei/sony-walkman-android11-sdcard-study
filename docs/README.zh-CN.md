# Android 11+ DAP SD 卡存储行为研究：以 Sony Walkman 为例

[返回主 README](../README.md) · [繁體中文](README.zh-TW.md) · [English](README.en.md) · [FAQ](faq.md) · [相容性矩阵](compatibility.md)

---

## 前言

这篇文章整理 **Sony Walkman + Android 11 + QQ 音乐** 的实测结果。我原本只是想确认 QQ 音乐能不能放到 1TB SD 卡，后来才发现问题不在“能不能看到 SD 卡”，而是在 Android 11 之后，系统、UI、App 数据三层常常不是同一件事。

测试中看到的情况很直接：系统能识别 SD 卡，QQ 音乐 App 本体也能通过 Sony 的设置界面搬移；但离线歌曲数据不一定跟着搬。Windows 通过 MTP 看到的容量，也不一定等于 Android 底层实际挂载的外接卷状态。

所以这篇不是单纯的搬 App 教学，而是一份存储行为记录。重点是确认：App 本体、App 下载数据、MTP 显示容量，到底是不是同一层。

---

## 问题定义

这次主要碰到几个实际问题：SD 卡明明已被系统识别，App 却仍然占用内置存储；App 明明可以搬移，离线歌曲或缓存却没有跟着搬；Sony 设置页看起来有更大的总容量，但 Windows / MTP 仍只看到约 103GB。

这些现象不能直接混成一句“SD 卡有没有成功”。比较准确的拆法是：

- 系统层：Android 是否看得到外接 SD 卡
- UI 层：设置画面是否允许搬移 App
- App 数据层：离线歌曲、缓存、下载数据是否真的写到 SD 卡

本文只把这三层拆开验证，不把 UI 显示成功直接当成数据搬移成功。

---

## 目前验证范围

目前已实测的范围是 Sony Walkman、Android 11、QQ 音乐，工具是 ADB 加 Sony 原生设置界面。已确认的结果是：**QQ 音乐 App 本体可搬，但离线歌曲数据不保证跟着搬。**

其他 Android 11 以上 DAP、国行版 / 国际版差异，以及网易云、汽水音乐、Apple Music、Spotify、Tidal、KKBOX 等 App，目前只列为后续回报项目，不在这篇直接下定论。

简单说，这篇是以 Sony Walkman 为例的技术验证记录，不是所有 Android DAP 与所有串流 App 的通用结论。

---

## 先看结论

目前确认的结果如下：

1. Sony Walkman 的外接 1TB SD 卡可以被系统识别，也可以成为 App 可选择的存储位置。
2. QQ 音乐 App 本体可以通过系统 UI 手动搬移。
3. QQ 音乐的离线歌曲数据不一定跟着搬移。
4. 比较稳的验证路线是：USB 调试、ADB 检查、Sony UI 手动搬移，再用 ADB 确认数据变化。
5. 本文不把 root 当前提，也不把重新格式化 SD 卡列成普通用户的主流程。

核心判断是：**App 本体搬移成功，不代表 App 下载数据搬移成功。**

---

## 研究目的

我一开始的目标是解决 Walkman 内置 128GB 不够用的问题。高音质音乐、串流离线数据和长期缓存都放在内置存储时，很快就会碰到容量限制，也会让内置存储长期承受写入压力。

直觉上，插入 1TB SD 卡后，应该能把主要音乐数据移出去。但 Android 11 的存储分层加上 Sony 的 OEM 设计，使“系统看得到容量”和“App 真的把数据写过去”变成两件事。这篇文章就是把这个落差记录下来。

---

## 测试环境

| 项目 | 内容 |
| :--- | :--- |
| 设备 | Sony Walkman |
| 系统 | Android 11 |
| 内置存储 | 约 128GB |
| 外接存储 | microSD 1TB |
| 权限条件 | non-root |
| 工具 | ADB（Windows PowerShell） |
| 验证 App | QQ 音乐 |

---

## 前置条件

先在 Sony Walkman 开启开发者选项与 USB 调试，用 USB 线接上电脑，并在设备画面允许 USB 调试授权。接着用以下命令确认连接：

```bash
adb devices
```

正常情况会看到 `device`。如果显示 `unauthorized`，代表还没在 Walkman 上允许授权，需要回到设备端按下允许。

---

## 底层模型：先分清楚三层

Android 11 的存储问题不能只看单一路径。我这次测试时，最容易混淆的是下面三层。

第一层是共享存储层，常见路径是：

```text
/storage/emulated/0
```

这是用户在文件管理器中最常看到的“内部共享存储空间”，也是很多 App 显示下载路径时会指向的位置。Windows 通过 MTP 通常也最容易看到这一层。

第二层是外接卷 / 扩展卷层，常见路径是：

```text
/mnt/expand/<UUID>
```

外接 SD 卡被系统挂载后，Package Manager 可能把 App 本体导向这里。但这一层不一定能被一般文件管理器直接操作，也不一定会像普通 USB 闪存盘一样被 Windows 直接看到。

第三层是 App Storage 管理层，也就是 Sony / Android 设置里的入口：

```text
设置 → 存储空间 → 应用程序 → 存储位置
```

这个入口可能改变 App 本体放在哪里，但不保证 App 自己的下载数据、离线歌曲、缓存和自建文件夹都会一起搬。

所以这次测试采用的基本判断是：App 安装位置、App 显示的下载路径、App 真正写入的离线数据路径，可能是三套不同逻辑。

---

## 主流程：目前确认的可用方法

最后比较稳的路线不是 root，也不是先把 SD 卡重新格式化成 private volume，而是先走 Sony 原生 UI，再用 ADB 验证。

操作顺序是：先开启 USB 调试，用 ADB 看当前存储状态；接着安装 QQ 音乐，打开一次后关闭；再到 Sony 设置界面手动搬移 App；最后用 ADB 对比 package 位置与离线数据写入位置。

### Step 1：确认系统是否看得到 SD 卡

```bash
adb shell sm list-disks
adb shell df -h
```

这一步只是建立基准状态，确认系统有看到外接 SD 卡，也确认内置与 SD 卡当前的容量结构。`sm list-disks` 显示的 `disk:179,192` 是 StorageManager 的磁盘设备代号，不是挂载路径，也不是内置存储本体。

### Step 2：安装并执行一次 QQ 音乐

本文案例中，QQ 音乐一开始先安装在 Walkman 本体上。安装后先打开一次，让它完成初始化，再关闭 App。这样后续才有 package 状态与数据行为可以观察。

### Step 3：从 Sony UI 手动搬移 App

进入：

```text
设置 → 存储空间 → 应用程序 → 存储位置
```

如果画面允许选择存储位置，就把 QQ 音乐搬到 SD 卡方向的存储位置。这一步只能代表 App 本体可能搬移，还不能代表离线歌曲也搬移。

### Step 4：用 ADB 验证 App 本体位置

先查 volume UUID：

```bash
adb shell dumpsys package com.tencent.qqmusic | findstr volumeUuid
```

再查 APK 路径：

```bash
adb shell pm path com.tencent.qqmusic
```

如果 `volumeUuid` 指向外接卷，且 `pm path` 也指向对应外接存储路径，才可以说 App 本体搬移成功。

### Step 5：验证离线数据是否跟着搬移

下载歌曲前后，各跑一次：

```bash
adb shell df -h /storage/emulated
adb shell df -h /mnt/expand/<UUID>
```

如果下载后 `/storage/emulated` 空间明显减少，而 `/mnt/expand/<UUID>` 几乎不变，代表离线数据仍主要写在内置共享存储层。反过来，如果外接卷容量有明显变化，才比较能支持“离线数据进入 SD 卡”这个判断。

---

## QQ 音乐案例：实测结果

这次的实际顺序是：安装 QQ 音乐、打开一次、关闭 App、到 Sony UI 手动搬移，再用 ADB 检查 package 状态和数据写入位置。

App 本体部分，如果 `dumpsys package` 的 `volumeUuid` 指向外接卷，且 `pm path` 指向外接路径，可以判定 QQ 音乐 App 本体已搬移成功。

但离线数据部分要看下载前后的 `df -h`。我这次观察到的重点是：App 本体搬移成功，不等于离线歌曲数据跟着搬。这就是本研究最重要的结论。

---

## 最终可用解法

系统层没有把 QQ 音乐的离线下载数据自动导到 1TB，但实务上仍有可用方案：先让 QQ 音乐下载，再把音乐数据文件夹搬到 SD 卡上的音乐区，例如：

```text
SD卡/Music/qqmusic/
```

这里的限制比较像是 Android 11 限制“写入位置”，但不一定完全限制“读取位置”。最后形成的使用模式是：

```text
下载 -> 128GB
存放 -> 1TB
播放 -> 从 1TB 读
```

这不是系统层完全整合成功，但对把 Walkman 当音乐机使用来说，已经是可用方案。

---

## 延伸：把 Walkman 当成纯音乐机

如果把 Walkman 定位成音乐机，而不是一般 Android 手机，存储策略会比较清楚：128GB 内置存储保留给 Android 系统、App 本体、必要缓存与少量临时数据；1TB SD 卡则保留给本地音乐主库、搬移后的音乐数据和长期收藏曲库。

也就是：**128GB 当系统盘，1TB 当音乐盘。**

这个策略的前提是接受下载层与存放层分离，不再要求 QQ 音乐天生自动把所有离线歌曲直接写进 1TB。

---

## 这次测试的结论

这次测试可以收敛成五点：Sony Walkman（Android 11）在 non-root 下可通过 UI 手动搬移 App；ADB 可以验证 App 本体实际存储位置与数据写入位置；QQ 音乐 App 本体可搬，但离线歌曲数据不保证跟着搬；最后可行方案不是让系统完全合并存储，而是接受“系统层分离、使用层解决”；本文目前是已验证案例与可重现方法的整理，不是所有 Android 11+ DAP 与所有串流 App 的最终定论。

整体来看，Sony Walkman 的 1TB SD 卡可以成为实际音乐盘，但不会自然成为所有 App 离线数据的主存储层。至少在 QQ 音乐案例中，App 本体搬移成功，离线歌曲搬移失败，最后要通过数据搬移与使用策略调整，才比较能实际利用 1TB。

---

## 方法整理

这次方法里有几个正确方向：先安装 App，再执行、关闭、搬移；用 ADB 验证 package 与存储行为，而不是只看 UI；把 UI 搬移与底层验证分开看。

容易写偏的地方也要保留：`sm set-force-adoptable true` 和 `sm partition ... private` 不适合写成所有人必做主流程；`disk:179,192` 不能写成内置空间本体；UI 可搬移也不能直接等同于离线数据一定会搬。

因此，本文主流程应该是“UI 手动搬移 + ADB 验证”，探索命令只留在研究历程中。

---

## 研究历程中的探索命令

测试过程中曾执行以下命令：

```bash
adb shell sm set-force-adoptable true
adb shell sm partition disk:179,192 private
adb reboot
adb shell sm list-volumes all
```

这些命令用来观察 adoptable storage、private volume 和底层 volume 状态。它们提供了研究线索，也帮助厘清系统层结构；但因为牵涉格式化与重建存储，风险较高，而且不是最后确认的稳定路线，所以不列为普通用户主流程。

比较合适的定位是：这些命令属于研究历程中的探索步骤，不是普通用户必做操作。

---

## 关于“修改系统设置”权限

QQ 音乐在“应用程序信息 → 高级 → 修改系统设置”的权限，主要影响它是否能改动某些系统层设置，例如显示层或控制层行为。就这次主题而言，目前没有足够证据显示这个权限会直接决定 QQ 音乐的离线歌曲是否能搬到外接存储。

所以这个权限可以记录为观察项，但不适合直接当作主因。

---

## 设备相容性与社区回报

目前只实测 Sony Walkman、Android 11、QQ 音乐。后续如果要扩充，可以把其他 Android 11 以上 DAP、国行版 / 国际版差异，以及其他串流 App 的存储行为放进同一套表格比较。

| 串流 App | 测试设备 (系统) | UI 可否手动搬移 App | 离线数据是否跟着搬移 | 备注 |
| :--- | :--- | :--- | :--- | :--- |
| QQ 音乐 | Sony Walkman (Android 11) | ✅ 可 | ❌ 否 | App 本体可搬，歌曲数据仍主要写入共享存储层 |
| 网易云音乐 | 待社区回报 |  |  |  |
| 汽水音乐 | 待社区回报 |  |  |  |
| Apple Music | 待社区回报 |  |  |  |
| Spotify | 待社区回报 |  |  |  |
| Tidal | 待社区回报 |  |  |  |
| KKBOX | 待社区回报 |  |  |  |

这一节的目的不是现在就替所有设备下定论，而是留下可扩充、可共编、可回报的入口。

---

## Windows / MTP 为什么还是只看到 103GB

本案例中，Sony 的设置页可能显示总容量约 1.13TB，但 Windows 通过 MTP 仍只看到约 103/104GB。原因是 MTP 主要 expose 的仍是共享存储层；外接扩展卷不一定会像一般 portable SD 一样，直接当普通磁盘暴露给电脑。

所以 Windows 看不到 1TB，不代表 1TB 没生效。它只代表该外接卷不是以一般外接盘逻辑提供给 MTP。
