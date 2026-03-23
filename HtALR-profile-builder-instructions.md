# HtALR — Profile Builder Instructions
## Profile Builder v1.0

A standalone HTML profile builder for generating HtALR session instruction blocks.
Open in any browser. No server, no framework, no build step required.

---

## Table of Contents

1. [What This Builds](#1-what-this-builds)
2. [Visual Style & Aesthetic](#2-visual-style--aesthetic)
3. [Structure & Layout](#3-structure--layout)
4. [Question Architecture](#4-question-architecture)
5. [Tier System](#5-tier-system)
6. [Output Block](#6-output-block)
7. [Flag Reference](#7-flag-reference)
8. [Question Data](#8-question-data)
9. [Customisation Checklist](#9-customisation-checklist)
10. [Prompt Template for Claude](#10-prompt-template-for-claude)

---

## 1. What This Builds

A single `.html` file that runs a complete HtALR profile builder. Features:

- **Progress bar** — tracks answered count across all questions in real time
- **Tier 1 gate** — 3 required questions visible upfront; completion unlocks Tier 2
- **Tier 2 reveal** — 11 deep profile questions animate in on Tier 1 completion
- **Question types** — single-select (radio), multi-select (checkbox), free-text custom fields
- **Custom override** — every question has a free-text field to extend or replace options
- **Profile name** — user names the profile before generating
- **Generate button** — compiles all answers into a structured markdown profile block
- **Copy to clipboard** — one-click copy on the output block
- **Reset** — clears all state and returns to initial view

---

## 2. Visual Style & Aesthetic

**Hacker terminal / dark shell aesthetic.**

| Property | Value |
|----------|-------|
| Background | `#0d0d0d` |
| Surface | `#141414` |
| Surface 2 | `#1a1a1a` |
| Border default | `#2a2a2a` |
| Border active | `#3a3a3a` |
| Accent (green) | `#00ff88` |
| Accent 2 (cyan) | `#00ccff` |
| Accent 3 (orange) | `#ff6b35` |
| Text primary | `#e8e8e8` |
| Text muted | `#666` |
| Text dim | `#444` |
| Font — mono | JetBrains Mono (Google Fonts) |
| Font — sans | Space Grotesk (Google Fonts) |
| Border radius | 3–4px — sharp, not rounded |
| Buttons | Lowercase labels — terminal feel |

**Grid background:**
- Fixed full-viewport grid overlay
- `rgba(0,255,136,0.03)` lines at 40px intervals
- Pointer-events none, z-index 0

**Color-coded elements:**
- Tier 1 label — `var(--accent)` green border + text
- Tier 2 label — `var(--accent2)` cyan border + text
- Selected single-select — green tint + green border
- Selected multi-select — cyan tint + cyan border
- Answered question marker — green `✓` appended to question number
- Progress bar fill — linear gradient green → cyan

---

## 3. Structure & Layout

```
Header (title + subtitle)
    |
Progress bar (answered count + percentage)
    |
Tier 1 label
    |
Q01 — Register
Q02 — Output Format
Q03 — Autonomy
    |
Tier 2 Gate (locked → unlocks on Tier 1 complete)
    |
Tier 2 label
    |
Q04 — Explanation Density
Q05 — Token Optimisation
Q06 — Inference Depth
Q07 — Scope
Q08 — Iteration Posture
Q09 — Constraints
Q10 — Optimised For
    |
Divider
    |
Q11 — Persona Mode
Q12 — Trigger Vocabulary
Q13 — Failure Mode
Q14 — Profile Name
    |
Generate button + Reset button
    |
Output section (hidden → visible on generate)
    |
Generated profile block + Copy button
```

All questions are visible on a single scrollable page. No navigation, no screen transitions.

---

## 4. Question Architecture

Each question block contains:

```
[QUESTION NUMBER] // [CATEGORY]
[Question text]
[Options — radio or checkbox]
[Custom free-text field]
```

### Single-select question (radio)

```html
<div class="q-block" id="qN">
  <div class="q-number">Q0N // CATEGORY</div>
  <div class="q-text">Question text?</div>
  <div class="options" data-q="qN" data-type="single">
    <label class="opt">
      <input type="radio" name="qN" value="Option label">
      <span class="opt-dot"></span>Option label
    </label>
    <!-- repeat per option -->
  </div>
  <div class="custom-wrap">
    <div class="custom-label">// CUSTOM LABEL</div>
    <textarea class="custom-input" placeholder="..." rows="2" data-q="qN-custom"></textarea>
  </div>
</div>
```

### Multi-select question (checkbox)

Same structure — swap `data-type="multi"` and add class `multi` to each `.opt` and `.opt-dot`.

```html
<div class="options" data-q="qN" data-type="multi">
  <label class="opt multi">
    <input type="checkbox" name="qN" value="Option label">
    <span class="opt-dot"></span>Option label
  </label>
</div>
```

### Custom-only question (no options)

For trigger vocabulary and similar free-form inputs — omit the `.options` block entirely.
Use multiple `.custom-wrap` blocks with distinct `data-q` keys.

```html
<div class="custom-wrap">
  <div class="custom-label">// FIELD LABEL</div>
  <input type="text" class="custom-input" data-q="qN-fieldname">
</div>
```

### Answered state

A question is marked answered when:
- Any radio/checkbox within it is selected, OR
- Its custom field has non-empty content

On answered: add class `answered` to `.q-block`. This appends `✓` to the question number via CSS.

---

## 5. Tier System

### Tier 1 — Questions 1–3

Always visible. Required before Tier 2 unlocks.

Covers: register, output format, autonomy.

### Tier 2 Gate

Sits between Tier 1 questions and Tier 2 questions.

**Locked state:**
```
border: 1px dashed var(--border)
text: "// TIER 2 LOCKED — complete Tier 1 to unlock"
subtext: "N more questions covering..."
```

**Unlocked state** (triggered when Q1 + Q2 + Q3 all answered):
```
border: 1px solid rgba(0,204,255,0.3)
background: rgba(0,204,255,0.03)
text: "// TIER 2 UNLOCKED"
subtext: "Deep profile active — N more questions below"
```

### Tier 2 — Questions 4–14

Hidden by default (`opacity: 0; max-height: 0; overflow: hidden`).
Revealed on Tier 1 completion via class `.visible` (`opacity: 1; max-height: 9999px`).
Transition: `opacity 0.5s ease, max-height 0.8s ease`.

Covers: explanation density, token optimisation, inference depth, scope, iteration posture, constraints, tools, persona, triggers, failure mode, profile name.

### Progress tracking

Total question count: 14.
Progress bar = `(answered.size / 14) * 100`%
Label format: `N / 14 answered`

---

## 6. Output Block

Generated on button click. Format is markdown, optimised for pasting into a Claude project.

### Structure

```markdown
# HtALR User Profile
## [Tier 1 — Quick Profile | Tier 2 — Deep Profile] v1.0

---

[PROFILE: name]
generated: YYYY-MM-DD

---

### Register
- Tone: [value]

### Output
- Default format: [value]
- Optimised for: [comma-separated tools]

### Autonomy
- Default: [value]

### Explanation density          ← Tier 2 only
- [value]

### Token optimisation           ← Tier 2 only
- [each selected option]

### Inference depth              ← Tier 2 only
- [value]

### Scope                        ← Tier 2 only
- [value]

### Iteration posture            ← Tier 2 only
- [value]

### Failure mode                 ← Tier 2 only
- [value]

### Persona                      ← Tier 2 only, if set
- [value]

### Constraints                  ← Tier 2 only, if any set
- [each selected constraint]

### Trigger vocabulary           ← Tier 2 only, if any set
- "[word]" = [behaviour]

---

### Flags

| Flag | Behaviour |
|------|-----------| 
| `-lfg` | ... |
...

Flags are stackable: `-lfg -q` = execute silently, conclusion only.

---

*HtALR Profile v1.0 — [Tier 1 | Tier 2]*
*Project: human to agentic language rosetta*
```

### Field resolution rules

For single-select fields: use custom text if present, otherwise use selected option value, otherwise `"not specified"`.

For multi-select fields: merge selected option values + custom text, filter empty strings.

Tier 2 sections only appear in output if `answered.size > 3`.

Persona section only appears if value is not `"No persona — default register"` and not empty.

---

## 7. Flag Reference

Always included in the generated output block regardless of tier.

| Flag | Behaviour |
|------|-----------|
| `-lfg` | Execute without clarifying questions — course-correct on output |
| `-yolo` | Maximum speed and efficiency — all responses in millennial speak |
| `-q` | Quiet mode — conclusion only, no rationale |
| `-v` | Verbose mode — full reasoning and working shown |
| `-o [format]` | Override output format e.g. `-o json`, `-o prose`, `-o table` |
| `-d` | Draft mode — output flagged with confidence and assumptions inline |
| `-halp` | Options mode — return 2-3 recommended approaches before committing |
| `-obey` | Strict mode — literal execution only, no inference beyond the query |
| `-help` | Help mode — show flag list, offer to add new flag |
| `-nevermore` | Poetry mode — all outputs phrased as poetry or prose |
| `-makeitso` | Commit mode — commits query context as a change to the instruction file |

---

## 8. Question Data

Full question set for reference when regenerating or extending the builder.

| Q | Category | Type | Options |
|---|----------|------|---------|
| 01 | Register | single | Direct and brief / Casual and conversational / Formal and precise / Collaborative — thinks out loud / Executive — decisions only |
| 02 | Output Format | single | Markdown / Plain prose / Code only / Structured data (JSON/YAML) / Tables preferred |
| 03 | Autonomy | single | Ask before acting / Assume and flag assumptions / Commit and let me course-correct / Offer 2-3 interpretations first |
| 04 | Explanation Density | single | Conclusion only / Conclusion + one-line rationale / Full working shown / On-demand via trigger word |
| 05 | Token Optimisation | multi | No preamble / No postamble / Trim hedges and qualifiers / Layered — summary first / Single pass / Reference compression |
| 06 | Inference Depth | single | Literal / Contextual / Inferential / Anticipatory |
| 07 | Scope | single | Scope-locked / Flag adjacent issues / Anticipate / Goal-oriented |
| 08 | Iteration Posture | single | One strong output / Two variants / Three options / Draft first |
| 09 | Constraints | multi | No bullets in prose / No encouragement phrases / No unrequested suggestions / Never restate question / No passive voice / No closing summary |
| 10 | Optimised For | multi | Obsidian / Notion / GitHub / IDEs / Terminal / Google Docs / Slack / Email |
| 11 | Persona Mode | single | No persona / Mirror my writing style / Custom persona |
| 12 | Trigger Vocabulary | custom | slow / fast / options / commit |
| 13 | Failure Mode | single | Ask for clarification / Attempt and flag / Attempt silently / Surface as question first |
| 14 | Profile Name | single + custom | default / dev / writing / research / strategy |

Every question has a free-text custom field. Custom input overrides selected option in output generation.

---

## 9. Customisation Checklist

Before generating a new version of the builder:

**Questions**
- [ ] Review Q01–Q14 options — add, remove, or reword as the methodology evolves
- [ ] Add new questions by incrementing Q count and updating `TOTAL` in JS
- [ ] Remove questions by decrementing `TOTAL` and removing the block + data-q references
- [ ] Adjust Tier 1 / Tier 2 split — currently Q01–03 are Tier 1, Q04–14 are Tier 2

**Flags**
- [ ] Update the flag table in section 7 if new flags are added via `-makeitso`
- [ ] Update the flag table in the output generation function in the JS

**Styling**
- [ ] CSS variables are in `:root` — update accent colours here for theme variants
- [ ] Grid background opacity is `0.03` — increase for more prominent grid

**Output format**
- [ ] Profile block structure is in the `generateProfile()` function
- [ ] Add new sections by extending the `out +=` block
- [ ] Field resolution order: custom text → selected value → `"not specified"`

---

## 10. Prompt Template for Claude

Use this prompt to regenerate or extend the HtALR Profile Builder HTML file:

```
Build a standalone HtALR Profile Builder HTML file using the Profile Builder
instructions I am providing. Output a single complete .html file I can open
in any browser with no server or build step required.

Changes from base version:
  [LIST ANY CHANGES — new questions, removed questions, style changes, etc.]

New questions to add (if any):
  Q[N]: [Category] — [Question text]
    Options: [option 1] / [option 2] / [option 3]
    Type: [single | multi | custom]

Questions to remove (if any):
  Q[N], Q[N]

Flag updates (if any):
  Add: `[flag]` — [behaviour]
  Remove: `[flag]`

Style changes (if any):
  [e.g. "change accent colour to #ff6b35", "use different font"]

Keep the terminal / dark shell aesthetic as documented.
Output a single complete .html file.
```

---

## Technical Notes

**No template literals in `<script>` tags.** Use string concatenation. Template literals containing `</` can break HTML parsers.

**Radio inputs are hidden.** Interaction is handled by clicking the parent `.opt` label. The JS wires click events to toggle `.selected` class and `input.checked` state directly.

**Checkbox multi-select.** Click toggles `input.checked` and `.selected` class. A question is marked answered if any checkbox in its group is checked.

**Custom inputs trigger answered state.** Any non-empty custom field marks its parent question as answered. Key format: `qN-custom`, `qN-fieldname`.

**Tier 2 reveal is CSS-driven.** The JS adds class `.visible` — the transition handles animation. `max-height: 9999px` is the standard trick for unknown-height transitions.

**Output is plain string concatenation.** No template engine. The `generateProfile()` function builds the markdown string line by line. This avoids parser issues and keeps the output predictable.

**`answered` is a `Set`.** Stores question IDs (`q1`, `q2`, etc.). Progress = `answered.size / TOTAL`.

---

*HtALR Profile Builder Instructions v1.0*
*Project: human to agentic language rosetta*
