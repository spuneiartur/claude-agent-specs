# UX Pattern Library (internal)

Curated pattern-decision reference for recurring UI problems in Artur's projects (React/Next.js Pages Router + Tailwind). Not user research — a precedent/tested-pattern reference at the same evidence tier as Baymard/NN/g, but scoped to this stack and component set. Use it to speed up and standardize option comparison; it does not override real product evidence when available.

## Contents

1. Tables
2. Forms
3. Filters and search
4. Pagination and large lists
5. Empty, loading, and error states

## 1. Tables

Problem: user must review, edit, or act on many structured rows (admin CRUD, PIM catalogs, order lists).

| Pattern | When to use | Trade-off | Notes for this stack |
|---|---|---|---|
| Inline edit | Edits are 1-2 fields, frequent, low-risk | Loses context on long scroll; needs clear save/cancel per row | Pair with optimistic update + `{Entity}Error` fallback |
| Bulk edit | Same change applied to many rows at once (status change, delete, export) | Requires selection state + confirmation for destructive actions | Checkbox column + sticky action toolbar above the table |
| Row detail drawer/modal | Edit needs more than ~4 fields or has its own validation | Extra click before editing | Reuse `Modal`; do not duplicate the full form inline |
| Read-only table + separate edit page | Editing is rare, high-stakes, or has a long form | Slower for frequent small edits | Default for anything using `form-builder` (Yup schemas) |

Default for this stack: read-only table + edit page/modal, unless edits are proven frequent and simple (then inline edit). Always design the loading (`Bone` skeleton), empty, and error variants — see section 5.

## 2. Forms

Problem: user must input or correct structured data.

| Pattern | When to use | Trade-off |
|---|---|---|
| Single-page form | ≤ ~8 fields, one logical entity | Can feel dense if fields are unrelated |
| Multi-step/wizard | Fields span multiple concerns (e.g. account + payment + confirmation) or conditional branching is heavy | Adds navigation overhead; must show progress and allow back |
| Inline field validation | Validation is fast/local (format, required) | Noisy if fired on every keystroke — validate on blur or debounce |
| Submit-time validation only | Validation depends on server state (uniqueness, availability) | User only learns of the problem after submitting — must scroll to and focus the first error |

Default for this stack: React Hook Form + Yup via the project's `HookForm`/`Field` system (see `form-builder` skill where present). Always design: validation error, submit error (network/server), submit loading (disable + spinner on the action button, not a full-page block), and success confirmation.

## 3. Filters and search

Problem: user must narrow a large set to what matters right now.

| Pattern | When to use | Trade-off |
|---|---|---|
| Inline filter bar (always visible) | ≤ ~4 filter dimensions, used on nearly every visit | Takes permanent vertical space |
| Collapsible filter panel/drawer | > 4 dimensions, or filtering is occasional | Extra click to open; must show active-filter count when collapsed |
| Search-as-you-type | Users know roughly what they want by name/keyword | Needs debounce + loading indicator; must handle zero-result state explicitly |
| Faceted filters with live counts | Users are exploring, not sure what exists yet | Requires backend support for counts per facet; more expensive to build |

Default for this stack: inline filter bar for admin lists with few dimensions; collapsible panel once filters exceed the available header width. Always show the active filters applied (as removable pills) and a clear "reset filters" action.

## 4. Pagination and large lists

Problem: the full result set is too large to render or scan at once.

| Pattern | When to use | Trade-off |
|---|---|---|
| Numbered pagination | Task-oriented use (admin lists, order history) — user needs to return to a specific page or count results | Requires stable sort/ordering; less natural on mobile |
| Infinite scroll | Browsing/discovery use (catalogs, feeds) | Loses "position" on back-navigation; must handle scroll restoration |
| "Load more" button | Middle ground — browsing but wants explicit control | Extra interaction per batch; simplest to implement correctly |
| Virtualized list | Very large sets (100s-1000s of rows) rendered at once | Adds complexity; only justified by a measured performance problem |

Default for this stack: numbered pagination for admin/PIM lists (use `hooks/use-infinite-query.js`'s cursor pattern only when the UX is genuinely a feed/catalog, not a management table). Always design the last-page and zero-results states.

## 5. Empty, loading, and error states

Problem: every list, table, or data-driven view has states beyond the happy path with data.

| State | Minimum requirement |
|---|---|
| Loading | Use the project's `{Entity}Loading.jsx` + `Bone` skeleton matching final layout shape — never a generic spinner over the whole page for a list that will show a table |
| Empty (no data yet) | Explain why it's empty and give the one primary action to fill it (e.g. "No products yet — Add your first product"), not just "No results" |
| Empty (filtered to zero) | Distinguish from true-empty: show which filters are active and a "clear filters" action |
| Error | Use `{Entity}Error.jsx`; explain what failed in plain language and offer retry; never show a raw error/stack trace |
| Partial success | If a bulk action partially fails, report exactly which rows succeeded/failed, not just an aggregate count |

Default for this stack: every `useQuery`-backed view ships all three status components (`Loading`/`Error`/`Success`) per the project's convention — treat a view missing any of the three as incomplete, not just unpolished.
