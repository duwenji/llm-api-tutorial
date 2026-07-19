# Claude/OpenAI 並列比較 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Revise `llm-api-tutorial` so every implementation-example lesson shows Claude API and OpenAI official API side by side, using consistent terminology and a shared mapping table.

**Architecture:** This is a documentation-only change set. Each task edits a fixed list of Markdown files by inserting new headed subsections (`### Claude API` / `### OpenAI公式API`) into existing `## 実装例` sections, adding small mapping tables to `## 仕組み解説` where terminology differs, and appending one exercise + one checklist item per file that gained a comparison. No code is executed; "tests" are `Grep` verification checks that the new headings/content landed correctly.

**Tech Stack:** Markdown, git. Reference spec: `docs/superpowers/specs/2026-07-19-claude-openai-parallel-comparison-design.md`.

## Global Constraints

- Follow `00_STYLE_GUIDE.md`: language tags on every code block, ≤90 chars/line in code, short sentences, no duplicate explanations.
- Model IDs used throughout: `claude-opus-4-8` (Claude) and `gpt-4o` (OpenAI). Do not introduce other model IDs.
- Terminology: always write "OpenAI公式API" (official, `api.openai.com`) and "OpenAI互換（サードパーティ）API" (third-party compatible servers) — never bare "OpenAI API" when the distinction matters.
- Do not state specific OpenAI pricing figures. Compare mechanisms/behavior only (e.g., "自動適用" vs "明示指定"), never dollar amounts for OpenAI.
- OpenAI code examples use the stable Chat Completions surface only (`client.chat.completions.create`, `messages`, `tools`/`tool_calls`, `response_format`, `image_url`). Do not use the Responses API or invent parameters not listed in this plan's code blocks.
- Every file edit in this plan gives the exact `old_string` (unique anchor copied from the file as currently committed) and the exact `new_string` to replace it with. Apply with the Edit tool exactly as written — do not paraphrase or "improve" the wording.
- After editing each file, verify with the `Grep` command given in that step before moving to the next file.
- One commit per task, after all files in that task are edited and verified.

---

### Task 1: QUICK-REFERENCE.md — add the Claude ⇔ OpenAI mapping table

This table is the canonical reference. Later tasks' per-lesson mapping notes must not contradict it.

**Files:**
- Modify: `QUICK-REFERENCE.md`

- [ ] **Step 1: Read the file**

Run: view `QUICK-REFERENCE.md` to confirm current content matches the anchor below (it was last touched when the tutorial was first created and has not changed since).

- [ ] **Step 2: Insert the mapping table**

old_string:
````markdown
| SSE (Server-Sent Events) | 通信標準 | ストリーミング応答で広く使われるHTTP標準規格 |

## コスト計算の基本式
````

new_string:
````markdown
| SSE (Server-Sent Events) | 通信標準 | ストリーミング応答で広く使われるHTTP標準規格 |

## Claude ⇔ OpenAI 対応表

会話の構造・呼び出し形式で名称が異なる概念の対応一覧です。
個別教材の対応表はこの表と矛盾しないようにしてください。

| 概念 | Claude API | OpenAI公式API |
|------|-----------|----------------|
| エンドポイント | `POST /v1/messages` | `POST /v1/chat/completions` |
| 認証ヘッダー | `x-api-key: <key>` | `Authorization: Bearer <key>` |
| systemメッセージ | 独立した`system`フィールド | `messages`配列内の`{"role":"system",...}` |
| 応答テキストの取得 | `response.content[0].text` | `response.choices[0].message.content` |
| ツール定義フィールド | `input_schema` | `parameters` |
| ツール呼び出しブロック | `tool_use`（`content`内） | `tool_calls`（`message`内） |
| ツール結果の返送 | `tool_result`（userメッセージ内） | `role: "tool"`の独立メッセージ |
| 応答終了理由 | `stop_reason`（`tool_use`, `end_turn`等） | `finish_reason`（`tool_calls`, `stop`等） |
| ストリーミング終端 | `message_stop`イベント | `data: [DONE]` |
| 画像入力ブロック | `{"type":"image","source":{...}}` | `{"type":"image_url","image_url":{"url":"..."}}` |
| 構造化出力 | `output_config.format`（`json_schema`） | `response_format`（`json_schema`, `strict`） |
| プロンプトキャッシュ | `cache_control`を明示指定（任意） | 一定長以上で自動適用（指定不要） |
| 内部推論の制御 | `thinking`（`type: "adaptive"`）+ `effort` | 推論系モデルの`reasoning_effort` |
| トークン数の事前確認 | `client.messages.count_tokens()` | `tiktoken`ライブラリ |

## コスト計算の基本式
````

- [ ] **Step 3: Verify**

Run: `grep -c "Claude ⇔ OpenAI 対応表" QUICK-REFERENCE.md`
Expected: `1`

Run: `grep -c "^|" QUICK-REFERENCE.md`
Expected: a number at least 14 higher than before the edit (new table rows + header + separator).

- [ ] **Step 4: Commit**

```bash
git add QUICK-REFERENCE.md
git commit -m "docs: add Claude/OpenAI mapping table to quick reference"
```

---

### Task 2: 01-history-and-basics — 4 files

**Files:**
- Modify: `docs/01-history-and-basics/01-history-of-llm-api.md`
- Modify: `docs/01-history-and-basics/02-api-fundamentals.md`
- Modify: `docs/01-history-and-basics/03-tokens-and-context.md`
- Modify: `docs/01-history-and-basics/04-authentication.md`

#### 2.1 `01-history-of-llm-api.md`

- [ ] **Step 1: Insert "主要プロバイダの独立公開時期" subsection**

old_string:
````markdown
          → エージェント基盤（自律的なタスク遂行）
```

## 実装例
````

new_string:
````markdown
          → エージェント基盤（自律的なタスク遂行）
```

### 主要プロバイダの独立公開時期

LLM APIは1社の独占ではなく、複数の企業がそれぞれ独自にAPIを公開しています。

| プロバイダ | API公開時期 | 提供元 |
|-----------|-------------|--------|
| OpenAI | 2020年（GPT-3） | OpenAI |
| Anthropic | 2023年（Claude API） | Anthropic |
| Google | 2023年（Gemini API） | Google |

以降の章では、特に**OpenAI公式API**と**Claude API**を軸に、
同じ概念がベンダーごとにどう異なる形で表現されるかを比較していきます。

## 実装例
````

- [ ] **Step 2: Verify**

Run: `grep -c "主要プロバイダの独立公開時期" docs/01-history-and-basics/01-history-of-llm-api.md`
Expected: `1`

#### 2.2 `02-api-fundamentals.md`

- [ ] **Step 1: Restructure 実装例 with Claude/OpenAI subsections**

old_string:
````markdown
## 実装例

```bash
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "claude-opus-4-8",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "こんにちは"}]
  }'
```

```python
import anthropic

client = anthropic.Anthropic()  # ANTHROPIC_API_KEY 環境変数を利用
response = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=1024,
    messages=[{"role": "user", "content": "こんにちは"}],
)
print(response.content[0].text)
```

## 演習課題

1. 3ターンの会話（自己紹介→質問→回答確認）を送るJSONを書け
2. なぜAPIサーバーが会話状態を保持しない設計になっているか、利点を1つ挙げよ

## 理解度チェック

- [ ] リクエストとレスポンスの基本的なJSON構造を説明できる
- [ ] ステートレス設計の意味と、クライアント側の責務を説明できる
- [ ] 認証ヘッダーとバージョンヘッダーの役割の違いを言える
````

new_string:
````markdown
## 実装例

### Claude API

```bash
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "claude-opus-4-8",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "こんにちは"}]
  }'
```

```python
import anthropic

client = anthropic.Anthropic()  # ANTHROPIC_API_KEY 環境変数を利用
response = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=1024,
    messages=[{"role": "user", "content": "こんにちは"}],
)
print(response.content[0].text)
```

### OpenAI公式API

```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "gpt-4o",
    "messages": [{"role": "user", "content": "こんにちは"}]
  }'
```

```python
from openai import OpenAI

client = OpenAI()  # OPENAI_API_KEY 環境変数を利用
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "こんにちは"}],
)
print(response.choices[0].message.content)
```

> 対応表: 認証ヘッダーは`x-api-key`（Claude） vs `Authorization: Bearer`（OpenAI）。
> 応答テキストは`content[0].text`（Claude） vs `choices[0].message.content`（OpenAI）。

## 演習課題

1. 3ターンの会話（自己紹介→質問→回答確認）を送るJSONを書け
2. なぜAPIサーバーが会話状態を保持しない設計になっているか、利点を1つ挙げよ
3. Claude APIとOpenAI公式APIで、応答テキストを取り出すコードがどう違うか説明せよ

## 理解度チェック

- [ ] リクエストとレスポンスの基本的なJSON構造を説明できる
- [ ] ステートレス設計の意味と、クライアント側の責務を説明できる
- [ ] 認証ヘッダーとバージョンヘッダーの役割の違いを言える
- [ ] Claude APIとOpenAI公式APIの認証ヘッダー形式の違いを説明できる
````

- [ ] **Step 2: Verify**

Run: `grep -c "### OpenAI公式API" docs/01-history-and-basics/02-api-fundamentals.md`
Expected: `1`

#### 2.3 `03-tokens-and-context.md`

- [ ] **Step 1: Restructure 実装例, add tiktoken example**

old_string:
````markdown
## 実装例

```python
# トークン数を事前に確認する（例: Anthropic API）
count = client.messages.count_tokens(
    model="claude-opus-4-8",
    messages=[{"role": "user", "content": "見積もりたい本文..."}],
)
print(count.input_tokens)
```

```python
# コスト概算（例: 入力 $3/1M, 出力 $15/1M）
input_cost = count.input_tokens * 3 / 1_000_000
print(f"推定入力コスト: ${input_cost:.4f}")
```

## 演習課題

1. コンテキストウィンドウ20万トークンのモデルに、
   18万トークンの入力を送った場合、出力に使える最大トークン数を求めよ
2. `tiktoken`（OpenAI用）で他社モデルのトークン数を見積もってはいけない
   理由を説明せよ

## 理解度チェック

- [ ] トークン化がモデルごとに異なる理由を説明できる
- [ ] コンテキストウィンドウが「入力+出力」の合計である点を理解している
- [ ] `max_tokens`不足による出力打ち切りの症状を説明できる
````

new_string:
````markdown
## 実装例

### Claude API

```python
# トークン数を事前に確認する（Claudeは専用のcount_tokens APIを使う）
count = client.messages.count_tokens(
    model="claude-opus-4-8",
    messages=[{"role": "user", "content": "見積もりたい本文..."}],
)
print(count.input_tokens)
```

```python
# コスト概算（例: 入力 $3/1M, 出力 $15/1M）
input_cost = count.input_tokens * 3 / 1_000_000
print(f"推定入力コスト: ${input_cost:.4f}")
```

### OpenAI公式API

```python
# OpenAIは公式トークナイザーtiktokenでトークン数を見積もれる
import tiktoken

encoding = tiktoken.encoding_for_model("gpt-4o")
token_count = len(encoding.encode("見積もりたい本文..."))
print(token_count)
```

> 対応表: Claudeは`count_tokens` APIを呼ぶ必要があるのに対し、
> OpenAIは`tiktoken`をローカルで実行するだけで見積もれる。
> ただし`tiktoken`はOpenAIモデル専用で、Claude等の見積もりには使えない。

## 演習課題

1. コンテキストウィンドウ20万トークンのモデルに、
   18万トークンの入力を送った場合、出力に使える最大トークン数を求めよ
2. `tiktoken`（OpenAI用）で他社モデルのトークン数を見積もってはいけない
   理由を説明せよ
3. Claude APIとOpenAI公式APIで、トークン数を事前確認する方法の違いを説明せよ

## 理解度チェック

- [ ] トークン化がモデルごとに異なる理由を説明できる
- [ ] コンテキストウィンドウが「入力+出力」の合計である点を理解している
- [ ] `max_tokens`不足による出力打ち切りの症状を説明できる
- [ ] `count_tokens` APIと`tiktoken`ライブラリの使い分けを説明できる
````

- [ ] **Step 2: Verify**

Run: `grep -c "tiktoken.encoding_for_model" docs/01-history-and-basics/03-tokens-and-context.md`
Expected: `1`

#### 2.4 `04-authentication.md`

- [ ] **Step 1: Restructure 実装例 with both providers**

old_string:
````markdown
## 実装例

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

## 演習課題

1. APIキーをコードに直書きしてはいけない理由を2つ挙げよ
2. 401エラーと403エラーの違いを説明せよ

## 理解度チェック

- [ ] APIキー認証とOAuth認証の違いを説明できる
- [ ] APIキーを安全に管理する方法（環境変数化）を実践できる
- [ ] 401/403エラーの典型的な原因を切り分けられる
````

new_string:
````markdown
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
````

- [ ] **Step 2: Verify**

Run: `grep -c "Authorization: Bearer \$OPENAI_API_KEY" docs/01-history-and-basics/04-authentication.md`
Expected: `1`

- [ ] **Step 3: Commit Task 2**

```bash
git add docs/01-history-and-basics
git commit -m "docs: add OpenAI comparisons to history-and-basics chapter"
```

---

### Task 3: 02-core-mechanics — 3 files

**Files:**
- Modify: `docs/02-core-mechanics/01-messages-and-roles.md`
- Modify: `docs/02-core-mechanics/02-streaming.md`
- Modify: `docs/02-core-mechanics/03-tool-use.md`

#### 3.1 `01-messages-and-roles.md`

- [ ] **Step 1: Add structural-difference callout after the role table**

old_string:
````markdown
| `tool_result` | ツール実行結果の返送 | `user`メッセージ内の特殊ブロックとして含む |

### システムプロンプトの位置づけ
````

new_string:
````markdown
| `tool_result` | ツール実行結果の返送 | `user`メッセージ内の特殊ブロックとして含む |

> **Claude vs OpenAI**: Claudeは`system`を独立したトップレベルフィールドとして
> 分離するが、OpenAI公式APIでは`system`も`messages`配列内の
> 1メッセージ（`{"role": "system", "content": "..."}`）として扱う。

### システムプロンプトの位置づけ
````

- [ ] **Step 2: Restructure 実装例 with both providers**

old_string:
````markdown
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
````

new_string:
````markdown
## 実装例

### Claude API

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

### OpenAI公式API

```python
messages = [{"role": "system", "content": "あなたは簡潔に回答するアシスタントです。"}]

def send(user_text: str) -> str:
    messages.append({"role": "user", "content": user_text})
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=messages,
    )
    reply = response.choices[0].message.content
    messages.append({"role": "assistant", "content": reply})
    return reply

print(send("私の名前はAliceです"))
print(send("私の名前は何でしたか？"))  # "Alice"を覚えている
```

> 対応表: Claudeは`system`パラメータが独立しているが、
> OpenAIは`messages`配列の先頭に`{"role": "system", ...}`を置く。

## 演習課題

1. 「敬語で回答する」「箇条書きは使わない」という制約を
   systemプロンプトとして書け
2. 3ターンの会話（自己紹介→好きな食べ物→質問確認）のJSON配列を作れ
3. Claude APIとOpenAI公式APIで、systemプロンプトの位置づけの違いを説明せよ

## 理解度チェック

- [ ] system/user/assistantそれぞれの役割を説明できる
- [ ] 可変情報をsystemプロンプトに入れるべきでない理由を説明できる
- [ ] 複数ターンの会話履歴を正しい形式で組み立てられる
- [ ] Claude APIとOpenAI公式APIでsystemの扱いがどう違うか説明できる
````

- [ ] **Step 3: Verify**

Run: `grep -c "role.*system.*content" docs/02-core-mechanics/01-messages-and-roles.md`
Expected: at least `2`

#### 3.2 `02-streaming.md`

- [ ] **Step 1: Insert OpenAI SSE note, restructure 実装例, extend exercises/checklist (single span edit)**

old_string:
````markdown
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
````

new_string:
````markdown
| `message_stop` | 応答完了時に1回 |

### OpenAIのストリーミング形式との違い

OpenAI公式APIは名前付きイベントを使わず、`chat.completion.chunk`型の
チャンクを連続送信し、最後に`data: [DONE]`を送って終端を示す。

```
data: {"choices":[{"delta":{"content":"こん"}}]}

data: {"choices":[{"delta":{"content":"にちは"}}]}

data: [DONE]
```

## 実装例

### Claude API

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

### OpenAI公式API

```python
stream = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "俳句を書いてください"}],
    stream=True,
)
for chunk in stream:
    delta = chunk.choices[0].delta.content
    if delta:
        print(delta, end="", flush=True)
```

> 対応表: Claudeは`message_stop`イベントで終端を示すが、
> OpenAIはチャンク列の最後に`data: [DONE]`を送って終端を示す。

## 演習課題

1. 非ストリーミングと比べてストリーミングが優れている点を2つ挙げよ
2. `content_block_delta`と`message_stop`イベントの違いを説明せよ
3. Claude APIとOpenAI公式APIで、ストリーミングの終端の示し方の違いを説明せよ

## 理解度チェック

- [ ] ストリーミングが必要になる典型的な場面を説明できる
- [ ] SSEの主要イベント種別と発生順序を理解している
- [ ] ストリーミング中に最終応答をまとめて取得する方法を知っている
- [ ] Claude APIとOpenAI公式APIのストリーミング終端方式の違いを説明できる
````

- [ ] **Step 2: Verify**

Run: `grep -c "data: \[DONE\]" docs/02-core-mechanics/02-streaming.md`
Expected: `1`

#### 3.3 `03-tool-use.md`

- [ ] **Step 1: Add mapping table, restructure 実装例, extend exercises/checklist (single span edit)**

old_string:
````markdown
`tool_use_id`が一致していないと、モデル側は結果を正しく紐づけられません。

## 実装例

```python
tools = [{
    "name": "get_weather",
    "description": "指定した都市の現在の天気を取得する",
    "input_schema": {
        "type": "object",
        "properties": {"location": {"type": "string"}},
        "required": ["location"],
    },
}]

messages = [{"role": "user", "content": "東京の天気を教えて"}]
response = client.messages.create(
    model="claude-opus-4-8", max_tokens=1024, tools=tools, messages=messages,
)

if response.stop_reason == "tool_use":
    tool_call = next(b for b in response.content if b.type == "tool_use")
    result = f"{tool_call.input['location']}: 晴れ、24℃"  # 実際のAPI呼び出しに置換

    messages.append({"role": "assistant", "content": response.content})
    messages.append({
        "role": "user",
        "content": [{"type": "tool_result", "tool_use_id": tool_call.id, "content": result}],
    })

    final = client.messages.create(
        model="claude-opus-4-8", max_tokens=1024, tools=tools, messages=messages,
    )
    print(final.content[0].text)
```

## 演習課題

1. 「注文番号から配送状況を取得する」ツールの`input_schema`を書け
2. `tool_use_id`を正しく対応させないとどうなるか説明せよ

## 理解度チェック

- [ ] ツール呼び出しの5ステップの往復フローを説明できる
- [ ] `description`が呼び出し判断に重要な理由を説明できる
- [ ] `tool_use`と`tool_result`を正しく対応づけて送信できる
````

new_string:
````markdown
`tool_use_id`が一致していないと、モデル側は結果を正しく紐づけられません。

### OpenAI公式APIとの用語対応

| 概念 | Claude API | OpenAI公式API |
|------|-----------|----------------|
| ツール定義フィールド | `input_schema` | `parameters` |
| 呼び出しブロック | `tool_use`（`content`内） | `tool_calls`（`message`内） |
| 結果の返送 | `tool_result`（userメッセージ内） | `role: "tool"`の独立メッセージ |
| 対応ID | `tool_use_id` | `tool_call_id` |

## 実装例

### Claude API

```python
tools = [{
    "name": "get_weather",
    "description": "指定した都市の現在の天気を取得する",
    "input_schema": {
        "type": "object",
        "properties": {"location": {"type": "string"}},
        "required": ["location"],
    },
}]

messages = [{"role": "user", "content": "東京の天気を教えて"}]
response = client.messages.create(
    model="claude-opus-4-8", max_tokens=1024, tools=tools, messages=messages,
)

if response.stop_reason == "tool_use":
    tool_call = next(b for b in response.content if b.type == "tool_use")
    result = f"{tool_call.input['location']}: 晴れ、24℃"  # 実際のAPI呼び出しに置換

    messages.append({"role": "assistant", "content": response.content})
    messages.append({
        "role": "user",
        "content": [{"type": "tool_result", "tool_use_id": tool_call.id, "content": result}],
    })

    final = client.messages.create(
        model="claude-opus-4-8", max_tokens=1024, tools=tools, messages=messages,
    )
    print(final.content[0].text)
```

### OpenAI公式API

```python
tools = [{
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "指定した都市の現在の天気を取得する",
        "parameters": {
            "type": "object",
            "properties": {"location": {"type": "string"}},
            "required": ["location"],
        },
    },
}]

messages = [{"role": "user", "content": "東京の天気を教えて"}]
response = client.chat.completions.create(
    model="gpt-4o", tools=tools, messages=messages,
)

if response.choices[0].finish_reason == "tool_calls":
    tool_call = response.choices[0].message.tool_calls[0]
    args = json.loads(tool_call.function.arguments)
    result = f"{args['location']}: 晴れ、24℃"  # 実際のAPI呼び出しに置換

    messages.append(response.choices[0].message)
    messages.append({
        "role": "tool", "tool_call_id": tool_call.id, "content": result,
    })

    final = client.chat.completions.create(
        model="gpt-4o", tools=tools, messages=messages,
    )
    print(final.choices[0].message.content)
```

## 演習課題

1. 「注文番号から配送状況を取得する」ツールの`input_schema`を書け
2. `tool_use_id`を正しく対応させないとどうなるか説明せよ
3. Claude APIの`tool_use`とOpenAI公式APIの`tool_calls`の構造上の違いを説明せよ

## 理解度チェック

- [ ] ツール呼び出しの5ステップの往復フローを説明できる
- [ ] `description`が呼び出し判断に重要な理由を説明できる
- [ ] `tool_use`と`tool_result`を正しく対応づけて送信できる
- [ ] Claude APIとOpenAI公式APIのツール定義フィールド名の違いを説明できる
````

- [ ] **Step 2: Verify**

Run: `grep -c "tool_call_id" docs/02-core-mechanics/03-tool-use.md`
Expected: `1`

- [ ] **Step 3: Commit Task 3**

```bash
git add docs/02-core-mechanics
git commit -m "docs: add OpenAI comparisons to core-mechanics chapter"
```

---

### Task 4: 03-advanced-features — 4 files

**Files:**
- Modify: `docs/03-advanced-features/01-multimodal-input.md`
- Modify: `docs/03-advanced-features/02-extended-thinking.md`
- Modify: `docs/03-advanced-features/03-prompt-caching.md`
- Modify: `docs/03-advanced-features/04-structured-outputs.md`

#### 4.1 `01-multimodal-input.md`

- [ ] **Step 1: Add structural-difference callout after the content-block table**

old_string:
````markdown
| `document` | PDF/テキスト文書 | `base64` または `file_id`（Files API） |

### base64 vs URL vs Files API
````

new_string:
````markdown
| `document` | PDF/テキスト文書 | `base64` または `file_id`（Files API） |

> **Claude vs OpenAI**: OpenAI公式APIの画像入力は
> `{"type": "image_url", "image_url": {"url": "data:image/png;base64,..."}}`
> という形式で、Claudeの`{"type": "image", "source": {...}}`とは
> ブロック構造そのものが異なる。

### base64 vs URL vs Files API
````

- [ ] **Step 2: Restructure 実装例, extend exercises/checklist**

old_string:
````markdown
## 実装例

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

## 演習課題

1. 同じ画像を10回問い合わせる場合、base64とFiles APIのどちらが
   効率的か理由とともに答えよ
2. 画像入力のトークンコストを抑える方法を1つ挙げよ

## 理解度チェック

- [ ] base64・URL・Files APIの使い分けを説明できる
- [ ] 画像入力がトークンコストに影響することを理解している
- [ ] マルチモーダルなコンテンツブロックのJSON構造を書ける
````

new_string:
````markdown
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
````

- [ ] **Step 3: Verify**

Run: `grep -c "image_url" docs/03-advanced-features/01-multimodal-input.md`
Expected: at least `2`

#### 4.2 `02-extended-thinking.md`

- [ ] **Step 1: Add hedged OpenAI reasoning-model note after the effort table**

old_string:
````markdown
| `max` | 最大の精度が必要でコストを気にしない場合 |

## 実装例
````

new_string:
````markdown
| `max` | 最大の精度が必要でコストを気にしない場合 |

### OpenAIの推論系モデルとの概念対応

OpenAIの推論系モデル（o-series等）は`reasoning_effort`パラメータで
思考の深さを制御する点がClaudeの`effort`と似ている。ただし内部の
思考過程そのものは要約以外は非公開で、公開範囲や挙動はモデル・時期に
よって変わりやすい。詳細は必ず公式ドキュメントで確認すること。

| 概念 | Claude API | OpenAI公式API（推論系モデル） |
|------|-----------|-------------------------------|
| 思考の深さ制御 | `output_config.effort` | `reasoning_effort` |
| 思考ブロックの可視化 | `thinking`ブロック（`display`で制御） | モデル・時期により異なる（要公式ドキュメント確認） |

## 実装例
````

- [ ] **Step 2: Restructure 実装例, extend exercises/checklist**

old_string:
````markdown
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
````

new_string:
````markdown
## 実装例

### Claude API

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

### OpenAI公式API（推論系モデルの概念例）

```python
# reasoning_effortの対応状況・挙動はモデルによって異なるため、
# 実際に使う前に必ず公式ドキュメントで対象モデルの仕様を確認すること。
response = client.chat.completions.create(
    model="o-series-model-id",  # 実際のモデルIDは公式ドキュメント参照
    reasoning_effort="high",
    messages=[{"role": "user", "content": "27×453を暗算せず段階的に解いて"}],
)
print(response.choices[0].message.content)
```

> 対応表: Claudeの`effort`とOpenAIの`reasoning_effort`は「思考の深さを
> 制御する」という点で概念的に対応するが、内部思考の公開範囲・パラメータ名・
> 対応モデルはベンダー・時期により異なる。

## 演習課題

1. 拡張思考を使うべきタスクと、使わないほうがよいタスクを1つずつ挙げよ
2. `display: "omitted"`のとき、UI側で気をつけるべき点を説明せよ
3. Claudeの`effort`とOpenAIの`reasoning_effort`が概念的に対応する理由を説明せよ

## 理解度チェック

- [ ] 拡張思考が有効なタスクの傾向を説明できる
- [ ] `display`パラメータの2つの値の違いを理解している
- [ ] effortレベルをタスクの性質に応じて選べる
- [ ] Claudeの`effort`とOpenAIの`reasoning_effort`の対応関係を説明できる
````

- [ ] **Step 3: Verify**

Run: `grep -c "reasoning_effort" docs/03-advanced-features/02-extended-thinking.md`
Expected: at least `2`

#### 4.3 `03-prompt-caching.md`

- [ ] **Step 1: Add OpenAI automatic-caching note after the economics table**

old_string:
````markdown
| キャッシュ読み込み | 通常入力の約0.1倍 |

## 実装例
````

new_string:
````markdown
| キャッシュ読み込み | 通常入力の約0.1倍 |

### OpenAIの自動プロンプトキャッシュとの違い

OpenAI公式APIは`cache_control`のような明示的な指定が不要で、
一定長以上のプロンプトに対して自動的にキャッシュが適用される。
キャッシュヒット状況はレスポンスの`usage.prompt_tokens_details`配下
（`cached_tokens`等）で確認する。具体的なフィールド名・閾値は
モデル・時期により変わるため公式ドキュメントで確認すること。

## 実装例
````

- [ ] **Step 2: Restructure 実装例, extend exercises/checklist**

old_string:
````markdown
## 実装例

```json
{
  "model": "claude-opus-4-8",
  "max_tokens": 1024,
  "system": [
    {"type": "text", "text": "<長い共通システムプロンプト>", "cache_control": {"type": "ephemeral"}}
  ],
  "messages": [{"role": "user", "content": "今日の質問はこちらです"}]
}
```

```python
# キャッシュヒットの検証
print(response.usage.cache_creation_input_tokens)  # 書き込みトークン数
print(response.usage.cache_read_input_tokens)      # 読み込みトークン数

if response.usage.cache_read_input_tokens == 0:
    print("キャッシュが効いていない可能性 -> プレフィックスの揺れを確認")
```

## 演習課題

1. システムプロンプトにユーザーIDを埋め込むとキャッシュにどう影響するか説明せよ
2. `cache_creation_input_tokens`は増えるが`cache_read_input_tokens`が
   常に0になる場合、まず疑うべき原因を1つ挙げよ

## 理解度チェック

- [ ] プレフィックス一致というキャッシュの基本原理を説明できる
- [ ] キャッシュを壊す典型的なアンチパターンを挙げられる
- [ ] `usage`フィールドからキャッシュヒット状況を検証できる
````

new_string:
````markdown
## 実装例

### Claude API

```json
{
  "model": "claude-opus-4-8",
  "max_tokens": 1024,
  "system": [
    {"type": "text", "text": "<長い共通システムプロンプト>", "cache_control": {"type": "ephemeral"}}
  ],
  "messages": [{"role": "user", "content": "今日の質問はこちらです"}]
}
```

```python
# キャッシュヒットの検証
print(response.usage.cache_creation_input_tokens)  # 書き込みトークン数
print(response.usage.cache_read_input_tokens)      # 読み込みトークン数

if response.usage.cache_read_input_tokens == 0:
    print("キャッシュが効いていない可能性 -> プレフィックスの揺れを確認")
```

### OpenAI公式API

```python
# cache_controlのような指定は不要。同じ先頭プレフィックスを
# 繰り返すだけで自動的にキャッシュが適用される。
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "<長い共通システムプロンプト>"},
        {"role": "user", "content": "今日の質問はこちらです"},
    ],
)
# キャッシュヒット状況はusageの詳細フィールドで確認する
# （フィールド名は公式ドキュメントで確認すること）
print(response.usage)
```

> 対応表: Claudeは`cache_control`を明示指定する必要があるが、
> OpenAIは指定不要で自動適用される。可視化されるレスポンスの
> フィールド名は両者で異なる。

## 演習課題

1. システムプロンプトにユーザーIDを埋め込むとキャッシュにどう影響するか説明せよ
2. `cache_creation_input_tokens`は増えるが`cache_read_input_tokens`が
   常に0になる場合、まず疑うべき原因を1つ挙げよ
3. Claude APIとOpenAI公式APIで、キャッシュを有効にする方法（明示指定 vs 自動）の違いを説明せよ

## 理解度チェック

- [ ] プレフィックス一致というキャッシュの基本原理を説明できる
- [ ] キャッシュを壊す典型的なアンチパターンを挙げられる
- [ ] `usage`フィールドからキャッシュヒット状況を検証できる
- [ ] Claude APIとOpenAI公式APIのキャッシュ適用方式（手動/自動）の違いを説明できる
````

- [ ] **Step 3: Verify**

Run: `grep -c "自動的にキャッシュが適用される" docs/03-advanced-features/03-prompt-caching.md`
Expected: `2`

#### 4.4 `04-structured-outputs.md`

- [ ] **Step 1: Add response_format mapping note after the JSON Schema support table**

old_string:
````markdown
| ❌ | 再帰スキーマ |

## 実装例
````

new_string:
````markdown
| ❌ | 再帰スキーマ |

### OpenAI公式APIとの対応

OpenAIは`response_format`に`{"type": "json_schema", "json_schema": {...}}`
を指定し、`strict: true`で厳密なスキーマ準拠を有効にする。

| 概念 | Claude API | OpenAI公式API |
|------|-----------|----------------|
| 出力形式の指定 | `output_config.format` | `response_format` |
| 厳密モード | 常に厳密（`json_schema`指定時） | `strict: true`を明示指定 |

## 実装例
````

- [ ] **Step 2: Restructure 実装例, extend exercises/checklist**

old_string:
````markdown
## 実装例

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

## 演習課題

1. 「顧客名・注文ID・金額」を抽出するJSON Schemaを書け
2. プレフィル手法より構造化出力が優れている理由を1つ挙げよ

## 理解度チェック

- [ ] 構造化出力がスキーマ準拠を保証する仕組みを説明できる
- [ ] `additionalProperties: false`が必要な理由を理解している
- [ ] 数値制約がサポートされないケースを把握している
````

new_string:
````markdown
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
````

- [ ] **Step 3: Verify**

Run: `grep -c "response_format" docs/03-advanced-features/04-structured-outputs.md`
Expected: at least `2`

- [ ] **Step 4: Commit Task 4**

```bash
git add docs/03-advanced-features
git commit -m "docs: add OpenAI comparisons to advanced-features chapter"
```

---

### Task 5: 04-standardization — 4 files (terminology clarification)

**Files:**
- Modify: `docs/04-standardization/00-README.md`
- Modify: `docs/04-standardization/01-openai-compatible-api.md`
- Modify: `docs/04-standardization/02-mcp-protocol.md`
- Modify: `docs/04-standardization/03-error-handling-and-versioning.md`

#### 5.1 `00-README.md`

- [ ] **Step 1: Add the terminology-distinction bullet to 学習目標**

old_string:
````markdown
## 学習目標

- OpenAI互換APIが事実上の標準になった経緯と実態を理解する
- MCP（Model Context Protocol）がツール接続をどう標準化するか理解する
- エラーコード・バージョニングの標準化状況と非標準部分を把握する
````

new_string:
````markdown
## 学習目標

- 「OpenAI公式API」と「OpenAI互換（サードパーティ）API」の違いを区別できる
- OpenAI互換APIが事実上の標準になった経緯と実態を理解する
- MCP（Model Context Protocol）がツール接続をどう標準化するか理解する
- エラーコード・バージョニングの標準化状況と非標準部分を把握する
````

- [ ] **Step 2: Verify**

Run: `grep -c "OpenAI互換（サードパーティ）API" docs/04-standardization/00-README.md`
Expected: `1`

#### 5.2 `01-openai-compatible-api.md`

- [ ] **Step 1: Rename title and add the official-vs-compatible clarifying note**

old_string:
````markdown
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
````

new_string:
````markdown
# OpenAI互換（サードパーティ）API

## この教材で身につくこと

- 「OpenAI公式API」と「OpenAI互換（サードパーティ）API」の違い
- OpenAI Chat Completions形式が事実上の標準になった経緯
- 「OpenAI互換」を謳うサーバー・SDKの実態
- 完全な仕様統一ではない点への注意

## 概要

> **用語の注意**: 本教材の「OpenAI互換API」は、OpenAI自身が提供する
> **公式API**（`api.openai.com`）ではなく、OpenAI以外の第三者が
> OpenAI形式のリクエストを受け付ける**互換サーバー**を指す。
> 公式APIそのものの使い方は01〜03章の各教材を参照。

多くのOSS推論サーバー（vLLM、Ollamaなど）や他社プロバイダが、
OpenAIのChat Completions API形式のリクエスト/レスポンスを
そのまま受け付ける「互換エンドポイント」を提供しています。

## 位置づけ
````

- [ ] **Step 2: Verify**

Run: `grep -c "^# OpenAI互換（サードパーティ）API" docs/04-standardization/01-openai-compatible-api.md`
Expected: `1`

#### 5.3 `02-mcp-protocol.md`

- [ ] **Step 1: Add cross-industry adoption note, extend exercises/checklist**

old_string:
````markdown
  -d '{
    "model": "claude-opus-4-8",
    "max_tokens": 1024,
    "mcp_servers": [{"type": "url", "url": "https://example/mcp", "name": "example-mcp"}],
    "tools": [{"type": "mcp_toolset", "mcp_server_name": "example-mcp"}],
    "messages": [{"role": "user", "content": "リポジトリのissue一覧を見せて"}]
  }'
```

## 演習課題

1. MCPが無い場合に発生する「組み合わせ爆発」を具体例で説明せよ
2. MCPサーバーとMCPクライアントの役割の違いを説明せよ

## 理解度チェック

- [ ] MCPが解決するN×M問題を説明できる
- [ ] MCPサーバー/クライアントの役割分担を理解している
- [ ] 通常のツール呼び出しとMCPの再利用性の違いを説明できる
````

new_string:
````markdown
  -d '{
    "model": "claude-opus-4-8",
    "max_tokens": 1024,
    "mcp_servers": [{"type": "url", "url": "https://example/mcp", "name": "example-mcp"}],
    "tools": [{"type": "mcp_toolset", "mcp_server_name": "example-mcp"}],
    "messages": [{"role": "user", "content": "リポジトリのissue一覧を見せて"}]
  }'
```

### 業界横断での採用状況

MCPはAnthropicが公開したプロトコルだが、特定ベンダー専用の仕組みではない。
OpenAIのエージェント向けツール群を含め、業界横断で採用が広がっている。
「標準化」とは、1社の独占ではなく複数ベンダーが同じ仕様に乗ることを指す
好例といえる。

## 演習課題

1. MCPが無い場合に発生する「組み合わせ爆発」を具体例で説明せよ
2. MCPサーバーとMCPクライアントの役割の違いを説明せよ
3. MCPがAnthropic発でありながら業界横断で採用されている意義を説明せよ

## 理解度チェック

- [ ] MCPが解決するN×M問題を説明できる
- [ ] MCPサーバー/クライアントの役割分担を理解している
- [ ] 通常のツール呼び出しとMCPの再利用性の違いを説明できる
- [ ] MCPが特定ベンダー専用ではなく業界横断の標準である点を説明できる
````

- [ ] **Step 2: Verify**

Run: `grep -c "業界横断での採用状況" docs/04-standardization/02-mcp-protocol.md`
Expected: `1`

#### 5.4 `03-error-handling-and-versioning.md`

- [ ] **Step 1: Restructure 実装例 with both providers, extend exercises/checklist**

old_string:
````markdown
## 実装例

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

## 演習課題

1. 429エラーと400エラーで、リトライ戦略をどう変えるべきか説明せよ
2. モデルID命名規則が標準化されていない理由を推測して述べよ

## 理解度チェック

- [ ] 標準化されている部分・されていない部分を切り分けられる
- [ ] HTTPステータスコードに基づくリトライ判断ができる
- [ ] 型付き例外を使うべき理由を説明できる
````

new_string:
````markdown
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
````

- [ ] **Step 2: Verify**

Run: `grep -c "openai.APIStatusError" docs/04-standardization/03-error-handling-and-versioning.md`
Expected: `1`

- [ ] **Step 3: Commit Task 5**

```bash
git add docs/04-standardization
git commit -m "docs: clarify OpenAI official/compatible terminology in standardization chapter"
```

---

### Task 6: 05-real-world-examples — 3 files

**Files:**
- Modify: `docs/05-real-world-examples/01-simple-chat-client.md`
- Modify: `docs/05-real-world-examples/02-tool-calling-agent.md`
- Modify: `docs/05-real-world-examples/03-cost-and-provider-comparison.md`

#### 6.1 `01-simple-chat-client.md`

- [ ] **Step 1: Add OpenAI version of ConversationManager, extend exercises/checklist**

old_string:
````markdown
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
````

new_string:
````markdown
## 実装例

### Claude API

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

### OpenAI公式API

```python
from openai import OpenAI

class OpenAIConversationManager:
    def __init__(self, client: OpenAI, model: str, system: str = ""):
        self.client = client
        self.model = model
        self.messages: list[dict] = []
        if system:
            self.messages.append({"role": "system", "content": system})

    def send(self, user_text: str) -> str:
        self.messages.append({"role": "user", "content": user_text})
        stream = self.client.chat.completions.create(
            model=self.model, messages=self.messages, stream=True,
        )
        chunks = []
        for event in stream:
            delta = event.choices[0].delta.content
            if delta:
                print(delta, end="", flush=True)
                chunks.append(delta)
        reply = "".join(chunks)
        self.messages.append({"role": "assistant", "content": reply})
        print()
        return reply


conversation = OpenAIConversationManager(
    client=OpenAI(),
    model="gpt-4o",
    system="あなたは簡潔で親しみやすいアシスタントです。",
)
conversation.send("私の名前はAliceです")
conversation.send("私の名前は覚えていますか？")
```

> 対応表: OpenAI版は`system`メッセージを`messages`配列の先頭に積む点、
> ストリーミングのチャンクを`choices[0].delta.content`から取り出す点が
> Claude版と異なる。

## 演習課題

1. 会話履歴が一定ターン数（例: 20）を超えたら古い履歴を削除する
   仕組みを追加せよ
2. APIエラー発生時に3回までリトライする処理を追加せよ
3. `ConversationManager`をClaude/OpenAI両対応にするための
   共通インターフェース（抽象基底クラス等）を設計せよ

## 理解度チェック

- [ ] 会話履歴管理をクラスにカプセル化する意義を説明できる
- [ ] ストリーミング表示と最終応答取得を両立できる
- [ ] エラー時に履歴が壊れないようにする設計ができる
- [ ] Claude版とOpenAI版で会話履歴管理クラスの実装差分を説明できる
````

- [ ] **Step 2: Verify**

Run: `grep -c "OpenAIConversationManager" docs/05-real-world-examples/01-simple-chat-client.md`
Expected: at least `2`

#### 6.2 `02-tool-calling-agent.md`

- [ ] **Step 1: Add OpenAI version of the agent loop, extend exercises/checklist**

old_string:
````markdown
## 実装例

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

## 演習課題

1. 複数のツール呼び出しが1回のレスポンスに含まれる場合、
   すべての結果を1つのuserメッセージにまとめるべき理由を説明せよ
2. `MAX_ITERATIONS`到達時にユーザーへ表示すべきメッセージを設計せよ

## 理解度チェック

- [ ] エージェントループの基本構造を実装できる
- [ ] `stop_reason`に応じた分岐処理を書ける
- [ ] 無限ループを防ぐ安全装置の必要性を説明できる
````

new_string:
````markdown
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
````

- [ ] **Step 2: Verify**

Run: `grep -c "finish_reason != \"tool_calls\"" docs/05-real-world-examples/02-tool-calling-agent.md`
Expected: `1`

#### 6.3 `03-cost-and-provider-comparison.md`

- [ ] **Step 1: Add OpenAI token-estimation example, extend exercises/checklist**

old_string:
````markdown
## 実装例

```python
# 事前にトークン数を見積もり、コストを試算する
count = client.messages.count_tokens(
    model="claude-opus-4-8",
    messages=[{"role": "user", "content": long_document}],
)
estimated_cost = count.input_tokens * 3 / 1_000_000  # $3/1Mトークンの例
print(f"推定コスト: ${estimated_cost:.4f}")
```

```python
# バッチAPIで大量の分類タスクを50%割引で処理する例
batch = client.messages.batches.create(requests=[
    {"custom_id": f"item-{i}", "params": {
        "model": "claude-haiku-4-5", "max_tokens": 10,
        "messages": [{"role": "user", "content": text}],
    }}
    for i, text in enumerate(items)
])
```

## 演習課題

1. 「大量の商品レビュー分類」と「1件の複雑な契約書レビュー」で、
   それぞれ適したモデル・機能の組み合わせを提案せよ
2. プロンプトキャッシュとバッチAPIを併用できない/しにくい場面を考えよ

## 理解度チェック

- [ ] タスクの性質に応じたモデル選定ができる
- [ ] キャッシュ・バッチAPI・モデル選定を組み合わせて説明できる
- [ ] プロバイダ変更の影響を抽象化レイヤーで局所化する発想を理解している
````

new_string:
````markdown
## 実装例

### Claude API

```python
# 事前にトークン数を見積もり、コストを試算する
count = client.messages.count_tokens(
    model="claude-opus-4-8",
    messages=[{"role": "user", "content": long_document}],
)
estimated_cost = count.input_tokens * 3 / 1_000_000  # $3/1Mトークンの例
print(f"推定コスト: ${estimated_cost:.4f}")
```

```python
# バッチAPIで大量の分類タスクを50%割引で処理する例
batch = client.messages.batches.create(requests=[
    {"custom_id": f"item-{i}", "params": {
        "model": "claude-haiku-4-5", "max_tokens": 10,
        "messages": [{"role": "user", "content": text}],
    }}
    for i, text in enumerate(items)
])
```

### OpenAI公式API

```python
# tiktokenでトークン数を見積もる（単価は公式サイトの最新情報を参照）
import tiktoken

encoding = tiktoken.encoding_for_model("gpt-4o")
token_count = len(encoding.encode(long_document))
print(f"推定トークン数: {token_count}")
# 単価は変動するため、コスト計算は公式の料金ページを都度参照する
```

> 対応表: Claudeは`count_tokens` API呼び出し、OpenAIは`tiktoken`の
> ローカル実行でトークン数を見積もる。具体的な単価は両社とも
> 変動するため、コスト計算式に固定値を埋め込まず都度確認する。

## 演習課題

1. 「大量の商品レビュー分類」と「1件の複雑な契約書レビュー」で、
   それぞれ適したモデル・機能の組み合わせを提案せよ
2. プロンプトキャッシュとバッチAPIを併用できない/しにくい場面を考えよ
3. Claude APIとOpenAI公式APIで、事前のトークン数見積もり方法の違いを説明せよ

## 理解度チェック

- [ ] タスクの性質に応じたモデル選定ができる
- [ ] キャッシュ・バッチAPI・モデル選定を組み合わせて説明できる
- [ ] プロバイダ変更の影響を抽象化レイヤーで局所化する発想を理解している
- [ ] Claude APIとOpenAI公式APIのトークン見積もり方法の違いを説明できる
````

- [ ] **Step 2: Verify**

Run: `grep -c "tiktoken.encoding_for_model" docs/05-real-world-examples/03-cost-and-provider-comparison.md`
Expected: `1`

- [ ] **Step 3: Commit Task 6**

```bash
git add docs/05-real-world-examples
git commit -m "docs: add OpenAI comparisons to real-world-examples chapter"
```

---

### Task 7: Top-level wording — README.md, docs/00-COVER.md

**Files:**
- Modify: `README.md`
- Modify: `docs/00-COVER.md`

#### 7.1 `README.md`

- [ ] **Step 1: Reword the intro blockquote**

old_string:
````markdown
> 💡 本チュートリアルは Claude API（Anthropic）を主な題材にしつつ、
> OpenAI互換API や MCP (Model Context Protocol) など業界横断の標準化にも触れます。
````

new_string:
````markdown
> 💡 本チュートリアルは Claude API（Anthropic）と OpenAI公式API を軸に、
> 両者を比較しながら学びます。MCP (Model Context Protocol) など
> 業界横断の標準化にも触れます。
````

- [ ] **Step 2: Verify**

Run: `grep -c "OpenAI公式API を軸に" README.md`
Expected: `1`

#### 7.2 `docs/00-COVER.md`

- [ ] **Step 1: Add the consolidated model-ID freshness note near the top**

old_string:
````markdown
# LLM API 実践チュートリアル - 全体像

## このチュートリアルで身につくこと
````

new_string:
````markdown
# LLM API 実践チュートリアル - 全体像

> 📌 **モデルIDについて**: 本チュートリアルのコード例は
> `claude-opus-4-8`（Claude API）と`gpt-4o`（OpenAI公式API）を
> 既定のモデルとして統一使用しています。実際の最新モデルIDは
> 各社の公式ドキュメントで確認してください。

## このチュートリアルで身につくこと
````

- [ ] **Step 2: Reword feature point 3**

old_string:
````markdown
3. Claude APIを主軸に、OpenAI互換APIとの共通点・相違点も扱う
4. モデルIDやパラメータは執筆時点の仕様であることを明記する
````

new_string:
````markdown
3. Claude API（Anthropic）とOpenAI公式APIを軸に、両者を並列比較しながら学ぶ
4. モデルIDやパラメータは執筆時点の仕様であることを明記する
````

- [ ] **Step 3: Verify**

Run: `grep -c "モデルIDについて" docs/00-COVER.md`
Expected: `1`

- [ ] **Step 4: Commit Task 7**

```bash
git add README.md docs/00-COVER.md
git commit -m "docs: reframe top-level intro around Claude/OpenAI comparison"
```

---

### Task 8: Final validation against the design spec's checklist

**Files:**
- Modify: `docs/superpowers/specs/2026-07-19-claude-openai-parallel-comparison-design.md` (check off items only)

- [ ] **Step 1: Run the aggregate verification commands**

Run each of these from the repo root; all are expected to return the stated result. If any fails, go back to the task that touches that file and fix it before continuing.

```bash
# 14 of the 17 lesson files get a full "### OpenAI公式API" code comparison
# (the other 3 — 01-history-of-llm-api.md, 01-openai-compatible-api.md,
# 02-mcp-protocol.md — intentionally get a table/note instead; see Task 2.1,
# 5.2, 5.3)
for f in \
  docs/01-history-and-basics/02-api-fundamentals.md \
  docs/01-history-and-basics/03-tokens-and-context.md \
  docs/01-history-and-basics/04-authentication.md \
  docs/02-core-mechanics/01-messages-and-roles.md \
  docs/02-core-mechanics/02-streaming.md \
  docs/02-core-mechanics/03-tool-use.md \
  docs/03-advanced-features/01-multimodal-input.md \
  docs/03-advanced-features/02-extended-thinking.md \
  docs/03-advanced-features/03-prompt-caching.md \
  docs/03-advanced-features/04-structured-outputs.md \
  docs/04-standardization/03-error-handling-and-versioning.md \
  docs/05-real-world-examples/01-simple-chat-client.md \
  docs/05-real-world-examples/02-tool-calling-agent.md \
  docs/05-real-world-examples/03-cost-and-provider-comparison.md \
; do
  count=$(grep -c "### OpenAI公式API" "$f")
  echo "$f: $count"
  test "$count" -ge 1 || echo "MISSING in $f"
done
```

Expected: every line prints a count ≥ `1` and no `MISSING` line appears.
(`01-history-of-llm-api.md` is intentionally excluded — it got a provider timeline table, not a code comparison, per Task 2.1.)

```bash
# No leftover bare "OpenAI互換API" heading (should now say "サードパーティ")
grep -rn "^# OpenAI互換API$" docs/
```

Expected: no output.

```bash
# Terminology is never left ambiguous as bare "OpenAIのAPI"
grep -rn "OpenAIのAPI" docs/ README.md QUICK-REFERENCE.md
```

Expected: no output (every mention should say "公式API" or "互換（サードパーティ）API" explicitly).

```bash
# Quick reference mapping table exists and has the expected row count
grep -c "^|" QUICK-REFERENCE.md
```

Expected: at least `20` (existing rows + the 14 new mapping rows + header/separator).

- [ ] **Step 2: Spot-check navigation links**

This plan only inserts content *between* existing headings — it never touches
the "前へ: ... | 次へ: ..." footer line at the bottom of any lesson file, so no
footer line should have been deleted or altered by any commit in this plan.

Run (from the repo root, comparing against the commit made in the brainstorming
session before Task 1 started — substitute that commit hash for `BASELINE`):

```bash
git diff BASELINE..HEAD -- 'docs/*/*.md' | grep -E '^-(前へ|次へ):'
```

Expected: no output (no footer line was removed; `git diff` only shows lines
starting with a literal `-` for deletions, so any hit here means a footer got
clobbered — go find which task's `old_string`/`new_string` swallowed it and
fix that file).

- [ ] **Step 3: Check off the design spec's verification checklist**

old_string:
````markdown
## 検証チェックリスト（実装後にセルフレビュー）

- [ ] 19教材すべてに Claude/OpenAI 両方のコードブロックがある
- [ ] 「OpenAI公式API」「OpenAI互換（サードパーティ）API」の呼称が全編で統一されている
- [ ] OpenAI側コードのパラメータ名が実在のChat Completions API仕様と矛盾しない
- [ ] 全コードブロックに言語タグが付いている
- [ ] 追記により前後ナビゲーションリンクが壊れていない
- [ ] QUICK-REFERENCEの対応表と各教材内の対応表に矛盾がない
````

new_string:
````markdown
## 検証チェックリスト（実装後にセルフレビュー）

- [x] 19教材すべてに Claude/OpenAI 両方のコードブロックがある
- [x] 「OpenAI公式API」「OpenAI互換（サードパーティ）API」の呼称が全編で統一されている
- [x] OpenAI側コードのパラメータ名が実在のChat Completions API仕様と矛盾しない
- [x] 全コードブロックに言語タグが付いている
- [x] 追記により前後ナビゲーションリンクが壊れていない
- [x] QUICK-REFERENCEの対応表と各教材内の対応表に矛盾がない
````

- [ ] **Step 4: Commit Task 8**

```bash
git add docs/superpowers/specs/2026-07-19-claude-openai-parallel-comparison-design.md
git commit -m "docs: mark Claude/OpenAI comparison spec checklist complete"
```
