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

| Scenario type | Meaning |
| --- | --- |
| `render-interaction` | A Testing Library role/text/label query plus an interaction or assertion outcome |
| `call-return-throw` | A `services/*` call that returns successfully or throws/rejects |
| `state-transition` | A hook or context state change, including failure/empty-state transitions |

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

| Column | Meaning |
| --- | --- |
| `targetFile` | The exact source file path the scenario covers |
| `behaviorDescription` | Prose describing the asserted behavior |
| `scenarioType` | One of the three bounded types above |
| `acceptanceRef` | The specific `requirements` acceptance criterion satisfied |
| `requirementId` | The `requirements` id whose content unit produced the row (e.g. `req-u6-graph-components`) |

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

| File | Status | Owning requirement | Evidence |
| --- | --- | --- | --- |
| `graphTypes.ts` | Reassigned — covered | `req-u6-graph-components` | Exports behavior-bearing functions, not pure types |
| `businessRoleIcons.ts` | Reassigned — covered | `req-u10-settings-auth-misc` | Exports a lookup function, not a static constant |
| `tool-presets.ts` | Reassigned — covered | `req-u8-admin-components` | Exports behavior-bearing preset-resolution logic |
| `phaseDisplay.ts` | Reassigned — covered | `req-u4-pages` | Exports a formatting function, not a static constant |

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

### Chapter: u6-graph-components (pending)

### Chapter: u7-observability-components (pending)

### Chapter: u8-admin-components (pending)

### Chapter: u9-discussion-components (pending)

### Chapter: u10-settings-auth-misc (pending)

### Chapter: u11-ui-primitives (pending)

### Chapter: u12-icons-layout-theme (pending)
