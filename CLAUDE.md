# 法令ダウンロードアプリ - 技術ドキュメント

## プロジェクト概要

e-Gov法令APIから日本の法令を検索し、Markdown形式でダウンロードするWebアプリケーション。

## 技術スタック

- **フロントエンド**: HTML5 + CSS3 + Vanilla JavaScript（フレームワークなし）
- **外部ライブラリ**: JSZip（CDN経由、ZIP生成用）
- **API**: e-Gov法令API v1
  - 法令一覧: `https://elaws.e-gov.go.jp/api/1/lawlists/1`
  - 法令本文: `https://elaws.e-gov.go.jp/api/1/lawdata/{lawId}`
- **バックエンド**: なし（完全クライアントサイド処理）

## ファイル構成

| ファイル | 説明 |
|---------|------|
| `index.html` | メインアプリ（現行の正式版） |
| `index-option1.html` | 旧UIバリエーション（参考用） |
| `index-option2.html` | 旧UIバリエーション（参考用） |

## 主要機能

1. **法令検索**: 部分一致によるリアルタイム検索
2. **複数選択**: チェックボックスによる法令選択 + 全選択/解除
3. **選択中表示**: 選択した法令を別枠で常時表示（法令種類バッジ付き）
4. **ダウンロード**:
   - 1件: 直接MDファイルダウンロード
   - 複数件: ZIPファイルでまとめてダウンロード
5. **法令種類分類**: 法律/政令/省令/その他の色分けバッジ

## コードの重要ポイント

### XMLからMarkdown変換 (`parseToMarkdown`)

e-Gov APIが返すXMLの完全な階層構造（公式スキーマ: XMLSchemaForJapaneseLaw_v3.xsd）:

```
Law
└── LawBody
    ├── LawTitle（法令名）
    ├── EnactStatement（制定文）
    ├── TOC（目次）
    ├── Preamble（前文）
    ├── MainProvision（本則）
    │   ├── Part（編）> Chapter（章）> Section（節）> Subsection（款）> Division（目）> Article
    │   ├── Article（条）※ Chapter等なしで直接配置される場合もある
    │   │   ├── ArticleCaption（条見出し: （定義）等）
    │   │   ├── ArticleTitle（条番号: 第一条）
    │   │   └── Paragraph（項）
    │   │       ├── ParagraphNum（項番号: 空/２/３...）
    │   │       ├── ParagraphSentence > Sentence（本文）
    │   │       └── Item（号）
    │   │           ├── ItemTitle（号番号: 一/二...）
    │   │           ├── ItemSentence > Sentence（本文）
    │   │           └── Subitem1 > Subitem2 > ... > Subitem10
    │   └── Paragraph（項）※ Articleなしで直接配置される短い省令等
    ├── SupplProvision（附則）※複数存在しうる
    │   ├── 属性: AmendLawNum（改正法令番号。空=""なら原始附則）
    │   ├── 属性: Extract（"true"なら抄録）
    │   ├── SupplProvisionLabel（「附　則」等）
    │   ├── Article / Paragraph（附則本文）
    │   └── Chapter等（章立ての附則）
    ├── AppdxTable（別表）
    ├── AppdxNote（別記）
    ├── AppdxStyle（別記様式）
    ├── AppdxFig（別図）
    └── AppdxFormat（別記書式）
```

**重要な注意点**:
- `ParagraphNum`: 第1項は空要素。第2項以降に「２」「３」等が入る
- `ParagraphSentence` > `Sentence`: Sentenceが複数ある場合あり（ただし書き等）
- `Sentence`属性: `Function="main"/"proviso"`, `WritingMode="vertical"/"horizontal"`
- `SupplProvision`: `AmendLawNum`属性がある場合は改正法令の附則（原始附則のみ出力すべき）
- 短い省令は`Article`なしで`MainProvision`直下に`Paragraph`が配置される
- 法令によって`Chapter`/`Section`等の階層構造が異なる
- **`ItemSentence`内にColumnがある場合**: `ItemSentence > Column（Sentenceなし）` 形式でテキスト直書きされる法令がある（例: 民法第98条2項 令和5年改正版）。`querySelectorAll("Sentence")` がヒットしない場合は `querySelectorAll("Column")` の `textContent` を全角スペース区切りで結合する

**公式ドキュメント**:
- XMLスキーマ: https://laws.e-gov.go.jp/file/XMLSchemaForJapaneseLaw_v3.xsd
- 構造説明: https://laws.e-gov.go.jp/docs/law-data-basic/8ebd8bc-law-structure-and-xml/
- API仕様: https://laws.e-gov.go.jp/docs/law-data-basic/8529371-law-api-v1/

### ファイル名サニタイズ (`sanitizeFilename`)

`/ \ : * ? " < > |` を `_` に置換。
**現状、ファイル名の長さ制限は未実装。**

## e-Gov法令API仕様メモ

- レスポンス形式: XML
- CORS: ブラウザから直接アクセス可能（CORS対応済み）
- 時点指定パラメータ: `asof=YYYYMMDD`（特定日時点の法令を取得可能）
- レート制限: 明確な公式情報なし（大量アクセスは避ける）

## UI/UX設計メモ

### `law-list-container` は常時表示

- `law-list-container` は検索欄の状態に関わらず常に表示する（`display: none` にしない）
- 検索結果がないときは法令一覧エリアに案内メッセージ「キーワードを入力して法令を検索してください」を表示
- 「全選択」エリア（`select-all-area`）のみ、検索結果があるときだけ `display: flex` で表示
- これによりダウンロードボタン・選択件数・オプションが検索状態に関係なく常にアクセス可能

### 「選択中の法令」セクションの配置

- `law-list-container`の**外**に配置すること（中に入れると連動して消える可能性がある）

## 検証済み項目リスト

### 選択中の法令セクションの表示維持
- **試したこと**: `selected-laws-section`を`law-list-container`内に配置
- **結果**: 検索欄を空にすると`law-list-container`ごと非表示になり、選択中の法令も消える
- **解決策**: セクションを`law-list-container`の外に配置 + 検索欄クリア時に`updateSelectedLawsDisplay()`を呼び出す
- **日時**: 2026-02-20

## 注意事項

- `index-option1.html` と `index-option2.html` は過去のUIバリエーション。変更は `index.html` に対して行う
- CORSエラーが出る環境ではブラウザ拡張機能が必要
