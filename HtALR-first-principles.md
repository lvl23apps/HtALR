# HtALR — First Principles
## Human to Agentic Language Rosetta v1.0

A foundational methodology for reducing translation loss between human intent and agentic execution.

---

## Table of Contents

1. [The Problem](#1-the-problem)
2. [Principle 1 — Translation Burden](#2-principle-1--translation-burden)
3. [Principle 2 — Minimum Viable Specification](#3-principle-2--minimum-viable-specification)
4. [Principle 3 — Mode Detection](#4-principle-3--mode-detection)
5. [Principle 4 — Failure as Signal](#5-principle-4--failure-as-signal)
6. [The Stack](#6-the-stack)
7. [Design Implications](#7-design-implications)

---

## 1. The Problem

When a human sends a query to an agent, something is always lost in translation.

The human expresses intent in natural language — abstract, affective, partially formed, shaped by context the agent cannot see. The agent operates on tokens, probability, and pattern — precise, literal, statistically averaged toward the most common interpretation.

The gap between these two systems produces unoptimised execution paths. The agent either guesses wrong and ships confidently, asks clarifying questions serially until the user gives up, or defaults to an averaged output that serves no one particularly well.

This is the **designer's paradox** applied to human-agent interaction. A client commissioning a designer can't fully specify what they want — that's why they hired the designer. The brief is always incomplete. The real work is the negotiation toward clarity. The same dynamic plays out every time a human talks to an agent.

HtALR is a methodology for making that negotiation explicit, persistent, and efficient. It operates like a shell profile — loaded at session start, shaping all subsequent interactions. It distributes the translation burden intelligently, builds a shared compressed language between human and agent, and makes failure fast, cheap, and recoverable.

---

## 2. Principle 1 — Translation Burden

### Statement

> The translation burden should be distributed according to contextual advantage. The human expresses intent in whatever form is natural. The protocol surfaces only the ambiguities the agent cannot resolve from context. The agent owns the translation from minimum specification into execution.

### Unpacked

Translation loss happens at the interface between two information processing systems operating at different levels of abstraction. The human has context on feeling and desired outcome. The agent has context on what is executable and what is ambiguous. Neither party should carry the full translation burden.

**Position A — human-side:** The human learns structured prompt engineering. Predictable, but it inverts the value proposition of AI. A cognitive tax on the person the tool is meant to serve.

**Position B — agent-side:** The agent infers everything from natural language input. Zero friction for the human, but this is precisely where the unoptimised path problem lives. With a massive solution space and underspecified input, the agent either guesses or loops.

**Position C — shared protocol:** A lightweight handshake before execution. The human provides natural input. The protocol surfaces minimum necessary disambiguation. The agent confirms its interpretation, then acts. This is the HtALR model.

### Design implication

The profile builder is the handshake mechanism. The tier system reflects the principle directly — Tier 1 captures the highest-leverage disambiguation with minimum friction. Tier 2 goes deeper only when the user chooses to invest.

---

## 3. Principle 2 — Minimum Viable Specification

### Statement

> The minimum viable specification is not a static threshold — it is an active seeking behaviour. The agent defaults to context-seeking before execution, trusts what has been explicitly declared, and infers only what remains. The profile-building process is a language negotiation: human and agent constructing a shared compressed language — flags, triggers, conventions, slang — that reduces translation loss over time. Both parties are computers speaking at different levels of abstraction. HtALR is the interface layer between them.

### The Intent Tuple

The atomic unit of human intent is not a request. It is a tuple:

```
intent = (desired state change, constraint set, confidence threshold)
```

| Element | Definition | Who owns it |
|---------|------------|-------------|
| Desired state change | What needs to be different after execution | User, per query |
| Constraint set | What the solution must and must not do | Profile, persistent |
| Confidence threshold | How certain does the result need to be | Profile + flags, overridable |

Most queries today express only the desired state change. The constraint set and confidence threshold are implicit, missing, or buried. HtALR makes all three explicit.

### The MVS Formula

```
MVS = specific desired state change
    + constraints that delta from agent defaults
    + one confidence signal (draft vs. verified)
```

Only the delta from defaults needs to be stated. If the user wants markdown and the agent defaults to markdown, that constraint is silent. The profile captures only divergence.

### Zero Trust Posture

HtALR defaults to **verify before acting, trust what's declared** — not trust by default and correct on failure. The agent actively closes specification gaps before execution rather than papering over them with inference.

This is not paranoid clarification. It is structured context-seeking. Once declared, the profile is trusted. Until declared, the agent seeks.

### Harmony and Shared Language

Human languages develop slang because repeated interaction between people who share context creates compression. "You know what I mean" becomes true — references land, shorthand works, meaning transfers with fewer words.

A well-built HtALR profile is the beginning of a compressed shared language between a specific human and a specific agent context. The flags are early slang. The trigger vocabulary is personalised slang. The persona sample is a dialect reference. The profile builder is not a settings panel — it is a language negotiation session.

The goal is harmony: a state where the agent's default behaviour is close enough to the user's actual preferences that the gap requiring explicit translation is small.

---

## 4. Principle 3 — Mode Detection

### Statement

> HtALR operates in two distinct modes: execution-mode, where the intent tuple is specifiable and the agent acts with fidelity; and discovery-mode, where the desired state change is unknown and the agent's job is to surface possibilities and capture reactions rather than execute against a specification. The protocol detects which mode it is in, operates differently in each, and recognises the transition moment when discovery becomes specification. Mode is declared explicitly via flags.

### The Two Modes

| | Execution-mode | Discovery-mode |
|---|---|---|
| User state | Knows what they want | Doesn't know yet |
| Tuple status | Specifiable | Desired state change unknown |
| Agent posture | Specify → act → deliver | Surface → react → narrow |
| User input | Desired state change | Reaction to artifacts |
| Success signal | Output matches specification | User gains clarity to specify |
| Failure mode | Wrong output | Premature commitment |

### Why Execution-Mode Thinking Fails in Discovery-Mode

Applying execution-mode protocol to a discovery-mode query produces two failure patterns:

**Premature commitment** — the agent picks an interpretation and executes confidently. The output is wrong in ways the user can't articulate. They know it isn't right but can't say why. Looks like success. Isn't.

**Infinite clarification** — the agent asks questions the user cannot answer because the answers don't exist yet. The protocol collapses because it requires specification that genuinely hasn't formed.

### What Discovery-Mode Needs

Not specification-seeking. **Possibility-surfacing.** The agent shows the user things they didn't know they could want, then watches what they react to. Reaction is data. Strong reaction in any direction narrows the space. Indifference signals a wrong axis.

### The Transition Moment

Discovery-mode ends not with a deliverable but with a specification — the point where the user has enough clarity to switch to execution-mode. The signal is "yes, like that but—". The "but" is the beginning of a constraint set. The agent treats it as a mode switch and shifts from possibility-surfacing to specification-building.

### Flag Implementation

| Flag | Mode | Behaviour |
|------|------|-----------|
| `-lfg` | Execution | Commit and execute, no clarifying questions |
| `-obey` | Execution | Literal execution only, no inference |
| `-toboldlygo` | Discovery | Explore possibility space, surface options, capture reactions. Success = clarity gained, not output delivered. Exits when direction is confirmed. |
| `-halp` | Bridge | Options before committing — execution-mode with multiple paths |

`-toboldlygo` makes mode **declared rather than inferred** — consistent with the zero-trust posture of Principle 2. The agent no longer guesses whether the user is exploring or executing.

---

## 5. Principle 4 — Failure as Signal

### Statement

> Failure in HtALR exists on a spectrum from undetected to anticipated. The protocol's goal is not to eliminate failure but to make it detectable, diagnosable, and recoverable. Every failure is a signal about where the profile is underspecified — the correction should be committed back to the profile. The failure posture always prioritises trust-preserving behaviour over confidence-projecting output.

### The Failure Spectrum

| Type | Description | Cost |
|------|-------------|------|
| Undetected | Output is wrong, neither party knows | Highest — user acts on bad information |
| Detected, unknown cause | User knows something is wrong, can't diagnose | High — trust erodes, iteration is blind |
| Detected, known cause | Both parties can point to the divergence | Recoverable — fast iteration possible |
| Anticipated | Agent surfaces uncertainty before delivering | Lowest — redirect before cost is incurred |

HtALR's goal is to push all failures toward the right end of this spectrum.

### Failure Origins Mapped to the Tuple

| Tuple element | Failure type | Symptom |
|---------------|-------------|---------|
| Desired state change | Agent executed against wrong goal | Output looks plausible but misses the point |
| Constraint set | Agent violated unstated constraint | Format wrong, tone off, scope exceeded |
| Confidence threshold | Agent shipped when it should have flagged | High-stakes output without uncertainty markers |

### The Detection Problem

LLM outputs are fluent and confident regardless of correctness. The signal that something is wrong is weak. HtALR addresses this two ways:

**Declared specification creates a comparison baseline.** When the constraint set is explicit, a violation is visible. The user points to the declared constraint that was broken rather than intuiting that something feels off.

**Uncertainty markers make model confidence legible.** The `-d` flag and failure mode profile setting surface the agent's confidence inline. The user sees where the agent is guessing before acting on the output.

### Failure as Learning Loop

Every failure is a signal about where the profile is underspecified. A constraint violation means that constraint wasn't in the profile. A wrong inference means the confidence threshold wasn't clear. A premature commitment means `-toboldlygo` should have been set.

The correction that follows a failure is exactly the kind of delta that should be committed to the profile:

```
failure → correction → -makeitso → profile update → shared language improves
```

This is the mechanism by which the human-agent shared language gets more precise over time. Failure isn't a breakdown of the protocol — it's the protocol's primary learning input.

### Trust Posture

Failure damages trust asymmetrically:
- A confident wrong answer damages trust more than an uncertain right answer
- An undetected failure damages trust more than a detected one
- An unrecoverable failure damages trust more than one the user can correct

HtALR always prioritises **trust-preserving failure** over **confidence-projecting output.** Surface uncertainty. Flag assumptions. Make recovery paths obvious.

---

## 6. The Stack

The four principles form a coherent architecture. Each builds on the one before.

| # | Principle | Core question | Answer |
|---|-----------|---------------|--------|
| 1 | Translation burden | Who does the translation work? | Shared — each party owns what they have context for |
| 2 | Minimum viable specification | What needs to be specified, and how much? | The intent tuple — seek context, trust declarations, build shared language |
| 3 | Mode detection | How does the agent know how to behave? | Detect execution vs discovery — declare explicitly via flags |
| 4 | Failure as signal | What happens when it breaks? | Make failure detectable, commit corrections back to profile |

### How the principles interact

Principle 1 establishes **who**. Principle 2 establishes **what and how much**. Principle 3 establishes **how the agent behaves given what it knows**. Principle 4 establishes **what happens when the system produces a wrong answer and how that makes the system better**.

Together they describe a protocol that is not static — it learns, compresses, and becomes more efficient with use. The shared language deepens. The translation loss narrows. The profile becomes a more accurate model of the user's actual preferences over time.

---

## 7. Design Implications

Every HtALR tool and convention traces back to one or more of these principles.

| Tool / convention | Principle(s) |
|-------------------|-------------|
| Profile builder — Tier 1 | P1 — minimum friction handshake |
| Profile builder — Tier 2 | P2 — full tuple capture |
| Flag system | P2 (confidence signal), P3 (mode), P4 (recovery) |
| `-toboldlygo` | P3 — explicit discovery-mode entry |
| `-makeitso` | P4 — commit corrections to profile |
| `-d` flag | P4 — uncertainty markers |
| Trigger vocabulary | P2 — personalised shared language |
| Commit log | P4 — failure learning loop, traceable history |
| Skills builder | P2 — MVS for reusable agent task modules |
| Zero-trust posture | P2 — verify before acting |

---

*HtALR First Principles v1.0*
*Project: human to agentic language rosetta*
