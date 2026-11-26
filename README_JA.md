<p align="center">
  <a href="../../">简体中文</a> | <a href="./README_TW.md">繁體中文</a> | <a href="./README_EN.md">English</a> | <b>日本語</b> | <a href="./README_KO.md">한국어</a>
</p>

<div align="center">

# 🚀 OmniMirror Sync Engine
### 全自動・無制限・デュアルトラック方式のオープンソース資産＆コンテナ災害復旧ハブ

<p align="center">
  <a href="../../actions"><code>⚡ Engine: GitHub Actions</code></a> &nbsp;•&nbsp;
  <a href="./.github/workflows/batch-mirror.yml"><code>🔄 Mode: Dual-Track Incremental</code></a> &nbsp;•&nbsp;
  <a href="https://ghcr.io"><code>🐳 Registry: GHCR.io</code></a> &nbsp;•&nbsp;
  <a href="./sync_state.json"><code>📒 Ledger: sync_state.json</code></a>
</p>

**ステート台帳駆動（State-Driven）の分散型自動ミラーリングシステム**  
*Gitソースコード、Wiki、全Releaseバイナリ資産、マルチアーキテクチャDockerイメージの永続的アーカイブを実現*

</div>

---

## 📖 プロジェクト概要 (Overview)

**OmniMirror Sync Engine** は、高可用性セルフホスティングおよびオープンソース資産の永続的な保全のために設計された自動オーケストレーションハブです。従来の同期スクリプトにありがちな**タイムアウト、重複取得、タグの取りこぼし、公開元のリポジトリ削除（HTTP 404）による資産の永久喪失**といった課題を解決するため、構造化された状態台帳（`sync_state.json`）とデュアルトラック・スケジューリングアルゴリズムを導入し、ミリ秒単位の差分フィルタリングと全自動のネットワーク全体サルベージ（救出）を実現します。

---

## ✨ コアアーキテクチャと機能 (Core Architecture)

### 1. 🧭 基準時刻デュアルトラック振り分けアルゴリズム (Dual-Track Scheduling)
* **リアルタイム最新追跡（Zero-Quota Fast Track）**：初回登録時の `baseline_time`（基準時刻）以降に公開された新バージョンは、履歴補完の割り当て枠を消費せず**無条件で最優先同期**されます。
* **段階的履歴バックフィル（Progressive History Backfill）**：数千の履歴バージョンを持つ大規模リポジトリに対し、無制限ページネーション（`--paginate`）で全量をマッピングし、安全なバッチ単位で順次補完します。完了後は自動的に `history_completed: true` が記録され、軽量な監視モードへ移行します。

### 2. 🔐 ダイジェストレベルの差分重複排除 (Digest-Level Deduplication)
* **コードとWikiのオンデマンド複製**：公開元の `pushed_at` タイムスタンプをリアルタイム比較し、ソースに変更がない場合は `git clone --mirror` をスキップすることで、ネットワークとI/Oの負荷を90%以上削減します。
* **コンテナイメージ SHA256 検証**：`buildx imagetools` でリモートの Manifest Digest を抽出し、イメージのハッシュ値に実質的な変更がある場合や新規タグ追加時のみ `ghcr.io` へプッシュします。

### 3. 🚑 最終フォールバック＆404ネットワーク救出 (Fallback & 404 Rescue)
* **キューレベル自動リトライ**：通常タスク完了後に最終フォールバックフェーズを実行し、一時的な失敗項目を複数回自動リトライしてゼロになるまで追跡します。
* **削除検知と全ネットサルベージ**：公開元の `HTTP 404` ステータスを正確に検知。公開元が削除された場合、GitHub全体の高評価フォーク・ミラーリポジトリを自動検索してRelease資産を救出します。完全に絶版となった項目は `archived_dead` へアーカイブし、無限ループを防止します。

### 4. 🛡️ グレースフル・サーキットブレーカーと即時保存 (Graceful Circuit Breaker)
* **プロジェクト単位のアトミックコミット**：1つのプロジェクトの同期完了ごとに `rebase` 経由で `sync_state.json` を即座にプッシュ保存し、進捗の喪失を防ぎます。
* **安全実行時間ウィンドウ**：内蔵タイマーにより実行時間が5時間20分に達するとメインループを安全に停止し、フォールバック処理と状態保存の時間を確保してCIタスクを常に正常終了（グリーン）させます。

---

## 📂 ディレクトリ構成 (Directory Structure)

```text
.
├── .github/
│   ├── workflows/
│   │   └── batch-mirror.yml   # コアスケジューリングワークフロー（定期実行 + 変更トリガー + 手動実行）
│   └── .last_run              # 自動キープアライブ用タイムスタンプ（60日間無操作による停止を防止）
├── repos.txt                  # 同期タスク宣言型マニフェスト
├── sync_state.json            # グローバル状態台帳（基準時刻、同期済みタグ、ダイジェスト、救出記録）
├── README.md                  # 简体中文ドキュメント
├── README_TW.md               # 繁體中文ドキュメント
├── README_EN.md               # English documentation
├── README_JA.md               # 日本語ドキュメント
└── README_KO.md               # 한국어 문서
```

---

## ⚙️ マニフェスト仕様 (`repos.txt`)

パイプ区切り（`|`）形式で記述します。`repos.txt` を更新してプッシュすると自動的に差分同期がトリガーされます：

```text
# 形式：GitHubソース | Dockerソース(なしの場合は none) | ターゲットリポジトリ名 | 動的追跡タグ数
n8n-io/n8n               | docker.n8n.io/n8nio/n8n         | n8n          | 10
teableio/teable          | ghcr.io/teableio/teable         | teable       | 10
langgenius/dify          | langgenius/dify-api             | dify-api     | 10
none                     | postgres                        | postgres     | 10
Panniantong/Agent-Reach  | none                            | agent-reach  | 10
```

| 列 | パラメータ名 | 説明 |
| :--- | :--- | :--- |
| **第 1 列** | `GitHub Source` | 公開元の `owner/repo` パス（イメージのみの場合は `none`） |
| **第 2 列** | `Docker Source` | Docker Hub、GHCR、独自レジストリに対応（なしの場合は `none`） |
| **第 3 列** | `Target Name` | 現在のアカウント配下の同期先リポジトリ名および `ghcr.io` イメージ名 |
| **第 4 列** | `Tag Limit` | 巡回ごとに動的に追跡する最新イメージタグ数（デフォルト：`5`） |

---

## 🐳 イメージのプル方法 (Usage)

同期されたすべてのコンテナイメージは GitHub Container Registry（GHCR）にホストされており、マルチアーキテクチャ（`linux/amd64` / `linux/arm64`）に対応しています：

```bash
# 最新版イメージの取得
docker pull ghcr.io/liufengtao393/<ターゲットリポジトリ名>:latest

# 特定バージョンタグの取得
docker pull ghcr.io/liufengtao393/<ターゲットリポジトリ名>:<バージョンタグ>
```

---

## 📊 状態台帳スキーマ (`sync_state.json`)

自動メンテナンスされるJSONステートマシンにより、各プロジェクトの同期状況をいつでも監査できます：

* `baseline_time`：プロジェクトが初めて登録されたUTC基準タイムスタンプ。
* `history_completed`：基準時刻以前の全履歴資産が100%同期完了したことを示すブール値フラグ。
* `synced_releases`：同期が完了した全Releaseタグの配列。
* `synced_images`：各イメージタグに対応する最新 `sha256` ダイジェストと同期日時。
* `failed_releases`：最終フォールバックフェーズで自動リトライされる異常キュー。
* `archived_dead`：公開元が `HTTP 404` かつ公開ミラーも存在しない絶版資産の監査アーカイブ。

---

<div align="center">

**Powered by GitHub Actions & Buildx Imagetools**  
*Automated · Resilient · Self-Healing*

</div>
