# メッセージとロール

## この教材で身につくこと

- system/user/assistantの3つのロールの役割
- システムプロンプトとユーザーメッセージの使い分け
- 複数ターンの会話履歴を正しく組み立てる方法

## 概要

チャット形式のLLM APIでは、会話は「ロール付きメッセージの配列」として
表現されます。ロールを正しく使い分けることで、モデルの振る舞いを
安定して制御できます。

## 位置づけ

01章の「APIの基本仕組み」で学んだステートレス設計を踏まえ、
実際にどうメッセージを組み立てるかを扱います。

## 仕組み解説

### ロールの役割

| ロール | 役割 | 会話履歴に含むか |
|--------|------|-------------------|
| `system` | モデルの振る舞い・制約を規定する指示 | 通常は別フィールド（`system`）として分離 |
| `user` | 人間側の発話・質問 | 含む |
| `assistant` | モデル側の応答（過去の応答） | 含む |
| `tool_result` | ツール実行結果の返送 | `user`メッセージ内の特殊ブロックとして含む |

### システムプロンプトの位置づけ

システムプロンプトは会話全体に効く「前提条件」です。
毎ターン変える必要がない内容（口調、制約、役割設定）を入れます。

```json
// ✅ 良い例: 安定した指示をsystemに、可変情報はuserに
{
  "system": "あなたは丁寧な日本語で回答するカスタマーサポート担当です。",
  "messages": [
    {"role": "user", "content": "注文番号12345の状況を教えてください"}
  ]
}
```

```json
// ❌ 悪い例: 毎回変わる情報をsystemに埋め込む（キャッシュ効率も悪化）
{
  "system": "現在時刻は2026-07-19 14:32です。注文番号12345について回答してください。",
  "messages": [{"role": "user", "content": "状況を教えてください"}]
}
```

### 複数ターンの組み立て

```json
{
  "messages": [
    {"role": "user", "content": "私の名前はAliceです"},
    {"role": "assistant", "content": "こんにちは、Aliceさん。"},
    {"role": "user", "content": "私の名前は何でしたか？"}
  ]
}
```

**ルール**: 最初のメッセージは必ず`user`。連続する同ロールのメッセージは
1ターンとして結合される。

## 実装例

```python
messages = []

def send(user_text: str) -> str:
    messages.append({"role": "user", "content": user_text})
    response = client.messages.create(
        model="claude-opus-4-8",
        max_tokens=1024,
        system="あなたは簡潔に回答するアシスタントです。",
        messages=messages,
    )
    reply = response.content[0].text
    messages.append({"role": "assistant", "content": reply})
    return reply

print(send("私の名前はAliceです"))
print(send("私の名前は何でしたか？"))  # "Alice"を覚えている
```

## 演習課題

1. 「敬語で回答する」「箇条書きは使わない」という制約を
   systemプロンプトとして書け
2. 3ターンの会話（自己紹介→好きな食べ物→質問確認）のJSON配列を作れ

## 理解度チェック

- [ ] system/user/assistantそれぞれの役割を説明できる
- [ ] 可変情報をsystemプロンプトに入れるべきでない理由を説明できる
- [ ] 複数ターンの会話履歴を正しい形式で組み立てられる

---
前へ: [../01-history-and-basics/04-authentication.md](../01-history-and-basics/04-authentication.md) | 次へ: [02-streaming.md](02-streaming.md)
