# シンプルなチャットクライアント

## この教材で身につくこと

- 会話履歴を管理するクラスの設計方法
- ストリーミング表示とエラーハンドリングの統合
- システムプロンプトによるキャラクター設定

## 概要

01〜02章で学んだメッセージ管理・ストリーミング・認証を組み合わせ、
最小構成の対話型チャットクライアントを構築します。

## 位置づけ

このチュートリアルの集大成の入口です。ここまでの全カテゴリの知識を使います。

## 仕組み解説

### 設計の全体像

```
ConversationManager
  ├── messages: 会話履歴（役割付き配列）
  ├── system: システムプロンプト（固定）
  └── send(user_text) -> str
        1. messagesにuserメッセージを追加
        2. APIを呼び出す（ストリーミング推奨）
        3. assistantの応答をmessagesに追加
        4. 応答テキストを返す
```

### 良い設計・悪い設計の対比

```python
# ✅ 良い例: 履歴管理をクラスにカプセル化し、エラー処理も統合
class ConversationManager:
    def __init__(self, client, model, system=None):
        self.client, self.model, self.system = client, model, system
        self.messages = []

    def send(self, user_text: str) -> str:
        self.messages.append({"role": "user", "content": user_text})
        try:
            response = self.client.messages.create(
                model=self.model, max_tokens=2000,
                system=self.system, messages=self.messages,
            )
        except Exception as e:
            self.messages.pop()  # 失敗時は履歴を戻す
            raise
        reply = response.content[0].text
        self.messages.append({"role": "assistant", "content": reply})
        return reply
```

```python
# ❌ 悪い例: グローバル変数で管理し、失敗時に履歴が壊れる
messages = []
def send(text):
    messages.append({"role": "user", "content": text})
    response = client.messages.create(model="claude-opus-4-8", max_tokens=2000, messages=messages)
    messages.append({"role": "assistant", "content": response.content[0].text})
    return response.content[0].text
```

## 実装例

```python
import anthropic

class ConversationManager:
    def __init__(self, client: anthropic.Anthropic, model: str, system: str = ""):
        self.client = client
        self.model = model
        self.system = system
        self.messages: list[dict] = []

    def send(self, user_text: str) -> str:
        self.messages.append({"role": "user", "content": user_text})
        with self.client.messages.stream(
            model=self.model, max_tokens=2000,
            system=self.system, messages=self.messages,
        ) as stream:
            for chunk in stream.text_stream:
                print(chunk, end="", flush=True)
            final = stream.get_final_message()
        reply = final.content[0].text
        self.messages.append({"role": "assistant", "content": reply})
        print()
        return reply


conversation = ConversationManager(
    client=anthropic.Anthropic(),
    model="claude-opus-4-8",
    system="あなたは簡潔で親しみやすいアシスタントです。",
)
conversation.send("私の名前はAliceです")
conversation.send("私の名前は覚えていますか？")
```

## 演習課題

1. 会話履歴が一定ターン数（例: 20）を超えたら古い履歴を削除する
   仕組みを追加せよ
2. APIエラー発生時に3回までリトライする処理を追加せよ

## 理解度チェック

- [ ] 会話履歴管理をクラスにカプセル化する意義を説明できる
- [ ] ストリーミング表示と最終応答取得を両立できる
- [ ] エラー時に履歴が壊れないようにする設計ができる

---
前へ: [../04-standardization/03-error-handling-and-versioning.md](../04-standardization/03-error-handling-and-versioning.md) | 次へ: [02-tool-calling-agent.md](02-tool-calling-agent.md)
