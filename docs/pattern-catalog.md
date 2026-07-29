# CCAF Architecture Pattern Catalog

Eight production-tested patterns for Claude-powered systems. Each pattern includes the problem it solves, the architecture, CCAF domain mapping, and implementation guidance.

---

## 1. Sequential Pipeline with Async Handoff

**Problem**: Multi-agent workflows where each stage depends on the previous stage's output. Running them in parallel causes race conditions. Running them synchronously in one conversation burns context.

**Architecture**: Each agent runs independently on a schedule. Agents communicate via shared-state files (JSON or markdown), not conversation context. Agent A writes its output to a file. Agent B reads that file on its next run.

```
Agent 1 (6am) → writes .handoff.json
Agent 2 (5pm) → reads .handoff.json → writes .digest.json
Agent 3 (7:30am next day) → reads .digest.json → writes .ledger.json
Agent 4 (8am) → reads .ledger.json → writes to database
```

**Key rules**:
- Agents are loosely coupled. Each can fail independently without crashing the pipeline.
- Shared-state files have defined schemas. New fields are documented before agents populate them.
- Pipeline stages are ordered. You can draw the DAG.

**CCAF domains**: D1 (pipeline architecture), D5 (shared state as source of truth).

**Anti-pattern**: Putting all four stages in one agent prompt. Context window fills up, later stages get worse outputs, and a failure in stage 2 kills stages 3-4.

---

## 2. Two-Hop Security Boundary

**Problem**: Your CEO blocked automated access to a sensitive system (payment processor, banking API, credential store). But downstream agents need data from that system.

**Architecture**: Split into two hops with a human trigger at the security boundary.

```
Hop 1: Human triggers a snapshot (Claude Code with credentials)
  → Reads sensitive system
  → Writes sanitized output to a shared file

Hop 2: Automated agent reads the shared file
  → Processes data
  → Writes to downstream systems (database, dashboard)
```

**Key rules**:
- Hop 1 is human-triggered. No automated credential use.
- The shared file contains processed data, not raw credentials or PII.
- If the security policy changes later (e.g., MCP access approved), swap Hop 1 to a live query. No logic change in Hop 2.

**CCAF domains**: D3 (human-triggered at credential boundary), D2 (transport-agnostic design).

**Anti-pattern**: Embedding API keys in agent prompts to bypass the security boundary. One leaked conversation = compromised credentials.

---

## 3. Circuit Breaker for OAuth Dependencies

**Problem**: Scheduled agents depend on external services (Slack, Notion, CRM) via OAuth tokens. Tokens expire. Non-interactive scheduled sessions cannot re-authenticate. When tokens expire, the entire pipeline fails silently for weeks.

**Architecture**: Every agent that calls an OAuth-dependent service starts with a canary call. If auth fails, the agent enters degraded mode.

```
Normal mode:
  Agent → calls Slack API → posts digest → done

Degraded mode (auth expired):
  Agent → canary call fails
  Agent → writes output to local fallback file
  Agent → writes failure to .auth-status.json
  Agent → sends alert via separate channel (email, different service)
  Agent → stops Slack-dependent work, continues local work
```

**Key rules**:
- The canary call is the FIRST external call in every agent run.
- Alerts go through a SEPARATE auth boundary (e.g., email when Slack is down). If your alert channel uses the same auth as your failed channel, you've built a single point of failure.
- `.auth-status.json` tracks consecutive failures. A monitoring agent (e.g., standup deck) reads this and surfaces stale auth warnings.
- Recovery is human-triggered: open an interactive session (forces OAuth re-auth), then run a backfill command.

**CCAF domains**: D5 (silent failure is the worst pattern), D3 (human-triggered recovery), D1 (graceful degradation).

**Anti-pattern**: No canary check. Agent silently fails for 33 days. Nobody notices because the failure is silent. Data loss accumulates.

---

## 4. Human Gate with Tiered Autonomy

**Problem**: Some agent actions are safe to auto-execute. Others need human approval. Others need a specific senior person's approval. Treating all actions the same either creates bottlenecks (everything needs approval) or risks (nothing needs approval).

**Architecture**: Classify every agent action into one of three tiers.

```
Tier 1 (Auto-act): Agent executes without asking.
  Examples: filter personal emails, classify by keyword, update cache files.

Tier 2 (Recommend): Agent researches, writes recommendation, human confirms.
  Examples: qualify a sales lead, draft an outreach email, suggest content changes.

Tier 3 (Escalate): Agent flags for senior decision-maker review.
  Examples: pricing decisions, deals above a threshold, compliance-sensitive content.
```

**Key rules**:
- Tier classification is explicit in the agent's instructions, not emergent from prompt behavior.
- Human verdicts are logged (with timestamps) for audit trail.
- Agents cannot regress human decisions. If a human marked a lead as "Pass," an agent cannot reopen it. Agents can annotate ("new signal detected") but only humans change terminal states.
- Reviewer hierarchy is explicit: who supersedes whom on conflicts.

**CCAF domains**: D1 (tiered autonomy), D3 (human-triggered at decision boundaries), D5 (audit trail).

**Anti-pattern**: Putting tier logic in a single prompt with "if X then auto-act, if Y then ask." This is in-prompt branching. Separate tiers should be separate code paths.

---

## 5. Self-Healing Derived State

**Problem**: A dashboard or index needs to show "all active deals" or "all published files." If you maintain this list manually (add when created, remove when closed), it drifts. Deals get added directly to the database, the index doesn't know, and the dashboard shows stale data.

**Architecture**: The index is DERIVED, not maintained. A scheduled agent rebuilds it from scratch on every run by querying the source of truth.

```
Every Agent 4 run:
  1. Query database: filter by status IN (active statuses)
  2. Build complete index from query results
  3. Overwrite the index file

Result: manually-added items appear automatically on next run.
No sync logic. No "add to index when created." Just rebuild.
```

**Key rules**:
- One authoritative writer rebuilds the full index. Other agents or sessions may write courtesy updates between runs for immediate visibility, but the authoritative writer's next run self-heals any drift.
- Use structured database queries (deterministic, filter-based), not semantic search (probabilistic, false negatives).
- The rebuild is idempotent. Running it twice produces the same result.

**CCAF domains**: D1 (derived state > maintained state, single authoritative writer), D2 (query vs search tool selection), D5 (self-healing).

**Anti-pattern**: Maintaining the index by listening for events ("when a deal is created, add it to the index"). Event listeners miss events. Manual database edits bypass listeners. The index drifts silently.

---

## 6. Overwrite vs. Archive Tracks

**Problem**: Your scheduled agents generate files. Some should replace the previous version (this week's standup deck). Others should be preserved as historical records (each newsletter edition). Without this distinction, you get 47 date-stamped standup decks cluttering a folder.

**Architecture**: Every generated file follows one of two tracks.

```
Track 1: OVERWRITE (recurring outputs)
  - Single canonical filename, no date suffix
  - Each run overwrites the previous version
  - Examples: standup-deck-latest.html, dashboard.html, scorecard.html

Track 2: ARCHIVE (historical records)
  - Date-stamped filename, keep all editions
  - Each is a unique record worth preserving
  - Examples: weekly-report-2026-07-10.html, newsletter-edition-2026-07-14/
```

**Decision test**: "Will next week's version replace this one?" Yes = Track 1 (overwrite). No = Track 2 (archive).

**Key rules**:
- When overwriting a Track 1 file, check for and delete old date-suffixed versions of the same deliverable.
- Working drafts stay in a scratch folder. Don't copy to the workspace until the deliverable is final.
- Track 1 files have predictable names. Dashboards, monitoring tools, and bookmarks can point to a stable URL.

**CCAF domains**: D1 (file lifecycle architecture), D5 (cleanup prevents monotonic folder growth).

**Anti-pattern**: Every run creates `standup_deck_draft_YYYYMMDD.html`. After 3 months you have 60+ files. Nobody knows which is current. The folder becomes unusable.

---

## 7. Billboard Mode (Delivery-Context Overrides)

**Problem**: Your slide deck CSS works perfectly on a laptop screen but is unreadable when projected to a 250-person venue. Font sizes that look fine at 14" are tiny at 15 feet. Gray text that's readable on a Retina display washes out on a projector.

**Architecture**: Override CSS variables for the delivery context. Don't fork the stylesheet.

```
Base CSS (screen):
  --h1-size: 36px
  --text-color: #333333
  --text-secondary: #6e6e73

Billboard overrides (projection):
  --h1-size: 56px
  --text-color: #000000
  --text-secondary: #333333
```

**Key rules**:
- Scale up 30-40% for projected presentations. If content fills less than 60% of the canvas, elements are undersized.
- Replace gray text with a black scale for projection: #000000 (primary), #333333 (secondary), #555555 (tertiary). All pass WCAG AAA.
- Title anchors at the top of the slide. Content centers in the remaining space below. Wrapping both in a centered flex container pushes the title to mid-slide (readability failure at distance).
- One headline and one key visual per slide. Speaker does 80% of the communication. Slides are prompts, not scripts.

**CCAF domains**: D3 (config overrides for delivery context), D4 (output format matches delivery).

**Anti-pattern**: Creating a separate "presentation version" CSS file. Now you maintain two stylesheets. When the base changes, the presentation version drifts.

---

## 8. Two-Layer Read Pattern

**Problem**: You need to display "all active deals" in a dashboard. Semantic search (`search(query="active deals")`) seems obvious but returns false negatives: deals silently disappear from the dashboard because the search algorithm decided they weren't relevant enough.

**Architecture**: Separate "which items exist?" from "what are the details?"

```
Layer 1 (enumeration): Structured database query returns all item IDs
  → query-database(filter: {status IN [active statuses]})
  → Returns: list of page IDs + basic metadata

Layer 2 (detail): Fetch each item by known ID
  → get-page(id: known_page_id)
  → Returns: full item data including all properties
```

**Key rules**:
- Layer 1 uses a structured query (deterministic, filter-based). NEVER semantic search.
- Layer 2 fetches by known ID (zero ambiguity).
- A scheduled agent refreshes the Layer 1 index periodically (see Pattern 5: Self-Healing Derived State).
- The dashboard reads the index on open, then fetches live detail for each item. No semantic search in the hot path.

**CCAF domains**: D2 (structured query vs semantic search, tool selection), D5 (eliminating silent data loss from false negatives).

**Anti-pattern**: `search("active deals")`. Works 90% of the time. The 10% it misses are the deals you forget to follow up on. Silent data loss is the worst failure pattern.

---

## Quick Reference: Which Pattern When?

| Situation | Pattern |
|-----------|---------|
| Multi-agent workflow with ordered stages | 1. Sequential Pipeline |
| Sensitive system access needed by agents | 2. Two-Hop Security Boundary |
| Agents depend on OAuth services | 3. Circuit Breaker |
| Mix of auto/manual agent decisions | 4. Human Gate + Tiered Autonomy |
| Index or dashboard showing "all X" | 5. Self-Healing Derived State |
| Recurring file generation | 6. Overwrite vs Archive |
| Same content, different display contexts | 7. Billboard Mode |
| Need to enumerate items from a database | 8. Two-Layer Read |
