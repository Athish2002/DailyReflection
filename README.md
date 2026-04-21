# The Daily Reflection Tree — README

A fully deterministic, end-of-day reflection tool. **No LLM is called at runtime.** Every conversation path is determined entirely by the employee's selections from fixed options.

---

## Repository Structure

```
/tree/
  reflection-tree.json   ← Part A: the complete tree data (38 nodes)
  tree-diagram.md        ← Part A: Mermaid visual diagram + path tables

write-up.md              ← Part A: design rationale (psychological sources,
                            branching decisions, trade-offs, future improvements)

README.md                ← this file
```

---

## How to Read the Tree File

`reflection-tree.json` is structured in two top-level sections:

### `meta`
Contains everything needed to *interpret* the tree without running code:
- **`axes`** — the three psychological axes, their sources, and pole definitions
- **`signals`** — what each signal string means and how it mutates state
- **`state_schema`** — the complete state object the agent maintains
- **`dominant_rules`** — how `axis.dominant` is computed for routing (critical for decision nodes)
- **`interpolation_rules`** — how `{placeholder}` strings in text fields are resolved
- **`node_types`** — behavior of each node type
- **`routing_condition_syntax`** — how to read condition strings in decision nodes

### `summary_data`
A lookup table used by the SUMMARY node:
- **`axis1/2/3`** — maps each pole to a `label` and `summary` sentence for interpolation
- **`closing_insights`** — 9 entries (8 specific combinations + 1 default) that produce a closing paragraph based on all three dominant axes

### `nodes`
An array of 38 node objects. Each node has:

| Field | Present on | Meaning |
|-------|-----------|---------|
| `id` | all | Unique identifier |
| `parentId` | most | Primary parent in tree hierarchy (null for shared/multi-parent nodes) |
| `parents_note` | multi-parent | Documents all nodes that can reach this node |
| `type` | all | One of: start, question, decision, reflection, bridge, summary, end |
| `text` | all except decision | What the employee sees. May contain `{placeholder}` strings. |
| `options` | question | Array of `{id, label, text, signal}` objects |
| `next` | start, question, reflection, summary | Default next node after this one |
| `target` | bridge | Jump target (may cross tree hierarchy) |
| `routing` | decision | Ordered array of `{condition, target}` objects |
| `default_target` | decision | Fallback if no condition matches |
| `signal` | question options | Which state tally to increment when this option is selected |
| `axis` | question, reflection | Which axis this node belongs to (documentation only) |
| `note` | decision, some others | Human-readable explanation of routing logic |

---

## How to Trace a Path

To trace any conversation path through the tree without running code:

1. Start at node `START`. Its `next` → `INTRO_Q`.
2. At any **question** node: pick an option. Record `state.answers[nodeId] = optionId`. Apply the selected option's `signal` (increment the corresponding tally). Follow `next` to the next node.
3. At any **decision** node: evaluate `routing` conditions top-to-bottom against current state. Follow the first matching `target`. (If no match: use `default_target`.)
4. At any **reflection** or **bridge** node: follow `next` or `target`.
5. At **SUMMARY**: resolve all `{placeholder}` strings using `summary_data` and current state.
6. At **END**: session complete.

### Example Path — "Victor/Contributor/Altrocentric" persona

| Step | Node | Action |
|------|------|--------|
| 1 | START | Auto-advance |
| 2 | INTRO_Q | Select B ("Overcast") → state.answers.INTRO_Q = 'B' |
| 3 | INTRO_D | B in ['A','B'] → go to A1_Q1_LIGHT |
| 4 | A1_Q1_LIGHT | Select A ("I prepared") → axis1.internal=1 |
| 5 | A1_D1_LIGHT | A in ['A','B','D'] → go to A1_Q2_INT |
| 6 | A1_Q2_INT | Select B ("Trust someone placed in me") → axis1.internal=2 |
| 7 | A1_D2_INT | internal(2) ≥ external(0) → go to A1_R_INT |
| 8 | A1_R_INT | Read reflection. next → BRIDGE_1_2_A |
| 9 | BRIDGE_1_2_A | Auto-advance. target → A2_Q1 |
| 10 | A2_Q1 | Select C ("Stepped into a gap") → axis2.contribution=1 |
| 11 | A2_D1 | C in ['A','C'] → go to A2_Q2_CONTRIB |
| 12 | A2_Q2_CONTRIB | Select A ("Obvious what to do") → axis2.contribution=2 |
| 13 | A2_D2_CONTRIB | contribution(2) > entitlement(0) → dominant='contribution' → A2_R_CONTRIB |
| 14 | A2_R_CONTRIB | Read reflection. next → BRIDGE_2_3_A |
| 15 | BRIDGE_2_3_A | target → A3_Q1 |
| 16 | A3_Q1 | Select C ("A specific colleague") → axis3.altrocentric=1 |
| 17 | A3_D1 | C in ['B','C','D'] → go to A3_Q2_OTHER |
| 18 | A3_Q2_OTHER | Select B ("Maybe, but I tried") → axis3.altrocentric=2 |
| 19 | A3_D2_OTHER | altrocentric(2) > selfcentric(0) → dominant='altrocentric' → A3_R_OTHER |
| 20 | A3_R_OTHER | Read reflection. next → SUMMARY |
| 21 | SUMMARY | Resolve: axis1.label="more internal", axis2.label="contribution-oriented", axis3.label="outward-facing". closing_insight matches first condition. |
| 22 | END | Session complete |

---

## Psychological Framework

The three axes are grounded in peer-reviewed psychology, applied in sequence:

| Axis | Spectrum | Core Theory | Key Signal |
|------|----------|-------------|------------|
| 1 — Locus | Victim ↔ Victor | Rotter (1954) Locus of Control; Dweck (2006) Growth Mindset | Did they see their own hand in today's events? |
| 2 — Orientation | Entitlement ↔ Contribution | Campbell et al. (2004) Psychological Entitlement; Organ (1988) OCB | Was their attention on what they gave or what they were owed? |
| 3 — Radius | Self-Centrism ↔ Altrocentrism | Maslow (1969) Self-Transcendence; Batson (2011) Perspective-Taking | Was their frame self-referential or did it include others? |

The axes build on each other intentionally: someone who recognizes their agency (Axis 1) is primed to consider what they *chose to give* (Axis 2), which naturally opens the question of *who else* was in the frame (Axis 3).

---

## Design Constraints (All Met)

| Constraint | Status |
|-----------|--------|
| No LLM at runtime | ✅ Zero API calls in the tree. All text is static or template-interpolated. |
| Fully deterministic | ✅ Same answers → identical path → identical reflection, every time |
| Fixed options only | ✅ All 11 question nodes have 4 fixed options. No free text. |
| No moralizing | ✅ Reflections name, reframe, and invite. They do not grade or shame. |
| Three axes in sequence | ✅ Axis 1 → Bridge → Axis 2 → Bridge → Axis 3 → Summary |
| 25+ nodes | ✅ 38 nodes |
| 8+ question nodes | ✅ 11 question nodes |
| 4+ decision nodes | ✅ 11 decision nodes |
| 4+ reflection nodes | ✅ 8 reflection nodes |
| 2+ bridge nodes | ✅ 5 bridge nodes |
| 1+ summary node | ✅ 1 summary node with 9 closing insight variants |
