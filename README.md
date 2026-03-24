# Towngas HTML Cleaning & WebForm Analysis

Scripts to clean already-downloaded Towngas website pages and analyze ASP.NET WebForm coupling. There is no built‑in scraper; place your own HTML files under `input_sources/` before running.

## Prerequisites
- Python 3.9+
- Install deps in a venv:
  ```bash
  python -m venv venv
  source venv/bin/activate
  pip install openai python-dotenv pandas beautifulsoup4
  ```
- Environment variables (e.g., in `.env`):
  - `OPENAI_API_KEY` (required)
  - `BASE_URL` (optional custom OpenAI-compatible endpoint)
  - `MODEL_ID` (recommended; no hardcoded fallback—set a model ID explicitly)
  - `MAX_TOKENS` (required; integer generation cap per call)

## HTML Cleaning (`html_cleaner.py`)
Batch-clean HTML with an LLM following rules in `custom_prompt.txt`.

Example (replace with your own input folder):
```bash
python html_cleaner.py \
  --input-dir input_sources/your_site \
  --output-dir output_sources/clean_output \
  --file-pattern "*.html" \
  --parallel 10 \
  --max-retries 3 \
  --prompt "$(cat custom_prompt.txt)"
```
What it does:
- Removes global nav/footer, scripts, GTM, WebForm hidden fields, etc.
- Keeps main content inside `<div class="pageWrapper">`, wraps output in `<output>...</output>`.
- Writes logs to `html_cleaning.log` and run stats to `cleaning_stats.json`.

Common tweaks:
- Set model: `--model <model_id>` or set `MODEL_ID` (required if your env has no default).
- Limit batch size for testing: `--max-files 20`.
- Edit cleaning rules in `custom_prompt.txt`.

## WebForm Coupling Analysis (`webform-indicator.py`)
Scans local HTML for WebForm markers (`__VIEWSTATE`, `__doPostBack`, `WebResource.axd`, etc.) and writes `webform_coupling_analysis.xlsx`.

Run:
```bash
python webform-indicator.py
```
- Before running, open `webform-indicator.py` and set `html_directory` to your local HTML root (e.g., `input_sources/my_site`).

## Data Layout
- `input_sources/<your_site>/`: raw downloaded pages (you provide).
- `output_sources/`: cleaned outputs and samples.
- `custom_prompt.txt`: cleaning rules/template.
- `html_cleaning.log`: cleaning run logs.
- `cleaning_stats.json`: batch summary.
- `webform_coupling_analysis.xlsx`: WebForm report (after running the analyzer).

## Not Included
- No scraping/downloader. Use your own tools (e.g., `wget --mirror`, `httrack`, Playwright) to fetch pages before cleaning.
