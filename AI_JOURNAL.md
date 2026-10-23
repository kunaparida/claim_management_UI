# AI Working Journal — Claim Management UI

| | |
|---|---|
| **Project** | Claim Management UI (`claim_management_UI`) — Angular front end for `chubb-claims-backendService` |
| **Author** | Kuna Parida |
| **Document purpose** | Transparent record of how AI assistance is used to build the UI: what was requested, what was produced, and what was accepted, challenged or overridden, with the reasoning. |
| **Last updated** | 2026-10-22 |

The author reviews and takes responsibility for every line in this repository. AI output is treated as a proposal, not a
result, until it has been verified by a build, a test run or a manual check. Each entry says what was **actually verified**
and what was **not**.

> **How to read the Decision line.** `Decision` is the *author's* call. Where the author has not yet reviewed an entry it says
> `Pending author review` — it is deliberately not pre-filled. Facts the AI found or got wrong are recorded under *Result* and
> *Corrections*, not disguised as the author's decisions.

## 1. Conventions

| Decision | Meaning |
|---|---|
| **ACCEPTED** | AI output used as generated. |
| **CHALLENGED** | AI output questioned or verified, then kept or changed. |
| **OVERRODE** | The author decided differently from the AI. |

Prompts appear in the order they were sent. Each entry follows the same structure: *Prompt, Result, Verification, Decision*.

## 2. Context

| Item | Where | Notes |
|---|---|---|
| **This UI** | `C:\project\claim_management_UI` | Angular **22.2**, zoneless, Vitest. Created with the CLI; untouched scaffold at the start of this journal. |
| **Backend (the one this UI integrates)** | `C:\project\chubb-claims-backendService` | Spring Boot 3.4.5 / Java 21. Own `CLAUDE.md`, `README.md`, `docs/AI-WORKING-JOURNAL.md`, Postman collection. Swagger: `http://localhost:8080/swagger-ui/index.html`. |
| **Assessment brief** | `V2_FullStack_candidate_assessment_brief.pdf` (Chubb APAC) | Backend first; "a well-built backend with a partial frontend will outscore a rushed attempt at both". The UI serves claimants, claims staff and managers from one Angular app. |

## 3. Status

| Area | State |
|---|---|
| `CLAUDE.md` | Written (Prompt 1), updated with the real state (Prompt 2). |
| Dev proxy / CORS workaround | Done and proven (see Prompt 2) |
| Models, `ApiService`, auth, interceptors, guards | Done — 19/19 backend endpoints covered |
| Claimant screens | Done — register, login, my claims, report incident, claim page (answer questions) |
| Officer screens | Done — work queue, pick up, assigned claims, review / request info / approve / reject / settle |
| Manager dashboard | Done — workload, liability per market, officer performance, assign picker |
| Tests (Vitest) | 216 passing in 15 files |
| Manual end-to-end script | `docs/E2E-VALIDATION-FLOW.txt` (Prompt 3) |
| Committed to git | **No** — nothing committed yet |

---

# Part A: Prompt Log

## Prompt 1: Create CLAUDE.md and this journal before integrating

**Prompt** (verbatim)
```text
I am working on a Chubb APAC Full Stack Developer Assessment.

Backend implementation is already completed and fully working.

Backend Location: C:\project\chubb-claims-backendService

Frontend Location: C:\project\claim_management_UI

Backend Technology:

Java 21
Spring Boot
Maven
JWT Authentication
REST APIs
Kafka Event Publishing
Swagger/OpenAPI
Swagger URL: http://localhost:8080/swagger-ui/index.html

Please use the Swagger specification as the source of truth for all request and response contracts.

**Result**

* Created `CLAUDE.md`: purpose, tooling, the full backend contract (19 endpoints, request/response shapes, error format), the
  status workflow, the backend facts that shape the UI, a UI action table, a brief→UI checklist, and architecture rules.
* Created this `AI_JOURNAL.md` in the same structure as the backend's journal.
* **No code was written, nothing was integrated, nothing was committed, and the backend repository was not modified.**

**What the AI read before writing** (so the files describe the real projects, not assumptions)

* Backend: `AuthController`, `ClaimController`, `ManagerDashboardController`, all 14 DTOs, `SecurityConfig`,
  `GlobalExceptionHandler`, `ClaimService` (the rules), `ManagerDashboardService`, `application.properties`, its `README.md` and `CLAUDE.md`.
* UI: `package.json`, `angular.json`, `tsconfig.json`, `.prettierrc`, `app.config.ts`, `app.routes.ts`, `app.ts`, the default README.

**Findings that shape the integration** (each is also in `CLAUDE.md`)

1. **The backend has no CORS configuration** (searched `src/main` and `pom.xml`: nothing). A browser app on `:4200` calling `:8080`
   will be blocked. The UI therefore needs the Angular dev **proxy** and relative `/api` URLs. *Not* fixed by touching the backend.
2. **No `allowedActions`.** The backend never says which buttons to show, so the UI needs one central action table
   (status × role × assignment) mirroring `ClaimService`. The server stays the authority — 403/404/409 must be handled.
3. **`pickup`/`assign` do not change status** (`Claim.assign` only sets the officer). The claim stays `SUBMITTED` until the officer calls `review`.
4. **No "list officers" endpoint.** `GET /api/dashboard/performance` returns *all* officers (verified in `ManagerDashboardService`),
   so it can feed the manager's assign picker.
5. **Money has no currency.** `amountClaimed` is a bare number, `market` a 2-letter code, and the dashboard's top-level totals sum
   across markets (mixed currencies). The UI must show `byMarket` lines first and caveat the grand total.
6. IDs are numeric (`Long`); claim list endpoints are **role-scoped** (officer's `GET /api/claims` = assigned only; the queue is a separate endpoint).

**Corrections** (things the AI got wrong or that were in the way — recorded honestly)

* **Wrong earlier claim:** the AI earlier told the author that Node.js was not installed. It *is* installed (v26.3.1 under
  `C:\Program Files\nodejs`); the AI's own shell simply did not have it on its PATH. No download is needed for this UI.
* **Port conflict:** `localhost:8080` and `localhost:4200` were occupied by servers the AI had started earlier for the unrelated
  `Claim_Management` build, so the Swagger link in the prompt would have shown *that* API instead of this backend (its tags are
  "Claims", whereas this backend's controller shows as `claim-controller`). The AI stopped only its own two processes; both ports are now free.
  Start the real backend with `mvn spring-boot:run` before opening Swagger.

**Verification**

* Done: read the sources listed above; searched the backend for CORS configuration (none); queried `:8080` and saw the *other* app's
  OpenAPI document; confirmed both ports free after stopping the AI's processes.
* **Not done:** the real backend has not been started in this session, so the contract in `CLAUDE.md` comes from reading its code,
  not from calling it. The UI has not been installed, built or tested. The Swagger link was not opened.

**Decision:** `Pending author review`

---

## Prompt 2: Proceed — integrate the backend APIs into the UI

**Prompt** (verbatim)
```text
please procced
```

**Context.** This answered the AI's proposal to start with the proxy, models and API service, then login/guards, then the three role
screens. The AI also asked whether to commit as it went; the answer did not cover that, so **nothing was committed** (the repo's
`CLAUDE.md` says "commit only when asked"). Before installing anything the AI asked, and the author approved: *"Yes, install and run"*
(`npm install` in this repo, and starting the backend with Maven).

**Result** (all under `src/app`; templates live in `.html` files next to the component)

* `core/` — `models.ts` (types mirroring the backend DTOs), `api.service.ts` (the only file with URLs), `auth.service.ts`,
  `auth.interceptor.ts`, `guards.ts`, `problem.ts`, `claim-actions.ts` (the one action table).
* `shell/` (role navigation), `features/login`, `register`, `claims` (list, new-claim, claim-detail), `queue`, `dashboard`, `shared/`.
* `proxy.conf.json` + `angular.json` `serve.options.proxyConfig`; global styles with light/dark tokens.

**Design decisions** (and why)

* **Dev proxy, not a backend change.** The backend has no CORS configuration. Proven both ways against the running backend: login through
  `:4200/api` works; a direct cross-origin preflight to `:8080` returns **401 with no CORS headers**, which a browser would block.
* **One action table** (`claim-actions.ts`) because the backend does not say which actions are allowed. It is tested against an independent
  copy of the backend's `TRANSITIONS` map (every status × role × assignment). The server stays the authority: every action handles
  403/404/409 by showing the server's `detail` and reloading the claim.
* **Money shows a market, never a currency symbol**, and the dashboard shows per-market liability first with the grand total labelled
  "unconverted, mixed markets" (the backend sums across markets with no currency).
* Zoneless Angular 22: all template state is signals; no zone-dependent code.

**Corrections — what went wrong, and how it was resolved**

1. **A real component bug, found by a unit test.** In the work queue, losing a pick-up race (409) set the error message and then reloaded the
   list, and the reload cleared the message — the officer would never have been told *why* the pick-up failed. Fixed with
   `load(keepError)`; the test `losing the race shows the server reason…` now passes.
2. **First full test run: 4 failures, none in app code except #1.** Two specs called `TestBed.inject` before configuring the module; one
   flushed a request that `forkJoin` had already cancelled (`verify({ ignoreCancelled: true })` is the right check); one spec did not register
   the auth interceptor, so a 401 could not sign anyone out in the test (the real app registers it). All fixed in the specs.
3. `GET /api/auth/me` was defined in `ApiService` but unused by any screen. It is now used on app load to re-validate the stored session
   (a changed role updates the navigation; a 401 signs the user out).
4. The browser tab title was still the CLI default ("ClaimManagementUi"); changed to "Claims".
5. One assumption confirmed rather than guessed: `pickup` leaves the claim `SUBMITTED` (checked live: it only sets the officer; `review` starts the review).

**Verification**

* **Contract:** live `/v3/api-docs` vs `ApiService` — 19 endpoints in the backend, 19 called by the UI, 0 missing either way (same verbs and paths).
* **Live lifecycle against the real backend** (through the dev proxy, same endpoints and bodies as the UI): 28 checks passed — register,
  login ×3, 401 / 400 field errors / 403 email-mismatch, create, queue, pick up (stays SUBMITTED), review, request-info, empty-note 400,
  409 invalid transition, claimant answer, approve, settle, 409 on a settled claim, 6-entry history, another claimant gets 404, manager
  sees all officers, dashboards, officer gets 403 on the dashboard, manager assigns.
* **Browser walkthrough** (real backend, real UI): registered a claimant → reported an incident (form prefilled from the session) → signed in as the
  seeded officer → claim in the queue → picked up → started review → requested information → signed in as the claimant → saw "Action needed" →
  answered → history showed all four steps → manager dashboard showed per-market liability with the unconverted-total label.
* **Unit tests:** 212 passing at the end of this prompt (216 after Prompt 3's fix); strict AOT production build clean (292.6 kB initial).
* **Mutation check:** made the action table offer SETTLE on `UNDER_REVIEW` (the backend forbids it) → 6 tests failed with
  "SETTLE must be legal from UNDER_REVIEW"; restored.

**Not verified** (be upfront in the walkthrough)

* In a real browser: officer **approve / reject / settle**, the manager's **assign picker**, dark mode, and narrow/mobile layout. The first three
  are covered by unit tests and the scripted API run, not by clicking.
* Accessibility beyond labels/roles/focus styles was not audited.
* `GET /api/claims` has only a status filter on the backend, so the claims list has no search or officer filter.

**Side effects on this machine**

* The AI started the backend with `mvn spring-boot:run` from `C:\project\chubb-claims-backendService` (this writes build output under that repo's
  `target/`; **no backend source file was changed**). Test data exists only in its in-memory H2 database and disappears on restart:
  claimants `ui.check.<timestamp>@example.com`, `other.<timestamp>@example.com`, `demo.claimant.ui@example.com`, and claims #1–#3.
* `npm install` created `node_modules/` (git-ignored). The backend (port 8080) and the Angular dev server (port 4200) were **left running**.

**Decision:** `Pending author review`

---

# Part B: Open questions for the backend

The UI repo does not change the backend. These are things the author may want to change there (or accept as limitations and explain in the walkthrough).

| # | Question | Impact on the UI |
|---|---|---|
| 1 | Add CORS (or accept a dev-proxy-only setup)? | **Confirmed needed:** a direct call from `:4200` gets a 401 with no CORS headers. The UI works only through the dev proxy or a same-origin deployment. |
| 2 | Expose an officers list (or accept `/dashboard/performance` as the source)? | Manager assign-picker. |
| 3 | Add a currency (or a market→currency mapping) to claims and the liability report? | Without it the UI cannot show correct money, and grand totals mix currencies. |
| 4 | Return allowed actions per claim? | Today the UI mirrors the workflow rules itself, which can drift from the backend. |
| 5 | Reassigning `REJECTED`/`SETTLED` claims is not forbidden (the backend's own `CLAUDE.md` notes this). | UI simply will not offer it. |

---

# Part C: Entry template

Copy this for each new prompt, newest at the bottom of Part A.

```markdown
## Prompt N: <short title>

**Prompt** (verbatim)
<what was typed>

**Result**
<what the AI produced / changed; list files>

**Corrections**
<anything the AI got wrong, or the author challenged, and how it was resolved — or "none">

**Verification**
<what was actually run or checked (build, tests, manual) and what was NOT>

**Decision:** ACCEPTED | CHALLENGED | OVERRODE — <author's reasoning>
```
