---
description: Figma デザイントークンを Tailwind CSS に同期するスキル
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash]
---

# Figma デザイントークンを Tailwind CSS に同期

このスキルは、Figma で定義されたデザイントークン（変数）を JSON ファイルから読み込み、Tailwind CSS で利用できる形式に変換して、プロジェクトに統合します。

## 前提条件

- Figma 変数定義が `.figma/Library.json` にエクスポートされていること
  - 対象: [Figma ファイル（ライブラリ）](https://www.figma.com/design/YOUR_FILE_KEY/Library)
- Typography データは Figma MCP サーバー経由で自動取得されます
  - 対象: [Figma タイポグラフィ（ライブラリ）](https://www.figma.com/design/YOUR_FILE_KEY/Library?node-id=560-46&m=dev)

## 実行フロー

以下のフェーズに分けて実行します。各フェーズ完了後、ユーザーに確認を求めてから次フェーズに進みます。

### Phase 1: 準備

#### 1-1. 更新対象の選択

ユーザーに更新対象を選択させる（デフォルト：両方）：

- **JSON変数**：Primitive/Semantic カラー、Number変数、Font Family を更新
- **Typography**：fontSize 定義（サイズ、行間、ウェイト）を更新
- **両方**：全て処理（デフォルト、従来通りの動作）

選択に応じて Phase 2-5 の処理範囲を制限し、実行時間を短縮します。

#### 1-2. JSON ファイルの最新化確認

- `.figma/Library.json` の存在と最新性を確認（JSON変数 または 両方 を選択時のみ）
- 未更新の場合は [`.figma/README.md`](../.figma/README.md) を参照し、Export/Import Variables プラグインでエクスポート

### Phase 2: データの解析

選択した更新対象に応じて処理範囲を制限：

**JSON変数を選択した場合:**
- `.figma/Library.json` から `variables` 配列を取得
- Primitive（基本カラー）、Semantic（意味的変数）、Number（数値）、Font Family に分類
- `app/globals.css` と `tailwind.config.ts` との差分を特定（新規/更新/削除/typo）

**Typography を選択した場合:**
- Figma MCP サーバーから Typography データを取得（サイズ、行間、ウェイト）
- `tailwind.config.ts` の fontSize セクションとの差分を特定

**両方を選択した場合:**
- 上記の両方を実行

**差分確認と提示:**
- 選択範囲の差分サマリーをユーザーに提示

### Phase 3: データ変換

選択した更新対象のデータのみ変換処理を実行：

**JSON変数を選択した場合:**
- **変数名変換:** `Semantic/TextAndIcon/Plane/Heading` → `--semantic-text-icon-plane-heading`（スラッシュをハイフン、小文字化）
- **カラー変換:** RGB → HSL 形式（`0 0% 23%`）で shadcn/ui 互換性維持、`<alpha-value>` 構文対応
- **エイリアス解決:** `alias` フィールドから参照先の値を解決
- **フォントファミリー:** STRING 型から CSS 変数形式に変換

**Typography を選択した場合:**
- **Typography 定義:** Figma MCP から取得したデータを Tailwind CSS 形式に変換（Headline, Title, Body, Label のサイズ・行間・ウェイト）

**両方を選択した場合:**
- 上記の両方を実行

### Phase 4: 更新内容の提案と承認

選択した更新対象の変更内容のみを提示し、ユーザーの承認を求める：

**JSON変数を選択した場合:**
- CSS 変数定義のサンプル（Primitive/Semantic/Number）
- Tailwind colors 定義のサンプル（semantic, primitive）
- fontFamily 設定のサンプル
- borderRadius 設定のサンプル
- typo 修正の提案

**Typography を選択した場合:**
- fontSize 設定のサンプル（サイズ、行間、ウェイト）

**両方を選択した場合:**
- 上記の全て

### Phase 5: ファイル更新

選択した更新対象のセクションのみを更新し、非選択範囲は既存値を保持：

#### 順序維持の原則（重要）

実行するたびに変数やキーの順序が変わると不要な差分が大量発生するため、以下のルールを厳守してください：

**globals.css の更新ルール:**
- カテゴリ内の変数は数字順にソート - 030, 050, 100, 200, 300... の昇順で並べる
- 新規変数の追加 - 同じカテゴリ（`/* Primitive > Gray */` など）内に数字順で挿入
- カテゴリコメントの保持 - 既存のカテゴリコメント（`/* Primitive > Gray */`）とその位置を維持
- 削除された変数 - 完全に削除する（コメントアウトせず、行ごと削除）
- カテゴリの順序 - Blue, Bluegray, Gray, Green, Parple, Red, WhiteAlpha, Black, Transparent の順序を維持
- 避けるべき: カテゴリ順序の変更

**コメントの書き方ルール:**
- エイリアス変数の場合: `/* via Primitive/Gray/050 */` のように参照元を記載
  - 例: `--semantic-background-component-hover: 227 100% 98%; /* via Primitive/Blue/030 */`
- 直接値の場合: RGB 16進数を記載（`/* #FFFFFF */`）
  - 例: `--primitive-blue-030: 227 100% 98%; /* #F6F8FF */`
- 透明度がある場合: RGB 16進数のみ記載（透明度は値に含まれる）
  - 例: `--primitive-whitealpha-030: 0 0% 100% / 0.1; /* #FFFFFF */`
- Numbers の場合: コメント不要
  - 例: `--number-icon-hero: 80px;`
- 避けるべき: コメント形式の変更、エイリアス情報の削除

**tailwind.config.ts の更新ルール:**
- 数字キーは昇順でソート - "030", "050", "100", "200"... の順序で並べる
- カテゴリキーの順序を維持 - blue, bluegray, gray, green, parple, red の順序を保持
- 新規キーの追加 - 同じネストレベル内に適切な位置（数字順）で挿入
- 値の更新のみ - キーの順序は変更しない
- フォーマット保持 - 既存のインデントやスタイルを維持
- 避けるべき: カテゴリキーの並び替え、オブジェクト構造の再編成

**実装手順:**
1. 既存ファイルを読み込み、現在の変数/キーの順序を記録
2. Figma データと既存データをマージ（既存順序を優先）
3. 変更は最小限に留め、不要な差分を発生させない

---

**JSON変数を選択した場合:**

`globals.css` の更新:
- 既存の shadcn/ui 定義を保持
- Figma Design Tokens セクションを追加・更新（Primitive/Semantic/Number カテゴリ）
- 削除された変数は完全に削除

`tailwind.config.ts` の更新:
- 既存の色定義（background, foreground など）は保持
- semantic/primitive 色定義を追加・更新（`hsl(var(...) / <alpha-value>)` 形式）
- fontFamily を更新
- borderRadius を更新
- **fontSize は更新しない**（既存値を保持）

**Typography を選択した場合:**

`tailwind.config.ts` の更新:
- fontSize のみを Figma MCP データで更新（`[size, { lineHeight, fontWeight }]` 形式）
- **colors, fontFamily, borderRadius は更新しない**（既存値を保持）

`globals.css` の更新:
- **更新しない**（既存値を保持）

**両方を選択した場合:**

`globals.css` の更新:
- 既存の shadcn/ui 定義を保持
- Figma Design Tokens セクションを追加・更新（カテゴリ別コメント付き）
- 削除された変数は完全に削除

`tailwind.config.ts` の更新:
- 既存の色定義（background, foreground など）は保持
- semantic/primitive 色定義を追加・更新（`hsl(var(...) / <alpha-value>)` 形式）
- fontFamily, borderRadius を更新
- fontSize を Figma MCP データで更新（`[size, { lineHeight, fontWeight }]` 形式）

**完了確認:**
- 選択範囲のファイル差分表示
- 変更サマリー報告（追加/更新/削除数）

### Phase 6: フォーマット実行

ファイル更新完了後、コードフォーマットを自動実行：

```bash
pnpm fmt
```

## 注意事項

- 既存の shadcn/ui 定義（background, foreground, primary など）は保持
- ダークモード対応は `.dark` セレクタで実装可能
- **Typography は Figma MCP 経由で自動同期**（フォントファミリーは Library.json、サイズ・行間・ウェイトは MCP）
- Figma 側の typo（Secound, Parple など）に従うが、修正を提案
