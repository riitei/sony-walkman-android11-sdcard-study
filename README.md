# Sony Walkman Android 11 SD Card Study

Research notes and experiments on Sony Walkman Android 11 storage behavior, adoptable storage, and QQ Music offline data handling.

---

## 📖 Documentation

- [繁體中文版](docs/README.zh-TW.md)
- [English Version](docs/README.en.md)

---

## 🎯 Key Finding

- App 可以搬到 1TB SD（adopted storage）
- 但離線音樂資料不一定會跟著搬
- 最終解法：**系統層分離，使用層解決**

---

## ⚠️ Scope

- Sony Walkman
- Android 11
- Non-root
- ADB required
