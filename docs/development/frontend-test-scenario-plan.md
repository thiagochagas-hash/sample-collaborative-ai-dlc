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

| #   | File                | Method                     | Kind                   | Behavior                                                                                                                           | Acceptance ref |
| --- | ------------------- | -------------------------- | ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | -------------- |
| A1  | `codeFiles.ts`      | `list(sprintId)`           | happy-path             | Resolves `CodeFile[]` for `sprint-fixture-1`                                                                                       | ac1            |
| A2  | `codeFiles.ts`      | `list(sprintId)`           | error                  | Rejects `ApiError` `status: 404` for an unknown sprint id                                                                          | ac1            |
| A3  | `codeFiles.ts`      | `get(sprintId, id)`        | happy-path             | Resolves one `CodeFile` for `code-file-fixture-1`                                                                                  | ac1            |
| A4  | `codeFiles.ts`      | `get(sprintId, id)`        | error                  | Rejects `ApiError` `status: 404` for an unknown file id                                                                            | ac1            |
| A5  | `generalInfo.ts`    | `list(sprintId)`           | happy-path             | Resolves `GeneralInfo[]` with mixed `type` values                                                                                  | ac1            |
| A6  | `generalInfo.ts`    | `list(sprintId)`           | error                  | Rejects `ApiError` `status: 404`                                                                                                   | ac1            |
| A7  | `generalInfo.ts`    | `get(sprintId, id)`        | happy-path             | Resolves one `GeneralInfo` record                                                                                                  | ac1            |
| A8  | `generalInfo.ts`    | `get(sprintId, id)`        | error                  | Rejects `ApiError` `status: 404`                                                                                                   | ac1            |
| A9  | `questions.ts`      | `list(sprintId)`           | happy-path             | Resolves `Question[]` including one answered (`structuredAnswer` set, `answeredBy` populated) and one unanswered question          | ac1            |
| A10 | `questions.ts`      | `list(sprintId)`           | error                  | Rejects `ApiError` `status: 404`                                                                                                   | ac1            |
| A11 | `questions.ts`      | `get(sprintId, id)`        | happy-path             | Resolves one `Question` with a `draftAnswer` present but no `structuredAnswer` yet                                                 | ac1            |
| A12 | `questions.ts`      | `get(sprintId, id)`        | error                  | Rejects `ApiError` `status: 404`                                                                                                   | ac1            |
| A13 | `requirements.ts`   | `list(sprintId)`           | happy-path             | Resolves `Requirement[]` for `sprint-fixture-1`                                                                                    | ac1            |
| A14 | `requirements.ts`   | `list(sprintId)`           | error                  | Rejects `ApiError` `status: 404`                                                                                                   | ac1            |
| A15 | `requirements.ts`   | `get(sprintId, id)`        | happy-path             | Resolves one `Requirement`                                                                                                         | ac1            |
| A16 | `requirements.ts`   | `get(sprintId, id)`        | error                  | Rejects `ApiError` `status: 404`                                                                                                   | ac1            |
| A17 | `reviews.ts`        | `get(sprintId)`            | happy-path (populated) | Resolves a `Review` with `status: 'PARTIAL'`, distinct `blindStatus`/`fullStatus`                                                  | ac1            |
| A18 | `reviews.ts`        | `get(sprintId)`            | happy-path (none-yet)  | Resolves `null` for a sprint with no review started, exercising the `Review \| null` return type                                   | ac1            |
| A19 | `reviews.ts`        | `get(sprintId)`            | error                  | Rejects `ApiError` `status: 404` for an unknown sprint                                                                             | ac1            |
| A20 | `sprints.ts`        | `list(projectId)`          | happy-path             | Resolves `Sprint[]` spanning all four `SprintPhase` values                                                                         | ac1            |
| A21 | `sprints.ts`        | `list(projectId)`          | error                  | Rejects `ApiError` `status: 404` for an unknown project                                                                            | ac1            |
| A22 | `sprints.ts`        | `get(projectId, sprintId)` | happy-path (tracker)   | Resolves a `Sprint` with `tracker` populated (`provider`, `resourceUrl` set) and legacy `issueNumber`/`issueUrl` both `null`       | ac1            |
| A23 | `sprints.ts`        | `get(projectId, sprintId)` | happy-path (legacy)    | Resolves a `Sprint` with `tracker: null` but legacy `issueNumber`/`issueUrl` populated, confirming both read shapes stay supported | ac1            |
| A24 | `sprints.ts`        | `get(projectId, sprintId)` | error                  | Rejects `ApiError` `status: 404` for an unknown sprint                                                                             | ac1            |
| A25 | `tasks.ts`          | `list(sprintId)`           | happy-path             | Resolves `Task[]` including one with `dependencies` set and one without                                                            | ac1            |
| A26 | `tasks.ts`          | `list(sprintId)`           | error                  | Rejects `ApiError` `status: 404`                                                                                                   | ac1            |
| A27 | `tasks.ts`          | `get(sprintId, id)`        | happy-path             | Resolves one `Task`                                                                                                                | ac1            |
| A28 | `tasks.ts`          | `get(sprintId, id)`        | error                  | Rejects `ApiError` `status: 404`                                                                                                   | ac1            |
| A29 | `timelineEvents.ts` | `list(sprintId)`           | happy-path             | Resolves `TimelineEvent[]` spanning at least a `question_asked` and a `question_answered` event sharing one `questionId`           | ac1            |
| A30 | `timelineEvents.ts` | `list(sprintId)`           | error                  | Rejects `ApiError` `status: 404`                                                                                                   | ac1            |
| A31 | `userStories.ts`    | `list(sprintId)`           | happy-path             | Resolves `UserStory[]` for `sprint-fixture-1`                                                                                      | ac1            |
| A32 | `userStories.ts`    | `list(sprintId)`           | error                  | Rejects `ApiError` `status: 404`                                                                                                   | ac1            |
| A33 | `userStories.ts`    | `get(sprintId, id)`        | happy-path             | Resolves one `UserStory`                                                                                                           | ac1            |
| A34 | `userStories.ts`    | `get(sprintId, id)`        | error                  | Rejects `ApiError` `status: 404`                                                                                                   | ac1            |

#### Group B — platform-admin mutation/config clients (`adminUsers.ts`, `codecommit.ts`)

Per `business-rules` Rule Set 3 (R3.2) and `security-design`'s 401-vs-403
scenario design, these two files are this unit's sole owners of explicit
`authorization-denied` (403) scenarios, satisfying `req-u1-services`'s second
acceptance criterion. A 403 scenario never asserts a session-expiry outcome
(per `security-design`'s assertion-boundary rule).

| #   | File            | Method                                        | Kind                      | Behavior                                                                                                                                                                                                                                                                                                            | Acceptance ref |
| --- | --------------- | --------------------------------------------- | ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------- |
| B1  | `adminUsers.ts` | `list()`                                      | happy-path                | Resolves `AdminUser[]` with a mix of `platformAdmin: true/false`                                                                                                                                                                                                                                                    | ac1            |
| B2  | `adminUsers.ts` | `list()`                                      | session-expiry            | Rejects `ApiError` `status: 401` after a forced-refresh still comes back expired; scenario text reads "session expired, sign in again", never "locked out"                                                                                                                                                          | ac2            |
| B3  | `adminUsers.ts` | `list()`                                      | authorization-denied      | Rejects `ApiError` `status: 403` for a caller without the platform-admin role; scenario text does not imply a session-expiry outcome                                                                                                                                                                                | ac2            |
| B4  | `adminUsers.ts` | `setPlatformAdmin(username, isAdmin)`         | happy-path                | `setPlatformAdmin('user-fixture-1', true)` resolves `{ username: 'user-fixture-1', platformAdmin: true }`                                                                                                                                                                                                           | ac1            |
| B5  | `adminUsers.ts` | `setPlatformAdmin(username, isAdmin)`         | authorization-denied      | Rejects `ApiError` `status: 403` for a non-platform-admin caller                                                                                                                                                                                                                                                    | ac2            |
| B6  | `adminUsers.ts` | `setPlatformAdmin(username, isAdmin)`         | error                     | Rejects `ApiError` `status: 404` for an unknown `username`                                                                                                                                                                                                                                                          | ac1            |
| B7  | `codecommit.ts` | `codecommitAdminService.getConfig()`          | happy-path (unconfigured) | Resolves `CodeCommitAdminConfig` with `roleArn: null`, `region: null`                                                                                                                                                                                                                                               | ac1            |
| B8  | `codecommit.ts` | `codecommitAdminService.getConfig()`          | happy-path (configured)   | Resolves a config with `roleArn: 'arn:aws:iam::000000000000:role/example-role'` and a filled `trustPolicy`/`permissionsPolicy`                                                                                                                                                                                      | ac1            |
| B9  | `codecommit.ts` | `codecommitAdminService.getConfig()`          | authorization-denied      | Rejects `ApiError` `status: 403` for a non-platform-admin caller                                                                                                                                                                                                                                                    | ac2            |
| B10 | `codecommit.ts` | `codecommitAdminService.updateConfig(update)` | happy-path                | `updateConfig({ roleArn: 'arn:aws:iam::000000000000:role/example-role', region: 'us-east-1' })` resolves the fresh `CodeCommitAdminConfig`                                                                                                                                                                          | ac1            |
| B11 | `codecommit.ts` | `codecommitAdminService.updateConfig(update)` | error                     | Rejects `ApiError` `status: 500` for an unrelated backend failure (the generic-throw scenario `business-rules` R3.4 distinguishes from the trust-probe case below)                                                                                                                                                  | ac1            |
| B12 | `codecommit.ts` | `codecommitAdminService.updateConfig(update)` | authorization-denied      | Rejects `ApiError` `status: 403` for a non-platform-admin caller                                                                                                                                                                                                                                                    | ac2            |
| B13 | `codecommit.ts` | `codecommitAdminService.updateConfig(update)` | trust-probe-failure       | Backend's pre-write `ListRepositories` probe fails on the submitted role: rejects `ApiError` `status: 424`, **and** the scenario asserts the mocked write transport was never invoked and the previously-saved config is unchanged — the one data-integrity invariant this method depends on, per `security-design` | ac1, ac2       |

#### Group C — reusable building-blocks client (`blocks.ts`)

Per `domain-entities`' Group C note, `readOnly` distinguishes the SYSTEM-owned
imported baseline from the user-created/forked catalog; scenarios A6/A7 below
cover both a `readOnly: true` read and a mutation attempt against a
`readOnly` block.

| #   | Method                    | Kind                      | Behavior                                                                                                             | Acceptance ref |
| --- | ------------------------- | ------------------------- | -------------------------------------------------------------------------------------------------------------------- | -------------- |
| C1  | `list(type)`              | happy-path                | `list('stage')` resolves `{ blocks: Block[] }` with a mix of `readOnly: true`/`false` entries                        | ac1            |
| C2  | `list(type)`              | error                     | Rejects `ApiError` `status: 500`                                                                                     | ac1            |
| C3  | `get(type, id)`           | happy-path (readOnly)     | `get('stage', 'block-fixture-system-1')` resolves a `Block` with `readOnly: true`                                    | ac1            |
| C4  | `get(type, id)`           | error                     | Rejects `ApiError` `status: 404` for an unknown block id                                                             | ac1            |
| C5  | `getBody(type, id)`       | happy-path                | Resolves `{ body: '<markdown>' }` for a block with `hasBody: true`                                                   | ac1            |
| C6  | `getBody(type, id)`       | error                     | Rejects `ApiError` `status: 404` for a block with `hasBody: false`                                                   | ac1            |
| C7  | `getScript(type, id)`     | happy-path                | Resolves `{ script: '<script source>' }` for a `sensor` block with `hasScript: true`                                 | ac1            |
| C8  | `getScript(type, id)`     | error                     | Rejects `ApiError` `status: 404` for a block with `hasScript: false`                                                 | ac1            |
| C9  | `create(type, input)`     | happy-path                | `create('rule', { id: 'rule-fixture-1', name: 'Example Rule' })` resolves a new `Block` with `readOnly: false`       | ac1            |
| C10 | `create(type, input)`     | error                     | Rejects `ApiError` `status: 400` for a missing required `id`                                                         | ac1            |
| C11 | `update(type, id, input)` | happy-path                | Updates a user-created block and resolves the updated `Block`                                                        | ac1            |
| C12 | `update(type, id, input)` | error (readOnly mutation) | `update('stage', 'block-fixture-system-1', input)` against a `readOnly: true` block rejects `ApiError` `status: 403` | ac1            |
| C13 | `delete(type, id)`        | happy-path                | Resolves (void) for a user-created block                                                                             | ac1            |
| C14 | `delete(type, id)`        | error (readOnly mutation) | `delete('stage', 'block-fixture-system-1')` against a `readOnly: true` block rejects `ApiError` `status: 403`        | ac1            |

#### Group D — source-control metadata client (`sourceControl.ts`)

`defaultAuthTypeFor` is a pure synchronous function (no network call, never
throws); the two scenarios below exercise its two branches directly. The
remaining methods are `api`-backed and follow the standard happy/error
pairing. Per `domain-entities`' Group D note, `bind` needs both a
selected-provider case and an empty-selection case.

| #   | Method                                                            | Kind                         | Behavior                                                                                       | Acceptance ref |
| --- | ----------------------------------------------------------------- | ---------------------------- | ---------------------------------------------------------------------------------------------- | -------------- |
| D1  | `defaultAuthTypeFor(provider)`                                    | happy-path                   | `defaultAuthTypeFor('github')` returns `'github-app'` (first/preferred option)                 | ac1            |
| D2  | `defaultAuthTypeFor(provider)`                                    | edge-case                    | `defaultAuthTypeFor('')` (no provider picked yet) returns the `'github-app'` fallback          | ac1            |
| D3  | `getStatus(projectId)`                                            | happy-path                   | Resolves `ProjectSourceControlStatus` with `ready: true` and one repository `status: 'active'` | ac1            |
| D4  | `getStatus(projectId)`                                            | error                        | Rejects `ApiError` `status: 404` for an unknown project                                        | ac1            |
| D5  | `bind(projectId, providers)`                                      | happy-path (selected)        | `bind(projectId, { github: { authType: 'github-app' } })` resolves the updated status          | ac1            |
| D6  | `bind(projectId, providers)`                                      | happy-path (empty selection) | `bind(projectId, {})` resolves a status with `ready: false` and an unchanged repository list   | ac1            |
| D7  | `bind(projectId, providers)`                                      | error                        | Rejects `ApiError` `status: 422` for an invalid provider/authType combination                  | ac1            |
| D8  | `unbind(projectId)`                                               | happy-path                   | Resolves (void)                                                                                | ac1            |
| D9  | `unbind(projectId)`                                               | error                        | Rejects `ApiError` `status: 404` for an unknown project                                        | ac1            |
| D10 | `listBranches(projectId, provider, repository)`                   | happy-path                   | Resolves `{ branches: [...], defaultBranch: 'main' }`                                          | ac1            |
| D11 | `listBranches(projectId, provider, repository)`                   | error                        | Rejects `ApiError` `status: 404` for an unbound repository                                     | ac1            |
| D12 | `getTree(projectId, provider, repository, branch?)`               | happy-path                   | Resolves `{ tree: GitFile[] }`, confirming the bare array from `api.get` is wrapped            | ac1            |
| D13 | `getTree(projectId, provider, repository, branch?)`               | error                        | Rejects `ApiError` `status: 404` for an unknown branch                                         | ac1            |
| D14 | `getFileContents(projectId, provider, repository, path, branch?)` | happy-path                   | Resolves `GitFileContent` for a known `path`                                                   | ac1            |
| D15 | `getFileContents(projectId, provider, repository, path, branch?)` | error                        | Rejects `ApiError` `status: 404` for an unknown `path`                                         | ac1            |
| D16 | `getReviewComments(projectId, provider, repository, reviewId)`    | happy-path                   | Resolves `{ comments: GitComment[] }`                                                          | ac1            |
| D17 | `getReviewComments(projectId, provider, repository, reviewId)`    | error                        | Rejects `ApiError` `status: 404` for an unknown `reviewId`                                     | ac1            |

#### Group E — auth-error classification utility (`authErrors.ts`)

Per `business-rules` Rule Set 3 (R3.3), this file is the SSO-layer analogue of
the 401-vs-403 distinction, detected from error-object shape rather than HTTP
status. These five scenarios cover `isSsoAccessDeniedError`,
`normalizeSsoLoginError`, and the three exported error classes directly —
there is no network call, so "call" below means invoking the exported
function/constructor.

| #   | Function                     | Kind                                       | Behavior                                                                                                                                                                                                                                                                                                           | Acceptance ref |
| --- | ---------------------------- | ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------- |
| E1  | `normalizeSsoLoginError`     | authorization-denied (403-equivalent)      | Given an error shaped `{ code: 'SSO_ACCESS_DENIED' }`, `isSsoAccessDeniedError` returns `true` and `normalizeSsoLoginError` returns a `SsoAccessDeniedError` wrapping the cause                                                                                                                                    | ac2            |
| E2  | `normalizeSsoLoginError`     | session-expiry (401-equivalent, transient) | Given an already-constructed `SsoLoginTimeoutError` instance as input, `normalizeSsoLoginError` returns the same instance unchanged (the `instanceof Error` passthrough branch), confirming the timeout/transient case is never reclassified as access-denied; scenario text reads "try again", never "locked out" | ac2            |
| E3  | `normalizeSsoLoginError`     | generic passthrough                        | Given a plain `new Error('network failure')` with no SSO markers, `isSsoAccessDeniedError` returns `false` and `normalizeSsoLoginError` returns the same `Error` instance unchanged                                                                                                                                | ac1            |
| E4  | `normalizeSsoLoginError`     | generic wrap                               | Given a non-`Error` thrown value (e.g. a bare string), `normalizeSsoLoginError` returns a new `Error('Enterprise sign-in failed', { cause: error })`                                                                                                                                                               | ac1            |
| E5  | `PasswordResetRequiredError` | happy-path (construction)                  | `new PasswordResetRequiredError()` exposes `name: 'PasswordResetRequiredError'` and the fixed reset-required message, so callers can `instanceof`-discriminate it without going through `normalizeSsoLoginError`                                                                                                   | ac1            |

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

| Method                                      | Happy-path scenario                                                               | Error scenario                                                                             |
| ------------------------------------------- | --------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `list(publishedOnly?)`                      | Resolves `ManagedEnvironment[]` spanning a `PUBLISHED` and a `DRAFT` entry        | Rejects `ApiError 500`                                                                     |
| `capabilities()`                            | Resolves `EnvironmentCapabilities` with both `microvms`/`instances` compute flags | Rejects `ApiError 500`                                                                     |
| `getDefault()`                              | Resolves `DefaultEnvironment` naming `env-fixture-1`                              | Rejects `ApiError 404` (no default set)                                                    |
| `setDefault(environmentId)`                 | `setDefault('env-fixture-1')` resolves success                                    | Rejects `ApiError 404` for an unknown `environmentId`                                      |
| `get(environmentId)`                        | Resolves `EnvironmentDetail` including its `EnvironmentRevision[]` history        | Rejects `ApiError 404`                                                                     |
| `create(input)`                             | Resolves `EnvironmentMutationResult` with the new environment in `DRAFT` status   | Rejects `ApiError 400` for an invalid recipe                                               |
| `update(environmentId, input)`              | Resolves the updated `EnvironmentMutationResult`                                  | Rejects `ApiError 404`                                                                     |
| `build(environmentId, revisionId)`          | Resolves the revision transitioning to `BUILDING`                                 | Rejects `ApiError 409` for a revision already `BUILDING`                                   |
| `retry(environmentId, revisionId)`          | Resolves the `FAILED` revision transitioning back to `BUILDING`                   | Rejects `ApiError 409` for a revision not in `FAILED`                                      |
| `acceptFindings(environmentId, revisionId)` | Resolves the revision moving past `SECURITY_REVIEW` to `VERIFYING`                | Rejects `ApiError 409` when there are no findings to accept                                |
| `publish(environmentId, revisionId)`        | Resolves the revision `PUBLISHED`, environment's `publishedRevisionId` updated    | Rejects `ApiError 409` for a revision not `READY`                                          |
| `rebuild(environmentId)`                    | Resolves a freshly queued revision for the environment                            | Rejects `ApiError 404`                                                                     |
| `rebuildAll(environmentIds?)`               | `rebuildAll()` resolves a queued-rebuild result for every published environment   | Rejects/reports `ApiError 404` for one unknown id inside an explicit `environmentIds` list |
| `retire(environmentId)`                     | Resolves the environment moved to `RETIRED`                                       | Rejects `ApiError 404`                                                                     |
| `logs(environmentId, revisionId)`           | Resolves the build-log payload for a `BUILDING`/terminal revision                 | Rejects `ApiError 404` for a `DRAFT` revision with no logs yet                             |

**`toolsService`**

| Method                                         | Happy-path scenario                                                | Error scenario                                               |
| ---------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------ |
| `list(publishedOnly?)`                         | Resolves `ManagedTool[]` spanning a `PUBLISHED` and a `DRAFT` tool | Rejects `ApiError 500`                                       |
| `get(toolId)`                                  | Resolves `ManagedTool` with its `ManagedToolVersion[]` history     | Rejects `ApiError 404`                                       |
| `create(input)`                                | Resolves `ToolMutationResult` with the new tool in `DRAFT`         | Rejects `ApiError 400` for an invalid definition             |
| `update(toolId, input)`                        | Resolves the updated `ToolMutationResult`                          | Rejects `ApiError 404`                                       |
| `createVersion(toolId, definition)`            | Resolves a new `ToolVersionDefinition` for `arm64`                 | Rejects `ApiError 400` for an unsupported `ToolArchitecture` |
| `updateVersion(toolId, versionId, definition)` | Resolves the updated version definition                            | Rejects `ApiError 404`                                       |
| `build(toolId, versionId)`                     | Resolves the version transitioning to `BUILDING`                   | Rejects `ApiError 409` for a version already `BUILDING`      |
| `retry(toolId, versionId)`                     | Resolves the `FAILED` version transitioning back to `BUILDING`     | Rejects `ApiError 409` for a version not `FAILED`            |
| `acceptFindings(toolId, versionId)`            | Resolves the version moving past its security review               | Rejects `ApiError 409` when there are no findings to accept  |
| `publish(toolId, versionId)`                   | Resolves the version published                                     | Rejects `ApiError 409` for a version not ready               |
| `recommend(toolId, versionId)`                 | Resolves the version marked recommended                            | Rejects `ApiError 404`                                       |
| `logs(toolId, versionId)`                      | Resolves the build-log payload                                     | Rejects `ApiError 404` for a version with no logs yet        |

#### Chapter totals

- **135 scenario entries** across 15 files (34 Group A, 13 Group B admin-gated
  - 27×2=54 Group B environment/tool lifecycle, 14 Group C, 17 Group D, 5
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

### Chapter: u2-hooks

Owned by `comp-u2-hooks`, which delivers `req-u2-hooks` alone, depending on
`comp-plan-governance` and `comp-u1-services` (`contract-service-call-signatures`
is this unit's mock boundary for every hook that calls a `services/*` client).
Per `unit-of-work`'s `unit-u2-hooks` definition, `business-rules-u2-hooks`,
`domain-entities-u2-hooks`, and `security-design-u2-hooks`, this chapter covers
the 18 untested `frontend/src/hooks/*.ts` files. Every scenario below is one of
the three types `contract-scenario-type-bound` fixes — `render-interaction` (via
`renderHook`, never snapshot/shallow rendering), `call-return-throw` (a mocked
`services/*` call), or `state-transition` (a before/after on the state shapes
`domain-entities-u2-hooks` catalogs) — per Rule 1 of `business-rules-u2-hooks`'
Plan-Governance Rules.

#### Known discrepancy — `useSprintGraph.ts` does not exist; the file is `useIntentGraph.ts`

Direct inspection of `frontend/src/hooks/` (per `req-wont-have-verification`'s
evidence standard, applied here to a naming check rather than a Wont-Have
classification) found no `useSprintGraph.ts`. The file `domain-entities-u2-hooks`
describes under that name is `useIntentGraph.ts` — its own leading comment
reads "the v2 port of v1's `useSprintGraph`" — and its actual returned shape
differs from the `SprintGraphState` entity on file: it is `{ nodes, edges,
loading, error, getNeighbors, hasCodeTraceability, derivedItems, itemsByArtifact,
reload }`, not `{ graph, loading, error, reload, getNeighbors, hasRelationships,
nodeCount, edgeCount }`. Following this repository's guidance to investigate
such a discrepancy rather than trust either side at face value (and the
precedent `business-rules-u6-graph-components`' BR-7 set for a stale upstream
label), the two scenarios below are drafted against `useIntentGraph.ts`'s actual
exported shape, and this unit's `TraceabilityRow` for this file names
`useIntentGraph.ts` as `targetFile`, not the stale `useSprintGraph.ts` name.

#### Known gap — three untested `hooks/` files are outside this unit's named scope

Direct inspection of `frontend/src/hooks/` also found three untested files not
named by `req-u2-hooks` or `unit-of-work`'s `unit-u2-hooks` file list:
`useAutoSave.ts`, `useCollaborativeInception.ts`, and
`useCollaborativeIntentDraft.ts` (none has a sibling `.test.ts`, unlike
`useDiscussion.ts`, `useIntentEvents.ts`, and `useProjectsCache.ts`, which do and
are correctly excluded from both this chapter and `req-u2-hooks`). This chapter
does not draft scenarios for these three: `req-u2-hooks` and `unit-u2-hooks`
fix this unit's scope at exactly 18 files, and expanding that count is a
requirements-layer decision (per this repository's guidance to investigate
rather than silently assume either side is correct), not something a single
unit's code-generation pass makes unilaterally. These three files are flagged
here as a gap against the 90% goal for a future requirements-analysis
correction to `req-u2-hooks` or a new unit — they are not claimed as covered by
any row below, and they are not Wont-Have (each is a stateful hook with
independently testable behavior).

#### Scenario entries and traceability rows

Per `contract-traceability-ledger`, every row below is one `ScenarioEntry`
linked into this plan's single ledger, with `requirementId: req-u2-hooks`
(omitted from each row for brevity — no row in this chapter uses a different
requirement id). Fixture values follow `security-design-u2-hooks`' Fixture and
Artifact Hygiene Design: synthetic ids (`task-fixture-1`, `sprint-fixture-1`),
a synthetic execution ARN (`arn:aws:states:us-east-1:000000000000:execution:example:fixture-1`),
never a realistic-looking secret or a value adapted from a real execution.

| targetFile                            | behaviorDescription                                                                                                                                                                                    | scenarioType       | acceptanceRef                                           |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------ | ------------------------------------------------------- |
| `useAgentStatus.ts`                   | No-op transition — `refresh()` with both `currentArn` and the `executionId` prop absent makes no service call and leaves `status`/`questions` unchanged                                                | state-transition   | ac1                                                     |
| `useAgentStatus.ts`                   | Populated transition — `refresh()` with `currentArn` set resolves `status` from `agentsService.getCurrentExecution` and updates `status.status` through `RUNNING` → `SUCCEEDED`                        | state-transition   | ac1                                                     |
| `useAgentStatus.ts`                   | Duplicate/out-of-order rejection — dispatching an `agent.chunk` event whose `seq` the hook's `SeqDeduplicator` already accepted leaves `streamingText`/`toolCalls` unchanged                           | state-transition   | ac1; security-design-u2-hooks realtime-integrity design |
| `useAgentStatus.ts`                   | Routing-boundary isolation — an `agent.error` event carrying a non-null `agentTaskId` leaves this hook's agent-level state unchanged (task-level errors are `useConstructionStatus.ts`'s concern)      | state-transition   | ac1; security-design-u2-hooks realtime-integrity design |
| `useConstructionStatus.ts`            | No-op transition — `refreshTasks()` with `projectId`/`sprintId` and `executionArn` all absent makes no service call                                                                                    | state-transition   | ac1                                                     |
| `useConstructionStatus.ts`            | Orchestrator-routing isolation — an event with `agentTaskId` absent (or `'orchestrator'`) updates only `orchestratorStream`; `taskStreams` is unchanged                                                | state-transition   | ac1; security-design-u2-hooks realtime-integrity design |
| `useConstructionStatus.ts`            | Task-routing isolation — an event naming a specific `agentTaskId` updates only `taskStreams[agentTaskId]`; `orchestratorStream` and sibling task entries are unchanged                                 | state-transition   | ac1; security-design-u2-hooks realtime-integrity design |
| `useConstructionStatus.ts`            | Duplicate/out-of-order rejection — a repeated `seq` on an `agent.tool_update` event leaves the targeted stream's `toolCalls` unchanged                                                                 | state-transition   | ac1; security-design-u2-hooks realtime-integrity design |
| `useConstructionStatus.ts`            | `agent.completed` reclaims every non-`'orchestrator'` stream buffer from `taskStreams` while `orchestratorStream` survives unchanged                                                                   | state-transition   | ac1                                                     |
| `useAuthSession.ts`                   | Authenticated scenario — rendered with a fixture `user` supplied, `isAuthenticated === true` and every derived field (`userId`, `username`, `displayName`, `avatarUrl`) is populated from that fixture | render-interaction | ac2                                                     |
| `useAuthSession.ts`                   | Unauthenticated scenario — rendered with no session, `isAuthenticated === false`, `user === null`, and each derived field is explicitly asserted `toBeUndefined()`                                     | render-interaction | ac2                                                     |
| `useAuthSession.ts`                   | Loading scenario — rendered mid-resolution, `isLoading === true` while `isAuthenticated === false`, asserted independently of the unauthenticated scenario above                                       | render-interaction | ac1                                                     |
| `useGitProviderStatus.ts`             | No-provider scenario — `provider === ''` makes no network call; `status=null`, `error=null`, `loading=false`                                                                                           | state-transition   | ac2                                                     |
| `useGitProviderStatus.ts`             | Connected scenario — a resolved `GitProviderStatus` sets `status={connected:true,...}`                                                                                                                 | state-transition   | ac2                                                     |
| `useGitProviderStatus.ts`             | Disclosed-error scenario — a rejected call carrying `ApiError.body.error` surfaces that exact string as `error`                                                                                        | state-transition   | ac2; security-design-u2-hooks error-shape-reuse design  |
| `useGitProviderStatus.ts`             | Fallback-error scenario — a rejected call with no `body.error` surfaces the generic `Could not check {label} connection status.` string                                                                | state-transition   | ac2; security-design-u2-hooks error-shape-reuse design  |
| `useBlockLibraryRelease.ts`           | Precedence scenario — supplying both `requestedLibraryRelease` and `requestedRelease` resolves against `requestedLibraryRelease`                                                                       | state-transition   | ac1                                                     |
| `useBlockLibraryRelease.ts`           | Stable-fallback scenario — neither prop supplied resolves the stable catalog (`release.stable === true`)                                                                                               | state-transition   | ac1                                                     |
| `useBlockLibraryRelease.ts`           | Stale-request cancellation scenario — a rapid prop change's earlier in-flight request resolving after the newer one does not clobber the state the newer request already set                           | state-transition   | ac1                                                     |
| `useCollaborativeArtifact.ts`         | No-op-diff scenario — `setField` called with a value equal to the current `Y.Text` content against a mocked `useYjsDocument` return does not re-trigger a write                                        | state-transition   | ac3                                                     |
| `useCollaborativeArtifact.ts`         | Seed-once scenario — `initFields` after a remote edit already populated the field does not overwrite it                                                                                                | state-transition   | ac3                                                     |
| `useCollaborativeArtifact.ts`         | Autosave-gate scenario — autosave does not fire when `isEditing && synced && !!onAutoSave` is false on any one of its three conjuncts                                                                  | state-transition   | ac1; ac3                                                |
| `useCollaborativeArtifactContent.ts`  | No-op-diff scenario — `setContent` called with the current `Y.Text` value against a mocked `useYjsDocument` return does not re-trigger a write                                                         | state-transition   | ac3                                                     |
| `useCollaborativeArtifactContent.ts`  | Seed-once scenario — `initContent` after a remote edit already populated `contentText` does not overwrite it                                                                                           | state-transition   | ac3                                                     |
| `useCollaborativeArtifactContent.ts`  | Autosave-gate scenario — `flush()`'s autosave path does not fire when its enabled gate is false                                                                                                        | state-transition   | ac1; ac3                                                |
| `useCollaborativeStructuredAnswer.ts` | No-op-diff scenario — `setFreeText` called with the current text content against a mocked `useYjsDocument` return does not re-trigger a write                                                          | state-transition   | ac3                                                     |
| `useCollaborativeStructuredAnswer.ts` | Seed-once scenario — `initFromDraft` after a remote edit already populated a selection does not overwrite it                                                                                           | state-transition   | ac3                                                     |
| `useCollaborativeStructuredAnswer.ts` | `toStructuredAnswer()` scenario — an unanswered sub-question defaults to `{selectedOptions: [], freeText: undefined}` in the returned `StructuredAnswer`                                               | state-transition   | ac1; ac3                                                |
| `usePresence.ts`                      | Self-exclusion scenario — an awareness fixture including an entry for the hook's own `clientID` is excluded from the returned `users` list                                                             | render-interaction | ac3                                                     |
| `usePresence.ts`                      | Default-fill scenario — a remote awareness state carrying only `user` (no `presence`) resolves `name='Anonymous'`, `color='#888'`, `activity='idle'`                                                   | render-interaction | ac1; ac3                                                |
| `useObservability.ts`                 | `markBlocked` replaces an existing `blocked_question` entry for the same `sprintId` rather than duplicating it                                                                                         | state-transition   | ac1                                                     |
| `useObservability.ts`                 | `clearStuck` removes every `stuckDetections` entry for a sprint                                                                                                                                        | state-transition   | ac1                                                     |
| `useObservability.ts`                 | Idle-detection transition — a project moving from not-running to running-and-idle gets exactly one `idle` entry added, not a duplicate on a second poll                                                | state-transition   | ac1                                                     |
| `useObservability.ts`                 | Idle-detection transition — a project moving from running-and-idle to running-and-not-idle has its `idle` entry removed                                                                                | state-transition   | ac1                                                     |
| `useObservability.ts`                 | `refreshOnEvent` always calls `refreshProjects(true)` (forced) even when a prior cached fetch is fresh                                                                                                 | state-transition   | ac1                                                     |
| `useProjectSort.ts`                   | `loadStoredSort` falls back to `'activity'` when `localStorage.getItem('aidlc.projectSort')` holds an unrecognized value                                                                               | state-transition   | ac1                                                     |
| `useProjectSort.ts`                   | `loadStoredSort` falls back to `'activity'` when `localStorage` access throws                                                                                                                          | state-transition   | ac1                                                     |
| `useProjectSort.ts`                   | `setProjectSort`'s persistence failure (`localStorage.setItem` throws) does not prevent a second mounted hook instance from re-rendering with the new value                                            | state-transition   | ac1                                                     |
| `useProjectSort.ts`                   | `projectComparator`'s `activity` branch ties a `null` `lastActivityAt` to epoch-0 for ordering purposes                                                                                                | state-transition   | ac1                                                     |
| `useResizablePanel.ts`                | `clamp()` — `minWidth` wins even when it exceeds the viewport-fraction cap                                                                                                                             | state-transition   | ac1                                                     |
| `useResizablePanel.ts`                | `clamp()` — the viewport-fraction cap applies when it is the binding constraint                                                                                                                        | state-transition   | ac1                                                     |
| `useResizablePanel.ts`                | Anchor-dependent drag — `anchor='right'` inverts the drag-delta sign relative to `anchor='left'`                                                                                                       | render-interaction | ac1                                                     |
| `useResizablePanel.ts`                | Unmount-mid-drag cleanup — listeners registered by `handleResizeStart` are removed via the teardown ref on unmount                                                                                     | render-interaction | ac1                                                     |
| `useDebounce.ts`                      | Ordinary scenario — the debounced function fires once, after the configured delay, for a single call                                                                                                   | state-transition   | ac1                                                     |
| `useDebounce.ts`                      | Unmount-flush scenario — a pending debounced call is invoked, not dropped, on unmount                                                                                                                  | state-transition   | ac1                                                     |
| `useTrackerProviders.ts`              | A rejected `refresh()` call sets `providers=[]` and `failed=true` together                                                                                                                             | state-transition   | ac1                                                     |
| `useTrackerProviders.ts`              | A subsequent successful `refresh()` after a failure clears `failed` back to `false`                                                                                                                    | state-transition   | ac1                                                     |
| `useIntentGraph.ts`                   | `getNeighbors(id)` for an artifact with no edges returns `[]`, not `undefined`                                                                                                                         | state-transition   | ac1                                                     |
| `useIntentGraph.ts`                   | `getNeighbors(id)` filters out an edge referencing a node absent from the node index, rather than including it as a partial entry                                                                      | state-transition   | ac1                                                     |
| `useQuestionAnchor.ts`                | Bounded-retry scenario — the target element is found on the first 100 ms poll                                                                                                                          | render-interaction | ac1                                                     |
| `useQuestionAnchor.ts`                | Bounded-retry scenario — the target element is never found and polling gives up after 20 attempts                                                                                                      | render-interaction | ac1                                                     |
| `useQuestionAnchor.ts`                | `useQuestionLink` — the question is already rendered on the page: the hook scrolls to it and does not navigate                                                                                         | render-interaction | ac1                                                     |
| `useQuestionAnchor.ts`                | `useQuestionLink` — the question is not rendered: the hook navigates with the `#question-{id}` hash, gated on `projectId`/`sprintId` both being present                                                | render-interaction | ac1                                                     |
| `useMediaQuery.ts`                    | A query-string change re-subscribes to a new `MediaQueryList` and reflects its `matches` value                                                                                                         | state-transition   | ac1                                                     |
| `useSprintEvents.ts`                  | An event lacking `sprintId` always fires `onEvent`, regardless of the hook's own `sprintId` argument                                                                                                   | state-transition   | ac1                                                     |
| `useSprintEvents.ts`                  | An event carrying a `sprintId` that mismatches the hook's own `sprintId` argument is dropped (`onEvent` not called)                                                                                    | state-transition   | ac1                                                     |

Row count: 58, covering all 18 files `req-u2-hooks` names (`useSprintGraph.ts`
drafted against its actual current file, `useIntentGraph.ts`, per the
discrepancy note above), every row with `requirementId: req-u2-hooks` per Rule
2 of `business-rules-u2-hooks`. Both AC1's "primary transition + failure/empty
state" pairing and AC2's authenticated/unauthenticated/loading coverage for
`useAuthSession.ts` and `useGitProviderStatus.ts` are satisfied above, and AC3's
`renderHook`-only, no-live-Yjs-server constraint holds for every
`useCollaborativeArtifact*.ts`/`usePresence.ts` scenario, per
`security-design-u2-hooks`' mocked-transport-only design.

No file in this chapter is Won't-Have, per Rule 3 of `business-rules-u2-hooks`
— each of the 18 files exports at least one stateful hook with independently
testable behavior.

#### Dependency ordering

Per Rule 5 of `business-rules-u2-hooks`, the `useGitProviderStatus.ts` and
`useTrackerProviders.ts` failure-path scenarios above mock the exact `ApiError`
shape `contract-service-call-signatures` fixes (`comp-u1-services`' chapter
above) rather than inventing a parallel error shape — this unit's drafting
could only finalize those rows because `u1-services` already published that
contract.

#### Advisory framing and tech-stack conformance

Every scenario above reads as coverage this plan is building toward
(`req-advisory-alignment`): "the scenario verifies..." / "resolves..." /
"leaves ... unchanged" phrasing throughout, never implying an active CI gate.
No scenario proposes snapshot testing, shallow rendering, or an assertion on a
hook's internal `Y.Doc`/ref structure rather than its public return shape
(`req-stack-scenario-bound`; `business-rules-u2-hooks` Rule 1) — every
`render-interaction` row is phrased as a `renderHook` result assertion or a
Testing Library query, and every `state-transition`/`call-return-throw` row
asserts a before/after on the state shapes `domain-entities-u2-hooks` catalogs.

### Chapter: u3-contexts

Owned by `comp-u3-contexts`, which delivers `req-u3-contexts` alone, depending
on `comp-plan-governance`, `comp-u1-services`, and `comp-u2-hooks` per
`unit-of-work`'s `unit-u3-contexts` implementation note. Per
`business-rules-u3-contexts`, `domain-entities-u3-contexts`,
`security-requirements-u3-contexts`, `security-design-u3-contexts`, and
`tech-stack-decisions-u3-contexts`, this chapter covers the one untested file
in scope, `frontend/src/contexts/SprintContext.tsx` — `AuthContext.tsx` and
`IntentContext.tsx` are excluded per `req-u3-contexts`'s third acceptance
criterion, since both already carry passing test files. Every scenario below
is `render-interaction`, the single type `tech-stack-decisions-u3-contexts`
binds this unit to per `contract-scenario-type-bound` — no `call-return-throw`
or `state-transition` row appears in this chapter, since the file under test
is exercised exclusively through a harness consumer calling `useSprint()`,
never through a direct call to a mocked dependency or a `renderHook` of the
context itself. `requirementId` is `req-u3-contexts` for every row, per
`contract-traceability-ledger`.

#### Known gap — `useSprintGraph.ts` is not actually covered by the published `u2-hooks` chapter

Direct inspection of `frontend/src/hooks/` (per `req-wont-have-verification`'s
evidence standard, applied here to a naming/coverage check rather than a
Wont-Have classification) finds that `useSprintGraph.ts` **does exist** — it
is a real, longstanding file (present since the repository's initial
release), distinct from `useIntentGraph.ts` (whose own leading comment calls
it "the v2 port of v1's `useSprintGraph`"). `frontend/src/contexts/SprintContext.tsx`
imports `useSprintGraph` from `@/hooks/useSprintGraph` directly, and its
returned shape — `{ graph, loading, error, reload, getNeighbors,
hasRelationships, nodeCount, edgeCount }` — matches exactly what
`domain-entities-u3-contexts` and `business-rules-u3-contexts` already
describe as this unit's `u2-hooks` dependency boundary.

This contradicts the published `u2-hooks` chapter above, whose own "Known
discrepancy" note concluded `useSprintGraph.ts` "does not exist" and drafted
its two graph-lookup scenarios against `useIntentGraph.ts` instead — a
different file, with a materially different returned shape (`{ nodes, edges,
loading, error, getNeighbors, hasCodeTraceability, derivedItems,
itemsByArtifact, reload }`), that `useSprintGraph.ts` neither imports nor
forwards to. The practical effect: the real `useSprintGraph.ts` — the hook
this unit's own `SprintContext.tsx` depends on — has no traceability row of
its own anywhere in this plan.

Per this repository's guidance to investigate a discrepancy rather than trust
either side at face value, this chapter does not silently absorb
`useSprintGraph.ts` into its own scope: `req-u3-contexts` and `unit-of-work`'s
`unit-u3-contexts` definition fix this unit at exactly one file,
`SprintContext.tsx`, and widening that scope is a requirements-layer
correction, not a single unit's code-generation-time call (the same boundary
the `u2-hooks` chapter's own "Known gap" note already drew for its own three
extra files). This chapter instead mocks `useSprintGraph`'s real, confirmed
return shape at the module boundary (see the Graph pass-through and
Polling-tier rows below) and flags the gap here for a future correction to
`req-u2-hooks` or a new corrective unit.

#### Scenario entries and traceability rows

Per `contract-traceability-ledger`, every row below is one `ScenarioEntry`
drafted via `component-methods`' `draftScenario` and linked via
`linkTraceability` into this plan's single ledger. Fixture values follow
`security-design-u3-contexts` and `security-requirements-u3-contexts`'s
Fixture and Artifact Hygiene convention: synthetic route-param ids
(`project-fixture-1`, `sprint-fixture-1`), synthetic entity fixtures reusing
`u1-services`' own fixture shapes, and a synthetic `SprintGraphState`
(`graph`, `getNeighbors`, `reload` as `vi.fn()` mocks) — never a value copied
from a live environment. Every mocked `services/*` rejection reuses the exact
`ApiError` shape `contract-service-call-signatures` fixes (per
`business-rules-u3-contexts` Rule 5 and `security-design-u3-contexts`'s
Error-Shape Reuse design), not a self-invented shape.

| #   | `targetFile`                 | `behaviorDescription`                                                                                                                                                                                                                                                                                                                | `scenarioType`     | `acceptanceRef`                                         |
| --- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------ | ------------------------------------------------------- |
| 1   | `contexts/SprintContext.tsx` | Mount gate — rendered with `useParams` supplying an empty `projectId` or `sprintId`: none of the nine `services/*` calls (`sprintsService`…`timelineEventsService`) fire, and `loading` stays `true`                                                                                                                                 | render-interaction | ac1 (mount-gate guard)                                  |
| 2   | `contexts/SprintContext.tsx` | Providing state to a consumer — rendered with both `projectId`/`sprintId` present: all nine `services/*` calls resolve once each, a harness consumer's `useSprint()` observes the populated `SprintContextValue` fields, and `loading` transitions from `true` to `false` across that resolution                                     | render-interaction | ac1                                                     |
| 3   | `contexts/SprintContext.tsx` | Fail-open on service rejection — one of the nine `services/*` calls (e.g. `sprintsService.get`) mocked to reject with an `ApiError`: `error` stays `null` and `loading` still resolves to `false`, confirming the consumer gets no visible failure signal today                                                                      | render-interaction | derived invariant — silent-failure coverage obligation  |
| 4   | `contexts/SprintContext.tsx` | `reviewsService` reset-not-stale — `reviewsService.get` mocked to reject after an earlier successful resolution: the consumer's `review` becomes `null` (reset), not left at the prior value                                                                                                                                         | render-interaction | derived invariant                                       |
| 5   | `contexts/SprintContext.tsx` | Context update propagating to multiple consumers — two or more sibling harness consumers rendered under the same `SprintProvider`: once `reload()` resolves (or `reloadSprint()` is invoked through one consumer), every sibling consumer observes the identical updated `SprintContextValue`, not a stale or partially-updated copy | render-interaction | ac2                                                     |
| 6   | `contexts/SprintContext.tsx` | Polling tier — fast poll — with `sprint.currentAgentStatus` set to `'running'` or `'waiting'`, Vitest fake timers advanced to `10000`ms: `reloadSprint` is invoked at that interval                                                                                                                                                  | render-interaction | derived invariant                                       |
| 7   | `contexts/SprintContext.tsx` | Polling tier — slow poll and re-arm — a status change to any other value (including `null`) clears the prior interval and re-arms it at `60000`ms, asserted by advancing fake timers past the old `10000`ms mark with no additional call, then past `60000`ms with exactly one                                                       | render-interaction | derived invariant                                       |
| 8   | `contexts/SprintContext.tsx` | Graph pass-through — `getNeighbors` and `reloadGraph` on the consumed context value are exactly the same function references the mocked `useSprintGraph(safeSprintId)` returned (reference equality), confirming no re-derivation of graph traversal logic                                                                           | render-interaction | derived invariant — dependency-boundary non-duplication |
| 9   | `contexts/SprintContext.tsx` | `useSprint()` outside a provider — a harness consumer calling `useSprint()` without a wrapping `SprintProvider` throws `Error('useSprint must be used within a SprintProvider')`; new coverage for this file, not satisfied by `AuthContext.tsx`'s already-tested analogous guard                                                    | render-interaction | derived invariant — provider-missing guard              |

Row count: 9, all against `contexts/SprintContext.tsx`, every row
`requirementId: req-u3-contexts` per `contract-traceability-ledger` (omitted
from each row above for brevity, consistent with the `u1-services`/`u2-hooks`
chapters' convention). This resolves `req-plan-traceability`'s "every
in-scope file appears in exactly one row" constraint for this unit's single
file — no row is split under a different requirement id, and no row names
`AuthContext.tsx` or `IntentContext.tsx` (`business-rules-u3-contexts` Rule 2).

Both of `req-u3-contexts`'s scenario-bearing acceptance criteria are satisfied
directly: ac1 by rows 1–2 (and reinforced by rows 3–4's fail-open/reset
coverage), ac2 by row 5. The third acceptance criterion (excluding
`AuthContext.tsx`) is satisfied by omission — no row above names that file —
and row 9 confirms that exclusion does not extend to `SprintContext.tsx`'s own
analogous guard, per `business-rules-u3-contexts`.

No file in this chapter is Won't-Have, per `business-rules-u3-contexts` Rule 3
— the unit's single file exports a stateful provider and hook with
independently testable behavior.

#### Dependency-boundary and advisory-framing conformance

Rows 3, 4, and 8 above mock `u1-services`' `ApiError` shape and `u2-hooks`'
`useSprintGraph` return shape exactly as published
(`contract-service-call-signatures`, `contract-hook-state-shapes`), per
`business-rules-u3-contexts` Rule 5 and `security-design-u3-contexts`'s
Error-Shape Reuse and Graph-Lookup Mocking Boundary designs — no row
re-derives either shape independently. Every scenario above is phrased as
coverage the plan is building toward ("the scenario verifies…" /
"asserts…" / "confirms…"), never as behavior a CI gate already enforces, per
`req-advisory-alignment` and `business-rules-u3-contexts` Rule 4. No scenario
proposes snapshot testing, shallow rendering, or an assertion on
`SprintContext`'s internal React Context object rather than what a consumer
observes via `useSprint()`, per `req-stack-scenario-bound` and
`tech-stack-decisions-u3-contexts`'s scenario-type binding.

### Chapter: u4-pages

Owned by `comp-u4-pages`, which delivers `req-u4-pages` alone, depending on
`comp-plan-governance`, `comp-u2-hooks`, and `comp-u3-contexts` per
`unit-of-work`'s `unit-u4-pages` implementation note. Per
`business-rules-u4-pages`, `domain-entities-u4-pages`, and
`security-design-u4-pages`, this chapter covers 16 source-verified
`frontend/src/pages/*` files: the four shared-shape v1 read-only sprint-phase
pages (Group 1), the standalone OAuth callback route (Group 2), the two
observability pages (Group 3), the sprint knowledge-graph page (Group 4), the
three block-authoring pages (Group 5), the three workflow-authoring files
(Group 6), and the two composer pure-logic modules `phaseDisplay.ts` and
`workflowLibraryRelease.ts` (Group 7). Every scenario below is either
`render-interaction` (Groups 1–6, each rendered through a `MemoryRouter`) or
`call-return-throw` (Group 7, calling the exported function directly) — the
two types `contract-scenario-type-bound` binds this unit to, per
`business-rules-u4-pages` Rule 1; no `state-transition` row appears, since
this unit owns no hook or context of its own. `requirementId` is
`req-u4-pages` for every row below, per `contract-traceability-ledger`
(omitted from each row for brevity).

#### File-count reconciliation

`requirements`' `req-u4-pages` and `unit-of-work`'s `unit-u4-pages` state this
unit's scope as "the 17 untested `pages/` files plus
`pages/composer/phaseDisplay.ts`" (18 files). Direct inspection of
`frontend/src/pages/` for `domain-entities-u4-pages` found **16** files whose
primary rendered state and loading-or-error state are not exercised by any
existing test. This chapter's scenario list is drawn from those 16
source-verified files, per `req-wont-have-verification`'s direct-inspection
standard; the 2-file discrepancy against the stated "17" is recorded here for
reconciliation at the plan-governance level
(`comp-plan-governance.buildTraceabilityMatrix`) rather than silently resolved
toward either source.

#### Won't-Have verification for this unit

`pages/composer/defaultPhases.ts` was directly read (not folder-inferred): it
exports one typed constant array, `DEFAULT_PHASE_NODES: PhaseNodeInput[]`
(five literal phase objects), and no functions. It stays Won't-Have — no row
is added for it below, per `req-wont-have-verification` and `req-u4-pages`'s
third acceptance criterion.

| File                              | Status                                    | Evidence                                               |
| --------------------------------- | ----------------------------------------- | ------------------------------------------------------ |
| `pages/composer/defaultPhases.ts` | Confirmed Won't-Have — no scenario needed | Direct read: one static constant array, zero functions |

#### Dependency-boundary and contract-reuse note

Per `business-rules-u4-pages` Rule 6 and `security-design-u4-pages`'s
Dependency-Boundary Mocking design: every scenario mocking `useSprint()`
below reuses `contract-context-state-shape` (`u3-contexts`'s
`SprintContextValue`) verbatim; every scenario mocking `useAgentStatus`,
`useConstructionStatus`, or `useObservability` reuses `contract-hook-state-shapes`
(`u2-hooks`'s fixed state shapes) verbatim. No row below re-derives either
shape independently. `ObservabilityDashboard.tsx`'s `useProjectsCache()` is
outside `u2-hooks`'s published `contract-hook-state-shapes` catalog
(`domain-entities-u4-pages` Group 3's direct-inspection-gap note); the one
scenario mocking it (row 22 below) documents its fixture shape inline rather
than treating it as contract-covered.

#### Scenario entries and traceability rows

| #   | `targetFile`                               | `behaviorDescription`                                                                                                                                                                                                                                                                     | `scenarioType`     | `acceptanceRef`                              |
| --- | ------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------ | -------------------------------------------- |
| 1   | `pages/AgentPage.tsx`                      | Primary rendered state — populated `useSprint()` renders the sprint's name as heading, the unconditional "This v1 space is read-only" banner, and no start/edit/delete/answer control (asserted absent via `queryByRole`)                                                                 | render-interaction | ac1 (primary)                                |
| 2   | `pages/AgentPage.tsx`                      | Loading-or-error state — `sprint: null` from `useSprint()` renders the heading fallback literal `'Loading...'`                                                                                                                                                                            | render-interaction | ac1 (loading-or-error)                       |
| 3   | `pages/AgentPage.tsx`                      | `badgeStatus` derivation — each of the three `useAgentStatus` branches (`'RUNNING'`→`running`, `'SUCCEEDED'`→`completed`, any other truthy status→`failed`) renders the mapped value on `AgentStatusBadge`; an absent `status.status` renders no badge                                    | render-interaction | derived invariant (BR Group 1)               |
| 4   | `pages/InceptionPage.tsx`                  | Primary rendered state — populated `useSprint()` renders heading, read-only banner, no write affordance                                                                                                                                                                                   | render-interaction | ac1 (primary)                                |
| 5   | `pages/InceptionPage.tsx`                  | Loading-or-error state — `sprint: null` renders `'Loading...'` heading                                                                                                                                                                                                                    | render-interaction | ac1 (loading-or-error)                       |
| 6   | `pages/InceptionPage.tsx`                  | `badgeStatus` sprint-level fallback — no live `agentStatus.status`, but `sprint.currentAgentStatus === 'cancelled'`, renders the badge as `'failed'`                                                                                                                                      | render-interaction | derived invariant (BR Group 1)               |
| 7   | `pages/ConstructionPage.tsx`               | Primary rendered state — populated `useSprint()` renders heading, read-only banner, no write affordance                                                                                                                                                                                   | render-interaction | ac1 (primary)                                |
| 8   | `pages/ConstructionPage.tsx`               | Loading-or-error state — `sprint: null` renders `'Loading...'` heading                                                                                                                                                                                                                    | render-interaction | ac1 (loading-or-error)                       |
| 9   | `pages/ConstructionPage.tsx`               | `tasksByStatus` partitions a mixed `tasks` array into its four buckets (`todo`/`in_progress`/`done`/`failed`) correctly                                                                                                                                                                   | render-interaction | derived invariant (BR Group 1)               |
| 10  | `pages/ConstructionPage.tsx`               | Restore-last-execution effect fires `agentsService.getCurrentExecution` only when `sprint.phase === 'CONSTRUCTION'`; a non-`'CONSTRUCTION'` phase asserts the service is never called                                                                                                     | render-interaction | derived invariant (BR Group 1)               |
| 11  | `pages/ReviewPage.tsx`                     | Primary rendered state — populated `useSprint()` renders heading, read-only banner, no write affordance                                                                                                                                                                                   | render-interaction | ac1 (primary)                                |
| 12  | `pages/ReviewPage.tsx`                     | Loading-or-error state — `sprint: null` renders `'Loading...'` heading                                                                                                                                                                                                                    | render-interaction | ac1 (loading-or-error)                       |
| 13  | `pages/ReviewPage.tsx`                     | `sprintGraphService.get` fetches exactly once per distinct `sprintId` (not on every `reload()`/branch-update re-render); a mocked rejection is logged via `console.error`, not silently swallowed                                                                                         | render-interaction | derived invariant (BR Group 1)               |
| 14  | `pages/ReviewPage.tsx`                     | Single-PR backward-compatibility fallback — with `prs` empty but `sprint.prUrl`/`sprint.prNumber` populated, `activePrUrl`/`activePrNumber` resolve from the sprint fields, not from `prs`                                                                                                | render-interaction | derived invariant (BR Group 1)               |
| 15  | `pages/GitOAuthCallback.tsx`               | Primary rendered state (success, no stored return path) — mocked successful exchange navigates, after the fake-timer-advanced 1500ms delay, to the derived default (`gitProvider` mapped from `trackerProviderId`)                                                                        | render-interaction | ac1 (primary)                                |
| 16  | `pages/GitOAuthCallback.tsx`               | Primary rendered state (success, stored return path) — with `oauth_return_to` set in session storage, the same successful exchange navigates to that stored path instead                                                                                                                  | render-interaction | derived invariant (BR Group 2)               |
| 17  | `pages/GitOAuthCallback.tsx`               | Loading-or-error state (missing code) — `searchParams` has no `code`; `status: 'error'` is set synchronously with no `fetch` call made at all                                                                                                                                             | render-interaction | ac1 (loading-or-error)                       |
| 18  | `pages/GitOAuthCallback.tsx`               | Loading-or-error state (provider error) — a rejected or `success: false` mocked `fetch` renders the generic `Failed to connect {tabLabel}` (or server-provided `data.error`) message                                                                                                      | render-interaction | derived invariant (BR Group 2)               |
| 19  | `pages/ObservabilityLayout.tsx`            | Primary rendered state — `?space=`/`?sprint=` both present and matching a project already in `useObservability`'s `projects` list renders `IterationDetailView` instead of `ObservabilityDashboard`                                                                                       | render-interaction | ac1 (primary)                                |
| 20  | `pages/ObservabilityLayout.tsx`            | Loading-or-error state (deep link) — `?space=`/`?sprint=` name a project whose latest sprint differs from the requested one; a mocked `fetchSprintInfo` resolves and `IterationDetailView` renders with that fetched info                                                                 | render-interaction | ac1 (loading-or-error)                       |
| 21  | `pages/ObservabilityLayout.tsx`            | Unknown-project guard — an unresolvable `?space=` id, once `projectsLoading` is `false` and `projects.length > 0`, triggers `navigate('/observability', { replace: true })`, stripping the params                                                                                         | render-interaction | derived invariant (BR Group 3)               |
| 22  | `pages/ObservabilityDashboard.tsx`         | Primary rendered state — populated `filtered`/`activityFeed` (via a harness `ObservabilityLayout`-equivalent provider) renders `ActivityFeed`, `BusinessView`, `IntentStatusCards`, and `AgentStatusCards` with their expected props                                                      | render-interaction | ac1 (primary)                                |
| 23  | `pages/ObservabilityDashboard.tsx`         | Loading-or-error state (loading) — `projectsLoading && filtered.length === 0` renders `DashboardSkeleton` (asserted via its `aria-label="Loading observability data"` region)                                                                                                             | render-interaction | ac1 (loading-or-error)                       |
| 24  | `pages/ObservabilityDashboard.tsx`         | Loading-or-error state (error) — `projectsError && filtered.length === 0` renders the error card with a Retry button that calls `refresh()`                                                                                                                                               | render-interaction | derived invariant (BR Group 3)               |
| 25  | `pages/ObservabilityDashboard.tsx`         | Provider-missing guard — calling `useObservabilityContext()` outside `ObservabilityLayout` throws `'useObservabilityContext must be used within ObservabilityLayout'`                                                                                                                     | render-interaction | derived invariant (BR Group 3)               |
| 26  | `pages/SprintGraph.tsx`                    | Primary rendered state — a resolved `sprintGraphService.get` with populated `nodes`/`edges` passes them to `GraphCanvas` with `loading: false` (call `clearGraphCacheForTests()` first)                                                                                                   | render-interaction | ac1 (primary)                                |
| 27  | `pages/SprintGraph.tsx`                    | Loading-or-error state (loading) — `loading: true` is passed to `GraphCanvas` synchronously on first render, before the service promise resolves                                                                                                                                          | render-interaction | ac1 (loading-or-error)                       |
| 28  | `pages/SprintGraph.tsx`                    | Loading-or-error state (swallowed rejection) — a rejected `sprintGraphService.get` renders `GraphCanvas`'s `emptyState` with `loading: false` and empty `nodes`/`edges`, not a distinct error UI                                                                                          | render-interaction | derived invariant (BR Group 4)               |
| 29  | `pages/SprintGraph.tsx`                    | Cache-hit path — calling the component twice with the same `sprintId` after a resolved fetch renders the second time with `loading: false` immediately and no second service call                                                                                                         | render-interaction | derived invariant (BR Group 4)               |
| 30  | `pages/BlockLibrary.tsx`                   | Primary rendered state — a resolved `blocksService.list` renders the grid of block cards, each delete button appearing only on a non-`readOnly` block; an unrecognized `:type` route param defaults to the `'stage'` tab                                                                  | render-interaction | ac1 (primary)                                |
| 31  | `pages/BlockLibrary.tsx`                   | Loading-or-error state (loading) — three skeleton cards render while `loadBlocks` is in flight                                                                                                                                                                                            | render-interaction | ac1 (loading-or-error)                       |
| 32  | `pages/BlockLibrary.tsx`                   | Loading-or-error state (empty) — the empty-state copy branches between "no blocks yet" (`blocks.length === 0`) and "no blocks match {query}" (`filtered.length === 0` but `blocks.length > 0`)                                                                                            | render-interaction | derived invariant (BR Group 5)               |
| 33  | `pages/BlockLibraryReleaseSection.tsx`     | Primary rendered state — a populated `release` prop filters `release.blocks` by `searchQuery` into `visibleBlocks`, and a block whose `id` also exists in `libraryBlocks` (non-read-only) shows the "Replaced by the library block with this id" notice                                   | render-interaction | ac1 (primary)                                |
| 34  | `pages/BlockLibraryReleaseSection.tsx`     | Loading-or-error state (no release) — `release: null` renders nothing beyond a possible `error` alert                                                                                                                                                                                     | render-interaction | ac1 (loading-or-error)                       |
| 35  | `pages/BlockLibraryReleaseSection.tsx`     | Loading-or-error state (error) — a non-null `error` renders the `role="alert"` paragraph regardless of whether `release` is also present                                                                                                                                                  | render-interaction | derived invariant (BR Group 5)               |
| 36  | `pages/BlockEditor.tsx`                    | Primary rendered state — resolved `blocksService.get`/`getBody`/`getScript` populate the form fields from `detail.block`/`detail.body`/`detail.script`; the `isNew` branch (`id === 'new'`) renders a blank form with `loading: false` immediately and no service calls                   | render-interaction | ac1 (primary)                                |
| 37  | `pages/BlockEditor.tsx`                    | Loading-or-error state (error) — a rejected `loadBlockContent` renders the thrown message (or the generic `'Failed to load block'` fallback for a non-`Error` throw)                                                                                                                      | render-interaction | ac1 (loading-or-error)                       |
| 38  | `pages/BlockEditor.tsx`                    | Stale-response guard — a route-param change mid-flight (switching `id`) asserts the first request's late resolution does not overwrite the form with stale data                                                                                                                           | render-interaction | derived invariant (BR Group 5)               |
| 39  | `pages/BlockEditor.tsx`                    | `readOnly` is forced `true` whenever `releaseId` is present, even when the mocked block itself has `readOnly: false`                                                                                                                                                                      | render-interaction | derived invariant (BR Group 5)               |
| 40  | `pages/WorkflowList.tsx`                   | Primary rendered state — resolved `workflowsService.list` renders each card's SYSTEM badge/delete-button visibility per `wf.readOnly`, and a resolved `stableRelease` renders the "Workflow in {label} (stable)" section                                                                  | render-interaction | ac1 (primary)                                |
| 41  | `pages/WorkflowList.tsx`                   | Loading-or-error state (loading) — skeleton cards render while workflows are in flight                                                                                                                                                                                                    | render-interaction | ac1 (loading-or-error)                       |
| 42  | `pages/WorkflowList.tsx`                   | Loading-or-error state (error) — a rejected `aidlcReleasesService.catalog` with `?libraryRelease=` set renders the `role="alert"` "could not be loaded" message, and both "New Workflow" buttons stay disabled (`releaseReady === false`)                                                 | render-interaction | derived invariant (BR Group 6)               |
| 43  | `pages/WorkflowComposer.tsx`               | Primary rendered state — resolved parallel service calls (`workflowsService.get`, `blocksService.list('stage')`, `blocksService.list('scope')`) render the loaded composer UI (phase lanes, `BlockPalette`, `WorkflowScopeGraph`)                                                         | render-interaction | ac1 (primary)                                |
| 44  | `pages/WorkflowComposer.tsx`               | Loading-or-error state (loading) — the `if (loading) ...` branch renders before the parallel fetches resolve                                                                                                                                                                              | render-interaction | ac1 (loading-or-error)                       |
| 45  | `pages/WorkflowComposer.tsx`               | Loading-or-error state (error-or-not-found) — the combined `error \|\| !workflow` branch renders `{error ?? 'Not found'}` for both a thrown load error and a successfully-resolved-but-absent workflow                                                                                    | render-interaction | derived invariant (BR Group 6)               |
| 46  | `pages/composer/BlockPalette.tsx`          | Primary rendered state — `phaseFilter` plus text `filter` compose (a stage must match both to appear); a `placed` stage renders its "placed" badge with drag disabled and its Add button disabled, independent of `readOnly`                                                              | render-interaction | ac1 (primary)                                |
| 47  | `pages/composer/BlockPalette.tsx`          | Loading-or-error state (degenerate input) — the "No stages match." empty state renders when the combined filter excludes every stage                                                                                                                                                      | render-interaction | ac1 (loading-or-error)                       |
| 48  | `pages/composer/phaseDisplay.ts`           | `visibleWorkflowPhases` — the standard five-phase list excludes the `'initialization'` phase and is sorted by `path` ascending                                                                                                                                                            | call-return-throw  | ac2 (`visibleWorkflowPhases`)                |
| 49  | `pages/composer/phaseDisplay.ts`           | `isLegacyHiddenInitializationSkeleton` (via `displayPhasePathForPlacement`) — a legacy-shaped phase list (no `'initialization'` phase; `ideation`/`inception`/`construction`/`operation` at legacy paths) returns the placed stage's own semantic phase `path`, not `placement.phasePath` | call-return-throw  | ac2 (`isLegacyHiddenInitializationSkeleton`) |
| 50  | `pages/composer/phaseDisplay.ts`           | Non-legacy companion case — an `'initialization'` phase present (or no legacy-matching phases at all) returns `placement.phasePath` verbatim, confirming the guard does not misfire on a current-shape workflow                                                                           | call-return-throw  | derived invariant (BR Group 7)               |
| 51  | `pages/composer/workflowLibraryRelease.ts` | `loadWorkflowLibraryRelease(false, null)` with a mocked `aidlcReleasesService.stableCatalog` resolves a `WorkflowLibraryRelease` with `explicit: false`                                                                                                                                   | call-return-throw  | derived invariant (BR Group 7)               |
| 52  | `pages/composer/workflowLibraryRelease.ts` | `loadWorkflowLibraryRelease(false, requestedRelease)` calls `aidlcReleasesService.catalog` instead and resolves `explicit: true`                                                                                                                                                          | call-return-throw  | derived invariant (BR Group 7)               |
| 53  | `pages/composer/workflowLibraryRelease.ts` | `loadWorkflowLibraryRelease(true, ...)` (read-only workflow) returns `null` immediately with neither service mock invoked                                                                                                                                                                 | call-return-throw  | derived invariant (BR Group 7)               |
| 54  | `pages/composer/workflowLibraryRelease.ts` | A `stableCatalog`/`catalog` resolution of `null`, or a catalog with no `contents.blocks.scope`/`stage` entries, returns `null` or an empty-array-populated result rather than throwing                                                                                                    | call-return-throw  | derived invariant (BR Group 7)               |

#### Chapter totals

- **54 scenario entries** across the 16 source-verified files in this unit's
  scope (14 rows for Group 1's four shared-shape pages, 4 for Group 2, 7 for
  Group 3's two pages, 4 for Group 4, 10 for Group 5's three files, 6 for
  Group 6's three files, 7 for Group 7's two pure-logic modules), every one
  appending a `TraceabilityRow` with `requirementId: req-u4-pages`.
- `req-u4-pages`'s first acceptance criterion (one primary-rendered-state and
  one loading-or-error scenario per untested page) is satisfied by rows 1–2,
  4–5, 7–8, 11–12, 15+17, 19–20, 22–23, 26–27, 30–31, 33–34, 36–37, 40–41,
  43–44, and 46–47; its second acceptance criterion is satisfied by rows
  48–49; its third is satisfied by the Won't-Have table above.
- No row in this chapter proposes snapshot testing, shallow rendering, or an
  assertion on internal state a consumer cannot observe, per
  `req-stack-scenario-bound` and `business-rules-u4-pages` Rule 1.
- Framing stays advisory throughout, per `req-advisory-alignment`: every row
  above is phrased as coverage the plan is building toward ("the scenario
  verifies…" / "renders…" / "resolves…"), never as behavior a CI gate already
  enforces.

### Chapter: u5-intent-components

Owned by `comp-u5-intent-components`, delivering `req-u5-intent-components`
alone per `unit-of-work`'s `unit-u5-intent-components` entry and
`unit-of-work-story-map`'s `map-u5-intent-components` mapping. Per
`business-rules-u5-intent-components`, `domain-entities-u5-intent-components`,
and `security-design-u5-intent-components`, this chapter covers the 19
untested files in `frontend/src/components/intent/` — confirmed by direct
read of the directory: every other file in that directory (`ArtifactContentEditor`,
`ArtifactHistoryDrawer`, `ArtifactHtmlFrame`, `ArtifactMarkdown`,
`ArtifactViewer`, `CodeSection`, `ComposePanel`, `DeliveryStatus`,
`IntentGraphPopover`, `MethodologyReleaseBadge`, `NativeExportSetupDialog`,
`PendingQuestionsTabs`, `ProvenanceTree`, `QuorumEditPanel`, `RecomposePanel`,
`SensorChips`, `SensorDiagnostics`, `StageDetail`, `StageReviewPanel`,
`TrackerAttachmentImport`, `UnitLaneBoard`, `UsageMetrics`,
`artifactReferences`, `documentHelpers`) already carries a passing
`*.test.tsx`/`*.test.ts` sibling. Per Rule 1 of `business-rules-u5-intent-components`
and `contract-scenario-type-bound`, every scenario below is `render-interaction`
except the six scenarios exercising `ArtifactEditControls.tsx`'s three direct
`intentsService.*` calls, which are `call-return-throw` per
`contract-service-call-signatures`. `requirementId` is `req-u5-intent-components`
for every row, per `contract-traceability-ledger`.

#### Known discrepancy — the stated `u3-contexts` dependency does not hold

`unit-of-work` and `requirements` both state this unit depends on `u1-services`,
`u2-hooks`, and `u3-contexts`. Direct inspection of all 19 files (grep for
`useIntent`/`useSprint` imports across `GateCard.tsx`, `AgentProgressCard.tsx`,
`HistorySection.tsx`, `WorkProductsSection.tsx`, `IntentConfigurationDialog.tsx`,
`IntentPhaseDiagram.tsx`, and `IntentStageList.tsx`) confirms every file that
reads shared run state imports `useIntent` from `@/contexts/IntentContext`,
never `useSprint` from `u3-contexts`' `SprintContext.tsx`. `IntentContext.tsx`
already has a passing test file and is excluded from `req-u3-contexts`' own
scope ("the one untested context"). Per `business-rules-u5-intent-components`
Rule 4 and the dependency-boundary design in `security-design-u5-intent-components`,
every scenario below that mocks shared intent state mocks `IntentContextValue`'s
fields directly; no scenario in this chapter constructs, imports, or mocks a
`SprintProvider` or `SprintContextValue` — doing so would test the wrong
contract. `contract-context-state-shape`'s unit-pack listing names this unit
as a consumer of the SprintContext shape; this chapter records that as not
concretely realized, mirroring the precedent `business-rules-u2-hooks` set
for `useSprintGraph.ts` and `business-rules-u9-discussion-components` set for
its own undeclared-dependency note. The `u1-services` edge (`ArtifactEditControls.tsx`'s
three `intentsService.*` calls) and the `u2-hooks` edge (`useIntentGraph` in
`GateCard.tsx`, `DerivedItemRow.tsx`, `WorkProductsSection.tsx` only — confirmed
by direct read) both hold as stated.

#### Undeclared cross-unit edge — `DiscussButton` (`comp-u9-discussion-components`)

`DerivedItemRow.tsx` and `HistorySection.tsx`'s `CompactQuestionRow` both
render `<DiscussButton />` from the already-covered `components/discussion/`
directory (`req-u9-discussion-components`, merged ahead of this unit per
`unit-of-work-story-map`'s ordering). Per `business-rules-u5-intent-components`
Rule 5 and the cross-unit trust-boundary design in
`security-design-u5-intent-components`, no scenario below re-derives
`DiscussButton`'s own behavior contract — that coverage belongs to
`u9-discussion-components`. Each of the two host files instead gets a
with-provider and a without-provider variant, confirming the host file's own
content renders correctly either way and that `DiscussButton` renders `null`
with no mounted `DiscussionProvider` (its documented fail-safe default).

#### Won't-Have verification

Per `req-wont-have-verification` and Rule 7 of `business-rules-u5-intent-components`,
all 19 files in scope export at least one component or function with
independently testable, non-trivial behavior — none is a pure re-export
barrel, type-only module, or static constant with no functions. No file in
this chapter is marked Won't-Have.

#### Acceptance-criteria coverage

`req-u5-intent-components`'s three acceptance criteria map as follows: ac1
("each untested file has at least one scenario asserting rendered output for
its primary prop/state combination") is satisfied once per file by the first
row listed for that file below; ac2 (`GateCard` and `WaitingCard` each get a
human-gate-pending scenario) is satisfied by the `question`-kind `GateCard`
row and the primary-lookup `WaitingCard` row, both called out explicitly
below; ac3 (`StageGridEditor` and `IntentStageList` each get an empty/zero-stage
scenario) is satisfied by the `stages: []` and `stageRows: []` rows below.
Rows beyond a file's first are additional coverage required by
`business-rules-u5-intent-components`'s per-file rules (BR12–BR16), by a
named scenario design in `security-design-u5-intent-components`, or by a
per-file variant catalogued in `domain-entities-u5-intent-components`' Entity
Catalog clusters — each cited inline.

#### Scenario entries and traceability rows

Per `contract-traceability-ledger`, every row below is one `ScenarioEntry`
linked into this plan's single ledger, with `requirementId: req-u5-intent-components`
(omitted from each row for brevity — no row in this chapter uses a different
requirement id). Fixture values follow `security-design-u5-intent-components`'s
Fixture and Artifact Hygiene Design: synthetic identifiers (`intent-fixture-1`,
`gate-fixture-1`, `artifact-fixture-1`), never a real intent id, artifact
body, or session token.

| targetFile | behaviorDescription | scenarioType | acceptanceRef |
| --- | --- | --- | --- |
| `AgentProgressCard.tsx` | Running, no pending gate: shows "Running", stage/phase badges, elapsed time, and the last two non-blank lines of `outputBuffers` | render-interaction | ac1 |
| `AgentProgressCard.tsx` | Waiting, no pending gate: shows "Resuming" and a lifecycle-event summary instead of output lines | render-interaction | derived invariant (BR12) |
| `AgentProgressCard.tsx` | Waiting WITH a pending gate: delegates entirely to `WaitingCard`, passing `intent`, `gates`, `stageRows`, `stageNameOf`, `rewindIntent` — `WaitingCard`'s content renders instead of the running UI | render-interaction | derived invariant (BR12) |
| `WaitingCard.tsx` | Human-gate-pending baseline: one `pending` gate matching `intent.pendingHumanTaskId` and a `WAITING_FOR_HUMAN` stage row renders "Waiting for your input", the resolved stage name, and the question preview text | render-interaction | ac2 |
| `WaitingCard.tsx` | Fallback lookup: no gate's `humanTaskId` matches `intent.pendingHumanTaskId` but ≥2 gates have `status: 'pending'` — the rendered preview reflects the LAST pending gate in array order (`findLast` semantics), not the first | render-interaction | security-design (gate-lookup fallback) |
| `WaitingCard.tsx` | "Restart stage" → confirm dialog → `rewindIntent(stageId)` called, dialog closes on success | render-interaction | ac1 |
| `WaitingCard.tsx` | "Restart stage" rejection: `rewindError` is shown and the confirm dialog stays open | render-interaction | derived invariant (domain-entities Cluster A) |
| `WaitingCard.tsx` | "Restart with guidance": textarea required non-empty; submitting calls `rewindIntent(stageId, guidance.trim())` | render-interaction | derived invariant (domain-entities Cluster A) |
| `ArtifactBodyStatus.tsx` | `contentUnavailable === 'too_large'` renders the too-large status text | render-interaction | ac1 |
| `ArtifactBodyStatus.tsx` | `contentUnavailable === 'error'` renders `role="alert"` text, with the Retry button present only when `onRetry` is supplied and absent when it is not | render-interaction | derived invariant (BR12) |
| `ArtifactBodyStatus.tsx` | `content === undefined` (still loading) renders `role="status"` spinner text | render-interaction | derived invariant (BR12) |
| `ArtifactBodyStatus.tsx` | None of the three conditions hold: the component renders `null` | render-interaction | derived invariant (BR12) |
| `ArtifactContent.tsx` | `rendersAsHtml(artifactType, content)` true: renders `ArtifactHtmlFrame` with `content`/`title` | render-interaction | ac1 |
| `ArtifactContent.tsx` | `rendersAsHtml` false: renders `ArtifactMarkdown` with all passed props | render-interaction | derived invariant (BR12) |
| `ArtifactEditControls.tsx` | `useArtifactEditability`: `artifact.supersededAt` set returns the "Superseded by a rewind…" reason, checked before run status | render-interaction | ac1 |
| `ArtifactEditControls.tsx` | `useArtifactEditability`: `detail.intent.status` of `CREATED`/`RUNNING` returns the "wait for the stage to park or finish" reason | render-interaction | derived invariant (domain-entities Cluster B) |
| `ArtifactEditControls.tsx` | `useArtifactEditability`: any `detail.quorumEdits[].state` in `{PLANNING, AWAITING_APPROVAL, APPLYING}` returns the "Quorum edit is in progress" reason | render-interaction | derived invariant (domain-entities Cluster B) |
| `ArtifactEditControls.tsx` | `useArtifactEditability`: none of the above holds, result is `null` (editable) | render-interaction | derived invariant (domain-entities Cluster B) |
| `ArtifactEditControls.tsx` | `ArtifactStaleBadge` renders `null` when `artifact.staleSince` is falsy | render-interaction | ac1 |
| `ArtifactEditControls.tsx` | `ArtifactStaleBadge` renders the "possibly stale" badge and "Mark verified" button when `artifact.staleSince` is set | render-interaction | derived invariant (domain-entities Cluster B) |
| `ArtifactEditControls.tsx` | `ArtifactStaleBadge` verify success: `intentsService.verifyArtifact` resolves, `reload()` is called, the badge is gone and the button is re-enabled | call-return-throw | security-design (mutating-action pairing) |
| `ArtifactEditControls.tsx` | `ArtifactStaleBadge` verify rejection: the thrown error is swallowed (logged via `console.error`, not surfaced in the UI) and `reload()` is NOT called | call-return-throw | security-design (mutating-action pairing) |
| `ArtifactEditControls.tsx` | "Edit document" impact-fetch success: `intentsService.artifactImpact` resolves, the drift-warning dialog opens showing `consumedBy`/`downstream` counts (including the zero-consumer, zero-downstream copy variant) | call-return-throw | ac1 |
| `ArtifactEditControls.tsx` | "Edit document" impact-fetch rejection (fail-closed): `error` is set to "Could not load the impact data — try again", the warning dialog never opens, and `onStartEdit` is never invoked | call-return-throw | security-design (quorum-edit safeguard, fail-closed) |
| `ArtifactEditControls.tsx` | From the drift-warning dialog, "Edit anyway" closes it and calls `onStartEdit()` — it does NOT open the Quorum dialog | render-interaction | derived invariant (domain-entities Cluster B) |
| `ArtifactEditControls.tsx` | From the drift-warning dialog, "Edit with Quorum" closes it and opens the Quorum dialog instead — it does NOT call `onStartEdit` | render-interaction | derived invariant (domain-entities Cluster B) |
| `ArtifactEditControls.tsx` | The Quorum dialog's submit button is disabled until `changeDescription.trim()` is non-empty | render-interaction | derived invariant (domain-entities Cluster B) |
| `ArtifactEditControls.tsx` | Quorum submit success: `intentsService.startQuorumEdit(projectId, intentId, artifact.id, description)` resolves; the dialog closes, `changeDescription` clears, and `reload()` is called | call-return-throw | security-design (mutating-action pairing) |
| `ArtifactEditControls.tsx` | Quorum submit rejection with an `Error` instance: `error` is set to `err.message`; the dialog stays open | call-return-throw | security-design (mutating-action pairing) |
| `ArtifactEditControls.tsx` | Both "Edit document" and "Edit with Quorum" are disabled whenever `blocked` is non-null; "Edit document" is additionally disabled while `impactLoading` | render-interaction | derived invariant (domain-entities Cluster B) |
| `DerivedItemCountChip.tsx` | `count === 0` renders `null` | render-interaction | ac1 |
| `DerivedItemCountChip.tsx` | `count > 0` renders the "N items" chip with correct singular/plural title text; clicking calls `focusWorkProduct({ kind: 'item', id: '', filterArtifactId: artifactId })` without propagating the click | render-interaction | derived invariant (domain-entities Cluster F) |
| `DerivedItemRow.tsx` | Clicking the row, or pressing Enter/Space while it is focused, calls `onPreview()` | render-interaction | ac1 |
| `DerivedItemRow.tsx` | `item.priority` present shows the priority badge; `item.priority` absent hides it | render-interaction | derived invariant (domain-entities Cluster F) |
| `DerivedItemRow.tsx` | `showGraph={false}` hides `IntentGraphPopover` entirely rather than passing it empty neighbors | render-interaction | derived invariant (domain-entities Cluster F) |
| `DerivedItemRow.tsx` | Rendered with a mocked `DiscussionProvider` mounted: the row's own content renders correctly alongside a present `DiscussButton` | render-interaction | security-design (cross-unit trust, with-provider) |
| `DerivedItemRow.tsx` | Rendered with no `DiscussionProvider`: the row's own content still renders correctly and `DiscussButton` renders `null` | render-interaction | security-design (cross-unit trust, without-provider) |
| `GateCard.tsx` | `question`-kind gate with parseable `questions` JSON renders `QuestionEditor` with the parsed `Question` and the `references` object built from `detail.artifacts`/mocked `useIntentGraph().derivedItems`; a non-blank steering note is included as `{ steering }` on `onAnswer` | render-interaction | ac2 |
| `GateCard.tsx` | Same `question`-kind gate with the steering note left blank: `onAnswer`'s second argument has NO `steering` key at all (not `steering: ''`) | render-interaction | security-design (steering-payload integrity) |
| `GateCard.tsx` | `validation`-kind gate renders `stageArtifacts` badges and a "Review stage" button that navigates to `/space/:projectId/intent/:intentId/review/:humanTaskId` | render-interaction | derived invariant (domain-entities Cluster C) |
| `GateCard.tsx` | `review-verdict`-kind gate renders `gate.options` as buttons (or the default Approve/Reject pair when `options` is empty), each calling `onAnswer(gate, { answer: opt, status: /reject/i.test(opt) ? 'rejected' : 'answered' })` | render-interaction | derived invariant (domain-entities Cluster C) |
| `GateCard.tsx` | `engineGateStatusFor` mapping: `/^(reject\|request-changes)/i` → `'rejected'`, `/^(approve\|accept-as-is\|override-and-approve)/i` → `'approved'` (including `override-and-approve`), anything else → `'answered'` | render-interaction | derived invariant (domain-entities Cluster C) |
| `GateCard.tsx` | An engine-approval option prefixed `request-changes` reveals the optional feedback `Textarea`; submitting with non-empty feedback sends `{ decision: opt, feedback }` | render-interaction | derived invariant (domain-entities Cluster C) |
| `GateCard.tsx` | `walkingSkeletonBranch`: `gate.prompt` starting with `'Walking skeleton'` and `gate.unitSlug` set strips the `(branch …)` suffix and renders a `UnitBranchEntry` row beneath the prompt | render-interaction | derived invariant (domain-entities Cluster C) |
| `HistorySection.tsx` | Both `historicalGates` (gates with `status !== 'pending'`) and `steering` empty: renders `null` | render-interaction | ac1 |
| `HistorySection.tsx` | Historical gates group by `stageInstanceId`, sorted by plan position (`phase`, then `order`) derived from `stageRows` — a non-arrival-order input still groups and orders by plan position, not by gate-arrival order | render-interaction | derived invariant (BR16) |
| `HistorySection.tsx` | Within a group, gates sort chronologically by `answeredAt ?? createdAt` | render-interaction | derived invariant (BR16) |
| `HistorySection.tsx` | `CompactQuestionRow`'s "Revise answer" affordance appears only when the gate is answered (`status !== 'pending'` or `answeredAt` set) AND `detail.intent.status` is not `SUCCEEDED`/`CANCELLED` | render-interaction | derived invariant (domain-entities Cluster E) |
| `HistorySection.tsx` | Revise submit success: `reviseGate(gate, revision.trim())` is called and the form clears | render-interaction | derived invariant (domain-entities Cluster E) |
| `HistorySection.tsx` | Revise submit failure: `reviseError` is set and the form stays populated | render-interaction | derived invariant (domain-entities Cluster E) |
| `HistorySection.tsx` | A `status === 'superseded'` gate shows "Retired unanswered." instead of an answer block, with a muted-gray status dot | render-interaction | derived invariant (domain-entities Cluster E) |
| `HistorySection.tsx` | `formatGateAnswer` resolves a plain string answer and a `{ decision }` object answer | render-interaction | derived invariant (domain-entities Cluster E) |
| `HistorySection.tsx` | `formatGateAnswer` resolves a structured `{ answers: [{ selectedOptions, freeText }] }` object, mapping `selectedOptions` indices back to each question's option labels | render-interaction | derived invariant (domain-entities Cluster E) |
| `HistorySection.tsx` | `formatGateAnswer`'s fallback `JSON.stringify` path for an unrecognized answer shape | render-interaction | derived invariant (domain-entities Cluster E) |
| `IntentConfigurationDialog.tsx` | Renders `null` when `!detail` | render-interaction | ac1 |
| `IntentConfigurationDialog.tsx` | Status badge renders the correct icon/style for each of `waiting`/`running`/`succeeded`/`failed` (derived from `intent.status`, including `CREATED` folding into `running`) | render-interaction | derived invariant (BR13) |
| `IntentConfigurationDialog.tsx` | "Reshape" button renders only when `onOpenReshape` is supplied and is absent otherwise; clicking it calls `onOpenChange(false)` BEFORE invoking `onOpenReshape()` | render-interaction | derived invariant (BR13) |
| `IntentConfigurationDialog.tsx` | The environment section renders "No environment snapshot was captured for this run." when `intent.environment` is absent | render-interaction | derived invariant (BR13) |
| `IntentPhaseDiagram.tsx` | `!compiled \|\| !workflowPhases`: renders three pulsing skeleton bars (`aria-hidden="true"`) and no phase content | render-interaction | ac1 |
| `IntentPhaseDiagram.tsx` | `compiled`/`workflowPhases` present but grouping `stageRows` by phase (after excluding `initializationPhasePaths`) yields `[]`: renders "No phases resolved yet." | render-interaction | derived invariant (BR12) |
| `IntentPhaseDiagram.tsx` | The active phase (per `derivePhaseState`) always renders expanded with a pulsing live-indicator dot and ignores `expandedManual` when clicked | render-interaction | derived invariant (BR12) |
| `IntentPhaseDiagram.tsx` | A non-active phase defaults to collapsed and toggles via `expandedManual` on click; a `done` non-active phase shows a check icon instead of the pending dot | render-interaction | derived invariant (BR12) |
| `IntentStageList.tsx` | `stageRows: []` renders the literal fallback text "No stages resolved yet." with no `Accordion` element present | render-interaction | ac3 |
| `IntentStageList.tsx` | A single-row, single-phase input renders exactly one `AccordionItem` with that phase's name and a `1/1`-style progress readout once the row's state is `SUCCEEDED` | render-interaction | ac1 |
| `IntentStageList.tsx` | A non-contiguous arrangement — unit-lane rows for slug A, then a flat row, then more unit-lane rows for slug A again — renders TWO separate per-unit segments, not one merged segment, because segmentation is positional, not a global group-by | render-interaction | derived invariant (BR16) |
| `IntentStageList.tsx` | A `FAILED`, `planned: true` row with `detail.intent.status` in `{SUCCEEDED, FAILED, WAITING, CANCELLED}` shows the retry icon-button, which calls `rewindIntent(row.stageId)` and swallows a thrown rejection; a row that is not `planned`, not `FAILED`, or whose intent status is `RUNNING`/`CREATED` does NOT show the retry button | render-interaction | derived invariant (domain-entities Cluster D) |
| `ScopeBadge.tsx` | `scopeColor(scope)` is deterministic: the SAME scope string maps to the SAME of the six palette entries across repeated calls | render-interaction | derived invariant (BR14) |
| `StageGridEditor.tsx` | `stages: []` renders the `data-testid="stage-grid-editor"` container present with zero `data-testid="grid-stage-*"` rows and zero phase `<Label>` groups (the component does NOT return `null`) | render-interaction | ac3 |
| `StageGridEditor.tsx` | Populated: one phase group per distinct `phasePath` (including the `''`/`'unphased'` fallback label), sorted by `phasePath`, then `order`, then `stageId` | render-interaction | ac1 |
| `StageGridEditor.tsx` | Each checkbox's `checked` reflects `grid[stageId] === 'EXECUTE'`; clicking an unlocked, non-disabled row calls `onToggle(stageId)` | render-interaction | derived invariant (domain-entities Cluster D) |
| `StageGridEditor.tsx` | A row with `lockedStageIds.has(stageId)` renders disabled with the lock icon and `onToggle` does NOT fire on click | render-interaction | derived invariant (domain-entities Cluster D) |
| `StageGridEditor.tsx` | A row is disabled and `onToggle` does NOT fire when the `disabled` prop is true, independent of `lockedStageIds` | render-interaction | derived invariant (domain-entities Cluster D) |
| `UnitBranchEntry.tsx` | `item.branch` falsy renders `null` | render-interaction | ac1 |
| `UnitBranchEntry.tsx` | `item.targets.length === 0` renders the single-row fallback showing the branch name with no link | render-interaction | derived invariant (domain-entities Cluster F) |
| `UnitBranchEntry.tsx` | `item.targets.length > 0` renders one row per target: a link when `target.url` is set, otherwise plain `<code>` text | render-interaction | derived invariant (domain-entities Cluster F) |
| `UnitBranchEntry.tsx` | The "Open PR #N" affordance renders only when `showPullRequests` is true AND `target.prUrl` is set; it is absent when either condition fails | render-interaction | derived invariant (domain-entities Cluster F) |
| `WorkProductsSection.tsx` | Zero artifacts, zero question gates, zero steering, zero code items, AND `detail.intent.status` not in `{CREATED, RUNNING, WAITING}`: renders `null` entirely | render-interaction | ac1 |
| `WorkProductsSection.tsx` | Same zero counts but `detail.intent.status` in `{CREATED, RUNNING, WAITING}`: still renders the Card shell with the "No work products yet…" placeholder | render-interaction | derived invariant (domain-entities Cluster E) |
| `WorkProductsSection.tsx` | `detail.artifacts.length > 0` or non-empty `codeItems`: renders `ProvenanceTree` with the assembled props instead of the placeholder | render-interaction | derived invariant (domain-entities Cluster E) |
| `WorkProductsSection.tsx` | The sort button appears only when `documentCount > 1`; clicking it flips `newestFirst` and persists via `setDocumentOrder` | render-interaction | derived invariant (domain-entities Cluster E) |
| `WorkProductsSection.tsx` | A non-empty `detail.intent.planWarnings` renders the collapsible warnings `<details>`, but only while `detail.intent.status !== 'SUCCEEDED'` | render-interaction | derived invariant (domain-entities Cluster E) |
| `artifactAccent.ts` | `null`/`undefined` input returns the fixed `FALLBACK` accent | render-interaction | ac1 |
| `artifactAccent.ts` | A non-empty string deterministically hashes onto one of the 7 entries in `ACCENTS`, the SAME entry across repeated calls with the same input | render-interaction | derived invariant (BR14) |
| `stageStyle.tsx` | `StageBadge` renders the correct label/style for each of the 6 `StageState` values, and falls back to `PENDING`'s style for an unrecognized value | render-interaction | ac1 |
| `stageStyle.tsx` | `formatDuration(null)` returns `null`; a set start returns `null` end-state formatting | render-interaction | derived invariant (domain-entities Cluster F) |
| `stageStyle.tsx` | `formatDuration` boundary formatting across sub-minute (`<60s`), minute (`<60m`), and hour (`>=60m`) ranges | render-interaction | derived invariant (domain-entities Cluster F) |
| `stageStyle.tsx` | `stageDurations`' "still parked" branch (`parkedAt` set, no `completedAt`) folds the open park window into `waitedMs` live; `waiting` is `null` when `waitedMs < 1000` | render-interaction | derived invariant (domain-entities Cluster F) |
| `stageStyle.tsx` | `useTick(active)` with fake timers: a re-render tick occurs every second only while `active` is true and stops cleanly (no lingering interval, no post-unmount `act` warning) once `active` flips `false` | render-interaction | derived invariant (BR15) |
| `workProductsFocus.ts` | A subscribed listener via `onWorkProductFocus` receives exactly the `WorkProductFocus` object passed to `focusWorkProduct` | render-interaction | ac1 |
| `workProductsFocus.ts` | After calling the returned unsubscribe function, the listener receives no further calls | render-interaction | derived invariant (domain-entities Cluster F) |
| `workProductsFocus.ts` | `scrollAndFlash` happy path: the element exists, `scrollIntoView` is called and the flash classes toggle off after the timeout | render-interaction | derived invariant (domain-entities Cluster F) |
| `workProductsFocus.ts` | `scrollAndFlash` retry path, with fake timers: the element is missing, the function retries up to 10 times at 80ms intervals, then gives up silently | render-interaction | derived invariant (BR15) |

Row count: 92, covering all 19 files `req-u5-intent-components` names.
`ArtifactEditControls.tsx` carries the largest share (16 rows) as its
richest state machine, consistent with `domain-entities-u5-intent-components`
calling it out as "the unit's most intricate business logic." The six
`call-return-throw` rows (two per `intentsService.*` call: `verifyArtifact`,
`artifactImpact`, `startQuorumEdit`) satisfy the minimum
`security-design-u5-intent-components` fixes for the mutating-action
authorization boundary. ac2's two named scenarios (`GateCard` question-gate,
`WaitingCard` primary-lookup) and ac3's two named scenarios (`StageGridEditor`
and `IntentStageList` empty input) are each present above.

#### Query/assertion, dependency-boundary, and advisory-framing conformance

Every scenario above is phrased as a Testing Library role/text/label query or
a captured-callback assertion plus an interaction or outcome, per
`req-stack-scenario-bound` and Rule 1 of `business-rules-u5-intent-components`
— no scenario proposes snapshot testing, shallow rendering, or an assertion
on `useIntent()`'s internal React Context object. Every scenario that mocks
shared run state mocks `IntentContextValue` directly, never a `SprintProvider`
or `SprintContextValue`, per the Known Discrepancy note above and the
dependency-boundary design in `security-design-u5-intent-components`. The
three files reading graph-derived data (`GateCard.tsx`, `DerivedItemRow.tsx`,
`WorkProductsSection.tsx`) mock `u2-hooks`' `useIntentGraph` return shape
directly via `vi.mock`, never a live graph provider. No scenario above
re-derives `DiscussButton`'s own behavior contract, per the Undeclared
Cross-Unit Edge note. Framing stays advisory throughout, per
`req-advisory-alignment`: every scenario above reads as coverage the plan is
building toward ("the scenario asserts…" / "renders…" / "calls…" phrasing),
never as behavior a CI gate already enforces. No deliverable above is a
literal `.test.ts`/`.test.tsx` file, per `req-plan-not-code` — writing or
running this unit's actual test code is a separate, later effort.

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

### Chapter: u7-observability-components

Scenario drafting for `comp-u7-observability-components`, satisfying
`requirements`'s `req-u7-observability-components`, depending only on
`comp-plan-governance` and `comp-u2-hooks` per `unit-of-work`'s
`unit-u7-observability-components` definition. The scenario-authoring rules
and entity shapes below are drafted per `business-rules-u7-observability-components`
and `domain-entities-u7-observability-components`, and the mock-boundary and
fixture-hygiene decisions follow `security-design-u7-observability-components`.
This unit owns no hooks or contexts and calls no `services/*` client
directly, so every row's `scenarioType` is one of the two types this unit
actually needs from the [bounded set](#scenario-types-bounded) —
`render-interaction` for the eleven presentational components and
`call-return-throw` for the two non-React files (`buildInsights.ts`,
`phaseConfig.ts`'s `phaseColorAt`) — per `contract-scenario-type-bound`.
`requirementId` is `req-u7-observability-components` for every row, per
`contract-traceability-ledger`.

#### Won't-Have verification for this unit

`components/observability/index.ts` was directly read (not folder-inferred):
it contains only eleven `export { ... }` re-export statements
(`ProjectDiagram`, `PhaseBlock`, `StepNode`, `AgentFocusCard`, `ActivityFeed`,
`TaskStatusRow`, `StuckAlert`, `BusinessView`, `AgentStatusCards`,
`IntentStatusCards`, `IterationDetailView`), with zero function or logic
definitions. It stays Won't-Have — no row is added for it below, per
`req-wont-have-verification` and `req-u7-observability-components`'s own
second acceptance criterion.

| File                                 | Status                                    | Evidence                                           |
| ------------------------------------- | ----------------------------------------- | --------------------------------------------------- |
| `components/observability/index.ts`  | Confirmed Won't-Have — no scenario needed | Direct read: pure re-export barrel, zero functions |

#### Dependency boundary — mock `useObservability`'s contract, do not re-derive it

Per `unit-of-work`'s implementation note ("Depends on `u2-hooks`
(`useObservability`)") and `contract-hook-state-shapes` (provided by
`u2-hooks`, consumed here), every scenario below that exercises a component
receiving `ProjectAgentInfo`, `SprintProgress`, `VelocityMetrics`,
`ActivityEvent`, `StuckDetection`, `LastToolMap`, or `PendingQuestionsMap`
constructs that shape as a literal mock prop value, never by instantiating
`useObservability` itself. The hook's own idle-detection transitions,
`markBlocked`/`clearStuck` dedup, and ring-buffer cap are `u2-hooks`'s
`state-transition` scenarios, already fixed in `business-rules-u2-hooks`'s
`useObservability.ts` rule set — re-deriving them here would duplicate
coverage across a unit boundary, the same anti-pattern
`business-rules-u6-graph-components`'s Known Discrepancy rule guards against.
`IntentStatusCards.tsx`'s `ProjectWithSprint` type (from the out-of-unit
`useProjectsCache` hook) follows the same rule: a literal mock value, never a
live hook call.

#### Scenario entries and traceability rows

| #   | `targetFile`            | `behaviorDescription`                                                                                                                                                           | `scenarioType`      | `acceptanceRef`                       |
| --- | ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------- | -------------------------------------- |
| 1   | `components/observability/ActivityFeed.tsx` | Consecutive `agent.tool` events sharing the same `semanticTool(detail)` label and the same `sprintId` collapse into one row with an incrementing `×N` count | render-interaction | AC1 (populated-data-state) |
| 2   | `components/observability/ActivityFeed.tsx` | A non-tool event, or a tool event with a different `semanticTool` label or a different `sprintId`, always starts a new row rather than collapsing | render-interaction | derived invariant |
| 3   | `components/observability/ActivityFeed.tsx` | Only the first 80 raw `events` are considered, and the collapsed row list is capped at 25 rows even when more distinct rows would otherwise result | render-interaction | derived invariant |
| 4   | `components/observability/ActivityFeed.tsx` | `multiProject` (more than one key in `projectNames`) gates whether a significant event's row shows the project-name suffix alongside the agent-type suffix | render-interaction | derived invariant |
| 5   | `components/observability/ActivityFeed.tsx` | `events = []` renders the `Card`/`CardContent` shell with zero rows in the `divide-y` container — no placeholder text, since none exists in source | render-interaction | AC1 (empty/loading-data-state) |
| 6   | `components/observability/AgentFocusCard.tsx` | All props populated (`agentStatus='running'`, `velocity` set, `pendingQuestions`, `branch`, `prUrl`) renders the focus sentence plus the velocity trend icon/percentage segment | render-interaction | AC1 (populated-data-state) |
| 7   | `components/observability/AgentFocusCard.tsx` | `agentStatus='running'` with `progress.taskCount > 0` and no `velocity` yet renders the italic "estimating velocity..." fallback instead of a trend segment | render-interaction | derived invariant |
| 8   | `components/observability/AgentFocusCard.tsx` | All props omitted renders an empty `div` with no badge or text node | render-interaction | AC1 (empty/loading-data-state) |
| 9   | `components/observability/AgentStatusCards.tsx` | `withSprints` filters out every project whose `effectiveSprintStatus(sprint) === 'idle'`, then sorts the remainder by the fixed order `running → waiting → failed → completed → passed` | render-interaction | AC1 (populated-data-state) |
| 10  | `components/observability/AgentStatusCards.tsx` | A card's `onClick` handler fires `onSelectSprint` only when that prop is supplied; a card with no `onSelectSprint` prop calls nothing when clicked | render-interaction | derived invariant |
| 11  | `components/observability/AgentStatusCards.tsx` | Zero non-idle projects renders the literal `"No recent agent activity."` message instead of the card grid | render-interaction | AC1 (empty/loading-data-state) |
| 12  | `components/observability/BusinessView.tsx` | Each of the four `Section`s (Highlights, Lowlights, Risks, Action Items) renders only when `buildInsights`'s corresponding array is non-empty | render-interaction | AC1 (populated-data-state) |
| 13  | `components/observability/BusinessView.tsx` | A `Risk` with `severity === 'high'` renders with a darker red item color than a `severity === 'medium'` risk | render-interaction | derived invariant |
| 14  | `components/observability/BusinessView.tsx` | An empty `projects` input (routed through `buildInsights`'s zero-project branch) renders only the single `"No spaces yet — create a space to get started"` highlight and no other `Section` — `BusinessView` itself never returns `null` | render-interaction | AC1 (empty/loading-data-state) |
| 15  | `components/observability/IntentStatusCards.tsx` | `visible` filters `items` to non-idle `effectiveIntentStatus` entries and sorts by the fixed order `running → waiting → failed → passed` | render-interaction | AC1 (populated-data-state) |
| 16  | `components/observability/IntentStatusCards.tsx` | Clicking a card calls `navigate` with `/space/{projectId}/intent/{intentId}/observability` | render-interaction | derived invariant |
| 17  | `components/observability/IntentStatusCards.tsx` | Zero non-idle items makes the component return `null` — a true unmount, not an empty-state message (the opposite pattern from `AgentStatusCards`) | render-interaction | AC1 (empty/loading-data-state) |
| 18  | `components/observability/IterationDetailView.tsx` | The `AgentStreamPanel` branch's `streamingText` prop prefers `agentStream.completedOutput` over `agentStream.streamingText` when both are present | render-interaction | AC1 (populated-data-state) |
| 19  | `components/observability/IterationDetailView.tsx` | `onNavigate`'s `workbenchPath` appends `/construction` or `/review` only when `sprint.phase` matches that segment, and falls back to the bare sprint/project path otherwise | render-interaction | derived invariant |
| 20  | `components/observability/IterationDetailView.tsx` | Neither `isAgentActive` nor a non-empty `streamingText`/`completedOutput` renders the `"No agent activity right now"` placeholder with a muted `Bot` icon | render-interaction | AC1 (empty/loading-data-state) |
| 21  | `components/observability/PhaseBlock.tsx` | The `{done}/{total}` progress fraction and its progress-bar width render only when `isCurrentPhase && (progress?.totalNodes ?? 0) > 0` | render-interaction | AC1 (populated-data-state) |
| 22  | `components/observability/PhaseBlock.tsx` | The running-tasks suffix (`"· N running"`) renders only when `config.key === 'CONSTRUCTION'` and at least one task has `executionStatus === 'RUNNING'` | render-interaction | derived invariant |
| 23  | `components/observability/PhaseBlock.tsx` | The `isFirstIncompleteMandatory` highlight lands on exactly one `StepNode` — the first mandatory step, in document order, not yet done, while the agent is `running` — not on every incomplete mandatory step | render-interaction | derived invariant |
| 24  | `components/observability/PhaseBlock.tsx` | `progress` null or `progress.totalNodes === 0` renders the em-dash `"—"` placeholder and no progress bar | render-interaction | AC1 (empty/loading-data-state) |
| 25  | `components/observability/ProjectDiagram.tsx` | `getStepsDone`/`getStepsSkipped` derive the `INCEPTION` phase's completed/skipped step sets from `progress`/`taskStatuses` | render-interaction | AC1 (populated-data-state) |
| 26  | `components/observability/ProjectDiagram.tsx` | `getStepsDone`/`getStepsSkipped` derive the `CONSTRUCTION` phase's completed/skipped step sets independently of the `INCEPTION` derivation | render-interaction | derived invariant |
| 27  | `components/observability/ProjectDiagram.tsx` | `getStepsDone`/`getStepsSkipped` derive the `REVIEW` phase's completed/skipped step sets independently of the other two phases | render-interaction | derived invariant |
| 28  | `components/observability/ProjectDiagram.tsx` | `isFuturePhase`/`isUnlocked` dims every phase after the current one, and only when `info.sprint` is set | render-interaction | derived invariant |
| 29  | `components/observability/ProjectDiagram.tsx` | `info.sprint === null` renders the `"No sprint started"` italic placeholder above the still-rendered, all-locked phase list | render-interaction | AC1 (empty/loading-data-state) |
| 30  | `components/observability/StepNode.tsx` | `isDone = true` (and not skipped) renders the green "✓ Done" variant | render-interaction | AC1 (populated-data-state) |
| 31  | `components/observability/StepNode.tsx` | `isSkipped = true` renders the dashed-border, strikethrough-label, "— Skipped" variant, visually and textually distinct from the done variant, and takes precedence even when `isDone` could also be computed true-ish | render-interaction | derived invariant |
| 32  | `components/observability/StepNode.tsx` | All state flags false (a future, unreached step) renders the grayscale/opacity-40 default variant with the step's `MANDATORY`/`CONDITIONAL` label and no done/skipped marker | render-interaction | AC1 (empty/loading-data-state) |
| 33  | `components/observability/StuckAlert.tsx` | A non-empty `detections` array renders one alert row per detection, sorted `critical → high → medium` by `severity` via a stable sort | render-interaction | AC1 (populated-data-state) |
| 34  | `components/observability/StuckAlert.tsx` | Each `reason` (`blocked_question`, `repeated_tool`, `idle`) maps to a fixed icon/color/label/border-width combination, and only `blocked_question` gets the pulsing/shadow treatment | render-interaction | derived invariant |
| 35  | `components/observability/StuckAlert.tsx` | `detections = []` makes the component return `null` | render-interaction | AC1 (empty/loading-data-state) |
| 36  | `components/observability/TaskStatusRow.tsx` | Each `executionStatus` value (`RUNNING`, `SUCCEEDED`, `FAILED`) maps to a fixed icon + color combination, one status chip per task | render-interaction | AC1 (populated-data-state) |
| 37  | `components/observability/TaskStatusRow.tsx` | A task with `executionStatus` undefined/missing falls through to the muted default combination, distinct from all three named states | render-interaction | derived invariant |
| 38  | `components/observability/TaskStatusRow.tsx` | `tasks = []` makes the component return `null` | render-interaction | AC1 (empty/loading-data-state) |
| 39  | `components/observability/buildInsights.ts` | `totalProjects === 0` returns a single `"No spaces yet — create a space to get started"` highlight via the calm-state early return, with empty `lowlights`/`risks`/`actions` | call-return-throw | AC1 (empty/loading-data-state) |
| 40  | `components/observability/buildInsights.ts` | Calm state (`activeCount === 0`) with `totalProjects > 0` returns a workspace-summary highlight plus, per project, a `lowlight` for a sprint paused at a phase or a high-severity `risk` for a sprint whose last run `failed` | call-return-throw | AC1 (populated-data-state) |
| 41  | `components/observability/buildInsights.ts` | Calm state with `completedSprints > 0` and `prCreated > 0` adds a completed-sprints highlight, a PRs-ready highlight, and one `actions` entry per project carrying a `sprint.prUrl` | call-return-throw | derived invariant |
| 42  | `components/observability/buildInsights.ts` | Active state, `phase === 'INCEPTION'` branches the highlight text on whether `progress.taskCount`/`progress.requirementCount` is populated | call-return-throw | derived invariant |
| 43  | `components/observability/buildInsights.ts` | Active state, `phase === 'CONSTRUCTION'` computes the `{done}/{total}` percentage and an `estimateEta` string only when `velocity.tasksPerHour > 0` and at least one task remains | call-return-throw | derived invariant |
| 44  | `components/observability/buildInsights.ts` | Active state, `phase === 'REVIEW'` adds a code-review highlight and, when `prUrl` is set, an "Approve PR" action | call-return-throw | derived invariant |
| 45  | `components/observability/buildInsights.ts` | Each `stuckDetections` entry maps by `reason`: `blocked_question`/`repeated_tool` each push a high-severity `risk` plus a matching `actions` entry, while `idle` pushes only a `lowlight` | call-return-throw | derived invariant |
| 46  | `components/observability/buildInsights.ts` | An active run that produces zero `highlights`/`lowlights`/`risks` falls back to the single `"Agents running — collecting data..."` highlight | call-return-throw | derived invariant |
| 47  | `components/observability/phaseConfig.ts` | `phaseColorAt(index)` for `index` within `[0, PHASE_CONFIGS.length)` returns the matching phase's five style fields (`headerBg`, `headerText`, `blockBg`, `blockBorder`, `mandatoryBg`, `conditionalBg`) directly | call-return-throw | AC1 (populated-data-state) |
| 48  | `components/observability/phaseConfig.ts` | `phaseColorAt(PHASE_CONFIGS.length)` wraps via modulo and returns the same five fields as `phaseColorAt(0)` | call-return-throw | AC1 (empty/loading-data-state analog — out-of-range index) |

All 48 rows resolve `req-plan-traceability`'s "every in-scope file appears in
exactly one row" constraint across this unit's 13 files (rows 1–5
`ActivityFeed.tsx`; 6–8 `AgentFocusCard.tsx`; 9–11 `AgentStatusCards.tsx`;
12–14 `BusinessView.tsx`; 15–17 `IntentStatusCards.tsx`; 18–20
`IterationDetailView.tsx`; 21–24 `PhaseBlock.tsx`; 25–29 `ProjectDiagram.tsx`;
30–32 `StepNode.tsx`; 33–35 `StuckAlert.tsx`; 36–38 `TaskStatusRow.tsx`; 39–46
`buildInsights.ts`; 47–48 `phaseConfig.ts`). Per `req-u7-observability-components`'s
acceptance criteria, every file carries at least one row tagged `AC1
(populated-data-state)` and one tagged `AC1 (empty/loading-data-state)` — for
the two pure-function files, which have no literal collection to empty, the
pairing maps to the zero-project early return (`buildInsights`, row 39) and
to the out-of-range wrap-around index (`phaseColorAt`, row 48), the nearest
structural equivalent to an empty/loading state for a pure lookup function.
Every fixture referenced above (`ProjectAgentInfo`, `ActivityEvent`,
`StuckDetection`, and similar shapes) is authored as a synthetic inline
literal per scenario, per `security-design-u7-observability-components`'s
Fixture and Data-Protection Design — synthetic ids only, no realistic-looking
secret or streamed-output text. Every component scenario (rows 1–38) asserts
via a Testing Library role/text/label query or an explicit DOM structural
fact (row/element presence or absence) — never `container.innerHTML`, a
snapshot, or a CSS-class/inline-style assertion — per `req-stack-scenario-bound`
and the same security design's no-snapshot-capture rule.

#### Advisory framing

Per `req-advisory-alignment`, every scenario description above reads as a
target this plan is building toward, never as an active CI gate — phrasing
throughout uses "renders...", "resolves...", "maps to..." rather than
asserting a gate already blocks on this behavior. No scenario in this unit
proposes snapshot testing, shallow rendering, or an assertion on CSS class
names/inline style strings, consistent with
`business-rules-u7-observability-components`'s BR-1 and
`req-stack-scenario-bound`.

### Chapter: u8-admin-components (pending)

### Chapter: u8-admin-components (pending)

### Chapter: u9-discussion-components

Scenario coverage for `req-u9-discussion-components`, owned by
`comp-u9-discussion-components`. Per `unit-of-work`'s `unit-u9-discussion-components`
definition, `business-rules-u9-discussion-components`,
`domain-entities-u9-discussion-components`, and `security-design-u9-discussion-components`,
this chapter covers the 5 untested files in `frontend/src/components/discussion/` —
`DiscussButton.tsx`, `DiscussionsTab.tsx`, `MentionToasts.tsx`,
`MessageBubble.tsx`, `ResolveDialog.tsx`. `DiscussionInput.tsx`,
`DiscussionPanel.tsx`, `DiscussionProvider.tsx`, and `DiscussionThread.tsx`
already each have a matching `*.test.tsx` file and are out of this unit's
scope, confirmed by direct inspection of the directory. This unit owns no
pure functions, hooks, or contexts of its own, so every row below uses the
`render-interaction` scenario type `contract-scenario-type-bound` fixes, per
BR-1 of `business-rules-u9-discussion-components`. `requirementId` is
`req-u9-discussion-components` for every row, per
`contract-traceability-ledger`.

#### Won't-Have verification for this unit

`components/discussion/index.ts` was directly read, not folder-inferred: it
contains only `export { ... }` / `export type { ... }` re-export statements
for the module's ten public exports, with zero function or logic
definitions. It stays Won't-Have — no row is added for it below, per
`req-wont-have-verification` and BR-8 of `business-rules-u9-discussion-components`.

| File                             | Status                                    | Evidence                                           |
| -------------------------------- | ----------------------------------------- | -------------------------------------------------- |
| `components/discussion/index.ts` | Confirmed Won't-Have — no scenario needed | Direct read: pure re-export barrel, zero functions |

#### Known discrepancy — stated `u2-hooks` dependency is not concretely exercised

`unit-of-work` and `requirements` both state this unit depends on `u2-hooks`
(`useCollaborativeArtifact*`, `usePresence`) via `contract-hook-state-shapes`,
and that contract lists `u9-discussion-components` as a consumer. Direct
inspection of this unit's five untested files, and of the already-tested
`DiscussionProvider.tsx` they read through `useDiscussions()`, found none of
them importing `useCollaborativeArtifact`, `useCollaborativeArtifactContent`,
`useCollaborativeStructuredAnswer`, or `usePresence` from `hooks/` —
`DiscussionProvider.tsx` wires its realtime behavior through `realtimeService`
(a plain WebSocket client) and `useAuth()`, not the Yjs-awareness-backed
`usePresence` hook `contract-hook-state-shapes` describes. Following this
repository's guidance to investigate such a discrepancy rather than assume
either side is correct (per BR-9 of `business-rules-u9-discussion-components`
and the precedent `business-rules-u2-hooks` set for `useSprintGraph.ts`), every
scenario below mocks `useDiscussions()` directly with plain in-memory fixture
data; no row mocks or renders against a `contract-hook-state-shapes` shape,
and no row below cites it. The formal `consumes` relationship to
`contract-hook-state-shapes` is preserved in this unit's traceability as a
real declared input, but is recorded here as not concretely realized in any
scenario. If a future change wires `usePresence` or a collaborative hook into
`DiscussionProvider.tsx` or any of this unit's five files, this note and this
unit's scenario set should be re-opened rather than trusted at face value.

#### Acceptance-criteria mapping per file

`req-u9-discussion-components`'s acceptance criteria require, per untested
file, "at least one scenario for posting/receiving a discussion message and
one for the resolved/unresolved thread state." Per BR-2 of
`business-rules-u9-discussion-components`, three files map directly and two
substitute the closest analogous state boundary the file actually renders
(recorded explicitly below rather than invented):

| File                 | "Post/receive message" scenario                                                                            | "Resolved/unresolved" scenario                                                                                      |
| -------------------- | ---------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `DiscussButton.tsx`  | Receiving: badge/label transitions as `unreadCount` moves 0 → N (direct fit)                               | Substituted: read-only v1 gate — sprint scope with no thread renders `null`, sprint scope with a thread opens it    |
| `DiscussionsTab.tsx` | Receiving: thread list/search results reflect posted/updated discussions (direct fit)                      | Direct fit: `status === 'resolved'` renders the check icon + summary; the `resolved` filter narrows the list        |
| `MentionToasts.tsx`  | Receiving: a mention toast is posted into the stack and renders the author/excerpt (direct fit)            | Substituted: the toast's own open → dismissed transition                                                            |
| `MessageBubble.tsx`  | Receiving: a `MessageGroup` renders its messages, including agent running/failed-retry states (direct fit) | Substituted: the per-message redacted vs. non-redacted render                                                       |
| `ResolveDialog.tsx`  | Substituted: the optional outcome-message picker over existing messages                                    | Direct fit: this file _is_ the unresolved → resolved transition — submitting resolves, cancelling leaves unresolved |

#### Scenario entries and traceability rows

Per `contract-traceability-ledger`, every row below is one `ScenarioEntry`
linked into this plan's single ledger, with `requirementId:
req-u9-discussion-components` (omitted from each row for brevity — no row in
this chapter uses a different requirement id). Fixture values follow
`security-design-u9-discussion-components`'s Fixture and Artifact Hygiene
Design: sequential synthetic identifiers (`msg-fixture-1`,
`thread-fixture-1`, `author: "Fixture User"`), never a real username, token,
or value copied from a live environment.

| targetFile           | behaviorDescription                                                                                                                                                                                                                         | scenarioType       | acceptanceRef                                                                                |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------ | -------------------------------------------------------------------------------------------- |
| `DiscussButton.tsx`  | Renders `null` when `useDiscussions()` returns falsy (no provider mounted)                                                                                                                                                                  | render-interaction | derived invariant (BR-3)                                                                     |
| `DiscussButton.tsx`  | Renders `null` when the resolved scope is `kind: 'sprint'` and `discussionFor(entityType, entityId)` returns no thread (v1 read-only gate)                                                                                                  | render-interaction | resolved/unresolved (substituted boundary)                                                   |
| `DiscussButton.tsx`  | Renders plain `"Discuss"` with no badge when `unread === 0` and `messageCount === 0`                                                                                                                                                        | render-interaction | derived invariant (BR-3)                                                                     |
| `DiscussButton.tsx`  | Renders `"Discuss (N unread)"` with a numeric pill, tinted icon, when `unread > 0`, capped at `"99+"` beyond 99                                                                                                                             | render-interaction | post/receive message                                                                         |
| `DiscussButton.tsx`  | Renders `"Discuss (ongoing)"` with a dot indicator, tinted icon, when `unread === 0` and `messageCount > 0`                                                                                                                                 | render-interaction | post/receive message                                                                         |
| `DiscussButton.tsx`  | Clicking with an existing thread calls `discussions.openDiscussionById(thread.id)` and stops event propagation                                                                                                                              | render-interaction | derived invariant (BR-3)                                                                     |
| `DiscussButton.tsx`  | Clicking with no thread (non-sprint scope) calls `discussions.openDiscussion({ entityType, entityId, entityTitle })` and stops event propagation                                                                                            | render-interaction | derived invariant (BR-3)                                                                     |
| `DiscussionsTab.tsx` | Renders `null` when `useDiscussions()` is falsy                                                                                                                                                                                             | render-interaction | derived invariant (BR-4)                                                                     |
| `DiscussionsTab.tsx` | A trimmed query of length ≥3 sets `searching = true`, then after the 350ms debounce calls `discussionsService.search(scope, { q, status })` and swaps the list view to search results                                                       | render-interaction | post/receive message                                                                         |
| `DiscussionsTab.tsx` | A query trimmed to <3 characters (including on backspace from a longer query) immediately clears `searchResults` to `null` and `searching` to `false`, reverting to the local filtered list                                                 | render-interaction | derived invariant (BR-4)                                                                     |
| `DiscussionsTab.tsx` | A failed search call resolves `searchResults` to an empty array (via `.catch(() => [])`) and renders the "No matches." copy, not a crash                                                                                                    | render-interaction | derived invariant (BR-4)                                                                     |
| `DiscussionsTab.tsx` | The three status filter buttons (`all`/`open`/`resolved`) are mutually exclusive and `filtered = discussions.filter(d => statusFilter === 'all' \|\| d.status === statusFilter)` narrows the local list accordingly                         | render-interaction | resolved/unresolved (direct fit)                                                             |
| `DiscussionsTab.tsx` | A thread row with `status === 'resolved'` renders the check icon with tooltip (`resolutionSummary` or `"Resolved by {resolvedByName \|\| 'a member'}"`); a non-resolved row renders neither                                                 | render-interaction | resolved/unresolved (direct fit)                                                             |
| `DiscussionsTab.tsx` | Empty-local-list copy is scope-aware: `scope?.kind === 'sprint'` renders "v1 discussions are read-only."; any other scope renders the open-from-anywhere hint                                                                               | render-interaction | derived invariant (BR-4)                                                                     |
| `DiscussionsTab.tsx` | Clicking a thread row or a search result calls `ctx.openDiscussionById` with that row's/result's discussion id                                                                                                                              | render-interaction | post/receive message                                                                         |
| `MentionToasts.tsx`  | Renders `null` when `toasts.length === 0` — no empty-state placeholder rendered                                                                                                                                                             | render-interaction | derived invariant (BR-5; security-design SEC-5)                                              |
| `MentionToasts.tsx`  | A toast fixture with `excerpt` explicitly `undefined` renders `{byName} mentioned you` with the excerpt region absent from the DOM, never the literal string `"undefined"`                                                                  | render-interaction | security-design SEC-5                                                                        |
| `MentionToasts.tsx`  | A toast fixture with a truthy `excerpt` renders both `{byName} mentioned you` and the excerpt text                                                                                                                                          | render-interaction | post/receive message                                                                         |
| `MentionToasts.tsx`  | "Jump to thread" calls `onOpen(t.discussionId)` then `onDismiss(t.id)`, in that order — not `onOpen` alone                                                                                                                                  | render-interaction | resolved/unresolved (substituted boundary: dismissed transition)                             |
| `MentionToasts.tsx`  | The dismiss (X) control calls only `onDismiss(t.id)`, never `onOpen`                                                                                                                                                                        | render-interaction | resolved/unresolved (substituted boundary: dismissed transition)                             |
| `MessageBubble.tsx`  | A non-agent author renders a hash-colored circle (`generateColor(authorId \|\| authorName)`) with the first character of `authorName` (or `'?'` when empty) uppercased; `authorType === 'agent'` renders the fixed bot icon instead         | render-interaction | post/receive message                                                                         |
| `MessageBubble.tsx`  | The "requested by … · command" caption renders only for an agent message whose first message in the group has `requestedByName` or `command` set; a user message never renders this caption                                                 | render-interaction | post/receive message                                                                         |
| `MessageBubble.tsx`  | A redacted message (`m.redacted`) renders muted italic content plus, only when `redactedAt` is set, a relative-time suffix, and never renders through `ReactMarkdown`; the original `content` string is absent from the rendered DOM        | render-interaction | resolved/unresolved (substituted boundary: redacted vs. non-redacted); security-design SEC-3 |
| `MessageBubble.tsx`  | A non-redacted message containing a Markdown link and an emphasis marker renders through `ReactMarkdown`/`remark-gfm` as a resolved `<a>` with accessible name and an emphasized text node, asserted via role/text query, never `innerHTML` | render-interaction | post/receive message; security-design SEC-4                                                  |
| `MessageBubble.tsx`  | `assistStatus === 'running'` on a non-redacted agent message renders a spinner caption                                                                                                                                                      | render-interaction | post/receive message                                                                         |
| `MessageBubble.tsx`  | `assistStatus === 'failed'` with both `requestId` and `onAssistRetry` supplied renders a "failed - retry" control, and clicking it calls `onAssistRetry(m.id)`; omitting either suppresses the control                                      | render-interaction | post/receive message                                                                         |
| `MessageBubble.tsx`  | The per-message redact control renders only when both `canRedact` is truthy and `onRedact` is supplied, and clicking its menu item calls `onRedact(m.id)`; either condition alone suppresses the control                                    | render-interaction | derived invariant (BR-6)                                                                     |
| `ResolveDialog.tsx`  | Outcome candidates render as `messages.filter(m => !m.redacted).slice(-8)` — redacted messages excluded, only the most recent 8 non-redacted messages offered                                                                               | render-interaction | post/receive message (substituted boundary)                                                  |
| `ResolveDialog.tsx`  | Clicking a candidate row toggles `outcomeId`: selecting an already-selected row clears the selection back to `null`; selecting a different row replaces it                                                                                  | render-interaction | post/receive message (substituted boundary)                                                  |
| `ResolveDialog.tsx`  | "Resolve" calls `onResolve({ resolutionSummary: summary.trim() \|\| undefined, outcomeMessageId: outcomeId \|\| undefined })` — a blank/whitespace-only summary is sent as `undefined`, not an empty string                                 | render-interaction | resolved/unresolved (direct fit)                                                             |
| `ResolveDialog.tsx`  | On a successful `onResolve`, the component clears `summary`/`outcomeId` and calls `onOpenChange(false)`                                                                                                                                     | render-interaction | resolved/unresolved (direct fit)                                                             |
| `ResolveDialog.tsx`  | On an `onResolve` rejection with an `Error` instance, the component sets the visible `error` string to `err.message`, keeps the dialog open, resets `busy` to `false`, and does not clear `summary`/`outcomeId`                             | render-interaction | resolved/unresolved (direct fit); security-design SEC-8                                      |
| `ResolveDialog.tsx`  | On an `onResolve` rejection with a non-`Error` thrown value, the component renders the fixed fallback string `"Failed to resolve"`, never a coercion of the thrown value                                                                    | render-interaction | resolved/unresolved (direct fit); security-design SEC-8                                      |
| `ResolveDialog.tsx`  | Both "Cancel" and "Resolve" are `disabled` while `busy` is `true`; "Cancel" calls `onOpenChange(false)` directly without invoking `onResolve`                                                                                               | render-interaction | resolved/unresolved (direct fit)                                                             |

Row count: 33, covering all 5 files `req-u9-discussion-components` names,
resolving `req-plan-traceability`'s "every in-scope file appears in exactly
one row" constraint for this unit (7 rows for `DiscussButton.tsx`, 8 for
`DiscussionsTab.tsx`, 4 for `MentionToasts.tsx`, 7 for `MessageBubble.tsx`, 7
for `ResolveDialog.tsx`). Every row satisfies BR-2's per-file "post/receive
message" and "resolved/unresolved" pairing, including the three substituted
boundaries recorded in the mapping table above rather than an invented direct
fit. The SEC-3 (redaction-integrity), SEC-4 (Markdown rendering-boundary),
SEC-5 (mention-toast exposure), and SEC-8 (error-path containment) scenario
obligations from `security-design-u9-discussion-components` are each
satisfied by name above.

#### Query/assertion and tech-stack conformance

Every scenario above is phrased as a Testing Library role/text/label query
plus a callback-invocation or rendered-output assertion, per
`req-stack-scenario-bound` and BR-1 of
`business-rules-u9-discussion-components` — no scenario proposes snapshot
testing, shallow rendering, or an assertion on CSS class names or other
implementation internals. No authorization-denial scenario is drafted for
this unit's five files, per `security-design-u9-discussion-components`'s
SEC-2 scope closure: none renders an admin-only or privileged control, and
that boundary belongs to `DiscussionProvider.tsx` (out of this unit's scope)
and to `req-u10-settings-auth-misc`'s own `ProtectedRoute.tsx` scenarios.
Framing stays advisory throughout, per `req-advisory-alignment`: every
scenario above reads as coverage the plan is building toward ("the scenario
verifies..." / "renders..." / "calls..." phrasing), never as behavior a CI
gate already enforces.

### Chapter: u10-settings-auth-misc (pending)

### Chapter: u11-ui-primitives

Scenario coverage for `req-u11-ui-primitives`: the 26 untested `components/ui/`
Radix/shadcn primitives, per `unit-of-work`'s `u11-ui-primitives` definition,
`business-rules-u11-ui-primitives`, `domain-entities-u11-ui-primitives`, and
`security-design-u11-ui-primitives`. This unit `depends_on` only
`comp-plan-governance` — no `services`/hooks/context dependency exists to mock
— so, per `contract-scenario-type-bound` and Rule R3, every scenario below
uses the `render-interaction` branch only.

Per `domain-entities-u11-ui-primitives`' Primitive Classification (verified
directly against `frontend/src/components/ui/`, confirming the file count
matches `req-u11-ui-primitives` and satisfying Rule R6), the 26 files split
into three classes:

- **Content-only** (12, Rule R1 render scenario only): `avatar.tsx`,
  `badge.tsx`, `button.tsx`, `card.tsx`, `input.tsx`, `label.tsx`,
  `progress.tsx`, `resizable.tsx`, `scroll-area.tsx`, `separator.tsx`,
  `skeleton.tsx`, `textarea.tsx`.
- **Interactive-toggle** (9, Rule R1 render scenario plus Rule R2's mandatory
  toggle scenario): `accordion.tsx`, `collapsible.tsx`, `dialog.tsx`,
  `dropdown-menu.tsx`, `select.tsx`, `switch.tsx`, `tabs.tsx`,
  `toggle-group.tsx`, `tooltip.tsx`.
- **Composite-stateful** (5, Rule R1 render scenario plus Rule R9's advisory,
  non-blocking interaction scenario): `alert-dialog.tsx`, `command.tsx`,
  `popover.tsx`, `sheet.tsx`, `tree-view.tsx`.

No file in this chapter is Won't-Have; all 26 are covered below.

#### Scenario entries and traceability rows

Per `contract-traceability-ledger`, every row below is one `ScenarioEntry`
drafted via `component-methods`' `draftScenario` and linked via
`linkTraceability` into this plan's single ledger.

| targetFile                        | behaviorDescription                                                                                                                                                                                                                                                                                                                       | scenarioType       | acceptanceRef                              | requirementId         |
| --------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------ | ------------------------------------------ | --------------------- |
| `components/ui/accordion.tsx`     | `AccordionContent` renders its children via `getByText` for an item whose `value` is in the expanded `defaultValue`/`value` set.                                                                                                                                                                                                          | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/accordion.tsx`     | Clicking an `AccordionTrigger` via `user-event` expands its `AccordionContent` (content becomes queryable via `getByText`); for `type="single" collapsible`, clicking the same trigger again collapses it.                                                                                                                                | render-interaction | req-u11-ui-primitives AC-2                 | req-u11-ui-primitives |
| `components/ui/alert-dialog.tsx`  | `AlertDialogContent` renders its `AlertDialogTitle`/`AlertDialogDescription` children via `getByText` when rendered with `open=true`.                                                                                                                                                                                                     | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/alert-dialog.tsx`  | Clicking `AlertDialogTrigger` via `user-event` opens the dialog (`getByRole('alertdialog')`); clicking `AlertDialogAction` or `AlertDialogCancel` closes it.                                                                                                                                                                              | render-interaction | business-rules R9 (advisory, non-blocking) | req-u11-ui-primitives |
| `components/ui/avatar.tsx`        | `Avatar` renders `AvatarFallback` text content via `getByText` when no `AvatarImage` loads successfully.                                                                                                                                                                                                                                  | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/badge.tsx`         | `Badge` renders its children text via `getByText` with the default variant class applied.                                                                                                                                                                                                                                                 | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/button.tsx`        | `Button` renders its children label accessible via `getByRole('button', { name: ... })`.                                                                                                                                                                                                                                                  | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/card.tsx`          | `Card` composed with `CardHeader`, `CardTitle`, and `CardContent` renders every supplied child's text via `getByText`.                                                                                                                                                                                                                    | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/collapsible.tsx`   | `CollapsibleContent` renders its children via `getByText` when `Collapsible` is rendered with `open=true`.                                                                                                                                                                                                                                | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/collapsible.tsx`   | Clicking `CollapsibleTrigger` via `user-event` toggles `open` from `false` to `true` (content becomes queryable) and back to `false` (content becomes absent).                                                                                                                                                                            | render-interaction | req-u11-ui-primitives AC-2                 | req-u11-ui-primitives |
| `components/ui/command.tsx`       | `CommandList` renders its `CommandItem` labels via `getByText` for the supplied items.                                                                                                                                                                                                                                                    | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/command.tsx`       | Typing a filter string into `CommandInput` via `user-event` narrows the visible `CommandItem` entries, asserted by a non-matching item becoming unqueryable via `queryByText`.                                                                                                                                                            | render-interaction | business-rules R9 (advisory, non-blocking) | req-u11-ui-primitives |
| `components/ui/dialog.tsx`        | `DialogContent` renders its `DialogTitle`/`DialogDescription` children via `getByText` when `Dialog` is rendered with `open=true`.                                                                                                                                                                                                        | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/dialog.tsx`        | Clicking `DialogTrigger` via `user-event` opens the dialog (`getByRole('dialog')`); clicking `DialogClose` or pressing Escape closes it, asserted by the dialog becoming absent via `queryByRole`.                                                                                                                                        | render-interaction | req-u11-ui-primitives AC-2                 | req-u11-ui-primitives |
| `components/ui/dropdown-menu.tsx` | `DropdownMenuContent` renders its `DropdownMenuItem` labels via `getByText` once the menu is open.                                                                                                                                                                                                                                        | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/dropdown-menu.tsx` | Clicking `DropdownMenuTrigger` via `user-event` opens the menu (items queryable via `getByRole('menuitem')`); selecting an item closes the menu.                                                                                                                                                                                          | render-interaction | req-u11-ui-primitives AC-2                 | req-u11-ui-primitives |
| `components/ui/input.tsx`         | `Input` renders with the supplied value/placeholder accessible via `getByRole('textbox')` (or `getByPlaceholderText`).                                                                                                                                                                                                                    | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/label.tsx`         | `Label` renders its text content via `getByText` and associates with a form control via its `htmlFor` attribute.                                                                                                                                                                                                                          | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/popover.tsx`       | `PopoverContent` renders its supplied children via `getByText` when `Popover` is rendered with `open=true`.                                                                                                                                                                                                                               | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/popover.tsx`       | Clicking `PopoverTrigger` via `user-event` opens the popover content (queryable via `getByText`); clicking outside or pressing Escape closes it.                                                                                                                                                                                          | render-interaction | business-rules R9 (advisory, non-blocking) | req-u11-ui-primitives |
| `components/ui/progress.tsx`      | `Progress` renders with `role="progressbar"` and exposes the supplied `value` via `aria-valuenow`, asserted via `getByRole('progressbar')`.                                                                                                                                                                                               | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/resizable.tsx`     | `ResizablePanelGroup` composed with `ResizablePanel` and `ResizableHandle` renders the supplied panel children via `getByText`.                                                                                                                                                                                                           | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/scroll-area.tsx`   | `ScrollArea` renders its scrollable children content via `getByText`.                                                                                                                                                                                                                                                                     | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/select.tsx`        | `SelectContent` renders its `SelectItem` labels via `getByText`/`getByRole('option')` once the select is open.                                                                                                                                                                                                                            | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/select.tsx`        | Clicking `SelectTrigger` via `user-event` opens the listbox (`getByRole('listbox')`); selecting a `SelectItem` updates the displayed `SelectValue` text and closes the listbox.                                                                                                                                                           | render-interaction | req-u11-ui-primitives AC-2                 | req-u11-ui-primitives |
| `components/ui/separator.tsx`     | `Separator` renders via `getByRole`, exposing `role="separator"` by default or `role="none"` when `decorative` is true, per its orientation/decorative props.                                                                                                                                                                             | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/sheet.tsx`         | `SheetContent` renders its supplied children via `getByText` when `Sheet` is rendered with `open=true`.                                                                                                                                                                                                                                   | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/sheet.tsx`         | Clicking `SheetTrigger` via `user-event` opens the sheet panel (content queryable via `getByText`); clicking `SheetClose` or pressing Escape closes it.                                                                                                                                                                                   | render-interaction | business-rules R9 (advisory, non-blocking) | req-u11-ui-primitives |
| `components/ui/skeleton.tsx`      | `Skeleton` renders its placeholder container without throwing when no children are passed, queried via a supplied `data-testid`.                                                                                                                                                                                                          | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/switch.tsx`        | `Switch` renders with `role="switch"` and reflects its initial checked/unchecked state via `aria-checked`.                                                                                                                                                                                                                                | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/switch.tsx`        | Clicking the `Switch` via `user-event` toggles `aria-checked` from `false` to `true` (or vice versa) and fires `onCheckedChange` with the new value.                                                                                                                                                                                      | render-interaction | req-u11-ui-primitives AC-2                 | req-u11-ui-primitives |
| `components/ui/tabs.tsx`          | `TabsContent` renders the active tab's children via `getByText` for the tab matching the default/current `value`.                                                                                                                                                                                                                         | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/tabs.tsx`          | Clicking a `TabsTrigger` via `user-event` switches the active `TabsContent` — the newly selected tab's content becomes visible and the previously active tab's content becomes absent/hidden.                                                                                                                                             | render-interaction | req-u11-ui-primitives AC-2                 | req-u11-ui-primitives |
| `components/ui/textarea.tsx`      | `Textarea` renders with the supplied value/placeholder accessible via `getByRole('textbox')`.                                                                                                                                                                                                                                             | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/toggle-group.tsx`  | `ToggleGroup` renders its `ToggleGroupItem` children accessible via `getByRole('button')` for each item.                                                                                                                                                                                                                                  | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/toggle-group.tsx`  | Clicking a `ToggleGroupItem` via `user-event` marks it selected (its `data-state` attribute becomes `"on"`); for a `type="single"` group, the previously selected item becomes deselected.                                                                                                                                                | render-interaction | req-u11-ui-primitives AC-2                 | req-u11-ui-primitives |
| `components/ui/tooltip.tsx`       | `TooltipContent` renders its supplied text via `getByText` once the `Tooltip` is opened.                                                                                                                                                                                                                                                  | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/tooltip.tsx`       | Hovering or focusing `TooltipTrigger` via `user-event` opens the tooltip (content becomes queryable via `getByText`); moving focus/hover away closes it.                                                                                                                                                                                  | render-interaction | req-u11-ui-primitives AC-2                 | req-u11-ui-primitives |
| `components/ui/tree-view.tsx`     | `TreeView` renders its node labels via `getByText` for the supplied `TreeDataItem` data, with a parent node's children hidden until the parent is expanded.                                                                                                                                                                               | render-interaction | req-u11-ui-primitives AC-1                 | req-u11-ui-primitives |
| `components/ui/tree-view.tsx`     | Clicking a parent node's expand control via `user-event` toggles its children's visibility (backed by the `AccordionPrimitive` expand/collapse state `tree-view.tsx` composes internally), and clicking a leaf node marks it selected, asserted via the item's selection-driven render output rather than a binary `aria-checked` toggle. | render-interaction | business-rules R9 (advisory, non-blocking) | req-u11-ui-primitives |

Row count: 40 (12 content-only × 1, 9 interactive-toggle × 2, 5
composite-stateful × 2), covering all 26 files named in
`req-u11-ui-primitives` with no omission or duplicate `targetFile`-under-a-
different-`requirementId` mapping (Rule R5). Every row uses
`requirementId: "req-u11-ui-primitives"` per `domain-entities-u11-ui-primitives`'
Traceability Row entity — this unit maps to exactly one requirement with no
split across units.

#### Unsafe-render (XSS-class) verification

Per `security-design-u11-ui-primitives`' Unsafe-Render Verification Design,
each of the 26 files was inspected directly for a caller-controlled content
sink (`dangerouslySetInnerHTML` or equivalent). A direct source read of every
file in `frontend/src/components/ui/` found no such sink in any of the 26
files — all are either thin `React.forwardRef` wrappers around a Radix root
(re-exporting sub-parts with `cn()`-merged class names) or, for `tree-view.tsx`
and `command.tsx`, bespoke components that render only caller-supplied React
children/props through normal JSX interpolation, never through an unescaped
render path. The outcome for every file above is therefore (a) **no
additional security scenario needed, confirmed by source read** — no file in
this chapter needed an added unsafe-render scenario under outcome (b).

No authentication, authorization, session, or browser-storage scenario is
added for any file in this chapter, per `security-design-u11-ui-primitives`'
Authentication and Authorization Design: none of these 26 presentational
primitives gate rendering on identity, and none is confirmed by source read to
read or write clipboard/local-storage/session content.

#### Query/assertion and tech-stack conformance

Every scenario above is phrased as a Testing Library role/text/label query
plus (where applicable) a `user-event` interaction or a rendered-output
assertion, per `tech-stack-decisions-u11-ui-primitives`' Query and Assertion
Patterns — no scenario proposes snapshot testing, shallow rendering, or an
assertion on Radix internal state, CSS class names, or DOM structure beyond
accessible role/label/text (Rule R4). No scenario requires tooling beyond the
already-confirmed Vitest 4 / Testing Library / `user-event` / jsdom stack.

### Chapter: u12-icons-layout-theme

Owned by `comp-u12-icons-layout-theme`, which delivers `req-u12-icons-layout-theme`
alone. Per `business-rules-u12-icons-layout-theme` and `domain-entities-u12-icons-layout-theme`,
this chapter covers the 8 untested files spanning `components/icons/git-providers.tsx`,
the five `components/layout/` files, and the two `components/theme/` files. Direct
inspection of all eight files (per `req-wont-have-verification`'s evidence standard)
confirms none is a re-export barrel, type-only module, or static constant — every
file is a `covered` entry below; this chapter adds no `WontHaveEntry`. This unit's
only upstream dependency is `comp-plan-governance` (the contracts above); no other
content unit's chapter gates this one, per `services`' choreography framing.

Per Rule U12-1 (`req-stack-scenario-bound`), seven of the eight files are
`render-interaction` — none is a `services/*` client, so `call-return-throw` never
applies in this unit. `ThemeProvider.tsx` alone is `state-transition`, since its
observable behavior is a context-value change, not markup of its own.

Per `security-design-u12-icons-layout-theme`'s mock-at-the-boundary design, the four
files that call collaborators owned by other units (`ActivityPanel`, `AppHeader`,
`CommandPalette`, `SprintPipelineBar`) mock those collaborators (`realtimeService`,
`agentsService`, `timelineEventsService`, `useAuth`, `useProjectCache` /
`useProjectSprintsCache` / `useProjectsCache`, `useIntent`, and the four
`PhaseMetrics` services) at their existing call/hook signatures per
`contract-service-call-signatures` / `contract-hook-state-shapes` — no scenario here
re-exercises a collaborator's own internals, and no denied-access scenario is
fabricated for this unit's components (that obligation belongs to `req-u10-settings-auth-misc`,
which owns `useAuth` and `ProtectedRoute.tsx`). Direct inspection of all five
`components/layout/` and `components/theme/` files confirms none uses
`dangerouslySetInnerHTML` or an equivalent unescaped-render path, so no XSS-class
scenario is added for them; `GitProviderIcon`'s `provider` lookup is a closed,
static table that never renders a caller-supplied string as markup, so its two
scenarios below are correctness scenarios, not security scenarios, per
`security-design-u12-icons-layout-theme`'s Input Validation and Injection Surface
Design.

#### Scenarios

| `targetFile`                              | `behaviorDescription`                                                                                                                                                                                                                            | `scenarioType`       | `acceptanceRef`                | `requirementId`              |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------- | ------------------------------ | ---------------------------- |
| `components/icons/git-providers.tsx`      | `GitProviderIcon` — known provider (`'github'`) renders the mapped icon component                                                                                                                                                                | `render-interaction` | derived invariant — Rule U12-4 | `req-u12-icons-layout-theme` |
| `components/icons/git-providers.tsx`      | `GitProviderIcon` — unknown/null `provider` renders nothing (the `null`-return fallback branch)                                                                                                                                                  | `render-interaction` | derived invariant — Rule U12-4 | `req-u12-icons-layout-theme` |
| `components/theme/ThemeProvider.tsx`      | Default-theme resolution — no stored `aidlc-theme` preference; `resolvedTheme` (observed via `useTheme()` in a consuming component) follows the mocked `window.matchMedia('(prefers-color-scheme: dark)')` result                                | `state-transition`   | acceptance criterion 1         | `req-u12-icons-layout-theme` |
| `components/theme/ThemeProvider.tsx`      | User-toggled theme — calling `setTheme('dark')` updates `resolvedTheme` exposed via `useTheme()`, persists `'dark'` to `window.localStorage` under the fixed key `aidlc-theme`, and swaps the `light`/`dark` class on `document.documentElement` | `state-transition`   | acceptance criterion 1         | `req-u12-icons-layout-theme` |
| `components/theme/ThemeToggle.tsx`        | Menu-driven theme selection — opening the dropdown and selecting "Dark" invokes `setTheme('dark')`                                                                                                                                               | `render-interaction` | derived invariant — Rule U12-1 | `req-u12-icons-layout-theme` |
| `components/theme/ThemeToggle.tsx`        | Active-theme menu item is visually highlighted when the current `theme` from `useTheme()` matches that item                                                                                                                                      | `render-interaction` | derived invariant — Rule U12-1 | `req-u12-icons-layout-theme` |
| `components/layout/AppHeader.tsx`         | Structural slots (logo, breadcrumb nav, search trigger, theme toggle, user menu) render given representative route/props/context                                                                                                                 | `render-interaction` | acceptance criterion 2         | `req-u12-icons-layout-theme` |
| `components/layout/AppHeader.tsx`         | Platform-admin-gated "Admin Panel" menu item renders when the mocked `useAuth()` reports `isPlatformAdmin: true` and is absent when `false`                                                                                                      | `render-interaction` | derived invariant — Rule U12-3 | `req-u12-icons-layout-theme` |
| `components/layout/ActivityPanel.tsx`     | Default/idle structural state — no agent running, no stream text, no tool calls — renders the idle placeholder                                                                                                                                   | `render-interaction` | acceptance criterion 2         | `req-u12-icons-layout-theme` |
| `components/layout/ActivityPanel.tsx`     | Tab-switch interaction — selecting the Timeline tab swaps the rendered content from the Agent tab to the Timeline tab                                                                                                                            | `render-interaction` | derived invariant — Rule U12-3 | `req-u12-icons-layout-theme` |
| `components/layout/CommandPalette.tsx`    | Keyboard-shortcut interaction — dispatching a `Cmd/Ctrl+K` `keydown` toggles the dialog's open state via `onOpenChange`                                                                                                                          | `render-interaction` | derived invariant — Rule U12-3 | `req-u12-icons-layout-theme` |
| `components/layout/CommandPalette.tsx`    | Item-selection interaction — selecting a navigation command item closes the palette (`onOpenChange(false)`) and navigates                                                                                                                        | `render-interaction` | derived invariant — Rule U12-3 | `req-u12-icons-layout-theme` |
| `components/layout/SprintLayout.tsx`      | Structural-slot scenario — the routed `Outlet` content renders inside the `SprintProvider` wrapper                                                                                                                                               | `render-interaction` | acceptance criterion 2         | `req-u12-icons-layout-theme` |
| `components/layout/SprintPipelineBar.tsx` | Phase-strip structural render — given `projectId`/`sprintId` route params and an in-flight sprint, the Inception/Construction/Review phase buttons render with the correct done/current/future styling                                           | `render-interaction` | acceptance criterion 2         | `req-u12-icons-layout-theme` |
| `components/layout/SprintPipelineBar.tsx` | No-render guard — when `projectId` or `sprintId` is absent from route params, the component renders nothing                                                                                                                                      | `render-interaction` | derived invariant — Rule U12-3 | `req-u12-icons-layout-theme` |

All 8 files in this unit's scope resolve to `covered` rows above under
`requirementId: req-u12-icons-layout-theme`, satisfying Rule U12-7
(`req-plan-traceability`) before this chapter is reported complete.
