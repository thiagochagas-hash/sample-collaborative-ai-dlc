# Frontend test-scenario plan

This is the single published test-scenario plan for `frontend/src/`. Per
`requirements`' Intent Analysis, the goal is a concrete, prioritized plan so a
future 90% line/statement coverage gate can be reached deliberately rather
than reactively — the deliverable is the plan itself, never test code (see
[Plan-vs-code boundary](#plan-vs-code-boundary) below). `business-rules` and
`domain-entities` define the governance rules and data shapes this chapter
publishes; `security-design` defines the controls on the content every
chapter must follow; `unit-of-work` fixes the chapter order and ownership.

This chapter (`comp-plan-governance`) is maintained once, up front. The
twelve content chapters below are each owned and appended by their own unit's
later code-generation run — this chapter does not draft their scenarios.

## Scenario types (bounded)

Per `business-rules` Rule 1 (`req-stack-scenario-bound`) and
`domain-entities`' `ScenarioType` value object, every scenario in every
chapter of this plan must use exactly one of these three types. No other
type — snapshot, shallow-render, implementation-internals inspection,
mutation testing, AI-assisted generation, or a new test runner — is
permitted:

| Scenario type        | Meaning                                                                          |
| -------------------- | -------------------------------------------------------------------------------- |
| `render-interaction` | A Testing Library role/text/label query plus an interaction or assertion outcome |
| `call-return-throw`  | A `services/*` call that returns successfully or throws/rejects                  |
| `state-transition`   | A hook or context state change, including failure/empty-state transitions        |

A content chapter that proposes a fourth type is wrong at draft time, not at
review time — the set above is fixed by `requirements`' Non-Functional
Requirements ("Stack compatibility") and Constraints (T-1/T-4) and is not
negotiable per chapter.

## Traceability ledger

Per `business-rules` Rule 2 (`req-plan-traceability`) and `domain-entities`'
`TraceabilityRow` entity, every in-scope file must appear in exactly one row
below (covered) or in the Won't-Have table (no scenario needed) — never both,
never omitted, never duplicated under two different requirement ids. A row's
shape is fixed:

| Column                | Meaning                                                                                    |
| --------------------- | ------------------------------------------------------------------------------------------ |
| `targetFile`          | The exact source file path the scenario covers                                             |
| `behaviorDescription` | Prose describing the asserted behavior                                                     |
| `scenarioType`        | One of the three bounded types above                                                       |
| `acceptanceRef`       | The specific `requirements` acceptance criterion satisfied                                 |
| `requirementId`       | The `requirements` id whose content unit produced the row (e.g. `req-u6-graph-components`) |

A row is immutable once appended — a correction is a new row plus a recorded
supersession, never a silent in-place edit. Each content chapter appends its
own rows under its own heading below; this governance chapter does not draft
rows itself.

### Authorization-adjacent pairing

Per `security-design` (`sec-paired-authz-scenarios`), a ledger row for an
authorization-adjacent file (`services/authErrors.ts`, other `services/auth.ts`
callers, `ProtectedRoute.tsx`, `PasswordResetForm.tsx`) is only complete when
the chapter lists **both** the success scenario and the corresponding
denial/redirect scenario. A row citing only the success path fails this
plan's traceability completeness check for that requirement, independent of
overall scenario count.

### Fixture-value rule

Per `security-design` (`sec-no-secret-material`), every scenario's mocked
inputs/outputs must use structurally-typed placeholders (e.g.
`token: "mock-jwt-abc"`, `status: 403`) rather than realistic-looking strings
that could be mistaken for live Cognito tokens, API keys, or provider/model
secrets. `npm run secretlint` is the backstop, not the primary control.

### Won't-Have entries

A file is marked "no scenario needed" only after direct inspection of its
actual source confirms it is a pure re-export barrel, type-only module, or
static constant with no functions — a folder-level heuristic alone is never
sufficient (`business-rules` Rule 3, `req-wont-have-verification`). The four
files below were already reclassified out of Won't-Have during requirements
analysis and must stay reassigned; any future file found to export
behavior-bearing code follows the same reassignment path, never a new
disposition:

| File                   | Status               | Owning requirement           | Evidence                                             |
| ---------------------- | -------------------- | ---------------------------- | ---------------------------------------------------- |
| `graphTypes.ts`        | Reassigned — covered | `req-u6-graph-components`    | Exports behavior-bearing functions, not pure types   |
| `businessRoleIcons.ts` | Reassigned — covered | `req-u10-settings-auth-misc` | Exports a lookup function, not a static constant     |
| `tool-presets.ts`      | Reassigned — covered | `req-u8-admin-components`    | Exports behavior-bearing preset-resolution logic     |
| `phaseDisplay.ts`      | Reassigned — covered | `req-u4-pages`               | Exports a formatting function, not a static constant |

No other file is currently classified Won't-Have. Each content chapter adds
its own confirmed Won't-Have entries (with direct-read evidence) alongside
its `TraceabilityRow`s as it drafts.

## Coverage-scope globs

Per `business-rules` Rule 4 (`req-coverage-scope-parity`) and
`domain-entities`' `CoverageScopeGlobs`, any coverage percentage computed
against this plan must use these fixed globs, matching the scope CI will
eventually use:

- **Include:** `src/**/*.{ts,tsx}`
- **Exclude:** `src/**/*.test.*`, `src/test/**`

This is a publication, not a measurement — no coverage provider is
configured today (`requirements`' Constraints); an ephemeral install is
needed only when a measurement is eventually taken. A measurement computed
with different include/exclude scoping must be flagged as non-comparable to
the 90% target, not silently reported alongside plan-scoped numbers.

## Risk-based unit sequence

Per `business-rules` Rule 5 (`req-risk-based-prioritization`) and
`domain-entities`' `UnitSequence`, the chapters below are ordered by
risk/dependency tier — auth, API clients, and shared state first — never by
raw untested-file count. `u11-ui-primitives` has more untested files than
several earlier units but must never move ahead of them; the order below is
fixed and is not re-sorted for apparent efficiency:

1. `u1-services`
2. `u2-hooks`
3. `u3-contexts`
4. `u4-pages`
5. `u5-intent-components`
6. `u6-graph-components`
7. `u7-observability-components`
8. `u8-admin-components`
9. `u9-discussion-components`
10. `u10-settings-auth-misc`
11. `u11-ui-primitives`
12. `u12-icons-layout-theme`

## Plan-vs-code boundary

Per `business-rules` Rule 6 (`req-plan-not-code`): **this plan's deliverable,
and every content chapter's deliverable, is scenario prose plus the
structured `TraceabilityRow`/`ScenarioEntry` blocks above — never a
`.test.ts` or `.test.tsx` file.** Writing or running the actual test code
described here is a separate, later effort, out of scope for this plan
(`requirements`' Out of Scope).

## Advisory framing

Per `business-rules` Rule 7 (`req-advisory-alignment`): **the 90% coverage
figure is an advisory target, not an active gate.** The frontend test job
today runs plain `npm run test` with no `--coverage` flag and no
percentage-based PR gate. Nothing in this plan proposes a CI pipeline change
or implies that 90% currently blocks merges; wiring an enforcing gate is an
explicit future decision outside this plan.

## Content chapters

Each chapter below is owned and appended by its own unit's code-generation
run, which drafts `ScenarioEntry` blocks and calls out its `TraceabilityRow`s
into the [ledger](#traceability-ledger) above using the
`contract-scenario-type-bound` and `contract-traceability-ledger` contracts
this governance chapter publishes. A chapter failing this plan's checks
(Rules 2, 3, or 5) is sent back to that chapter's own drafting pass — it
never invalidates chapters other units already completed.

### Chapter: u1-services (pending)

### Chapter: u2-hooks (pending)

### Chapter: u3-contexts (pending)

### Chapter: u4-pages (pending)

### Chapter: u5-intent-components (pending)

### Chapter: u6-graph-components

Scenario drafting for `comp-u6-graph-components`, satisfying `requirements`'
`req-u6-graph-components`. The scenario-authoring rules and entity shapes
below are drafted per `business-rules-u6-graph-components` and
`domain-entities-u6-graph-components`, and the fixture/mocking/assertion-surface
decisions follow `security-design-u6-graph-components`. Every row's
`scenarioType` is one of the two types this unit actually needs from the
[bounded set](#scenario-types-bounded) — this unit has no hooks or contexts,
so only `render-interaction` (the nine presentational components) and
`call-return-throw` (the nine pure functions across `graphTypes.ts`,
`nodeStyles.ts`, and `scope-graph-utils.ts`) appear below, per
`contract-scenario-type-bound`. `requirementId` is `req-u6-graph-components`
for every row, per `contract-traceability-ledger`.

#### Won't-Have verification for this unit

`components/v2/index.ts` was directly read (not folder-inferred): it contains
only `export { ... }` / `export type { ... }` re-export statements from
`WorkflowScopeGraph` and `unit-lane-graph`, with zero function or logic
definitions. It stays Won't-Have — no row is added for it below, per
`req-wont-have-verification`.

| File                     | Status                                    | Evidence                                           |
| ------------------------ | ----------------------------------------- | -------------------------------------------------- |
| `components/v2/index.ts` | Confirmed Won't-Have — no scenario needed | Direct read: pure re-export barrel, zero functions |

`components/graph/GraphCanvas.tsx` and `components/v2/WorkflowScopeGraph.tsx`
are out of scope for this unit (and for this plan): both already have their
own `*.test.tsx` suite in the repository, so they are not among the 13
untested files `req-u6-graph-components` names.

#### Scenario entries and traceability rows

| #   | `targetFile`                             | `behaviorDescription`                                                                                                                                                                                                                                                                | `scenarioType`     | `acceptanceRef`                              |
| --- | ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------ | -------------------------------------------- |
| 1   | `components/graph/graphTypes.ts`         | `getNodeCfg` — known type (e.g. `'Requirement'`) returns the matching `NODE_TYPES` entry (`color`, `label`, `icon`, etc.)                                                                                                                                                            | call-return-throw  | AC1 (happy path)                             |
| 2   | `components/graph/graphTypes.ts`         | `getNodeCfg` — unknown type (e.g. `'NotAType'`) returns the fixed `UNKNOWN_NODE_CFG`                                                                                                                                                                                                 | call-return-throw  | derived invariant (fallback)                 |
| 3   | `components/graph/graphTypes.ts`         | `hullPath([], padding)` → `''`                                                                                                                                                                                                                                                       | call-return-throw  | AC2 (degenerate: zero points)                |
| 4   | `components/graph/graphTypes.ts`         | `hullPath([oneNode], padding)` → circle-as-path of radius `padding` centered on the node                                                                                                                                                                                             | call-return-throw  | AC2 (degenerate: one point)                  |
| 5   | `components/graph/graphTypes.ts`         | `hullPath([nodeA, nodeB], padding)` → padded ribbon path, perpendicular-offset by `padding`                                                                                                                                                                                          | call-return-throw  | AC2 (degenerate: two points)                 |
| 6   | `components/graph/graphTypes.ts`         | `hullPath(threeOrMoreNonCollinearNodes, padding)` → closed polygon path (`M ... L ... Z`) whose vertex count matches the convex hull, each vertex expanded outward from the hull's centroid by `padding` (the only externally reachable exercise of the module-private `convexHull`) | call-return-throw  | AC1 (happy path for `convexHull`/`hullPath`) |
| 7   | `components/graph/graphTypes.ts`         | `deriveNodeLabel` — explicit `node.label` set and not `'(unnamed)'` → passthrough                                                                                                                                                                                                    | call-return-throw  | AC1 (happy path)                             |
| 8   | `components/graph/graphTypes.ts`         | `deriveNodeLabel` — `Question` with valid JSON `questions` array → first question's `text`/`question`, truncated at 48 chars + `'...'` beyond 50, with a `(+N)` suffix for N additional questions                                                                                    | call-return-throw  | derived invariant                            |
| 9   | `components/graph/graphTypes.ts`         | `deriveNodeLabel` — `Question` with malformed/non-JSON `questions` → falls back to literal `'Question'`                                                                                                                                                                              | call-return-throw  | derived invariant                            |
| 10  | `components/graph/graphTypes.ts`         | `deriveNodeLabel` — `Review` with/without `status` → `'Review (<status>)'` vs `'Review'`                                                                                                                                                                                             | call-return-throw  | derived invariant                            |
| 11  | `components/graph/graphTypes.ts`         | `deriveNodeLabel` — `PullRequest`/`UnitPullRequest` with/without `pr_number` → numbered (`'PR #<n>'` / `'Unit PR #<n>'`) vs generic literal                                                                                                                                          | call-return-throw  | derived invariant                            |
| 12  | `components/graph/graphTypes.ts`         | `deriveNodeLabel` — `GeneralInfo` with `title`, else `content` (truncated at 48+`'...'` beyond 50), else `'General Info'`                                                                                                                                                            | call-return-throw  | derived invariant                            |
| 13  | `components/graph/graphTypes.ts`         | `deriveNodeLabel` — unhandled type → `node.label \|\| node.type` default                                                                                                                                                                                                             | call-return-throw  | derived invariant                            |
| 14  | `components/graph/graphTypes.ts`         | `splitLabelTwoLines('')` → `['', null]`                                                                                                                                                                                                                                              | call-return-throw  | AC1 (happy path)                             |
| 15  | `components/graph/graphTypes.ts`         | `splitLabelTwoLines` — length ≤17 → `[label, null]` unchanged                                                                                                                                                                                                                        | call-return-throw  | AC1 (happy path)                             |
| 16  | `components/graph/graphTypes.ts`         | `splitLabelTwoLines` — >17 chars with a space at/before index 17 → split at the last such space, second line truncated to 15 chars + `'...'` if it still exceeds 17                                                                                                                  | call-return-throw  | derived invariant                            |
| 17  | `components/graph/graphTypes.ts`         | `splitLabelTwoLines` — >17 chars with no space in the first 17 → hard break at character 17                                                                                                                                                                                          | call-return-throw  | derived invariant                            |
| 18  | `components/graph/graphTypes.ts`         | `sliderTToWidth(widthToSliderT(300))` ≈ 300 (`ZOOM_MIN_W`, t≈1, epsilon comparison)                                                                                                                                                                                                  | call-return-throw  | AC3 (round-trip boundary)                    |
| 19  | `components/graph/graphTypes.ts`         | `sliderTToWidth(widthToSliderT(12000))` ≈ 12000 (`ZOOM_MAX_W`, t≈0, epsilon comparison)                                                                                                                                                                                              | call-return-throw  | AC3 (round-trip boundary)                    |
| 20  | `components/graph/nodeStyles.ts`         | `nodeTypeBadge` — known type → its fixed badge class string                                                                                                                                                                                                                          | call-return-throw  | derived invariant                            |
| 21  | `components/graph/nodeStyles.ts`         | `nodeTypeBadge` — unknown type → `'bg-muted text-muted-foreground'` fallback                                                                                                                                                                                                         | call-return-throw  | derived invariant                            |
| 22  | `components/graph/nodeStyles.ts`         | `nodeTypeTextColor` — known type → the `text-*` class parsed out of its badge                                                                                                                                                                                                        | call-return-throw  | derived invariant                            |
| 23  | `components/graph/nodeStyles.ts`         | `nodeTypeTextColor` — unknown type → still resolves to `'text-muted-foreground'` via the fallback badge's own `text-*` token                                                                                                                                                         | call-return-throw  | derived invariant                            |
| 24  | `components/graph/nodeStyles.ts`         | `shortNodeType` — mapped type (e.g. `'StoryMapEntry'` → `'Map'`)                                                                                                                                                                                                                     | call-return-throw  | derived invariant                            |
| 25  | `components/graph/nodeStyles.ts`         | `shortNodeType` — unmapped type returned unchanged (e.g. `'Requirement'` → `'Requirement'`)                                                                                                                                                                                          | call-return-throw  | derived invariant                            |
| 26  | `components/graph/nodeStyles.ts`         | `humanEdgeLabel` — mapped edge label (e.g. `'CONTAINS'` → `'contains'`)                                                                                                                                                                                                              | call-return-throw  | derived invariant                            |
| 27  | `components/graph/nodeStyles.ts`         | `humanEdgeLabel` — unmapped label lower-cased with underscores replaced by spaces (e.g. `'FOO_BAR'` → `'foo bar'`)                                                                                                                                                                   | call-return-throw  | derived invariant                            |
| 28  | `components/v2/scope-graph-utils.ts`     | `computeLayout` — a node whose `phasePath` matches a provided phase lands in that phase's column, ordered by `order`                                                                                                                                                                 | call-return-throw  | derived invariant                            |
| 29  | `components/v2/scope-graph-utils.ts`     | `computeLayout` — a node whose `phasePath` matches no provided phase lands in the `__other__` fallback column, which is only added to `phaseColumns` when non-empty                                                                                                                  | call-return-throw  | derived invariant                            |
| 30  | `components/v2/scope-graph-utils.ts`     | `paletteColorForIndex` — `index` within `[0, PHASE_PALETTE.length)` returns the corresponding color directly                                                                                                                                                                         | call-return-throw  | derived invariant                            |
| 31  | `components/v2/scope-graph-utils.ts`     | `paletteColorForIndex` — `index >= PHASE_PALETTE.length` wraps via modulo (e.g. `index = PHASE_PALETTE.length` returns the same color as `index = 0`)                                                                                                                                | call-return-throw  | derived invariant                            |
| 32  | `components/v2/scope-graph-utils.ts`     | `cubicEdgePath` — two calls for the same `from`/`to` pair with different `offset` values produce path strings with different control-point coordinates, while the start (`from` right edge) and end (`to` left edge) points stay fixed                                               | call-return-throw  | derived invariant                            |
| 33  | `components/v2/scope-graph-utils.ts`     | `countScopeStats` — `scopeGrid[scopeId]` has at least one `'EXECUTE'` entry → `executeCount` reflects only those, `liveEdges` counts only edges whose `from`/`to` are both in the resolved execute set                                                                               | call-return-throw  | derived invariant                            |
| 34  | `components/v2/scope-graph-utils.ts`     | `countScopeStats` — `scopeGrid[scopeId]` has none → falls back to scanning `scopeGrid[stageKey][scopeId]` (the inverse key orientation) and counts matches there                                                                                                                     | call-return-throw  | derived invariant                            |
| 35  | `components/graph/GraphFilterBar.tsx`    | Renders exactly one toggle per type in `presentTypes` whose `typeCounts[type] > 0`; a present type with a zero count renders nothing                                                                                                                                                 | render-interaction | derived invariant                            |
| 36  | `components/graph/GraphFilterBar.tsx`    | Clicking a type toggle calls `onToggleTypeFilter(type)`                                                                                                                                                                                                                              | render-interaction | derived invariant                            |
| 37  | `components/graph/GraphFilterBar.tsx`    | "Clear all" control is present only when `typeFilters.size > 0` and calls `onClearAll` when clicked                                                                                                                                                                                  | render-interaction | derived invariant                            |
| 38  | `components/graph/GraphKeyboardHelp.tsx` | Renders the fixed shortcut list, including the `'f or /'` → `'Search nodes'` row                                                                                                                                                                                                     | render-interaction | derived invariant                            |
| 39  | `components/graph/GraphKeyboardHelp.tsx` | Clicking the close control calls `onClose`                                                                                                                                                                                                                                           | render-interaction | derived invariant                            |
| 40  | `components/graph/GraphLegend.tsx`       | Renders one legend row per type in `presentTypes` whose `typeCounts[type] > 0`; a zero-count present type renders nothing                                                                                                                                                            | render-interaction | derived invariant                            |
| 41  | `components/graph/GraphLegend.tsx`       | Clicking a legend row calls `onToggleTypeFilter(type)`                                                                                                                                                                                                                               | render-interaction | derived invariant                            |
| 42  | `components/graph/GraphMinimap.tsx`      | Renders one `<circle>` per id present in `filteredNodeIds`                                                                                                                                                                                                                           | render-interaction | derived invariant                            |
| 43  | `components/graph/GraphMinimap.tsx`      | Renders one `<line>` per entry in `filteredEdges` whose `source`/`target` both resolve via `nodeMap`; an edge with a missing endpoint renders no line                                                                                                                                | render-interaction | derived invariant                            |
| 44  | `components/graph/GraphMinimap.tsx`      | The circle matching `selectedNode` carries `stroke="#fff"`; every other circle carries `stroke="none"`                                                                                                                                                                               | render-interaction | derived invariant                            |
| 45  | `components/graph/GraphNodePanel.tsx`    | Renders the selected node's label and type badge, plus `"<n> outgoing"` / `"<n> incoming"` counts matching the filtered edge sets                                                                                                                                                    | render-interaction | derived invariant                            |
| 46  | `components/graph/GraphNodePanel.tsx`    | Renders `"No relationships yet"` when `nodeEdges.length === 0`                                                                                                                                                                                                                       | render-interaction | derived invariant                            |
| 47  | `components/graph/GraphNodePanel.tsx`    | For a `Requirement`-typed node with `description`/`acceptance_criteria` set, renders both fields; renders `"No details available yet"` when neither is set                                                                                                                           | render-interaction | derived invariant                            |
| 48  | `components/graph/GraphNodePanel.tsx`    | Clicking the close control calls `onSelectNode(null)`                                                                                                                                                                                                                                | render-interaction | derived invariant                            |
| 49  | `components/graph/GraphNodePanel.tsx`    | Clicking an outgoing/incoming relationship row calls `onSelectNode` with that edge's other endpoint id                                                                                                                                                                               | render-interaction | derived invariant                            |
| 50  | `components/graph/GraphStatsPanel.tsx`   | Renders the six stat values from `graphStats` (`nodeCount`, `edgeCount`, `typeCount`, `density`, `maxDegree`, `avgDegree`)                                                                                                                                                           | render-interaction | derived invariant                            |
| 51  | `components/graph/GraphStatsPanel.tsx`   | Renders the Hub Node section (label + degree badge) only when `graphStats.hubNode` is set, and omits it cleanly when it is `null`/`undefined`                                                                                                                                        | render-interaction | derived invariant                            |
| 52  | `components/graph/GraphStatsPanel.tsx`   | Renders Edge Types sorted by count descending, humanizing a known label via `EDGE_LABELS` and falling back to the raw label for an unknown one                                                                                                                                       | render-interaction | derived invariant                            |
| 53  | `components/graph/GraphStatsPanel.tsx`   | Clicking the close control calls `onClose`                                                                                                                                                                                                                                           | render-interaction | derived invariant                            |
| 54  | `components/graph/GraphToolbar.tsx`      | Renders the provided `title`, or the literal `"Knowledge graph"` when `title` is `undefined`                                                                                                                                                                                         | render-interaction | derived invariant                            |
| 55  | `components/graph/GraphToolbar.tsx`      | Clicking the Force/Hierarchy buttons calls `onLayoutModeChange('force'/'hierarchical')`                                                                                                                                                                                              | render-interaction | derived invariant                            |
| 56  | `components/graph/GraphToolbar.tsx`      | The Filters button's count badge renders only when `typeFilterCount > 0`                                                                                                                                                                                                             | render-interaction | derived invariant                            |
| 57  | `components/graph/GraphToolbar.tsx`      | Typing into the search input calls `onSearchChange` with the new value; the clear-search control appears only when `search` is non-empty and calls `onSearchChange('')`                                                                                                              | render-interaction | derived invariant                            |
| 58  | `components/graph/GraphZoomControls.tsx` | Clicking Zoom-in/Zoom-out/Fit-to-content calls `onZoomIn`/`onZoomOut`/`onFitToContent` respectively                                                                                                                                                                                  | render-interaction | derived invariant                            |
| 59  | `components/graph/GraphZoomControls.tsx` | The range input's value equals `widthToSliderT(viewBox.width)`                                                                                                                                                                                                                       | render-interaction | derived invariant                            |
| 60  | `components/graph/GraphZoomControls.tsx` | Changing the range input to a new `t` calls `onViewBoxChange` with a box whose `width = sliderTToWidth(t)`, whose `height` preserves the original aspect ratio, and whose center point is unchanged from the input `viewBox`'s center                                                | render-interaction | derived invariant                            |
| 61  | `components/v2/UnitLaneGraph.tsx`        | Renders nothing when `computeUnitLaneLayout` returns `null` (e.g. `unitLanes.sectionStageIds` empty)                                                                                                                                                                                 | render-interaction | derived invariant                            |
| 62  | `components/v2/UnitLaneGraph.tsx`        | Renders a phase column label and a truncated unit-slug node for a populated layout                                                                                                                                                                                                   | render-interaction | derived invariant                            |
| 63  | `components/v2/UnitLaneGraph.tsx`        | Clicking a clickable stage node (one with `onStageClick` and either `kind === 'stage'` or a non-null `rowKey`) calls `onStageClick` with that node's `stageId` and `rowKey`                                                                                                          | render-interaction | derived invariant                            |
| 64  | `components/v2/UnitLaneGraph.tsx`        | Pressing Enter or Space on a focused clickable node calls the same handler (the `onKeyDown` path)                                                                                                                                                                                    | render-interaction | derived invariant                            |
| 65  | `components/v2/unit-lane-graph.ts`       | Already covered by `unit-lane-layout.test.ts` — verify, do not duplicate (see [Known discrepancy](#known-discrepancy-unit-lane-graphts) below)                                                                                                                                       | call-return-throw  | derived invariant (BR-7)                     |

All 65 rows resolve `req-plan-traceability`'s "every in-scope file appears in
exactly one row" constraint across this unit's 13 files (rows 1–19 and 65 for
`graphTypes.ts`/`unit-lane-graph.ts`; rows 20–27 for `nodeStyles.ts`; rows
28–34 for `scope-graph-utils.ts`; rows 35–64 for the nine components). Every
fixture referenced above (node/edge payloads, `Question`/`Review`/
`PullRequest`/`GeneralInfo` shapes, graph phases/lanes) is authored as a
synthetic inline literal per scenario, never copied from live data, per
`security-design-u6-graph-components`'s fixture-hygiene design. Every
component scenario (rows 35–64) asserts via a Testing Library role/text/label
query or an explicit DOM attribute value — never `container.innerHTML`,
snapshot, or CSS-class/inline-style assertions — per `req-stack-scenario-bound`
and the same security design's rendering-safety design.

#### Known discrepancy: `unit-lane-graph.ts`

`requirements` and `unit-of-work` both classify `components/v2/unit-lane-graph.ts`
as untested, but `components/v2/unit-lane-layout.test.ts` — committed in the
same change as the source file, well before this intent began — already
exercises every reachable branch of `computeUnitLaneLayout`: both `null`-return
guards, dropped-empty-column behavior, synthetic unit/stage-cell node
emission, per-unit vertical chaining, cross-section edge remapping, and edge
dedup. Row 65 above records this as `already-covered — verify, do not
duplicate` rather than drafting a redundant scenario set, per
`business-rules-u6-graph-components`'s Known Discrepancy Handling rule. If a
future coverage measurement shows a real gap here, re-open row 65 rather than
trusting the "untested" label at face value.

### Chapter: u7-observability-components (pending)

### Chapter: u8-admin-components (pending)

### Chapter: u9-discussion-components (pending)

### Chapter: u10-settings-auth-misc (pending)

### Chapter: u11-ui-primitives (pending)

### Chapter: u12-icons-layout-theme (pending)
