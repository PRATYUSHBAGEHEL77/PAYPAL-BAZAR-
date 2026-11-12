# PayPal Voice Bazaar — Free Demo Build

A browser-based website (not an APK) with a FastAPI backend and SQLite database. This build is designed to run without OpenAI API credits.

## Free-mode behaviour

- Voice input uses the browser's built-in Web Speech Recognition when supported (Chrome/Edge support varies by platform and may use the browser vendor's speech service; it does not use your OpenAI API key).
- If speech recognition is unavailable, type a sentence into the free manual-entry field.
- Product name and price are extracted locally with deterministic rules. This is not an LLM and supports straightforward English phrases such as: `I sell handmade soy candles for fifteen dollars each.`
- Read-back uses the browser's built-in SpeechSynthesis.
- Demo checkout is simulated. It does not charge a buyer or move money.
- QR share cards are generated locally with Pillow and qrcode.
- Inventory check-in/check-out uses SQLite and prevents stock from going below zero.
- OpenAI and Playwright are not required for the free demo.

## Run locally

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate
# macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload
```

Open `http://127.0.0.1:8000`.

## Deploy on Render Free

1. Push the contents of this folder to the root of a GitHub repository.
2. In Render choose **New + → Blueprint**, connect the repository, and apply `render.yaml`.
3. Select the Free web service plan if prompted.
4. No OpenAI variables or PayPal credentials are required for the free demo.

### Important free-tier limitations

Render Free web services have an ephemeral filesystem and can spin down when idle. This build stores SQLite at `/tmp/voice_bazaar.db`, so listings/inventory may be lost after a restart, redeploy, or instance replacement. For durable production data, connect an external database with a free allowance or use a paid persistent disk/database plan. Do not use this ephemeral database for real customer records.

### Payments

Without PayPal Sandbox credentials, the app creates a clearly-labelled simulated checkout only. To test actual PayPal Sandbox payments, configure `PAYPAL_CLIENT_ID`, `PAYPAL_SECRET`, `PAYPAL_ENV=sandbox`, and a webhook ID. Sandbox testing does not charge real money, but PayPal account setup and availability apply. Never enable live payments without full security and end-to-end testing.

### Voice support

Browser speech recognition requires a supported browser and microphone permission, normally over HTTPS. If it is unavailable, use the manual sentence field. This free build does not promise offline speech recognition or AI-quality multilingual extraction.
