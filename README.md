<div id="chinese"></div>
<p align="center">
  <a href="#user-content-english">English</a>
</p>

<div align="center">

# 🚀 OmniMirror Sync Engine
### 全自动 · 无上限 · 双轨制开源资产与容器镜像容灾中枢

<p align="center">
  <a href="../../actions"><code>⚡ Engine: GitHub Actions</code></a> &nbsp;•&nbsp;
  <a href="#核心架构与特性-core-architecture"><code>🔄 Mode: Dual-Track Incremental</code></a> &nbsp;•&nbsp;
  <a href="https://ghcr.io"><code>🐳 Registry: GHCR.io</code></a> &nbsp;•&nbsp;
  <a href="./sync_state.json"><code>📒 Ledger: sync_state.json</code></a>
</p>

**基于状态账本驱动（State-Driven）的分布式自动化镜像同步系统**  
*实现 Git 源码、Wiki 百科、全量 Releases 二进制资产与多架构 Docker 镜像的永久防丢失归档*

</div>

---

## 📖 项目概述 (Overview)

**OmniMirror Sync Engine** 是一套专为高可用私有化部署与开源资产永久归档设计的自动化调度中枢。针对传统同步脚本普遍存在的**运行超时、重复拉取、版本漏抓、源站删库（404）资产永久丢失**等痛点，本系统引入了结构化状态账本（`sync_state.json`）与双轨制调度算法，实现毫秒级差分过滤与全自动全网打捞救援。

---

## ✨ 核心架构与特性 (Core Architecture)

### 1. 🧭 基准时间双轨分流算法 (Dual-Track Scheduling)
* **实时追新通道（Zero-Quota Fast Track）**：以项目首次建档的 `baseline_time` 为界，凡在基准线之后发布的新版本，**无条件全量优先同步**，不占用历史补齐配额，确保上游最新动态秒级触达。
* **渐进历史清零（Progressive History Backfill）**：针对拥有数千个历史版本的庞大仓库，采用无上限底层分页接口（`--paginate`）全量测绘，按批次平稳回溯补齐；清零后自动标记 `history_completed: true`，切换至轻量巡检模式。

### 2. 🔐 指纹级差分去重 (Digest-Level Deduplication)
* **代码与文档按需克隆**：实时比对上游 `pushed_at` 时间戳，源码无变更时跳过 `git clone --mirror`，节省 90% 以上的网络与 I/O 开销。
* **容器镜像 SHA256 校验**：通过 `buildx imagetools` 提取远程 Manifest Digest 指纹，仅在镜像哈希发生实质变化或新增 Tag 时执行跨库推送至 `ghcr.io`。

### 3. 🚑 压轴兜底与全网打捞救援 (Fallback & 404 Rescue)
* **队列级重试机制**：所有正常任务执行完毕后，自动触发压轴兜底阶段，对本轮及历史遗留失败项执行多轮自动重试，直至清零。
* **删库检测与全网救援**：精确捕获上游 `HTTP 404` 状态；一旦确认原作者删库或撤回版本，引擎将自动检索全网高星公开备份库（Forks/Mirrors）并抢救打捞 Release 资产；确认全网绝版后自动归档至 `archived_dead`，保持账目绝对清晰。

### 4. 🛡️ 优雅熔断与实时事务存档 (Graceful Circuit Breaker)
* **单项目原子提交**：每完成一个项目的同步即刻通过 `rebase` 合并并推送更新 `sync_state.json`，彻底杜绝网络波动导致的进度丢失。
* **安全时间窗口控制**：内置运行计时器，单次执行满 5 小时 20 分钟自动触发优雅刹车，预留充足时间完成压轴兜底与状态落盘，确保每次 CI 任务全绿收官。

---

## 📂 仓库目录结构 (Directory Structure)

```text
.
├── .github/
│   ├── workflows/
│   │   └── batch-mirror.yml   # 核心调度引擎工作流（定时巡检 + 变更触发 + 手动调度）
│   └── .last_run              # 自动化保活心跳时间戳（防止 60 天无活动休眠）
├── repos.txt                  # 同步任务声明式配置清单
├── sync_state.json            # 全局状态账本（记录基准时间、已同步版本、指纹与救援档案）
└── README.md                  # 双语系统架构文档
```

---

## ⚙️ 配置清单规范 (`repos.txt`)

系统采用声明式管道符（`|`）配置格式，修改并推送 `repos.txt` 将自动触发增量同步：

```text
# 格式：GitHub源仓库 | Docker源镜像(无则填 none) | 目标仓库名 | 镜像动态追新数量
n8n-io/n8n               | docker.n8n.io/n8nio/n8n         | n8n          | 10
teableio/teable          | ghcr.io/teableio/teable         | teable       | 10
langgenius/dify          | langgenius/dify-api             | dify-api     | 10
none                     | postgres                        | postgres     | 10
Panniantong/Agent-Reach  | none                            | agent-reach  | 10
```

| 字段位置 | 参数名称 | 填写说明 |
| :--- | :--- | :--- |
| **第 1 列** | `GitHub Source` | 上游 `owner/repo` 路径；纯镜像项目填 `none` |
| **第 2 列** | `Docker Source` | 支持 Docker Hub、GHCR 及第三方私有域名；无镜像填 `none` |
| **第 3 列** | `Target Name` | 同步至当前账号下的目标仓库名及 `ghcr.io` 镜像名 |
| **第 4 列** | `Tag Limit` | 每次巡检动态追新的最新镜像 Tag 数量（默认 `5`） |

---

## 🐳 镜像拉取使用指南 (Usage)

所有同步的容器镜像均托管于 GitHub Container Registry（GHCR），支持多架构（`linux/amd64` / `linux/arm64`）原生拉取：

```bash
# 拉取最新版镜像
docker pull ghcr.io/liufengtao393/<目标仓库名>:latest

# 拉取指定历史版本镜像
docker pull ghcr.io/liufengtao393/<目标仓库名>:<版本号>
```

---

## 📊 状态账本结构说明 (`sync_state.json`)

系统全自动维护的 JSON 状态机，支持随时审计每个项目的同步健康度：

* `baseline_time`：项目首次纳入监管的 UTC 基准时间戳。
* `history_completed`：布尔值标记，`true` 代表基准线之前的所有历史资产已 100% 搬运清零。
* `synced_releases`：已完成镜像的全部 Release Tag 集合。
* `synced_images`：各镜像 Tag 对应的最新 `sha256` Digest 指纹及同步时间。
* `failed_releases`：待压轴兜底任务自动重试的异常队列。
* `archived_dead`：经全网检索确认上游已删库且无公开备份的绝版历史档案。
---

<div id="english"></div>
<p align="center">
  <a href="#user-content-chinese">简体中文</a>
</p>

<div align="center">

# 🚀 OmniMirror Sync Engine
### Autonomous · Unlimited · Dual-Track Open-Source Asset & Container Disaster Recovery Hub

<p align="center">
  <a href="../../actions"><code>⚡ Engine: GitHub Actions</code></a> &nbsp;•&nbsp;
  <a href="#-core-architecture"><code>🔄 Mode: Dual-Track Incremental</code></a> &nbsp;•&nbsp;
  <a href="https://ghcr.io"><code>🐳 Registry: GHCR.io</code></a> &nbsp;•&nbsp;
  <a href="./sync_state.json"><code>📒 Ledger: sync_state.json</code></a>
</p>

**State-Driven Distributed Mirroring & Disaster Recovery System**  
*Zero-loss archival for Git repositories, Wikis, full Release assets, and multi-arch Docker images*

</div>

---

## 📖 Overview

**OmniMirror Sync Engine** is an automated orchestration hub engineered for high-availability self-hosting and permanent open-source asset preservation. Designed to eliminate common mirroring bottlenecks—such as **CI timeouts, redundant pulls, missed tags, and permanent asset loss from upstream repository deletions (HTTP 404)**—the engine utilizes a structured state ledger (`sync_state.json`) and a dual-track scheduling algorithm to achieve millisecond-level diff filtering and automated network-wide asset recovery.

---

## ✨ Core Architecture

### 1. 🧭 Dual-Track Scheduling Algorithm
* **Zero-Quota Fast Track**: Anchored by each project's initial `baseline_time`, all newly published releases after the baseline are **prioritized and synced unconditionally** without consuming historical backfill quotas.
* **Progressive History Backfill**: Leverages unthrottled pagination (`--paginate`) to map repositories with thousands of historical releases and backfills them in safe batches. Once cleared, `history_completed: true` is flagged to switch into lightweight monitoring mode.

### 2. 🔐 Digest-Level Deduplication
* **On-Demand Git & Wiki Mirroring**: Compares upstream `pushed_at` timestamps in real time, skipping `git clone --mirror` when no source changes exist to save over 90% of network and I/O overhead.
* **SHA256 Image Manifest Verification**: Inspects remote manifest digests via `buildx imagetools`, pushing to `ghcr.io` only when image hashes change or new tags appear.

### 3. 🚑 Final Fallback & 404 Network Rescue
* **Multi-Attempt Retry Queue**: Executes a dedicated fallback phase after primary tasks complete, retrying transient failures across runs until resolved.
* **Upstream Deletion Detection & Rescue**: Accurately detects `HTTP 404` responses. If an upstream repository or release is deleted, the engine automatically searches high-star public forks/mirrors across GitHub to salvage release assets, or cleanly archives unrecoverable items into `archived_dead` to prevent infinite loops.

### 4. 🛡️ Graceful Circuit Breaker & Atomic Commits
* **Per-Project Transaction Commits**: Commits and pushes updates to `sync_state.json` via `rebase` immediately after each project finishes, preventing progress loss.
* **Safe Execution Window**: Built-in runtime timer gracefully exits the main loop at 5h 20m, reserving ample time for fallback retries and state persistence so every CI workflow finishes green.

---

## 📂 Directory Structure

```text
.
├── .github/
│   ├── workflows/
│   │   └── batch-mirror.yml   # Core orchestration workflow (Cron + Push + Manual Dispatch)
│   └── .last_run              # Keep-alive heartbeat timestamp (prevents 60-day inactivity disablement)
├── repos.txt                  # Declarative mirroring manifest
├── sync_state.json            # Global state ledger (baselines, synced tags, digests & rescue logs)
└── README.md                  # Bilingual system documentation
```

---

## ⚙️ Manifest Specification (`repos.txt`)

Configured via pipe-delimited (`|`) entries. Pushing updates to `repos.txt` automatically triggers an incremental sync:

```text
# Format: GitHub Source | Docker Source (or none) | Target Repo Name | Dynamic Tag Limit
n8n-io/n8n               | docker.n8n.io/n8nio/n8n         | n8n          | 10
teableio/teable          | ghcr.io/teableio/teable         | teable       | 10
langgenius/dify          | langgenius/dify-api             | dify-api     | 10
none                     | postgres                        | postgres     | 10
Panniantong/Agent-Reach  | none                            | agent-reach  | 10
```

| Column | Field Name | Description |
| :--- | :--- | :--- |
| **Col 1** | `GitHub Source` | Upstream `owner/repo` path; set to `none` for image-only targets |
| **Col 2** | `Docker Source` | Supports Docker Hub, GHCR, and custom registries; set to `none` if unavailable |
| **Col 3** | `Target Name` | Destination repository name under your account and `ghcr.io` namespace |
| **Col 4** | `Tag Limit` | Number of latest image tags to track dynamically per run (default: `5`) |

---

## 🐳 Container Usage

All mirrored container images are hosted on GitHub Container Registry (GHCR) with native multi-architecture (`linux/amd64` / `linux/arm64`) support:

```bash
# Pull the latest image
docker pull ghcr.io/liufengtao393/<target-repo>:latest

# Pull a specific version tag
docker pull ghcr.io/liufengtao393/<target-repo>:<version-tag>
```

---

## 📊 State Ledger Schema (`sync_state.json`)

The auto-maintained JSON state machine allows instant auditing of every mirrored asset:

* `baseline_time`: UTC timestamp when the repository was first registered.
* `history_completed`: Boolean flag indicating `100%` completion of pre-baseline releases.
* `synced_releases`: Array of all successfully mirrored release tags.
* `synced_images`: Map of image tags to their verified `sha256` digests and sync timestamps.
* `failed_releases`: Queue of transient failures scheduled for the final fallback phase.
* `archived_dead`: Permanent audit log for upstream `HTTP 404` releases with no available public mirrors.

---

<div align="center">

**Powered by GitHub Actions & Buildx Imagetools**  
*Automated · Resilient · Self-Healing*

</div>
