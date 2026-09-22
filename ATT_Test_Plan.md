# ATT — Test Plan (Full System, ATT-FR-01 → ATT-FR-10)

**Project:** Smart Attendance & Classroom Verification
**Test Lead:** You (Lead + QA)
**Scope reference:** ATT_User_Stories.md (US-01 → US-22, Phases 1–4)

## 1. Objectives
Verify the complete system against the original brief, ATT-FR-01 through ATT-FR-10, across all four roles (Student, Lecturer/TA, Administrator, Auditor). Testing runs **phase by phase**, in step with the build: each phase gets a full test pass and a go/no-go before the next phase's features are demoed, but every phase's test cases exist in the plan from day one — nothing waits to be written until "later."

**Delivery form:** one Flutter app, role-based flow after login; rule-based and ML logic run as backend modules.

## 2. In Scope (all phases, nothing excluded)
- **Phase 1:** Auth/RBAC (4 roles), course/section/enrollment/timetable/room management incl. CSV import, session lifecycle, rotating signed QR, attendance capture with duplicate prevention
- **Phase 2:** Live roster & manual correction, student correction requests with evidence, filters & exports (CSV/PDF), core dashboard metrics
- **Phase 3:** Time-window/geofence validation, AI rule-based flags + logistic-regression risk model (baseline comparison, precision/recall, calibration), auditor immutable history views
- **Phase 4 (stretch, still tracked):** offline scan queue, push notifications, consent-based face-verification experiment

## 3. Out of Scope (for this plan, not for the project)
- Full penetration testing (cybersecurity does a focused review per phase, not a formal pentest engagement)
- Load/performance testing beyond basic sanity checks until Phase 2 is stable
- Anything not in the original brief or the User Stories document

## 4. Test Levels
| Level | Owner | Notes |
|---|---|---|
| Unit | Backend, AI, Flutter devs | Each dev tests their own endpoint/widget/rule before marking a task done |
| Integration | Lead/QA | API ↔ Flutter app; Backend ↔ rule/ML modules; import pipeline ↔ DB |
| End-to-end / Acceptance | Lead/QA + squad | Runs the phase's acceptance scenario (Section 7) at the end of each phase |
| Security review | Cybersecurity | Runs every phase: token integrity (P1), evidence-upload handling & export data exposure (P2), geofence/location data minimization & model fairness inputs (P3) |
| Data/Model evaluation | AI + Data Analyst | Phase 3 only: baseline vs. model comparison, precision/recall, calibration, documented dataset split |

## 5. Environment
- Docker Compose stack: Postgres + backend (rule/ML modules run in-process)
- Synthetic seed data only — including a synthetic CSV file (valid and deliberately invalid versions) for import testing, and a synthetic historical attendance dataset large enough to train/evaluate the Phase 3 model
- Four test accounts, one per role (student, lecturer/TA, administrator, auditor)
- At least two physical/emulated phones for live QR and geofence testing

## 6. Entry / Exit Criteria (per phase)
**Entry criteria for a phase's testing:**
- All of that phase's endpoints/screens are deployed to the local/dev stack
- Seed/synthetic data required for that phase exists (e.g., historical attendance for Phase 3's model)

**Exit criteria for a phase (go/no-go to start the next phase's build):**
- All Must-priority test cases for that phase pass (see Test Cases sheet, filter by Phase + Priority)
- No open Critical/High severity defects in that phase's features
- That phase's acceptance scenario runs start-to-finish without manual DB intervention

## 7. Acceptance Scenarios by Phase

**Phase 1 (Foundation):**
1. Administrator imports a CSV with 30+ students, including a few deliberately malformed rows → valid rows create records, malformed rows are rejected with a report.
2. Administrator sets up a room and timetable slot.
3. Lecturer opens a session; two students scan successfully; an expired token and a duplicate scan are safely rejected.
4. Auditor logs in and confirms they can view but not edit any record.

**Phase 2 (Correction & Reporting):**
1. Lecturer manually corrects a student's status with a reason → audit log shows before/after.
2. Student submits a correction request with an attached evidence file → lecturer approves it → student sees "approved."
3. Administrator filters attendance by section and date range, exports CSV → totals match the filtered view.
4. Dashboard attendance rate matches a manual count of the same section's raw events.

**Phase 3 (Advanced):**
1. A scan outside the configured time window/geofence is flagged with a specific reason, and no continuous location history is stored.
2. The AI risk model produces a probability for a seeded student, alongside the rule-based baseline; report shows precision/recall and which one performed better.
3. Model service is stopped → attendance capture and rule-based flags continue to work (fallback holds).
4. Auditor views the full immutable history for a session, including every correction ever made to it.

**Phase 4 (Stretch):**
1. A scan made while offline queues locally and syncs once connectivity returns, with no duplicate created on sync.
2. A student opts into the face-verification pilot, consents explicitly, and can revoke consent — with a non-biometric fallback always available.

## 8. Roles & Responsibilities
| Person | Testing responsibility |
|---|---|
| Lead/QA (you) | Owns this plan, maintains the Test Cases sheet, runs integration + acceptance testing every phase, triages defects, calls each phase's go/no-go |
| Backend | Unit tests for auth, session, QR, scan, import, correction, export, geofence endpoints |
| Flutter #1 | Student-facing flows: login, scan, my attendance, correction request + evidence upload, consent screen (P4) |
| Flutter #2 | Staff-facing flows: session/roster/correction review, admin CRUD screens, dashboard, auditor views |
| AI | Rule-engine logic (P1 onward) and the risk model (P3): baseline definition, evaluation metrics, documented limitations |
| Data Analyst | Seed/synthetic data generation for every phase (including the historical dataset needed for P3); dashboard reconciliation |
| Cybersecurity | Token integrity (P1); evidence-upload and export data-exposure review (P2); geofence data minimization and model input fairness review (P3) |

## 9. Defect Severity Definitions
| Severity | Definition | Example |
|---|---|---|
| Critical | Blocks that phase's acceptance scenario entirely | Import corrupts existing enrollment data |
| High | Core Must-story broken but workaround exists | Correction request evidence fails to upload |
| Medium | Should-story or non-blocking issue | Dashboard count off by a small margin |
| Low | Cosmetic / polish | Export PDF formatting |

## 10. Risks & Mitigations
| Risk | Mitigation |
|---|---|
| Full scope is large; team may lose track of what's "done" vs. "in progress" | Every phase has its own explicit go/no-go gate (Section 6) — no phase is called complete without its Must test cases passing |
| Phase 3's ML model needs real-looking historical data that doesn't exist yet | Data Analyst generates a synthetic historical dataset early, in parallel with Phase 1/2 build, not after |
| Evidence upload (P2) and geofence (P3) introduce new privacy/security surface area | Cybersecurity reviews each new data type (files, location) the phase it's introduced, not retroactively |
| Scope discipline slipping into "let's also add X" | Any new requirement not in the User Stories doc goes through the same phase-and-dependency review before it's added — this plan is the single source of truth for what's in scope |

## 11. Deliverables
- This Test Plan
- ATT_Test_Cases.xlsx (all phases, filterable by Phase/Priority/Status)
- Defect log in ClickUp (ATT- prefixed tasks, tagged by phase)
- A go/no-go note at the end of each phase, based on that phase's Exit Criteria (Section 6)
