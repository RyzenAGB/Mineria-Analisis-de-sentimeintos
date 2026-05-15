# AGENTS.md — Sentiment Analysis (Amazon Phone Reviews)

## Project Status

Implemented. See `analisis_sentimientos.ipynb` for the complete analysis.

## What This Is

University final project: lexicon-based sentiment analysis (VADER) on ~91 MB CSV of Amazon phone/accessory reviews. Goal is to classify reviews as positive/negative/neutral and compare against star ratings.

## Dataset

- File: `Amazon-Reviews-Essential.csv` — columns: `Rating` (1–5), `Reviews` (text)
- Domain: mobile phones and accessories
- Language: English
- Large file (~91 MB) — uses a stratified sample (~50k rows) during development

## Stack

Python 3.x with: `pandas`, `nltk` (VADER + WordNetLemmatizer), `matplotlib`, `seaborn`, `wordcloud`, `scikit-learn`

Install: `pip install -r requirements.txt`

## Key Implementation Details

- VADER compound score thresholds: ≥0.05 = positive, ≤−0.05 = negative, else neutral
- Rating-to-sentiment mapping: 1–2 = negative, 3 = neutral, 4–5 = positive
- Custom domain lexicon (30 terms like `laggy`, `overheating`, `bricked`, `bloatware`, etc.) is defined in `implementation_plan.md` §2.2 — load it via `SentimentIntensityAnalyzer().lexicon.update(custom_dict)`
- Run analysis **before and after** lexicon enrichment to measure delta (Phase 3.2)
- Deliverable is a **Jupyter notebook** with step-by-step documentation, plus exported visualizations

## Conventions

- Text language is English (reviews are English)
- All prose/plan docs are in Spanish
- Keep development fast: sample first, full dataset last
- No test suite expected (this is an academic notebook project)

## Files Generated

- `analisis_sentimientos.ipynb` — Main notebook with all 6 phases
- `requirements.txt` — Python dependencies
- `01_distribucion_ratings.png` through `07_metricas_por_clase.png` — Visualizations
- `resultados_analisis_sentimientos.csv` — Full results export
