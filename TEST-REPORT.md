# Verification report

## Passed locally
- `python -m py_compile app/main.py`
- `node --check` on the inline frontend JavaScript
- HTML DOM lookup audit: 43 JavaScript ID references, 0 missing IDs, 0 duplicate IDs
- FastAPI TestClient: `/`, `/api/health`, `/api/demo`
- Demo listing creation and status retrieval
- QR share card returns a valid PNG (Pillow fallback exercised because local Chromium was unavailable)
- Demo payment capture updates and persists status
- Inventory check-in and check-out persist; insufficient-stock checkout returns HTTP 409
- Listing data reloads from SQLite after simulated application reinitialization

## Not verified in this environment
- OpenAI transcription, extraction, translation, and TTS: requires a valid `OPENAI_API_KEY`
- PayPal order creation, buyer approval, capture, and webhook signature verification: requires valid PayPal Sandbox credentials and configured webhook
- Actual browser microphone permission and cross-device UI behavior: requires browser/device testing
- Playwright Chromium screenshot renderer: local Chromium binary was not installed; Pillow fallback was verified instead

This is an integration-tested demo build, not a guarantee of production readiness. Keep PayPal in Sandbox until the unverified items are tested.
