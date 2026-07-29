# Working Memory: Acme Analytics

## About the Team
- **Company**: Acme Analytics. B2B SaaS platform for market intelligence.
- **Team size**: 4 people. CEO (product + engineering), Head of Ops (sales, marketing, content), 2 engineers.
- **Key constraint**: Small team means Claude agents handle repeatable workflows. Every agent output is human-reviewed before it ships.

## Writing Rules (MANDATORY)
- **No em dashes.** Use periods, commas, colons, or rewrite. Em dashes are an AI writing tell.
- **Write like a human.** Short sentences. Conversational. No "furthermore," "moreover," "it's worth noting."
- **No fabricated metrics.** Every number must trace to a verified source. If a number cannot be sourced, write `[VERIFY]` and flag for review. Do not estimate, round creatively, or use placeholder numbers that look real.

## Precision Rules (MANDATORY)
- **No cherry-picked statistics.** Never present a stat from a subset of data as if it represents the whole. If the stat only applies to a subset, say so explicitly.
- **No implied conclusions.** Never juxtapose facts to manufacture a narrative the data doesn't support. State facts, let readers decide.
- **Source + context test.** Every factual claim must pass two checks: (1) Can you cite the primary source? (2) Are you presenting it in full context? If either fails, use `[VERIFY]` or remove the claim.
- **No editorializing in data sections.** Data sections present data. Save analysis for clearly labeled editorial content.

## Template Rules (MANDATORY)
- **Templates are design references, NOT data sources.** When generating output from a template, ALWAYS pull real data from the source of truth (your databases, APIs, local files). NEVER use sample data from the template itself.
- **Validate before delivering.** After generating any data-driven document, verify key numbers match the source. Flag mismatches.

## Quantitative Rigor Standard (MANDATORY)
Every quantitative claim in customer-facing output must pass four checks:
1. **Show the formula.** Display as a visual equation with proper typesetting.
2. **Show the calculation.** Plug in the actual numbers. The reader should be able to verify every step.
3. **Show sensitivity.** State the key assumption and how the result changes if the assumption is wrong.
4. **Cite the source.** Academic paper, practitioner reference, or primary data source for the methodology.

## Design System

### Global Rules
- **Default aesthetic**: Clean white widget. White base, brand accent color, light gray (#f5f5f7).
- **Never dark-mode a full document.** Content surfaces stay white.
- **No black borders.** Use #e0e0e0 for all borders.

### CSS Systems (match delivery format to the right system)
| Delivery Format | CSS System | Key Pattern |
|---|---|---|
| PDF slide deck | `brand-deck.css` | Fixed canvas per slide, overflow hidden |
| Dashboard / briefing | `brand-dashboard.css` | Scrollable, max-width container |
| Email / newsletter | Inline CSS | No external stylesheets, table-based layout |

### CSS Enforcement Rule
Every new HTML document MUST use the correct CSS system based on delivery format. No exceptions.

## MCP Server
- **What it does**: Exposes your API endpoints as MCP tools Claude can call directly.
- **Rule 1**: Always call `search_datasets` at the start of any session that uses MCP data. The tool list changes when engineers push updates.
- **Rule 2**: Retry + fallback on errors. If an MCP call returns 500 or times out, retry once. If retry fails, fall back to cached data and log the error.
- **Rule 3**: Pagination awareness. For small datasets, set limit high enough to get all results. For large datasets, use filters to narrow before paginating.

## Scheduled Agents
- **Architecture**: Sequential pipeline with async handoff via shared-state files. NOT parallel.
- **Tiered autonomy**: Tier 1 auto-act (keyword filtering, auto-classification). Tier 2 recommend (agent researches, human confirms). Tier 3 priority review (flagged for senior decision-maker).
- **Human gate**: Every customer-facing output gets human review before shipping. Agents recommend, humans approve.

## File Output Hygiene (MANDATORY)
Every output follows one of two tracks:
- **Track 1: OVERWRITE** (recurring outputs). Single canonical filename. Each run overwrites the previous version.
- **Track 2: ARCHIVE** (historical records). Date-stamped, keep all editions.
- **Decision test**: "Will next week's version replace this one?" Yes = Track 1. No = Track 2.

## Token Efficiency Rules (MANDATORY)
- **Default model: Sonnet.** Use Opus ONLY for new strategy/architecture design, new logic that doesn't exist yet, and creative writing.
- **Keep responses lean.** Don't repeat the question back. Don't over-explain.
- **Batch tool calls.** Run independent reads/searches in parallel, not sequentially.
- **Load context on demand.** Read memory files only when the task requires that context.

## Folder Structure
| Folder | Contents |
|--------|----------|
| `sales/` | Decks, proposals, shared CSS |
| `marketing/` | Newsletter editions, analytics, advertising |
| `ops/` | Dashboards, standup decks, study materials |
| `product/` | Customer insights, metrics |
| `business-development/` | Pipeline state, deliverables, research |
| `memory/` | Context files, people, projects, glossary |
| `scheduled-agents/` | Canonical SKILL.md files for all agents |
| `.claude/skills/` | Cowork skill definitions |
| `.claude/commands/` | Slash commands |
| `.claude/rules/` | Path-scoped rules |

## Session-End Rules (MANDATORY)
- **Handoff files**: After any strategy or architecture session, write a handoff file with: summary, decisions made, open questions, next actions (tagged by model), files changed, constraints, recommended prompt for next session.
- **Spec-first rule**: When making agent pipeline changes, update CLAUDE.md (the contract) FIRST, before editing any agent files. The spec precedes the implementation.
