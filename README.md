# RecallDSA

Search DSA lessons from **Striver** and **Padho with Pratyush** in English or Hinglish. RecallDSA finds useful transcript moments and lets you jump straight to the matching video timestamp.

<p align="center">
  <img src="frontend/src/assets/RecallDSA%20Searchable%20Video%20Hero.png" alt="RecallDSA searchable video interface" width="100%" />
</p>

## What it does

- Search indexed playlist captions with a natural-language question.
- Get a short answer grounded in transcript excerpts, with video sources and timestamps.
- Play a suggested video from the matching moment.
- Search in English or Hinglish.
- Add playlists to the knowledge base while keeping already indexed transcripts.

## Quick start

### Requirements

- Python 3.11 or later
- [uv](https://docs.astral.sh/uv/)
- Node.js and npm
- A Groq API key

### 1. Install dependencies

From the repository root:

```bash
uv sync
```

Create a `.env` file in the repository root:

```dotenv
GROQ_API_KEY=your_groq_api_key
QDRANT_MODE=local
```

Qdrant uses a local database under `data/qdrant` by default. Keep API keys in `.env`; do not put them in frontend environment variables or commit them.

### 2. Build the local vector index

The repository includes transcript chunks in `knowledge.json`. Index them into the local Qdrant database:

```bash
uv run python create_vector_db.py
```

The first run downloads the embedding model used by FastEmbed and creates the vector index. Later runs add missing chunks. To recreate the collection from scratch, use `--rebuild`.

### 3. Start the backend

In one terminal, from the repository root:

```bash
uv run uvicorn api:app --reload
```

The API runs at `http://127.0.0.1:8000`; interactive API docs are at `http://127.0.0.1:8000/docs`.

### 4. Start the frontend

In a second terminal:

```bash
cd frontend
npm ci
npm run dev
```

Open the Vite URL, usually `http://localhost:5173`.

## Add another playlist

Collect captions from a public YouTube playlist, then update the vector index:

```bash
uv run python create_knowledge.py "https://www.youtube.com/playlist?list=YOUR_PLAYLIST_ID"
uv run python create_vector_db.py
```

The collector keeps existing knowledge and skips videos already indexed. If YouTube blocks caption requests, rerun the same command later to retry videos that are still missing. Videos without accessible captions are skipped; RecallDSA does not transcribe audio. Use `--reset` only when you want to replace the knowledge file with the selected playlist.

## How search works

1. `create_knowledge.py` turns playlist captions into timestamped transcript chunks in `knowledge.json`.
2. `create_vector_db.py` embeds each chunk with `intfloat/multilingual-e5-small` and stores vectors and metadata in Qdrant.
3. The FastAPI backend retrieves relevant captions and asks Groq to answer using that context.
4. The frontend shows the answer and source videos; selecting a source opens the video at its timestamp.

## Deploy

For the Render API, Vercel frontend, and Qdrant Cloud setup, see [DEPLOYMENT.md](DEPLOYMENT.md).

## Stack

**Frontend:** Vite, JavaScript, CSS  
**API:** FastAPI, Groq  
**Retrieval:** Qdrant, FastEmbed, multilingual E5 embeddings  
**Transcript collection:** yt-dlp, YouTube Transcript API
