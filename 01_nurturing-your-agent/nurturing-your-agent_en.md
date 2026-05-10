---
title: Nurturing your agent — how to train an AI through interaction
version: v0
date: 2026-04-26
language: EN
authors: "[Author]"
---

> **Note:** This is a sanitized version for public sharing. Personal names,
> company names, and identifying details have been replaced with generic labels.
> The methodology and patterns are fully preserved.

# Nurturing your agent
## How to train an AI through day-to-day interaction

---

## 1. Why this guide

An out-of-the-box AI agent does not know its operator, is unaware of house conventions, and cannot tell what counts as a professional deliverable in a specific context. Value emerges when the agent is **tuned over time** by whoever uses it. This guide documents the tuning process used with the CEO (the virtual CEO of the Owner's AI-powered Enterprise) and can serve as a template for other operators.

Three premises:

1. **Tuning is a process, not an event.** It is not solved by a long initial instruction. It is built over weeks and months.
2. **Human feedback is the raw material.** Whatever the Owner says, corrects, approves or rejects becomes a rule.
3. **Agent memory must be structured.** Otherwise feedback dissipates and the same corrections repeat.

---

## 2. The three channels of tuning

### 2.1 Direct feedback (correction)

The Owner flags something that went wrong:

> "You must check the visual formatting before delivering."

> "PT-PT diacritics are not optional."

> "Never regenerate a DOCX after I have adjusted the layout."

**What the CEO does with it:**
1. Captures the rule **together with the "why"** (why is the Owner correcting).
2. Writes a `feedback`-type memory at `~/.claude/projects/.../memory/feedback_<topic>.md`.
3. Adds a line to `MEMORY.md` (the queryable index loaded each session).
4. Applies the rule from the next opportunity — without the Owner having to repeat it.

### 2.2 Indirect feedback (confirmation)

Subtler. The Owner accepts a non-obvious choice without objection, or states explicitly "that was the right call".

> "Yes, the single bundled PR was the right move here."

> "Excellent." (after receiving output in a new convention)

**Why this matters:** if the CEO only logged corrections, it would drift conservative and cautious — losing validations that are also instruction. Logging confirmations consolidates patterns that work.

### 2.3 Meta-feedback (architecture)

The Owner redesigns the system:

> "I want to hire a new agent for X."
> "Create a folder under Projects for this."
> "I want a parallel log of every interaction."
> "Rename [previous name] to the CEO."

**What this does:**
- Adds/removes agents from the roster
- Creates new project folders with their own conventions
- Installs new hooks or skills
- Refactors memory (e.g. the 2026-04-10 rebrand)

Meta-feedback requires immediate execution and updates to `CLAUDE.md` or `rules/`.

---

## 3. Anatomy of a good correction

The Owner's most useful corrections follow three patterns:

### Pattern A — "Rule + why + stake"

> "Open every PPTX visually before delivery. On the [Company] file I opened it and the text overlapped. If it happens again we lose the client."

Clear rule, incident history, concrete stake. The memory becomes rich and the CEO knows when to flex the rule at edge cases.

### Pattern B — "Example as rule"

> "Font is Arial. Always." (after seeing Calibri in a DOCX)

Short correction, referring to a specific artefact. The CEO generalises to **all** future DOCX/PPTX.

### Pattern C — "Preference vs requirement signalling"

> "I prefer the Micah style for avatars." (preference)
> "PT-PT diacritics are mandatory in deliverables." (requirement)

Distinguishing the two keeps the CEO from treating preferences as hard law or requirements as hints.

---

## 4. Canonical cases (already instituted)

Each case below represents a real correction → persistent rule → automatic application in later sessions.

### Case 1 — Visual validation before delivery

**Trigger:** 2026-04-15, [Company] PPTX with overlapping text passed the Quality Gate because the CEO only verified that shapes existed, not the final render.

**Rule:** PPTX/DOCX/PDF must be opened visually (or rendered to PNG/PDF) before delivery. Added as blocking check #12 of the `quality-gate` skill.

**Memory:** `feedback_visual_validation_pptx_docx.md`

### Case 2 — PT-PT diacritics

**Trigger:** 2026-04-13, deliverable without diacritics sent to third parties. Flagged as "immediately recognisable as machine-generated".

**Rule:** final deliverables with full diacritics. Internal system files (memories, scripts) can remain without.

**Memory:** `feedback_pt_pt_accents_in_deliverables.md`

### Case 3 — Default font Arial

**Trigger:** corporate templates defaulting to Calibri.

**Rule:** Arial (or Arial Narrow for dense text) is the default in every DOCX/PPTX/PDF, even when the base template suggests another face.

**Memory:** `feedback_font_preference.md`

### Case 4 — Naming with last-updated date

**Trigger:** 2026-04-10, confusion between stale and current versions of the enterprise-technical-blueprint.

**Rule:** when updating a deliverable, rename to `YYYY-MM-DD_<slug>.<ext>` (last-updated date) + `**Last updated:**` line in internal metadata + version bump.

**Memory:** `feedback_deliverable_naming.md`

### Case 5 — Surgical editing of Owner-edited DOCX

**Trigger:** Owner said "you misread this" after the CEO regenerated a DOCX from scratch when asked to "fix this" on his version (which already had layout adjustments).

**Rule:** when the Owner says "I already made v3" or "I adjusted the layout", the CEO applies run-level fixes via `python-docx` on his file, preserving bold/italic and layout. Never regenerate from scratch.

**Memory:** `feedback_owner_edits_docx_directly.md`

### Case 6 — CEO callouts 🔶/🔷

**Trigger:** 2026-04-11, Owner worried that rereading class notes months later he might attribute to the lecturer interpretations that were the CEO's.

**Rule:** in class notes, all CEO interpretation goes in an orange box (heavy) or blue box (light), with specific bibliography. Plain text = what the lecturer actually said.

**Memory:** `feedback_ceo_callout_convention.md`

### Case 7 — Gamma for slides

**Trigger:** 2026-04-13, Owner confirmed preference for Gamma over Reveal.js / python-pptx for any presentation.

**Rule:** default for slides is producing **structured Gamma prompts** (10-slide limit), not native HTML/PPTX. Exceptions: interactive dashboards (front-end agent) or standalone infographics.

**Memory:** `feedback_gamma_preferred_slides.md`

### Case 8 — Sanitization for public sharing

**Trigger:** 2026-04-16, Owner decided to share guides (Enterprise Blueprint, NYA, StylesBook, WhoAmI) with third parties. Audit revealed 100+ occurrences of personal data, names with copyright issues, agent names, institutions, and even national ID/IBAN numbers in older files.

**Rule:** before sharing any document, create a separate `_PUBLIC` version (original untouched). Apply substitution table: CEO (orchestrator), Owner (proprietor), agent names→role only, companies→[Company], institutions→[University]/[Hospital], personal data→removed. Automated spot-check (grep/script) mandatory before delivery. For DOCX/XLSX, use python-docx/openpyxl.

**Memory:** `feedback_public_sanitization_process.md`

### Case 9 — Quality Gate as mandatory process

**Trigger:** 2026-04-11, Owner evaluated hiring an external professional editor. Cost-benefit analysis showed that an internal skill (`/quality-gate`) with a structured checklist was more effective. Decision: Option C — internal skill with monthly review.

**Rule:** before delivering any deliverable to the Owner or third parties, run `/quality-gate` on the file. Checklist: 8 blocking structural checks (naming, metadata, no adjacent duplicates, correct image flow, callout convention, consistent language, valid refs, proper table/box formatting) + 3 content warnings (density, actionability, coherence with style book). Minimum score: 9/11 + zero blockers. For quality-sensitive deliverables (visual, client-facing, publication): score >= 10/11, suggest review to Owner, max 2 correction rounds. Result logged in DB (`deliverables` table with `quality_score`, `iterations_to_satisfaction`, tags) for monthly KPI tracking.

**Memory:** `project_editor_hire_evaluation.md`

### Case 10 — Agent hiring pipeline

**Trigger:** 2026-04-16, Owner requested hiring a Research Grants & EU Funding Specialist. Process followed a 3-phase pipeline with two specialized agents (Skills Researcher + HR Director) and full DB logging.

**Rule:** hiring a new agent always follows 3 sequential phases: (1) the Skills Researcher researches competencies, frameworks, and ideal profile, delivers Skills Profile Report to `.claude/db/research/`; (2) the HR Director designs the full persona (name, age, background, DISC, attributes) and creates the agent file in `.claude/agents/`; (3) the CEO registers the agent in the DB (`insert_agent`), updates the Team Roster in `CLAUDE.md`, updates Smart Routing in `.claude/rules/smart-routing.md`, and confirms to the Owner. The Skills Researcher brief must include full functional scope, target programmes/domains, expected KPIs, and interactions with other agents. The HR Director brief must include the Skills Profile + scope + KPIs. Each phase is logged as a task in the DB.

**Memory:** procedural — documented in `CLAUDE.md` (section "Agent hiring procedure")

---

## 5. Memory architecture

The system uses three tiers (documented in `CLAUDE.md`):

| Tier | Purpose | Access |
|------|---------|--------|
| **Hot** | Owner profile, preferences, rules. Loaded on EVERY session. | `memory-hot.md` |
| **Warm** | Project decisions, learned patterns. Queryable. | `memory_manager.py get_warm` |
| **Cold** | Historical archive. FTS5 search. | `memory_manager.py query` |

**Automatic promote/demote:**
- Warm → Hot when accessed ≥5 times
- Warm → Cold when stale >30 days and confidence <0.5
- Hot → Warm when confidence <0.3

This prevents obsolete memories from conditioning the agent's behaviour.

---

## 6. What does NOT go into memory

Equally important. Claude Code's auto-memory makes the exclusions explicit:

- **Code patterns, architecture, paths** — derivable from the current repo
- **Git history** — `git log` / `git blame` are authoritative
- **Debugging recipes** — the fix is in the code, context is in the commit
- **Ephemeral state** — in-progress work, current conversation context

When the Owner asks to "save" something from these categories, the CEO asks **what was surprising or non-obvious** and saves only that.

---

## 7. Operational flow — the full cycle

```
1. Owner interacts with the CEO
2. Feedback is given (direct, indirect or meta)
3. The CEO captures the "why"
4. Writes feedback_<topic>.md with Rule + Why + How to apply
5. Updates MEMORY.md (index)
6. Applies it at the next opportunity
7. If misapplied, back to step 2 (iterative correction)
8. Old memories decay automatically (confidence −5%/30 days)
```

A typical session runs this cycle 3–10 times.

---

## 8. Signs of good tuning

- **The Owner repeats less.** Corrections that showed up 3×/week now show up 1×/month.
- **Memories are specific**, not generic ("Arial font" instead of "care about formatting").
- **Confirmations exist, not only corrections.** The Owner validates patterns explicitly ("that was the right move").
- **The agent anticipates.** Applies rules to domains where they have not yet been explicitly stated, by analogy.
- **Architecture evolves.** New agents, skills, rules emerge as limits surface.

---

## 9. Signs of poor tuning

- Owner repeats the same correction >3 times — memory is not being consulted or was poorly written.
- Agent applies a rule in the wrong context — memory lacks edge cases.
- Owner has to line-validate every deliverable — the Quality Gate isn't filtering.
- Deliverables arrive without the "why" — agent is executing without understanding.

---

## 10. Recommendations to the operator (Owner)

1. **Always give the why with the correction.** A rule without its reason becomes blind rigidity.
2. **Distinguish preference from requirement** when it is obvious to you but may not be to the agent.
3. **Confirm patterns that worked** — not only the ones that failed.
4. **Ask for visible tracking** (project folder, log file) when the topic is complex and multi-session. Conversations dissolve; files persist.
5. **Review MEMORY.md periodically** — if entries no longer make sense, remove them.
6. **Accept decay.** A memory the agent stopped using may be obsolete — let it sink to cold.

---

## 11. Measuring maturity — when is the agent "ready"?

Tuning is not infinite. As the system accumulates patterns, sessions generate fewer and fewer new rules — interactions shift from corrections to confirmations. This plateau is measurable.

### 11.1 Core metric: New Insight Rate (NIR)

```
NIR = new patterns in session / total log entries in session × 100
```

Each entry in the parallel log (`registo-interacoes/YYYY-MM-DD_log.md`) is classified as:
- **New pattern** — generates a rule that did not exist in the guide
- **Confirmation** — validates an already-documented rule (reinforces confidence, does not alter the guide)

### 11.2 Expected phases

| Phase | Typical NIR | What happens |
|-------|------------|--------------|
| **1 — Bootstrap** | 70–100% | Almost every interaction generates a new rule. The agent is learning everything for the first time. |
| **2 — Consolidation** | 30–50% | Many confirmations, some discoveries. The core rule set is forming. |
| **3 — Plateau** | <20% | Most entries confirm existing patterns. New interactions refine but do not transform. |

### 11.3 Maturity threshold for publication

The guide is ready to share when **two criteria are met simultaneously:**

1. **3 consecutive sessions with NIR < 20%** — the agent has stabilised; new sessions do not produce fundamentally new rules
2. **At least 25 generalisable patterns** in the guide — enough coverage for another operator to replicate the process

When both are reached, the CEO signals: *"The NYA guide has reached plateau. Ready for final review and sharing."*

### 11.4 Per-session metrics block

At the end of each daily log, record:

```
## NYA Metrics
- Entries: N
- New patterns: N
- Session NIR: XX%
- Accumulated patterns in guide: N
- Consecutive sessions with NIR < 20%: N/3
```

### 11.5 Why this metric

The novelty rate directly measures what matters: **the system's learning velocity**. When NIR drops, the agent has already captured the essence of how the Owner works. Iterating indefinitely adds no value — and the diminishing-returns curve is visible in the data.

This is analogous to the learning curve in psychometrics: the first items in a test reveal more information about the examinee than the last ones. Here, the first sessions reveal more about the Owner than the later ones.

---

## Appendix A — Internal references

- `CLAUDE.md` — the CEO's persona and rules
- `.claude/rules/smart-routing.md` — five routing lanes
- `.claude/rules/writing-styles.md` — Owner's Style Book
- `.claude/skills/quality-gate/` — pre-delivery checklist
- `~/.claude/projects/.../memory/MEMORY.md` — memory index
- `Projects/nurturing-your-agent/registo-interacoes/` — parallel feedback log

## Appendix B — feedback memory template

```markdown
---
name: <short title>
description: <one line, specific>
type: feedback
originSessionId: <uuid>
---
<rule>.

**Why:** <concrete reason, incident if any>

**How to apply:** <when, where, how, exceptions>
```
