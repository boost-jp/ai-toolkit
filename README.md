# AI Toolkit

BOOST の AI 開発支援ツールキットです。Claude Code Skills、スクリプト、設定ファイルなどを集約管理します。

## 構成

```
ai-toolkit/
├── claude/
│   └── skills/           # Claude Code Skills
│       ├── figma-to-code.md        # Figma デザイン → UI コンポーネント実装
│       └── sync-design-tokens.md   # Figma デザイントークン → Tailwind CSS 同期
└── README.md
```

## Claude Code Skills

### `/figma-to-code` - Figma デザインから UI 実装

Figma デザインから UI コンポーネントの要件定義を行い、実装を支援するスキルです。

**主な機能:**
- Figma MCP 経由でデザイン情報を自動取得
- 既存コンポーネントの再利用判定（fileKey + node-id による一意特定）
- コンポーネントツリーの生成と Plan モードによる承認フロー
- 新規コンポーネントのみ詳細取得によるコンテキスト削減

**技術スタック:** React / Next.js, Tailwind CSS, TypeScript, Figma MCP

### `/sync-design-tokens` - デザイントークン同期

Figma で定義されたデザイントークン（Variables）を Tailwind CSS に同期するスキルです。

**主な機能:**
- Figma Variables の JSON → CSS 変数（HSL 形式）変換
- Tailwind config への自動統合
- Typography データの MCP 経由自動同期
- 差分検出による最小限の更新

## セットアップ

### Skills の利用方法

プロジェクトの `.claude/commands/` にスキルファイルをコピーまたはシンボリックリンクを作成してください。

```bash
# 例: シンボリックリンクで配置
ln -s /path/to/ai-toolkit/claude/skills/figma-to-code.md .claude/commands/figma-to-code.md
ln -s /path/to/ai-toolkit/claude/skills/sync-design-tokens.md .claude/commands/sync-design-tokens.md
```

### 前提条件

- [Figma Desktop](https://www.figma.com/downloads/) がインストール・起動されていること
- [Figma MCP](https://www.figma.com/community/plugin/figma-mcp) が設定されていること
- [Export/Import Variables](https://www.figma.com/community/plugin/1256972111705530093/export-import-variables) プラグイン（sync-design-tokens 用）

## 参考

- [Figma デザインから実装まで - Claude Code Skills による UI 自動生成の精度を劇的に上げる方法](https://note.com/meru_ra/n/n7b3cbcf1d934)
