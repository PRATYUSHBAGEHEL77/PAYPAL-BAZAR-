# PayPal Voice Bazaar — Full-stack project

A voice-first storefront for independent makers. Sellers speak a product and price, review an AI-generated listing, create a PayPal checkout link, and share a QR-enabled product card. The app also tracks stock check-ins/check-outs in SQLite and displays live payment notifications.

## Included

- Responsive blue-and-white frontend with your supplied PayPal Voice Bazaar logo and transparent blue creator signatures.
- FastAPI backend with health, demo, transcription, listing confirmation, checkout, capture, share-card, inventory and webhook endpoints.
- SQLite database for listings, payment status, webhook event deduplication, inventory balances and inventory activity.
- OpenAI Whisper transcription, GPT listing extraction, translated payment notifications and text-to-speech.
- PayPal Orders v2 integration using Sandbox by default.
- Safe simulated checkout when PayPal credentials are not configured and `DEMO_MODE=true`.
- QR-code share cards and server-sent events for live seller updates.

## Run locally

Requires Python 3.11+.

```bash
python -m venv .venv
# Windows: .venv\\Scripts\\activate
# macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
playwright install chromium
cp .env.example .env
uvicorn app.main:app --reload
```

Open `http://127.0.0.1:8000`. The demo listing and simulated checkout work without API credentials. AI transcription/listing/TTS require `OPENAI_API_KEY`. Real PayPal Sandbox checkout requires `PAYPAL_CLIENT_ID` and `PAYPAL_SECRET`.

## PayPal setup

1. Create a REST app in the PayPal Developer Dashboard and copy its **Sandbox** client ID and secret into `.env`.
2. Keep `PAYPAL_ENV=sandbox` while testing.
3. Add a webhook at `https://YOUR-RENDER-DOMAIN/api/webhook` for `PAYMENT.CAPTURE.COMPLETED` and set `PAYPAL_WEBHOOK_ID` to the webhook ID.
4. Test with PayPal Sandbox buyer and seller accounts.
5. Only switch `PAYPAL_ENV=live` after reviewing PayPal's live integration requirements and adding live credentials. Never commit secrets.

## Deploy on Render

1. Upload the contents of this repository to the **root** of a GitHub repository. `render.yaml` must be at the repository root.
2. In Render choose **New + → Blueprint**, connect the repository and apply the Blueprint.
3. Add `OPENAI_API_KEY`, `PAYPAL_CLIENT_ID`, `PAYPAL_SECRET`, and `PAYPAL_WEBHOOK_ID` in the Render environment settings. Leave PayPal in Sandbox for testing.
4. The Blueprint mounts a persistent disk at `/var/data` for the SQLite database. Persistent disks are a paid Render feature; if your plan does not support disks, use a managed PostgreSQL database and migrate the persistence layer before relying on production data.
5. Wait for `/api/health` to report `ok: true`.

## API overview

- `GET /api/health` — service and integration configuration.
- `GET /api/demo` — sample listing (no API key needed).
- `POST /api/transcribe` — multipart audio upload; transcribe and extract listing.
- `POST /api/confirm` — create a PayPal order or safe demo checkout.
- `GET /demo-checkout/{listing_id}` and `POST /demo-checkout/{listing_id}/complete` — simulated checkout only.
- `GET /api/card/{listing_id}.png` — QR share card.
- `POST /api/capture` — capture an approved Sandbox order.
- `GET /api/events/{listing_id}` — SSE seller notification stream.
- `POST /api/webhook` — verified PayPal webhook receiver.
- `GET /api/inventory` and `POST /api/inventory` — persistent stock history and balances.
- `GET /api/status/{listing_id}` — current payment state.

## Important notes

- This is a hackathon-ready reference implementation, not a payment-platform certification or a production compliance guarantee. Test every flow in Sandbox before any live use.
- The UI's demand percentage cards are illustrative demo indicators, not live market research or a real-time marketplace feed.
- Inventory updates are persisted, and check-out is rejected if stock would become negative.
- SQLite works for a single app instance. For multi-instance scaling, use managed PostgreSQL and a shared event broker; in-process SSE listeners are local to the running process.
- Configure a persistent Render disk or external database to retain data across deploys/restarts. Do not rely on an ephemeral filesystem for business records.
- Keep API credentials in Render environment variables or local `.env`; never put them in frontend code or GitHub.
- Browser microphone access requires HTTPS (Render provides it) or localhost.

## Verification performed for this package revision

The following offline checks were run after repairing the missing `contextmanager` import and making static/database paths independent of the launch working directory:

- Python bytecode compilation for `app/main.py`.
- JavaScript syntax validation with `node --check`.
- DOM reference audit: all 43 JavaScript ID lookups resolve; no duplicate HTML IDs were found.
- FastAPI integration checks with credentials disabled: health, homepage, demo listing, listing creation, payment status, demo capture, QR card PNG generation, inventory check-in/check-out, insufficient-stock rejection, and listing persistence after reinitializing the database.

External OpenAI and PayPal transactions cannot be verified without valid API credentials. The browser microphone permission/recording flow and actual PayPal buyer approval/webhook delivery still need a live browser + Sandbox test. Do not enable live payments until those tests pass.
