# /premortem — Claude Code Skill

A unified Claude Code skill combining **PreMortem** (AI-Powered Predictive DevOps Governance) and **StandupSync** (cross-department daily standup intelligence, Microsoft Excel task registry, and PR checklist enforcement).

---

## What is PreMortem + StandupSync?

**PreMortem** predicts and prevents production failures before deployment by connecting standup intent → code → risk analysis → human-approved remediation.

**StandupSync** captures structured tasks from every department's daily standup, writes them to Microsoft Excel, and turns them into mandatory PR checklists — so no undocumented change can reach production.

```
Standup Intent → Structured Tasks → Microsoft Excel → PR Checklist → Risk Gate → Safe Deployment
```

---

## Why This Skill

Building this unified platform spans 8 architecture layers, 15+ modules, 10 department schemas, Microsoft Excel integration, GitHub Actions enforcement, and a strict AI safety contract. Without a shared reference, every Claude Code session starts cold.

This skill embeds the complete architecture the moment you invoke it — so you jump straight to building.

---

## When to Use

| Situation | Use `/premortem` |
|---|---|
| Starting a new coding session on any layer | Yes — loads full context instantly |
| Generating a module (standup extractor, sheets sync, PR checklist) | Yes — produces code at the correct file path |
| Building for a specific department schema | Yes — returns exact column schema |
| Writing a Claude API prompt for any AI module | Yes — outputs system prompt + JSON schema |
| Unsure which layer handles a responsibility | Yes — explains scope boundaries |
| Reviewing the build sequence before a sprint | Yes — returns the 4-phase task order |
| Working on unrelated code outside this platform | No — skill context is PreMortem-specific |

---

## Installation

```
.claude/commands/premortem.md  →  <your-project>/.claude/commands/premortem.md
```

Available immediately as `/premortem`. No configuration needed.

---

## Usage

```
/premortem <request>
```

Invoke with no arguments to see all capabilities:

```
/premortem
```

---

## What You Can Ask

| Command | Example | Output |
|---|---|---|
| Generate a module | `/premortem generate gsheets_sync` | Complete `backend/storage/gsheets_sync.py` |
| Generate a module | `/premortem generate standup_extractor` | Complete `backend/ai/standup_extractor.py` |
| Generate a module | `/premortem generate pr_checklist_builder` | Complete `backend/ai/pr_checklist_builder.py` |
| Generate actions | `/premortem generate checklist action` | `.github/workflows/standup-checklist.yml` |
| Get department schema | `/premortem department schema security` | Security task columns + severity values |
| Get department schema | `/premortem department schema sap` | SAP columns + module values |
| List all departments | `/premortem departments` | All 10 department schemas |
| Process a standup | `/premortem standup finance [transcript]` | Structured Finance tasks as JSON |
| Build PR checklist | `/premortem checklist devops 2024-05-14` | Markdown checklist for DevOps tasks |
| Generate sheets sync | `/premortem sheets sync testing` | Sheets upsert code for Testing/QA |
| Explain a layer | `/premortem explain Layer 3` | PR Checklist Gate explanation |
| Write a Claude prompt | `/premortem write prompt for standup extractor` | System prompt + user message + JSON schema |
| Get build order | `/premortem build order` | 4-phase task sequence |
| List risk categories | `/premortem risk categories` | All 5 valid DevOps risk category values |

---

## Architecture Layers

| Layer | Name | Responsibility |
|---|---|---|
| 1 | Standup Intelligence | Microsoft Graph API → Claude transcript extraction → department-specific tasks |
| 2 | Microsoft Excel Registry | Upsert tasks to Sheets (one tab per department), live status tracking |
| 3 | PR Checklist Gate | Pending tasks → GitHub PR checklist → Actions posts visibility comment (merge never blocked) |
| 4 | Infrastructure Risk | Infra diffs + metrics → risk prediction |
| 5 | AI Remediation Engine | Risk data → safer config generation |
| 6 | Teams Approval Workflow | Adaptive Card → human approve/reject |
| 7 | Remediation PR Generation | AI-authored GitHub PR with standup task traceability |
| 8 | GitOps Deployment | ArgoCD → Kubernetes safe release → task marked Done in Excel |

---

## Supported Departments

| Department | Microsoft Excel Tab | Key Schema Fields |
|---|---|---|
| DevOps | `devops` | engineer, service, task, risk_category, environment |
| AI Deployment | `ai-deployment` | engineer, model_name, service, task, risk_category, environment |
| Testing / QA | `testing` | tester, test_type, component, task, priority, target_sprint |
| INTF (Integration) | `intf` | engineer, integration_point, system_a, system_b, task |
| AI Adoption | `ai-adoption` | owner, use_case, business_unit, task, adoption_stage, kpi |
| Finance | `finance` | owner, process, system, task, impact_area, deadline |
| Security | `security` | engineer, control_domain, asset, task, severity, compliance_tag |
| Data Governance | `data-governance` | owner, dataset, domain, task, governance_tier, regulation |
| SAP | `sap` | consultant, module, transaction_code, task, change_type, go_live_date |
| Business | `business` | owner, initiative, stakeholder, task, priority, kpi |

---

## Key Modules the Skill Can Generate

```
backend/
├── main.py                              FastAPI entry point
├── models/
│   ├── task.py                          Pydantic task schema (all departments)
│   └── departments.py                   Department registry + schema definitions
├── storage/
│   ├── excel_sync.py                    Microsoft Excel upsert (MS Graph API)
│   └── audit_log.py                     SQLite governance event log
├── ai/
│   ├── standup_extractor.py             Claude: transcript → department tasks
│   ├── pr_checklist_builder.py          Claude: pending tasks → PR checklist markdown
│   ├── pr_correlator.py                 Claude: PR diff ↔ task correlation
│   ├── risk_predictor.py                Claude: infra diff → risk JSON
│   ├── remediator.py                    Claude: risk → safer config
│   └── pr_generator.py                  Claude: generate AI PR description
├── integrations/
│   ├── graph_client.py                  Microsoft Graph API (Teams transcripts)
│   ├── teams_card.py                    Adaptive Card builder
│   ├── teams_sender.py                  Teams webhook sender
│   └── github_client.py                 Branch + PR creator + PR body updater
└── routes/
    ├── standup.py                        POST /standup (process transcript by department)
    ├── pr_checklist.py                   POST /pr-checklist (inject checklist into PR)
    ├── pr_webhook.py                     POST /analyze-pr (GitHub webhook)
    └── teams_webhook.py                  POST /webhook/teams (Approve/Reject)

.github/
└── workflows/
    └── standup-checklist.yml            Block merge on unchecked standup tasks
```

---

## Technical Stack

| Component | Technology |
|---|---|
| AI Engine | Claude API (`claude-sonnet-4-6` / `claude-opus-4-7`) |
| Meeting Integration | Microsoft Graph API |
| Collaboration | Microsoft Teams |
| Notifications | Teams Adaptive Cards |
| Task Registry | Microsoft Excel (SharePoint / OneDrive via MS Graph API) |
| PR Enforcement | GitHub Actions status check |
| Source Control | GitHub |
| GitOps | ArgoCD |
| Infrastructure | Kubernetes |
| Metrics | Prometheus |
| Backend | FastAPI (Python) |
| Audit Storage | SQLite |

---

## Safety Principle

> AI NEVER deploys directly.
> **AI Suggests → Human Approves → GitOps Deploys**

Every remediation goes through a Microsoft Teams approval card before any code is committed. Every task extracted from standup is traceable to a PR and a deployment — across every department.

---

## File Location

```
.claude/
└── commands/
    └── premortem.md    ←  this skill
```

Scope: **project-level** — available only within this repository.
To make it global, copy to `~/.claude/commands/`.
