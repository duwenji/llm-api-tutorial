# ツール呼び出しエージェント

## この教材で身につくこと

- ツール呼び出しをループ化した自律エージェントの構築方法
- `stop_reason`に応じた分岐処理の設計
- 無限ループを防ぐ安全装置の入れ方

## 概要

02章で学んだ1回限りのツール呼び出しを、
「モデルがツールを呼ばなくなるまで繰り返す」ループに拡張します。
これがエージェントの基本形です。

## 位置づけ

02章のツール呼び出し、04章の標準化知識（MCP）を踏まえた、
実践的なエージェント実装の教材です。

## 仕組み解説

### エージェントループの全体像

```
while True:
    response = モデルを呼び出す(messages, tools)
    if response.stop_reason == "end_turn":
        break  # 最終回答が出た
    ツール呼び出しを実行し、結果をmessagesに追加
```

### 安全装置の必要性

ツール呼び出しは理論上無限に継続しうるため、
上限回数を必ず設けます。

```python
# ✅ 良い例: 最大反復回数を設けて無限ループを防ぐ
MAX_ITERATIONS = 10
for _ in range(MAX_ITERATIONS):
    response = call_model(messages, tools)
    if response.stop_reason != "tool_use":
        break
    messages = handle_tool_calls(response, messages)
else:
    raise RuntimeError("反復上限に到達しました")
```

```python
# ❌ 悪い例: 上限なしのwhile Trueで、バグ時に無限課金の恐れ
while True:
    response = call_model(messages, tools)
    if response.stop_reason != "tool_use":
        break
    messages = handle_tool_calls(response, messages)
```

## 実装例

### Claude API

```python
def execute_tool(name: str, tool_input: dict) -> str:
    if name == "get_order_status":
        return f"注文{tool_input['order_id']}: 発送済み"
    return f"不明なツール: {name}"


tools = [{
    "name": "get_order_status",
    "description": "注文IDから配送状況を取得する",
    "input_schema": {
        "type": "object",
        "properties": {"order_id": {"type": "string"}},
        "required": ["order_id"],
    },
}]

messages = [{"role": "user", "content": "注文12345の状況を教えて"}]

for _ in range(10):
    response = client.messages.create(
        model="claude-opus-4-8", max_tokens=1024, tools=tools, messages=messages,
    )
    if response.stop_reason != "tool_use":
        print(response.content[0].text)
        break

    messages.append({"role": "assistant", "content": response.content})
    tool_results = []
    for block in response.content:
        if block.type == "tool_use":
            result = execute_tool(block.name, block.input)
            tool_results.append({
                "type": "tool_result", "tool_use_id": block.id, "content": result,
            })
    messages.append({"role": "user", "content": tool_results})
```

### OpenAI公式API

```python
import json


def execute_tool(name: str, tool_input: dict) -> str:
    if name == "get_order_status":
        return f"注文{tool_input['order_id']}: 発送済み"
    return f"不明なツール: {name}"


tools = [{
    "type": "function",
    "function": {
        "name": "get_order_status",
        "description": "注文IDから配送状況を取得する",
        "parameters": {
            "type": "object",
            "properties": {"order_id": {"type": "string"}},
            "required": ["order_id"],
        },
    },
}]

messages = [{"role": "user", "content": "注文12345の状況を教えて"}]

for _ in range(10):
    response = client.chat.completions.create(
        model="gpt-4o", tools=tools, messages=messages,
    )
    choice = response.choices[0]
    if choice.finish_reason != "tool_calls":
        print(choice.message.content)
        break

    messages.append(choice.message)
    for tool_call in choice.message.tool_calls:
        args = json.loads(tool_call.function.arguments)
        result = execute_tool(tool_call.function.name, args)
        messages.append({
            "role": "tool", "tool_call_id": tool_call.id, "content": result,
        })
```

> 対応表: Claudeは`stop_reason == "tool_use"`、OpenAIは
> `finish_reason == "tool_calls"`で分岐する。ツール結果の返送も
> Claudeは1つのuserメッセージにまとめるが、OpenAIはツールごとに
> 独立した`role: "tool"`メッセージを追加する。

## 演習課題

1. 複数のツール呼び出しが1回のレスポンスに含まれる場合、
   すべての結果を1つのuserメッセージにまとめるべき理由を説明せよ
2. `MAX_ITERATIONS`到達時にユーザーへ表示すべきメッセージを設計せよ
3. Claude APIとOpenAI公式APIで、複数ツール呼び出しの結果返送方法の違いを説明せよ

## 理解度チェック

- [ ] エージェントループの基本構造を実装できる
- [ ] `stop_reason`に応じた分岐処理を書ける
- [ ] 無限ループを防ぐ安全装置の必要性を説明できる
- [ ] Claude APIとOpenAI公式APIのエージェントループ終了条件の違いを説明できる

---
前へ: [01-simple-chat-client.md](01-simple-chat-client.md) | 次へ: [03-cost-and-provider-comparison.md](03-cost-and-provider-comparison.md)
