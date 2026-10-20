<!--
Sync Impact Report
==================
Version change: (unfilled template) → 1.0.0
Bump rationale: initial ratification; every placeholder replaced with project content.

Principles (template slot → new title):
- [PRINCIPLE_1_NAME] → I. Single User, Self-Hosted
- [PRINCIPLE_2_NAME] → II. Italian, Mobile-First PWA
- [PRINCIPLE_3_NAME] → III. API-First, Versioned Contract
- [PRINCIPLE_4_NAME] → IV. Swappable, Private AI Extraction
- [PRINCIPLE_5_NAME] → V. Inventory Integrity (NON-NEGOTIABLE)
- Added: VI. Zero Licences, Bounded AI Cost
- Added: VII. Tests Where Correctness Matters
- Added: VIII. Simplicity Over Features

Added sections:
- Security, Data & Operations Constraints (was [SECTION_2_NAME])
- Development Workflow & Quality Gates (was [SECTION_3_NAME])
- Governance (filled)

Removed sections: none

Templates and commands: not modified by this command; they read this file at runtime.
- .specify/templates/plan-template.md: Constitution Check gate should check principles I–VIII
- .specify/templates/spec-template.md: no change expected
- .specify/templates/tasks-template.md: inventory-logic and label-accuracy test tasks are mandatory

Revision (same day, before first commit): Principle II now requires the full Italian
it-IT conventions: dd/mm/yyyy dates, 24-hour time, decimal comma, EUR default (CHF optional),
Europe/Rome, Italian collation, Italian-Excel CSV. The API and storage stay locale-neutral.
Version kept at 1.0.0 because 1.0.0 was never committed or adopted.

Follow-up TODOs:
- docs/PRD.md conflicts with Principle II: NFR-L10N-003 (dd.mm.yyyy, CHF default,
  Europe/Zurich) and FR-008 ("default CHF") need amending to match.
Ratification date set to the date of first adoption (2026-10-20).
Source: docs/PRD.md v0.2, sections 8 (NFRs) and 14.2 (constraints).
-->

# In Vino Memoria Constitution

## Core Principles

### I. Single User, Self-Hosted

- The system serves exactly one user (the owner). Multi-user, sharing, roles and social
  features MUST NOT be built.
- The application, database and photos MUST run as containers on the owner's home server,
  deployable with one command and configured only through environment variables.
- The only permitted runtime dependency outside the home server is the cloud AI provider used
  for label extraction (plus the off-site backup target).

Rationale: personal tool with no need for tenancy; keeping data at home keeps it private and
cheap (PRD 14.2, NFR-DEPLOY-001).

### II. Italian, Mobile-First PWA

- Every user-visible string (labels, buttons, messages, errors, prompts, notifications,
  controlled values such as wine type and adjustment reason, export column headers) MUST be in
  Italian. No English strings may reach the UI.
- Label data (producer, wine name, appellation) MUST be stored and shown exactly as printed.
- Code, API identifiers and developer documentation MAY be in English; API enum values MUST be
  stable codes, translated to Italian only at the presentation layer.
- The UI MUST follow Italian (`it-IT`) conventions throughout:
  - **Dates**: `dd/mm/yyyy` (e.g. `20/10/2026`); long form `20 ottobre 2026`, with month and
    weekday names in Italian and lowercase. Weeks start on Monday.
  - **Time**: 24-hour `HH:mm` (e.g. `21:30`); no AM/PM.
  - **Numbers**: decimal comma and dot as thousands separator (`1.234,56`); ABV as `13,5%`,
    bottle size as `0,75 l`.
  - **Currency**: EUR is the default, shown as `12,50 €`; CHF is allowed as an option per
    purchase and is shown as `12,50 CHF`. Amounts are never converted between currencies.
  - **Time zone**: Europe/Rome for all displayed dates and times and for "today"/date-range
    logic.
  - **Sorting and search**: Italian collation, ignoring case and accents.
- Locale formatting happens only at the presentation layer. The API and database MUST use
  locale-neutral values: timestamps in ISO 8601 UTC, amounts as decimal numbers with an ISO 4217
  currency code (`EUR`, `CHF`), vintage as a number or `NV`.
- CSV exports MUST open correctly in Italian Excel/Numbers: `;` as field separator, decimal
  comma, `dd/mm/yyyy` dates, UTF-8 with BOM, Italian headers. JSON exports keep the API's
  locale-neutral values.
- The UI MUST be designed mobile-first for iPhone Safari (current and previous iOS major),
  installable as a PWA, with all primary actions reachable one-handed at 390 px width.
- Opening a bottle from the inventory MUST take ≤ 3 taps.

Rationale: NFR-L10N-001/004, NFR-COMPAT-001, NFR-USAB-001/003. The Italian date, currency and
time-zone conventions replace PRD v0.2 NFR-L10N-003 (`dd.mm.yyyy`, CHF default, Europe/Zurich);
keeping the API locale-neutral lets a future iOS app apply its own formatting.

### III. API-First, Versioned Contract

- All functionality MUST be exposed through a versioned REST/JSON API (`/api/v1/...`) that the
  PWA consumes; the PWA MUST NOT rely on server-rendered behaviour or private endpoints.
- The API MUST be described by an OpenAPI document kept in the repository and updated in the
  same change as the endpoint.
- Breaking changes to a published API version are forbidden; they require a new version
  (`v2`) while the previous one keeps working.

Rationale: a future native iOS app must reuse the API unchanged (NFR-INTOP-001, Phase 3).

### IV. Swappable, Private AI Extraction

- Label extraction MUST sit behind a provider interface; the UI and domain code MUST NOT
  reference a specific provider. Swapping providers MUST require only a new adapter and
  configuration.
- The AI API key MUST live server-side as a secret; it MUST never appear in client code,
  responses or logs. The browser never calls the AI provider directly.
- EXIF metadata (including GPS) MUST be stripped from every photo before it is sent to the
  provider; photos SHOULD be resized/compressed (≤ 500 KB stored) before upload.
- Extraction MUST leave fields empty when they are not on the label — never guessed — and MUST
  flag low-confidence fields for review.
- AI failure or timeout MUST NOT lose data: the photo is kept and manual entry stays possible.
- Providers that train on submitted API data SHOULD NOT be used.

Rationale: NFR-INTOP-002, NFR-SEC-003, NFR-PRIV-001/002, NFR-REL-001, FR-002/005, risk R-05.

### V. Inventory Integrity (NON-NEGOTIABLE)

- Stock quantities MUST never be negative; the server MUST reject any operation (open, move,
  adjust, undo) that would make a quantity negative, enforced in domain logic and backed by a
  database constraint.
- Every mutating inventory operation (add, open, move, adjust, undo) MUST be idempotent: the
  client sends an idempotency key and a replay returns the original result without applying
  the change twice. A double tap on "Apri" decrements exactly once.
- Operations that touch several rows (e.g. open = decrement stock + create diary entry; move =
  decrement one location + increment another) MUST be atomic in a single transaction.
- Every inventory change SHOULD be recorded as a movement with date, quantity and reason.
- Validation on save: vintage is a 4-digit year or `NV`; rating is an integer from 1 to 5.

Rationale: an inventory that drifts from reality defeats the product (G-01, NFR-DATA-001/002,
NFR-REL-002, NFR-AUDIT-001).

### VI. Zero Licences, Bounded AI Cost

- Only open-source or free dependencies, tools and services may be used; paid subscriptions
  and licences are forbidden (0 CHF).
- Total AI spend MUST stay ≤ 5 CHF/month, enforced by a hard spending limit set at the provider.
- Each extraction request SHOULD log duration, outcome and estimated cost so a monthly cost
  report is possible.

Rationale: NFR-COST-001/002, NFR-OBS-001, G-04.

### VII. Tests Where Correctness Matters

- Inventory logic (add, open, move, adjust, undo) MUST have automated tests covering 100 % of
  those operations, including the negative-quantity guard and duplicate (idempotent) requests.
  These tests MUST pass before any change is merged or deployed.
- A reference set of 30 real label photos with expected fields MUST be maintained. It MUST be
  run before any change of provider, model or extraction prompt, and the change is accepted
  only if field accuracy is ≥ 90 % and vintage and producer accuracy are each ≥ 95 %, across
  Italian, French, German and Spanish labels.
- Beyond these, tests are added where they pay for themselves; exhaustive UI test suites are
  not required.

Rationale: NFR-TEST-001/002, NFR-L10N-002, risk R-01.

### VIII. Simplicity Over Features

- When a feature conflicts with speed or ease of the core flows (capture a bottle, open a
  bottle, log a diary entry), the feature is deferred.
- Choose the smallest architecture that meets the PRD: a single deployable backend, one
  database, no message brokers, microservices or extra infrastructure without a documented
  need.
- PRD non-goals (multi-user, barcode scanning, external ratings/prices, buying/selling) and
  Phase 2/3 items MUST NOT be pulled into the MVP.
- Any added complexity MUST be justified in the plan's Complexity Tracking section.

Rationale: existing apps were rejected for complexity; simplicity is the product (G-05, PRD 11).

## Security, Data & Operations Constraints

- **Authentication**: login is required for every screen and every API endpoint (single
  account); unauthenticated requests are rejected.
- **Transport**: remote access only over an encrypted channel — VPN or HTTPS with a valid
  certificate (TLS 1.2+).
- **Secrets**: provided via environment variables or the container platform's secret store;
  never committed to the repository.
- **Backup**: database and photos backed up daily, 30-day retention, with a copy off the home
  server (RPO 24 h, RTO 4 h); a restore is tested at least quarterly.
- **Data retention**: data and photos are kept indefinitely; no automatic deletion. Full
  export (CSV and JSON) SHOULD be available.
- **Deployment safety**: schema migrations MUST be versioned and non-destructive so that a
  rollback to the previous release is possible without data loss (target ≤ 15 min).
- **Performance targets**: extraction p95 ≤ 10 s on 4G; inventory and diary screens p95 ≤ 1 s
  at 500 wines / 2,000 diary entries.

## Development Workflow & Quality Gates

- All code lives in Git; the README covers setup, configuration, backup/restore and API key
  rotation.
- Work follows the Spec Kit flow (specify → plan → tasks → implement). Every plan MUST pass the
  Constitution Check against principles I–VIII before design starts and again after design.
- Gates before merging to `main`:
  1. Inventory-logic tests pass (Principle VII).
  2. OpenAPI document matches the implemented endpoints (Principle III).
  3. No English strings added to the UI, and dates, times, numbers and amounts are formatted
     with the `it-IT` conventions (Principle II).
  4. No secrets or provider-specific code in the client (Principle IV).
  5. No new paid dependency (Principle VI).
- Gate before changing AI provider, model or prompt: the 30-label accuracy set meets its
  thresholds (Principle VII).

## Governance

- This constitution takes precedence over specs, plans, tasks and ad-hoc practice. If it
  conflicts with docs/PRD.md, the conflict is resolved by amending one of the two explicitly,
  not by silently diverging.
- Amendments are made by editing this file in a dedicated commit, with a Sync Impact Report
  describing the change; the owner (sole approver) approves them.
- Versioning follows semantic versioning:
  - MAJOR: a principle is removed or redefined in a backward-incompatible way.
  - MINOR: a principle or section is added or materially expanded.
  - PATCH: clarifications and wording fixes with no change in meaning.
- Compliance is reviewed at every `/speckit-plan` (Constitution Check), at every merge to
  `main` (quality gates above) and before each release. Justified deviations are recorded in
  the plan's Complexity Tracking table; unjustified ones block the change.

**Version**: 1.0.0 | **Ratified**: 2026-10-20 | **Last Amended**: 2026-10-20
