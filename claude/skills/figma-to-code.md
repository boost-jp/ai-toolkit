---
description: Figma デザインから UI コンポーネントの要件定義を行い、実装を支援するスキル
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, EnterPlanMode, ExitPlanMode]
---

# UI デザイン要件定義と実装

Figma デザインから UI コンポーネントの要件定義を行い、実装を支援するスキルです。

## 実行フロー

### Phase 1: 情報収集

以下の情報をユーザーに記入・選択させる

1. Figma の URL（node-id を含む）を確認
2. 参考にすべき API Inerface のファイル（任意）を確認

### Phase 2: 既存コンポーネントの確認

1. `/components/ui/` 配下のコンポーネントファイルを読み取り
2. 各ファイルの先頭コメントから Figma 情報（fileKey, node-id）を抽出
3. 新規追加する Figma ノードの情報と照合し、同一コンポーネントを特定
4. Figma 情報がないコンポーネントは、名前やプロパティから再利用可能性を判断

### Phase 3: Figma デザイン情報の取得

**新規コンポーネントのみ詳細取得**：

1. **URL から node-id を抽出**
   - URL パターン: `https://figma.com/design/:fileKey/:fileName?node-id=X-Y`
   - node-id 部分を抽出（例: `node-id=11-3265` → `11:3265`）

2. **proto ファイルの読み取り**（提供された場合のみ）
   - `proto/` 配下のファイルを読み取って型定義を参照

3. **既存コンポーネントで対応可能か判断**
   - 既存の `/components/ui/` で対応可能 → 名前とパスのみ記録
   - 新規作成必要 → 以下の MCP ツールを並列実行

4. **Figma MCP ツールでデザイン情報を取得**（新規コンポーネントのみ）

   #### Step 1: スクリーンショットとメタデータを取得

   ```typescript
   // 並列で呼び出す
   mcp__figma-desktop__get_screenshot({
     nodeId: "抽出したnode-id",
     clientLanguages: "typescript",
     clientFrameworks: "react"
   })

   mcp__figma-desktop__get_metadata({
     nodeId: "抽出したnode-id",
     clientLanguages: "typescript",
     clientFrameworks: "react"
   })

   mcp__figma-desktop__get_variable_defs({
     nodeId: "抽出したnode-id",
     clientLanguages: "typescript",
     clientFrameworks: "react"
   })
   ```

   #### Step 2: デザインコンテキストを段階的に取得

   `get_design_context` はレスポンスの Token 量が大きくなりやすいため、**段階的に分割取得**する。

   1. **まずルートノードの構造を取得**
      ```typescript
      mcp__figma-desktop__get_design_context({
        nodeId: "ルートのnode-id",
        clientLanguages: "typescript",
        clientFrameworks: "react"
      })
      ```

   2. **Token Limit エラーが発生した場合、子ノード単位で分割取得**
      - Step 1 で取得した `get_metadata` の結果から子ノードの node-id を特定
      - 各子ノードに対して個別に `get_design_context` を呼び出す
      ```typescript
      // 子ノードごとに並列で呼び出す
      mcp__figma-desktop__get_design_context({
        nodeId: "子ノード1のnode-id",
        clientLanguages: "typescript",
        clientFrameworks: "react"
      })
      mcp__figma-desktop__get_design_context({
        nodeId: "子ノード2のnode-id",
        clientLanguages: "typescript",
        clientFrameworks: "react"
      })
      // ...
      ```

   3. **それでも Token Limit エラーが発生する場合、さらに深い階層で分割**
      - エラーが出たノードの子ノードを特定し、再帰的に分割取得を繰り返す

   > **注意**: 取得した各パーツのデザイン情報を統合して、全体のコンポーネント構造を把握すること。

### Phase 4: 実装方針の提案

以下の情報を含む実装方針を **ExitPlanMode ツール** で提案し、ユーザーの承認を待ちます：

#### 提案内容

1. **デザイン概要**
   - Figma URL
   - Node ID
   - スクリーンショットの参照

2. **コンポーネント構造（ツリー形式）** [**最重要**]
   - 全コンポーネントを階層構造で表示
   - 各コンポーネントに（既存）または（新規）のラベルを付ける
   - 例:
     ```
     ページ名
     ├── Header（既存活用）
     │   ├── Logo（新規）
     │   └── Navigation（既存）
     └── Content
         ├── Card（既存）
         │   ├── CardTitle（既存）
         │   └── CustomIcon（新規）
         └── Footer（新規）
     ```

3. **既存コンポーネント活用リスト**
   - コンポーネント名、ファイルパス、用途

4. **新規作成コンポーネントリスト**
   - `/components/ui/` 配下（汎用・Storybook 必須）
   - `/app/components/` 配下（ページ固有）
   - アイコンやユーティリティ

5. **実装手順**
   - Phase 単位で段階的な実装計画

#### ⚠️ 重要な注意事項

**ExitPlanMode の plan パラメータには、上記の詳細なツリー構造を省略せずにすべて含めること。**
簡略化や要約は禁止。ユーザーが全体像を把握できるよう、完全な情報を提供すること。

### Phase 5: 実装

承認後、以下の順序で実装：

1. **TodoWrite ツールでタスク管理**
2. **コンポーネント実装**
   - Figma デザインをもとに実装
   - 既存パターンを踏襲
3. **Storybook ファイル作成**（/components/ui/ のみ）
   - 各コンポーネントの `.stories.tsx` を作成
4. **型定義作成**
   - proto ファイルがある場合は参照
5. **既存コードへの統合**
6. **完了報告**

## 配置ルール

| ディレクトリ | 対象 | 例                           | Storybook |
|-------------|------|-------------------------------|:----------|
| `/components/ui/` | Figma ライブラリから取り込んだコンポーネント | Button, Input, Messaging, Tag | 必須 |
| `/components/icons/` | Figma ライブラリの icon コンポーネント | Check, ChevronDown, TriangleAlert, X | 任意 |
| `/app/components/pages/` | ページごとに固有のコンポーネント | MemberCard, ProjectList       | 不要 |

**判断基準**：
- Figma ライブラリから取り込んだものは `/components/ui/`
- Figma ライブラリの icon は `/components/icons/`
- ページ固有の実装は `/app/components/pages/`

## Figma 連携ルール

### Figma 情報の記載方法

すべての `/components/ui/` および `/components/icons/` コンポーネントには、ファイル先頭に以下のフォーマットで Figma 情報をコメントとして記載します：

```typescript
/**
 * Figma Information:
 * fileKey: YOUR_LIBRARY_FILE_KEY
 * node-id: 88-2525
 * URL: https://www.figma.com/design/YOUR_LIBRARY_FILE_KEY/Library?node-id=88-2525
 */
```

**取得方法**：

- MCP ツール経由で fileKey と node-id を取得
- URL から node-id を抽出する際は `X-Y` 形式を `X:Y` に変換

**適用対象**：

- `/components/ui/` - 必須
- `/components/icons/` - 必須
- `/app/components/pages/` - 不要（ページ固有のため）

### 既存コンポーネントの判定方法

1. **Figma 情報による判定**
   - コンポーネントファイルの先頭コメントから fileKey と node-id を抽出
   - 新規追加する Figma ノードの fileKey + node-id と一致する場合は「既存」と判定

2. **Figma 情報がない場合**
   - 目視でコンポーネント名やプロパティから類似性を判断
   - 再利用可能と判断した場合は、既存コンポーネントに Figma 情報を追加

### 命名規則の柔軟性

- **Figma 上のコンポーネント名と Web 実装の名前は異なっても構いません**
- 例：
  - Figma: `Button/Primary` → Web: `PrimaryButton`
  - Figma: `Icon/Check` → Web: `CheckIcon`
- Web 実装では React や TypeScript の命名慣習を優先してください
- Figma 情報のコメントがあれば、名前が異なっても同一コンポーネントと判定できます

## タイポグラフィトークンの使用

### 基本原則

**`text-[15px]`や`leading-[22.5px]`のようなハードコード値は使わず、`tailwind.config.ts`で定義されたタイポグラフィトークンを使用してください。**

- トークンが存在する場合 → そのまま使用
- トークンが存在しない場合 → `tailwind.config.ts`に追加してから使用
- font-weight は 400 または 600 のみ（`font-medium`は使用不可）

### 使用例

```tsx
// ✅ 正しい
<h2 className="text-title-xlarge">プロジェクト選択</h2>
<p className="text-body-medium">説明テキスト</p>
<label className="text-title-small">氏名</label>
<button className="text-label-button-large">送信</button>

// ❌ 間違い
<h2 className="text-[24px] leading-[36px] font-semibold">プロジェクト選択</h2>
<p className="text-sm">説明テキスト</p>
```

### よくあるパターン

| 使用ケース | トークン |
|----------|-------|
| ページタイトル | `text-title-xlarge` |
| セクション見出し | `text-title-small` |
| 本文・説明 | `text-body-medium` |
| ヘルパーテキスト | `text-body-small` |
| フォームラベル | `text-title-small` |
| エラーメッセージ | `text-body-small` |
| タグ/バッジ | `text-label-tag` |
| ボタン（Large） | `text-label-button-large` |
| ボタン（Medium/Small） | `text-label-button-small` |

Figmaデザインのfont-size/line-height/font-weightを確認し、`tailwind.config.ts`の定義と照合してください。

---

## フォーム実装ガイドライン

> **Note:** フォームライブラリや参照ファイルはプロジェクトに合わせて変更してください。
> ただし、Schema（バリデーション定義）とコンポーネント（UI）は必ず分離してください。
> 分離しておくことで、AI によるコンポーネント自動改修時にバリデーションロジックへの影響を最小限に抑えられます。

### 推奨構成

```
feature/
├── schema.ts      # Zod 等のバリデーション定義
└── page.tsx       # UI コンポーネント（schema を import して使用）
```

---

## ロジックとUIの分離

### 基本原則

**JSX内に複雑なロジックを直接書かず、コンポーネント上部でハンドラー関数として定義することで、ロジックとUIを分離します。**

### 複雑なロジックの定義

以下のいずれかに該当する場合、ハンドラー関数として分離すべきです：

1. **3行以上のインラインコールバック**
2. **ループや条件分岐を含む**
   - `for`, `map`, `find`, `filter`, `if`文など
3. **データ変換・検索を含む**
   - 配列操作、オブジェクト検索などのビジネスロジック

### 推奨パターン: 名前付きハンドラー関数

複雑なロジックはコンポーネント上部で名前付き関数として定義します。

```tsx
// ✅ 良い例: ハンドラーを分離
export function UserSelector({ users }: Props) {
  const handleUserSelect = (userId: string) => {
    const selectedUser = users.find((user) => user.id === userId);

    if (selectedUser) {
      // 選択されたユーザーの詳細を取得
      fetchUserDetails(selectedUser.id);
      // フォームに値をセット
      setFormData(selectedUser);
    }
  };

  return (
    <Select onValueChange={handleUserSelect}>
      {users.map((user) => (
        <SelectItem key={user.id} value={user.id}>
          {user.name}
        </SelectItem>
      ))}
    </Select>
  );
}

// ❌ 悪い例: JSX内に複雑なロジックを直接記述
export function UserSelector({ users }: Props) {
  return (
    <Select
      onValueChange={(userId) => {
        const selectedUser = users.find((user) => user.id === userId);

        if (selectedUser) {
          // 選択されたユーザーの詳細を取得
          fetchUserDetails(selectedUser.id);
          // フォームに値をセット
          setFormData(selectedUser);
        }
      }}
    >
      {users.map((user) => (
        <SelectItem key={user.id} value={user.id}>
          {user.name}
        </SelectItem>
      ))}
    </Select>
  );
}
```

### 例外

以下の場合は、インラインコールバックの使用が許容されます：

1. **1行程度のシンプルなコールバック**
   ```tsx
   <Button onClick={() => router.push(pagesPath.$url().pathname)}>
     ホームへ戻る
   </Button>
   ```

2. **propsから受け取ったコールバックをそのまま使用**
   ```tsx
   <Button onClick={onSubmit}>送信</Button>
   <Button onClick={onBack}>戻る</Button>
   ```

---

## 前提条件

### Figma データ取得の前提条件

- **MCP ツール**: Figma Desktop アプリが起動している必要がある
- **Figma ファイルキー**:
  - 画面デザイン: `YOUR_DESIGN_FILE_KEY`
  - 共通コンポーネント、トンマナ: `YOUR_LIBRARY_FILE_KEY`

**エラー発生時**: 上記のファイルキーを使用して再試行してください
