# LLM API クイックリファレンス

チュートリアル全体で扱う概念とパラメータの早見表です。

## リクエストの基本構造（Messages API系）

| フィールド | 役割 | 代表値 |
|-----------|------|--------|
| `model` | 使用モデルのID | `claude-opus-4-8`, `gpt-4o` |
| `max_tokens` | 出力トークン数の上限 | 1024, 16000 |
| `messages` | 会話履歴（role/content配列） | `[{role, content}]` |
| `system` | システムプロンプト | 文字列 or ブロック配列 |
| `stream` | ストリーミング有無 | true, false |
| `tools` | ツール定義の配列 | `[{name, description, input_schema}]` |
| `temperature` / `top_p` | サンプリング制御 | 0.0〜1.0 |

## ロール（role）の種類

| ロール | 役割 |
|--------|------|
| `system` | モデルの振る舞いを規定する指示（会話に含まれない場合もある） |
| `user` | 人間側の発話 |
| `assistant` | モデル側の応答 |
| `tool` / `tool_result` | ツール実行結果の返送 |

## stop_reason（応答終了理由）

| 値 | 意味 |
|----|------|
| `end_turn` | 正常終了 |
| `max_tokens` | 出力上限に到達（トークン数増加 or ストリーミングを検討） |
| `tool_use` | ツール呼び出しを要求している |
| `stop_sequence` | 指定した停止文字列に到達 |
| `refusal` | 安全性の観点で拒否 |

## HTTPエラーコード早見表

| コード | 種別 | 主な原因 | リトライ |
|--------|------|----------|----------|
| 400 | invalid_request_error | パラメータ不正 | ❌ |
| 401 | authentication_error | APIキー不正・欠落 | ❌ |
| 403 | permission_error | 権限不足 | ❌ |
| 404 | not_found_error | モデルID・エンドポイント誤り | ❌ |
| 429 | rate_limit_error | レート制限超過 | ✅（`retry-after`に従う） |
| 500 | api_error | サーバー側の一時的な問題 | ✅ |
| 529 | overloaded_error | 過負荷 | ✅ |

## 標準化キーワード早見表

| キーワード | 分類 | 概要 |
|-----------|------|------|
| OpenAI Chat Completions形式 | 事実上の標準 | `messages`配列ベースのAPI形式。多くのOSS推論サーバーが互換実装 |
| MCP (Model Context Protocol) | ツール接続標準 | LLMと外部ツール/データソースを繋ぐ共通プロトコル |
| JSON Schema / Structured Outputs | 出力形式標準 | スキーマに従った厳密なJSON出力を強制する仕組み |
| SSE (Server-Sent Events) | 通信標準 | ストリーミング応答で広く使われるHTTP標準規格 |

## Claude ⇔ OpenAI 対応表

会話の構造・呼び出し形式で名称が異なる概念の対応一覧です。
個別教材の対応表はこの表と矛盾しないようにしてください。

| 概念 | Claude API | OpenAI公式API |
|------|-----------|----------------|
| エンドポイント | `POST /v1/messages` | `POST /v1/chat/completions` |
| 認証ヘッダー | `x-api-key: <key>` | `Authorization: Bearer <key>` |
| systemメッセージ | 独立した`system`フィールド | `messages`配列内の`{"role":"system",...}` |
| 応答テキストの取得 | `response.content[0].text` | `response.choices[0].message.content` |
| ツール定義フィールド | `input_schema` | `parameters` |
| ツール呼び出しブロック | `tool_use`（`content`内） | `tool_calls`（`message`内） |
| ツール結果の返送 | `tool_result`（userメッセージ内） | `role: "tool"`の独立メッセージ |
| 応答終了理由 | `stop_reason`（`tool_use`, `end_turn`等） | `finish_reason`（`tool_calls`, `stop`等） |
| ストリーミング終端 | `message_stop`イベント | `data: [DONE]` |
| 画像入力ブロック | `{"type":"image","source":{...}}` | `{"type":"image_url","image_url":{"url":"..."}}` |
| 構造化出力 | `output_config.format`（`json_schema`） | `response_format`（`json_schema`, `strict`） |
| プロンプトキャッシュ | `cache_control`を明示指定（任意） | 一定長以上で自動適用（指定不要） |
| 内部推論の制御 | `thinking`（`type: "adaptive"`）+ `effort` | 推論系モデルの`reasoning_effort` |
| トークン数の事前確認 | `client.messages.count_tokens()` | `tiktoken`ライブラリ |
| バッチリクエストの渡し方 | リクエストを直接JSONで渡す | JSONLファイルをFiles APIにアップロード |

## コスト計算の基本式

```
コスト = (入力トークン数 × 入力単価 + 出力トークン数 × 出力単価) / 1,000,000
```

キャッシュ読み込みは通常入力の約0.1倍、キャッシュ書き込みは約1.25〜2倍の単価が目安。
