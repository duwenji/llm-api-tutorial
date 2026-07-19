# OpenAI互換API

## この教材で身につくこと

- OpenAI Chat Completions形式が事実上の標準になった経緯
- 「OpenAI互換」を謳うサーバー・SDKの実態
- 完全な仕様統一ではない点への注意

## 概要

多くのOSS推論サーバー（vLLM、Ollamaなど）や他社プロバイダが、
OpenAIのChat Completions API形式のリクエスト/レスポンスを
そのまま受け付ける「互換エンドポイント」を提供しています。

## 位置づけ

これまで学んだメッセージ形式（02章）が、
ベンダーを超えてどの程度共通化されているかを理解する教材です。

## 仕組み解説

### 互換性が生まれた理由

| 理由 | 説明 |
|------|------|
| 先行者優位 | OpenAIのAPIが最初に広く普及し、既存のクライアントコードが大量に存在 |
| 移行コスト低減 | 既存コードのbase_urlを変えるだけで別プロバイダに切り替え可能 |
| エコシステム | LangChain等のライブラリがOpenAI形式を前提に設計されている |

### 共通する概念

| 概念 | OpenAI形式 | Anthropic形式 |
|------|-----------|----------------|
| 会話配列 | `messages: [{role, content}]` | `messages: [{role, content}]` |
| ツール定義 | `tools: [{type: "function", function: {...}}]` | `tools: [{name, description, input_schema}]` |
| ストリーミング | SSE | SSE |
| 停止理由 | `finish_reason` | `stop_reason` |

### 「互換」の限界

```json
// ✅ 良い例: base_urlの切り替えだけで動く基本的なチャット呼び出し
{"model": "some-model", "messages": [{"role": "user", "content": "Hi"}]}
```

```json
// ❌ 悪い例: ベンダー固有パラメータをそのまま使い回すと動かない
{"model": "some-model", "thinking": {"type": "adaptive"}}  // 一部プロバイダ固有
```

拡張思考パラメータ、エフォート設定、独自ツール（メモリ・コード実行等）は
互換レイヤーではサポートされないことが多く、**基本チャット機能のみ**が
安全に互換とみなせる範囲です。

## 実装例

```python
# OpenAI公式SDKを使って、互換エンドポイントに向ける例
from openai import OpenAI

client = OpenAI(
    base_url="https://example-compatible-provider.com/v1",
    api_key="provider-specific-key",
)
response = client.chat.completions.create(
    model="some-model",
    messages=[{"role": "user", "content": "こんにちは"}],
)
print(response.choices[0].message.content)
```

```python
# LiteLLMのようなラッパーで複数ベンダーを統一的に扱う例（概念）
import litellm

response = litellm.completion(
    model="anthropic/claude-opus-4-8",
    messages=[{"role": "user", "content": "こんにちは"}],
)
```

## 演習課題

1. OpenAI互換APIが「移行コストを下げる」とはどういう意味か説明せよ
2. 互換エンドポイントに切り替える際、動作確認すべきでない前提を1つ挙げよ

## 理解度チェック

- [ ] OpenAI形式が事実上の標準になった背景を説明できる
- [ ] 「互換」がカバーする範囲とカバーしない範囲を区別できる
- [ ] ベンダー固有パラメータをそのまま移植できない理由を説明できる

---
前へ: [../03-advanced-features/04-structured-outputs.md](../03-advanced-features/04-structured-outputs.md) | 次へ: [02-mcp-protocol.md](02-mcp-protocol.md)
