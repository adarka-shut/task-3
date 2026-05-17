# Report — Task 3: Here There Be Dragons

## Project overview

NerdHug is an AI-powered personal learning assistant delivered as a Telegram bot. It helps users learn from web articles through intelligent summarization and interactive quiz generation. Built entirely on n8n workflow automation platform with no custom backend code.

**Bot:** [@nerdhug_tutor_bot](https://t.me/nerdhug_tutor_bot)

---

## Tools and techniques used

### Platform
- **n8n cloud** — workflow automation platform, free trial. All logic is implemented as a visual workflow with 40+ interconnected nodes.

### AI
- **n8n built-in AI Agent nodes** with GPT model — used for two distinct AI roles:
  - **Teacher agent** — analyzes article content and produces structured summaries with key points, main concepts, and difficulty assessment
  - **Examiner agent** — generates 5 multiple-choice questions with correct answers and explanations

### Data storage
- **Google Sheets** — two sheets (Materials and Quizzes) used for persistent data storage across sessions. Chosen for simplicity and visibility during debugging.

### Messaging
- **Telegram Bot API** — user interface via Telegram. Combination of native n8n Telegram nodes (for simple messages) and direct HTTP Request nodes to Telegram API (for inline keyboard buttons).

### Key n8n nodes used
- Telegram Trigger — catches incoming messages and callback queries
- Code nodes (JavaScript) — parsing commands, cleaning HTML, formatting responses, processing quiz answers
- AI Agent nodes — Teacher and Examiner roles
- Google Sheets nodes — read, append, update operations
- HTTP Request nodes — fetching article content and sending Telegram inline keyboards
- Switch/IF nodes — routing logic between commands and quiz states

---

## Architecture decisions

### Dual trigger approach
The Telegram Trigger node listens for both `message` and `callback_query` events. A single Code node (Parse Input) classifies every incoming event into one of five types: start, learn, quiz, select_material, or quiz_answer. A Switch node then routes to the appropriate branch.

### Inline keyboards via HTTP Request
The native n8n Telegram node does not reliably support dynamic inline keyboards. This is a known limitation documented in the n8n community. The solution is to use HTTP Request nodes that call the Telegram Bot API directly (`/sendMessage` with `reply_markup`), which gives full control over button layout and callback data.

### Answer history in callback data
The original design stored quiz progress in Google Sheets and read it back for each answer. This created a dependency on the Update Row operation, which proved unreliable with the sheet configuration. The final solution passes answer history directly in the callback_data of each button (e.g., `ans_SESSION_2_B_A,C`). This makes scoring completely independent of intermediate storage and ensures accurate results regardless of sheet update failures.

### Language detection
The AI prompts include an instruction to respond in the same language as the source material. If the article is in Russian, the summary and quiz are generated in Russian. This was not an original requirement but significantly improves usability.

---

## What worked well

- **n8n's visual workflow editor** made it easy to see the full flow and debug individual nodes by inspecting input/output data at each step.
- **Built-in AI tokens** in n8n cloud eliminated the need for a separate OpenAI subscription.
- **Google Sheets as storage** provided transparency — I could see exactly what data was being saved and debug issues by looking at the spreadsheet directly.
- **Telegram's inline keyboard buttons** create a smooth quiz experience — users just tap A/B/C/D without typing.
- **The Code node** (JavaScript) was flexible enough to handle all data transformation: HTML cleaning, JSON parsing, answer scoring, message formatting.

## What did not work well

- **Google Sheets OAuth2 permissions** — initially configured with restricted access ("only files created by n8n"), which prevented n8n from accessing the manually created spreadsheet. Took time to diagnose; fixed by reconnecting with full Google Sheets permissions.
- **n8n Telegram node inline keyboards** — the native node cannot handle dynamically generated inline keyboards reliably. Had to switch to raw HTTP Request nodes for all messages with buttons.
- **Google Sheets "No columns found"** — the node could not detect column headers when the sheet was empty or when using certain URL formats. Worked around by using "Map Automatically" mode and sheet ID instead of name where needed.
- **Markdown parsing in Telegram** — MarkdownV2 requires escaping special characters, which caused "Bad request" errors. Solved by switching to Parse Mode "None" for most messages.
- **Update Row reliability** — the Google Sheets Update Row operation failed to find the matching column, preventing intermediate quiz progress from being saved. Solved by carrying answer history in callback_data instead.
- **Imported workflow node versions** — some nodes imported from JSON had parameter structures that did not fully match the current n8n cloud version, requiring manual reconfiguration of several nodes.

---

## Summary

The final bot successfully implements all required features: three commands (/start, /learn, /quiz), real content extraction from URLs, structured AI-generated summaries, dynamic quiz generation with inline buttons, instant answer feedback with explanations, score calculation, persistent material storage, and topic selection for quizzes. The main challenges were around n8n's Telegram node limitations and Google Sheets integration quirks, both solved with practical workarounds.
