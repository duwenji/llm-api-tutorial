# マルチモーダル入力

## この教材で身につくこと

- 画像・PDFをAPIリクエストに含める方法
- base64埋め込みとURL参照の使い分け
- マルチモーダル入力のコスト（トークン）に対する影響

## 概要

現在のLLM APIの多くは、テキストだけでなく画像やPDFを
コンテンツブロックとして受け取れます。これを「マルチモーダル」と呼びます。

## 位置づけ

02章の「メッセージとロール」で学んだcontentの構造を拡張し、
テキスト以外のブロック種別を扱います。

## 仕組み解説

### コンテンツブロックの種類

| type | 用途 | ソース指定 |
|------|------|-----------|
| `text` | 通常のテキスト | 文字列 |
| `image` | 画像入力 | `base64` または `url` |
| `document` | PDF/テキスト文書 | `base64` または `file_id`（Files API） |

> **Claude vs OpenAI**: OpenAI公式APIの画像入力は
> `{"type": "image_url", "image_url": {"url": "data:image/png;base64,..."}}`
> という形式で、Claudeの`{"type": "image", "source": {...}}`とは
> ブロック構造そのものが異なる。

### base64 vs URL vs Files API

| 方式 | 向いている場面 |
|------|----------------|
| base64埋め込み | ローカルファイル、1回限りの利用 |
| URL参照 | 既に公開URLがある画像 |
| Files API（`file_id`） | 同じファイルを複数リクエストで再利用する場合 |

```json
// ✅ 良い例: 同じPDFを何度も使う場合はFiles APIで一度だけアップロード
{"type": "document", "source": {"type": "file", "file_id": "file_abc123"}}
```

```json
// ❌ 悪い例: 毎回同じPDFをbase64で埋め込み直す（無駄な帯域とトークン）
{"type": "document", "source": {"type": "base64", "media_type": "application/pdf", "data": "..."}}
```

### コストへの影響

画像は解像度に応じてトークン換算されます。
高解像度画像は1枚で数千トークンを消費することがあるため、
不要な高解像度は事前に縮小するのが安全です。

## 実装例

### Claude API

```python
import base64

with open("photo.png", "rb") as f:
    image_data = base64.standard_b64encode(f.read()).decode("utf-8")

response = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {"type": "image", "source": {
                "type": "base64", "media_type": "image/png", "data": image_data,
            }},
            {"type": "text", "text": "この画像に写っているものを説明してください"},
        ],
    }],
)
print(response.content[0].text)
```

```python
# PDFをFiles APIで一度アップロードし、複数回参照する
uploaded = client.beta.files.upload(file=("report.pdf", open("report.pdf", "rb"), "application/pdf"))

response = client.beta.messages.create(
    model="claude-opus-4-8", max_tokens=1024,
    messages=[{"role": "user", "content": [
        {"type": "text", "text": "要点を3つ挙げて"},
        {"type": "document", "source": {"type": "file", "file_id": uploaded.id}},
    ]}],
    betas=["files-api-2025-04-14"],
)
```

### OpenAI公式API

```python
import base64

with open("photo.png", "rb") as f:
    image_data = base64.standard_b64encode(f.read()).decode("utf-8")

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "この画像に写っているものを説明してください"},
            {"type": "image_url", "image_url": {
                "url": f"data:image/png;base64,{image_data}",
            }},
        ],
    }],
)
print(response.choices[0].message.content)
```

> 対応表: Claudeは`{"type":"image","source":{"type":"base64",...}}`、
> OpenAIは`{"type":"image_url","image_url":{"url":"data:..."}}`という
> data URIスキームでbase64画像を渡す。

## 演習課題

1. 同じ画像を10回問い合わせる場合、base64とFiles APIのどちらが
   効率的か理由とともに答えよ
2. 画像入力のトークンコストを抑える方法を1つ挙げよ
3. Claude APIとOpenAI公式APIで、base64画像を渡すコンテンツブロックの
   構造の違いを説明せよ

## 理解度チェック

- [ ] base64・URL・Files APIの使い分けを説明できる
- [ ] 画像入力がトークンコストに影響することを理解している
- [ ] マルチモーダルなコンテンツブロックのJSON構造を書ける
- [ ] Claude APIとOpenAI公式APIの画像コンテンツブロック構造の違いを説明できる

---
前へ: [../02-core-mechanics/03-tool-use.md](../02-core-mechanics/03-tool-use.md) | 次へ: [02-extended-thinking.md](02-extended-thinking.md)
