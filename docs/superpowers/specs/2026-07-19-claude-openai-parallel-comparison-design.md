# Claude/OpenAI 並列比較 改善設計書

- 日付: 2026-07-19
- 対象: `llm-api-tutorial` 教材一式

## 背景・課題

現状の教材は、実装例のほぼ全てが Claude API（Anthropic）を主軸に書かれており、
OpenAI公式APIへの言及は歴史的経緯（GPT-3、ChatGPT）と04章の
「OpenAI互換API」教材（実際は第三者製のOpenAI互換サーバーを扱う内容）に
限定されていた。

「OpenAIとAnthropicがそれぞれ独自にAPIを公開している」という前提が
教材全体を通じて明確に伝わらず、読者がClaude API固有の仕様を
業界標準と誤解する恐れがある。

## ゴール

Claude API（Anthropic）と OpenAI公式API を2大主軸として、
実装例を持つ全教材で並列比較できる構成に改訂する。

## スコープ

- 対象: `llm-api-tutorial` 配下、実装例を持つ19個の個別教材ファイル
  ＋トップレベル4ファイル（README/00-COVER/MASTER-INDEX/QUICK-REFERENCE）
- 軸: Claude API と OpenAI公式API の2社。Gemini等その他は
  既存の言及箇所（QUICK-REFERENCEの表、04章）に留め、新規に全編展開はしない
- 非対象: 実行環境構築、APIキー取得手順などの運用面

## 用語の区別

- **OpenAI公式API**: `api.openai.com` が提供する公式エンドポイント
- **OpenAI互換（サードパーティ）API**: vLLM等、OpenAI形式のリクエストを
  受け付ける第三者サーバー

04章の教材タイトルを「OpenAI互換（サードパーティ）API」に変更し、
冒頭でOpenAI公式APIとは別概念であることを明示する。
全編の用語はこの2つを明確に呼び分け、混同表記をしない。

## コンテンツパターン

### 実装例セクションのテンプレート

```
## 実装例

### Claude API
（既存のPython/curlコード。model="claude-opus-4-8"）

### OpenAI公式API
（対応するPython/curlコード。model="gpt-4o"）

> 対応表: <この教材固有の用語対応を1〜2行で>
```

### 仕組み解説への対応表追加

概念名がベンダー間で異なる教材は、「仕組み解説」セクション内に
小さな対応表を追加する（既存の04-01, 04-03と同じ形式）。

### 演習課題・理解度チェックの拡張

対応表が意味を持つ教材には、演習課題に1問、
理解度チェックに1項目を追加する。

### 表記ルール

- モデルIDは `claude-opus-4-8` / `gpt-4o` を既定の例として統一使用
- モデルIDの鮮度注記（「執筆時点の例。最新は各社ドキュメント参照」）は
  `docs/00-COVER.md` 冒頭1箇所に集約し、各教材で重複させない
- 04章の教材名は「OpenAI互換（サードパーティ）API」に変更

## ファイル別変更内容

### 01. History and Basics

| ファイル | 追加・変更内容 |
|----------|----------------|
| 01-history-of-llm-api.md | 「主要プロバイダの独立公開時期」表を追加 |
| 02-api-fundamentals.md | OpenAI公式APIのcurl/Python例を追加 |
| 03-tokens-and-context.md | OpenAI向けは`tiktoken`が正しい選択肢である対比を明示 |
| 04-authentication.md | `x-api-key` と `Authorization: Bearer` の対比表・コード追加 |

### 02. Core Mechanics

| ファイル | 追加・変更内容 |
|----------|----------------|
| 01-messages-and-roles.md | OpenAIは`system`もmessages配列内の1要素という構造差を対応表化＋コード追加 |
| 02-streaming.md | SSEイベント形式の違い（named event vs `data: [DONE]`終端）を対応表化＋コード追加 |
| 03-tool-use.md | `tool_use`/`input_schema` vs `tool_calls`/`parameters` の対応表＋コード追加 |

### 03. Advanced Features

| ファイル | 追加・変更内容 |
|----------|----------------|
| 01-multimodal-input.md | `image`/`source` vs `image_url` のコンテンツブロック差を対応表化＋コード追加 |
| 02-extended-thinking.md | OpenAI推論モデルの`reasoning_effort`との概念対応を追加（仕様変動が大きい点は注記で保守的に記述） |
| 03-prompt-caching.md | OpenAIの自動プロンプトキャッシュ（`cache_control`不要・自動適用）との仕組みの違いを対比 |
| 04-structured-outputs.md | `output_config.format` vs `response_format` の対応表＋コード追加 |

### 04. Standardization

| ファイル | 追加・変更内容 |
|----------|----------------|
| 00-README.md | 用語（公式API/互換API）の定義を学習目標に追加 |
| 01-openai-compatible-api.md | タイトルを「OpenAI互換（サードパーティ）API」に変更し、冒頭でOpenAI公式APIとの違いを明示 |
| 02-mcp-protocol.md | MCPがAnthropic発でありながら業界横断で採用されている点（OpenAI含む）を追記 |
| 03-error-handling-and-versioning.md | OpenAI SDKの型付き例外コード例を追加 |

### 05. Real World Examples

| ファイル | 追加・変更内容 |
|----------|----------------|
| 01-simple-chat-client.md | `ConversationManager`のOpenAI版実装を追加 |
| 02-tool-calling-agent.md | エージェントループのOpenAI版実装を追加（`finish_reason == "tool_calls"`分岐） |
| 03-cost-and-provider-comparison.md | 料金体系・トークン計算をClaude/OpenAI両方で比較する内容に拡張 |

### トップレベル

| ファイル | 追加・変更内容 |
|----------|----------------|
| README.md / docs/00-COVER.md | 「Claude APIを主軸に」→「Claude APIとOpenAI公式APIを軸に比較しながら学ぶ」に文言修正。モデルIDの鮮度注記をCOVER冒頭に集約 |
| QUICK-REFERENCE.md | 「Claude ⇔ OpenAI 対応表」セクションを新設し、役割構造・ツール呼び出し・stop reason・ストリーミング・認証・キャッシュ・構造化出力・マルチモーダルの対応を1箇所に集約 |
| MASTER-INDEX.md | 変更なし |

## 正確性管理

- OpenAI側のコードは、長期間安定している **Chat Completions API**
  （`messages`/`tools`/`tool_calls`/`response_format`/`image_url`等）の
  仕様に統一する。新しいAPI体系（Responses API等）が存在する場合は
  脚注で軽く触れるに留め、本文の主軸コードには使わない
- 具体的な料金数値を新規に断定的に書かない。OpenAI側は「相対的な位置づけ」
  「仕組みの違い」で比較する（例: キャッシュが自動か手動か）
- モデルIDの鮮度注記は00-COVER.md冒頭1箇所に集約し重複させない

## 検証チェックリスト（実装後にセルフレビュー）

- [x] 19教材すべてに Claude/OpenAI 両方のコードブロックがある
- [x] 「OpenAI公式API」「OpenAI互換（サードパーティ）API」の呼称が全編で統一されている
- [x] OpenAI側コードのパラメータ名が実在のChat Completions API仕様と矛盾しない
- [x] 全コードブロックに言語タグが付いている
- [x] 追記により前後ナビゲーションリンクが壊れていない
- [x] QUICK-REFERENCEの対応表と各教材内の対応表に矛盾がない
