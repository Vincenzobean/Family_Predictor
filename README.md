# Family_Predictor
 This repository contains a rule-based predictor implemented in `Family Owned&Operated Predictor.ipynb` and Docker configuration to run it inside a container.

## What’s included
- `Family Owned&Operated Predictor.ipynb` — main notebook (analysis + scoring functions).
- `requirements.txt` — Python dependencies.

## Headless / production usage
- Convert the notebook to a script with `jupyter nbconvert --to script "Family Owned&Operated Predictor.ipynb"` and refactor into a CLI (`run_predictor.py`) for reliable headless runs.
- Update the `Dockerfile` `CMD` to run the script instead of Jupyter for automated pipelines.

## Selenium / Browser notes
The provided `Dockerfile` includes Chromium and `chromium-driver` so Selenium-based scraping should work. If you use a different browser or need a specific driver version, update the Dockerfile accordingly.

# Family Owned & Operated Predictor

This repository contains a rule-based predictor implemented in the notebook `Family Owned&Operated Predictor.ipynb`. The notebook scrapes company websites, extracts leadership and about-page signals, and computes a probabilistic score for whether a company is family-owned and family-operated.

What’s included
- `Family Owned&Operated Predictor.ipynb` — main notebook (scraping, feature extraction, scoring).
- `requirements.txt` — Python dependencies used by the notebook.
- `Dockerfile`, `start.sh`, `docker-compose.yml` — container setup for reproducible runs (Chromium + chromedriver included for Selenium).
- `tools/run_leak_test.py` — small helper script that executes the notebook cells and runs a quick score on sample records.

Quick start (local, notebook)
1. Create a Python environment and install dependencies:

```powershell
python -m venv .venv
.\.venv\Scripts\activate
pip install -r requirements.txt
```

2. Open the notebook and run cells in order (recommended using Jupyter Lab):

```powershell
jupyter lab
```

3. After running the pipeline, the notebook creates a variable `recs` (scored records). A final helper cell saves results to `family_owned_results.xlsx` in the notebook working directory.

Headless / Docker
- Build the image:

```powershell
docker build -t family-predictor:latest .
```

- Run interactively (mount repo, open Jupyter):

```powershell
docker run --rm -p 8888:8888 -v "${PWD}:/app" family-predictor:latest
```

- Or build & run with docker-compose:

```powershell
docker-compose up --build
```

Notes on usage
- The notebook is intentionally defensive: it prefers About-page signals for ownership, and applies an operation gate that uses leadership surname clustering to reduce false positives. Recent updates include improved detection for header-based name/title patterns (e.g., `<h3>NAME</h3>` followed by `<h4>TITLE</h4>`) and acceptance of ALL-CAPS names.
- The notebook writes an Excel file `family_owned_results.xlsx` when the final save cell runs. If you prefer CSV, open the last cell and replace `df.to_excel(...)` with `df.to_csv(...)`.
- For robust bulk runs, convert the notebook to a script and add a CLI wrapper. See the `Next steps` section below.

Testing the extractor locally
- Quick test using the provided helper script (executes notebook cells and scores sample inputs):

```powershell
python tools/run_leak_test.py
```

Recommended next steps
- Add a small CLI wrapper `run_predictor.py` that accepts a list of URLs and writes results to CSV/Excel.
- Add unit tests for `looks_like_name()`, `extract_leaders_from_text_blocks()`, and `leadership_cluster()` using representative HTML snippets (helps prevent regressions when improving heuristics).
- If you want me to, I can add the CLI wrapper and a GitHub Actions workflow to run the notebook/script on push.

Security & operational cautions
- Jupyter in the Docker image may be started without a token for convenience — do not expose it to public networks. Configure `NotebookApp.token` or use authentication/reverse-proxy for shared deployments.
- The scraper uses Selenium and visits remote sites; ensure you comply with each site's robots policies and terms of use.

Contact / ownership
- This project is maintained in this repository; open an issue or request if you'd like help tuning thresholds, adding tests, or exporting different output formats.
