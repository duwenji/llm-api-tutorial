# 構造化出力

## この教材で身につくこと

- 構造化出力（Structured Outputs）が保証する内容
- JSON Schemaによる出力形式の制約方法
- 従来の「プレフィル」による代用手法との違い

## 概要

構造化出力は、モデルの応答を指定したJSON Schemaに
厳密に従わせる仕組みです。パース失敗のリスクをなくせます。

## 位置づけ

02章のツール呼び出しと組み合わせて、
05章のエージェント実装で頻繁に使う技術です。

## 仕組み解説

### 構造化出力 vs 旧来のプレフィル手法

| 手法 | 仕組み | 課題 |
|------|--------|------|
| プレフィル（旧） | assistantターンの先頭を`{`などで埋めてJSONを誘導 | 保証がなく、壊れたJSONが混じる |
| 構造化出力（新） | `output_config.format`にJSON Schemaを指定 | スキーマ通りの出力を保証 |

```json
// ❌ 悪い例（旧手法）: プレフィルでJSON開始を強制するが壊れる場合がある
{"messages": [
  {"role": "user", "content": "名前を抽出して"},
  {"role": "assistant", "content": "{\"name\": \""}
]}
```

```json
// ✅ 良い例: output_configでスキーマを直接指定する
{
  "output_config": {
    "format": {
      "type": "json_schema",
      "schema": {
        "type": "object",
        "properties": {"name": {"type": "string"}},
        "required": ["name"],
        "additionalProperties": false
      }
    }
  }
}
```

### JSON Schemaでサポートされる範囲

| サポート | 対象 |
|----------|------|
| ✅ | 基本型、`enum`、`const`、`anyOf` |
| ✅ | `additionalProperties: false`（必須） |
| ❌ | `minLength`/`maximum`などの数値・文字数制約 |
| ❌ | 再帰スキーマ |

### OpenAI公式APIとの対応

OpenAIは`response_format`に`{"type": "json_schema", "json_schema": {...}}`
を指定し、`strict: true`で厳密なスキーマ準拠を有効にする。

| 概念 | Claude API | OpenAI公式API |
|------|-----------|----------------|
| 出力形式の指定 | `output_config.format` | `response_format` |
| 厳密モード | 常に厳密（`json_schema`指定時） | `strict: true`を明示指定 |

## 実装例

### Claude API

```python
from pydantic import BaseModel

class ContactInfo(BaseModel):
    name: str
    email: str
    plan: str

response = client.messages.parse(
    model="claude-opus-4-8",
    max_tokens=1024,
    messages=[{"role": "user", "content":
        "Jane Doe (jane@example.com) はEnterpriseプランを希望"}],
    output_format=ContactInfo,
)
contact = response.parsed_output
print(contact.name, contact.email, contact.plan)
```

### OpenAI公式API

```python
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content":
        "Jane Doe (jane@example.com) はEnterpriseプランを希望"}],
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "contact_info",
            "strict": True,
            "schema": {
                "type": "object",
                "properties": {
                    "name": {"type": "string"},
                    "email": {"type": "string"},
                    "plan": {"type": "string"},
                },
                "required": ["name", "email", "plan"],
                "additionalProperties": False,
            },
        },
    },
)
import json
contact = json.loads(response.choices[0].message.content)
print(contact["name"], contact["email"], contact["plan"])
```

> 対応表: Claudeは`output_config.format`、OpenAIは`response_format`。
> OpenAIは`strict: true`を明示しないと厳密なスキーマ準拠にならない。

## 演習課題

1. 「顧客名・注文ID・金額」を抽出するJSON Schemaを書け
2. プレフィル手法より構造化出力が優れている理由を1つ挙げよ
3. Claude APIとOpenAI公式APIで、構造化出力を指定するパラメータ名の違いを説明せよ

## 理解度チェック

- [ ] 構造化出力がスキーマ準拠を保証する仕組みを説明できる
- [ ] `additionalProperties: false`が必要な理由を理解している
- [ ] 数値制約がサポートされないケースを把握している
- [ ] Claude APIとOpenAI公式APIの構造化出力パラメータ名の違いを説明できる

---
前へ: [03-prompt-caching.md](03-prompt-caching.md) | 次へ: [../04-standardization/00-README.md](../04-standardization/00-README.md)
