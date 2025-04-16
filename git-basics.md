# Git の基本知識

## 1. Git の概要

Gitは分散型バージョン管理システムであり、以下の特徴を持つ：

- オープンソースソフトウェア（Linus Torvalds が開発）
- コードの変更履歴を追跡・管理
- ローカルで完全に動作可能
- 複数の開発者間での協業をサポート

## 2. Git の基本構造

### リポジトリの種類

| 種類 | 説明 | 場所 |
|------|------|------|
| ローカルリポジトリ | 開発者のPC上にある完全なリポジトリ | `.git`ディレクトリ内 |
| リモートリポジトリ | サーバー上にあるリポジトリ | GitHub, GitLab, Bitbucket など |

### Git データの 3 つの状態

| 状態 | 説明 |
|------|------|
| ワーキングディレクトリ | 実際に作業しているファイル |
| ステージングエリア | 次のコミットに含める変更（インデックス） |
| コミット履歴 | 永続的に保存された変更の記録 |

## 3. 基本的な Git コマンド

| コマンド | 説明 | 使用例 |
|---------|------|--------|
| `git init` | 新しいリポジトリを初期化 | `git init` |
| `git clone` | リモートリポジトリをコピー | `git clone https://github.com/user/repo.git` |
| `git add` | 変更をステージングエリアに追加 | `git add file.txt` または `git add .` |
| `git commit` | 変更をリポジトリに記録 | `git commit -m "メッセージ"` |
| `git status` | 現在の状態を確認 | `git status` |
| `git log` | コミット履歴を表示 | `git log` |
| `git pull` | リモートの変更を取得してマージ | `git pull origin main` |
| `git push` | ローカルの変更をリモートに送信 | `git push origin main` |
| `git branch` | ブランチの一覧表示・作成 | `git branch` または `git branch new-branch` |
| `git checkout` | ブランチの切り替え | `git checkout branch-name` |
| `git merge` | ブランチを統合 | `git merge branch-name` |

## 4. Git 設定

### config レベル

| レベル | 適用範囲 | ファイル場所 | コマンド例 |
|-------|----------|------------|----------|
| system | 全ユーザー＆全リポジトリ | `/etc/gitconfig` | `git config --system user.name "name"` |
| global | 現ユーザーの全リポジトリ | `~/.gitconfig` | `git config --global user.name "name"` |
| local | 単一リポジトリのみ | `.git/config` | `git config --local user.name "name"` |

### 主要な設定項目

```bash
# ユーザー情報
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# デフォルトブランチ名
git config --global init.defaultBranch main

# エディタ設定
git config --global core.editor "vim"
```

## 5. リモートリポジトリ連携

### リモート設定

```bash
# リモートリポジトリの追加
git remote add origin git@github.com:username/repo.git

# リモートの確認
git remote -v

# アップストリーム設定でプッシュ
git push -u origin main
```

### リモート用語

| 用語 | 説明 |
|------|------|
| origin | リモートリポジトリの標準的な名前（任意の名前可） |
| upstream | ローカルブランチとリモートブランチの追跡関係 |
| fork | リモートリポジトリのコピー（別アカウントへ） |

## 6. SSH 認証

### SSH 鍵の設定手順

1. SSH 鍵ペアの生成
   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```

2. 公開鍵を GitHub に登録
   - GitHub の Settings > SSH and GPG keys > New SSH key

3. 接続テスト
   ```bash
   ssh -T git@github.com
   ```

### SSH 設定ファイル（~/.ssh/config）

```
Host github.com
  IdentityFile ~/.ssh/id_ed25519
  User git
  AddKeysToAgent yes
  UseKeychain yes  # Mac のみ
```

### SSH エージェント設定

```bash
# エージェント起動
eval "$(ssh-agent -s)"

# 鍵の追加（Mac の場合）
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

## 7. ブランチ戦略

### 主なブランチ種類

| ブランチタイプ | 目的 | 命名例 |
|--------------|------|--------|
| main/master | 安定した公開コード | `main`, `master` |
| feature | 新機能開発 | `feature/login`, `feature-user-profile` |
| hotfix | 緊急バグ修正 | `hotfix/critical-bug` |
| release | リリース準備 | `release/v1.0.0` |

### ブランチ操作

```bash
# ブランチ作成と切り替え
git checkout -b feature/new-feature

# 変更をメインブランチにマージ
git checkout main
git merge feature/new-feature

# 作業完了後のブランチ削除
git branch -d feature/new-feature
```

## 8. セルフホスティング vs サービス

### Git ホスティングの選択肢

| タイプ | 例 | 特徴 |
|-------|-----|------|
| パブリックサービス | GitHub.com, GitLab.com | 無料プラン有り、簡単に開始可能 |
| セルフホスティング | GitLab Self-managed, GitHub Enterprise | 完全な管理権限、社内データ保持 |
| プライベートインスタンス | 社内 GitLab (`gitlab.company.com`) | セキュリティ、カスタマイズ、社内システム連携 |

