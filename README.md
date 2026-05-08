# Blog Writing Agent

## What This Project Is
This project is an AI-powered blog generation system built with **LangGraph** and **Streamlit**.
It takes a topic, decides whether research is needed, creates a structured writing plan, writes section-by-section content, optionally generates technical images, and returns a complete Markdown blog post.

The repo includes:
- `bwa_backend.py`: LangGraph pipeline (routing, research, planning, writing, image flow)
- `bwa_frontend.py`: Streamlit UI for generating, previewing, and downloading blogs
- `requirements.txt`: Python dependencies
- `*_bwa_*.ipynb`: iterative development notebooks

## What It Does
- Accepts a blog topic and as-of date
- Chooses one of three modes:
  - `closed_book`: no web research needed
  - `hybrid`: mix of internal reasoning and web-backed claims
  - `open_book`: recent/news-style content with recency constraints
- Collects and normalizes evidence using Tavily (if research is needed)
- Builds a multi-section blog plan with goals, bullets, and target word counts
- Fans out section writing tasks to worker nodes
- Merges sections into one Markdown article
- Plans and generates up to 3 technical images
- Saves final Markdown and images locally
- Supports download of:
  - Markdown only
  - Markdown + images bundle (`.zip`)
  - Images only (`.zip`)

## How It Works (Architecture)
The backend graph follows this flow:

1. `router`
2. `research` (optional)
3. `orchestrator` (plan generation)
4. `worker` fanout (section writing)
5. `reducer` subgraph:
   - `merge_content`
   - `decide_images`
   - `generate_and_place_images`

### Routing Behavior
- `closed_book`: evergreen topics, no dependency on external sources
- `hybrid`: grounded examples with citations when needed
- `open_book`: volatile or latest topics, recency filtered evidence

### Research Behavior
- Uses Tavily via `langchain_community.tools.tavily_search`
- Deduplicates sources by URL
- Attempts to normalize dates
- Applies recency filter for `open_book`

### Image Behavior
- Image planning inserts placeholders like `[[IMAGE_1]]`
- Images are generated with Gemini image model (via `google-genai`)
- Images are stored per blog slug:
  - `images/<blog_slug>/<filename>`
- Placeholder replacement inserts Markdown image + caption
- If image generation fails, fallback content is inserted so output is still usable

## Inputs and Outputs
### Inputs
- Topic (required)
- As-of date (used for recency/routing context)
- API keys via `.env`

### Outputs
- `<blog_slug>.md` (saved in project root)
- `images/<blog_slug>/...` (if images were generated)
- Downloadable bundles from Streamlit UI

## Setup
From project root:

```bash
cd "/Users/abhinavgoyal/Documents/New project/blog-writing-agent"
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Create `.env`:

```env
OPENAI_API_KEY=your_openai_key
TAVILY_API_KEY=your_tavily_key
GOOGLE_API_KEY=your_google_key
```

Key usage:
- `OPENAI_API_KEY`: required for planning/writing/routing
- `TAVILY_API_KEY`: required for web research mode
- `GOOGLE_API_KEY`: required for image generation

## Run
```bash
streamlit run bwa_frontend.py
```

Open the local URL shown in terminal (usually `http://localhost:8501`).

## UI Overview
- **Generate New Blog** sidebar:
  - Topic input
  - As-of date
  - Generate button
- **Past blogs** sidebar:
  - Lists previously saved `.md` files
  - Loads selected blog into preview state
- Main tabs:
  - `Plan`: generated outline and tasks
  - `Evidence`: source table
  - `Markdown Preview`: rendered blog
  - `Images`: image plan and generated files
  - `Logs`: execution snapshots

## Design Notes
- Streamed graph execution is used to update UI progress
- Final output state is stored in Streamlit `session_state`
- Section merge order is stable (`task.id` sorted)
- Worker prompts enforce scope and citation constraints
- News-style mode avoids unsupported claims when evidence is weak

## Known Limits
- Output quality depends on model behavior and prompt compliance
- Research freshness depends on search availability and source quality
- Image generation depends on API quota/safety responses
- No lockfile is included yet (dependency versions are floating)


```

