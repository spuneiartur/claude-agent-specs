---
name: anti-slop
description: "Human-writing editorial pass for AI-polished, corporate-polished, rhetorically over-constructed, or fact-thin prose. Invoke when the user types /anti-slop or /human-writing, asks to humanize writing, remove AI writing, make prose sound less AI, less polished, less corporate, less like a comms deck, clean up parallelisms, flatten the rhetoric, remove tricolons, or make this sound like someone actually thought it through. Applies to comms drafts, narrative documents, briefings, op-eds, position papers, web copy, and other prose where polish, rhythm, generic confidence, or invented specificity is substituting for judgment."
---

# Anti-Slop: Editorial Detox

Codex adaptation: use this automatically for prose intended to leave Bolt, as required by `AGENTS.md`. `/anti-slop` is the canonical invocation; `/human-writing` is the public-facing concept and trigger phrase. Internal notes, `TASKS.md`, and log entries are exempt unless the user explicitly asks.

This skill turns AI-polished, corporate-polished, or rhetorically over-constructed prose into writing that sounds like a serious person with judgment wrote it. The goal is not to make prose casual, quirky, or artificially "human". The goal is to restore specificity, texture, and conviction while keeping the appropriate professional register.

The original human-writing job remains central: catch places where rhythm and symmetry are doing more work than the content. But the broader anti-slop test is now:

**Is this sentence making a real claim, or just sounding like one?**

---

## Modes

Detect the mode from the user's request.

- **Audit + Rewrite** *(default)* — flag the main problems, explain them, then provide revised text.
- **Audit only** — use when the user asks to scan, diagnose, flag, review, or not rewrite.
- **Rewrite only** — use when the user asks for a clean final version and does not need commentary. Keep the rewrite faithful; do not add a long diagnosis.
- **Minimal edit** — use for sensitive, legal, executive, or stakeholder-owned prose where meaning and voice must be preserved tightly. Fix only what weakens the prose.
- **Checklist only** — use when the user explicitly asks for a checklist, line-by-line scan, manual edit notes, or Google Docs-style editing queue. Output flagged items in document order with quote + fix. Do not include score, severity, or a full rewrite unless asked.

If the user provides a file path and asks you to edit the source file, follow the normal Codex editing rules and get explicit approval before changing source prose when the input is over 500 words.

---

## Diagnostic Categories

### 1. Rhetorical Over-Construction

These are the original human-writing patterns. They remain first-class issues.

- **Corporate tricolon** — three tidy items that build to false weight: "fast, safe, sustainable"; "listen, learn, act".
- **Anaphora as manifesto** — repeated openers that accumulate rhythm instead of meaning: "We believe... We believe...".
- **"Not X, but Y" frame** — a false contrast or inflated pivot: "Not a company, but a movement."
- **"X. Not Y." correction pair** — denying an objection nobody raised: "This is about safety. Not optics."
- **Balanced antithesis as throat-clearing** — "The challenge isn't speed. It's trust."
- **Positive-declarative pair** — two short sentences where the second restates or inverts the first without advancing the thought.
- **Symmetrical bullets** — bullets with the same length, structure, and rhythm even when the content differs.
- **Stacked parallel clauses** — "a platform that connects..., empowers..., reimagines..." where each clause is vague.
- **Correlative inflation** — "not just X, but Y", "not only X, but also Y", "not because X, but because Y" when the contrast is decorative or avoids the direct claim.
- **Staccato triad** — three clipped declarative sentences in a row that create momentum without logic: "Documents become templates. Macros scale intelligence. Knowledge propagates."
- **Formulaic negation** — "No X. No Y. Just Z." when it sounds like a campaign line rather than a useful distinction.
- **Dramatic reveal** — suspense built around an obvious or unsupported point: "The finding alone changed everything."

**Earned exception:** keep parallelism when the ideas are genuinely parallel, the contrast is real, and the rhythm clarifies something that would still matter without the structure. Put these in **What is working**, not in Major issues.

### 2. AI and Corporate Tells

Flag language that reads like generated confidence rather than human judgment:

- inflated importance: `pivotal`, `transformative`, `crucial`, `vital`, `significant`, `noteworthy`, `groundbreaking`, `cutting-edge`, `revolutionary`
- fake-depth verbs: `showcasing`, `reflecting`, `underscoring`, `highlighting`, `leveraging`, `unlocking`, `unleashing`, `harnessing`, `empowering`, `illuminating`
- empty scale words: `ecosystem`, `landscape`, `platform`, `solutions`, `stakeholders`, `impactful`, `holistic`, `scalable`, `future-proof`
- chatbot scaffolding: "It is important to note", "It is worth noting", "In today's rapidly evolving...", "In the ever-evolving landscape of...", "In the realm of...", "Let's explore", "Let's dive in", "At its core"
- AI-scent vocabulary: `delve`, `tapestry`, `reimagined`, `deep dive`, `navigate the complexities`, `nuanced`, `multifaceted`, `seamless`, `comprehensive`, `meticulous`, `bespoke`, `foster`, `spearhead`, `optimize`
- unnecessary formality: `utilize`, `plethora`, `myriad`, `commence`, `facilitate`, `optimal`, `prior to`, `subsequently`, `whilst`, `amongst`
- false enthusiasm: "Absolutely!", "Certainly!", "Great question", "That's a fantastic point", "I'd be happy to help"
- excess hedging: "Generally speaking", "It can be argued that", "To some extent", "Based on the information provided", overused `might`, `could`, `perhaps`, `arguably`, `potentially`, `somewhat`
- generic conclusions: "This marks an important step forward", "The future is bright", "Only time will tell", "One thing is clear", "The bottom line is", "Moving forward"
- over-signposted transitions: `moreover`, `furthermore`, `additionally`, `consequently`, `thus`, `hence`, `therefore`, `accordingly`, `notably`, `essentially`, `ultimately`, `indeed` when they add ceremony rather than logic

Do not ban words mechanically. Flag them when they let the writer avoid saying the specific thing.

### 3. Press-Release Prose

Flag PR language that creates distance from the claim:

- `serves as`, `boasts`, `features`, `is set to`, `aims to`, `is proud to announce`, `has emerged as`, `represents a significant milestone`, `stands as a testament to`, `plays a crucial role`
- vague attributions: "experts say", "industry leaders agree", "research shows" without naming the source
- vague authority: "studies show", "research indicates", "according to recent reports" without a named source, date, number, or link
- self-congratulation in place of reader value
- product descriptions that start with what the company did rather than what changes for the audience

### 4. Structural Neatness

Flag writing that is too clean to be credible:

- every paragraph roughly the same length
- every section built as a three-part framework
- headings that sound interchangeable
- lists that are complete-looking but not complete-thinking
- conclusions that merely restate the introduction in tidier language
- every bullet starts with the same bold-label-plus-colon structure
- too many sentences in the same 15-25 word band
- repeated participial endings: "X does Y, enabling Z", "X creates Y, helping Z"
- exhausted metaphors: `treasure trove`, `double-edged sword`, `tip of the iceberg`, `cornerstone`, `uncharted waters`, `beacon`, `crossroads`, `catalyst`, `blueprint`, `symphony`, `mosaic`

### 5. Voice Flatness

Flag places where the prose has no human judgment:

- no concrete stakes
- no useful uncertainty or qualification
- no sense of what the writer thinks
- no visible tradeoff, tension, or decision
- all claims phrased at the same bland confidence level
- perpetual balance: "both sides present valid points", "reasonable people may disagree", "there are pros and cons" when the piece needs a view
- vague emotional or transformation claims: "the clarity came back in a different form" without saying what changed

The fix is not slang. The fix is specificity, asymmetry, and judgment.

---

## Required Passes

### 1. Substance Test

Before rewriting any weak sentence, ask:

- What claim is this sentence making?
- Could a smart reader disagree with it?
- What concrete thing is being said?
- Is the rhythm clarifying the argument or decorating it?
- If the rhetorical structure is removed, does a meaningful point remain?

If no meaningful point remains, rewrite from the underlying idea. Do not polish an empty sentence.

### 2. Evidence Boundary

Never invent names, numbers, dates, examples, sources, causality, product claims, outcomes, or stakes to make empty prose feel concrete.

When the source text lacks the facts needed for a strong rewrite:

- narrow the claim to what the text actually supports
- delete the empty claim if it adds no meaning
- use a visible placeholder such as `[specific evidence needed: customer proof, market, date, or number]`
- in Major Issues, label the fix as **needs source fact** rather than fabricating a confident replacement

The rewrite may make logic clearer, but it must not smuggle in unsupported evidence. Human writing is allowed to be unfinished when the thinking is unfinished.

### 3. AI Tell Sweep

Before rewriting, scan for the pattern library above:

- generic openers and chatbot frames
- AI-scent vocabulary and unnecessary formality
- stock templates and exhausted metaphors
- correlative constructions and unearned contrasts
- formal transition overuse
- vague authority claims
- staccato triads, `No X. No Y. Just Z.`, dramatic reveals, and clipped fragments
- false enthusiasm, excess hedging, perpetual balance, and corporate buzzwords
- generic conclusions

Use this as detection support, not as a word ban. Only flag a pattern when it weakens the claim, hides missing thinking, or makes the prose sound more certain than it is.

### 4. Voice Restoration

When rewriting:

- Replace generic confidence with specific conviction.
- Let some sentences be shorter, plainer, or less symmetrical.
- Keep the writer's professional register unless the user asks for a different voice.
- Use concrete nouns and visible stakes.
- Allow useful uncertainty: "the risk is", "the harder question is", "this only works if".
- Add texture through judgment and specificity, not jokes, slang, or performative warmth.
- Preserve visible placeholders where source facts are missing.

The revised text should feel less sanded down, not less serious.

---

## Output Format

### Score: Before

Rate the original text on a 1-100 scale for human editorial strength: how much it reads like real thought rather than performed polish.

Guide:
- **90-100**: Specific, varied, and grounded. Any issues are minor.
- **70-89**: Mostly human and credible, with some polish or structural habits.
- **50-69**: Noticeably corporate, AI-polished, or rhetorically over-shaped.
- **Below 50**: Dependent on generic confidence, symmetry, or AI-like scaffolding.

Format: **Before: [score]/100** — one sentence on the main drag on the score.

### Major Issues

Group by severity: High, Medium, Low. For each issue include:

- **Quote** — enough of the original to locate it
- **Type** — diagnostic category and pattern
- **Why it weakens the prose** — what the wording is hiding or inflating
- **Rewrite** — a concrete replacement

Do not flag everything. Focus on the issues that materially improve the piece.

### What Is Working

Name the strongest parts: earned contrast, useful specificity, real stakes, a good sentence worth preserving, or rhythm that clarifies rather than decorates.

### Revised Text

- **Under 500 words:** include the full revised text in chat.
- **500 words or more:** rewrite only the flagged passages by default, as before/after pairs.
- **Full rewrite over 500 words:** ask before creating a Markdown rewrite file or editing the source file. Never silently edit source files.
- **File-based input over 500 words:** audit in chat first, then ask before changing the source file. Offer a separate Markdown rewrite file as the non-destructive option.

### Checklist Only

When in Checklist only mode, use this compact structure:

```
SEQUENTIAL EDITING CHECKLIST:
(In document order)

- [ ] Line/section: "quoted problem"
  Fix: concrete instruction or replacement direction.
```

Keep it minimal and actionable. Do not include severity levels, confidence scores, or a full rewrite.

### Second-Pass Audit

After the rewrite, check the revised text for surviving problems:

- remaining AI/corporate tells
- remaining unearned parallelism
- lost meaning or over-casualisation
- unsupported new claims introduced by the rewrite
- invented names, numbers, dates, examples, sources, causality, product claims, outcomes, or stakes
- missing placeholders where the source text does not support a concrete rewrite

If clean, write: **Second-pass audit: Clean.** Use that line only when no AI tells, unearned rhetoric, invented facts, or hidden evidence gaps remain.

---

## Tone

Be direct and specific. Quote the actual text. Do not perform hostility, and do not sand the prose into blandness. The best output is a small number of sharp interventions plus a rewrite that carries the same meaning with more evidence of human judgment.

For external comms, policy, executive narrative, op-eds, and strategic prose, professional restraint matters. Human writing is not "chatty". It is concrete, alive to tradeoffs, and less afraid of saying the actual thing.
