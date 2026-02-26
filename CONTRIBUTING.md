# コントリビューションガイド

ai-toolkit へのコントリビューションに関心をお持ちいただきありがとうございます。
このガイドでは、貢献方法と規約について説明します。

## 歓迎するコントリビューション

- **スキルの改善** — 既存スキルの精度向上、エラー処理の改善
- **新規スキルの追加** — Figma・Claude Code を活用した新しいワークフローの提案
- **ドキュメント改善** — README・コメントの修正、利用例の追加
- **バグ報告** — 動作しないケースの Issue 報告

---

## 開発フロー

### 1. Fork とクローン

```bash
# リポジトリを Fork した後
git clone https://github.com/<your-username>/ai-toolkit.git
cd ai-toolkit
```

### 2. ブランチを作成

ブランチ名はプレフィックスを付けて作成してください。

| プレフィックス | 用途 |
|---|---|
| `feat/` | 新機能・新規スキルの追加 |
| `fix/` | バグ修正 |
| `docs/` | ドキュメントのみの変更 |

```bash
# 例
git checkout -b feat/figma-component-audit
git checkout -b fix/sync-design-tokens-hsl
git checkout -b docs/setup-guide
```

### 3. 変更を加えてコミット

[Conventional Commits](#conventional-commits-規約) に従ってコミットしてください。

### 4. Pull Request を作成

`main` ブランチに向けて PR を作成し、[PR ガイドライン](#pr-ガイドライン) に沿って説明を記入してください。

---

## Conventional Commits 規約

コミットメッセージは以下のフォーマットに従ってください。

```
type(scope): 変更内容の説明（日本語）
```

### type 一覧

| type | 用途 |
|---|---|
| `feat` | 新機能・新規スキルの追加 |
| `fix` | バグ修正 |
| `docs` | ドキュメントのみの変更 |
| `refactor` | 機能変更を伴わないコードの整理 |
| `test` | テストの追加・修正 |
| `chore` | ビルド・ツール設定の変更 |

### scope 例

| scope | 対象 |
|---|---|
| `figma-to-code` | figma-to-code スキル |
| `sync-design-tokens` | sync-design-tokens スキル |
| `docs` | ドキュメント全般 |

### コミットメッセージ例

```
feat(figma-to-code): 既存コンポーネントの再利用判定ロジックを追加
fix(sync-design-tokens): HSL 変換時の小数点丸め処理を修正
docs: セットアップ手順にシンボリックリンクの例を追加
chore: .gitignore に OS 生成ファイルを追加
```

---

## スキルファイルのフォーマット規約

### 配置場所

スキルファイルは `claude/skills/` ディレクトリに配置してください。

```
claude/
└── skills/
    ├── figma-to-code.md
    └── sync-design-tokens.md   ← ここに追加
```

### ファイル名

kebab-case の `.md` ファイルとして作成してください。

```
# 良い例
figma-component-audit.md
sync-design-tokens.md

# 悪い例
FigmaComponentAudit.md
sync_design_tokens.md
```

### Frontmatter

ファイルの先頭に YAML frontmatter を記述してください。

```yaml
---
description: スキルの概要を 1 行で説明
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash]
---
```

| フィールド | 必須 | 説明 |
|---|---|---|
| `description` | 必須 | スキルの概要（Claude Code のスキル一覧に表示される） |
| `allowed-tools` | 推奨 | スキルが使用するツールのリスト |

### プレースホルダーの使用

スキルはプロジェクト非依存に設計してください。プロジェクト固有のパスや設定は、プレースホルダーを使って利用者がカスタマイズできるようにします。

**プレースホルダーの形式:** `{{PLACEHOLDER_NAME}}`

```markdown
# 良い例（プロジェクト非依存）
コンポーネントは `{{COMPONENT_DIR}}` に生成してください。
設定ファイルは `{{PROJECT_ROOT}}/tailwind.config.ts` を参照してください。

# 悪い例（プロジェクト固有パスのハードコード）
コンポーネントは `src/components/ui/` に生成してください。
設定ファイルは `/Users/yuta/my-project/tailwind.config.ts` を参照してください。
```

**よく使われるプレースホルダー例:**

| プレースホルダー | 説明 |
|---|---|
| `{{PROJECT_ROOT}}` | プロジェクトのルートディレクトリ |
| `{{COMPONENT_DIR}}` | コンポーネントの配置ディレクトリ |
| `{{STYLES_DIR}}` | スタイルファイルの配置ディレクトリ |
| `{{FIGMA_FILE_KEY}}` | Figma ファイルの識別子 |

---

## PR ガイドライン

- **変更を絞る** — 1 つの PR で 1 つのスキルや機能に集中してください。複数の無関係な変更は別の PR に分けてください。
- **説明を明確に** — PR の説明には変更の背景・目的・動作確認の方法を記載してください。
- **Issue を参照** — 関連する Issue がある場合は `Closes #123` や `Refs #456` の形式で参照してください。

### PR 説明のテンプレート

```markdown
## 変更内容

<!-- 何を変更したか、なぜ変更したかを説明してください -->

## 動作確認

<!-- 変更を確認した手順を記載してください -->

## 関連 Issue

<!-- Closes #123 -->
```

---

## 質問・サポート

質問やフィードバックは [GitHub Issues](https://github.com/boost-jp/ai-toolkit/issues) にお気軽にどうぞ。
バグ報告の際は、再現手順と環境情報（OS、Claude Code バージョンなど）を添えてください。
