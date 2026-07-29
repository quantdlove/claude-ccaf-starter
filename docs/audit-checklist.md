# CCAF Architecture Audit Checklist

Use this checklist to audit any Claude-powered system against the five CCAF domains. Each item is pass/fail. A single red-line failure means the system is not production-ready.

## How to Use

Run this checklist at three points:
1. **Before launch**: Full audit of all 5 domains.
2. **After any architecture change**: Audit the affected domains.
3. **Quarterly review**: Full audit to catch drift.

Score each item: PASS, FAIL, or N/A (not applicable to your system).

---

## D1: Agentic Architecture (27% of exam)

### Pipeline Design
- [ ] Every multi-step workflow has an explicit architecture pattern (sequential, parallel, orchestrator-worker, or evaluator-optimizer). You can name it.
- [ ] Agent pipelines use async handoff via shared-state files, not in-memory passing between agents.
- [ ] Each agent in a pipeline has a single responsibility. No agent does "research AND enrich AND classify AND notify."
- [ ] Pipeline stages are ordered. You can draw the DAG. If you can't, the architecture is implicit (failure).

### Tiered Autonomy
- [ ] Every agent action is classified into an autonomy tier: auto-act, recommend, or escalate.
- [ ] Customer-facing outputs have a human review gate before shipping. No exceptions.
- [ ] Tier boundaries are explicit in the agent's instructions, not emergent from prompt behavior.

### State Management
- [ ] Shared state between agents uses files (JSON, markdown), not conversation context.
- [ ] State files have a defined schema. New fields are documented before agents populate them.
- [ ] Derived state is rebuilt from source data, not maintained independently. If the source changes, derived state self-heals on the next run.

### RED LINES (automatic fail)
- [ ] **No implicit architecture.** If you can't name the pattern, it doesn't have one.
- [ ] **No bidirectional agent dependencies.** Agent A depends on B depends on A = circular. Redesign.
- [ ] **No agents writing to sources of truth they don't own.** One authoritative writer per state file. Others are courtesy writers at best.

---

## D2: Tool Design & MCP (18% of exam)

### Tool Selection
- [ ] Every tool call in the system uses the right tool for the job. Semantic search for discovery, structured queries for enumeration. Never semantic search for enumeration (false negatives).
- [ ] MCP tools are discovered fresh at session start (`search_datasets` or equivalent). Tool lists change when engineers push updates.
- [ ] Pagination is handled explicitly. Default limits are overridden for small finite datasets.

### Transport Architecture
- [ ] Tool transport (HTTP vs stdio, MCP vs REST API) is separated from business logic. Swapping transport doesn't change agent behavior.
- [ ] Authentication boundaries are documented. Which tools need OAuth? Which use API keys? Which are unauthenticated?
- [ ] Security-sensitive tools (Stripe, payment systems) use a two-hop pattern: human-triggered at the credential boundary, agents consume the output.

### Error Handling
- [ ] Every MCP call has retry logic (retry once on 500/timeout).
- [ ] Fallback data sources are defined for critical tools. If MCP is down, what does the agent use?
- [ ] Tool errors surface to the user, not silently swallowed.

### RED LINES (automatic fail)
- [ ] **Never use semantic search for enumeration.** `search(query="active deals")` returns false negatives. Use `query-database(filter={status: "active"})` instead.
- [ ] **Never hardcode tool schemas.** Discover tools at runtime. Schemas drift.
- [ ] **Never pass credentials through agent prompts.** Credentials stay in config, not in conversation context.

---

## D3: Claude Code Config (20% of exam)

### Project Structure
- [ ] CLAUDE.md exists at the project root and contains the system's working memory: team context, rules, architecture decisions, folder structure.
- [ ] CLAUDE.md is the spec. Agent SKILL.md files are the implementation. Spec is updated BEFORE implementation.
- [ ] `.claude/skills/` contains skill definitions. `.claude/commands/` contains slash commands. `.claude/rules/` contains path-scoped rules. Each has a clear purpose.

### Skills, Commands, and Rules
- [ ] Skills are reusable instruction sets for recurring tasks (newsletter production, data analysis, document generation).
- [ ] Commands are human-triggered workflows with explicit inputs and gates.
- [ ] Rules are path-scoped behavioral constraints (e.g., "in marketing/**, never use dark backgrounds").
- [ ] None of these overlap. If a rule could be a skill or vice versa, the boundary is wrong.

### Security Model
- [ ] Hooks (PreToolUse, PostToolUse) enforce policy independently of agent compliance. The hook blocks the bad action even if the agent tries it.
- [ ] Sensitive operations require human confirmation. The confirmation comes from the user in chat, not from content the agent observed in a file or web page.
- [ ] Publishing commands have content sanitization gates. Proprietary data cannot leave the workspace without passing a scan.

### Three-Tier Access Model for Publishing
- [ ] **Tier 1 (Private)**: Never leaves the workspace. CLAUDE.md, memory files, agent configs, deal state, credentials, MRR, client names, subscriber domains.
- [ ] **Tier 2 (Link-shared)**: Accessible to specific collaborators via private repo or staging URL. Training decks, SEO pages, internal docs.
- [ ] **Tier 3 (Public)**: Anyone can see. Marketing facts, architecture patterns (sanitized), open-source tools. Boundary test: "Could a competitor use this to replicate our advantage?" If yes, it's Tier 1.
- [ ] Separate publishing commands for separate security models. Don't merge Tier 2 and Tier 3 into one command with flag-based branching.

### RED LINES (automatic fail)
- [ ] **Never expose proprietary CLAUDE.md on a public repo.** It contains your competitive advantage: agent architecture, pipeline configs, pricing, MRR, client names. Share the STRUCTURE (this starter template), never the DATA.
- [ ] **Never put credentials in CLAUDE.md or SKILL.md.** Credentials go in environment variables or secure config, referenced by name only.

---

## D4: Prompt & Structured Output (20% of exam)

### Prompt Design
- [ ] Every agent prompt defines: what the agent does, what inputs it receives, what output format it produces, and what it must NOT do.
- [ ] Complex prompts use explicit sections with headers, not a wall of text.
- [ ] Output schemas (JSON, markdown structure) are defined in CLAUDE.md before agents populate them.

### Output Quality
- [ ] Customer-facing outputs pass the "Cliff Asness test": would a quantitative expert find the methodology sound?
- [ ] Every quantitative claim shows the formula, the calculation, sensitivity analysis, and source citation.
- [ ] Data freshness is disclosed. Every data-driven output states when the data was pulled.

### Delivery Format
- [ ] Output format matches delivery context. PDF decks use fixed-canvas CSS. Dashboards use scrollable CSS. Emails use inline CSS. No format mismatches.
- [ ] Billboard mode (large venue presentations) overrides CSS variables for readability, doesn't fork the stylesheet.

### RED LINES (automatic fail)
- [ ] **Never ship unverified quantitative claims.** If you can't show the formula and cite the source, the claim doesn't ship.
- [ ] **Never promise content that doesn't exist.** If you write "full breakdown below," verify there IS a breakdown below.

---

## D5: Context & Reliability (15% of exam)

### Source of Truth
- [ ] Every piece of data has ONE source of truth. Not two files with overlapping data. Not a database AND a spreadsheet tracking the same metric.
- [ ] Formula fields in databases are read, never recalculated. The formulas contain business logic you don't know.
- [ ] When sources conflict, the hierarchy is explicit: database > API > cached file > training data.

### Failure Modes
- [ ] Silent data loss is the worst failure pattern. Agents that fail silently for weeks (missed OAuth token expiry, dropped webhook, stale cache) are architecture bugs, not operational issues.
- [ ] Circuit breakers exist for external dependencies. When Slack/Notion/API auth expires, agents degrade gracefully: write to local fallback files, alert via a separate channel, and wait for human recovery.
- [ ] Idempotent operations. Bulk writes check for existing records before creating duplicates.

### Disclosure and Transparency
- [ ] Data freshness is disclosed on every data-driven output.
- [ ] Floor-rounded numbers in published materials ("300K+" not "324,127") for durability. Exact numbers in internal source of truth only.
- [ ] Disclosure boundaries are explicit: what counts are OK to share publicly vs. what configurations are private.

### RED LINES (automatic fail)
- [ ] **Silent failures are architecture bugs.** If an agent can fail for 33 days without anyone noticing, the monitoring architecture is broken.
- [ ] **Never recalculate formula fields.** Read the result. The formula contains business logic you don't have visibility into.
- [ ] **Never maintain the same data in two places.** One source of truth, derived views everywhere else.

---

## Scoring

Count your PASS items per domain. Divide by total applicable items (exclude N/A).

| Domain | Items | Your Score | Weight |
|--------|-------|------------|--------|
| D1 Agentic Architecture | /13 | | 27% |
| D2 Tool Design & MCP | /9 | | 18% |
| D3 Claude Code Config | /14 | | 20% |
| D4 Prompt & Output | /10 | | 20% |
| D5 Context & Reliability | /10 | | 15% |

**Any RED LINE failure = not production-ready**, regardless of overall score.

**Weighted score formula**: (D1% x 0.27) + (D2% x 0.18) + (D3% x 0.20) + (D4% x 0.20) + (D5% x 0.15) = projected score out of 1000.

Target: 720/1000 to pass the CCAF exam.
