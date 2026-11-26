# 🚀 Mirror Hub — 全量渐进式总控搬运系统

[简体中文](./README.md) | [繁體中文](./README.zh-TW.md) | [English](./README.en.md) | [日本語](./README.ja.md) | [한국어](./README.ko.md)

基于 GitHub Actions 的自动化开源资产镜像与容灾归档系统。支持 **Git 源码、Wiki、全量历史 Releases（突破 2GB 单文件限制）以及 Docker 多架构镜像** 的全自动同步与全网救援。

## ✨ 核心特性

- **双轨制追新与历史补齐**：首次建档锁定 `baseline_time`。基准时间后的新版本**实时追新**；基准时间前的海量历史版本按批次（默认 15 个/轮）**渐进式补齐**。
- **突破 GitHub 2GB 单文件限制**：自动检测超过 `1900MB` 的 Release 资产，使用 `split` 无损切割为 `.part_000` 分卷上传，并在发布说明中自动生成合并命令。
- **动态时间熔断与断点续传**：实时监控运行耗时，在触及 GitHub 6 小时硬限制前（5 小时处）优雅保存 `sync_state.json` 状态账本并安全退出，绝不丢进度。
- **404 删库全网救援**：兜底任务对失败项自动重试 3 次；若确认源仓库或版本已删库（HTTP 404），自动搜索全网公开镜像库抢救恢复，彻底绝版则归档至 `archived_dead`。
- **自动保活机制**：每次运行自动更新 `.github/.last_run`，防止仓库因 60 天无活动被官方停用定时任务。

## ⚙️ 快速配置

### 1. 配置密钥 (`MIRROR_TOKEN`)
在仓库 `Settings -> Secrets and variables -> Actions` 中新建 Secret，名称为 `MIRROR_TOKEN`，填入具备 `repo`、`workflow`、`write:packages`、`read:packages` 权限的 Personal Access Token (Classic)。

### 2. 编辑搬运清单 (`repos.txt`)
每行一个项目，使用 `|` 分隔四个参数（支持 `#` 注释）：
```text
# 格式：源GitHub仓库 | 源Docker镜像 | 目标仓库名 | 每次同步Docker标签数
# 无对应项请填 none
dify-ai/dify | langgenius/dify-api | dify | 5
n8n-io/n8n | n8nio/n8n | n8n | 5
torvalds/linux | none | linux-kernel | 0
```

## 🧩 大文件分卷合并指南
当搬运的单个文件超过 2GB 时，系统会自动将其拆分为 `文件名.part_000`、`文件名.part_001` 等分卷。下载所有分卷至同一目录后执行：
- **Linux / macOS**：`cat 文件名.part_* > 文件名`
- **Windows (CMD)**：`copy /b 文件名.part_* 文件名`
