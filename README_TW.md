<p align="center">
  <a href="../../">简体中文</a> | <b>繁體中文</b> | <a href="./README_EN.md">English</a> | <a href="./README_JA.md">日本語</a> | <a href="./README_KO.md">한국어</a>
</p>

<div align="center">

# 🚀 OmniMirror Sync Engine
### 全自動 · 無上限 · 雙軌制開源資產與容器映像檔容災中樞

<p align="center">
  <a href="../../actions"><code>⚡ Engine: GitHub Actions</code></a> &nbsp;•&nbsp;
  <a href="./.github/workflows/batch-mirror.yml"><code>🔄 Mode: Dual-Track Incremental</code></a> &nbsp;•&nbsp;
  <a href="https://ghcr.io"><code>🐳 Registry: GHCR.io</code></a> &nbsp;•&nbsp;
  <a href="./sync_state.json"><code>📒 Ledger: sync_state.json</code></a>
</p>

**基於狀態帳本驅動（State-Driven）的分散式自動化映像同步系統**  
*實現 Git 原始碼、Wiki 百科、全量 Releases 二進位資產與多架構 Docker 映像檔的永久防遺失歸檔*

</div>

---

## 📖 專案概述 (Overview)

**OmniMirror Sync Engine** 是一套專為高可用私有化部署與開源資產永久歸檔設計的自動化排程中樞。針對傳統同步腳本普遍存在的**執行逾時、重複拉取、版本漏抓、源站刪庫（404）資產永久遺失**等痛點，本系統引入了結構化狀態帳本（`sync_state.json`）與雙軌制排程演算法，實現毫秒級差分過濾與全自動全網打撈救援。

---

## ✨ 核心架構與特性 (Core Architecture)

### 1. 🧭 基準時間雙軌分流演算法 (Dual-Track Scheduling)
* **即時追新通道（Zero-Quota Fast Track）**：以專案首次建檔的 `baseline_time` 為界，凡在基準線之後發布的新版本，**無條件全量優先同步**，不佔用歷史補齊配額，確保上游最新動態秒級觸達。
* **漸進歷史清零（Progressive History Backfill）**：針對擁有數千個歷史版本的龐大儲存庫，採用無上限底層分頁介面（`--paginate`）全量測繪，按批次平穩回溯補齊；清零後自動標記 `history_completed: true`，切換至輕量巡檢模式。

### 2. 🔐 指紋級差分去重 (Digest-Level Deduplication)
* **程式碼與文件按需複製**：即時比對上游 `pushed_at` 時間戳，原始碼無變更時跳過 `git clone --mirror`，節省 90% 以上的網路與 I/O 開銷。
* **容器映像檔 SHA256 校驗**：透過 `buildx imagetools` 提取遠端 Manifest Digest 指紋，僅在映像檔雜湊發生實質變化或新增 Tag 時執行跨庫推送至 `ghcr.io`。

### 3. 🚑 壓軸兜底與全網打撈救援 (Fallback & 404 Rescue)
* **佇列級重試機制**：所有正常任務執行完畢後，自動觸發壓軸兜底階段，對本輪及歷史遺留失敗項執行多輪自動重試，直至清零。
* **刪庫偵測與全網救援**：精確捕獲上游 `HTTP 404` 狀態；一旦確認原作者刪庫或撤回版本，引擎將自動檢索全網高星公開備份庫（Forks/Mirrors）並搶救打撈 Release 資產；確認全網絕版後自動歸檔至 `archived_dead`，保持帳目絕對清晰。

### 4. 🛡️ 優雅熔斷與即時事務存檔 (Graceful Circuit Breaker)
* **單專案原子提交**：每完成一個專案的同步即刻透過 `rebase` 合併並推送更新 `sync_state.json`，徹底杜絕網路波動導致的進度遺失。
* **安全時間視窗控制**：內建執行計時器，單次執行滿 5 小時 20 分鐘自動觸發優雅煞車，預留充足時間完成壓軸兜底與狀態落盤，確保每次 CI 任務全綠收官。

---

## 📂 儲存庫目錄結構 (Directory Structure)

```text
.
├── .github/
│   ├── workflows/
│   │   └── batch-mirror.yml   # 核心排程引擎工作流（定時巡檢 + 變更觸發 + 手動排程）
│   └── .last_run              # 自動化保活心跳時間戳（防止 60 天無活動休眠）
├── repos.txt                  # 同步任務宣告式設定清單
├── sync_state.json            # 全域狀態帳本（記錄基準時間、已同步版本、指紋與救援檔案）
├── README.md                  # 简体中文说明文档
├── README_TW.md               # 繁體中文說明文檔
├── README_EN.md               # English documentation
├── README_JA.md               # 日本語ドキュメント
└── README_KO.md               # 한국어 문서
```

---

## ⚙️ 設定清單規範 (`repos.txt`)

系統採用宣告式管道符（`|`）設定格式，修改並推送 `repos.txt` 將自動觸發增量同步：

```text
# 格式：GitHub源儲存庫 | Docker源映像檔(無則填 none) | 目標儲存庫名 | 映像檔動態追新數量
n8n-io/n8n               | docker.n8n.io/n8nio/n8n         | n8n          | 10
teableio/teable          | ghcr.io/teableio/teable         | teable       | 10
langgenius/dify          | langgenius/dify-api             | dify-api     | 10
none                     | postgres                        | postgres     | 10
Panniantong/Agent-Reach  | none                            | agent-reach  | 10
```

| 欄位位置 | 參數名稱 | 填寫說明 |
| :--- | :--- | :--- |
| **第 1 欄** | `GitHub Source` | 上游 `owner/repo` 路徑；純映像檔專案填 `none` |
| **第 2 欄** | `Docker Source` | 支援 Docker Hub、GHCR 及第三方私有網域；無映像檔填 `none` |
| **第 3 欄** | `Target Name` | 同步至目前帳號下的目標儲存庫名及 `ghcr.io` 映像檔名 |
| **第 4 欄** | `Tag Limit` | 每次巡檢動態追新的最新映像檔 Tag 數量（預設 `5`） |

---

## 🐳 映像檔拉取使用指南 (Usage)

所有同步的容器映像檔均託管於 GitHub Container Registry（GHCR），支援多架構（`linux/amd64` / `linux/arm64`）原生拉取：

```bash
# 拉取最新版映像檔
docker pull ghcr.io/liufengtao393/<目標儲存庫名>:latest

# 拉取指定歷史版本映像檔
docker pull ghcr.io/liufengtao393/<目標儲存庫名>:<版本號>
```

---

## 📊 狀態帳本結構說明 (`sync_state.json`)

系統全自動維護的 JSON 狀態機，支援隨時審計每個專案的同步健康度：

* `baseline_time`：專案首次納入監管的 UTC 基準時間戳。
* `history_completed`：布林值標記，`true` 代表基準線之前的所有歷史資產已 100% 搬運清零。
* `synced_releases`：已完成映像的全部 Release Tag 集合。
* `synced_images`：各映像檔 Tag 對應的最新 `sha256` Digest 指紋及同步時間。
* `failed_releases`：待壓軸兜底任務自動重試的異常佇列。
* `archived_dead`：經全網檢索確認上游已刪庫且無公開備份的絕版歷史檔案。

---

<div align="center">

**Powered by GitHub Actions & Buildx Imagetools**  
*Automated · Resilient · Self-Healing*

</div>
