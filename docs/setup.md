# Setup — 初回セットアップ詳細

このドキュメントは [README.md](../README.md) の「Use this template したらやること」を補足する。

## 前提

- GitHub アカウント（本テンプレを派生させる先のリポジトリを作成できる）
- ChatGPT Plus / Pro / Team アカウント（Codex Web 版を使うため）
- ローカルに `git` と `gh` CLI
- bash 4 以上（macOS デフォルトの 3.2 でも `apply-template.sh` は動くが `mapfile` の代替実装が要るので、できれば `brew install bash` 推奨）

## 詳細手順

### 1. テンプレ派生

GitHub の本テンプレリポジトリページで **"Use this template" → "Create a new repository"** をクリック。
- Owner：自分の user / org
- Repository name：新プロジェクト名
- Public / Private：任意
- "Include all branches" は OFF（main のみで OK）

### 2. clone と placeholder 置換

```sh
git clone git@github.com:<your-org>/<new-repo>.git
cd <new-repo>
bash scripts/apply-template.sh
```

対話で以下を聞かれる：

| 質問 | 例 | デフォルト |
|---|---|---|
| GitHub owner | `your-handle` | （なし） |
| Repository name | `<new-repo>` | （なし） |
| Human-readable project name | `My New Project` | （なし） |
| Default branch | `main` | `main` |
| Task branch prefix | `task` / `feature` | `task` |
| Build / verify command | `npm run build` / `cmake --build build` | （なし） |
| Test / lint command | `npm test` | （空可） |
| Screenshot directory | `docs/screenshots` | `docs/screenshots` |
| App binary hint | `./build/app` 等の起動説明 | （なし） |
| Codex bot login | `chatgpt-codex-connector[bot]` | 同左 |
| Review language | `日本語` / `English` | `日本語` |

完了後、placeholder 残存検査が走り、すべて置換されていれば `✅ 全 placeholder が置換されました。` と出る。

### 3. CLAUDE.md の埋め込み

`CLAUDE.md.template` → `CLAUDE.md` にリネームされた後、HTML コメント（`<!-- ... -->`）で示された箇所を実プロジェクトの内容に書き換える：

- プロジェクト概要
- 現在のフェーズ
- ビルドと実行
- アーキテクチャ
- 「利用可能なエージェント」表（プロジェクト固有エージェントを足す）
- Codex 連携の重点観点（AGENTS.md 側にも同じセクションあり）
- 「ビルド / 環境の罠」（最初は空、kaizen で蓄積していく）

### 4. ChatGPT Codex 連携

1. [chatgpt.com/codex](https://chatgpt.com/codex) を開く
2. 右上アバター → **Settings** → **Connectors / GitHub**
3. GitHub OAuth → 新リポジトリへの権限を付与
4. GitHub 側の **Settings → Installed Apps** に "ChatGPT" / "OpenAI Codex" 等が出ていれば成功

### 5. gh CLI 認証

```sh
gh auth status
```

未認証なら：

```sh
gh auth login
# → GitHub.com → HTTPS → ブラウザでログイン
```

### 6. 疎通確認（最初のダミー PR）

```sh
git checkout -b task/0-smoke
# 適当な変更（README に空行を足す等）
git add README.md
git commit -m "Task 0: smoke test"
git push -u origin task/0-smoke
gh pr create --fill
gh pr comment <PR番号> --body "@codex review"
```

2〜5 分で Codex が PR コメントで日本語のレビューを返せば成功。返ってこない場合は [troubleshooting](#troubleshooting) を参照。

## Troubleshooting

### Codex が無反応（30 分経っても）

- chatgpt.com の Codex Settings で GitHub 連携が切れていないか確認
- 該当リポジトリへの権限が付与されているか（GitHub Installed Apps）
- Codex の自動レビュー設定が ON か（Codex Web 上の "Auto-review pull requests" 等）
- `@codex review` コメントを再度投稿してみる

### `apply-template.sh` が `mapfile: command not found`

- macOS デフォルトの bash 3.2 で起きる。`brew install bash` して `/opt/homebrew/bin/bash scripts/apply-template.sh` で実行する

### placeholder が一部置換されない

- `{{TASK_LIST_PLACEHOLDER}}` のような **意図的に残してある placeholder** はコメント内に置かれており、利用者が手で内容を埋める。詳細は [customize.md](customize.md) を参照
