# 🚀 Mirror Hub — Progressive Full-Scale Asset Mirroring System

[简体中文](./README.md) | [繁體中文](./README.zh-TW.md) | [English](./README.en.md) | [日本語](./README.ja.md) | [한국어](./README.ko.md)

An automated open-source asset mirroring and disaster-recovery archiving system powered by GitHub Actions. It supports full-scale synchronization and cross-network rescue for **Git repositories, Wikis, historical Releases (bypassing the 2GB file limit), and multi-arch Docker images**.

## ✨ Key Features

- **Dual-Track Sync Engine**: Locks a `baseline_time` upon initial indexing. Releases published after the baseline are mirrored **immediately**, while older historical releases are backfilled **progressively** in configurable batches (default: 15 per run).
- **Bypass GitHub 2GB Single-File Limit**: Automatically detects release assets larger than `1900MB`, splits them losslessly into `.part_000` chunks, uploads all parts, and appends restoration commands to the release notes.
- **Dynamic Timeout & Checkpointing**: Continuously monitors execution time and gracefully saves `sync_state.json` at the 5-hour mark before hitting GitHub's 6-hour hard kill limit.
- **404 Deleted-Repo Rescue**: Retries failed releases 3 times. If an upstream repo or tag returns HTTP 404, it searches public GitHub mirrors to rescue the assets, or logs permanently lost versions under `archived_dead`.
- **Workflow Keep-Alive**: Automatically updates `.github/.last_run` on every execution to prevent GitHub from disabling scheduled workflows after 60 days of inactivity.

## ⚙️ Quick Setup

### 1. Configure Secret (`MIRROR_TOKEN`)
Navigate to `Settings -> Secrets and variables -> Actions`, create a new secret named `MIRROR_TOKEN`, and paste a Personal Access Token (Classic) with `repo`, `workflow`, `write:packages`, and `read:packages` scopes.

### 2. Configure `repos.txt`
Add one project per line separated by `|` (`#` comments supported):
```text
# Format: SOURCE_GH_REPO | SOURCE_DOCKER_IMAGE | TARGET_REPO_NAME | DOCKER_TAG_COUNT
# Use "none" if not applicable
dify-ai/dify | langgenius/dify-api | dify | 5
n8n-io/n8n | n8nio/n8n | n8n | 5
torvalds/linux | none | linux-kernel | 0
```

## 🧩 Merging Split Archives (>2GB)
Files exceeding 2GB are automatically split into `filename.part_000`, `filename.part_001`, etc. Download all parts into the same folder and run:
- **Linux / macOS**: `cat filename.part_* > filename`
- **Windows (CMD)**: `copy /b filename.part_* filename`
