# Gitの一般的な使い方まとめ

## 目次
- [基本概念](#基本概念)
- [初期設定](#初期設定)
- [基本的なGitコマンド](#基本的なgitコマンド)
- [ブランチ操作](#ブランチ操作)
- [リモートリポジトリ操作](#リモートリポジトリ操作)
- [プルとクローンの詳細](#プルとクローンの詳細)
- [マージとリベース](#マージとリベース)
- [履歴の確認](#履歴の確認)
- [変更の取り消し](#変更の取り消し)
- [よく使うワークフロー](#よく使うワークフロー)
- [トラブルシューティング](#トラブルシューティング)
- [便利なTips](#便利なtips)

## 基本概念

### Gitとは
Gitは分散型バージョン管理システムで、ファイルの変更履歴を効率的に管理できます。

### 重要な用語
- **リポジトリ（Repository）**: プロジェクトのファイルと履歴を保存する場所
- **コミット（Commit）**: ファイルの変更を記録すること
- **ブランチ（Branch）**: 開発の分岐点
- **マージ（Merge）**: ブランチを統合すること
- **プル（Pull）**: リモートリポジトリから変更を取得すること
- **プッシュ（Push）**: ローカルの変更をリモートリポジトリに送信すること

## 初期設定

### Gitの初期設定
```bash
# ユーザー名とメールアドレスの設定
git config --global user.name "あなたの名前"
git config --global user.email "your-email@example.com"

# デフォルトブランチ名の設定
git config --global init.defaultBranch main

# エディタの設定（例：VS Code）
git config --global core.editor "code --wait"

# 設定の確認
git config --list
```

### リポジトリの初期化
```bash
# 新しいリポジトリを作成
git init

# 既存のリポジトリをクローン
git clone https://gitlab.com/username/repository.git
```

## 基本的なGitコマンド

### ファイルの追加とコミット
```bash
# ファイルの状態を確認
git status

# ファイルをステージングエリアに追加
git add filename.txt
git add .  # すべてのファイルを追加

# コミットを作成
git commit -m "コミットメッセージ"

# 追加とコミットを同時に実行（追跡済みファイルのみ）
git commit -am "コミットメッセージ"
```

### ファイルの変更確認
```bash
# 変更内容を確認
git diff

# ステージングエリアの変更を確認
git diff --staged

# 特定のファイルの変更を確認
git diff filename.txt
```

## ブランチ操作

### ブランチの基本操作
```bash
# 現在のブランチを確認
git branch

# すべてのブランチを確認（リモートブランチも含む）
git branch -a

# 新しいブランチを作成
git branch feature-branch

# ブランチを作成して切り替え
git checkout -b feature-branch
# または（Git 2.23以降）
git switch -c feature-branch

# ブランチを切り替え
git checkout main
# または
git switch main

# ブランチを削除
git branch -d feature-branch

# 強制削除
git branch -D feature-branch
```

### ブランチの名前変更
```bash
# 現在のブランチ名を変更
git branch -m new-branch-name

# 他のブランチ名を変更
git branch -m old-name new-name
```

## リモートリポジトリ操作

### リモートリポジトリの管理
```bash
# リモートリポジトリを追加
git remote add origin https://gitlab.com/username/repository.git

# リモートリポジトリの確認
git remote -v

# リモートリポジトリの情報を更新
git fetch

# リモートリポジトリから変更を取得してマージ
git pull origin main

# ローカルの変更をリモートにプッシュ
git push origin main

# 新しいブランチを初回プッシュ
git push -u origin feature-branch
```

### リモートブランチの操作
```bash
# リモートブランチをローカルにチェックアウト
git checkout -b local-branch origin/remote-branch

# リモートブランチを削除
git push origin --delete branch-name
```

## プルとクローンの詳細

### プル（Pull）のやり方と仕組み

#### 基本的なプルの使い方
```bash
# 現在のブランチにリモートの変更を取得
git pull

# 特定のリモートとブランチを指定してプル
git pull origin main

# プルの前にリモートの状態を確認
git fetch origin
git log HEAD..origin/main --oneline  # リモートの新しいコミットを確認
git pull origin main

# プルでリベースを使用（マージコミットを作らない）
git pull --rebase origin main
```

#### プルの内部動作
`git pull` は実際には以下の2つのコマンドの組み合わせです：
```bash
# git pull は以下と同等
git fetch origin    # リモートの変更を取得
git merge origin/main  # 現在のブランチにマージ

# リベースを使う場合
git fetch origin
git rebase origin/main
```

### クローンとプルの違い

| 項目 | git clone | git pull |
|------|-----------|----------|
| **目的** | リポジトリ全体を初回取得 | 既存リポジトリの更新 |
| **使用タイミング** | 初めてリポジトリを取得する時 | 定期的な更新作業 |
| **作成されるもの** | 新しいディレクトリとリポジトリ | 既存リポジトリに変更を適用 |
| **取得する内容** | 全履歴、全ブランチ情報 | 指定ブランチの最新変更のみ |
| **前提条件** | ローカルにリポジトリが存在しない | 既にローカルリポジトリが存在 |

#### 使い分けの例
```bash
# 【初回】プロジェクトに参加する時
git clone https://gitlab.com/username/project.git
cd project

# 【日常】作業開始時の更新
git pull origin main

# 【誤解例】既にクローン済みなのに再度クローンは不要
# ❌ git clone https://gitlab.com/username/project.git  # 不要
# ✅ git pull origin main  # これで十分
```

### 特定のブランチをリモートから取得してcheckoutする方法

#### リモートブランチの確認と取得
```bash
# リモートの全ブランチを確認
git branch -r

# リモートの情報を更新
git fetch origin

# 特定のリモートブランチをローカルにチェックアウト
git checkout -b local-branch-name origin/remote-branch-name

# より簡潔な方法（Git 2.23以降）
git switch remote-branch-name  # 同名でローカルブランチを作成

# リモートブランチを追跡するローカルブランチを作成
git checkout --track origin/remote-branch-name
```

#### 実践的な例
```bash
# 1. 開発チームが作成した新機能ブランチを取得
git fetch origin
git checkout -b feature-login origin/feature-login

# 2. 他の開発者のブランチで作業を継続
git fetch origin
git switch colleague-feature-branch

# 3. 特定のリモートブランチの内容だけを確認（チェックアウトしない）
git fetch origin
git log origin/feature-branch --oneline
git show origin/feature-branch:filename.txt
```

#### 複数のリモートブランチを一度に取得
```bash
# すべてのリモートブランチ情報を取得
git fetch --all

# 特定のリモートのすべてのブランチを取得
git fetch origin

# プルーニング（削除されたリモートブランチの情報をローカルからも削除）
git fetch --prune
```

### クローンし直す際の注意点

#### クローン前の確認事項
```bash
# 1. 現在の作業状態を確認
git status
git stash list  # 退避した作業があるかチェック

# 2. 未プッシュのコミットがないか確認
git log origin/main..HEAD --oneline

# 3. ローカル設定の確認とバックアップ
git config --local --list > git-config-backup.txt
```

#### 安全なクローンし直し手順
```bash
# 1. 重要な設定やファイルをバックアップ
cp -r .git/hooks/ ~/git-hooks-backup/  # Git hooksのバックアップ
git config --local --list > ~/git-config-backup.txt

# 2. 未コミットの変更を退避
git stash push -m "クローン前の一時退避"

# 3. 未プッシュのコミットをリモートにプッシュ
git push origin current-branch

# 4. 親ディレクトリに移動してクローン
cd ..
rm -rf old-project-directory
git clone https://gitlab.com/username/project.git new-project-directory

# 5. 設定を復元
cd new-project-directory
# 必要に応じて設定を復元
git config user.name "あなたの名前"
git config user.email "your-email@example.com"

# 6. 退避した作業を復元（必要な場合）
git stash list  # 退避した作業を確認
git stash apply  # 作業を復元
```

#### クローン時のよくある問題
```bash
# 権限エラーの場合
git clone https://username:token@gitlab.com/username/project.git

# 大きなリポジトリの場合（浅いクローン）
git clone --depth 1 https://gitlab.com/username/project.git

# 特定のブランチのみクローン
git clone -b specific-branch https://gitlab.com/username/project.git

# サブモジュールも含めてクローン
git clone --recursive https://gitlab.com/username/project.git
```

### OneDriveでのパス長問題と対策

#### OneDriveパス長問題の概要
OneDrive上でGitリポジトリをクローンする際、Windowsの260文字パス制限により以下の問題が発生することがあります：

- ビルドエラー
- ファイルアクセスエラー  
- npmインストールの失敗
- 深いディレクトリ構造での操作不能

#### 対策方法

##### 1. クローン場所の工夫
```bash
# ❌ 問題のあるパス例
C:\Users\Username\OneDrive - Company\Documents\Projects\very-long-project-name\

# ✅ 推奨パス例
C:\dev\project-name\
C:\workspace\proj\
D:\git\project\
```

##### 2. シンボリックリンクの活用
```bash
# PowerShell（管理者権限で実行）
# OneDrive上のプロジェクトに短いパスでアクセス
New-Item -ItemType SymbolicLink -Path "C:\proj" -Target "C:\Users\Username\OneDrive\Documents\Projects\long-project-name"

# 以降は短いパスでアクセス可能
cd C:\proj
```

##### 3. Git設定での対策
```bash
# 長いパスを有効化（Windows 10 バージョン1607以降）
git config --global core.longpaths true

# Windows設定でも長いパス名を有効化
# レジストリエディタで以下を設定：
# HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem
# LongPathsEnabled を 1 に設定
```

##### 4. 代替クローン戦略
```bash
# 1. 浅いクローンで最初は軽量化
git clone --depth 1 https://gitlab.com/username/project.git C:\dev\proj
cd C:\dev\proj

# 2. 必要に応じて履歴を取得
git fetch --unshallow

# 3. 特定のディレクトリのみクローン（sparse-checkout）
git clone --filter=blob:none https://gitlab.com/username/project.git
cd project
git sparse-checkout init --cone
git sparse-checkout set src/ docs/  # 必要なディレクトリのみ
```

##### 5. ビルドツール固有の対策
```bash
# Node.js プロジェクトの場合
# package.jsonでフラットな依存関係を設定
npm config set prefer-flat true
npm install --legacy-peer-deps

# .npmrc ファイルでnode_modulesの場所を変更
echo "prefix=C:\\npm-global" > .npmrc

# 環境変数でnpmキャッシュの場所を短縮
npm config set cache C:\npm-cache
```

##### 6. OneDrive同期の除外設定
```bash
# OneDriveの同期から特定のフォルダを除外
# 1. OneDriveアプリを開く
# 2. 設定 > アカウント > フォルダーの選択
# 3. node_modules, .git, build などを同期対象から除外

# または .gitignore でOneDrive固有ファイルを除外
echo "*.tmp" >> .gitignore
echo "desktop.ini" >> .gitignore
echo ".DS_Store" >> .gitignore
```

##### 7. 推奨フォルダ構造
```
C:\
├── dev\              # 開発プロジェクト用（短いパス）
│   ├── proj1\
│   ├── proj2\
│   └── ...
├── workspace\         # 一時作業用
└── Users\Username\OneDrive\  # ドキュメントのみ
    └── Documents\
        └── backup\    # 完成したプロジェクトのバックアップのみ
```

## マージとリベース

### マージ
```bash
# ブランチをマージ
git checkout main
git merge feature-branch

# マージコミットを作成せずにマージ（Fast-forward）
git merge --ff-only feature-branch

# 必ずマージコミットを作成
git merge --no-ff feature-branch
```

### リベース
```bash
# 現在のブランチをmainブランチにリベース
git rebase main

# インタラクティブリベース（コミットの編集）
git rebase -i HEAD~3

# リベースの中断
git rebase --abort

# リベースの続行
git rebase --continue
```

## 履歴の確認

### ログの表示
```bash
# コミット履歴を表示
git log

# 1行でコミット履歴を表示
git log --oneline

# グラフ形式で表示
git log --graph --oneline --all

# 特定のファイルの履歴
git log filename.txt

# 特定の期間のログ
git log --since="2024-01-01" --until="2024-12-31"

# 特定の作者のログ
git log --author="作者名"
```

### 差分の確認
```bash
# 2つのコミット間の差分
git diff commit1 commit2

# ブランチ間の差分
git diff main feature-branch

# 特定のコミットの内容
git show commit-hash
```

## 変更の取り消し

### ワーキングディレクトリの変更を取り消し
```bash
# 特定のファイルの変更を取り消し
git checkout -- filename.txt
# または
git restore filename.txt

# すべての変更を取り消し
git checkout -- .
# または
git restore .
```

### ステージングエリアから取り消し
```bash
# 特定のファイルをアンステージ
git reset HEAD filename.txt
# または
git restore --staged filename.txt

# すべてのファイルをアンステージ
git reset HEAD
```

### コミットの取り消し
```bash
# 最後のコミットを取り消し（変更は保持）
git reset --soft HEAD~1

# 最後のコミットを取り消し（変更も破棄）
git reset --hard HEAD~1

# 特定のコミットまで戻る
git reset --hard commit-hash

# コミットを打ち消す新しいコミットを作成
git revert commit-hash
```

## よく使うワークフロー

### 基本的な開発フロー
```bash
# 1. 最新の変更を取得
git pull origin main

# 2. 新しいブランチを作成
git checkout -b feature/new-feature

# 3. 開発作業
# ファイルを編集...

# 4. 変更をコミット
git add .
git commit -m "新機能を追加"

# 5. リモートにプッシュ
git push -u origin feature/new-feature

# 6. マージリクエストを作成

# 7. マージ後、ローカルブランチを削除
git checkout main
git pull origin main
git branch -d feature/new-feature
```

### ホットフィックスのワークフロー
```bash
# 1. mainから緊急修正ブランチを作成
git checkout main
git pull origin main
git checkout -b hotfix/critical-bug

# 2. 修正作業
# バグを修正...

# 3. コミットとプッシュ
git add .
git commit -m "緊急バグ修正"
git push -u origin hotfix/critical-bug

# 4. mainにマージ後、ブランチを削除
```

## トラブルシューティング

### よくある問題と解決方法

#### マージコンフリクトの解決
```bash
# コンフリクトが発生した場合
git status  # コンフリクトファイルを確認

# ファイルを編集してコンフリクトを解決
# <<<<<<<, =======, >>>>>>> マーカーを削除

# 解決後、変更をステージング
git add conflicted-file.txt

# マージを完了
git commit
```

#### 間違ったコミットの修正
```bash
# 最後のコミットメッセージを修正
git commit --amend -m "修正されたメッセージ"

# 最後のコミットにファイルを追加
git add forgotten-file.txt
git commit --amend --no-edit
```

#### プッシュできない場合
```bash
# リモートの変更を先に取得
git pull origin main

# コンフリクトがある場合は解決してから再プッシュ
git push origin main
```

#### 誤って削除したファイルの復元
```bash
# 特定のコミットからファイルを復元
git checkout commit-hash -- filename.txt

# 最後のコミットからファイルを復元
git checkout HEAD -- filename.txt
```

### 緊急時の対処法
```bash
# 作業を一時的に退避
git stash

# 退避した作業を復元
git stash pop

# 退避リストを確認
git stash list

# 特定の退避を適用
git stash apply stash@{0}
```

#### OneDrive環境でのトラブルシューティング
```bash
# パス長エラーが発生した場合の対処
# 1. 短いパスに移動
move "C:\Users\Username\OneDrive\Documents\Projects\very-long-name" "C:\dev\proj"

# 2. 新しい場所でGit操作を続行
cd C:\dev\proj
git status

# 3. OneDriveの同期設定を調整
# OneDriveアプリ > 設定 > 同期と バックアップ > フォルダーの選択
# 開発用フォルダを同期対象から除外

# 4. ビルドエラーが発生する場合
# node_modules の場所を変更
npm config set prefix C:\npm-global
npm config set cache C:\npm-cache

# 5. 権限エラーの場合
# 管理者権限でコマンドプロンプトを開いて実行
git config --global core.longpaths true
```

## 便利なTips

### エイリアスの設定
```bash
# よく使うコマンドのエイリアス
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual '!gitk'
```

### .gitignoreファイル
```gitignore
# 依存関係
node_modules/
*.log

# ビルド成果物
dist/
build/

# 環境設定
.env
.env.local

# IDE設定
.vscode/
.idea/

# OS固有ファイル
.DS_Store
Thumbs.db

# 一時ファイル
*.tmp
*.swp
```

### 便利なコマンド組み合わせ
```bash
# 変更されたファイルのみを表示
git diff --name-only

# 最後のコミット以降の変更ファイル
git diff --name-only HEAD~1

# 特定の文字列を含むコミットを検索
git log --grep="検索文字列"

# ファイル内の特定の行の履歴を追跡
git blame filename.txt

# ファイルの変更履歴を詳細表示
git log -p filename.txt

# 削除されたファイルを検索
git log --diff-filter=D --summary
```

### パフォーマンス向上
```bash
# ガベージコレクション
git gc

# リポジトリの最適化
git gc --aggressive

# リモートブランチの情報を更新
git remote prune origin
```

## GitLabとの連携

### GitLab特有の機能
```bash
# マージリクエスト用のブランチプッシュ
git push -u origin feature-branch

# GitLab CIでよく使われるタグ
git tag -a v1.0.0 -m "バージョン1.0.0リリース"
git push origin v1.0.0

# GitLab CI/CDパイプライン実行のトリガー
git push origin main  # .gitlab-ci.ymlが存在する場合、自動実行

# GitLab Pagesへのデプロイ（静的サイト）
git push origin main  # pagesジョブが設定されている場合
```

### GitLab特有のリモートURL形式
```bash
# HTTPS形式（推奨）
git remote add origin https://gitlab.com/username/repository.git

# SSH形式（SSH鍵設定済みの場合）
git remote add origin git@gitlab.com:username/repository.git

# GitLab自体がセルフホストされている場合
git remote add origin https://your-gitlab-instance.com/username/repository.git
```

### GitLab Flowの推奨ワークフロー
1. **Issue作成**: 作業内容を明確にし、ラベルやマイルストーンを設定
2. **ブランチ作成**: Issueに基づいてfeatureブランチを作成（Issue番号を含む）
3. **開発**: 小さなコミットを心がけ、コミットメッセージでIssueを参照
4. **マージリクエスト**: レビューを依頼し、適切なレビュアーを指定
5. **CI/CDパイプライン**: 自動テストとビルドの実行を確認
6. **レビューと修正**: フィードバックに対応し、必要に応じて追加コミット
7. **マージ**: mainブランチに統合（Squash mergeやRebase mergeも選択可能）
8. **自動デプロイ**: GitLab CI/CDによる本番環境への自動デプロイ

### GitLabでのIssue管理
```bash
# コミットメッセージでIssueを参照
git commit -m "feat: ユーザー認証機能を追加 (refs #123)"

# コミットでIssueを自動クローズ
git commit -m "fix: ログインエラーを修正 (closes #456)"

# マージリクエストでIssueを参照
git commit -m "feature: 新機能実装 (see merge request !789)"
```

## セキュリティのベストプラクティス

### 機密情報の管理
```bash
# 機密ファイルを履歴から完全削除
git filter-branch --force --index-filter \
'git rm --cached --ignore-unmatch secret-file.txt' \
--prune-empty --tag-name-filter cat -- --all

# または git-filter-repo を使用（推奨）
git filter-repo --path secret-file.txt --invert-paths
```

### コミット署名
```bash
# GPG署名の設定
git config --global commit.gpgsign true
git config --global user.signingkey YOUR_GPG_KEY_ID
```

## まとめ

このガイドでは、Gitの基本的な使い方から応用まで幅広くカバーしました。日常的な開発作業では以下のコマンドを覚えておくと便利です：

- `git status` - 現在の状態確認
- `git add` - ファイルをステージング
- `git commit` - 変更をコミット
- `git push` - リモートに送信
- `git pull` - リモートから取得
- `git branch` - ブランチ操作
- `git merge` - ブランチをマージ

困ったときは `git help <command>` でヘルプを確認できます。

---

**参考リンク**:
- [Git公式ドキュメント](https://git-scm.com/docs)
- [GitLab Docs](https://docs.gitlab.com/)
- [Pro Git Book（日本語版）](https://git-scm.com/book/ja/v2)