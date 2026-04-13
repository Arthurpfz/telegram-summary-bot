# Telegram Content Summarizer Bot - Setup Guide

## Prerequisites

Before setting up the n8n workflow, you'll need accounts and API keys from:

1. **n8n Cloud** - https://n8n.io
2. **Telegram Bot** - via @BotFather
3. **OpenRouter** - https://openrouter.ai
4. **Supadata.ai** - https://supadata.ai
5. **Airtable** - https://airtable.com

---

## Step 1: Create Telegram Bot

1. Open Telegram and search for **@BotFather**
2. Send `/newbot`
3. Follow the prompts:
   - Enter a name for your bot (e.g., "Content Summarizer")
   - Enter a username (must end in `bot`, e.g., `my_summarizer_bot`)
4. **Save the API token** - you'll need this for n8n

### Set Bot Commands (Optional)

Send these commands to @BotFather to configure your bot's menu:

```
/setcommands
```

Then select your bot and paste:

```
start - Get started with the bot
help - Show available features
history - View your last 5 summaries
```

---

## Step 2: Set Up Airtable

1. Go to https://airtable.com and sign in
2. Click **"Create a base"** → **"Start from scratch"**
3. Name it: `Telegram Summaries`
4. Rename the default table to: `Summaries`
5. Configure the columns (see `docs/airtable-schema.md` for details)

### Get Airtable Credentials

1. Go to https://airtable.com/create/tokens
2. Click **"Create new token"**
3. Name: `n8n-telegram-bot`
4. Scopes: Select `data.records:read` and `data.records:write`
5. Access: Add your "Telegram Summaries" base
6. **Save the token** - you'll need this for n8n

### Get Base ID

1. Open your base in Airtable
2. Look at the URL: `https://airtable.com/appXXXXXXXXXXXXXX/...`
3. The `appXXXXXXXXXXXXXX` part is your **Base ID**

---

## Step 3: Get OpenRouter API Key

1. Go to https://openrouter.ai
2. Sign up or log in
3. Navigate to **Keys** → https://openrouter.ai/keys
4. Click **"Create Key"**
5. Name it: `telegram-summarizer`
6. **Save the API key**

### Recommended Models

| Model | Cost | Best For |
|-------|------|----------|
| `openai/gpt-4o-mini` | $0.15/1M input | General summaries (recommended) |
| `openai/gpt-4o` | $2.50/1M input | Complex content |
| `anthropic/claude-3.5-sonnet` | $3/1M input | Nuanced analysis |

---

## Step 4: Get Supadata.ai API Key

1. Go to https://supadata.ai
2. Sign up for an account
3. Navigate to your dashboard
4. Copy your **API Key**

---

## Step 5: Set Up n8n Cloud

1. Go to https://n8n.io and sign up for n8n Cloud
2. Create a new workflow
3. Import the workflow from `workflow/telegram-summarizer.json`

### Add Credentials in n8n

Go to **Settings** → **Credentials** and add:

#### Telegram API
- **Name:** Telegram Bot
- **Access Token:** Your BotFather token

#### HTTP Header Auth (OpenRouter)
- **Name:** OpenRouter API
- **Header Name:** `Authorization`
- **Header Value:** `Bearer YOUR_OPENROUTER_KEY`

#### HTTP Header Auth (Supadata)
- **Name:** Supadata API
- **Header Name:** `x-api-key`
- **Header Value:** `YOUR_SUPADATA_KEY`

#### Airtable Personal Access Token
- **Name:** Airtable
- **Personal Access Token:** Your Airtable token

---

## Step 6: Configure Workflow

1. Open the imported workflow
2. Click each node that shows a warning (missing credentials)
3. Select the appropriate credential from the dropdown

### Nodes to Configure:

| Node | Credential |
|------|-----------|
| Telegram Trigger | Telegram Bot |
| Supadata API | Supadata API |
| OpenRouter LLM | OpenRouter API |
| Airtable Search | Airtable |
| Airtable Create | Airtable |
| Telegram Send | Telegram Bot |

---

## Step 7: Activate Workflow

1. Click the **"Inactive"** toggle in the top right
2. Set it to **"Active"**
3. n8n will automatically register the webhook with Telegram

---

## Step 8: Test Your Bot

1. Open Telegram and find your bot
2. Send `/start` - should receive welcome message
3. Send a YouTube link - should receive summary
4. Send an article URL - should receive summary
5. Send 500+ characters of text - should receive summary

---

## Troubleshooting

### Bot not responding
- Check if workflow is active in n8n
- Verify Telegram credentials are correct
- Check n8n execution logs for errors

### YouTube transcript fails
- Video may not have captions available
- Check Supadata.ai API usage limits
- Verify API key is correct

### Article extraction fails
- Some sites block scrapers
- Try a different article
- Jina Reader works best with standard news/blog sites

### Summary too long for Telegram
- Telegram has 4096 character limit
- The workflow truncates long summaries automatically

---

## Cost Estimation

| Service | Free Tier | Typical Cost |
|---------|-----------|--------------|
| n8n Cloud | 2,500 executions/month | $20/month (starter) |
| OpenRouter | Pay per use | ~$0.01-0.05/summary |
| Supadata | 100 requests/month | Pay as you go |
| Airtable | 1,000 records | Free for most use |

**Estimated cost for 100 summaries/month:** $5-10
