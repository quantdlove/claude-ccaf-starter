# CCAF Architecture Starter Kit

A starter kit for building production Claude systems that pass the [Claude Certified Architect Foundations](docs/ccaf-exam-guide.pdf) exam.

Built by [David Love](https://linkedin.com/in/quantdlove), Head of Sales & Operations at [Quiver Quant](https://quiverquant.com). Quiver runs 9 scheduled Claude agents, 27 MCP endpoints, and two newsletter pipelines serving 300K+ subscribers, all architected around CCAF principles.

## What's Inside

### Core Files

| File | What It Is |
|------|-----------|
| [CLAUDE.md](CLAUDE.md) | Starter template showing the STRUCTURE of a production CLAUDE.md. Generic "Acme Analytics" example. Covers: team context, writing rules, precision rules, design system, MCP config, agent architecture, file hygiene, token efficiency. |
| [Audit Checklist](docs/audit-checklist.md) | 56 pass/fail items across all 5 CCAF domains. Red-line items that are automatic failures. Scoring formula. Use this to audit your own system. |
| [Pattern Catalog](docs/pattern-catalog.md) | 8 production-tested architecture patterns with diagrams, implementation guidance, CCAF domain mapping, and anti-patterns. |
| [Worked Audit Example](docs/example-audit.md) | Complete D1-D5 audit of a fictional "Acme Analytics" system with Mermaid architecture diagrams, per-domain scoring, and prioritized fix list. |

### .claude/ Skeleton

Example skill, rules, and command files showing the structure of each config type:

| File | Purpose |
|------|---------|
| [Example Skill](claude-config/skills/example-skill/SKILL.md) | Weekly report generator. Shows: inputs, ordered steps, output format, validation, constraints. |
| [Example Rules](claude-config/rules/example-rules.md) | Path-scoped marketing rules. Shows: tone, data integrity, brand, review gates. |
| [Example Command](claude-config/commands/example-command.md) | Publishing command. Shows: path validation, content scan, CSS dependency check, push, verify. |

> **Note**: These files are under `claude-config/` in this repo. In your own project, move them to `.claude/` (skills/, rules/, commands/).

### Reference Materials

| File | What It Is |
|------|-----------|
| [CCAF Exam Guide](docs/ccaf-exam-guide.pdf) | Anthropic's official exam guide. Domain weights, exam format, study topics. |
| [Ambassador Deck](docs/ambassador-deck.html) | Slide deck from the Capital Factory presentation on CCAF architecture. Open in a browser or print to PDF. |
| [Live Examples (GitHub Pages)](https://quantdlove.github.io/quiver-learn/) | Production training decks and filing guides built with CCAF patterns. Browse rendered output to see what finished architecture looks like. |

## The Five CCAF Domains

| Domain | Weight | Core Question |
|--------|--------|--------------|
| D1: Agentic Architecture | 27% | Can you draw the pipeline? Is each agent's role, autonomy tier, and handoff pattern explicit? |
| D2: Tool Design & MCP | 18% | Are you using the right tool for the job? Structured queries for enumeration, semantic search for discovery? |
| D3: Claude Code Config | 20% | Is CLAUDE.md the spec? Are skills, commands, and rules cleanly separated? Do hooks enforce policy? |
| D4: Prompt & Structured Output | 20% | Does every agent define its inputs, outputs, and constraints? Does output format match delivery context? |
| D5: Context & Reliability | 15% | One source of truth per metric? Circuit breakers on external dependencies? Silent failures caught? |

## Quick Start

1. Copy `CLAUDE.md` to your project root. Replace "Acme Analytics" with your company. Fill in your team, rules, agents, and folder structure.
2. Create `.claude/skills/`, `.claude/commands/`, `.claude/rules/` directories. Use the examples as templates.
3. Run the [audit checklist](docs/audit-checklist.md) against your system. Fix any red-line failures first.
4. Read the [pattern catalog](docs/pattern-catalog.md) when you hit a design decision. Each pattern links to the CCAF domain it addresses.

## License

MIT
