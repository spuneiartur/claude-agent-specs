# UI Workflow Playbook

## Contents

1. Intake and classification
2. Create workflow
3. Redesign workflow
4. Audit workflow
5. Code-first implementation workflow
6. Polish workflow
7. Responsive and content behavior
8. Visual validation loop

## 1. Intake and classification

Classify three axes before acting.

### Operation

- `Create`: no stable interface exists.
- `Redesign`: the interface exists and a material change is requested.
- `Audit`: diagnose and prioritize without implementing everything.
- `Translate`: move between design and implementation.
- `Polish`: improve craft while preserving the main structure.

### Surface

- `Brand surface`: marketing, campaign, editorial, portfolio, or launch page. Distinctiveness and emotional framing can carry more weight.
- `Product surface`: application, dashboard, settings, operations, or tool. Task clarity, density, predictability, and state completeness carry more weight.
- `Hybrid surface`: onboarding, pricing, upgrade, or product-led growth. Balance persuasion with transparent control.

### Experience mode

- `Persuade`: help a person evaluate or commit.
- `Operate`: help a person complete frequent or consequential tasks.
- `Read`: help a person comprehend, compare, or learn.
- `Experience`: create immersion, expression, or narrative.

A surface may use a primary and secondary mode. Name both and resolve conflicts explicitly.

## 2. Create workflow

### Step A: Write the brief

Use this compact brief:

```text
Product:
Surface:
Primary user:
Primary job:
Primary action:
Secondary actions:
Content/data:
Platform/viewports:
Design system:
Brand/tone:
Constraints:
Definition of done:
Assumptions:
```

### Step B: Define direction

Create one direction by default. Use alternatives only for a real unresolved decision.

```text
Visual thesis:
Hierarchy:
Typography roles:
Color/token strategy:
Layout and density:
Interaction and motion:
Signature element:
Anti-goals:
```

The signature element must arise from the product subject or behavior. It is not a mandatory visual effect.

### Step C: Shape the interface

1. Order content by user decision needs.
2. Identify the one primary action for each state.
3. Map every block to a component role.
4. Remove sections that do not help comprehension, trust, comparison, or action.
5. Write real or clearly marked sample copy.
6. Define responsive transformations before coding.
7. Define the state matrix before final styling.

### Step D: Build

- Inspect the codebase and component library first.
- Map requirements to existing components.
- Verify component props and examples from documentation.
- Prefer composition over cloning a component.
- Use semantic structure and native behavior.
- Centralize tokens and repeated values.
- Keep styling local enough to avoid accidental specificity conflicts.
- Add motion only after the static hierarchy works.
- Respect reduced motion and device capability.

### Step E: Validate

Run the bounded loop in section 8 and score with the rubric.

## 3. Redesign workflow

### Step A: State why change is needed

Classify the cause:

- task failure;
- hierarchy or comprehension failure;
- design-system drift;
- accessibility failure;
- responsive or content failure;
- brand mismatch;
- conversion or adoption hypothesis;
- technical debt;
- aesthetic preference only.

Aesthetic preference alone requires a small change budget unless the user explicitly requests a new direction.

### Step B: Preserve the useful parts

List:

- behaviors users may already understand;
- components that work and should remain;
- content or data that must remain stable;
- interaction contracts that cannot change;
- measured strengths or known positive feedback.

### Step C: Set the change budget

- `Repair`: fix defects, preserve structure.
- `Refactor`: change hierarchy or components, preserve the mental model.
- `Reframe`: change the mental model or journey; requires stronger evidence and migration planning.

### Step D: Compare before and after

For each material change, state:

```text
Observed problem:
Likely cause:
Change:
Expected user effect:
Trade-off:
How to validate:
```

Do not present a visual difference as an improvement without a reason.

## 4. Audit workflow

### Inspection order

1. Primary task and hierarchy.
2. Content clarity and action naming.
3. Navigation and information architecture.
4. State, feedback, and recovery.
5. Accessibility and input methods.
6. Responsive behavior and content stress.
7. Design-system fidelity.
8. Visual craft and distinctiveness.
9. Performance-affecting UI choices.

### Severity

- `Critical`: blocks a primary task, causes likely harm, or excludes users.
- `High`: creates frequent failure, serious confusion, or trust loss.
- `Medium`: slows the task or creates avoidable inconsistency.
- `Low`: polish or rare edge case with limited impact.

### Audit finding format

```text
Finding:
Severity:
Evidence:
Affected users/task:
Why it matters:
Recommended fix:
Validation:
Confidence:
```

Rank by impact, frequency, confidence, and repair cost. Do not create a flat list of minor observations.

## 5. Code-first implementation workflow

1. Read the selected frame, variables, components, layout, and relevant neighboring states.
2. Read the product design-system and code conventions.
3. Query Storybook or component documentation before using any component API.
4. Create a node-to-component map.
5. Flag unmapped nodes and decide whether to compose, extend, or create.
6. Preserve semantic behavior rather than reproducing coordinates blindly.
7. Implement responsive intent, not only the captured viewport.
8. Create stories or examples for meaningful states.
9. Run interaction and accessibility checks.
10. Compare rendered output with the design and document deliberate differences.

### Node-to-component map

```text
Figma node | Production component | Props/tokens | State | Gap/decision
```

Never assume a component prop from its name. Verify it.

## 6. Polish workflow

Polish in this order:

1. remove unclear or redundant content;
2. repair hierarchy;
3. align layout and spacing rhythm;
4. repair typography roles and line lengths;
5. improve action labels and feedback;
6. align color and contrast;
7. reduce unnecessary containers and decoration;
8. refine interaction states;
9. add restrained motion where it communicates state or continuity;
10. stress test content and viewports.

Do not start with shadows, gradients, radii, or animation when the information structure is weak.

## 7. Responsive and content behavior

Define transformations, not scaled screenshots.

For each major region specify:

```text
Region:
Large viewport:
Medium viewport:
Small viewport:
Long content behavior:
Interaction change:
Priority retained:
```

Stress test:

- headings at 150 to 200 percent expected length;
- translated labels 30 to 50 percent longer;
- large numbers and empty values;
- tables with overflow or column priority;
- keyboard open on mobile;
- zoom to 200 percent;
- reduced width and increased text size;
- images missing or delayed;
- no hover capability;
- touch and pointer input.

## 8. Visual validation loop

### Pass 1: Inspect

Capture representative desktop and mobile views. Inspect:

- first five seconds: purpose, hierarchy, primary action;
- reading order and scan path;
- spacing rhythm and alignment;
- content density and line length;
- contrast and focus visibility;
- state consistency;
- clipping, overflow, and reflow;
- asset and font loading;
- interaction feedback.

### Pass 2: Exercise

Exercise:

- primary path;
- keyboard-only path;
- one error or recovery path;
- one long-content or localization case;
- reduced motion if motion exists.

### Pass 3: Repair

Cluster defects by root cause, such as tokens, component API, layout rule, content rule, or missing state. Apply one coherent repair batch rather than many unrelated tweaks.

### Pass 4: Confirm

Re-run only affected checks. Stop when critical defects are resolved and the score is at least 85, or report what prevents completion.
