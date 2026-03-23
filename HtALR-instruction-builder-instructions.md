# HtALR — Instruction Builder Instructions
## Instruction Builder v1.0

A standalone HTML tool for generating universal HtALR instruction blocks.
Paste the output into the system prompt or custom instructions field of any AI assistant.
Open in any browser. No server, no framework, no build step required.

---

## Table of Contents

1. [What This Builds](#1-what-this-builds)
2. [Visual Style & Aesthetic](#2-visual-style--aesthetic)
3. [Structure & Layout](#3-structure--layout)
4. [Import Shortcut](#4-import-shortcut)
5. [Question Architecture](#5-question-architecture)
6. [Tier System](#6-tier-system)
7. [Output Block Format](#7-output-block-format)
8. [Question Data](#8-question-data)
9. [Customisation Checklist](#9-customisation-checklist)
10. [Prompt Template for Claude](#10-prompt-template-for-claude)

---

## 1. What This Builds

A single `.html` file that generates a universal instruction block — a structured markdown prompt the user pastes into any AI assistant's system prompt or custom instructions field.

Features:

- **Import shortcut** — paste an existing HtALR profile block to pre-fill the form automatically
- **Progress bar** — tracks answered count across all 10 questions
- **Tier 1 gate** — 5 core behaviour questions visible upfront; completion unlocks Tier 2
- **Tier 2 reveal** — 5 depth setting questions animate in on Tier 1 completion
- **Question types** — single-select (radio), multi-select (checkbox), free-text fields
- **Custom override** — every question has a free-text escape hatch
- **Generate button** — compiles all answers into a universal instruction block
- **Copy to clipboard** — one-click copy on the output block
- **Reset** — clears all state including the import field

Accent colour is purple (`#b388ff`) to distinguish from the Profile Builder (green) and Skills Builder (orange).

---

## 2. Visual Style & Aesthetic

Same terminal / dark shell aesthetic as the other builders with one change:

| Property | Value |
|----------|-------|
| Accent (primary) | `#b388ff` — purple |
| Accent 2 | `#00ccff` — cyan (Tier 2) |
| Answered border | `rgba(179,136,255,0.2)` |
| Answered ✓ | `var(--accent4)` purple |
| Generate button | purple background |
| Progress bar | gradient purple → cyan |
| Grid overlay | `rgba(179,136,255,0.03)` |

All other tokens identical to Profile Builder (`--bg`, `--surface`, `--border`, fonts, etc.).

---

## 3. Structure & Layout

```
Header (title + subtitle)
    |
Progress bar (answered count + percentage)
    |
Import label — purple (optional shortcut)
    |
Import block — paste existing HtALR profile to pre-fill
    |
Tier 1 label — green
    |
Q01 — Register
Q02 — Output Format
Q03 — Autonomy
Q04 — Constraints (multi-select)
Q05 — Failure Mode
    |
Tier 2 Gate (locked → unlocks on Tier 1 complete)
    |
Tier 2 label — cyan
    |
Q06 — Explanation Density
Q07 — Inference Depth
Q08 — Persona
Q09 — Context (free text — who the user is)
Q10 — Flags (include all / core only / exclude)
    |
Generate button + Reset button
    |
Output section (hidden → visible on generate)
    |
Generated instruction block + Copy button
```

---

## 4. Import Shortcut

The import block sits above Tier 1. It is optional — users can skip it and fill the form manually.

### Behaviour

User pastes a generated HtALR profile block (output from the Profile Builder) into the textarea and clicks "parse profile". The JS parser reads the markdown and pre-fills matching form fields.

### Parser logic

The parser uses regex to extract values from the profile block by section heading:

| Profile field | Maps to |
|---------------|---------|
| `- Tone:` | Q01 register — matched against radio options, falls back to custom field |
| `- Default format:` | Q02 output format |
| `- Default:` (Autonomy section) | Q03 autonomy |
| `### Constraints` block | Q04 constraints — matched against checkbox values |
| `- Failure mode:` or `### Failure mode` | Q05 failure mode |
| `### Explanation density` | Q06 explanation density |
| `### Inference depth` | Q07 inference depth |
| `### Persona` | Q08 persona |

Fields that don't match a radio option fall back to the custom text field for that question.

### Visual state

- Default: dashed purple border
- After successful parse: solid border + `rgba(179,136,255,0.03)` background + `✓ profile parsed` status message

---

## 5. Question Architecture

Same pattern as Profile Builder and Skills Builder. See Profile Builder Instructions §4 for full HTML patterns.

Key differences:

- Q04 (Constraints) is multi-select with 8 options covering the most common agent behaviour anti-patterns
- Q09 (Context) is free-text only — a large textarea for the user to describe who they are and what they do
- Q10 (Flags) controls whether the flag table is included in output, and at what depth

Custom field key format: `qN-custom`, `q9-context`.

---

## 6. Tier System

### Tier 1 — Core Behaviour (Q01–Q05)

Always visible. Gates Tier 2. Covers the minimum viable instruction set — enough for Tier 1 output to be usable in any AI assistant.

### Tier 2 — Depth Settings (Q06–Q10)

Unlocks when Q01–Q05 are all answered.

| Q | What it captures |
|---|-----------------|
| Q06 | Explanation density — how much reasoning to show |
| Q07 | Inference depth — how much to read between the lines |
| Q08 | Persona — voice mirroring or custom persona |
| Q09 | Context — who the user is, what they do |
| Q10 | Flags — whether to include HtALR flag conventions |

### Progress tracking

Total: 10 questions.
Progress bar = `(answered.size / 10) * 100`%

---

## 7. Output Block Format

Generated on button click. Universal markdown — paste into any system prompt field.

```markdown
# Agent Instructions
## HtALR Universal Profile — generated YYYY-MM-DD

---

## Context

[Q09 — who the user is and what they do — only if provided]

---

## Communication

Tone and register: [Q01]

Persona: [Q08 — only if set and not "no persona"]

## Output

Default format: [Q02]

When producing code: declare the language and format before the code block begins.

## Autonomy

When a query is ambiguous: [Q03]

Inference depth: [Q07 — Tier 2 only]

## Constraints

Never do any of the following:

- [each item from Q04]

## Explanation Density

[Q06 — Tier 2 only]

## Failure Mode

When uncertain or at the edge of knowledge: [Q05]

---

## Flags        ← only if Q10 is not "exclude"

[flag table — full or core depending on Q10 selection]

Flags are stackable: `-lfg -q` = execute silently, conclusion only.

---

## How to Use These Instructions

Paste this block into the system prompt or custom instructions field of any AI assistant.
These instructions define a baseline — individual queries can override them using flags or plain language.
To update these instructions, revise the relevant section and re-paste.

---

*HtALR Universal Instruction Block*
*Project: human to agentic language rosetta*
```

### Field resolution rules

- Single-select: custom text → selected value → default fallback string
- Multi-select: merge selected values + custom text, filter empty strings
- Free text: use as-is, preserve line breaks
- Context block only rendered if Q09 has content
- Persona line only rendered if Q08 is not "No persona" and not empty
- Flags block only rendered if Q10 is not "exclude"
- Tier 2 sections (explanation density, inference depth, persona) only rendered if `answered.size > 5`

### Flag inclusion modes

| Q10 selection | Flag table content |
|---------------|-------------------|
| Include full flag set | All 10 flags including `-toboldlygo`, `-nevermore`, `-yolo`, `-obey`, `-d`, `-o` |
| Core flags only | `-lfg`, `-q`, `-v`, `-halp` |
| Exclude | No flag section rendered |

Custom flags from Q10 free-text are appended to the table. Format expected: `-flag = description` (one per line).

---

## 8. Question Data

| Q | Category | Type | Options |
|---|----------|------|---------|
| 01 | Register | single | Direct and brief / Casual and conversational / Formal and precise / Collaborative / Executive |
| 02 | Output Format | single | Markdown / Plain prose / Code only / Structured data / Tables preferred |
| 03 | Autonomy | single | Ask before acting / Assume and flag / Commit, I'll correct / Offer interpretations |
| 04 | Constraints | multi | No encouragement phrases / Never restate question / No unrequested suggestions / No bullets in prose / No closing summaries / No passive voice / No preamble / No postamble |
| 05 | Failure Mode | single | Ask for clarification / Attempt and flag inline / Attempt silently / Surface as question first |
| 06 | Explanation Density | single | Conclusion only / Conclusion + one-line why / Full reasoning / On demand |
| 07 | Inference Depth | single | Literal / Contextual / Inferential / Anticipatory |
| 08 | Persona | single | No persona / Mirror writing style / Custom persona |
| 09 | Context | custom | Large textarea — user describes role, domain, use case |
| 10 | Flags | single | Include full flag set / Core flags only / Exclude |

---

## 9. Customisation Checklist

**Import parser**
- [ ] Update regex patterns if profile block format changes
- [ ] Add new field mappings as new profile sections are added
- [ ] Test against both Tier 1 and Tier 2 profile blocks

**Questions**
- [ ] Q04 constraint options — add new anti-patterns as they emerge
- [ ] Q10 flag table — update when new flags are added to the HtALR flag set
- [ ] Update `TOTAL` in JS if question count changes
- [ ] Tier 1 gate currently requires Q01–Q05 — update `tier1qs` array if split changes

**Output format**
- [ ] `generateInstructions()` builds the instruction string — extend `out +=` blocks for new sections
- [ ] Context block is gated on non-empty Q09 — adjust if context should always appear
- [ ] Flag table content is controlled by Q10 value — add new flags to both `include-all` branch and flag reference

**Styling**
- [ ] Purple accent is `#b388ff` (`--accent4`) — change in `:root` for theme variants
- [ ] Grid overlay uses purple at `0.03` opacity — consistent with other builders

---

## 10. Prompt Template for Claude

```
Build a standalone HtALR Instruction Builder HTML file using the Instruction
Builder instructions I am providing. Output a single complete .html file I
can open in any browser with no server or build step required.

Changes from base version:
  [LIST ANY CHANGES]

New constraint options to add (if any):
  [constraint text]

New flag to add (if any):
  `-[flag]` — [behaviour]

Questions to add (if any):
  Q[N]: [Category] — [Question text]
    Options: [option 1] / [option 2]
    Type: [single | multi | custom]

Style changes (if any):
  [e.g. "change accent to match profile builder green"]

Keep the terminal / dark shell aesthetic.
Output a single complete .html file.
```

---

## Technical Notes

Same as Profile Builder and Skills Builder — no template literals in `<script>`, custom inputs trigger answered state, Tier 2 reveal is CSS-driven, output is plain string concatenation. Uses `var` and traditional `function` syntax throughout JS.

**Import parser is best-effort.** It matches on partial strings and falls back gracefully — unmatched values go to the custom field rather than failing silently. It does not validate the pasted content — garbage in, best-guess out.

**Flag custom input parser.** Expects one flag per line in the format `-flag = description`. Lines that don't match this pattern are included as-is inside a table row. No validation enforced.

**Tier 1 gate requires all 5 questions.** Unlike the Profile Builder (3 questions), Tier 1 here is 5 questions — the `tier1qs` array in JS controls this. Update it if the Tier 1 / Tier 2 split changes.

---

*HtALR Instruction Builder Instructions v1.0*
*Project: human to agentic language rosetta*
