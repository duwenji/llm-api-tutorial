# 認証の仕組み

## この教材で身につくこと

- APIキー認証の基本と安全な扱い方
- OAuth/Bearerトークン認証との違い
- 認証エラー（401/403）の典型的な原因

## 概要

LLM APIの多くはAPIキーによる認証を基本としますが、
CLIツールや社内システムではOAuthベースの認証も使われます。

## 位置づけ

02章以降のすべての実装例で共通して必要となる、実務上の前提知識です。

## 仕組み解説

### 認証方式の比較

| 方式 | 送信ヘッダー | 主な用途 |
|------|--------------|----------|
| APIキー | `x-api-key: <key>` | サーバー間の直接呼び出し |
| Bearerトークン | `Authorization: Bearer <token>` | OAuth連携・CLI認証 |
| Workload Identity Federation | 専用ヘッダー不要（JWT交換） | クラウド環境の無鍵運用 |

### 安全な扱い方

```python
# ✅ 良い例: 環境変数からAPIキーを読む
import os
import anthropic

client = anthropic.Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])
```

```python
# ❌ 悪い例: コード中に直接書く（漏洩・履歴混入のリスク）
client = anthropic.Anthropic(api_key="sk-ant-xxxxxxxx")
```

### よくある認証エラー

| コード | 原因 | 対処 |
|--------|------|------|
| 401 | APIキー未設定・無効・失効 | 環境変数を確認、再発行 |
| 403 | 権限不足（モデルへのアクセス権なし） | プランやワークスペース設定を確認 |
| 401（OAuth） | `Authorization`ヘッダーの形式誤り | `Bearer <token>`形式か確認 |

## 実装例

### Claude API

```bash
export ANTHROPIC_API_KEY="your-api-key"

curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{"model": "claude-opus-4-8", "max_tokens": 100, "messages": [{"role":"user","content":"Hi"}]}'
```

```python
# OAuthトークンを使う場合の例（概念）
headers = {
    "Authorization": f"Bearer {access_token}",
    "anthropic-beta": "oauth-2025-04-20",
}
```

### OpenAI公式API

```bash
export OPENAI_API_KEY="your-api-key"

curl https://api.openai.com/v1/chat/completions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "gpt-4o", "messages": [{"role":"user","content":"Hi"}]}'
```

```python
import os
from openai import OpenAI

client = OpenAI(api_key=os.environ["OPENAI_API_KEY"])
```

> 対応表: ClaudeはAPIキーを`x-api-key`ヘッダーに、
> OpenAIは`Authorization: Bearer <key>`ヘッダーに載せる。
> どちらもOAuthベースの認証（`Authorization: Bearer`）も別途サポートする。

## 演習課題

1. APIキーをコードに直書きしてはいけない理由を2つ挙げよ
2. 401エラーと403エラーの違いを説明せよ
3. Claude APIとOpenAI公式APIで、APIキーを載せるヘッダー名の違いを説明せよ

## 理解度チェック

- [ ] APIキー認証とOAuth認証の違いを説明できる
- [ ] APIキーを安全に管理する方法（環境変数化）を実践できる
- [ ] 401/403エラーの典型的な原因を切り分けられる
- [ ] Claude APIとOpenAI公式APIの認証ヘッダー名の違いを説明できる

---
前へ: [03-tokens-and-context.md](03-tokens-and-context.md) | 次へ: [../02-core-mechanics/00-README.md](../02-core-mechanics/00-README.md)
