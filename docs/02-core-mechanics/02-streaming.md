# ストリーミング応答

## この教材で身につくこと

- ストリーミング（SSE）による逐次応答受信の仕組み
- ストリーミングが必要になる場面
- イベントの種類と最終応答の取得方法

## 概要

ストリーミングは、モデルが生成したトークンを逐次配信する仕組みです。
チャットUIの「タイプライター効果」や、長文出力時のタイムアウト回避に使われます。

## 位置づけ

01章で学んだ`max_tokens`の制約と組み合わせて、
長文生成を安全に扱うための実務技術です。

## 仕組み解説

### ストリーミングが必要な場面

| 場面 | 理由 |
|------|------|
| チャットUI | ユーザーに逐次表示し体感速度を上げる |
| `max_tokens`が大きい（1万トークン超） | 非ストリーミングだとHTTPタイムアウトの恐れ |
| 進捗を可視化したいエージェント | ツール呼び出しの過程を都度表示できる |

### Server-Sent Events (SSE) の構造

```
event: message_start
data: {"type":"message_start","message":{...}}

event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"text_delta","text":"こん"}}

event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"text_delta","text":"にちは"}}

event: message_stop
data: {"type":"message_stop"}
```

### 主なイベント種別

| イベント | 発生タイミング |
|----------|----------------|
| `message_start` | メッセージ開始時に1回 |
| `content_block_delta` | トークン生成ごとに複数回 |
| `message_delta` | `stop_reason`や使用量の更新 |
| `message_stop` | 応答完了時に1回 |

## 実装例

```python
with client.messages.stream(
    model="claude-opus-4-8",
    max_tokens=8000,
    messages=[{"role": "user", "content": "俳句を書いてください"}],
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)

    final_message = stream.get_final_message()
    print(f"\n\n使用トークン数: {final_message.usage.output_tokens}")
```

```javascript
const stream = client.messages.stream({
  model: "claude-opus-4-8",
  max_tokens: 8000,
  messages: [{ role: "user", content: "俳句を書いてください" }],
});

for await (const event of stream) {
  if (event.type === "content_block_delta" && event.delta.type === "text_delta") {
    process.stdout.write(event.delta.text);
  }
}
```

## 演習課題

1. 非ストリーミングと比べてストリーミングが優れている点を2つ挙げよ
2. `content_block_delta`と`message_stop`イベントの違いを説明せよ

## 理解度チェック

- [ ] ストリーミングが必要になる典型的な場面を説明できる
- [ ] SSEの主要イベント種別と発生順序を理解している
- [ ] ストリーミング中に最終応答をまとめて取得する方法を知っている

---
前へ: [01-messages-and-roles.md](01-messages-and-roles.md) | 次へ: [03-tool-use.md](03-tool-use.md)
