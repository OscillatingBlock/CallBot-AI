# **CALLBOT STUDIO – FULL API DOCS**  
*(WebSocket + REST Hybrid Architecture)*

---

## BASE URL
```
https://api.callbot.studio
```

---

## AUTHENTICATION
- **None (MVP)** – Public for testing  
- **Future**: API Key in header: `X-API-Key: your_key`

---

## 1. **GO SERVICE – BOT MANAGEMENT (REST)**

### `POST /bots`
**Create a new AI phone bot**

#### Request
```json
{
  "goal": "Take restaurant reservations",
  "webhook": "https://api.eatery.com/bot",
  "context": "Open 5PM–10PM. Tables for 2,4,6. No pets.",
  "voice": "elevenlabs:emma"
}
```

| Field | Type | Required | Description |
|------|------|----------|-----------|
| `goal` | string | Yes | What the bot should do |
| `webhook` | string | Yes | Your endpoint to receive user input |
| `context` | string | No | Business info, FAQs, rules |
| `voice` | string | No | `elevenlabs:emma`, `polly:joanna`, `playht:sarah` |

#### Response
```json
{
  "id": "abc123",
  "number": "+15551234567",
  "status": "active",
  "created_at": "2025-04-05T10:00:00Z"
}
```

---

### `GET /bots`
**List all your bots**

#### Response
```json
[
  {
    "id": "abc123",
    "number": "+15551234567",
    "goal": "Take restaurant reservations",
    "status": "active",
    "calls_today": 12
  }
]
```

---

### `GET /bots/{id}`
**Get bot details**

#### Response
```json
{
  "id": "abc123",
  "number": "+15551234567",
  "goal": "Take restaurant reservations",
  "webhook": "https://api.eatery.com/bot",
  "context": "...",
  "voice": "elevenlabs:emma",
  "created_at": "2025-04-05T10:00:00Z"
}
```

---

### `DELETE /bots/{id}`
**Delete bot & release number**

#### Response
```json
{ "message": "Bot deleted" }
```

---

## 2. **PYTHON SERVICE – CALL HANDLING (WebSocket)**

### `POST /voice`
**Twilio calls this when someone dials your bot**

> **You don’t call this directly** — Twilio does.

#### Query Params
| Param | Example |
|------|--------|
| `bot_id` | `abc123` |

#### Returns TwiML
```xml
<Response>
  <Connect>
    <Stream url="wss://api.callbot.studio/stream/abc123" />
  </Connect>
  <Say voice="Polly.Joanna">Please start speaking...</Say>
</Response>
```

---

### `wss://api.callbot.studio/stream/{bot_id}`
**Real-time voice stream**

#### Events (Bidirectional)

| Direction | Event | Payload |
|---------|-------|--------|
| **Twilio → You** | `connected` | `{"event": "connected", "streamSid": "..." }` |
| **Twilio → You** | `media` | `{"event": "media", "media": {"payload": "base64_audio"}}` |
| **You → Twilio** | `media` | `{"event": "media", "streamSid": "...", "media": {"payload": "base64_audio"}}` |
| **You → Twilio** | `stop` | `{"event": "stop"}` |

---

## 3. **USER WEBHOOK (YOUR SERVER)**

### `POST {your_webhook_url}`
**We call this on every user message**

#### Request
```json
{
  "bot_id": "abc123",
  "user_text": "Table for 4 at 7PM",
  "llm_suggestion": "Ask for name and phone",
  "history": [
    {"role": "user", "content": "Hi"},
    {"role": "bot", "content": "Hello! How can I help?"}
  ],
  "timestamp": "2025-04-05T10:05:00Z"
}
```

#### Response (Required)
```json
{
  "say": "Got it! Name please?",
  "action": "continue"
}
```

| Field | Type | Required | Options |
|------|------|----------|--------|
| `say` | string | Yes | What bot says next |
| `action` | string | No | `continue`, `hangup`, `transfer` |

---

## VOICE OPTIONS

| Provider | Voice ID | Example |
|--------|--------|--------|
| **ElevenLabs** | `elevenlabs:emma` | British female |
| **PlayHT** | `playht:sarah` | American female |
| **Twilio (Polly)** | `polly:joanna` | AWS Polly |

> Use full ID: `"voice": "elevenlabs:emma"`

---

## ERROR CODES

| Code | Meaning |
|-----|--------|
| `400` | Invalid request |
| `404` | Bot not found |
| `408` | Webhook timeout |
| `429` | Rate limit |
| `500` | Server error |

---

## WEBHOOK BEST PRACTICES

```python
# Example: Flask webhook
@app.route("/bot", methods=["POST"])
def bot_webhook():
    data = request.json
    user_text = data["user_text"]

    if "table" in user_text.lower():
        return jsonify({
            "say": "For how many people?",
            "action": "continue"
        })
    
    return jsonify({"say": "Sorry, I didn't get that."})
```

- **Respond in <3s**
- **Always return `say`**
- **Use HTTPS**
- **Handle duplicates** (Twilio retries)

---

## RATE LIMITS (MVP)

| Resource | Limit |
|--------|-------|
| Bots per user | 10 |
| Calls per bot/day | 100 |
| Webhook calls/sec | 5 |

---

## FULL EXAMPLE FLOW

```bash
# 1. Create bot
curl -X POST https://api.callbot.studio/bots \
  -H "Content-Type: application/json" \
  -d '{
    "goal": "Book dentist",
    "webhook": "https://clinic.com/api/callbot",
    "context": "Dr. Smith, Mon-Fri 9-5"
  }'

→ { "id": "dent123", "number": "+15559876543" }

# 2. Caller dials +15559876543
# 3. Twilio → POST /voice?bot_id=dent123
# 4. We open WebSocket
# 5. Real-time STT → LLM → Your webhook → TTS
```

---

## OPENAPI (Swagger) SPEC

```yaml
openapi: 3.0.0
info:
  title: CallBot Studio API
  version: 1.0.0
paths:
  /bots:
    post:
      summary: Create AI phone bot
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateBot'
      responses:
        '200':
          description: Bot created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Bot'
```

*(Full spec available on request)*

---

## REPLY: `SEND API DOCS + OPENAPI`

I’ll send you:

1. `openapi.yaml` – Full Swagger spec  
2. `postman-collection.json` – Ready to import  
3. `webhook-examples/` – Python, Node, PHP  
4. `curl-commands.sh` – Test everything  
5. `swagger-ui/` – Host your own docs  

---
