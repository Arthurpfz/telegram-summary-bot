# Telegram YouTube Summarizer Bot

## Project Overview
Telegram bot that takes a YouTube video URL, extracts the transcript, and replies with a detailed summary.

## n8n Workflow
- **Workflow ID**: `18pdrJhw7kadBACg`
- **Workflow name**: Telegram YouTube Summarizer
- **n8n instance**: Cloud (v2.4.8)
- **Status**: Active

## Architecture
```
Telegram Trigger → Send Typing → Get Transcript (Supadata) → Summarize Transcript (OpenRouter) → Split Message (Code) → Send Summary (Telegram)
                                        ↓ error                       ↓ error            ↘
                                   Send Error ←─────────────────── Send Error        Save to Airtable
```

## Nodes

### 1. Telegram Trigger (v1.1)
- Listens for incoming messages
- Credential: `Telegram account 2` (id: `q0azn03e68tE0j8L`)

### 2. Send Typing — Telegram (v1.2)
- Sends "typing..." indicator so user knows the bot is working
- Uses `sendChatAction` operation

### 3. Get Transcript — HTTP Request (v4.2)
- **API**: Supadata `GET https://api.supadata.ai/v1/transcript`
- **Auth**: Header Auth (`x-api-key`)
- **Query param**: `url` from incoming message text
- **Output**: `content[]` array with `.text`, `.offset`, `.duration`, `.lang`

### 4. Summarize Transcript — HTTP Request (v4.2)
- **API**: OpenRouter `POST https://openrouter.ai/api/v1/chat/completions`
- **Auth**: Header Auth (`Authorization: Bearer ...`)
- **Model**: `google/gemini-2.0-flash-001` (1M context, cheap, good for summarization)
- **Body**: Built with `JSON.stringify()` to handle special chars in transcript
- **Prompt style**: HTML format — `<b>` headers, • bullets, ◦ sub-bullets, `<i>` notable quotes. No markdown.
- **Telegram parse mode**: HTML (set in Send Summary node)

### 5. Save to Airtable — Airtable (v2.1)
- **Operation**: Create record (runs in parallel with Split Message)
- **Base**: SummaryYT (`appdSnWVVHJVXGH8C`)
- **Table**: Table 1 (`tblzS7WLfNceAeLFc`)
- **Fields**: URL (from Telegram message), Summary (from OpenRouter response), Date (auto-filled)
- **Credential**: `AirtableSummaryYT` (id: `8uTGhcCmR4nQgume`)

### 6. Split Message — Code (v2)
- Splits summary into chunks of max 4000 chars (Telegram limit is 4096)
- Converts `<br>` tags to newlines, strips unsupported HTML tags (only keeps `<b>`, `<i>`, `<u>`, `<s>`, `<a>`, `<code>`, `<pre>`)
- Splits on newlines to avoid breaking mid-sentence
- Adds `(1/3)` style numbering when multiple chunks
- Sanitizes HTML: closes unclosed `<b>` and `<i>` tags per chunk
- Outputs one item per chunk → Telegram Send node runs once per chunk

### 7. Send Summary — Telegram (v1.2)
- Sends each chunk from Split Message back to the original chat

### 8. Send Error — Telegram (v1.2)
- Receives error output from Get Transcript or Summarize Transcript
- Sends user-friendly error message explaining possible causes
- No parse mode (plain text)

## Credentials (configured in n8n)
| Credential | Type | Used by |
|---|---|---|
| Telegram account 2 | Telegram API | Trigger + Send Typing + Send Summary + Send Error |
| Header Auth (Supadata) | Header Auth, name=`x-api-key` | Get Transcript |
| Header Auth (OpenRouter) | Header Auth, name=`Authorization` | Summarize Transcript |
| AirtableSummaryYT | Airtable Personal Access Token | Save to Airtable |

## Cost
- **Model**: Gemini 2.0 Flash — $0.10/M input, $0.40/M output
- **1h video**: ~$0.003 (a third of a cent)
- **~300 videos per $1**
- Claude Sonnet comparison: ~$0.10/video (30x more expensive)

## Key Decisions
- **Gemini 2.0 Flash over Claude Sonnet**: ~30x cheaper, same quality for summarization tasks. Sonnet is overkill here.
- **JSON.stringify() for body**: Raw JSON templates break when transcript contains quotes/special chars.
- **Telegram Trigger v1.1**: v1.2 caused `execute` error on this n8n Cloud instance.
- **Error handling via error output**: Get Transcript and Summarize Transcript use `onError: continueErrorOutput` to route failures to a Send Error node.

## Known Issues & Fixes
- Header Auth credentials get swapped between nodes when doing full workflow updates via API — always verify which credential is assigned to which node after changes. Prefer partial updates.
- The Supadata API returns segments in `content[]` — we join them with `.map(item => item.text).join(' ')`.
- Raw JSON body breaks with transcript special chars — must use `JSON.stringify()` in expression.
- Telegram has a 4096 char limit — long summaries are split into numbered chunks via Code node.
- Full workflow updates via API change the Telegram Trigger webhook ID — must toggle workflow off/on in n8n UI to re-register the webhook after any full update.
- Telegram Markdown parser is too strict — switched to HTML parse mode instead. LLM outputs `<b>`/`<i>` tags which are much more forgiving.
- LLM sometimes outputs `<br>` tags which Telegram doesn't support — Split Message node strips unsupported HTML tags.
- Airtable resource mapper `columns.value` must contain the field expressions — partial updates can silently leave it as `{}` if not structured correctly.
- Code node JS via API: avoid arrow functions and template literals — escaping breaks them. Use plain `function` and string concatenation.

## Potential Improvements
- ~~Add error handling (reply with user-friendly message on failure)~~ ✅ Done
- ~~Add a "typing" indicator while processing~~ ✅ Done
- Handle non-YouTube URLs gracefully
- Support language selection for transcripts
- Add video title/metadata to the summary
- ~~Split long summaries if they exceed Telegram's 4096 char limit~~ ✅ Done
- ~~Save summaries to Airtable~~ ✅ Done
