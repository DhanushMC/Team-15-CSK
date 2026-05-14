# PreMortem + StandupSync

**AI-Powered Predictive DevOps Governance with Cross-Department Standup Intelligence**

> Built for DEWA · Hackathon 2026 · Team 15 CSK

---

## The Problem

Modern government IT teams are **reactive**. Tasks discussed in daily standups end up in scattered notes. Pull requests get merged without anyone knowing which standup item they cover. Production incidents are discovered after deployment, not before.

## The Solution

```
Daily Standup (Teams) → Claude Extracts Tasks → Microsoft Excel Registry
        ↓
Tasks become PR Checklists → Risk Analysis → Human Approval → GitOps Deploy
```

**PreMortem + StandupSync** bridges engineering intent and production governance:

- Every department's standup is automatically captured and structured by Claude AI
- Those tasks become a live checklist on every GitHub PR — full traceability from conversation to code
- Infrastructure risk is predicted **before** deployment with confidence scores
- Human approval governs every AI-generated remediation via Microsoft Teams

---

## Core Innovation

| Traditional | PreMortem + StandupSync |
|---|---|
| Meeting notes → Lost → Undocumented PR | Standup Intent → Structured Tasks → PR Checklist |
| Deploy → Incident → Post-mortem | Risk Predicted → Human Approved → Safe Deploy |
| Per-team silos | 10 departments, one unified governance platform |

---

## Supported Departments

| Department | Sheet Tab | Key Schema Fields |
|---|---|---|
| DevOps | `devops` | engineer, service, risk_category, environment |
| AI Deployment | `ai-deployment` | engineer, model_name, service, risk_category |
| Testing / QA | `testing` | tester, test_type, component, priority |
| INTF (Integration) | `intf` | engineer, integration_point, system_a, system_b |
| AI Adoption | `ai-adoption` | owner, use_case, business_unit, adoption_stage |
| Finance | `finance` | owner, process, system, impact_area, deadline |
| Security | `security` | engineer, control_domain, asset, severity, compliance_tag |
| Data Governance | `data-governance` | owner, dataset, domain, governance_tier, regulation |
| SAP | `sap` | consultant, module, transaction_code, change_type |
| Business | `business` | owner, initiative, stakeholder, priority, kpi |

---

## Architecture — 8 Layers

| Layer | Name | Responsibility |
|---|---|---|
| 1 | Standup Intelligence | MS Graph API → Teams transcript → Claude extracts department tasks |
| 2 | Microsoft Excel Registry | Tasks upserted to Excel workbook (one worksheet per department) via MS Graph |
| 3 | PR Checklist Gate | Pending tasks injected into PR body; GitHub Actions posts visibility comment |
| 4 | Infrastructure Risk Intelligence | Infra diffs + Prometheus metrics → risk JSON with confidence score |
| 5 | AI Remediation Engine | Risk data → safer config diff with rollback strategy |
| 6 | Teams Approval Workflow | Adaptive Card → human approve / reject |
| 7 | Remediation PR Generation | AI-authored GitHub PR with standup task traceability |
| 8 | GitOps Deployment | ArgoCD → Kubernetes rollout → task marked Done in Excel |

---

## `/premortem` — Claude Code Skill

This repo includes a Claude Code skill (`.claude/commands/premortem.md`) that loads the full platform architecture as context. Invoke it with:

```
/premortem <request>
```

### Commands

| Command | Example | Output |
|---|---|---|
| Generate a module | `/premortem generate risk_predictor` | Complete `backend/ai/risk_predictor.py` |
| Extract standup tasks | `/premortem standup security <transcript>` | Security tasks as Excel-ready JSON |
| Build PR checklist | `/premortem checklist devops 2026-05-14` | GitHub PR checklist markdown |
| Sync Excel | `/premortem sheets sync finance` | Complete `backend/storage/excel_sync.py` |
| Get department schema | `/premortem department schema sap` | Full SAP column schema |
| List all departments | `/premortem departments` | All 10 department schemas |
| Explain a layer | `/premortem explain Layer 4` | Infrastructure Risk Intelligence |
| Write Claude prompt | `/premortem write prompt for standup extractor` | System prompt + JSON schema |
| Get build order | `/premortem build order` | 4-phase build sequence |

---

## Key Modules

```
backend/
├── main.py                          FastAPI entry point
├── models/
│   ├── task.py                      Pydantic task schema (all departments)
│   └── departments.py               Schema definitions for all 10 departments
├── storage/
│   ├── excel_sync.py                Microsoft Excel upsert (MS Graph API)
│   └── audit_log.py                 SQLite governance event log
├── ai/
│   ├── standup_extractor.py         Transcript → department-structured tasks
│   ├── pr_checklist_builder.py      Pending tasks → PR checklist markdown
│   ├── pr_correlator.py             PR diff ↔ standup tasks (MATCH/PARTIAL/UNKNOWN)
│   ├── risk_predictor.py            Infra diff → risk JSON with confidence score
│   ├── remediator.py                Risk → safer config diff
│   └── pr_generator.py              AI-generated remediation PR description
├── integrations/
│   ├── graph_client.py              Microsoft Graph API (Teams + Excel)
│   ├── teams_card.py                Adaptive Card builder
│   ├── teams_sender.py              Teams webhook sender
│   └── github_client.py             Branch + PR creator
└── routes/
    ├── standup.py                   POST /standup
    ├── pr_checklist.py              POST /pr-checklist
    ├── pr_webhook.py                POST /analyze-pr
    └── teams_webhook.py             POST /webhook/teams

.github/
└── workflows/
    └── standup-checklist.yml        PR checklist visibility notification

.claude/
└── commands/
    └── premortem.md                 Claude Code skill (this repo)
```

---

## Technical Stack

| Component | Technology |
|---|---|
| AI Engine | Claude API (`claude-sonnet-4-6` / `claude-opus-4-7`) |
| Meeting Integration | Microsoft Graph API (Teams transcripts) |
| Task Registry | Microsoft Excel (SharePoint / OneDrive via MS Graph) |
| Notifications | Microsoft Teams Adaptive Cards |
| Source Control | GitHub + GitHub Actions |
| GitOps | ArgoCD |
| Infrastructure | Kubernetes |
| Metrics | Prometheus |
| Backend | FastAPI (Python ≥ 3.11) |
| Audit Log | SQLite (`audit_log.db`) |

---

## PR Checklist Gate

When a developer opens a PR, pending standup tasks are injected as a checklist in the PR body:

```markdown
## Standup Task Checklist
> Auto-generated from daily standup — check items as you complete them.

### Security Tasks · 2026-05-14
- [ ] `SEC-001` Rotate service account credentials — *Aisha* (IAM)
- [ ] `SEC-002` Patch CVE-2026-1234 on API gateway — *Omar*

### DevOps Tasks · 2026-05-14
- [ ] `DEVOPS-001` Increase memory limit for checkout-service — *John*
```

GitHub Actions posts a visibility comment showing pending vs. completed counts. **Merge is never blocked** — the goal is awareness and traceability, not friction.

---

## Safety Principle

> AI NEVER deploys directly.
> **AI Suggests → Human Approves → GitOps Deploys**

Every AI-generated remediation goes through a Microsoft Teams Adaptive Card before a single line is committed. Every task is traceable from standup discussion → Excel registry → GitHub PR → production deployment.

---

## User Manual

A full 20-page A4 print-ready user manual is included in this repo:

- [`PreMortem StandupSync - User Manual.html`](PreMortem%20StandupSync%20-%20User%20Manual.html) — browser viewer
- [`PreMortem StandupSync - User Manual-print.html`](PreMortem%20StandupSync%20-%20User%20Manual-print.html) — auto-prints on open

---

## Environment Variables

```env
# Microsoft Graph API (Teams + Excel — single Azure AD app)
GRAPH_TENANT_ID=
GRAPH_CLIENT_ID=
GRAPH_CLIENT_SECRET=
TEAMS_MEETING_IDS={"devops":"AAMkAG...","security":"AAMkAH..."}

# Microsoft Excel (SharePoint / OneDrive)
EXCEL_WORKBOOK_ID=          # SharePoint/OneDrive item ID
EXCEL_DRIVE_ID=             # SharePoint drive ID

# GitHub
GITHUB_TOKEN=
GITHUB_REPO=your-org/your-repo

# Claude AI
ANTHROPIC_API_KEY=
CLAUDE_MODEL=claude-sonnet-4-6

# Teams notifications
TEAMS_WEBHOOK_URL=
```
