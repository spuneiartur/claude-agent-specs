# UI Quality Rubric

## Contents

1. Scoring
2. Critical failures
3. State matrix
4. Anti-pattern review
5. Accessibility and responsive checks
6. Critique language

## 1. Scoring

Score each category with evidence. Do not award points for unverified claims.

| Category | Weight | Full-credit standard |
|---|---:|---|
| Task clarity and hierarchy | 20 | Purpose, primary task, status, and next action are immediately understandable. |
| Design-system fidelity | 15 | Real tokens and documented components are reused; exceptions are justified. |
| Accessibility and input | 15 | Semantic behavior, keyboard, focus, contrast, zoom, text reflow, labels, and reduced motion are addressed. |
| Content and UX writing | 10 | Labels are specific, consistent, plain, and tied to user-recognizable outcomes. |
| State and recovery completeness | 10 | Loading, empty, error, partial, success, permission, destructive, and recovery states are designed as relevant. |
| Responsive and content resilience | 10 | Layout transforms intentionally and survives long content, localization, zoom, and small screens. |
| Visual craft | 10 | Typography, spacing, color, alignment, imagery, and density form a coherent system. |
| Interaction and motion | 5 | Feedback is timely; motion communicates continuity or state and respects reduced motion. |
| Product-specific distinctiveness | 5 | The interface has an appropriate point of view grounded in the product, not a generic template. |

### Score interpretation

- `90-100`: ship-ready after normal product review.
- `85-89`: strong; small documented gaps may remain.
- `70-84`: useful direction but not ready to claim completion.
- `Below 70`: material redesign or implementation work remains.

A score above 85 does not override a critical failure.

## 2. Critical failures

Mark `FAIL` if any applies:

- Primary task or consequence is unclear.
- Critical interaction cannot be completed with keyboard or assistive technology where applicable.
- Known contrast, focus, labeling, or reflow defect blocks meaningful use.
- Likely failure or destructive action has no clear recovery.
- A component prop, token, product fact, or test result was invented.
- Layout fails at a required viewport, zoom level, or realistic content length.
- Sensitive or consequential data is disclosed without need or control.
- An opt-out, cancellation, or rejection path is intentionally harder than opt-in without legitimate reason.
- Validation is claimed but was not run.

## 3. State matrix

Use this table for each meaningful component or screen. Mark `N/A` with a reason rather than leaving cells blank.

| State | Content | Available actions | Feedback | Accessibility | Recovery |
|---|---|---|---|---|---|
| Default |  |  |  |  |  |
| Hover/focus/pressed |  |  |  |  |  |
| Loading |  |  |  |  |  |
| Empty/first use |  |  |  |  |  |
| Validation error |  |  |  |  |  |
| System error |  |  |  |  |  |
| Partial success |  |  |  |  |  |
| Success |  |  |  |  |  |
| Disabled/restricted |  |  |  |  |  |
| Offline/stale/timeout |  |  |  |  |  |
| Destructive/undo |  |  |  |  |  |
| Long/localized content |  |  |  |  |  |

## 4. Anti-pattern review

Treat these as prompts for judgment, not universal bans.

### Generic visual output

- defaulting to the same neutral SaaS font, blue-purple gradient, floating cards, icon tiles, and oversized rounded corners regardless of subject;
- using numbered sections when no sequence exists;
- using fake charts, metrics, testimonials, or activity;
- making every region a card or nesting cards within cards;
- adding ambient blobs, glass effects, or motion with no information role;
- applying a novel style that conflicts with the established product without an explicit redesign mandate.

### Weak hierarchy

- multiple equal primary actions;
- headings that describe containers rather than user decisions;
- labels such as `Submit`, `Continue`, or `Learn more` when a specific outcome is available;
- visual emphasis driven by decoration instead of task priority;
- crucial consequences hidden in secondary text.

### Fragile implementation

- undocumented component props;
- one-off colors, spacing, or radii instead of tokens;
- CSS rules that cancel one another through accidental specificity;
- absolute positioning used to reproduce a screenshot rather than encode layout intent;
- hover-only actions;
- fixed heights around variable content;
- animation that blocks input or ignores reduced motion.

### Incomplete product behavior

- empty states that only announce emptiness;
- errors that do not explain correction or recovery;
- skeletons that misrepresent final layout;
- disabled actions with no explanation;
- optimistic success with no reconciliation or failure path;
- destructive controls without consequence, confirmation, or undo appropriate to risk.

## 5. Accessibility and responsive checks

### Accessibility

Check, as relevant:

- semantic landmarks and heading order;
- accessible names, descriptions, and error associations;
- native controls before custom widgets;
- keyboard order, focus visibility, and focus restoration;
- no keyboard trap;
- contrast and non-color indicators;
- touch target size and spacing;
- zoom and text reflow;
- captions, alternatives, and meaningful image treatment;
- live-region behavior without excessive announcements;
- reduced motion and pause/stop controls;
- authentication and cognitive burden;
- ARIA roles, states, properties, and keyboard behavior against the APG when custom widgets are unavoidable.

Automated checks do not replace keyboard and screen-reader-informed review.

### Responsive

Check:

- priority order at each breakpoint;
- navigation and action placement;
- tables, charts, and dense data;
- overlays with the mobile keyboard open;
- text wrapping and truncation consequences;
- safe areas and viewport units;
- pointer, touch, and no-hover modes;
- loading and error states at all target widths;
- loss of context when content stacks or moves.

## 6. Critique language

Use specific, causal language.

Weak:

> The page feels cluttered.

Strong:

> `Observed`: Four controls use equal visual weight above the first data row. `Impact`: A first-time operator cannot identify the required next action. `Proposed`: Keep `Create report` as the only filled action, move export options into the overflow menu, and verify task completion time with first-time users.

For every important finding include observation, user or task impact, recommendation, and validation method.
