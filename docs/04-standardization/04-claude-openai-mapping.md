# Claude API ⇔ OpenAI公式API 対応表

## この教材で身につくこと

- Claude APIとOpenAI公式APIで、同じ概念がどう異なる名称・構造で
  表現されるかの全体像
- 用語やパラメータ名がベンダー間で統一されていない理由
- 対応表を使って、片方のAPI向けコードをもう片方に書き換える考え方

## 概要

これまでの教材（02〜03章、05章）では、実装例ごとに個別の対応表を
示してきました。この教材はその総まとめで、全ての対応関係を
1ページに集約します。

## 位置づけ

このチュートリアル全体で登場したClaude/OpenAI比較の集大成です。
04章（標準化）の最後に置き、「標準化されていない部分は結局こう対応する」
という実務的な着地点を示します。

## 仕組み解説

### なぜ用語が統一されていないか

- 両社は01章で見た通り、それぞれ独立にAPIを設計した
- OpenAI形式が事実上の標準（04-01）にはなったが、完全な仕様統一ではない
- 拡張思考・キャッシュなど新しい機能ほど、各社の設計思想の違いが
  そのままパラメータ名の違いとして表れる

### 対応表の読み方

会話の構造・呼び出し形式・応答フィールドなど、概念単位で1行にまとめています。
自分が書いているコードがどちらのAPI向けか分からなくなったら、
まずこの表で該当する概念を探してください。

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

## 実装例

対応表を使ってコードを読み替える手順を、擬似コードで示します。

```python
# Claude向けコードをOpenAI向けに変換する際のチェックリスト
# 1. model="claude-opus-4-8"            -> model="gpt-4o"
# 2. client.messages.create(...)        -> client.chat.completions.create(...)
# 3. system="..."                       -> messages配列の先頭に{"role": "system", ...}
# 4. response.content[0].text           -> response.choices[0].message.content
# 5. tools内の"input_schema"            -> "parameters"
# 6. response.stop_reason == "tool_use" -> response.choices[0].finish_reason == "tool_calls"
```

完全な実装（両方の言語で動くコード）は、この手順の元になった各教材を
参照してください。

- ツール呼び出し: [02-core-mechanics/03-tool-use.md](../02-core-mechanics/03-tool-use.md)
- systemメッセージ: [02-core-mechanics/01-messages-and-roles.md](../02-core-mechanics/01-messages-and-roles.md)
- ストリーミング: [02-core-mechanics/02-streaming.md](../02-core-mechanics/02-streaming.md)
- 構造化出力: [03-advanced-features/04-structured-outputs.md](../03-advanced-features/04-structured-outputs.md)
- プロンプトキャッシュ: [03-advanced-features/03-prompt-caching.md](../03-advanced-features/03-prompt-caching.md)
- 会話クライアント全体: [05-real-world-examples/01-simple-chat-client.md](../05-real-world-examples/01-simple-chat-client.md)

## 演習課題

1. [02-core-mechanics/03-tool-use.md](../02-core-mechanics/03-tool-use.md)の
   OpenAI版ツール呼び出しコードを、対応表だけを見ながらClaude版に
   「逆翻訳」し、実際のClaude版コードと一致するか確認せよ
2. 対応表にまだ載っていない概念（例: エラーハンドリング、認証）について、
   自分で対応表に1行追加してみよ

## 理解度チェック

- [ ] Claude APIとOpenAI公式APIで用語が異なる理由を説明できる
- [ ] 対応表を参照しながら、片方のAPI向けコードをもう片方に書き換えられる
- [ ] 本チュートリアル内のどの教材にどの比較が載っているか把握している

---
前へ: [03-error-handling-and-versioning.md](03-error-handling-and-versioning.md) | 次へ: [../05-real-world-examples/00-README.md](../05-real-world-examples/00-README.md)
