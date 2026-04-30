# ADR: Local RAG Engine as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `rag-local`
**Category**: ai

## Context

Operators want to "ask their docs" — manuals, runbooks, prior incident reports — without uploading them to a cloud LLM. Retrieval-Augmented Generation does most of that job by retrieving the relevant chunks; the actual generation can happen on the host the user is already using (their laptop, a downstream LLM cog), so the seed only needs to be a competent retriever.

The seed already ships micro-hnsw (vector index) and a small int8 embedder via the `local-llm` feature flag. Wiring those together with a chunker and a query API is enough to deliver a useful RAG endpoint at a 14 KB cog footprint.

## Decision

Standalone armhf binary on Pi Zero 2 W. The cog watches `~/.cognitum/rag-local/docs/` for `.md` and `.txt` files, chunks each at 512 tokens with 64-token overlap, embeds chunks via the local int8 embedder (dim=128), and inserts them into the `micro-hnsw` cog over its socket. On `/query` the cog embeds the query, fetches top-K (default 8) neighbours, optionally re-ranks with a BM25 lexical pass over the same chunks, and returns chunk_id + text + score JSON.

State machine: **scanning → indexing → ready → reindexing (on file change)**. Chunk metadata persists as `chunks.cbor`; the actual vectors live in micro-hnsw.

## Consequences

### Positive
- No cloud round-trip; documents never leave the seed.
- Reuses micro-hnsw rather than carrying its own index — small binary.
- Hybrid (vector + BM25) re-rank improves precision on keyword-heavy queries with negligible CPU cost.

### Negative
- 4096-vector micro-hnsw cap caps the corpus at roughly 2 MB of plain text after chunking.
- Chunk boundaries are fixed-size, not semantic; a paragraph crossing a boundary may need both chunks.

### Neutral
- Generation is out of scope; downstream LLM cogs or operator tooling do that.

## Alternatives considered
- **In-process FAISS**: rejected — binary size and float32 cost.
- **Pure BM25**: rejected — semantic recall is the whole point of RAG.

## Plugin invocation
- `/rag-local`
- `/rag-local --once`
- `/rag-local --console "query <text>"`
- `/rag-local --stop`
- `/rag-local --logs`

## Resource budget
- Binary: 400-460 KB armhf
- RAM: < 2 MB
- CPU: < 5%

## See also
- Source: `cognitum-one/cogs:src/cogs/rag-local/`
- ADR-001
- `micro-hnsw`, `time-series-forecast`
