# ツール呼び出し

## この教材で身につくこと

- ツール呼び出し（Function Calling）の往復フロー
- ツール定義（JSON Schema）の書き方
- `tool_use`と`tool_result`の対応関係

## 概要

ツール呼び出しは、モデルが「外部関数を呼びたい」という意図を
構造化データで表現し、実行結果を受け取って会話を継続する仕組みです。
モデル自身はコードを実行せず、実行はクライアント側の責務です。

## 位置づけ

エージェント（05章）を構築する上で最も重要な仕組みです。
02章の「メッセージとロール」の応用にあたります。

## 仕組み解説

### 往復フローの全体像

```
1. クライアント → モデル: ツール定義 + ユーザーの質問を送信
2. モデル → クライアント: tool_use ブロックを返す（呼びたい関数と引数）
3. クライアント: 実際に関数を実行する
4. クライアント → モデル: tool_result として実行結果を返送
5. モデル → クライアント: 実行結果を踏まえた最終回答
```

### ツール定義の書き方

```json
{
  "name": "get_weather",
  "description": "指定した都市の現在の天気を取得する",
  "input_schema": {
    "type": "object",
    "properties": {
      "location": {"type": "string", "description": "都市名。例: 東京"}
    },
    "required": ["location"]
  }
}
```

**良い定義のポイント**

| ポイント | 理由 |
|----------|------|
| `description`を具体的に書く | モデルは説明文だけで呼び出し判断をする |
| 「いつ呼ぶか」を明記する | 呼び出し過多・過少を防げる |
| `required`を正しく設定する | 不要なパラメータを強制しない |

### tool_use ↔ tool_result の対応

```json
// モデルからの応答（tool_use）
{"type": "tool_use", "id": "toolu_abc123", "name": "get_weather", "input": {"location": "東京"}}
```

```json
// クライアントからの返送（tool_result）
{"type": "tool_result", "tool_use_id": "toolu_abc123", "content": "東京: 晴れ、24℃"}
```

`tool_use_id`が一致していないと、モデル側は結果を正しく紐づけられません。

### OpenAI公式APIとの用語対応

| 概念 | Claude API | OpenAI公式API |
|------|-----------|----------------|
| ツール定義フィールド | `input_schema` | `parameters` |
| 呼び出しブロック | `tool_use`（`content`内） | `tool_calls`（`message`内） |
| 結果の返送 | `tool_result`（userメッセージ内） | `role: "tool"`の独立メッセージ |
| 対応ID | `tool_use_id` | `tool_call_id` |

## 実装例

### Claude API

```python
tools = [{
    "name": "get_weather",
    "description": "指定した都市の現在の天気を取得する",
    "input_schema": {
        "type": "object",
        "properties": {"location": {"type": "string"}},
        "required": ["location"],
    },
}]

messages = [{"role": "user", "content": "東京の天気を教えて"}]
response = client.messages.create(
    model="claude-opus-4-8", max_tokens=1024, tools=tools, messages=messages,
)

if response.stop_reason == "tool_use":
    tool_call = next(b for b in response.content if b.type == "tool_use")
    result = f"{tool_call.input['location']}: 晴れ、24℃"  # 実際のAPI呼び出しに置換

    messages.append({"role": "assistant", "content": response.content})
    messages.append({
        "role": "user",
        "content": [{"type": "tool_result", "tool_use_id": tool_call.id, "content": result}],
    })

    final = client.messages.create(
        model="claude-opus-4-8", max_tokens=1024, tools=tools, messages=messages,
    )
    print(final.content[0].text)
```

### OpenAI公式API

```python
tools = [{
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "指定した都市の現在の天気を取得する",
        "parameters": {
            "type": "object",
            "properties": {"location": {"type": "string"}},
            "required": ["location"],
        },
    },
}]

messages = [{"role": "user", "content": "東京の天気を教えて"}]
response = client.chat.completions.create(
    model="gpt-4o", tools=tools, messages=messages,
)

if response.choices[0].finish_reason == "tool_calls":
    tool_call = response.choices[0].message.tool_calls[0]
    args = json.loads(tool_call.function.arguments)
    result = f"{args['location']}: 晴れ、24℃"  # 実際のAPI呼び出しに置換

    messages.append(response.choices[0].message)
    messages.append({
        "role": "tool", "tool_call_id": tool_call.id, "content": result,
    })

    final = client.chat.completions.create(
        model="gpt-4o", tools=tools, messages=messages,
    )
    print(final.choices[0].message.content)
```

## 演習課題

1. 「注文番号から配送状況を取得する」ツールの`input_schema`を書け
2. `tool_use_id`を正しく対応させないとどうなるか説明せよ
3. Claude APIの`tool_use`とOpenAI公式APIの`tool_calls`の構造上の違いを説明せよ

## 理解度チェック

- [ ] ツール呼び出しの5ステップの往復フローを説明できる
- [ ] `description`が呼び出し判断に重要な理由を説明できる
- [ ] `tool_use`と`tool_result`を正しく対応づけて送信できる
- [ ] Claude APIとOpenAI公式APIのツール定義フィールド名の違いを説明できる

---
前へ: [02-streaming.md](02-streaming.md) | 次へ: [../03-advanced-features/00-README.md](../03-advanced-features/00-README.md)
