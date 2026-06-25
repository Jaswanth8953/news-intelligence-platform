# News Intelligence Platform

A news research tool that pulls articles from RSS, GDELT, and GNews, stores them in SQLite, and lets you search across them using a combination of keyword matching and OpenAI vector embeddings. FinBERT runs in the background to score sentiment on each article.

![Python](https://img.shields.io/badge/Python-3.9+-blue?logo=python)
![Streamlit](https://img.shields.io/badge/Streamlit-1.x-red?logo=streamlit)
![OpenAI](https://img.shields.io/badge/OpenAI-Embeddings-412991?logo=openai)
![SQLite](https://img.shields.io/badge/Storage-SQLite-003B57?logo=sqlite)
![License](https://img.shields.io/badge/License-MIT-green)

## How it works

Articles come in from 8 RSS feeds (BBC, CNBC, MarketWatch, Reuters business, Reuters world, Reuters energy, Al Jazeera, Economic Times), GDELT, and GNews. Each one gets stored in a local SQLite database using WAL journal mode to handle concurrent reads/writes without locking.

In the background the app runs three passes over any new articles it hasn't processed yet:
1. Language detection via `langdetect` - filters out non-English articles
2. Embedding generation using `text-embedding-3-small` through the OpenAI API
3. FinBERT sentiment scoring (positive/negative/neutral with calibrated probabilities)

Search works by embedding your query, then computing cosine similarity against all stored article embeddings. There's also an optional LLM query expansion step that asks OpenAI to rewrite your query into a broader version before searching. Results are ranked by a weighted combination of cosine similarity and keyword overlap. Articles below `MIN_SIMILARITY = 0.25` and `MIN_RELEVANCE_SCORE = 0.30` get filtered out.

## Setup

```bash
git clone https://github.com/Jaswanth8953/news-intelligence-platform.git
cd news-intelligence-platform
pip install -r requirements.txt
```

You need an OpenAI API key. Add it to `.streamlit/secrets.toml`:
```toml
[openai]
api_key = "sk-..."
```

Or export it as an environment variable:
```bash
export OPENAI_API_KEY="sk-..."
```

Then:
```bash
streamlit run app.py
```

The SQLite database (`news_articles.db`) gets created automatically on first run.

## Stack

| What | How |
|---|---|
| News sources | RSS (feedparser), GDELT API, GNews |
| Storage | SQLite with WAL mode, indexed on published date, language, sentiment_computed |
| Embeddings | OpenAI text-embedding-3-small |
| Similarity | Cosine similarity (numpy dot product on normalized vectors) |
| Sentiment | ProsusAI/FinBERT via Hugging Face Transformers + PyTorch |
| Language detection | langdetect |
| Dashboard | Streamlit + Plotly |

## Config constants

| Name | Default | What it does |
|---|---|---|
| `MAX_ARTICLES_DEFAULT` | 150 | Max results to show in the UI |
| `MIN_SIMILARITY` | 0.25 | Drop results below this cosine score |
| `MIN_RELEVANCE_SCORE` | 0.30 | Combined threshold after weighting |
| `GDELT_MAX_RECORDS` | 40 | GDELT fetch limit per request |
| `GNEWS_MAX_RESULTS` | 40 | GNews fetch limit per request |

## What's missing / TODO

- Background workers so embedding and sentiment jobs don't block the UI thread
- Trend view showing how sentiment shifts for a given topic over time
- Export to Excel (currently CSV only via pandas)
- Docker setup for reproducible deployment

## License

MIT
