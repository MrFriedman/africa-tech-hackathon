# Project: Rafiki SMS AI Gateway
Build single-file Python Flask backend `server.py` (port 5000) connecting TextBee SMS gateway + SQLite memory + Gemini API.

## Env Vars (.env)
- `GEMINI_API_KEY`
- `TEXTBEE_API_KEY`
- `TEXTBEE_DEVICE_ID`

## Dependencies
`flask`, `requests`, `google-genai`, `python-dotenv`

## Database (`rafiki.db`, SQLite)
Table `messages(id INTEGER PRIMARY KEY, phone TEXT, role TEXT, text TEXT, ts DATETIME DEFAULT CURRENT_TIMESTAMP)`
- Query last 6 rows per `phone` ordered ASC by `id`.
- Command `RESET` (case-insensitive): delete rows for `phone`, reply `"Memory cleared."`.

## Endpoints & APIs
1. `POST /webhook` (Incoming TextBee SMS):
   - JSON input: `{"sender": "+27...", "message": "..."}`
   - Ignore if `sender` or `message` empty (return 200).
   - Load history for `sender`, append user message, call Gemini, save user & model turns, send SMS reply, return `{"ok": true}`, 200.

2. Gemini Call (`google-genai` SDK):
   - Model: `gemini-2.5-flash`
   - System prompt: `"SMS AI assistant for rural Africa. Plain text only, no markdown, max 300 chars."`
   - Pass conversation history (`role`: `"user"` | `"model"`) + current prompt.

3. Outgoing TextBee SMS:
   - `POST https://api.textbee.dev/api/v1/gateway/devices/{TEXTBEE_DEVICE_ID}/send-sms`
   - Header: `x-api-key: {TEXTBEE_API_KEY}`
   - JSON: `{"recipients": [sender], "message": reply}`

## Code Rules
- Single file `server.py`, concise, no boilerplate comments, production-ready error handling.