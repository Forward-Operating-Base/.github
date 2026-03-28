# FOB Command Center

**Operational command platform for the Forward Operating Base program**

Last Updated: 2026-03-28 | Phase 3 Complete | Production Live

---

## What This Is

FOB Command Center is the operational backbone for the Forward Operating Base (FOB) program -- a 12-week workforce development initiative serving veterans and at-risk youth (ages 15-18) through a dual-population mentorship model. The platform provides program management, compliance tracking, grant lifecycle management, AI-powered reporting, and the Buddy Backup crisis intervention system.

FOB is built and maintained by **True North Data Strategies LLC**, an SBA-certified VOSB/SDVOSB based in Colorado Springs, CO. The platform is currently structured as a for-profit operation with a planned nonprofit transition.

## Who It Serves

The FOB program operates across four pillars, each supported by dedicated modules and tooling:

- **Leadership and Life Skills** -- Cohort-based mentorship, lesson delivery, and participant tracking
- **Business and Career Readiness** -- SMB employer pipeline, apprenticeship placement, and workforce outcomes
- **Crisis Response and Wellness** -- Buddy Backup Button emergency system, status escalation, and wellness monitoring
- **Operations and Governance** -- Grant compliance, board reporting, document management, and constitutional AI controls

Participants include transitioning veterans, at-risk youth (15-18), volunteer mentors, SMB employer partners, and grant-funded program administrators.

## Tech Stack

**Frontend:** Next.js 14 (React 18), TypeScript, Tailwind CSS, Firebase Auth (RBAC), deployed on Vercel

**Backend:** 14 FastAPI microservices (Python 3.11+), each containerized and deployed to Google Cloud Run

**Data:** Firebase Firestore (primary store), Google Cloud KMS (field-level encryption with separate youth PII keys)

**LLM Layer:** 5 agent services (Anthropic production, Ollama local dev) with constitutional governance, fail-closed execution gates, prompt anonymization, and token budget enforcement

**Infrastructure:** Turbo monorepo, npm workspaces, GitHub Actions CI/CD, Docker Compose for local dev, Firebase emulators

## Architecture

```
FOB-Revamp-ALYP/                    Workspace root (planning + program docs)
├── fob-command-center/              Implementation repo (GitHub source of truth)
│   ├── packages/
│   │   ├── core/                    Shared libs: auth, db, ui, api-client, compliance-utils, module-registry
│   │   ├── dashboard/               Next.js 14 frontend (Vercel)
│   │   └── modules/                 FastAPI microservices (Cloud Run)
│   │       ├── mod-cohort           12-week cycle tracking, enrollment, sessions
│   │       ├── mod-mentor           Roster, assignments, availability, session logs
│   │       ├── mod-participant      Youth/veteran profiles, COPPA/FERPA gates, encrypted guardian IDs
│   │       ├── mod-partner          SMB employer pipeline, apprenticeship slots, placements
│   │       ├── mod-grant            Grant applications, compliance, deliverables, budgets
│   │       ├── mod-vault            Secure document storage, version tracking, access logging
│   │       ├── mod-buddy-status     Buddy Backup operational status board
│   │       ├── mod-outcomes         Completion rates, placements, anonymized risk metrics
│   │       ├── mod-board            Auto-generated board narratives and outcome snapshots
│   │       ├── mod-calendar         Event scheduling, cohort calendar (Phase 4)
│   │       ├── mod-comms            Communication logging, email/SMS audit trail (Phase 4)
│   │       ├── mod-compliance-command   Federal compliance package generation (TNDS wrapper)
│   │       ├── mod-contract-tracker     Contract lifecycle management (TNDS wrapper)
│   │       ├── mod-proposal-generator-command   Proposal drafting (TNDS wrapper)
│   │       └── mod-onboarding-bot       Workspace provisioning (TNDS wrapper)
│   ├── llm-agents/                  LLM agent services (Cloud Run)
│   │   ├── agent-base               RAG-powered knowledge queries, multi-provider
│   │   ├── agent-compliance         Policy validation, compliance reasoning
│   │   ├── agent-reporting          Board narrative and outcomes report generation
│   │   ├── agent-grant-writer       Grant narrative drafting with budgets/deliverables
│   │   ├── agent-intake             Smart intake forms and routing recommendations
│   │   └── governance/              Constitutional rules, execution gates, validation contracts
│   ├── infra/                       Module registry, Firestore rules, indexes
│   └── docs/                        Architecture decisions, compliance mapping, runbooks
├── 00_FOB/                          Authoritative program and pillar documentation
│   ├── FOB-Pillars/                 4 revamped pillar packages
│   └── FOB-Tools/                   14 pillar-aligned tool packages
├── llm/                             LLM source inputs and knowledge assets
├── compliance-gov-module/           Phase 3 agent-compliance source package
├── compliance-content/              Federal compliance corpus and templates
└── compliane-security/              Constitutional governance, CMMC/govcon source
```

## Modules by Phase

### Phase 1 -- Foundation (Complete)

- **mod-cohort** -- 12-week cohort lifecycle: enrollment, sessions, completion tracking
- **mod-mentor** -- Mentor roster, participant assignments, availability, session logging

### Phase 2 -- Core Operations (Complete)

- **mod-participant** -- Youth and veteran profiles with COPPA/FERPA consent gates, encrypted guardian IDs, age bracket validation
- **mod-partner** -- SMB employer pipeline, apprenticeship opportunity tracking, placement management
- **mod-grant** -- Grant application lifecycle, compliance checklists, deliverable tracking, budget management
- **mod-vault** -- Secure document storage with classification, retention policies, version tracking, and access audit logging

### Phase 3 -- Intelligence Layer (Complete)

- **mod-buddy-status** -- Operational status board for the Buddy Backup crisis intervention system
- **mod-outcomes** -- Metrics dashboard: completion rates, placement percentages, anonymized risk metrics
- **mod-board** -- Auto-generated board-ready narratives and outcome snapshots
- **agent-base** -- RAG-powered knowledge queries with multi-provider LLM support
- **agent-compliance** -- Constitutional compliance reasoning with EAG enforcement
- **agent-reporting** -- Board narrative generation with anonymization and token limits
- **agent-grant-writer** -- Grant narrative drafting with fail-closed action gates
- **agent-intake** -- Smart intake analysis with PII redaction and routing recommendations

### Phase 4 -- Hardening (In Progress)

- **mod-calendar** -- Event scheduling, cohort calendar, session planning
- **mod-comms** -- Communication logging, email/SMS audit trail
- Health monitor service
- Export pipeline service
- Production regression sweep after each deploy

## LLM Governance

All five agent services operate under constitutional governance controls:

**Execution Authority Gate (EAG):** Fail-closed logic -- any missing scope, provenance, or entitlement results in DENY. No agent action executes without passing the gate.

**Prompt Anonymization:** All PII is stripped from prompts before any LLM call. Field-level encryption via Google Cloud KMS with separate key rings for general data and youth PII.

**Token Budgets:** Per-agent enforcement with 80% alert threshold and 100% hard stop. No unbounded generation.

**Audit Trail:** All agent actions are logged before execution. Audit logs are immutable (create-only, no update or delete).

**Multi-Provider Support:** Anthropic (production default), Ollama (local development). Provider routing is runtime-configurable.

## Data Protection

Youth participants (ages 15-18) receive layered protections:

- **COPPA/FERPA compliance** enforced at the Firestore rules level
- **Consent tracking** with guardian verification (paper, digital, or verbal methods)
- **Age bracket validation** (under_13, 13_15, 16_18, adult) gates data access
- **Read/write blocks** on minor records without `consent_status: granted` and valid guardian reference
- **Field-level encryption** via Google Cloud KMS with dedicated youth PII key ring
- **Guardian IDs** encrypted separately from general participant data

## RBAC Roles

Firestore security rules enforce role-based access:

- `super_admin` -- Full platform access
- `program_manager` -- Cohort, mentor, participant, and grant management
- `mentor` -- Assigned participant access and session logging
- `auditor` -- Read-only access to audit logs and compliance records
- `grant_reviewer` -- Grant application review and scoring

## Current Status

**Production:** Vercel frontend live, Cloud Run endpoints active for all Phase 1-3 modules

**SOC 2 alignment:** Constitutional governance controls, immutable audit logging, encrypted PII, RBAC enforcement

**Known blockers:** Missing upstream Apps Script webapp URL and API key environment values for `mod-onboarding-bot` and `mod-contract-tracker` wrapper modules in production Cloud Run

**Branch protection:** PR-only merge workflow on main, CI/CD via GitHub Actions

## Local Development

```bash
# Prerequisites: Node 18+, Python 3.11+, Firebase CLI, Docker

# Clone and install
git clone <repo-url> fob-command-center
cd fob-command-center
npm install

# Start Firebase emulators
firebase emulators:start

# Start module services (each on its own port, 8001+)
cd packages/modules/mod-cohort && uvicorn app.api:app --port 8001
# Repeat for each module as needed

# Start dashboard
npm run dev
```

Refer to `docs/runbooks/developer-runbook.md` for full local setup, staging deployment, and production configuration.

## Key Documentation

| Document | Location | Purpose |
|---|---|---|
| Phase checklist | `FOB_TODO.md` | Current execution state and next priorities |
| Continuation prompt | `FOB_CONTINUATION_PROMPT.md` | Session context and architecture decisions |
| Module map | `FOB_UNIFIED_MODULE_MAP.md` | Complete module and asset mapping |
| Governance principles | `docs/architecture/governance-principles.md` | Constitutional control baseline |
| LLM governance | `docs/architecture/llm-governance-integration.md` | EAG architecture and enforcement |
| Compliance mapping | `docs/compliance/README.md` | COPPA/FERPA/SOC2 mapping |
| Developer runbook | `docs/runbooks/developer-runbook.md` | Local setup, deploy, and troubleshooting |
| Security baseline | `docs/runbooks/github-security-baseline.md` | Branch protection and CI hardening |

## Business Context

FOB Command Center is one of three operations under **True North Data Strategies LLC**:

- **TNDS Core** -- Data operations consulting for small businesses (Command Center builds, Battle Rhythm installs, Command Partner retainers)
- **Pipeline Punks** -- Education and publishing for system design and automation; also home to Fleet-Compliance Sentinel SaaS
- **Forward Operating Base** -- Veteran and youth workforce development with the Buddy Backup crisis intervention system

The FOB program is designed by Jacob Johnston, a 20-year Army veteran (Airborne Infantry), service-disabled, Bronze Star recipient.

---

True North Data Strategies LLC | Colorado Springs, CO | SBA-Certified VOSB/SDVOSB

Jacob Johnston | 719-204-6365 | jacob@truenorthstrategyops.com
