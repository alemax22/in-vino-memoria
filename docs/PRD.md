# PRD – Wine Cellar & Diary

| Field | Value |
|---|---|
| Version | 0.2 |
| Status | Draft |
| Owner | Alessandro Cattapan |
| Date | 2026-10-14 |
| Approvers | Alessandro Cattapan |

## Change log

| Version | Date | Author | Change |
|---|---|---|---|
| 0.1 | 2026-10-14 | Alessandro Cattapan | Initial draft |
| 0.2 | 2026-10-14 | Alessandro Cattapan | App UI entirely in Italian (MVP); Q-05 resolved |

## 1. Summary

I keep about 60 bottles split between a wine fridge (cantinetta) and a cellar (cantina), with no reliable view of what I have and where. I also have no record of the wines I drink, either my own or those at restaurants or friends' homes. The app is a self-hosted, phone-friendly web app. It adds bottles from a photo of the label using AI extraction followed by a quick review, tracks quantity per location, and logs each opened bottle into a personal wine diary with rating and notes. External bottles go into the same diary without touching inventory. Expected outcome: an inventory that matches reality, and a diary that covers at least 90 % of the bottles I drink, at a running cost of 5 CHF/month or less.

## 2. Background & problem

### 2.1 Current situation
- About 30 bottles in the cantinetta and about 30 in the cantina.
- Nothing is tracked. To know what I have, I have to go and look.
- Wines drunk at restaurants and friends' homes are forgotten, along with any impressions of them.

### 2.2 Evidence
- Personal experience: there is no inventory view across the two storage locations.
- Existing apps (Vivino, CellarTracker) were evaluated and rejected: they need an expensive subscription and are too complex for this need.

### 2.3 Why now & cost of doing nothing
- Without tracking, I keep buying duplicates, forget bottles in the cellar, and lose the memory of wines I liked.
- The cellar sometimes goes above 20 °C, so knowing what sits there matters for drinking it in time.
- Cost of doing nothing for 12 months: no tasting history, and an inventory that stays guesswork.

## 3. Goals & success metrics

| ID | Goal | KPI | Baseline | Target | Measurement |
|---|---|---|---|---|---|
| G-01 | Accurate inventory per location | % of bottles matching a physical count | N/A – not tracked today | ≥ 98 % | Manual spot count of both locations, 1 month after go-live |
| G-02 | Effortless bottle capture | Median time from photo to saved bottle | N/A – no tool | ≤ 30 s, ≤ 1 field corrected on average | App timestamps + correction counter, first 50 captures |
| G-03 | Complete wine diary | % of bottles drunk that are logged (own + external) | 0 % | ≥ 90 % | Self-assessment, monthly for 3 months |
| G-04 | Low running cost | Monthly cost (AI API + extras) | N/A | ≤ 5 CHF/month [ASSUMPTION] | AI provider billing, monthly |
| G-05 | Simpler than existing apps | Taps to open/drink a bottle in inventory | N/A | ≤ 3 taps | Usability walkthrough before MVP release |

## 4. Non-goals

- Multi-user use, sharing or social features.
- Native iOS app in the MVP (planned for a later phase).
- Drinking-window alerts (Phase 2).
- Cellar temperature monitoring via Home Assistant (Phase 2).
- Community ratings, prices or reviews from external wine databases.
- Buying or selling wine, and market valuation of the cellar.
- Barcode scanning.

## 5. Users & personas

| Persona | Description | Key needs |
|---|---|---|
| Owner (Ale) | Single user and wine enthusiast with a focus on Italian wines. Technically skilled, runs a home server. Stores wine in a cantinetta and a cantina. | Know what he has and where; capture bottles fast from a photo; log every bottle drunk, at home or outside; add a rating and notes; low cost and low complexity |

## 6. User journeys / use cases

| ID | Use case | Actor | Description |
|---|---|---|---|
| UC-01 | Add new bottles | Owner | Takes a photo of the label. The app extracts the data and shows a review screen. Owner fixes any wrong fields, sets quantity, location, price and shop, then saves. |
| UC-02 | Add more of an existing wine | Owner | Takes a photo. The app recognises the same wine and vintage already in the inventory and offers to increase its quantity. |
| UC-03 | Open / drink own bottle | Owner | Finds the wine, taps "Open". Quantity at that location drops by 1 and a diary entry is created; owner can add a rating and notes now or later. |
| UC-04 | Log external bottle | Owner | At a restaurant or friend's home, takes a photo, reviews the extracted data, adds place, date, rating and notes. Inventory is unchanged. |
| UC-05 | Check inventory | Owner | Views totals per location, searches or filters (type, region, vintage) to decide what to drink. |
| UC-06 | Move bottles | Owner | Moves N bottles from cantina to cantinetta (e.g. to chill before drinking). |
| UC-07 | Browse diary | Owner | Scrolls the timeline of wines drunk and filters by own/external, rating or producer to recall what he liked. |
| UC-08 | Initial load | Owner | Scans about 60 existing bottles one wine at a time to populate the inventory. |

## 7. Functional requirements

| ID | Requirement | Priority | Source | Acceptance criteria |
|---|---|---|---|---|
| FR-001 | The system shall let the user capture a label photo with the phone camera or pick one from the gallery. | Must | UC-01, UC-04 | Given the add screen, when the user taps "Photo", then the camera opens and a gallery option is available. |
| FR-002 | The system shall extract from the photo: producer, wine name, vintage, appellation and classification (e.g. DOC, DOCG, IGT, AOC), region, country, grape varieties, wine type (red, white, rosé, sparkling, sweet, fortified), alcohol % and bottle size. | Must | G-02, UC-01 | Given a clear front-label photo, when extraction completes, then every field printed on the label is pre-filled, and fields that are not on the label are left empty, never guessed. |
| FR-003 | The system shall show a review screen with all extracted fields editable before saving, and highlight low-confidence fields. | Must | G-02, UC-01 | Given extraction has finished, when the review screen opens, then all fields are editable and low-confidence fields are visually marked. |
| FR-004 | The system shall let the user add a second photo (back label) to complete missing fields. | Should | UC-01 | Given the review screen, when the user adds a back-label photo, then empty fields are filled from it without overwriting fields the user has edited. |
| FR-005 | The system shall allow manual entry of all fields without a photo, and as a fallback when extraction fails. | Must | UC-01 | Given extraction fails or times out, when the error appears, then the user can continue with an empty, editable form and the photo kept. |
| FR-006 | The system shall detect when a scanned wine matches an existing wine (same producer, name, vintage and size) and offer to increase its quantity instead of creating a duplicate. | Must | UC-02 | Given a wine already in inventory, when the user scans the same label, then the app offers "Add to existing", and accepting increases the quantity. |
| FR-007 | The system shall record quantity per storage location, with "Cantinetta" and "Cantina" predefined and locations user-configurable. | Must | G-01, UC-01 | Given the save step, when the user sets quantity 3 and location "Cantina", then the inventory shows 3 bottles of that wine in Cantina. |
| FR-008 | The system shall record purchase price (with currency, default CHF), purchase date and place of purchase per batch of bottles added. | Must | UC-01 | Given the review screen, when the user enters price and shop, then they are saved and shown on the wine detail page. |
| FR-009 | The system shall show the inventory with counts per location and an overall total. | Must | G-01, UC-05 | Given saved bottles, when the user opens the inventory, then counts per location and the total equal the sum of recorded quantities. |
| FR-010 | The system shall search the inventory by producer, wine name, grape and vintage, and filter by type, region, country and location. | Must | UC-05 | Given the inventory, when the user types "Barolo", then only matching wines are listed. |
| FR-011 | The system shall let the user open a bottle in at most 3 taps from the inventory. Opening decreases quantity at the chosen location by 1, records date and time, and creates a diary entry. | Must | G-03, G-05, UC-03 | Given 2 bottles in Cantinetta, when the user taps Open and confirms, then Cantinetta shows 1 and a diary entry dated now exists. |
| FR-012 | The system shall let the user undo an "Open" action for 10 seconds afterwards. | Should | UC-03 | Given a bottle was just opened, when the user taps Undo within 10 s, then quantity and diary are restored. |
| FR-013 | The system shall let the user move N bottles between locations. | Must | UC-06 | Given 4 bottles in Cantina, when the user moves 2 to Cantinetta, then Cantina shows 2 and Cantinetta shows 2. |
| FR-014 | The system shall let the user adjust quantity manually with a reason (broken, gifted, correction). | Should | G-01 | Given a wine, when the user removes 1 with reason "broken", then quantity decreases and no diary entry is created. |
| FR-015 | The system shall let the user log an external bottle (not from inventory) in the diary from a photo, with the same extraction and review as FR-002/FR-003, without changing inventory. | Must | G-03, UC-04 | Given the user logs an external bottle, when saved, then a diary entry marked "External" exists and inventory counts are unchanged. |
| FR-016 | The system shall record for each diary entry: date, source (own/external), place (e.g. restaurant name), occasion and companions (optional free text). | Must | UC-04 | Given an external entry, when the user enters "Ristorante X", then the place is shown on the entry. |
| FR-017 | The system shall let the user add a personal rating from 1 to 5 to any diary entry, optional and editable later. | Must | UC-03, UC-04 | Given a diary entry, when the user sets 4, then 4 is shown on the entry and can be changed later. |
| FR-018 | The system shall let the user add free-text notes to any diary entry, optional and editable later. | Must | UC-03, UC-04 | Given a diary entry, when the user writes notes and saves, then the notes are shown on the entry. |
| FR-019 | The system shall show the diary as a reverse-chronological timeline, filterable by source, rating, producer and date range. | Must | UC-07 | Given 10 entries, when the user filters "External", then only external entries are shown, newest first. |
| FR-020 | The system shall store the label photo with each wine and each diary entry. | Must | UC-01, UC-04 | Given a saved wine, when the user opens its detail page, then the label photo is shown. |
| FR-021 | The system shall show per-wine history: bottles bought, opened and remaining, with dates. | Should | UC-05 | Given a wine bought 6 times and opened 4 times, when its detail page opens, then it shows 6 bought, 4 opened and 2 remaining. |
| FR-022 | The system shall export inventory and diary as CSV and JSON. | Should | NFR-DATA | Given data exists, when the user taps Export, then CSV and JSON files are downloaded with all fields. |
| FR-023 | The system shall show total cellar value based on purchase prices. | Could | UC-05 | Given bottles with prices, when the inventory opens, then the total value per location and overall is shown. |
| FR-024 | The system shall let the user mark an external wine as "Buy again". | Could | UC-04 | Given an external entry, when flagged, then it appears in a "Buy again" list. |
| FR-025 | The system shall show drinking-window reminders. | Won't | Phase 2 | N/A – deferred to Phase 2. |
| FR-026 | The system shall show cellar temperature from Home Assistant. | Won't | Phase 2 | N/A – deferred to Phase 2. |

## 8. Non-functional requirements

### 8.1 Performance (PERF)
| ID | Requirement | Target | Priority | Verification |
|---|---|---|---|---|
| NFR-PERF-001 | Label extraction time from photo upload to pre-filled review screen | p95 ≤ 10 s on 4G | Must | Timed test with 20 labels |
| NFR-PERF-002 | Inventory and diary screens load time | p95 ≤ 1 s with 500 wines / 2,000 entries | Should | Load test with synthetic data |

### 8.2 Scalability & capacity (SCAL)
| ID | Requirement | Target | Priority | Verification |
|---|---|---|---|---|
| NFR-SCAL-001 | Data volume supported | 500 wines, 1,000 bottles, 3,000 diary entries, 5,000 photos (3-year horizon) [ASSUMPTION] | Should | Synthetic data test |

### 8.3 Availability (AVAIL)
| ID | Requirement | Target | Priority | Verification |
|---|---|---|---|---|
| NFR-AVAIL-001 | App reachable from home and remotely (restaurant use) | Best effort, ≥ 99 % monthly [ASSUMPTION]; no maintenance window needed | Should | Uptime monitor on home server |

### 8.4 Reliability & resilience (REL)
| ID | Requirement | Target | Priority | Verification |
|---|---|---|---|---|
| NFR-REL-001 | No data loss if the AI API fails: photo kept, manual entry possible | 0 lost photos or entries | Must | Fault test with API disabled |
| NFR-REL-002 | Inventory updates are idempotent; a double tap on "Open" decrements only once | 0 double decrements | Must | Automated test |
| NFR-REL-003 | External diary entry can be captured with poor connectivity and synced later | Queued entries synced when back online | Should | Offline test in airplane mode |

### 8.5 Backup & disaster recovery (DR)
| ID | Requirement | Target | Priority | Verification |
|---|---|---|---|---|
| NFR-DR-001 | Automated backup of database and photos | Daily, 30-day retention, copy kept off the home server; RPO 24 h, RTO 4 h | Must | Quarterly restore test |

### 8.6 Security (SEC)
| ID | Requirement | Target | Priority | Verification |
|---|---|---|---|---|
| NFR-SEC-001 | Access requires authentication; single user account | Login required for all screens and APIs | Must | Manual test of unauthenticated access |
| NFR-SEC-002 | Remote access only over an encrypted channel (VPN or HTTPS with valid certificate) | TLS 1.2+ | Must | Configuration review |
| NFR-SEC-003 | AI API key stored as a secret, never sent to the browser | 0 secrets in client code | Must | Code review |

### 8.7 Privacy & data protection (PRIV)
| ID | Requirement | Target | Priority | Verification |
|---|---|---|---|---|
| NFR-PRIV-001 | Photos sent to the AI provider are stripped of EXIF metadata (GPS) | 0 location data sent | Must | Inspect outgoing request |
| NFR-PRIV-002 | AI provider must not use submitted images for training | Provider terms confirm no training on API data | Should | Terms review |

### 8.8 Compliance & regulatory (COMP)
| ID | Requirement | Target | Priority | Verification |
|---|---|---|---|---|
| NFR-COMP-001 | N/A – personal, single-user app with no third-party personal data; Swiss nDSG obligations do not apply to purely private use | N/A | Won't | N/A |

### 8.9 Auditability (AUDIT)
| ID | Requirement | Target | Priority | Verification |
|---|---|---|---|---|
| NFR-AUDIT-001 | Every inventory change (add, open, move, adjust) is logged with date, quantity and reason | 100 % of changes | Should | Compare log with inventory after test session |

### 8.10 Logging, monitoring & observability (OBS)
| ID | Requirement | Target | Priority | Verification |
|---|---|---|---|---|
| NFR-OBS-001 | Log extraction requests: duration, success/failure, fields corrected by the user, cost | Per-request log; monthly cost report | Should | Log review |
| NFR-OBS-002 | Health check endpoint for home-server monitoring | Endpoint responds with status | Could | Manual test |

### 8.11 Usability (USAB)
| ID | Requirement | Target | Priority | Verification |
|---|---|---|---|---|
| NFR-USAB-001 | Open a bottle from inventory | ≤ 3 taps | Must | Walkthrough |
| NFR-USAB-002 | Add a bottle from photo, including review | Median ≤ 30 s | Must | Timed test, 20 labels |
| NFR-USAB-003 | Designed for one-handed phone use | All primary actions reachable at 390 px width | Should | Device test |

### 8.12 Accessibility (ACC)
| ID | Requirement | Target | Priority | Verification |
|---|---|---|---|---|
| NFR-ACC-001 | Basic accessibility: readable contrast, scalable text | WCAG 2.2 AA for contrast and text resize [ASSUMPTION] | Could | Automated audit (e.g. Lighthouse) |

### 8.13 Localisation (L10N)
| ID | Requirement | Target | Priority | Verification |
|---|---|---|---|---|
| NFR-L10N-001 | The entire UI (labels, messages, errors, prompts, notifications) is in Italian | 100 % of UI text in Italian; no English strings | Must | Screen-by-screen review |
| NFR-L10N-002 | Extraction handles Italian, French, German and Spanish labels | Correct fields on 90 % of test labels | Must | Test set of 30 labels |
| NFR-L10N-003 | Formats: dates dd.mm.yyyy, currency CHF default with EUR option, time zone Europe/Zurich | As stated | Should | Review |
| NFR-L10N-004 | Controlled values (wine type, adjustment reasons, source own/external) and export column headers are shown in Italian (e.g. Rosso, Bianco, Rosato, Spumante, Dolce, Liquoroso); label data such as producer and wine name are kept exactly as printed | 100 % of controlled values in Italian | Must | Review + extraction test set |

### 8.14 Compatibility (COMPAT)
| ID | Requirement | Target | Priority | Verification |
|---|---|---|---|---|
| NFR-COMPAT-001 | Works on iPhone Safari (current and previous iOS major version) and installs as a PWA on the home screen | iOS current and current−1 | Must | Device test |
| NFR-COMPAT-002 | Works on desktop Chrome/Safari for browsing and export | Latest versions | Could | Manual test |

### 8.15 Interoperability (INTOP)
| ID | Requirement | Target | Priority | Verification |
|---|---|---|---|---|
| NFR-INTOP-001 | Backend exposes a versioned REST/JSON API, so a future native iOS app reuses it without changes | API v1 documented (OpenAPI) | Must | Review of API spec |
| NFR-INTOP-002 | AI extraction is behind an interface, so the provider can be swapped without UI changes | Provider swap needs only configuration and an adapter | Should | Design review |

### 8.16 Data quality, retention & archiving (DATA)
| ID | Requirement | Target | Priority | Verification |
|---|---|---|---|---|
| NFR-DATA-001 | Quantities can never go negative | 0 negative quantities | Must | Automated test |
| NFR-DATA-002 | Vintage is a 4-digit year or "NV"; rating is an integer from 1 to 5 | Validation on save | Must | Automated test |
| NFR-DATA-003 | Data and photos kept indefinitely; full export available (FR-022) | No automatic deletion | Should | Review |

### 8.17 Maintainability & supportability (MAINT)
| ID | Requirement | Target | Priority | Verification |
|---|---|---|---|---|
| NFR-MAINT-001 | Code in a Git repository with a README covering setup, configuration and backup/restore | README complete | Should | Review |

### 8.18 Testability (TEST)
| ID | Requirement | Target | Priority | Verification |
|---|---|---|---|---|
| NFR-TEST-001 | Reference set of 30 real label photos with expected fields, to measure extraction accuracy | Field accuracy ≥ 90 %; vintage and producer ≥ 95 % | Must | Run the test set before each provider or prompt change |
| NFR-TEST-002 | Automated tests for inventory logic (add, open, move, undo) | 100 % of inventory operations covered | Should | CI run |

### 8.19 Deployability & portability (DEPLOY)
| ID | Requirement | Target | Priority | Verification |
|---|---|---|---|---|
| NFR-DEPLOY-001 | Runs as container(s) on the home server (Docker / Kubernetes) | One-command deploy; configuration via environment variables | Must | Fresh install test |
| NFR-DEPLOY-002 | Rollback to the previous version without data loss | Rollback ≤ 15 min | Should | Rollback test |

### 8.20 Operability (OPS)
| ID | Requirement | Target | Priority | Verification |
|---|---|---|---|---|
| NFR-OPS-001 | Self-operated by the owner; no external support | Runbook for backup restore and API key rotation | Could | Review |

### 8.21 Licensing & cost (COST)
| ID | Requirement | Target | Priority | Verification |
|---|---|---|---|---|
| NFR-COST-001 | Monthly AI API cost ceiling, with a hard spending limit set at the provider | ≤ 5 CHF/month [ASSUMPTION] | Must | Provider billing review |
| NFR-COST-002 | No paid subscriptions or licences; open-source dependencies only | 0 CHF in licences | Must | Dependency review |

### 8.22 Sustainability (SUST)
| ID | Requirement | Target | Priority | Verification |
|---|---|---|---|---|
| NFR-SUST-001 | Photos resized/compressed before upload and storage | ≤ 500 KB per stored photo | Should | Storage check |

## 9. Integrations & dependencies

| System / team | Direction | Purpose | Owner |
|---|---|---|---|
| Cloud vision AI API (provider TBD) | out | Label data extraction from photos | Alessandro Cattapan |
| Home server (Docker/Kubernetes) | both | Hosting app, database and photos | Alessandro Cattapan |
| Remote access (VPN or reverse proxy, TBD) | in | Use from restaurants / outside home | Alessandro Cattapan |
| Off-site backup storage (TBD) | out | Backup copy | Alessandro Cattapan |
| Home Assistant | in | Cellar temperature (Phase 2) | Alessandro Cattapan |

## 10. Data requirements

| Entity | Source of truth | Notes (migration, reporting) |
|---|---|---|
| Wine | App database | Producer, name, vintage, appellation and classification, region, country, grapes, type, ABV, bottle size, label photo. Unique key: producer + name + vintage + size. |
| Location | App database | Predefined: Cantinetta, Cantina; user-configurable. |
| Stock | App database | Quantity per wine per location; never negative. |
| Purchase | App database | Wine, quantity, price, currency, date, shop. |
| Inventory movement | App database | Add, open, move, adjust; date, quantity, reason (audit). |
| Diary entry | App database | Date, source (own/external), wine ref or wine data, place, occasion, companions, rating 1–5, notes, photo. |
| Initial load | Manual scan | About 60 existing bottles scanned at go-live (UC-08); no migration from other tools. |

## 11. UX notes

- Mobile-first PWA, entirely in Italian; three main tabs: Inventario, Aggiungi (fotocamera), Diario.
- "Add" opens the camera directly; one big "Open bottle" action on each wine.
- After "Open", show a lightweight prompt for rating and notes, which can be skipped and filled in later.
- Minimal by design: features that hurt simplicity (G-05) are deferred.

## 12. Release plan

| Phase | Scope | Target date |
|---|---|---|
| MVP | Web app (PWA): photo extraction + review, inventory per location, open/move, diary (own + external), rating, notes, purchase price and shop | TBD |
| Phase 2 | Drinking-window reminders, cellar temperature via Home Assistant, cellar value | TBD |
| Phase 3 | Native iOS app on the same API | TBD |

## 13. Risks

| ID | Risk | Impact | Likelihood | Mitigation | Owner |
|---|---|---|---|---|---|
| R-01 | AI misreads labels (vintage, producer, classification), especially artistic or Italian labels | H | M | Mandatory review screen, back-label photo, test set (NFR-TEST-001) | Alessandro Cattapan |
| R-02 | Owner forgets to log opened bottles, so inventory drifts | H | M | ≤ 3-tap open, monthly spot count, manual adjust | Alessandro Cattapan |
| R-03 | Exposing the home server to the internet creates a security risk | H | M | VPN preferred, authentication, TLS (NFR-SEC) | Alessandro Cattapan |
| R-04 | Home server downtime blocks use at restaurants | M | M | Offline capture queue (NFR-REL-003) | Alessandro Cattapan |
| R-05 | AI API price or terms change | L | M | Provider abstraction (NFR-INTOP-002), spending cap | Alessandro Cattapan |
| R-06 | Initial load of about 60 bottles is tedious, so the app is never adopted | M | M | Fast scan flow; load in batches per location | Alessandro Cattapan |

## 14. Assumptions & constraints

### 14.1 Assumptions
- [ASSUMPTION] Running-cost ceiling of 5 CHF/month is acceptable.
- [ASSUMPTION] One photo shows one wine; multi-bottle photos are out of scope.
- [ASSUMPTION] Location granularity is storage area only (no shelf or slot).
- [ASSUMPTION] The owner uses an iPhone.

### 14.2 Constraints
- Single user.
- Self-hosted on the owner's home server; cloud AI API for extraction only.
- Web app (PWA) for the MVP; the API must support a future native iOS app.
- No paid subscriptions.
- App UI entirely in Italian.

## 15. Alternatives considered

| Option | Why rejected |
|---|---|
| Vivino | Expensive subscription; too complex for the need |
| CellarTracker | Expensive subscription; too complex for the need |
| Spreadsheet | No photo extraction; manual entry too tedious to keep up |
| Native iOS app first | Slower to build; web app validates value first |
| Fully local AI model | Lower label-reading accuracy; home hardware limits |

## 16. Open questions

| ID | Question | Owner | Due |
|---|---|---|---|
| Q-01 | Remote access method: VPN (e.g. WireGuard/Tailscale) or public HTTPS reverse proxy? | Alessandro Cattapan | 2026-10-31 |
| Q-02 | Which AI provider/model for extraction, based on the test-set accuracy and cost? | Alessandro Cattapan | 2026-10-31 |
| Q-03 | Is shelf/slot position inside a location needed later? | Alessandro Cattapan | 2026-11-30 |
| Q-04 | Where to keep the off-site backup copy? | Alessandro Cattapan | 2026-10-31 |
| Q-05 | Should the UI be Italian from the MVP? – Resolved in v0.2: yes, entirely in Italian (NFR-L10N-001) | Alessandro Cattapan | 2026-10-14 |

## 17. Glossary

| Term | Definition |
|---|---|
| Cantinetta | Wine fridge in the house |
| Cantina | Basement cellar |
| Own bottle | Bottle from the owner's inventory |
| External bottle | Bottle drunk outside the inventory (restaurant, friend) |
| Diary entry | Record of a bottle drunk, with date, place, rating and notes |
| Extraction | AI reading of label data from a photo |
| PWA | Progressive Web App, a web app installable on the phone home screen |
| NV | Non-vintage |