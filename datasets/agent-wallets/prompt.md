# agent-wallets — daily tick

You maintain a catalog of agent wallets, registries, and payment
protocols, plus the tool/service endpoints each one exposes. Schema and
scope in `datasets/agent-wallets/schema.yml` and
`datasets/agent-wallets/README.md`.

Tools:
- `frame-agent-wallets` — write to the dataset
- Frames registry — Exa for discovery, GitHub for repo metadata,
  vendor docs for endpoint lists

## Loop

1. **Read state.** Query
   `frame-agent-wallets_query` mode=sql:
   `SELECT entity_id, name, status, last_verified_at FROM rows
    ORDER BY COALESCE(last_verified_at, '1970-01-01') ASC`

2. **Cold-start discovery (only if dataset is empty).** Run a broad Exa
   sweep over the past 90 days for the seed terms below. Admit up to
   **5 entities** in this single first run, each with a strong primary
   source (vendor blog, GitHub release, official docs page):
   - `"agent wallet" announcement`
   - `"x402" registry`
   - `"agent commerce" payment`
   - `"machine payments" wallet`
   - `agentwallet`, `agent registry`, `payments for agents`
   Also explicitly check known starting points:
   - **agentwallet** (`frames-engineering`) — the wallet wired into this
     repo via `AGENTWALLET_USERNAME` / `AGENTWALLET_API_TOKEN`
   - **Frames Registry** (`registry.frames.ag`) — service catalog used
     by this repo
   - **x402** (Coinbase) — payment protocol
   For each admitted entity, set fields per step 4 below and stop.
   Do not run step 3 on a cold start.

3. **Discover (steady state).** Run **one** Exa pass over the past 30
   days searching for new wallets, registries, or payment protocols
   not already in the dataset. Bar for admission:
   - Primary source must be a vendor blog post, GitHub release,
     official docs page, or filed announcement (no aggregator posts).
   - The product must be **agent-targeted** — generic crypto wallets
     and stablecoin issuers don't qualify unless they ship an
     agent-specific surface.
   - **At most 2 new entities admitted per run.** Strongest first.
   `entity_id` format: `<vendor-slug>-<product-slug>`, collapsing when
   vendor and product are the same (e.g. `crossmint`, `skyfire`,
   `coinbase-x402`, `frames-engineering-agentwallet`).

4. **Refresh stalest 3.** Pick the 3 entities with the oldest
   `last_verified_at`. For each:
   - **Liveness**: fetch `api_base_url` via Exa. If it 404s or
     redirects to a generic landing page, search for a replacement
     URL. If no replacement and no announcement in 12+ months,
     `set_fact status=dead`.
   - **Services**: hit the registry endpoint (or scrape docs) and
     update the `services` field. Format each line as
     `<name> — <endpoint> — <one-sentence>`. Keep top **10**
     most-recently-confirmed; trim the rest (full history is in
     `events.ndjson`).
   - **Announcements**: search the vendor's blog/changelog/GitHub
     releases for material updates since `last_verified_at`. If
     anything notable, update `announcement_url` to the newest entry.
   - **Status**: move to `beta` → `active`, or to `deprecated` /
     `dead` only with a vendor statement or 12-month silence + dead
     URL. Status is sticky-forward.
   - **Always** `set_fact last_verified_at` to now (ISO 8601).

5. **Stop.** Print: cold-start (y/n), admitted count, refreshed count,
   status changes, dead URLs found.

## Constraints

- Every fact must cite a vendor-controlled source (their blog, GitHub
  release, official docs page, regulatory filing). No "according to a
  thread" or aggregator-only sources.
- Never invent endpoints. If a service's URL isn't on the vendor's
  docs page, omit it.
- `services` field stores the **public** registry — paid endpoints
  count, but private/customer-only ones don't.
- ≤ 2 admissions + ≤ 3 refreshes per run. Cold-start may admit 5.
- `name` and `entity_id` are immutable once admitted. Vendor renames
  are PR territory.

## Done when

- Cold-start ran (if applicable), OR ≤ 2 new admitted + ≤ 3 refreshed.
- Summary printed.
