# HtALR — Skills Builder Instructions
## Skills Builder v1.0

A standalone HTML tool for generating HtALR SKILL.md files.
Open in any browser. No server, no framework, no build step required.

---

## Table of Contents

1. [What This Builds](#1-what-this-builds)
2. [Visual Style & Aesthetic](#2-visual-style--aesthetic)
3. [Structure & Layout](#3-structure--layout)
4. [Question Architecture](#4-question-architecture)
5. [Tier System](#5-tier-system)
6. [Output Block — SKILL.md Format](#6-output-block--skillmd-format)
7. [SKILL.md Anatomy](#7-skillmd-anatomy)
8. [Question Data](#8-question-data)
9. [Customisation Checklist](#9-customisation-checklist)
10. [Prompt Template for Claude](#10-prompt-template-for-claude)

---

## 1. What This Builds

A single `.html` file that generates a `SKILL.md` — a structured instruction file an agent loads before executing a specific category of task.

Features:

- **Progress bar** — tracks answered count across all 12 questions
- **Tier 1 gate** — 3 identity questions visible upfront; completion unlocks Tier 2
- **Tier 2 reveal** — 9 spec questions animate in on Tier 1 completion
- **Question types** — single-select (radio), multi-select (checkbox), free-text fields
- **Custom override** — every question has a free-text field to extend or replace options
- **Generate button** — compiles all answers into a structured `SKILL.md` block
- **Copy to clipboard** — one-click copy on the output block
- **Reset** — clears all state

Accent colour is orange (`#ff6b35`) to distinguish from the Profile Builder (green).

---

## 2. Visual Style & Aesthetic

Same terminal / dark shell aesthetic as the Profile Builder with one change:

| Property | Value |
|----------|-------|
| Accent (primary) | `#ff6b35` — orange |
| Accent 2 | `#00ccff` — cyan (Tier 2) |
| Answered border | `rgba(255,107,53,0.25)` |
| Answered ✓ | `var(--accent3)` orange |
| Generate button | orange background |
| Progress bar | gradient orange → cyan |

All other tokens identical to Profile Builder (`--bg`, `--surface`, `--border`, fonts, etc.).

---

## 3. Structure & Layout

```
Header (title + subtitle)
    |
Progress bar (answered count + percentage)
    |
Tier 1 label — orange
    |
Q01 — Skill Name (custom text only)
Q02 — Skill Category (single-select + custom)
Q03 — Trigger Condition (multi-select + custom)
    |
Tier 2 Gate (locked → unlocks on Tier 1 complete)
    |
Tier 2 label — cyan
    |
Q04 — Input Type (multi-select + custom)
Q05 — Output Type (multi-select + custom)
Q06 — Execution Steps (free text only)
Q07 — Quality Rules (multi-select + custom)
Q08 — Constraints (multi-select + custom)
Q09 — Dependencies (multi-select + custom)
    |
Divider
    |
Q10 — Bundled Resources (multi-select + custom)
Q11 — Failure Mode (single-select + custom)
Q12 — Skill Description (free text only)
    |
Generate button + Reset button
    |
Output section (hidden → visible on generate)
    |
Generated SKILL.md block + Copy button
```

---

## 4. Question Architecture

Same pattern as Profile Builder. See Profile Builder Instructions §4 for full HTML patterns.

Key differences for Skills Builder:

- Q01 is custom-text-only (no options) — skill name is a slug
- Q06 (Execution Steps) is a large free-text field — multi-line textarea
- Q12 (Skill Description) is a large free-text field — this maps directly to the YAML frontmatter `description` field

Custom field key format: `qN-fieldname` (e.g. `q1-name`, `q6-steps`, `q12-desc`).

---

## 5. Tier System

### Tier 1 — Skill Identity (Q01–Q03)

Always visible. Gates Tier 2.

| Q | What it captures |
|---|-----------------|
| Q01 | Skill name (slug) |
| Q02 | Domain / category |
| Q03 | Trigger conditions — when should the agent load this skill |

### Tier 2 — Skill Specification (Q04–Q12)

Unlocks when Q01 + Q02 + Q03 are answered.

| Q | What it captures |
|---|-----------------|
| Q04 | Input types |
| Q05 | Output types |
| Q06 | Execution steps (workflow) |
| Q07 | Quality rules |
| Q08 | Constraints |
| Q09 | Dependencies (tools, packages, other skills) |
| Q10 | Bundled resources (scripts/, references/, assets/) |
| Q11 | Failure mode |
| Q12 | Skill description (YAML frontmatter — primary trigger) |

### Progress tracking

Total: 12 questions.
Progress bar = `(answered.size / 12) * 100`%

---

## 6. Output Block — SKILL.md Format

Generated on button click. Paste into `SKILL.md` inside the skill folder.

```markdown
---
name: skill-name
description: [Q12 — triggering description]
compatibility:
  - [each dependency from Q09]
---

# skill-name

## Overview

Category: [Q02]

**Trigger conditions:**
- [each item from Q03]

## Input

- [each item from Q04]

## Output

- [each item from Q05]

## Execution Steps

[Q06 — free text, preserves formatting]

## Quality Rules

- [each item from Q07]

## Constraints

- [each item from Q08]

## Failure Mode

- [Q11]

## Bundled Resources

- [each item from Q10, if any]

---

*HtALR Skill — skill-name*
*Project: human to agentic language rosetta*
```

### Field resolution rules

- Single-select: custom text → selected value → `"not specified"`
- Multi-select: merge selected values + custom text, filter empty strings
- Free text: use as-is, preserve line breaks
- `compatibility` block only rendered if Q09 has values
- `Bundled Resources` block only rendered if Q10 has values and doesn't include "No bundled resources needed"
- Tier 2 sections only appear if `answered.size > 3`

---

## 7. SKILL.md Anatomy

A valid `SKILL.md` has three layers:

### Layer 1 — YAML Frontmatter (always in context)

```yaml
---
name: skill-slug
description: >
  Triggering description. Be specific and slightly pushy.
  Include what the skill does AND the contexts that should load it.
  Example: "Use this skill whenever the user mentions invoices,
  receipts, or billing documents — even if they don't say 'PDF'."
compatibility:
  - bash_tool
  - pip install pandas
---
```

Rules:
- `name` — slug format, matches folder name
- `description` — primary trigger mechanism; agent decides whether to load the skill based on this alone. Make it push a little — agents tend to undertrigger.
- `compatibility` — optional; list tools and packages required

### Layer 2 — SKILL.md Body (in context when skill triggers)

- Overview, trigger conditions, input/output spec
- Execution steps — ordered workflow the agent follows
- Quality rules and constraints
- Failure mode
- Pointers to bundled resources (not their content)
- Target: under 500 lines

### Layer 3 — Bundled Resources (loaded on demand)

```
skill-name/
├── SKILL.md
├── scripts/       — executable helpers (validate.py, transform.py, etc.)
├── references/    — docs loaded when needed (api-spec.md, schema.md)
├── assets/        — templates, fonts, icons
└── examples/      — sample inputs and outputs
```

Reference files from SKILL.md with a clear pointer:
```markdown
## Bundled Resources

- `scripts/validate.py` — validates output before delivery. Run after every generation.
- `references/schema.md` — field schema for this output type. Read before generating.
```

---

## 8. Question Data

| Q | Category | Type | Options |
|---|----------|------|---------|
| 01 | Skill Name | custom | slug text input |
| 02 | Skill Category | single | Document creation / Code generation / Data processing / Research and analysis / Communication and writing / File transformation / Workflow automation / UI and frontend |
| 03 | Trigger Condition | multi | File type mention / Create or generate / Read or extract / Specific tool / Multi-step workflow / Transform or convert |
| 04 | Input Type | multi | Free text / Uploaded file / Structured data / Code / URL / Image / Template |
| 05 | Output Type | multi | Markdown / DOCX / PPTX / XLSX / PDF / Code file / HTML / Inline response / Structured data |
| 06 | Execution Steps | custom | Multi-line textarea |
| 07 | Quality Rules | multi | Matches template / Validated before delivery / No hallucinated data / Immediately usable / Consistent structure / Sources cited / Edge cases handled |
| 08 | Constraints | multi | Never invent data / Never deviate from template / Never skip validation / Never use placeholders / Never exceed scope / Never proceed if ambiguous |
| 09 | Dependencies | multi | bash_tool / web_search / web_fetch / Python packages / Node packages / Another HtALR skill / MCP server |
| 10 | Bundled Resources | multi | scripts/ / references/ / assets/ / examples/ / No bundled resources |
| 11 | Failure Mode | single | Ask for missing info / Attempt with assumptions flagged / Partial output + note gaps / Abort and surface blocker |
| 12 | Skill Description | custom | Large textarea — maps to YAML frontmatter description |

---

## 9. Customisation Checklist

**Questions**
- [ ] Review Q02 category options — add domain-specific categories as needed
- [ ] Review Q09 dependency options — add project-specific tools
- [ ] Update `TOTAL` in JS if question count changes

**Output format**
- [ ] `generateSkill()` builds the SKILL.md string — extend `out +=` blocks for new sections
- [ ] YAML frontmatter is always rendered; body sections are Tier 2 gated
- [ ] Skill description (Q12) maps directly to `description:` in frontmatter — no transformation

**Styling**
- [ ] Orange accent is `#ff6b35` (`--accent3`) — change in `:root` for theme variants
- [ ] Keep orange distinct from Profile Builder green to avoid confusion when both tools are open

---

## 10. Prompt Template for Claude

```
Build a standalone HtALR Skills Builder HTML file using the Skills Builder
instructions I am providing. Output a single complete .html file I can open
in any browser with no server or build step required.

Changes from base version:
  [LIST ANY CHANGES]

New questions to add (if any):
  Q[N]: [Category] — [Question text]
    Options: [option 1] / [option 2] / [option 3]
    Type: [single | multi | custom]

New dependency options to add (if any):
  [tool or package name]

Style changes (if any):
  [e.g. "change accent to #00ff88 to match profile builder"]

Keep the terminal / dark shell aesthetic.
Output a single complete .html file.
```

---

## Technical Notes

Same as Profile Builder — no template literals in `<script>`, custom inputs trigger answered state, Tier 2 reveal is CSS-driven, output is plain string concatenation.

**Key difference:** Skills Builder uses `var` and traditional `function` syntax throughout JS (no arrow functions, no template literals) to maximise parser safety given the nested markdown/YAML content in the output block.

**YAML frontmatter in output.** The generated SKILL.md starts with `---`. This is plain text in the output block — no special handling needed. The copy button copies the raw string including frontmatter delimiters.

**Slug validation.** Q01 accepts any text — no validation enforced. Convention is lowercase, hyphen-separated. Document this expectation in the placeholder text.

---

*HtALR Skills Builder Instructions v1.0*
*Project: human to agentic language rosetta*
