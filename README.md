# PerformOS Trending Dashboard - Phase A: Data Engine

## Overview

Phase A establishes the data engine and sample dataset for the PerformOS Trending Intelligence Dashboard. This includes the keyword-based auto-scoring system, a sample dataset of 30 realistic AI trending stories, and a specification for how manual human ratings feed back into the scoring weights.

---

## Data Schema

### Story Object (`data/sample.json`)

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique identifier (e.g., `story_001`) |
| `title` | string | Headline of the trending story |
| `summary` | string | 1-2 sentence description |
| `link` | string | Source URL |
| `topic_tags` | string[] | Tags from: `agents`, `llm`, `private_ai`, `local_models`, `automation`, `general_ai` |
| `auto_score` | number (1-100) | Automatically computed relevance score |
| `sentiment` | string | One of: `positive`, `neutral`, `negative` |
| `manual_score` | number or null | Human-assigned score, null if unrated |
| `rated_good` | boolean or null | Whether a human rated this as good, null if unrated |
| `rated_bad` | boolean or null | Whether a human rated this as bad, null if unrated |
| `scraped_at` | string (ISO 8601) | Timestamp when the story was scraped |

---

## Scoring System

The auto-scoring algorithm combines keyword matching with sentiment analysis to produce a score from 1 to 100.

### Algorithm

1. **Base Score**: Every story starts at **30 points**.
2. **Keyword Bonus**: The title and summary are scanned against the keyword weights in `scoring-weights.json`. Each matching keyword adds its weight. The total keyword bonus is **capped at 60 points**.
3. **Sentiment Adjustment**:
   - `positive` stories: **+5 points**
   - `neutral` stories: **+0 points**
   - `negative` stories: **-5 points**
4. **Topic Match Bonus**: If the story's topic tags overlap with the user's watchlist preferences, an additional **+5 points** is added.
5. **Clamping**: The final score is clamped to the range **[1, 100]**.

### Keyword Weights (`data/scoring-weights.json`)

Weights are designed to reflect relevance to the PerformOS audience. Core terms like `agent`, `agents`, and `llm` carry the highest weights. Totals sum to exactly 100.

---

## Manual Rating Feedback

Human reviewers can rate stories as good or bad via the dashboard UI. These ratings feed back into the scoring system to improve future auto-scores.

### Feedback Mechanism

1. **Adjustment Signal**: When a story is rated `rated_good` or `rated_bad`, the difference between `manual_score` and `auto_score` is computed.
2. **Weight Delta**: If a story scored lower than a human would expect (rated_good with high manual_score), the keywords that matched have their weights **increased by 0.5**. If a story scored too high (rated_bad with low manual_score), matched keyword weights are **decreased by 0.5**.
3. **Normalization**: After all adjustments in a batch, weights are re-normalized to sum back to 100.
4. **Decay**: Weight adjustments decay by 10% per week to prevent overfitting to recent ratings.

### Partial Rating

Some stories in the sample have `manual_score`, `rated_good`, and `rated_bad` set to `null`. These represent stories that have not yet been reviewed by a human rater. The dashboard should handle null fields gracefully.

---

## File Structure

```
data/
  sample.json            -- 30 trending story objects
  scoring-weights.json   -- keyword weights and scoring rules
README.md                -- this file
```

---

## Next Phases

- **Phase B**: Build the dashboard UI (HTML/CSS/JS) that displays stories sorted by score.
- **Phase C**: Implement the manual rating UI and feedback loop.
- **Phase D**: Connect to live scraping sources (Hacker News, Twitter/X, Reddit) for real-time ingestion.
