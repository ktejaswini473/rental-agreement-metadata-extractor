# Metadata Extraction from Rental Agreements (Template-Agnostic)

Extracts the following metadata from rental agreements, regardless of template:
- Aggrement Value
- Aggrement Start Date
- Aggrement End Date
- Renewal Notice (Days)
- Party One
- Party Two

Supported input:
- Scanned images: `.png`, `.jpg`, `.jpeg`
- `.docx`

## Non-rule-based approach
1) Convert document to text (OCR for images, docx text for docx)
2) Send the document text to an instruction-following LLM that returns **only JSON** with the target schema.

No regex/template rules are used to *find* values.

## Setup

### 1) System dependencies (OCR)
You must have Tesseract installed for image OCR.

**macOS**
- `brew install tesseract`

**Ubuntu/Debian**
- `sudo apt-get update && sudo apt-get install -y tesseract-ocr`

**Windows**
- Install “Tesseract OCR” and ensure `tesseract` is in PATH.

### 2) Python environment
```bash
python -m venv .venv
source .venv/bin/activate      # mac/linux
# .venv\Scripts\activate     # windows

pip install -r requirements.txt
```

### 3) Configure LLM (OpenAI-compatible)
This repo expects an OpenAI-compatible server:
- ChatCompletions endpoint: `{LLM_BASE_URL}/v1/chat/completions`

Set environment variables:
```bash
export LLM_API_KEY="YOUR_KEY"
export LLM_BASE_URL="https://api.openai.com"
export LLM_MODEL="gpt-4.1-mini"
```

(If you use another provider / self-hosted vLLM, set `LLM_BASE_URL` accordingly.)

## Run extraction
Example:
```bash
python -m src.extract --input_dir ./data/test --out predictions.csv
```

## Evaluate recall (exact match)
```bash
python -m src.evaluate --truth_csv ./data/test.csv --pred_csv predictions.csv
```

## REST API (optional)
```bash
uvicorn src.api:app --reload --port 8000
```

Then:
```bash
curl -X POST "http://127.0.0.1:8000/extract" -F "file=@/path/to/agreement.png"
```

## Notes for exact-match scoring
- Dates are returned in `DD.MM.YYYY` exactly as written.
- Names are returned as written (extra whitespace trimmed).
- Aggrement Value is digits only.
