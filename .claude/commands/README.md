# /premortem — Claude Code Skill

A custom Claude Code skill that loads the full **PreMortem** system architecture as context, enabling instant code generation, architecture guidance, and Claude API prompt writing for every layer of the platform.

---

## What is PreMortem?

PreMortem is an AI-powered predictive DevOps governance platform that connects engineering standup discussions, GitHub pull requests, and infrastructure intelligence to prevent production failures **before** deployment.

```
Intent → Code → Prediction → Prevention → Safe Deployment
```

---

## Installation

This skill is already active if you are inside the `Hackathon` project. No installation needed.

To use it in another project, copy the file:

```
.claude/commands/premortem.md  →  <your-project>/.claude/commands/premortem.md
```

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
| Generate a module | `/premortem generate risk_predictor` | Complete `backend/ai/risk_predictor.py` with imports, types, error handling |
| Explain a layer | `/premortem explain Layer 4` | Concise explanation of Infrastructure Risk Intelligence |
| Write a Claude prompt | `/premortem write prompt for standup extractor` | System prompt + user message + JSON output schema |
| Get build order | `/premortem build order` | 4-phase task sequence |
| List risk categories | `/premortem risk categories` | All 5 valid risk category values |

---

## Architecture Layers Covered

| Layer | Name | Responsibility |
|---|---|---|
| 1 | Standup Intelligence | Microsoft Graph API → Claude transcript extraction |
| 2 | Task Governance | Excel/SQLite task tracking and traceability |
| 3 | PR Intelligence | GitHub webhook → Claude PR ↔ task correlation |
| 4 | Infrastructure Risk | Infra diffs + metrics → risk prediction |
| 5 | AI Remediation Engine | Risk data → safer config generation |
| 6 | Teams Approval Workflow | Adaptive Card → human approve/reject |
| 7 | Remediation PR Generation | AI-authored GitHub PR with full evidence |
| 8 | GitOps Deployment | ArgoCD → Kubernetes safe release |

---

## Key Modules the Skill Can Generate

```
backend/
├── main.py                          FastAPI entry point
├── models/
│   └── task.py                      Pydantic task schema
├── storage/
│   ├── task_store.py                Excel/SQLite CRUD
│   └── audit_log.py                 Governance event log
├── ai/
│   ├── standup_extractor.py         Transcript → structured tasks
│   ├── pr_correlator.py             PR diff ↔ standup task match
│   ├── risk_predictor.py            Infra diff → risk JSON
│   ├── remediator.py                Risk → safer config
│   └── pr_generator.py              AI-generated PR description
├── integrations/
│   ├── graph_client.py              Microsoft Graph API client
│   ├── teams_card.py                Adaptive Card builder
│   ├── teams_sender.py              Teams webhook sender
│   └── github_client.py             Branch + PR creator
└── routes/
    ├── tasks.py                     GET/POST/PATCH /tasks
    ├── pr_webhook.py                POST /analyze-pr
    └── teams_webhook.py             POST /webhook/teams
```

---

## Technical Stack

| Layer | Technology |
|---|---|
| AI Engine | Claude API (`claude-sonnet-4-6` / `claude-opus-4-7`) |
| Meeting Integration | Microsoft Graph API |
| Notifications | Microsoft Teams Adaptive Cards |
| Source Control | GitHub + GitHub Actions |
| GitOps | ArgoCD |
| Infrastructure | Kubernetes |
| Metrics | Prometheus |
| Backend | FastAPI (Python) |
| Task Storage | Excel / SQLite |

---

## Safety Principle

> AI NEVER deploys directly.
> **AI Suggests → Human Approves → GitOps Deploys**

Every remediation goes through a Microsoft Teams approval card before any code is committed. This keeps deployments safe, auditable, and reversible.

---

## File Location

```
.claude/
└── commands/
    └── premortem.md    ←  this skill
```

Scope: **project-level** — available only within this repository.
To make it global (available in all projects), copy it to `~/.claude/commands/`.
