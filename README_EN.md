<p align="center">
  <a href="../../">简体中文</a> | <a href="./README_TW.md">繁體中文</a> | <b>English</b> | <a href="./README_JA.md">日本語</a> | <a href="./README_KO.md">한국어</a>
</p>

<div align="center">

# 🚀 OmniMirror Sync Engine
### Autonomous · Unlimited · Dual-Track Open-Source Asset & Container Disaster Recovery Hub

<p align="center">
  <a href="../../actions"><code>⚡ Engine: GitHub Actions</code></a> &nbsp;•&nbsp;
  <a href="./.github/workflows/batch-mirror.yml"><code>🔄 Mode: Dual-Track Incremental</code></a> &nbsp;•&nbsp;
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
├── README.md                  # Simplified Chinese documentation
├── README_TW.md               # Traditional Chinese documentation
├── README_EN.md               # English documentation
├── README_JA.md               # Japanese documentation
└── README_KO.md               # Korean documentation
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
