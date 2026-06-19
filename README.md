# Nate's Custom Claude Skills Collection

A collection of ten custom [Claude Skills](https://support.claude.com/en/articles/12512180-use-skills-in-claude) spanning AI security and governance, research and strategy, and writing and design. Each skill is a self-contained folder with a `SKILL.md` instruction file that Claude loads automatically when your request matches the skill's description.

## What is a Skill?

A skill is a folder containing a required `SKILL.md` (YAML frontmatter with `name` and `description`, followed by Markdown instructions) plus optional `references/`, `scripts/`, and `assets/` directories. Claude reads the description to decide when a skill applies, then follows the instructions inside. Skills are instructions and resources, not executables — any tools or infrastructure a skill references (databases, approval gates, vaults) must exist in the environment that runs it.

## Skills

### AI Security & Governance

| Skill | Description | Bundled references |
| --- | --- | --- |
| **`agent-governance-architect`** | Moves AI pilots into secure, auditable production via human-linked auth, fine-grained authorization, and lifecycle governance through a unified control plane. | — |
| **`secure-ai-app-engineer`** | Security Architect that turns raw workflows into production-grade secure AI apps using OWASP, AppSec, and infrastructure best practices. | — |
| **`ai-bom-architect`** | Generates a compliance-ready AI Bill of Materials mapped to the EU AI Act, California AB 2013, and Colorado SB 24-205, with zero-trust provenance. | `references/testing_suite.md` |
| **`enterprise-data-auditor`** | Read-only, high-trust internal agent that audits database transactions for compliance anomalies with human approval gates, telemetry, and crash recovery. | — |
| **`typescript-security-audit-checklist`** | Audits TypeScript code or PRs for runtime-validation gaps, dangerous type escapes, prototype pollution, and insecure compiler/CI configuration. | `references/testing_suite.md` |

### Research & Strategy

| Skill | Description | Bundled references |
| --- | --- | --- |
| **`corporate-teardown-briefing`** | Senior equity-analyst-style 6-point teardown of any company: what it does, how it makes money, key numbers, competition, biggest risk, and the common misconception. | — |
| **`messaging-stress-tester`** | A "Skeptical AI Architect" that red-teams messaging, pitches, and strategy docs to expose logical gaps, fluff, and unsubstantiated claims. | — |
| **`ai-executive-briefing-generator`** | Turns raw AI news feeds into a high-signal executive briefing plus a companion LinkedIn draft, filtering out hype. | — |

### Writing & Design

| Skill | Description | Bundled references |
| --- | --- | --- |
| **`editorial-co-writer`** | Structural co-writer that wraps factual scaffolding and transitions around an author's immutable text without altering their voice; infers style from uploaded samples. | — |
| **`frontend-skill`** | Art-direction-led frontend skill enforcing restrained composition, image-led hierarchy, strong branding, and tasteful motion over generic card grids. | `references/frontend_tasks.md` |

## Installation

### Claude.ai (Pro, Max, Team, Enterprise with code execution enabled)

1. Download the skill `.zip` you want (or clone this repo and zip the skill folder yourself — keep the skill's own folder as the top level of the archive).
2. In Claude.ai, go to **Settings / Customize → Skills**.
3. Click **+ → Create skill** and upload the `.zip`.
4. The skill appears in your list with a toggle. Start a **new conversation** for it to load.

> Custom skills are private to your account and do not sync across surfaces — a skill uploaded to Claude.ai must be uploaded separately to the API, and vice versa.

### Repacking from source

Each skill lives in its own folder. To rebuild an uploadable archive:

```bash
zip -r corporate-teardown-briefing.zip corporate-teardown-briefing
```
The `SKILL.md` must sit one level inside the named folder at the root of the zip (e.g. `corporate-teardown-briefing/SKILL.md`).

## Repository Structure

```
.
├── agent-governance-architect/
│   └── SKILL.md
├── secure-ai-app-engineer/
│   └── SKILL.md
├── ai-bom-architect/
│   ├── SKILL.md
│   └── references/testing_suite.md
├── enterprise-data-auditor/
│   └── SKILL.md
├── typescript-security-audit-checklist/
│   ├── SKILL.md
│   └── references/testing_suite.md
├── corporate-teardown-briefing/
│   └── SKILL.md
├── messaging-stress-tester/
│   └── SKILL.md
├── ai-executive-briefing-generator/
│   └── SKILL.md
├── editorial-co-writer/
│   └── SKILL.md
├── frontend-skill/
│   ├── SKILL.md
│   └── references/frontend_tasks.md
└── README.md
```

## Authoring Notes

- **Names** are kebab-case (lowercase letters, digits, hyphens), max 64 characters, and must not contain reserved words (`anthropic`, `claude`).
- **Descriptions** are capped at 1024 characters and should state *what the skill does* and *when to trigger it*, since Claude uses the description for invocation.
- **Reference files** under `references/` are loaded on demand, keeping the main `SKILL.md` lean.
- Skills describe behavior; they do not provision tools. Skills that mention read-only SQL, approval gates, or secret vaults assume those capabilities exist in the host environment.

## License

Add your license of choice (e.g. MIT) here.
