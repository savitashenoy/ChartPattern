# Vercel Deployment Notes

## What was fixed

### 1. `api/index.py` — Path resolution
`Path(__file__).resolve().parent.parent` correctly resolves to the project root
on both local and Vercel. The original code was correct here; the real issue was
that `ScannerData.xlsx` must be **committed to the repository** so Vercel bundles
it with the function.

### 2. `api/index.py` — Stateless export
Vercel spins up a fresh serverless instance for each request, so `scan_tasks`
(an in-memory dict) is always empty on the export call.
Fix: the `/api/export/<task_id>` endpoint now reads the `rows` array sent
directly in the POST body (the frontend already does this). No state needed.

### 3. `api/index.py` — Smaller download chunks
Changed `chunk_size` from 80 → 20 tickers per `yf.download()` call.
Vercel functions have a 60-second wall-clock limit; downloading 80 tickers
at once frequently exceeds it.

### 4. `vercel.json` — Function timeout
Added `"maxDuration": 60` for the Python function and `"maxLambdaSize": "50mb"`
to accommodate pandas/yfinance.

## Critical: commit ScannerData.xlsx
Vercel only ships files that are tracked by Git. Run:
```
git add ScannerData.xlsx
git commit -m "add scanner workbook"
git push
```
Without this step the `/api/sheets` endpoint will return a 500 error on Vercel.

## Environment variables (optional)
| Variable | Default | Purpose |
|---|---|---|
| `FLASK_SECRET_KEY` | `quant_scanner_dev_secret` | Flask session secret |
| `SCAN_TASK_TTL_SECONDS` | `3600` | In-memory task TTL |
