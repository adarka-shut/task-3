# NerdHug — AI Learning Assistant Telegram Bot

Warm hugs, cold facts. Your AI study buddy that reads articles and quizzes you on them.

**Bot link:** [https://t.me/nerdhug_tutor_bot](https://t.me/nerdhug_tutor_bot)

---

## How to use the bot

### Step 1 — Start the bot

Open the bot in Telegram and send `/start`. You will receive a welcome message with instructions.

### Step 2 — Learn from an article

Send `/learn` followed by a URL:

```
/learn https://example.com/some-article
```

The bot will:
- Fetch and extract text from the page
- Analyze it using the Teacher AI agent
- Send you a structured summary with title, difficulty level, main concepts, and 5–7 key points
- Offer a "Take Quiz Now" button

### Step 3 — Take a quiz

Send `/quiz` or tap the "Take Quiz Now" button after learning.

- If you have multiple saved materials, the bot will show a list of topics to choose from
- The Examiner AI agent generates 5 multiple-choice questions specific to the material
- Questions are sent one by one with A/B/C/D inline buttons
- After each answer you get instant feedback (correct/incorrect with explanation)
- At the end you receive a full score summary with per-question breakdown

### Commands

| Command | Description |
|---|---|
| `/start` | Get started with NerdHug |
| `/learn [URL]` | Analyze an article and create a summary |
| `/quiz` | Take a quiz on any saved material |

---

## How to set up your own instance

### Prerequisites

- n8n cloud account (free trial works)
- Telegram account
- Google account (for Google Sheets storage)

### Setup steps

1. **Create a Telegram bot** via @BotFather, get the API token
2. **Create a Google Sheets** spreadsheet with two sheets:
   - `Materials` — columns: materialId, userId, chatId, url, title, content, difficulty, summary, keyPoints, mainConcepts, addedDate
   - `Quizzes` — columns: quizSessionId, materialId, userId, chatId, title, questions, currentQuestion, answers, startedAt, status, scorePercent
3. **Import** `workflow.json` into n8n (three dots menu → Import from File)
4. **Configure credentials** in n8n: Telegram API (bot token) and Google Sheets OAuth2
5. **Update all nodes**: select the correct credential in every Telegram and Google Sheets node, paste the spreadsheet URL/ID
6. **Replace the bot token** in HTTP Request nodes (Send Summary with Quiz Button, Send Topic Selection, Send Next Question, Send First Question) — put your actual token in the URL
7. **Publish** the workflow

The bot is now live and ready to receive messages.
