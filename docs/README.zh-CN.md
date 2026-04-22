# Android 11+ 随身听（DAP）SD 卡存储行为研究

## 前言

在 Android 11 之后，SD 卡不再是传统意义上的“扩展存储”，而是被系统分层管理：

- `/storage/emulated/0`（共享存储）
- `/mnt/expand/<UUID>`（扩展卷）

核心问题：

> App 安装位置 与 App 数据位置 是否一致？

---

## 核心结论

- App 可以通过系统 UI 移动到 SD 卡
- 离线数据（如音乐）不一定跟随
- 限制来自 Android 架构，而非厂商策略

---

## 方法概述

- USB 调试 + ADB 验证
- 系统 UI 手动迁移 App
- `df -h` 对比存储变化

---

## QQ 音乐案例

- App 成功迁移
- 离线歌曲仍写入内部存储

结论：

> App ≠ 数据

---

## 适用范围

适用于大多数 Android 11+ DAP：

- Sony
- iBasso
- HiBy
- FiiO
