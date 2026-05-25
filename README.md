# Claude Codex Ops Template

「**実装は Claude Code、レビューは ChatGPT Codex GitHub App**」運用を新規プロジェクトで即開始するための GitHub Template Repository。

[`hang-up33/hmi-platform`](https://github.com/hang-up33/hmi-platform) で実運用されている `.claude/` 配下のスキル / エージェント、`AGENTS.md`、PR・ブランチ・スクリーンショット運用ルール、汎用補助スクリプトを言語非依存に汎用化したもの。

---

## Use this template したらやること（5 分）

1. **GitHub UI** で "Use this template" → 新規リポジトリを作成
2. **ローカルに clone**：
   ```sh
   git clone git@github.com:<your-org>/<new-repo>.git
   cd <new-repo>
   ```
3. **placeholder を一括置換**：
   ```sh
   bash scripts/apply-template.sh
   ```
   対話で `{{OWNER}}` / `{{REPO}}` / `{{BUILD_CMD}}` 等を聞かれるので答える。完了後、`CLAUDE.md.template` → `CLAUDE.md` のリネームを促されるので Yes。
4. **`CLAUDE.md` を埋める**：プロジェクト概要・アーキテクチャ・ビルド手順を追記（テンプレ骨格にコメントで指示が入っている）。
5. **ChatGPT Codex Web で連携**：[chatgpt.com/codex](https://chatgpt.com/codex) → Settings → GitHub OAuth → 新リポジトリを連携。
6. **gh CLI 認証**：`gh auth status` で OK か確認。
7. **最初のダミー PR で疎通確認**：適当な変更を `task/0-smoke` ブランチで PR にして、Codex が日本語コメントを返すか確認。

---

## 含まれる資産

```
.claude/
├── settings.json                       # autoCompactEnabled のみ
├── skills/
│   ├── kaizen-close/SKILL.md           # タスク完了直前に kaizen を反映
│   ├── codex-pr/SKILL.md               # PR 作成 + Codex 自走レビューループ
│   └── next-task/SKILL.md              # 次タスク実装の標準ワークフロー
└── agents/
    └── build-error-resolver.md         # ビルド/依存エラー解決の汎用エージェント

.github/
├── PULL_REQUEST_TEMPLATE.md            # PR 本文の雛形（Summary / 変更点 / Test plan / Codex 向け補足）
└── ISSUE_TEMPLATE/                     # 任意

scripts/
├── apply-template.sh                   # placeholder 一括置換
└── capture-app-window.sh               # macOS のウィンドウキャプチャ（GUI アプリのスクショ用）

examples/
└── qt6/                                # Qt6 + CMake プロジェクト用の参考実装
                                        # （不要なら apply-template.sh の最後で削除可）

docs/
├── setup.md                            # 初回セットアップ詳細
├── workflow.md                         # 開発フロー（task ブランチ → PR → 自走レビュー）
└── customize.md                        # placeholder 一覧と書き換え指針

CLAUDE.md.template                      # Claude Code への指針の雛形
AGENTS.md                               # Codex GitHub App / Codex CLI への指示
```

---

## 主要ワークフロー

```
ユーザー: 次のタスクを進めて
   ↓
Claude (next-task SKILL)
   ↓ ブランチ作成 → 実装 → ビルド検証
   ↓
Claude (kaizen-close SKILL)
   ↓ 学びを CLAUDE.md / README に反映
   ↓
Claude (codex-pr SKILL)
   ↓ commit → push → gh pr create → @codex review
   ↓
ChatGPT Codex (PR コメントで日本語レビューを自動投稿)
   ↓
Claude (codex-pr SKILL の自走ループ)
   ↓ 指摘修正コミット → @codex review 再依頼 → 指摘 0 件まで繰り返し
   ↓
ユーザー: Codex のクリーン後にマージ
```

---

## カスタマイズ

- **placeholder 一覧**は [docs/customize.md](docs/customize.md) 参照。
- **`build-error-resolver` の「既知の罠リスト」** は最初は空。プロジェクトで踏んだ罠を `kaizen-close` 経由で追記していくと、本エージェントが早く解決できるようになる。
- **Qt6 / CMake プロジェクトでない場合**は `examples/qt6/` を丸ごと削除して可（`apply-template.sh` の最後で確認プロンプトが出る）。
- **CI を足したい場合**は `.github/workflows/` を任意に追加。本テンプレ自体は CI を強制しない。

---

## Maintenance — hmi-platform からの取り込み運用

本テンプレートは [`hang-up33/hmi-platform`](https://github.com/hang-up33/hmi-platform) を元に切り出されているため、上流で新しい kaizen / 罠 / ワークフロー改善が発生した時に、**汎用化に値するもの**を本テンプレ側にも反映していく。

### 取り込みの判断基準（hmi-platform 側で kaizen を発見した時）

| 上流の変更 | テンプレへの取り込み | 理由 |
|---|---|---|
| Qt / CMake / QML の罠 | **取り込まない**（`examples/qt6/` にのみ反映） | プロジェクト固有 |
| Codex 自走ループの判定ロジック改善（`codex-pr` SKILL 手順 7） | **取り込む** | 全プロジェクトに価値 |
| `kaizen-close` の反映先選定ルール変更 | **取り込む** | 全プロジェクトに価値 |
| PR 本文フォーマットの改善 | **取り込む**（PR テンプレと `codex-pr` SKILL 手順 5 を同期） | 全プロジェクトに価値 |
| ブランチ命名 / マージ戦略の変更 | **取り込む**（placeholder を介して反映） | 全プロジェクトに価値 |
| MVP タスク順 / フェーズ管理の具体内容 | **取り込まない**（プロジェクト固有） | hmi-platform 専用 |
| 新規 placeholder の追加 | **取り込む**（`scripts/apply-template.sh` と `docs/customize.md` も同期） | テンプレの拡張 |

### 取り込み手順

1. hmi-platform 側の該当 PR をレビューし、汎用化可能な箇所を特定
2. 本テンプレリポジトリで `task/sync-<topic>` ブランチを切る
3. 該当ファイルを汎用化（リテラル値 → placeholder 化、Qt 固有例は `examples/qt6/` に隔離）
4. 既存利用者への影響を `docs/customize.md` の changelog セクションに追記
5. PR 作成 → Codex レビュー → マージ

### バージョニング

- 本テンプレリポジトリは **タグでリリースを切る**（例：`v0.1.0`, `v0.2.0`）
- 大きな破壊的変更（placeholder の rename 等）は minor バージョンを上げる
- 既存の派生プロジェクトは "Use this template" 後にテンプレと切り離されるため、新版を取り込みたい場合は手動で差分を当てる（テンプレ更新の自動同期機構は GitHub には無いため）

---

## 関連リンク

- 元リポジトリ：[hang-up33/hmi-platform](https://github.com/hang-up33/hmi-platform)
- ChatGPT Codex：[chatgpt.com/codex](https://chatgpt.com/codex)
- Claude Code：[claude.com/claude-code](https://claude.com/claude-code)
