# PhishLens: explainable phishing and scam checker

**Flow:** text / URL / screenshot → (optional OCR) → indicator rules → risk score → explanation + safe actions → SQLite history.

## Run
```bash
# backend
cd backend && python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env        # optional: add LLM_API_KEY for AI-written explanations
pytest -q                   # 4 tests
uvicorn main:app --reload   # http://localhost:8000/docs

# frontend (new terminal)
cd frontend && npm install && npm run dev   # http://localhost:5173
```
Screenshot OCR is optional: `pip install pytesseract pillow` and install the Tesseract binary.

## Risk formula
`score = min(100, sum(weight of each triggered indicator))`, each indicator counts once.
Low < 30 ≤ Medium < 60 ≤ High. The LLM only rewrites the explanation from the detected signals; it can't change the score,
and if it is missing or fails, the built-in explanation is used. Message text is sent to the LLM as untrusted data (prompt-injection guard).

## API
- `POST /api/analyze` (form: `text`, `url`, `image`) → score, level, indicators, explanation, actions
- `GET /api/history`, `GET /api/history/{id}`

## Demo (3 minutes)
1. Click **Bank KYC scam** → High: urgency, OTP request, `.top` link, brand mismatch. Show the "Why this score" list.
2. Click **Normal message** → Low. Shows it doesn't cry wolf.
3. Paste only `http://paypa1.com/signin` in Link → look-alike + no https.
4. Open **Past checks** to show history.

## Honest limits / next steps
Rules catch common patterns, not every scam. Next: domain-age and blocklist lookups, a trained classifier next to the rules, multilingual rules.
