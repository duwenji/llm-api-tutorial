# Mermaid図への変換 設計書

- 日付: 2026-07-19
- 対象: `llm-api-tutorial` 教材一式

## 背景・課題

現在の教材には、矢印・罫線を使ったASCIIアート図が複数の教材に散在している。
GitHubはMarkdown内の ```mermaid``` ブロックをネイティブに描画できるため、
ASCII図をMermaid形式に置き換えることで、図としての可読性・保守性を高める。

## ゴール

既存のASCII図をすべてMermaid形式に置き換える。新規の図は追加しない
（まず既存箇所の視覚化品質を上げることに集中する）。

## スコープ

対象は以下8箇所。ビジュアルコンパニオンで①③⑦の変換イメージを実際に
レンダリングして確認済み。

| # | ファイル | 図の内容 | Mermaid種別 |
|---|----------|----------|--------------|
| 1 | `docs/00-COVER.md` | STEP 1〜5 学習の流れ | `flowchart TD` |
| 2 | `docs/01-history-and-basics/01-history-of-llm-api.md` | 機能拡張の方向性 | `flowchart TD`（確認済み） |
| 3 | `docs/01-history-and-basics/02-api-fundamentals.md` | クライアント⇄サーバー基本フロー | `sequenceDiagram`（確認済み） |
| 4 | `docs/02-core-mechanics/03-tool-use.md` | ツール呼び出し5ステップ往復 | `sequenceDiagram` |
| 5 | `docs/03-advanced-features/03-prompt-caching.md` | レンダー順序（tools→system→messages） | `flowchart LR`（最小構成） |
| 6 | `docs/04-standardization/02-mcp-protocol.md` | MCPなし/ありの比較 | `flowchart TD`（2つのsubgraphで対比） |
| 7 | `docs/05-real-world-examples/01-simple-chat-client.md` | ConversationManagerのクラス設計 | `classDiagram`（確認済み） |
| 8 | `docs/05-real-world-examples/03-cost-and-provider-comparison.md` | コスト削減手法の組み合わせ | `flowchart TD` |

### 適用範囲外（変更しないもの）

- `docs/superpowers/specs/`・`docs/superpowers/plans/`（過去の設計書・計画書。
  歴史的記録として現状維持）
- 各カテゴリ`00-README.md`の「01 → 03 の順に進める」のような1行の矢印テキスト
  （図ではないため対象外）

## ビルド基盤

css-tutorialが持つebook-build（Puppeteerによる画像化）用の仕組みは用意しない。
GitHubのネイティブMermaidレンダリングのみに依存する。

## スタイルガイドへの追記

`00_STYLE_GUIDE.md`の「4. コードブロックルール」に以下を追加する。

- 図解が必要な場合はMermaid形式（```mermaid` ブロック）を優先する。
  ASCIIアートの罫線・矢印による自作図は使わない
- Mermaidのテーマ・色は指定せず、GitHub既定のレンダリングに任せる

## 変換方針の詳細

### ①③⑦（ビジュアルコンパニオンで確認済み）

- ①機能拡張の方向性: 6ノードの直線フローチャート（`flowchart TD`、`-->`で連結）
- ③API基本フロー: `sequenceDiagram`で`participant`を2つ定義し、
  リクエスト/レスポンスの矢印と`Note over`でモデル処理を表現
- ⑦ConversationManagerのクラス設計: `classDiagram`でフィールドと
  `send()`メソッドを定義し、内部手順は`note for`で補足

### ②④⑤⑥⑧（新規変換方針）

- ②STEP 1〜5学習の流れ: 5ノードの直線フローチャート（①と同じパターン）
- ④ツール呼び出し5ステップ: `sequenceDiagram`で`クライアント`と`モデル`の
  2participant、5つの矢印（tool_use要求→実行→tool_result返送→最終回答）
- ⑤レンダー順序: 3ノードの`flowchart LR`（`tools --> system --> messages`）
- ⑥MCPなし/あり比較: `flowchart TD`内に`subgraph MCPなし`と
  `subgraph MCPあり`を並べ、組み合わせ爆発 vs 一元化を対比
- ⑧コスト削減の組み合わせ: 4ノードの直線フローチャート（①と同じパターン）

## 正確性・整合性の管理

- Mermaid構文はGitHubが対応する標準構文（`flowchart`/`sequenceDiagram`/
  `classDiagram`）のみを使う。プラグイン依存の拡張構文は使わない
- 図の内容（ノードの文言・矢印の向き）は元のASCII図の情報を過不足なく
  引き継ぐ。新しい情報を図に追加しない
- 日本語ノードラベルの改行には`<br/>`のみを使う（ビジュアルコンパニオンで
  例①③⑦において実際にレンダリングを確認済みの構文）

## 検証チェックリスト（実装後にセルフレビュー）

- [ ] 対象8箇所すべてが```mermaid```ブロックに置き換わっている
- [ ] 置き換え後、旧ASCII図のテキスト情報（ノード名・矢印の向き）が
      すべて図に反映されている
- [ ] `00_STYLE_GUIDE.md`にMermaid優先のルールが追記されている
- [ ] 対象外ファイル（superpowers配下、00-README.mdの矢印テキスト）に
      変更が入っていない
- [ ] 各Mermaidブロックの直前・直後の説明文と図の内容が矛盾しない
