# Stock Market Sentiment Analysis

Sentiment analysis of financial news headlines and its correlation with
short-term stock price movements, using **VADER** and **TextBlob** on
headlines scraped daily from **Finviz**. Apple Inc. (AAPL) is used as the
primary case study, with Amazon (AMZN), Alphabet (GOOG) and Walmart (WMT)
included for cross-stock comparison.

This repository accompanies the MSc dissertation *"Sentiment Analysis of
Stock Market Using Financial News Articles"* (Oxford Brookes University,
2024). The full report is in [`reports/MSc_Dissertation.pdf`](reports/MSc_Dissertation.pdf).

## Highlights

- **Automated daily scraping** of Finviz news headlines via GitHub Actions.
- **Two sentiment models compared** (VADER vs TextBlob) on the labelled
  Kaggle financial-news dataset — VADER reaches **55%** overall accuracy
  and is more balanced across positive/negative classes; TextBlob reaches
  **~58%** but heavily over-classifies neutral.
- **Correlation with price movements** using historical OHLC data from
  Yahoo Finance (`yfinance`), including the effect of the iPhone 16 launch
  on 20 September 2024.
- **Fully reproducible** — every figure in the dissertation can be
  regenerated from the notebook or by importing the modules in `src/`.

## Repository layout

```
stock-market-sentiment-analysis/
├── notebooks/
│   └── Stock_Market_Sentiment_Analysis.ipynb   # end-to-end walkthrough
├── src/
│   ├── scraper.py            # Finviz news scraper (also runs on a schedule)
│   ├── preprocessing.py      # Merge / clean / sort scraped CSVs
│   ├── sentiment_analysis.py # VADER + TextBlob scoring and evaluation
│   ├── stock_data.py         # yfinance price retrieval and joining helpers
│   ├── visualization.py      # All plots (distributions, heat maps, etc.)
│   └── utils.py              # Paths, thresholds, tickers
├── data/
│   ├── raw/                  # Daily scraped CSVs land here
│   ├── processed/            # Cleaned/merged output
│   └── Kaggle_dataset.csv    # Labelled dataset used for accuracy comparison
├── images/                   # Generated figures
├── reports/
│   └── MSc_Dissertation.pdf
├── results/                  # Model outputs (CSVs, reports)
├── requirements.txt
├── LICENSE
└── README.md
```

## Quick start

### 1. Clone and install

```bash
git clone https://github.com/Faizahmad-S/stock-market-sentiment-analysis.git
cd stock-market-sentiment-analysis
python -m venv .venv
source .venv/bin/activate    # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### 2. Run the scraper once

```bash
python -m src.scraper --tickers AAPL AMZN GOOG WMT
```

A timestamped CSV is written to `data/raw/`.

### 3. Analyse

Open the notebook to reproduce every figure end-to-end:

```bash
jupyter notebook notebooks/Stock_Market_Sentiment_Analysis.ipynb
```

Or drive the pipeline from Python:

```python
from src.preprocessing import preprocess
from src.sentiment_analysis import add_vader_columns, add_sentiment_category
from src.visualization import (
    plot_sentiment_distribution,
    plot_sentiment_heatmap,
    plot_ticker_sentiment_trend,
)

df = preprocess("data/raw")               # or a downloaded ZIP
df = add_vader_columns(df)
df = add_sentiment_category(df)

plot_sentiment_distribution(df, save_path="images/fig1_distribution.png")
plot_sentiment_heatmap(df,       save_path="images/fig8_heatmap.png")
plot_ticker_sentiment_trend(df, ticker="AAPL",
                            save_path="images/fig7_aapl_trend.png")
```

### 4. Evaluate against the labelled Kaggle dataset

```python
import pandas as pd
from src.sentiment_analysis import evaluate

data = pd.read_csv("data/Kaggle_dataset.csv", encoding="ISO-8859-1", header=None)
data.columns = ["True_Sentiment", "Headline"]

vader_result    = evaluate(data, "Headline", "True_Sentiment", kind="vader")
textblob_result = evaluate(data, "Headline", "True_Sentiment", kind="textblob")

print(vader_result)
print(textblob_result)
```

## Daily scraping via GitHub Actions

During the dissertation, the scraper ran on a daily GitHub Actions schedule
(11:00 UTC) and committed each new CSV back to the repository. The workflow
file is not included here, so the repository does not keep scraping. To
re-enable it, add `.github/workflows/scrape.yml`:

```yaml
name: Daily scrape
on:
  schedule:
    - cron: "0 11 * * *"
  workflow_dispatch:
jobs:
  scrape:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install -r requirements.txt
      - run: python -m src.scraper --tickers AAPL AMZN GOOG WMT
      - run: |
          git config user.name "github-actions"
          git config user.email "github-actions@users.noreply.github.com"
          git add data/raw
          git commit -m "Daily scrape" || echo "No changes"
          git push
```

## Results reproduced from the dissertation

| Model     | Overall accuracy | Best class | Weakest class |
|-----------|-----------------:|:-----------|:--------------|
| VADER     | 55.03 %          | Neutral    | Negative      |
| TextBlob  | 58.53 %          | Neutral    | Negative      |

VADER produced more balanced predictions across all three sentiment
classes, so it is used as the primary model for the Apple case study.
See §4.3 of the report for the full classification reports and
confusion matrices.

## Data sources

- **Headlines** — scraped from [Finviz](https://finviz.com/) (public news
  tables). No login required. Please respect Finviz's terms of use.
- **Prices** — retrieved via [`yfinance`](https://github.com/ranaroussi/yfinance),
  which wraps Yahoo Finance historical data.
- **Labelled sentiment corpus** — the "Sentiment Analysis for Financial
  News" dataset from Kaggle (`Kaggle_dataset.csv` in this repo).

## Limitations

- Both sentiment models are lexicon-based and general-purpose; they do not
  handle financial jargon, sarcasm or complex sentence structure well.
- Only headlines are scored, not article bodies.
- The scraping window used in the dissertation covers roughly two weeks,
  so the results describe short-term effects only.

## Future work

- Replace lexicon models with a domain-tuned transformer (e.g. **FinBERT**).
- Add more sources (Reuters, Bloomberg via RSS/APIs).
- Real-time streaming updates instead of daily batches.
- Predictive modelling of price movement from sentiment + price history.

## Citation

If you use this repository, please cite:

> Sangolli, F. (2024). *Sentiment Analysis of Stock Market Using Financial
> News Articles.* MSc Dissertation, Oxford Brookes University.

## License

Released under the [MIT License](LICENSE).
