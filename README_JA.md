# 🚀 Mirror Hub — 全量漸進型統合ミラーリングシステム

[简体中文](./README.md) | [繁體中文](./README.zh-TW.md) | [English](./README.en.md) | [日本語](./README.ja.md) | [한국어](./README.ko.md)

GitHub Actions を活用した自動オープンソース資産ミラーリング＆災害復旧アーカイブシステムです。**Git ソースコード、Wiki、全履歴 Releases（2GB単一ファイル制限を突破）、およびマルチアーキテクチャ Docker イメージ**の自動同期とネットワーク全体の救済をサポートします。

## ✨ 主な機能

- **デュアルトラック同期エンジン**：初回登録時に `baseline_time` をロックします。基準時刻以降の新規リリースは**即時追跡**し、過去の膨大な履歴リリースはバッチ単位（デフォルト15件/回）で**漸進的に補完**します。
- **GitHub 2GB ファイル制限の突破**：`1900MB` を超えるアセットを自動検出し、`split` で `.part_000` 形式に無劣化分割してアップロードします。リリースノートには結合コマンドが自動追記されます。
- **動的タイムアウト制御とチェックポイント**：実行時間を常時監視し、GitHub の6時間強制終了制限に達する前（5時間時点）に `sync_state.json` を安全に保存して終了します。
- **404 削除リポジトリの全網救済**：失敗した項目を最大3回再試行します。ソースが削除（HTTP 404）されている場合、公開ミラーリポジトリを自動検索して救出し、完全に失われた場合は `archived_dead` に記録します。
- **自動キープアライブ**：実行ごとに `.github/.last_run` を更新し、60日間の非アクティブによる定期実行の自動停止を防ぎます。

## ⚙️ クイックセットアップ

### 1. シークレットの設定 (`MIRROR_TOKEN`)
リポジトリの `Settings -> Secrets and variables -> Actions` で `MIRROR_TOKEN` を作成し、`repo`、`workflow`、`write:packages`、`read:packages` 権限を持つ Personal Access Token (Classic) を設定します。

### 2. 同期リストの編集 (`repos.txt`)
1行に1プロジェクトを `|` 区切りで記述します（`#` でコメント可能）：
```text
# 形式：ソースGitHubリポジトリ | ソースDockerイメージ | ターゲットリポジトリ名 | 同期Dockerタグ数
# 該当なしの場合は none を指定
dify-ai/dify | langgenius/dify-api | dify | 5
n8n-io/n8n | n8nio/n8n | n8n | 5
torvalds/linux | none | linux-kernel | 0
```

## 🧩 分割ファイル（2GB超）の結合方法
2GBを超えるファイルは `ファイル名.part_000`、`ファイル名.part_001` のように自動分割されます。すべてのパーツを同じフォルダにダウンロードし、以下のコマンドで復元してください：
- **Linux / macOS**：`cat ファイル名.part_* > ファイル名`
- **Windows (CMD)**：`copy /b ファイル名.part_* ファイル名`
