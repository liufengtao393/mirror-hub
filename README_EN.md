# 🚀 Mirror Hub — 全量漸進式總控搬運系統

[简体中文](./README.md) | [繁體中文](./README.zh-TW.md) | [English](./README.en.md) | [日本語](./README.ja.md) | [한국어](./README.ko.md)

基於 GitHub Actions 的自動化開源資產鏡像與容災歸檔系統。支援 **Git 原始碼、Wiki、全量歷史 Releases（突破 2GB 單檔案限制）以及 Docker 多架構鏡像** 的全自動同步與全網救援。

## ✨ 核心特性

- **雙軌制追新與歷史補齊**：首次建檔鎖定 `baseline_time`。基準時間後的新版本**即時追新**；基準時間前的海量歷史版本按批次（預設 15 個/輪）**漸進式補齊**。
- **突破 GitHub 2GB 單檔案限制**：自動偵測超過 `1900MB` 的 Release 資產，使用 `split` 無損切割為 `.part_000` 分卷上傳，並在發佈說明中自動生成合併指令。
- **動態時間熔斷與斷點續傳**：即時監控運行耗時，在觸及 GitHub 6 小時硬限制前（5 小時處）優雅儲存 `sync_state.json` 狀態帳本並安全退出，絕不遺失進度。
- **404 刪庫全網救援**：兜底任務對失敗項自動重試 3 次；若確認來源倉庫或版本已刪庫（HTTP 404），自動搜尋全網公開鏡像庫搶救恢復，徹底絕版則歸檔至 `archived_dead`。
- **自動保活機制**：每次運行自動更新 `.github/.last_run`，防止倉庫因 60 天無活動被官方停用排程任務。

## ⚙️ 快速配置

### 1. 配置金鑰 (`MIRROR_TOKEN`)
在倉庫 `Settings -> Secrets and variables -> Actions` 中新增 Secret，名稱為 `MIRROR_TOKEN`，填入具備 `repo`、`workflow`、`write:packages`、`read:packages` 權限的 Personal Access Token (Classic)。

### 2. 編輯搬運清單 (`repos.txt`)
每行一個專案，使用 `|` 分隔四個參數（支援 `#` 註解）：
```text
# 格式：來源GitHub倉庫 | 來源Docker鏡像 | 目標倉庫名 | 每次同步Docker標籤數
# 無對應項請填 none
dify-ai/dify | langgenius/dify-api | dify | 5
n8n-io/n8n | n8nio/n8n | n8n | 5
torvalds/linux | none | linux-kernel | 0
```

## 🧩 大檔案分卷合併指南
當搬運的單一檔案超過 2GB 時，系統會自動將其拆分為 `檔名.part_000`、`檔名.part_001` 等分卷。下載所有分卷至同一目錄後執行：
- **Linux / macOS**：`cat 檔名.part_* > 檔名`
- **Windows (CMD)**：`copy /b 檔名.part_* 檔名`
