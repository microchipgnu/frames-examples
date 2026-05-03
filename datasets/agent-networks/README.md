# agent-networks

Open questions about the emerging agent / machine-network economy. One
entity per question — the question text is stable, but the surrounding
evidence and current synthesis update over time.

The dataset answers: *what's the live state of debate on each question,
who's taking which position, and what just shipped that moves the
needle?*

## In scope

- Discovery, identity, payments, reputation, network effects, ownership,
  and governance questions for agent-to-agent commerce and machine
  networks.
- Named actors with public positions or shipped products (Stripe, Exa,
  Parallel, MoltBook, agentwallet, x402, Polygon, etc.).
- Concrete recent signals: launches, partnerships, regulatory moves,
  funding events, technical milestones.

## Out of scope

- Generic AI commentary not tied to a network/economy claim.
- Speculation without a named actor or shipped artifact.
- Adding new questions ad-hoc. The canonical question list lives in
  `prompt.md`. New questions go in via PR, not via a tick.

## Refresh policy

Daily tick. Each run refreshes the **2–3 stalest questions** (those with
the oldest `last_reviewed_at`). Full coverage rotates over ~4 days.

Status transitions:

- `open` → `narrowing` once a small set of credible answers is in play
- `narrowing` → `resolved` once a default answer has emerged across the
  major actors
- any → `moot` if the question stops being load-bearing
