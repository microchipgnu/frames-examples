# agent-networks — daily tick

You maintain a tracker of open questions on the agent / machine-network
economy. Schema and scope in `datasets/agent-networks/schema.yml` and
`datasets/agent-networks/README.md`.

Tools:
- `frame-agent-networks` — write to the dataset
- Frames registry — Exa for discovery and sourcing

## Loop

1. **Read state.** Query
   `frame-agent-networks_query` mode=sql:
   `SELECT entity_id, question, last_reviewed_at FROM rows ORDER BY
    COALESCE(last_reviewed_at, '1970-01-01') ASC`

2. **Seed if empty.** If the result is empty, seed the canonical 10
   questions from the list at the bottom of this prompt. For each, call
   `add_entity_with_facts`:
   - `entity_id`: the slug listed.
   - `source.url`: `https://github.com/microchipgnu/frames-examples`
     (the dataset's origin).
   - `source.retrieved_at`: now.
   - `facts`: `question`, `category`, `status: open`, and
     `last_reviewed_at` set to a timestamp in 1970 so first-tick refresh
     picks them up immediately.
   Then **stop** for this run — refresh in subsequent ticks.

3. **Pick targets.** Otherwise, take the **3 stalest** questions
   (oldest `last_reviewed_at`).

4. **Refresh each.** For each target question:
   - Run an Exa search keyed to the question's category and the past
     **30 days**. Filter for named actors and concrete signals
     (announcements, products, partnerships, regulation, funding).
   - Compare against the existing `recent_signals` (newline-joined). If
     anything new shows up, prepend up to 3 fresh entries in the format
     `<YYYY-MM-DD> — <one sentence> — <url>` and call
     `set_fact recent_signals` with the updated list (keep last 8 entries
     max — older context is in git history).
   - Update `key_actors` if a meaningfully new actor is now in play.
   - Update `current_thinking` (≤ 4 sentences) only if the new signals
     actually shift the picture. If nothing changed, leave it.
   - Update `tension` (1–2 sentences) — the core unresolved fork.
   - If a default answer has clearly emerged across major actors, move
     `status` from `open` → `narrowing`, or `narrowing` → `resolved`.
   - Always `set_fact last_reviewed_at` to now (ISO 8601).

5. **Stop.** Print: questions touched, status changes, new actors named,
   total signals added.

## Constraints

- Every fact must have a source URL on a real publication, vendor blog,
  GitHub release, regulatory filing, or filed announcement. No tweets
  unless the tweet is from an official company account announcing
  something material.
- Never invent a position. If a named actor's stance is inferred rather
  than stated, omit them.
- Do not delete signals. Trim the visible list to 8, but the full
  history lives in git via `events.ndjson`.
- Status transitions are sticky — only move `status` forward (or to
  `moot`); never back-revise without an explicit `attach_evidence` step.
- ≤ 3 questions touched per run.

## Done when

- The 3 chosen questions are refreshed (or seed step ran).
- Summary printed.

---

## Canonical question seed list

If the dataset is empty, seed these 10 entities (`entity_id` ↔
`question` ↔ `category`):

1. `network-effects-promiscuous-agents` — *Do traditional network
   effects survive when participants are infinitely promiscuous?* —
   `network-effects`
2. `who-owns-discovery` — *Who owns discovery? Parallel / Exa? Google?
   MoltBook / agent-native p2p network with something like DNS?* —
   `discovery`
3. `network-properties-machine-vs-human` — *Do agent / machine networks
   have the same properties as traditional human networks (increasing
   returns to scale, unassailable moat)?* — `network-effects`
4. `what-is-ownable` — *What is even ownable? Is there a concept of
   proprietary supply or demand when agents can join and leave millions
   of networks arbitrarily?* — `ownership`
5. `agents-as-economic-actors` — *Are agents semi-independent economic
   actors with a dependency on humans, or strict extensions of their
   operators?* — `economics`
6. `stripe-as-aggregator` — *Will Stripe (holder of human payment
   credentials and builder of the payments infra) become the aggregator
   of agent supply and demand?* — `payments`
7. `agent-acquisition-retention-churn` — *How should network operators
   think about agent acquisition, retention, and churn?* — `economics`
8. `agent-vs-web3-machine-networks` — *What are the similarities and
   differences between agent networks and web3 machine networks?* —
   `web3-comparison`
9. `who-handles-reputation-identity-fraud` — *Who handles reputation,
   identity, and fraud in an agent-to-agent economy?* — `reputation`
10. `micropayments-this-time` — *Will this time finally be different for
    micropayments on the internet?* — `payments`
