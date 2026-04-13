# Telegram Content Summarizer Bot

An n8n workflow that transforms a Telegram bot into an intelligent content curator. Send YouTube links, article URLs, or long text and receive structured AI-powered summaries.

## Features

- **YouTube Summarization** - Extracts transcripts and summarizes videos
- **Article Summarization** - Scrapes and summarizes web articles
- **Text Summarization** - Summarizes long text (500+ characters)
- **Duplicate Detection** - Returns cached summaries for repeated URLs
- **History Command** - View your last 5 summaries

## Architecture

```
Telegram → n8n Workflow → Content Extraction → LLM Summary → Airtable Cache → Telegram Response
```

| Component | Service |
|-----------|---------|
| Hosting | n8n Cloud |
| LLM | OpenRouter (GPT-4o-mini) |
| YouTube | Supadata.ai |
| Articles | Jina Reader |
| Database | Airtable |

## Quick Start

### 1. Get API Keys

| Service | URL | Free Tier |
|---------|-----|-----------|
| Telegram Bot | [@BotFather](https://t.me/botfather) | Unlimited |
| OpenRouter | https://openrouter.ai/keys | Pay per use |
| Supadata | https://supadata.ai | 100 req/month |
| Airtable | https://airtable.com/create/tokens | 1,000 records |
| Jina Reader | https://r.jina.ai | Free |

### 2. Set Up Airtable

Create a base with this schema (see `docs/airtable-schema.md` for details):

| Field | Type |
|-------|------|
| url | URL |
| user_id | Number |
| chat_id | Number |
| content_type | Single Select |
| title | Single Line Text |
| summary | Long Text |
| created_at | Created Time |

### 3. Import Workflow

1. Open n8n Cloud
2. Create new workflow
3. Click menu → Import from File
4. Select `workflow/telegram-summarizer.json`
5. Configure credentials (see `docs/setup-guide.md`)

### 4. Activate

Toggle the workflow to "Active" - n8n will register the Telegram webhook automatically.

## Usage

Send your bot:

| Input | Example |
|-------|---------|
| YouTube | `https://youtube.com/watch?v=...` |
| Article | `https://example.com/article` |
| Text | Any text over 500 characters |
| Help | `/help` or `/start` |
| History | `/history` |

## Output Format

```
**Title**: How to Build Better Habits

**Overview**: This video explains the science behind habit
formation and provides a 4-step framework for building
lasting behavioral changes.

**Key Points**:
1. Habits form through cue-routine-reward loops
2. Start with "atomic" habits that take 2 minutes or less
3. Environment design is more powerful than motivation
...

**Notable Insights**:
• "You don't rise to the level of your goals, you fall
  to the level of your systems"
• 1% daily improvement = 37x improvement over a year
```

## File Structure

```
telegram-summary-bot/
├── README.md
├── docs/
│   ├── setup-guide.md      # Detailed setup instructions
│   └── airtable-schema.md  # Database schema reference
├── workflow/
│   └── telegram-summarizer.json  # n8n workflow export
└── prompts/
    └── summarizer-system-prompt.txt  # LLM system prompt
```

## Configuration

### Change LLM Model

Edit the `Summarize (OpenRouter)` node and update the model:

```json
"model": "openai/gpt-4o"  // More capable, higher cost
"model": "anthropic/claude-3.5-sonnet"  // Alternative
"model": "openai/gpt-4o-mini"  // Default, cost-effective
```

### Adjust Summary Length

Edit the system prompt in `prompts/summarizer-system-prompt.txt` or directly in the OpenRouter node.

### Add Translation

Insert a translation node (DeepL or Google Translate) after the LLM summarization.

## Cost Estimation

| Usage | OpenRouter | Supadata | Total |
|-------|-----------|----------|-------|
| 50 summaries/month | ~$2-3 | Free tier | ~$3 |
| 200 summaries/month | ~$8-12 | ~$10 | ~$20 |

## Troubleshooting

**Bot not responding**
- Check workflow is active in n8n
- Verify Telegram credentials

**YouTube fails**
- Video may lack captions
- Check Supadata API limits

**Article extraction fails**
- Site may block scrapers
- Try a different URL

**Summary cut off**
- Telegram has 4096 char limit
- Content is auto-truncated

## License

MIT
