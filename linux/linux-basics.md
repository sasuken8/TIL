# Linux と AWS 環境の基礎知識

## Linux の基本概念

### Linux とは
Linux は 1991 年に Linus Torvalds によって開発が始まったオープンソースの OS で、厳密には「Linux カーネル」と呼ばれる中核部分を指す。このカーネルを基に様々な組織や企業が独自の「Linux ディストリビューション」を構築している。

### 主要な OS の比較

| OS | 開発元 | 特徴 | バージョン管理 |
|---|---|---|---|
| **Windows** | Microsoft 社 | 単一企業が開発・管理<br>一貫した体験 | Windows 10, 11 など |
| **macOS** | Apple 社 | 単一企業が開発・管理<br>主に Apple ハードウェア向け | macOS Ventura, Sonoma など |
| **Linux** | コミュニティ<br>(カーネルは Linus Torvalds 主導) | 多様な選択肢<br>用途に応じたカスタマイズ性 | カーネルバージョンと<br>ディストリビューション |

### Linux の主要コンポーネント

#### カーネル (Kernel)
OS の中心部分で、ハードウェアとソフトウェアの橋渡し役を担う。
- CPU の管理
- メモリの割り当てと管理
- ディスクへのアクセス制御
- ネットワーク通信の処理
- デバイスドライバの提供

簡単に言えば、アプリケーションがコンピュータのハードウェアと会話するための通訳のような役割。

#### ディストリビューション (Distribution)
Linux カーネルに様々なソフトウェアを組み合わせて作られた完全な OS パッケージ。

## Linux ディストリビューションと系統

### 主要な Linux ディストリビューションの系統

| 系統 | 代表的ディストリビューション | パッケージ管理 | 特徴 |
|---|---|---|---|
| **Red Hat 系** | Red Hat Enterprise Linux<br>Fedora<br>CentOS<br>**Amazon Linux** | RPM, yum/dnf | 企業環境での安定性重視<br>SELinux による強固なセキュリティ |
| **Debian 系** | Debian<br>Ubuntu<br>Linux Mint | DEB, apt/apt-get | 大規模なパッケージリポジトリ<br>幅広い用途に対応 |
| **SUSE 系** | SUSE Linux Enterprise<br>openSUSE | RPM, zypper | YaST という独自の管理ツール<br>ヨーロッパで特に人気 |
| **Arch 系** | Arch Linux<br>Manjaro | pacman | 最新のソフトウェア<br>ローリングリリースモデル |
| **Gentoo 系** | Gentoo<br>Calculate Linux | portage | ソースからのコンパイル<br>高度なカスタマイズ性 |

### Amazon Linux の特徴
Amazon Linux は AWS が提供する Linux ディストリビューションで、EC2 インスタンス向けに最適化されている。

- **系統**: Red Hat 系の特徴を持つ
- **バージョン**: 
  - Amazon Linux（初代、サポート終了）
  - Amazon Linux 2（2023 年 6 月 30 日までサポート）
  - Amazon Linux 2023（最新バージョン）
- **特徴**:
  - AWS サービスとの統合（CloudWatch, AWS CLI, Systems Manager）
  - セキュリティ重視（頻繁なセキュリティアップデート）
  - EC2 インスタンス向けパフォーマンス最適化
  - 開発者・運用者向けツールの充実

## パッケージ管理

### パッケージマネージャとは
OS にソフトウェアをインストール・更新・削除するためのツール。App Store や Microsoft Store と類似の役割を果たすが、主にコマンドライン操作が中心。

### 各 OS のパッケージ管理の比較

| OS | パッケージ管理方式 | GUI | CLI |
|---|---|---|---|
| **Windows** | .exe/.msi インストーラー | Microsoft Store | winget, chocolatey |
| **macOS** | .pkg/.dmg インストーラー | App Store | Homebrew |
| **Linux (Red Hat系)** | RPM パッケージ | GNOME Software | yum, dnf |
| **Linux (Debian系)** | DEB パッケージ | Ubuntu Software | apt, apt-get |

### パッケージリポジトリの構造

- **公式（コア）リポジトリ**: ディストリビューション開発者が厳選・管理
- **拡張リポジトリ**: コミュニティが維持する追加リポジトリ
- **サードパーティリポジトリ**: 外部組織や個人が管理
- **リポジトリミラー**: 世界中のサーバーがリポジトリのコピーを保持し、アクセス速度を向上

### yum パッケージマネージャーの基本コマンド (Amazon Linux)

| コマンド | 説明 | 例 |
|---|---|---|
| `yum search` | パッケージを検索 | `yum search httpd` |
| `yum info` | パッケージ情報の確認 | `yum info httpd` |
| `sudo yum install` | パッケージのインストール | `sudo yum install httpd` |
| `sudo yum update` | パッケージの更新 | `sudo yum update httpd` |
| `sudo yum remove` | パッケージの削除 | `sudo yum remove httpd` |
| `yum repolist` | 有効なリポジトリの一覧表示 | `yum repolist` |
| `sudo yum groupinstall` | パッケージグループのインストール | `sudo yum groupinstall "Development Tools"` |

### ソフトウェアのインストール方法の選択肢

1. **パッケージマネージャー経由**
   - メリット: 依存関係の自動解決、更新の一元管理
   - デメリット: バージョンが古い場合がある

2. **公式サイトからの直接インストール**
   - メリット: 最新バージョンを入手できる
   - デメリット: 更新が自動化されない、依存関係は手動解決

3. **ソースからのビルド**
   - メリット: 最大のカスタマイズ性
   - デメリット: 複雑で時間がかかる

## Linux のファイルシステム構造

Linux は「すべてはファイル」という哲学に基づいており、階層的なディレクトリ構造を持つ。

| ディレクトリ | 内容 |
|---|---|
| `/` | ルートディレクトリ（全ての始まり） |
| `/bin` | 基本的なコマンド |
| `/boot` | 起動に必要なファイル |
| `/dev` | デバイスファイル |
| `/etc` | システム設定ファイル |
| `/home` | ユーザーのホームディレクトリ |
| `/lib` | 共有ライブラリ |
| `/mnt`, `/media` | マウントポイント |
| `/opt` | オプションのアプリケーション |
| `/proc` | プロセス情報 |
| `/root` | root ユーザーのホームディレクトリ |
| `/sbin` | システム管理コマンド |
| `/tmp` | 一時ファイル |
| `/usr` | ユーザープログラム |
| `/var` | 可変データ（ログなど） |

## よく使う Linux コマンド

### 基本操作

| コマンド | 説明 | 例 |
|---|---|---|
| `ls` | ファイル一覧の表示 | `ls -la` |
| `cd` | ディレクトリの移動 | `cd /var/log` |
| `pwd` | 現在のディレクトリを表示 | `pwd` |
| `cp` | ファイルのコピー | `cp file1 file2` |
| `mv` | ファイルの移動・名前変更 | `mv old new` |
| `rm` | ファイルの削除 | `rm file` |
| `mkdir` | ディレクトリの作成 | `mkdir newdir` |

### システム情報

| コマンド | 説明 |
|---|---|
| `top`, `htop` | プロセス監視 |
| `df` | ディスク容量確認 |
| `free` | メモリ使用状況 |
| `uname` | システム情報表示 |

### ファイル操作

| コマンド | 説明 |
|---|---|
| `cat` | ファイル内容の表示 |
| `less`, `more` | ファイル内容のページング表示 |
| `grep` | パターンマッチング検索 |
| `find` | ファイルの検索 |

## AWS 環境での Linux 活用例

### EC2 インスタンスの初期設定
```bash
# システム更新
sudo yum update -y

# Web サーバーインストール
sudo yum install -y httpd

# サービス開始
sudo systemctl start httpd

# 自動起動設定
sudo systemctl enable httpd
```

### ログ解析
```bash
# 起動スクリプトのログ確認
sudo less /var/log/cloud-init-output.log

# エラーログの確認
sudo grep ERROR /var/log/httpd/error_log
```

### ディスク管理
```bash
# 新しい EBS ボリュームの確認
sudo file -s /dev/xvdf

# フォーマット
sudo mkfs -t xfs /dev/xvdf

# マウントポイント作成
sudo mkdir /data

# マウント
sudo mount /dev/xvdf /data
```