# エラー・バージョニング標準

## この教材で身につくこと

- HTTPステータスコードによるエラー分類の共通パターン
- APIバージョニングの標準化状況
- 標準化されていない部分を見極める視点

## 概要

エラーハンドリングは、HTTPステータスコードという既存の
Web標準に乗る形で、業界横断的にある程度共通化されています。
一方でモデル名・料金体系・拡張パラメータは統一されていません。

## 位置づけ

これまで学んだ標準化（OpenAI互換API、MCP）を踏まえ、
「何が標準化され、何がベンダー固有か」を整理する総まとめです。

## 仕組み解説

### 標準化されている部分

| 項目 | 標準化の状態 |
|------|--------------|
| HTTPステータスコード | 400/401/403/404/429/500/529などWeb標準に準拠 |
| リトライ指針 | 429/5xxはリトライ可、4xx（429除く）はリトライ不可、が共通認識 |
| SSEによるストリーミング | HTTP標準規格のため共通 |
| JSON Schemaベースの出力制約 | JSON Schema自体はIETF標準 |

### 標準化されていない部分

| 項目 | ベンダー差の例 |
|------|----------------|
| モデルID命名規則 | `claude-opus-4-8` vs `gpt-4o` vs `gemini-2.0-flash` |
| エラー本文の内部構造 | `error.type`のとりうる値がベンダーごとに異なる |
| バージョン指定方法 | ヘッダー指定（`anthropic-version`）vs URLパス埋め込み |
| 拡張思考・エフォート等の新機能 | パラメータ名・挙動が統一されていない |

### エラーハンドリングの実装パターン

```python
import anthropic

try:
    response = client.messages.create(...)
except anthropic.RateLimitError as e:
    retry_after = int(e.response.headers.get("retry-after", "60"))
    # 待ってからリトライ
except anthropic.APIStatusError as e:
    if e.status_code >= 500:
        pass  # リトライ可能
    else:
        raise  # 4xxは基本的にリトライ不可
except anthropic.APIConnectionError:
    pass  # ネットワークエラー、リトライ検討
```

```python
# ❌ 悪い例: すべてのエラーを一括でリトライする
try:
    response = client.messages.create(...)
except Exception:
    retry()  # 400や401までリトライしても無駄に失敗が続く
```

## 実装例

### Claude API

```python
# 型付き例外を最も具体的なものから順にキャッチする
try:
    response = client.messages.create(model="claude-opus-4-8", max_tokens=1024,
                                        messages=[{"role": "user", "content": "Hi"}])
except anthropic.NotFoundError:
    print("モデルIDを確認してください")
except anthropic.RateLimitError:
    print("しばらく待ってから再試行してください")
except anthropic.APIStatusError as e:
    print(f"APIエラー: {e.status_code} {e.message}")
```

### OpenAI公式API

```python
import openai

try:
    response = client.chat.completions.create(
        model="gpt-4o", messages=[{"role": "user", "content": "Hi"}],
    )
except openai.NotFoundError:
    print("モデルIDを確認してください")
except openai.RateLimitError:
    print("しばらく待ってから再試行してください")
except openai.APIStatusError as e:
    print(f"APIエラー: {e.status_code} {e.message}")
```

> 対応表: どちらのSDKもHTTPステータスコード（404/429/5xx等）に沿った
> 型付き例外クラスを提供し、キャッチする順序・考え方は共通している。

## 演習課題

1. 429エラーと400エラーで、リトライ戦略をどう変えるべきか説明せよ
2. モデルID命名規則が標準化されていない理由を推測して述べよ
3. Claude SDKとOpenAI SDKの型付き例外の使い方がどれだけ似ているか説明せよ

## 理解度チェック

- [ ] 標準化されている部分・されていない部分を切り分けられる
- [ ] HTTPステータスコードに基づくリトライ判断ができる
- [ ] 型付き例外を使うべき理由を説明できる
- [ ] Claude SDKとOpenAI SDKの型付き例外の共通パターンを説明できる

---
前へ: [02-mcp-protocol.md](02-mcp-protocol.md) | 次へ: [../05-real-world-examples/00-README.md](../05-real-world-examples/00-README.md)
