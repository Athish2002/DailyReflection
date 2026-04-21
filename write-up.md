# Write-Up: Design Rationale for the Daily Reflection Tree

---

## Why These Specific Questions

The hardest design constraint in this assignment is the prohibition on free text. Every question must offer fixed options that genuinely capture the *spectrum* — options that a tired person at 7pm would actually choose between, not options that telegraph the "right" answer.

**The opening question ("weather report")** is intentionally low-stakes and metaphorical. Direct questions like "was today productive?" activate self-assessment defenses — people anchor on how they *should* have felt, not how they *did*. A weather metaphor is disarming. It invites honesty before the employee knows they're being asked about agency.

**Axis 1 questions** are designed around Rotter's insight that locus of control is revealed in *attribution*, not in self-report. I don't ask "do you feel in control?" I ask "what made it happen?" and "what was your first move?" The options map cleanly to internal attribution (prepared, adapted, pushed through, found one thing to control) vs. external attribution (luck, waited, blocked). Crucially, I split the entry question by whether the day was light or heavy — because asking "what drove your success?" to someone who had a stormy day is tone-deaf and will produce defensive answers. The tree meets the employee where they are.

**Axis 2 questions** are the most psychologically delicate. Entitlement is invisible to the person holding it — Campbell et al.'s research shows people with high psychological entitlement consistently underestimate their sense of deserving. So I don't ask "did you feel entitled?" I ask "which of these feels most true?" with options that indirectly surface the entitlement/contribution distinction. The follow-up on the entitlement path asks what they *did* with the frustration — this is where Organ's OCB framework is useful. Option B ("quietly dialled back") is a classic entitlement response that few people would identify as entitlement in themselves. By naming it without shaming it, the tree makes it chooseable — and therefore visible.

**Axis 3 questions** are grounded in Maslow's late-career argument that self-transcendence is the highest order of human motivation — above self-actualization. Most people's mental models of their workday are almost entirely self-referential. The question "who else is in that picture?" introduces a frame the employee may not have considered at all. The follow-up on the self-centric path is gentle: "did someone cross your mind, even briefly?" This isn't designed to make them feel bad — it's designed to surface a small flash of other-awareness that may already be there but unnoticed.

---

## Branching Design and Trade-offs

**Key architectural choice: Entry path vs. accumulated signal routing**

For Axes 1 and 3, the first question routes the employee to a tailored follow-up (INT/EXT path, SELF/OTHER path). The follow-up then adds a second signal. The final decision node checks the *accumulated tally*, not just the last answer. This means an employee who enters on the "sunny day / agency" path but reveals reactive, unintentional behavior in Q2 will receive the mixed reflection — not the internal one. This is more honest than routing purely on first impressions.

For Axis 2, I use the same accumulated-tally approach, with a deliberate tiebreak toward entitlement. The reasoning: contribution-dominant employees won't be harmed by hearing the entitlement reflection (it asks them to look harder at what they gave, which they can easily do). Entitlement-dominant employees *need* the entitlement reflection, and routing a tie to contribution would miss the point.

**Trade-off: Three bridge variants vs. one**

I created three bridge nodes for the Axis 1 → 2 transition (one per reflection outcome) instead of a single bridge. This adds verbosity to the tree file but makes the bridge text contextually appropriate. A person who just received the "you kept a hand on the wheel" reflection doesn't need the same bridging language as someone who received the "a lot felt like it was happening to you" reflection. The extra nodes are worth it for the conversational feel.

**Trade-off: Mixed reflections**

Adding `_MIX` reflection nodes for Axes 1 and 3 (and allowing the tally logic to route there) adds complexity but increases psychological accuracy. Most people are not cleanly internal or external — their day contains both. The mixed reflection honors that reality without diluting the axis.

---

## Psychological Sources

- **Rotter (1954)**: The locus of control construct. Internal vs. external attribution is surfaced through behavioral questions, not self-report, following Rotter's original measurement approach.
- **Dweck (2006)**: Growth mindset — the belief that effort drives development — is threaded into the internal locus questions (prepared, adapted, pushed through) vs. fixed/luck-oriented ones.
- **Campbell et al. (2004)**: Psychological entitlement as a *stable trait* invisible to its holder. This informs the indirect questioning strategy on Axis 2.
- **Organ (1988)**: Organizational citizenship behavior as discretionary, beyond-role effort. The contribution options (helped, stepped in, held level, spoke up) are drawn from OCB taxonomy.
- **Maslow (1969)**: Self-transcendence as the movement from "what do I need?" to "what does the world need from me?" The radius framing of Axis 3 is a direct operationalization.
- **Batson (2011)**: Perspective-taking as cognitive empathy — not sympathy, but actively imagining another's experience. The Axis 3 questions probe whether this cognitive act occurred today.

---

## What I'd Improve with More Time

**1. A fourth axis: Learning orientation.** The three required axes don't capture *what the employee will do differently tomorrow*. A closing axis — "Was today a data point for growth?" — would close the loop from reflection to intention, grounded in Dweck's learning vs. performance goal framework.

**2. Weighted signals.** Currently, each option contributes exactly ±1 to a tally. A more psychologically calibrated system would weight some signals more heavily — e.g., "I quietly dialled back my effort" is a stronger entitlement signal than "I felt frustrated others weren't pulling their weight." This would require defining signal weights per option, which is straightforward in the data schema but needs careful calibration.

**3. Longitudinal pattern tracking.** The tree currently treats each session as independent. In production, a thin persistence layer could accumulate axis-level data across sessions, so that the summary says: "Over the past week, you've been consistently external on Axis 1. Here's what that pattern might be worth noticing." This doesn't require an LLM — just tallies stored across sessions.

**4. Testing with real employees.** The most important improvement is empirical: have 10–15 people walk through the tree and debrief afterward. The questions that feel thought-provoking in design often reveal themselves as ambiguous or uncomfortable in practice. Iteration on option wording is cheap and high-leverage.
