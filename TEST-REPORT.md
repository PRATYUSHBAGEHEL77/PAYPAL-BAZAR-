# Free Build — Local Verification Report

Tested in a local Python TestClient environment with a temporary SQLite database.

- Python compilation: PASS
- Frontend JavaScript syntax (`node --check`): PASS
- Homepage route: PASS (HTTP 200)
- Health endpoint: PASS (HTTP 200)
- Demo listing endpoint: PASS (HTTP 200)
- Local transcript parser for `I sell handmade soy candles for fifteen dollars each.`: PASS; extracted title and USD 15.00
- Listing creation in demo mode: PASS
- Local PNG share-card generation: PASS (HTTP 200, image/png)
- Inventory check-in: PASS
- Inventory check-out: PASS
- Insufficient-stock rejection: PASS (HTTP 409)

Not tested / not claimed as passing:
- Actual browser microphone permissions and Web Speech Recognition on every device/browser
- Render deployment itself
- PayPal Sandbox credentials, approval, capture, or webhook delivery
- Durable database persistence on Render Free (not provided by its ephemeral filesystem)

This is a free-mode demo build, not a production payment platform. Demo checkout is simulated and no money moves.
