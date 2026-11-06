# PayPal Voice Bazaar — Complete Hackathon MVP

**Tagline:** Your Voice. Your Business. Your Payment.

A voice-first commerce demo for micro-sellers. A seller records one sentence, reviews an AI-generated listing, creates a PayPal Sandbox payment link, downloads a QR share card, and receives a live voice notification when payment is completed.

## Features
- Responsive single-page frontend (vanilla HTML/CSS/JS; no build step)
- Browser MediaRecorder audio capture
- OpenAI Whisper transcription with automatic language detection
- GPT-4o-mini strict JSON listing extraction and missing-price/product validation
- Seller confirmation and browser speech read-back in English, Spanish or Hindi (depending on installed browser voices)
- PayPal Orders v2 Sandbox checkout (`intent: CAPTURE`)
- QR share card rendered from HTML to PNG using Playwright
- SMS share URI and card download
- PayPal webhook signature verification for `PAYMENT.CAPTURE.COMPLETED`
- Server-Sent Events (SSE) live payment notifications, translated and spoken with OpenAI TTS
- `Try demo` works without mic or OpenAI key for the canned listing; if PayPal credentials are absent and `DEMO_MODE=true`, checkout is simulated locally.

## Project structure
```text
paypal-voice-bazaar/
├── app/
│   ├── main.py
│   └── static/
│       └── index.html
├── requirements.txt
├── .env.example
├── .gitignore
└── README.md
```

## Setup (Windows PowerShell)
```powershell
py -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
playwright install chromium
Copy-Item .env.example .env
```
Edit `.env`. Add your OpenAI API key and PayPal **Sandbox** credentials. Do not commit `.env`.

Start:
```powershell
uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```
Open http://127.0.0.1:8000

### macOS / Linux
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
playwright install chromium
cp .env.example .env
uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

## Environment variables
- `OPENAI_API_KEY`: Required for live audio transcription, AI listing extraction and translated TTS payment messages.
- `OPENAI_MODEL`: Defaults to `gpt-4o-mini`.
- `OPENAI_TTS_VOICE`: Defaults to `alloy`.
- `PAYPAL_CLIENT_ID`, `PAYPAL_SECRET`: PayPal Developer **Sandbox** REST app credentials.
- `PAYPAL_WEBHOOK_ID`: ID of the webhook configured in PayPal Developer Dashboard.
- `DEMO_MODE`: Defaults to `true`. If PayPal credentials are absent, app creates a local simulated checkout. Set `false` to require PayPal credentials.

## Demo without API credentials
1. Start the app.
2. Click **Try demo**.
3. Review the sample candle listing.
4. Confirm and create link.
5. Open checkout and click **Complete demo payment**.
6. Return to the seller page; the SSE notification appears. The local demo does not move money.

The listing preview is offline. Share-card generation still needs Playwright Chromium installed.

## Real PayPal Sandbox webhook
1. Create a REST app in the PayPal Developer Dashboard under **Sandbox**.
2. Put its Client ID and Secret in `.env`.
3. Run the app locally.
4. Expose port 8000:
   ```bash
   ngrok http 8000
   ```
5. Register `https://YOUR-NGROK-DOMAIN/api/webhook` as a webhook URL in the PayPal Developer Dashboard.
6. Subscribe to `PAYMENT.CAPTURE.COMPLETED`.
7. Copy the webhook ID into `PAYPAL_WEBHOOK_ID` in `.env`, then restart the app.
8. Use a Sandbox personal buyer account to approve the order. Back on the seller page, click **Complete demo / capture Sandbox order** to submit capture. The server waits for PayPal's verified webhook before sending the notification.

The backend verifies webhook signatures using PayPal's `verify-webhook-signature` endpoint. It does not trust unsigned event payloads.

## API endpoints
- `GET /` — frontend
- `GET /api/health` — configuration health
- `GET /api/demo` — canned transcript/listing response
- `POST /api/transcribe` — multipart audio field `audio`
- `POST /api/confirm` — JSON listing, creates PayPal Sandbox or demo order
- `GET /api/card/{listing_id}.png` — share card PNG
- `GET /demo-checkout/{listing_id}` — local simulated checkout
- `POST /demo-checkout/{listing_id}/complete` — complete local demo
- `POST /api/capture` — capture a PayPal order or complete local demo
- `POST /api/webhook` — verified PayPal webhook
- `GET /api/events/{listing_id}` — SSE notification stream
- `GET /api/status/{listing_id}` — order status

## 90-second presentation script
- **0–10s:** “Small sellers often don't have time or technical skills to build online listings. Voice Bazaar starts with one sentence.”
- **10–25s:** Click Try demo (or record a sentence). Show title, exact price and two-sentence description.
- **25–38s:** Click Read it aloud. Explain multilingual voice accessibility.
- **38–55s:** Confirm listing. Show PayPal checkout link and QR share card.
- **55–72s:** Open local demo checkout and complete demo payment, or use a PayPal Sandbox buyer account for the real payment flow.
- **72–90s:** Show the live payment notification and explain webhook verification and TTS.

## Important limitations
This is a hackathon MVP, not production payment software:
- In-memory data disappears when the server restarts and does not work across multiple workers.
- No authentication, seller account isolation, database, idempotency store, payment reconciliation dashboard, rate limiting, audit logging or retry queue.
- Real payment notifications require a correctly configured PayPal webhook and public HTTPS URL.
- Browser speech voices vary by device and language. OpenAI TTS is used for the completed-payment notification when the OpenAI key is configured.
- The local demo checkout is simulated and never transfers money.
- Before real users: add authentication, persistent storage, seller authorization, webhook event deduplication, CSRF/rate controls, privacy consent, monitoring, secure secret storage and full payment reconciliation.
