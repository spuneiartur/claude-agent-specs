# UI Output Templates

## 1. Compact design direction

```markdown
## Context and assumptions
- Product/surface:
- User and primary job:
- Existing system:
- Constraints:
- Assumptions:

## Direction
- Experience mode:
- Visual thesis:
- Hierarchy:
- Typography:
- Color/tokens:
- Layout/density:
- Interaction/motion:
- Signature element:
- Anti-goals:

## Structure
[Short ordered outline or ASCII wireframe]

## Components and states
[Component inventory plus state matrix]

## Validation
- Checks run:
- Findings repaired:
- Unverified areas:
- Score:
```

## 2. UI audit

```markdown
# UI audit: [surface]

## Executive finding
[One paragraph: primary task, largest risk, recommended repair strategy]

## Evidence and scope
- Surface/version/date:
- Viewports:
- Paths exercised:
- Tools/checks:
- Limitations:

## Prioritized findings

### [Finding]
- Severity:
- Observed:
- User/task impact:
- Root cause:
- Recommended fix:
- Validation:
- Confidence:

## Repair sequence
1. Critical blockers
2. Structural fixes
3. Design-system and state fixes
4. Polish

## Scorecard
[Rubric table]
```

## 3. Code-first implementation map

```markdown
# Design implementation map

## Source
- Product brief:
- Repo / branch:
- Design system / tokens:
- Starter components / production components:
- Existing stories or examples:

## Component map
| Requirement | Starter / production component | Props/tokens | States | Gap/decision |
|---|---|---|---|---|

## Responsive behavior
| Region | Large | Medium | Small | Long content |
|---|---|---|---|---|

## Deliberate differences
| Difference | Reason | Owner/approval |
|---|---|---|---|

## Validation
- Browser checks:
- Interaction tests:
- Accessibility checks:
- Screenshots/viewports:
- Remaining gaps:
```

## 4. Design-system exception

```markdown
# Design-system exception

- Requirement:
- Existing components evaluated:
- Why composition is insufficient:
- Proposed extension or new component:
- Tokens and API:
- States and accessibility behavior:
- Migration/reuse potential:
- Validation:
- Owner and review date:
```

## 5. Critique response

```markdown
## Overall assessment
[What works, what prevents the interface from succeeding, and the highest-leverage change]

## Findings
1. `Observed`:
   `Impact`:
   `Proposed`:
   `Validate`:

## Keep
[Specific elements worth preserving]

## Change first
[One coherent repair batch]

## Quality score
[Score, critical failures, unverified checks]
```

## Example user requests

- "Build a dense operations dashboard using our starter components. Include loading, empty, permission, partial failure, and mobile states."
- "Audit this settings flow. Preserve the current design system and rank only evidence-backed defects."
- "Implement this screen in code using the starter component library. Verify every component prop and create or update stories for important states."
- "Find real mobile examples of account deletion, compare their control and recovery patterns, then propose a transparent flow without copying one product."
