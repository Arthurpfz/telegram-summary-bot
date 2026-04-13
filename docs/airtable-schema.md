# Airtable Schema: Telegram Summaries

## Base Setup

**Base Name:** `Telegram Summaries`
**Table Name:** `Summaries`

---

## Field Definitions

| Field Name | Field Type | Description | Required |
|------------|------------|-------------|----------|
| `url` | URL | The original URL or hash for text content | Yes |
| `user_id` | Number | Telegram user ID who sent the content | Yes |
| `chat_id` | Number | Telegram chat ID for sending responses | Yes |
| `content_type` | Single Select | Type of content processed | Yes |
| `title` | Single Line Text | Extracted or generated title | No |
| `summary` | Long Text | The AI-generated summary | Yes |
| `original_content` | Long Text | Raw extracted content (optional) | No |
| `created_at` | Created Time | Automatic timestamp | Auto |
| `message_id` | Number | Original Telegram message ID | No |
| `token_count` | Number | Tokens used for summarization | No |

---

## Field Details

### url (Primary Key)
- **Type:** URL
- **Purpose:** Unique identifier for duplicate detection
- **For text content:** Store a hash of the first 100 characters
- **Example:** `https://youtube.com/watch?v=dQw4w9WgXcQ`

### user_id
- **Type:** Number (Integer)
- **Purpose:** Track which user submitted the content
- **Example:** `123456789`

### chat_id
- **Type:** Number (Integer)
- **Purpose:** Where to send the summary response
- **Example:** `123456789` (same as user_id for DMs)

### content_type
- **Type:** Single Select
- **Options:**
  - `youtube` - YouTube video transcripts
  - `article` - Web articles and blog posts
  - `text` - Direct text submissions

### title
- **Type:** Single Line Text
- **Purpose:** Display-friendly title
- **Max Length:** 100 characters
- **Example:** `How to Build an AI Chatbot`

### summary
- **Type:** Long Text
- **Purpose:** The formatted AI summary
- **Enable:** Rich text formatting
- **Example:**
```
**Title**: Building AI Chatbots Made Easy

**Overview**: This video covers the fundamentals...

**Key Points**:
1. Start with a clear use case
2. Choose the right model
...
```

### original_content
- **Type:** Long Text
- **Purpose:** Store raw transcript/article for debugging
- **Optional:** Can be disabled to save storage

### created_at
- **Type:** Created Time
- **Purpose:** Automatic timestamp
- **Format:** ISO 8601

### message_id
- **Type:** Number (Integer)
- **Purpose:** Reference to original Telegram message
- **Example:** `42`

### token_count
- **Type:** Number (Integer)
- **Purpose:** Track API usage for cost monitoring
- **Example:** `1250`

---

## Views

### Default View: All Summaries
- Show all fields
- Sort by `created_at` descending

### Recent (Last 24h)
- Filter: `created_at` is within past day
- Sort by `created_at` descending

### By User
- Group by `user_id`
- Sort by `created_at` descending

### YouTube Only
- Filter: `content_type` = `youtube`
- Sort by `created_at` descending

---

## Airtable Formula Examples

### Duplicate Check (for n8n)
```
{url} = 'https://example.com/article'
```

### Get User History (last 5)
```
AND(
  {user_id} = 123456789,
  DATETIME_DIFF(NOW(), {created_at}, 'days') <= 7
)
```

### Calculate Storage Used
```
LEN({summary}) + LEN({original_content})
```

---

## API Configuration in n8n

### Search for Duplicates
```
Operation: Search
Base ID: appXXXXXXXXXXXXXX
Table: Summaries
Formula: {url} = '{{ $json.url }}'
```

### Create New Record
```
Operation: Create
Base ID: appXXXXXXXXXXXXXX
Table: Summaries
Fields:
  - url: {{ $json.url }}
  - user_id: {{ $json.userId }}
  - chat_id: {{ $json.chatId }}
  - content_type: {{ $json.contentType }}
  - title: {{ $json.title }}
  - summary: {{ $json.summary }}
```

---

## Data Retention

Consider implementing automatic cleanup:

1. **Via Airtable Automations:**
   - Trigger: Every week
   - Action: Delete records where `created_at` > 30 days

2. **Via n8n Scheduled Workflow:**
   - Trigger: Weekly schedule
   - Action: Query old records and delete

---

## Privacy Considerations

- Store minimal PII (only Telegram user_id)
- Consider encrypting `original_content`
- Implement user data deletion on request
- Don't store actual Telegram usernames
