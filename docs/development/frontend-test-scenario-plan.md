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

### Chapter: u1-services

Per `unit-of-work`'s `unit-u1-services` entry and `requirements`' `req-u1-services`,
this chapter covers the 15 untested `frontend/src/services/*` API-client files
that are the direct API/auth surface with no upstream frontend dependency —
`services/api.ts` and `services/auth.ts` are excluded (both already have
`api.test.ts`/`auth.test.ts`), per `req-u1-services`'s third acceptance
criterion and `business-rules` Rule Set 1 (R1.2). `domain-entities` fixes the
real exported shapes this chapter's scenarios mock and assert against;
`security-design` fixes the 401/403/424 scenario-kind design every
auth-adjacent scenario below follows. Every scenario in this chapter uses the
`call-return-throw` scenario type the plan-governance chapter's
`contract-scenario-type-bound` fixes, and every row below appends to this
plan's `contract-traceability-ledger` with `requirementId: req-u1-services`
(omitted from each row for brevity) — no row in this chapter uses a different
requirement id. Fixture values throughout follow `security-design`'s
Fixture-and-Artifact-Hygiene design: synthetic ids (`sprint-fixture-1`,
`project-fixture-1`, `user-fixture-1`), a synthetic IAM role
(`arn:aws:iam::000000000000:role/example-role`), never a realistic-looking
secret.

This chapter's scenarios fix the exact `api.get`/`post`/`put`/`patch`/`delete`
call signatures and `ApiError`/entity return shapes `domain-entities` catalogs,
satisfying this unit's `contract-service-call-signatures` obligation to
`u2-hooks`, `u5-intent-components`, and `u8-admin-components` (per
`business-rules` R5.1): those units may mock against the method/argument/
return/throw shapes below without re-reading `frontend/src/services/*` source.

#### Group A — v1 read-only list/get clients (9 files)

Per `domain-entities`' Group A, each file exposes only flat `list(sprintId)` /
`get(sprintId, id)` accessors (or, for `reviewsService`, only `get(sprintId)`)
backed by `api.get<T>`; there is no create/update/delete surface since v1
write routes were removed. Every error scenario below asserts a thrown
`ApiError` (status 404 for an unknown sprint/id, per the shared throw entity).

| # | File | Method | Kind | Behavior | Acceptance ref |
| - | --- | --- | --- | --- | --- |
| A1 | `codeFiles.ts` | `list(sprintId)` | happy-path | Resolves `CodeFile[]` for `sprint-fixture-1` | ac1 |
| A2 | `codeFiles.ts` | `list(sprintId)` | error | Rejects `ApiError` `status: 404` for an unknown sprint id | ac1 |
| A3 | `codeFiles.ts` | `get(sprintId, id)` | happy-path | Resolves one `CodeFile` for `code-file-fixture-1` | ac1 |
| A4 | `codeFiles.ts` | `get(sprintId, id)` | error | Rejects `ApiError` `status: 404` for an unknown file id | ac1 |
| A5 | `generalInfo.ts` | `list(sprintId)` | happy-path | Resolves `GeneralInfo[]` with mixed `type` values | ac1 |
| A6 | `generalInfo.ts` | `list(sprintId)` | error | Rejects `ApiError` `status: 404` | ac1 |
| A7 | `generalInfo.ts` | `get(sprintId, id)` | happy-path | Resolves one `GeneralInfo` record | ac1 |
| A8 | `generalInfo.ts` | `get(sprintId, id)` | error | Rejects `ApiError` `status: 404` | ac1 |
| A9 | `questions.ts` | `list(sprintId)` | happy-path | Resolves `Question[]` including one answered (`structuredAnswer` set, `answeredBy` populated) and one unanswered question | ac1 |
| A10 | `questions.ts` | `list(sprintId)` | error | Rejects `ApiError` `status: 404` | ac1 |
| A11 | `questions.ts` | `get(sprintId, id)` | happy-path | Resolves one `Question` with a `draftAnswer` present but no `structuredAnswer` yet | ac1 |
| A12 | `questions.ts` | `get(sprintId, id)` | error | Rejects `ApiError` `status: 404` | ac1 |
| A13 | `requirements.ts` | `list(sprintId)` | happy-path | Resolves `Requirement[]` for `sprint-fixture-1` | ac1 |
| A14 | `requirements.ts` | `list(sprintId)` | error | Rejects `ApiError` `status: 404` | ac1 |
| A15 | `requirements.ts` | `get(sprintId, id)` | happy-path | Resolves one `Requirement` | ac1 |
| A16 | `requirements.ts` | `get(sprintId, id)` | error | Rejects `ApiError` `status: 404` | ac1 |
| A17 | `reviews.ts` | `get(sprintId)` | happy-path (populated) | Resolves a `Review` with `status: 'PARTIAL'`, distinct `blindStatus`/`fullStatus` | ac1 |
| A18 | `reviews.ts` | `get(sprintId)` | happy-path (none-yet) | Resolves `null` for a sprint with no review started, exercising the `Review \| null` return type | ac1 |
| A19 | `reviews.ts` | `get(sprintId)` | error | Rejects `ApiError` `status: 404` for an unknown sprint | ac1 |
| A20 | `sprints.ts` | `list(projectId)` | happy-path | Resolves `Sprint[]` spanning all four `SprintPhase` values | ac1 |
| A21 | `sprints.ts` | `list(projectId)` | error | Rejects `ApiError` `status: 404` for an unknown project | ac1 |
| A22 | `sprints.ts` | `get(projectId, sprintId)` | happy-path (tracker) | Resolves a `Sprint` with `tracker` populated (`provider`, `resourceUrl` set) and legacy `issueNumber`/`issueUrl` both `null` | ac1 |
| A23 | `sprints.ts` | `get(projectId, sprintId)` | happy-path (legacy) | Resolves a `Sprint` with `tracker: null` but legacy `issueNumber`/`issueUrl` populated, confirming both read shapes stay supported | ac1 |
| A24 | `sprints.ts` | `get(projectId, sprintId)` | error | Rejects `ApiError` `status: 404` for an unknown sprint | ac1 |
| A25 | `tasks.ts` | `list(sprintId)` | happy-path | Resolves `Task[]` including one with `dependencies` set and one without | ac1 |
| A26 | `tasks.ts` | `list(sprintId)` | error | Rejects `ApiError` `status: 404` | ac1 |
| A27 | `tasks.ts` | `get(sprintId, id)` | happy-path | Resolves one `Task` | ac1 |
| A28 | `tasks.ts` | `get(sprintId, id)` | error | Rejects `ApiError` `status: 404` | ac1 |
| A29 | `timelineEvents.ts` | `list(sprintId)` | happy-path | Resolves `TimelineEvent[]` spanning at least a `question_asked` and a `question_answered` event sharing one `questionId` | ac1 |
| A30 | `timelineEvents.ts` | `list(sprintId)` | error | Rejects `ApiError` `status: 404` | ac1 |
| A31 | `userStories.ts` | `list(sprintId)` | happy-path | Resolves `UserStory[]` for `sprint-fixture-1` | ac1 |
| A32 | `userStories.ts` | `list(sprintId)` | error | Rejects `ApiError` `status: 404` | ac1 |
| A33 | `userStories.ts` | `get(sprintId, id)` | happy-path | Resolves one `UserStory` | ac1 |
| A34 | `userStories.ts` | `get(sprintId, id)` | error | Rejects `ApiError` `status: 404` | ac1 |

#### Group B — platform-admin mutation/config clients (`adminUsers.ts`, `codecommit.ts`)

Per `business-rules` Rule Set 3 (R3.2) and `security-design`'s 401-vs-403
scenario design, these two files are this unit's sole owners of explicit
`authorization-denied` (403) scenarios, satisfying `req-u1-services`'s second
acceptance criterion. A 403 scenario never asserts a session-expiry outcome
(per `security-design`'s assertion-boundary rule).

| # | File | Method | Kind | Behavior | Acceptance ref |
| - | --- | --- | --- | --- | --- |
| B1 | `adminUsers.ts` | `list()` | happy-path | Resolves `AdminUser[]` with a mix of `platformAdmin: true/false` | ac1 |
| B2 | `adminUsers.ts` | `list()` | session-expiry | Rejects `ApiError` `status: 401` after a forced-refresh still comes back expired; scenario text reads "session expired, sign in again", never "locked out" | ac2 |
| B3 | `adminUsers.ts` | `list()` | authorization-denied | Rejects `ApiError` `status: 403` for a caller without the platform-admin role; scenario text does not imply a session-expiry outcome | ac2 |
| B4 | `adminUsers.ts` | `setPlatformAdmin(username, isAdmin)` | happy-path | `setPlatformAdmin('user-fixture-1', true)` resolves `{ username: 'user-fixture-1', platformAdmin: true }` | ac1 |
| B5 | `adminUsers.ts` | `setPlatformAdmin(username, isAdmin)` | authorization-denied | Rejects `ApiError` `status: 403` for a non-platform-admin caller | ac2 |
| B6 | `adminUsers.ts` | `setPlatformAdmin(username, isAdmin)` | error | Rejects `ApiError` `status: 404` for an unknown `username` | ac1 |
| B7 | `codecommit.ts` | `codecommitAdminService.getConfig()` | happy-path (unconfigured) | Resolves `CodeCommitAdminConfig` with `roleArn: null`, `region: null` | ac1 |
| B8 | `codecommit.ts` | `codecommitAdminService.getConfig()` | happy-path (configured) | Resolves a config with `roleArn: 'arn:aws:iam::000000000000:role/example-role'` and a filled `trustPolicy`/`permissionsPolicy` | ac1 |
| B9 | `codecommit.ts` | `codecommitAdminService.getConfig()` | authorization-denied | Rejects `ApiError` `status: 403` for a non-platform-admin caller | ac2 |
| B10 | `codecommit.ts` | `codecommitAdminService.updateConfig(update)` | happy-path | `updateConfig({ roleArn: 'arn:aws:iam::000000000000:role/example-role', region: 'us-east-1' })` resolves the fresh `CodeCommitAdminConfig` | ac1 |
| B11 | `codecommit.ts` | `codecommitAdminService.updateConfig(update)` | error | Rejects `ApiError` `status: 500` for an unrelated backend failure (the generic-throw scenario `business-rules` R3.4 distinguishes from the trust-probe case below) | ac1 |
| B12 | `codecommit.ts` | `codecommitAdminService.updateConfig(update)` | authorization-denied | Rejects `ApiError` `status: 403` for a non-platform-admin caller | ac2 |
| B13 | `codecommit.ts` | `codecommitAdminService.updateConfig(update)` | trust-probe-failure | Backend's pre-write `ListRepositories` probe fails on the submitted role: rejects `ApiError` `status: 424`, **and** the scenario asserts the mocked write transport was never invoked and the previously-saved config is unchanged — the one data-integrity invariant this method depends on, per `security-design` | ac1, ac2 |

#### Group C — reusable building-blocks client (`blocks.ts`)

Per `domain-entities`' Group C note, `readOnly` distinguishes the SYSTEM-owned
imported baseline from the user-created/forked catalog; scenarios A6/A7 below
cover both a `readOnly: true` read and a mutation attempt against a
`readOnly` block.

| # | Method | Kind | Behavior | Acceptance ref |
| - | --- | --- | --- | --- |
| C1 | `list(type)` | happy-path | `list('stage')` resolves `{ blocks: Block[] }` with a mix of `readOnly: true`/`false` entries | ac1 |
| C2 | `list(type)` | error | Rejects `ApiError` `status: 500` | ac1 |
| C3 | `get(type, id)` | happy-path (readOnly) | `get('stage', 'block-fixture-system-1')` resolves a `Block` with `readOnly: true` | ac1 |
| C4 | `get(type, id)` | error | Rejects `ApiError` `status: 404` for an unknown block id | ac1 |
| C5 | `getBody(type, id)` | happy-path | Resolves `{ body: '<markdown>' }` for a block with `hasBody: true` | ac1 |
| C6 | `getBody(type, id)` | error | Rejects `ApiError` `status: 404` for a block with `hasBody: false` | ac1 |
| C7 | `getScript(type, id)` | happy-path | Resolves `{ script: '<script source>' }` for a `sensor` block with `hasScript: true` | ac1 |
| C8 | `getScript(type, id)` | error | Rejects `ApiError` `status: 404` for a block with `hasScript: false` | ac1 |
| C9 | `create(type, input)` | happy-path | `create('rule', { id: 'rule-fixture-1', name: 'Example Rule' })` resolves a new `Block` with `readOnly: false` | ac1 |
| C10 | `create(type, input)` | error | Rejects `ApiError` `status: 400` for a missing required `id` | ac1 |
| C11 | `update(type, id, input)` | happy-path | Updates a user-created block and resolves the updated `Block` | ac1 |
| C12 | `update(type, id, input)` | error (readOnly mutation) | `update('stage', 'block-fixture-system-1', input)` against a `readOnly: true` block rejects `ApiError` `status: 403` | ac1 |
| C13 | `delete(type, id)` | happy-path | Resolves (void) for a user-created block | ac1 |
| C14 | `delete(type, id)` | error (readOnly mutation) | `delete('stage', 'block-fixture-system-1')` against a `readOnly: true` block rejects `ApiError` `status: 403` | ac1 |

#### Group D — source-control metadata client (`sourceControl.ts`)

`defaultAuthTypeFor` is a pure synchronous function (no network call, never
throws); the two scenarios below exercise its two branches directly. The
remaining methods are `api`-backed and follow the standard happy/error
pairing. Per `domain-entities`' Group D note, `bind` needs both a
selected-provider case and an empty-selection case.

| # | Method | Kind | Behavior | Acceptance ref |
| - | --- | --- | --- | --- |
| D1 | `defaultAuthTypeFor(provider)` | happy-path | `defaultAuthTypeFor('github')` returns `'github-app'` (first/preferred option) | ac1 |
| D2 | `defaultAuthTypeFor(provider)` | edge-case | `defaultAuthTypeFor('')` (no provider picked yet) returns the `'github-app'` fallback | ac1 |
| D3 | `getStatus(projectId)` | happy-path | Resolves `ProjectSourceControlStatus` with `ready: true` and one repository `status: 'active'` | ac1 |
| D4 | `getStatus(projectId)` | error | Rejects `ApiError` `status: 404` for an unknown project | ac1 |
| D5 | `bind(projectId, providers)` | happy-path (selected) | `bind(projectId, { github: { authType: 'github-app' } })` resolves the updated status | ac1 |
| D6 | `bind(projectId, providers)` | happy-path (empty selection) | `bind(projectId, {})` resolves a status with `ready: false` and an unchanged repository list | ac1 |
| D7 | `bind(projectId, providers)` | error | Rejects `ApiError` `status: 422` for an invalid provider/authType combination | ac1 |
| D8 | `unbind(projectId)` | happy-path | Resolves (void) | ac1 |
| D9 | `unbind(projectId)` | error | Rejects `ApiError` `status: 404` for an unknown project | ac1 |
| D10 | `listBranches(projectId, provider, repository)` | happy-path | Resolves `{ branches: [...], defaultBranch: 'main' }` | ac1 |
| D11 | `listBranches(projectId, provider, repository)` | error | Rejects `ApiError` `status: 404` for an unbound repository | ac1 |
| D12 | `getTree(projectId, provider, repository, branch?)` | happy-path | Resolves `{ tree: GitFile[] }`, confirming the bare array from `api.get` is wrapped | ac1 |
| D13 | `getTree(projectId, provider, repository, branch?)` | error | Rejects `ApiError` `status: 404` for an unknown branch | ac1 |
| D14 | `getFileContents(projectId, provider, repository, path, branch?)` | happy-path | Resolves `GitFileContent` for a known `path` | ac1 |
| D15 | `getFileContents(projectId, provider, repository, path, branch?)` | error | Rejects `ApiError` `status: 404` for an unknown `path` | ac1 |
| D16 | `getReviewComments(projectId, provider, repository, reviewId)` | happy-path | Resolves `{ comments: GitComment[] }` | ac1 |
| D17 | `getReviewComments(projectId, provider, repository, reviewId)` | error | Rejects `ApiError` `status: 404` for an unknown `reviewId` | ac1 |

#### Group E — auth-error classification utility (`authErrors.ts`)

Per `business-rules` Rule Set 3 (R3.3), this file is the SSO-layer analogue of
the 401-vs-403 distinction, detected from error-object shape rather than HTTP
status. These five scenarios cover `isSsoAccessDeniedError`,
`normalizeSsoLoginError`, and the three exported error classes directly —
there is no network call, so "call" below means invoking the exported
function/constructor.

| # | Function | Kind | Behavior | Acceptance ref |
| - | --- | --- | --- | --- |
| E1 | `normalizeSsoLoginError` | authorization-denied (403-equivalent) | Given an error shaped `{ code: 'SSO_ACCESS_DENIED' }`, `isSsoAccessDeniedError` returns `true` and `normalizeSsoLoginError` returns a `SsoAccessDeniedError` wrapping the cause | ac2 |
| E2 | `normalizeSsoLoginError` | session-expiry (401-equivalent, transient) | Given an already-constructed `SsoLoginTimeoutError` instance as input, `normalizeSsoLoginError` returns the same instance unchanged (the `instanceof Error` passthrough branch), confirming the timeout/transient case is never reclassified as access-denied; scenario text reads "try again", never "locked out" | ac2 |
| E3 | `normalizeSsoLoginError` | generic passthrough | Given a plain `new Error('network failure')` with no SSO markers, `isSsoAccessDeniedError` returns `false` and `normalizeSsoLoginError` returns the same `Error` instance unchanged | ac1 |
| E4 | `normalizeSsoLoginError` | generic wrap | Given a non-`Error` thrown value (e.g. a bare string), `normalizeSsoLoginError` returns a new `Error('Enterprise sign-in failed', { cause: error })` | ac1 |
| E5 | `PasswordResetRequiredError` | happy-path (construction) | `new PasswordResetRequiredError()` exposes `name: 'PasswordResetRequiredError'` and the fixed reset-required message, so callers can `instanceof`-discriminate it without going through `normalizeSsoLoginError` | ac1 |

#### Group B (continued) — environment and tool lifecycle clients (`environments.ts`)

`environments.ts` exports two services, `environmentsService` (15 methods)
and `toolsService` (12 methods), covering the Managed Environments/Tools
admin builder lifecycle (`DRAFT → QUEUED → BUILDING → SCANNING →
SECURITY_REVIEW → VERIFYING → READY → PUBLISHED`, with `FAILED`/`RETIRED` as
terminal/side states). Per `business-rules` Rule Set 3 (R3.2), the explicit
403-vs-401 scenario-kind design is scoped to `adminUsersService` and
`codecommitAdminService` only — these two services instead get the standard
happy/error pair per method, per `business-rules` R2.2 ("the minimum pair
applies per method, not once per file"). Every happy-path cell is
`scenarioType: call-return-throw`, `acceptanceRef: req-u1-services#ac1`.
Fixture ids follow the `env-fixture-*` / `tool-fixture-*` / `revision-fixture-*`
/ `version-fixture-*` convention.

**`environmentsService`**

| Method | Happy-path scenario | Error scenario |
| --- | --- | --- |
| `list(publishedOnly?)` | Resolves `ManagedEnvironment[]` spanning a `PUBLISHED` and a `DRAFT` entry | Rejects `ApiError 500` |
| `capabilities()` | Resolves `EnvironmentCapabilities` with both `microvms`/`instances` compute flags | Rejects `ApiError 500` |
| `getDefault()` | Resolves `DefaultEnvironment` naming `env-fixture-1` | Rejects `ApiError 404` (no default set) |
| `setDefault(environmentId)` | `setDefault('env-fixture-1')` resolves success | Rejects `ApiError 404` for an unknown `environmentId` |
| `get(environmentId)` | Resolves `EnvironmentDetail` including its `EnvironmentRevision[]` history | Rejects `ApiError 404` |
| `create(input)` | Resolves `EnvironmentMutationResult` with the new environment in `DRAFT` status | Rejects `ApiError 400` for an invalid recipe |
| `update(environmentId, input)` | Resolves the updated `EnvironmentMutationResult` | Rejects `ApiError 404` |
| `build(environmentId, revisionId)` | Resolves the revision transitioning to `BUILDING` | Rejects `ApiError 409` for a revision already `BUILDING` |
| `retry(environmentId, revisionId)` | Resolves the `FAILED` revision transitioning back to `BUILDING` | Rejects `ApiError 409` for a revision not in `FAILED` |
| `acceptFindings(environmentId, revisionId)` | Resolves the revision moving past `SECURITY_REVIEW` to `VERIFYING` | Rejects `ApiError 409` when there are no findings to accept |
| `publish(environmentId, revisionId)` | Resolves the revision `PUBLISHED`, environment's `publishedRevisionId` updated | Rejects `ApiError 409` for a revision not `READY` |
| `rebuild(environmentId)` | Resolves a freshly queued revision for the environment | Rejects `ApiError 404` |
| `rebuildAll(environmentIds?)` | `rebuildAll()` resolves a queued-rebuild result for every published environment | Rejects/reports `ApiError 404` for one unknown id inside an explicit `environmentIds` list |
| `retire(environmentId)` | Resolves the environment moved to `RETIRED` | Rejects `ApiError 404` |
| `logs(environmentId, revisionId)` | Resolves the build-log payload for a `BUILDING`/terminal revision | Rejects `ApiError 404` for a `DRAFT` revision with no logs yet |

**`toolsService`**

| Method | Happy-path scenario | Error scenario |
| --- | --- | --- |
| `list(publishedOnly?)` | Resolves `ManagedTool[]` spanning a `PUBLISHED` and a `DRAFT` tool | Rejects `ApiError 500` |
| `get(toolId)` | Resolves `ManagedTool` with its `ManagedToolVersion[]` history | Rejects `ApiError 404` |
| `create(input)` | Resolves `ToolMutationResult` with the new tool in `DRAFT` | Rejects `ApiError 400` for an invalid definition |
| `update(toolId, input)` | Resolves the updated `ToolMutationResult` | Rejects `ApiError 404` |
| `createVersion(toolId, definition)` | Resolves a new `ToolVersionDefinition` for `arm64` | Rejects `ApiError 400` for an unsupported `ToolArchitecture` |
| `updateVersion(toolId, versionId, definition)` | Resolves the updated version definition | Rejects `ApiError 404` |
| `build(toolId, versionId)` | Resolves the version transitioning to `BUILDING` | Rejects `ApiError 409` for a version already `BUILDING` |
| `retry(toolId, versionId)` | Resolves the `FAILED` version transitioning back to `BUILDING` | Rejects `ApiError 409` for a version not `FAILED` |
| `acceptFindings(toolId, versionId)` | Resolves the version moving past its security review | Rejects `ApiError 409` when there are no findings to accept |
| `publish(toolId, versionId)` | Resolves the version published | Rejects `ApiError 409` for a version not ready |
| `recommend(toolId, versionId)` | Resolves the version marked recommended | Rejects `ApiError 404` |
| `logs(toolId, versionId)` | Resolves the build-log payload | Rejects `ApiError 404` for a version with no logs yet |

#### Chapter totals

- **135 scenario entries** across 15 files (34 Group A, 13 Group B admin-gated
  + 27×2=54 Group B environment/tool lifecycle, 14 Group C, 17 Group D, 5
  Group E), every one `scenarioType: call-return-throw`, every one appending
  a `TraceabilityRow` with `requirementId: req-u1-services`.
- Explicit 401/403/424 coverage (`req-u1-services`'s second acceptance
  criterion): B2/B3, B5, B9, B12/B13, E1/E2 above.
- No scenario is written for `services/auth.ts` or `services/api.ts`
  (`req-u1-services`'s third acceptance criterion; `business-rules` R1.2).
- No file in this chapter is Won't-Have; all 15 export behavior-bearing code
  confirmed by direct read.
- Framing stays advisory throughout, per `req-advisory-alignment`: every
  403/424 scenario above is phrased as coverage the plan is building toward
  (`scenario verifies ...` / `resolves ...` / `rejects ...`), never as
  behavior a CI gate already enforces.

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
