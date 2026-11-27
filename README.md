<div align="center">

<!-- 动态极简大厂风 Banner (Capsule Render 渐变波浪) -->
<img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&height=200&section=header&text=OmniMirror%20Hub&fontSize=60&fontAlignY=38&desc=Source%20%7C%20Wiki%20%7C%20Release%20%7C%20Container%20Mirror&descAlignY=62&descSize=16" width="100%" />

<p align="center">
  <strong>全自动 · 高容错 · 断点续传的开源数字资产容灾与多源镜像总控引擎</strong><br/>
  <sub>为关键开源基础设施提供源码、Wiki、全量 Release 二进制及多架构容器镜像的永久性容灾归档</sub>
</p>

<p align="center">
  <a href="../../actions"><img src="https://img.shields.io/badge/CI-Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white" alt="CI Engine"></a>
  <a href="../../packages"><img src="https://img.shields.io/badge/Registry-GHCR-2496ED?style=flat-square&logo=docker&logoColor=white" alt="Container Registry"></a>
  <a href="./sync_state.json"><img src="https://img.shields.io/badge/Ledger-Atomic-2ea44f?style=flat-square&logo=json&logoColor=white" alt="Atomic Ledger"></a>
  <a href="./LICENSE"><img src="https://img.shields.io/badge/License-MIT-yellow?style=flat-square" alt="License"></a>
</p>

<p align="center">
  <strong>简体中文</strong> &nbsp;·&nbsp;
  <a href="./README_TW.md">繁體中文</a> &nbsp;·&nbsp;
  <a href="./README_EN.md">English</a> &nbsp;·&nbsp;
  <a href="./README_JA.md">日本語</a> &nbsp;·&nbsp;
  <a href="./README_KO.md">한국어</a>
</p>

<p align="center">
  <a href="../../wiki"><strong>📖 探索完整 Wiki 文档</strong></a> &nbsp;&nbsp;|&nbsp;&nbsp;
  <a href="./repos.txt"><strong>⚙️ 镜像清单配置</strong></a> &nbsp;&nbsp;|&nbsp;&nbsp;
  <a href="./sync_state.json"><strong>📊 实时状态账本</strong></a> &nbsp;&nbsp;|&nbsp;&nbsp;
  <a href="../../issues"><strong>🐛 问题反馈</strong></a>
</p>

</div>

<br/>

## 概述

上游仓库突然删库、历史 Release 附件被清理、单文件突破 GitHub `2GB` 限制、或是多架构容器镜像拉取受限——这些都是自托管与企业内网基础设施面临的真实单点故障风险。

**OmniMirror Hub** 不是一个简单的定时同步脚本，而是一套经过生产级极限压测的**全量渐进式容灾总控架构**。它通过**双轨并发调度**、**1.9GB 智能二进制分卷**、**数学级资产完整性校验**以及**全网 404 自动抢救机制**，在完全零人工干预的前提下，将上游动态演进的软件资产安全沉淀为永久可控的私有/公有镜像库。

<br/>

## 核心架构优势

<table>
  <tr>
    <td width="50%" valign="top">
      <h3>⚡ 双轨制并发调度引擎</h3>
      <p>将同步任务解耦为<strong>「追新轨道」</strong>与<strong>「历史轨道」</strong>。以首次建档锁定时间（<code>baseline_time</code>）为分界线，上游新发布版本零延迟全量同步；海量历史版本按 <code>BATCH_SIZE</code> 渐进回溯，彻底规避 API 速率限制。</p>
    </td>
    <td width="50%" valign="top">
      <h3>✂️ 突破 2GB 限制与数学校验</h3>
      <p>单文件超过 <code>1.9GB</code> 自动执行无损二进制切割（<code>.part_000...</code>）。内置 <code>release_is_complete</code> 校验算法，通过源端字节数精确推算应有分卷数 <em>N</em>，自动清理网络中断留下的半成品并重建，零误伤。</p>
    </td>
  </tr>
  <tr>
    <td width="50%" valign="top">
      <h3>🚑 Anti-404 全网备份抢救</h3>
      <p>严格隔离 HTTP <code>403/429</code> 瞬时限流与真实 <code>404</code> 删库事件。已入库资产受绝对保护；确认上游 <code>404</code> 后，自动检索全网高星镜像与 Fork 仓库抢救历史版本，绝版资产自动归档至 <code>archived_dead</code>。</p>
    </td>
    <td width="50%" valign="top">
      <h3>🐳 跨源多架构容器原生透传</h3>
      <p>原生支持 Docker Hub、GHCR、LinuxServer 等主流 Registry。支持匿名 Bearer Token 自动换取，基于 Buildx Imagetools 实现 <code>amd64</code> / <code>arm64</code> / <code>armv7</code> 多架构 Manifest 零重新编译秒级透传。</p>
    </td>
  </tr>
</table>

<br/>

## 为什么选择 OmniMirror Hub？

| 评估维度 | 传统同步脚本 | **OmniMirror Hub** |
| :--- | :--- | :--- |
| **大仓库源码同步** | 超时中断即丢失进度，甚至误记为已同步 | **Clone + Push 双成功门控**，失败自动断点重试 |
| **超大 Release 附件** | 触发 GitHub 2GB 单文件限制直接报错 | **1.9GB 自动二进制分卷**，随附全平台一键还原指令 |
| **半成品残缺修复** | 检测到同名 Tag 即跳过，残缺 Release 永久留存 | **按资产名 + 分卷数学公式校验**，自动清除半成品重建 |
| **上游删库/删版本** | 遇到 404 报错中断，或误删本地已有历史存档 | **本地绝版资产绝对保护** + 全网同名高星库自动救援 |
| **并发与账本安全** | 多任务同时运行导致 JSON 冲突损坏 | **全局 Concurrency 队列锁** + `safe_jq` 原子写入校验 |

<br/>

## 快速部署

### 1. 配置访问凭证
前往仓库 **`Settings`** → **`Secrets and variables`** → **`Actions`**，新增以下 Repository Secret：

| Secret 名称 | 权限要求 | 用途说明 |
| :--- | :--- | :--- |
| `MIRROR_TOKEN` | `repo`, `workflow`, `write:packages` | 用于自动创建镜像仓库、推送源码/Wiki、发布 Release 及推送容器镜像 |

### 2. 定义同步清单（`repos.txt`）
在根目录 `repos.txt` 中按行配置目标项目（支持 `#` 注释与空行，使用 `|` 分隔四列）：

```text
# 格式：GitHub 源仓库      | 容器镜像源                 | 目标仓库名  | 镜像保留最新 Tag 数
n8n-io/n8n               | docker.n8n.io/n8nio/n8n   | n8n        | 10
langgenius/dify          | langgenius/dify-api       | dify-api   | 15
none                     | langgenius/dify-web       | dify-web   | 15
siyuan-note/siyuan       | b3log/siyuan              | siyuan     | 10
```

### 3. 启动总控工作流
* **自动化周期调度**：系统每日于 **`06:00 UTC`** 与 **`18:00 UTC`** 自动巡检并推进历史版本归档。
* **按需手动触发**：进入 **`Actions`** → **`全量渐进式总控搬运`** → **`Run workflow`**，可自定义单轮历史补齐上限（`batch_size`，默认 `15`）。

<br/>

## 进阶使用说明（点击展开）

<details>
<summary><strong>📦 如何还原超过 1.9GB 的自动分卷文件？</strong></summary>
<br/>

当 Release 中包含超过 `1.9GB` 的单文件时，系统会将其切割为 `filename.ext.part_000`、`filename.ext.part_001` 等分卷。将同一文件的所有分卷下载至同一目录后，执行对应平台的合并命令即可：

* **Linux / macOS**：
  ```bash
  cat filename.ext.part_* > filename.ext
  ```
* **Windows (Command Prompt)**：
  ```cmd
  copy /b filename.ext.part_* filename.ext
  ```
</details>

<details>
<summary><strong>📊 如何阅读实时状态账本（<code>sync_state.json</code>）？</strong></summary>
<br/>

系统通过 `safe_jq` 原子引擎维护每个项目的完整生命周期状态：
* `baseline_time`：首次建档时间，区分「追新轨道」与「历史轨道」的基准线。
* `synced_releases`：已通过完整性校验并归档的 Release Tag 集合。
* `failed_releases`：因瞬时网络波动或配额限制暂未完成的队列，将在压轴兜底任务中自动重试。
* `archived_dead`：经全网检索确认上游已彻底删除且无公共备份的绝版版本记录。

</details>

<details>
<summary><strong>🛡️ 极限容错与 API 配额保护机制</strong></summary>
<br/>

* **5 小时主流程熔断**：主循环运行满 `18000` 秒后自动优雅暂停并保存进度，预留充足时间进入压轴兜底阶段。
* **5 小时 40 分钟安全红线**：全局运行达 `20400` 秒时强制收尾提交保活记录，确保永不触发 GitHub Actions 6 小时强制终止（Kill）导致的进度丢失。
</details>

<br/>

## 文档体系与导航

* **[📖 Wiki 百科首页](../../wiki)** — 系统设计哲学与核心概念全景图
* **[⚙️ 架构原理与双轨机制](../../wiki/Architecture)** — 深入解析双轨调度、分卷数学校验与 404 救援算法
* **[🛠️ 配置详解与场景实战](../../wiki/Configuration-Guide)** — 纯源码、纯镜像、私有源及超大仓库的最佳实践
* **[🚑 运维监控与故障排查](../../wiki/Troubleshooting)** — 状态账本修复、定向体检与 API 限流应对指南

<br/>

<div align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&height=120&section=footer" width="100%" />
</div>
