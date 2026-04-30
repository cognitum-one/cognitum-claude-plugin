# ADR: PageRank Influence as Claude Code plugin

**Status**: Accepted
**Date**: 2026-04-29
**Cog**: `pagerank-influence`
**Category**: ai

## Context

A mesh of seeds in an open-plan office or a household sees co-occurrence events between people, devices, and rooms. "Who is the central node?" — the person whose presence most reliably precedes activity elsewhere — is a question raw counts answer badly because one chatty node distorts everything.

PageRank gives a stationary distribution over a directed graph that respects link structure rather than degree. Power iteration converges in 30–80 steps for graphs under 500 nodes, and per-step cost is the number of edges, both well inside the cog budget.

## Decision

Standalone armhf binary on Pi Zero 2 W. The cog consumes presence/co-occurrence events from the local event bus, maintains a sparse CSR adjacency matrix capped at 512 nodes / 4096 edges, and recomputes PageRank with damping d=0.85 each interval until the L1 delta < 1e-4.

State machine: **collecting (window edges) → iterating (power method) → ranked (publish top-K JSON) → idle**. Edges decay with half-life `decay_secs` so dropped relationships don't dominate the ranking forever.

## Consequences

### Positive
- Identifies non-obvious hubs that simple frequency counts hide.
- Sparse CSR + decay keeps memory bounded under unbounded event time.
- Output is a clean ranked JSON list other cogs can consume.

### Negative
- Power iteration is sensitive to disconnected components; isolated subgraphs report inflated rank.
- Edge decay is uniform; cannot weight relationships by event type without config.

### Neutral
- Damping factor 0.85 is conventional; configurable via `--damping`.

## Alternatives considered
- **Eigenvector centrality**: rejected — does not handle directed graphs cleanly and lacks the teleport term that protects against rank sinks.
- **Degree centrality**: rejected — measures volume, not influence.

## Plugin invocation
- `/pagerank-influence`
- `/pagerank-influence --once`
- `/pagerank-influence --console "top 5"`
- `/pagerank-influence --stop`
- `/pagerank-influence --logs`

## Resource budget
- Binary: 400-460 KB armhf
- RAM: < 2 MB
- CPU: < 5%

## See also
- Source: `cognitum-one/cogs:src/cogs/pagerank-influence/`
- ADR-001
- `pattern-sequence`, `federated-learning`
