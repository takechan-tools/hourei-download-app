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

e-Gov APIが返すXMLの階層構造:
```
Article（条）
├── ArticleTitle（条番号: 第1条）
├── ArticleCaption（条見出し）
└── Paragraph（項）
    ├── ParagraphNum（項番号: ２、３...）
    ├── Sentence（本文）
    └── Item（号）
        ├── ItemTitle（号番号: 一、二...）
        ├── Sentence（本文）
        └── Subitem1（サブ項目）
```

**注意**: e-Gov APIのXMLでは、第1項の`ParagraphNum`は空（テキストなし）。
第2項以降は「２」「３」等の数字が入る。

### ファイル名サニタイズ (`sanitizeFilename`)

`/ \ : * ? " < > |` を `_` に置換。
**現状、ファイル名の長さ制限は未実装。**

## e-Gov法令API仕様メモ

- レスポンス形式: XML
- CORS: ブラウザから直接アクセス可能（CORS対応済み）
- 時点指定パラメータ: `asof=YYYYMMDD`（特定日時点の法令を取得可能）
- レート制限: 明確な公式情報なし（大量アクセスは避ける）

## 検証済み項目リスト

（実装時に試した方法と結果をここに記録する）

## 注意事項

- `index-option1.html` と `index-option2.html` は過去のUIバリエーション。変更は `index.html` に対して行う
- CORSエラーが出る環境ではブラウザ拡張機能が必要
