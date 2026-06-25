# News Intelligence Platform

A full-featured news research tool that aggregates articles from RSS feeds, GDELT, and Google News, stores them in a local SQLite database, and provides semantic search powered by OpenAI embeddings alongside real-time FinBERT sentiment scoring.

![Python](https://img.shields.io/badge/Python-3.9+-blue?logo=python)
![Streamlit](https://img.shields.io/badge/Streamlit-1.x-red?logo=streamlit)
![OpenAI](https://img.shields.io/badge/OpenAI-Embeddings-412991?logo=openai)
![SQLite](https://img.shields.io/badge/Storage-SQLite-003B57?logo=sqlite)
![License](https://img.shields.io/badge/License-MIT-green)

## What It Does

This platform collects news from multiple sources, persists articles to SQLite with WAL-mode for concurrent access, detects article language, precomputes FinBERT sentiment labels, generates OpenAI vector embeddings, and enables hybrid keyword-plus-semantic search across the stored corpus.

## Key Features

- Multi-source ingestion: RSS feeds (BBC, CNBC, MarketWatch, Reuters, Al Jazeera, Economic Times), GDELT, and GNews
- SQLite database with indexed tables for fast retrieval by date, language, and sentiment state
- Semantic search using OpenAI `text-embedding-3-small` with cosine similarity ranking
- LLM-assisted query expansion to improve search recall
- FinBERT sentiment scoring with calibrated probability output
- Language detection for filtering non-English articles
- Weighted sentiment index calculation across a filtered article set

## Technologies Used

| Layer | Technology |
|---|---|
| Language | Python 3.9+ |
| NLP Model | ProsusAI/FinBERT (Hugging Face) |
| Embeddings | OpenAI text-embedding-3-small |
| Database | SQLite (WAL mode) |
| News Sources | RSS, GDELT API, GNews |
| Dashboard | Streamlit |
| Visualization | Plotly Express |

## Getting Started

```bash
git clone https://github.com/Jaswanth8953/news-intelligence-platform.git
cd news-intelligence-platform
pip install -r requirements.txt
```

Create `.streamlit/secrets.toml` with your OpenAI key:
```toml
[openai]
api_key = "your-openai-api-key-here"
```

Or set the environment variable:
```bash
export OPENAI_API_KEY="your-openai-api-key-here"
```

Run the app:
```bash
streamlit run app.py
```

## Architecture

```
news-intelligence-platform/
├── app.py                  # Main Streamlit app and all logic
├── requirements.txt        # Dependencies
├── .streamlit/
│   └── secrets.toml        # API keys (not committed)
├── news_articles.db        # SQLite database (auto-created)
└── README.md
```

**Data flow:**
1. Fetch articles from RSS, GDELT, and GNews on demand
2. Deduplicate by URL and persist to SQLite
3. Background pass: detect language, generate embeddings, score sentiment
4. User query runs hybrid search (cosine similarity + keyword match)
5. Optional LLM query expansion via OpenAI chat completion
6. Results ranked and returned with sentiment overlay

## Configuration

Key constants in `app.py`:

| Constant | Default | Description |
|---|---|---|
| `MAX_ARTICLES_DEFAULT` | 150 | Max articles to display |
| `MIN_SIMILARITY` | 0.25 | Cosine similarity threshold |
| `MIN_RELEVANCE_SCORE` | 0.30 | Combined relevance cutoff |
| `GDELT_MAX_RECORDS` | 40 | Records pulled per GDELT request |

## Future Improvements

- Move background embedding and sentiment computation to async workers
- Add a trend dashboard showing sentiment shifts over time by topic
- Support custom RSS feed management via the UI
- Export to CSV or Excel with full article metadata
- Deploy as a containerized service on AWS EC2 or ECS

## License

MIT
