# 拡張思考

## この教材で身につくこと

- 拡張思考（Extended Thinking）が解決する課題
- thinkingブロックの受け取り方と表示方法
- effort（思考の深さ）パラメータによる制御

## 概要

拡張思考は、モデルが最終回答を出す前に内部で行う推論過程を
`thinking`ブロックとして扱う仕組みです。複雑な推論タスクの精度向上に使われます。

## 位置づけ

02章のメッセージ形式を拡張し、応答内容に「思考」という
新しい種類のブロックが加わることを学びます。

## 仕組み解説

### 拡張思考が向くタスク

| タスクの種類 | 拡張思考の効果 |
|--------------|----------------|
| 数学的な多段階推論 | 途中式を経ることで精度が上がる |
| 複雑な計画立案 | 選択肢の比較検討を内部で行える |
| 単純な分類・要約 | 効果は薄く、コスト増になりやすい |

### thinkingの表示制御

`display`パラメータで、内部推論を要約表示するか非表示にするかを
制御できます。

| 値 | 挙動 |
|----|------|
| `"summarized"` | 要約された思考内容を表示 |
| `"omitted"`（省略時のデフォルト） | 思考は行うが本文は空文字 |

```json
// ✅ 良い例: UIに思考過程を見せたい場合
{"thinking": {"type": "adaptive", "display": "summarized"}}
```

```json
// ❌ 悪い例: 表示したいのにdisplayを省略し、空の思考ブロックに困惑する
{"thinking": {"type": "adaptive"}}
```

### effort（思考の深さ）

| 値 | 用途 |
|----|------|
| `low` | 短く軽いタスク、レイテンシ重視 |
| `medium` | 多くのユースケースでのバランス |
| `high` / `xhigh` | コーディング・複雑なエージェントタスク |
| `max` | 最大の精度が必要でコストを気にしない場合 |

## 実装例

```python
response = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=8000,
    thinking={"type": "adaptive", "display": "summarized"},
    messages=[{"role": "user", "content": "27×453を暗算せず段階的に解いて"}],
)

for block in response.content:
    if block.type == "thinking":
        print("[思考]", block.thinking)
    elif block.type == "text":
        print("[回答]", block.text)
```

```json
// 会話を継続する際は、thinkingブロックを改変せずそのまま返す
{"role": "assistant", "content": [
  {"type": "thinking", "thinking": "...", "signature": "..."},
  {"type": "text", "text": "..."}
]}
```

## 演習課題

1. 拡張思考を使うべきタスクと、使わないほうがよいタスクを1つずつ挙げよ
2. `display: "omitted"`のとき、UI側で気をつけるべき点を説明せよ

## 理解度チェック

- [ ] 拡張思考が有効なタスクの傾向を説明できる
- [ ] `display`パラメータの2つの値の違いを理解している
- [ ] effortレベルをタスクの性質に応じて選べる

---
前へ: [01-multimodal-input.md](01-multimodal-input.md) | 次へ: [03-prompt-caching.md](03-prompt-caching.md)
