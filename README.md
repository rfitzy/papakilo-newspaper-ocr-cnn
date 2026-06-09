# Hawaiian Newspaper OCR → Reference Audit → Translation

A Jupyter notebook pipeline that:

1. Loads a PDF of scanned Hawaiian-language newspaper pages,
2. Runs **Tesseract OCR** with the Hawaiian model (`haw`) to read what's on the page,
3. **Audits a reference transcription** you paste in against the page OCR — flagging where the reference may be wrong so you can correct it (the scanned page is the source of truth, not the reference),
4. Translates the corrected Hawaiian into English (free Google Translate by default; Claude optional).

## Setup

```bash
# 1. Install the Tesseract engine + Hawaiian model (system package, not pip)
#    macOS:
brew install tesseract tesseract-lang
#    Debian/Ubuntu:
# sudo apt-get install tesseract-ocr tesseract-ocr-haw

# 2. Create and activate a virtual environment (Python 3.9+)
python3 -m venv .venv
source .venv/bin/activate

# 3. Install the Python dependencies
pip install -r requirements.txt jupyter

# 4. (Optional) for Claude translation — better quality on archaic Hawaiian:
export ANTHROPIC_API_KEY=sk-ant-...

# 5. Launch
jupyter notebook hawaiian_newspaper_ocr.ipynb
```

Confirm the Hawaiian model installed: `tesseract --list-langs` should include `haw`.

## Inputs

- `newspaper.pdf` — your scanned newspaper (or change `PDF_PATH` in the notebook).
- **Reference transcription** — paste the text you want to check (e.g. from the Papakilo article
  page) into the `REFERENCE_TEXT` cell in Step 5. The notebook flags every place it disagrees with
  the page OCR so you can correct the reference. (Optional file fallback: `reference.txt`.)

## How the audit works

The page is the primary source. Neither the reference nor the OCR is assumed correct — the notebook
aligns them word-by-word and flags every discrepancy as a *candidate* to verify against the page
image. Because OCR is noisy, expect a mix of genuine reference errors and OCR slips; you adjudicate
(the optional Claude vision cell can read the actual page to help).

Typical loop: run the audit → review `discrepancies.txt` / `diff.html` against the image → fix the
reference → paste it back and re-run until only OCR-noise flags remain → translate.

## Outputs (in `output/`)

- `extracted_ocr.txt` — raw Tesseract OCR
- `discrepancies.txt` — numbered list of reference-vs-page disagreements
- `reference_annotated.txt` — the reference with each discrepancy marked inline
- `diff.html` — side-by-side visual diff (reference vs page OCR)
- `translation_en.txt` — English translation of the text you carried forward

## OCR notes

- Uses Tesseract's LSTM engine with `lang="haw"`. For mixed Hawaiian/English pages set `OCR_LANG = "haw+eng"`.
- If OCR is weak, raise `RENDER_DPI`, toggle `PREPROCESS`, or adjust `TESS_CONFIG` (`--psm` mode).

## Translation backends

- **Google Translate** (default) — free, no API key (needs internet); rougher on archaic Hawaiian. `TRANSLATION_BACKEND = "google"`.
- **Claude** (optional) — better on 19th-century / literary Hawaiian; needs `ANTHROPIC_API_KEY` and `TRANSLATION_BACKEND = "claude"` (uses `claude-opus-4-8`).
