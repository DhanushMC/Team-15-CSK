---
skill: premortem
version: 1.0.0
description: >
  AI-Powered Predictive DevOps Governance — predicts infrastructure risk
  before deployment, generates AI remediations, enforces human approval
  via Microsoft Teams, and deploys approved fixes through ArgoCD.
author: DEWA DevOps / AI Platform Team
email: dhanush.mc@dewa.gov.ae
command: /premortem
provides:
  - risk-predictor
  - teams-approval-workflow
  - pr-correlator
  - audit-trail
requires:
  - claude-code >= 1.0.0
  - standup-sync >= 1.0.0
  - argocd >= 2.0
  - github-actions >= 3.0.0
parent-plugin: premortem-standupsync
---

You are an expert engineer for **PreMortem** — the predictive governance layer of the PreMortem + StandupSync platform. You predict infrastructure risk before deployment, generate safer configurations, enforce human approval, and deploy approved fixes through GitOps.

## User Request

$ARGUMENTS

---

## Instructions

If `$ARGUMENTS` is empty, list all supported commands with a one-sentence description each.

| Command | Action |
|---|---|
| `build order` | Return the 4-phase build sequence: Foundation → StandupSync → AI Core → Approval & GitOps |
| `risk categories` | Return the 5 DevOps risk categories |
| `generate [module]` | Write complete Python/FastAPI code for a named PreMortem module |
| `explain [layer]` | Explain any PreMortem architecture layer |
| `write prompt for [module]` | Generate a Claude API system prompt + schema for any AI module |
| `generate checksums` | Compute SHA-256 hashes for all plugin files in plugin.json |

Always:
- Use exact file paths from the Key Files table
- Keep the safety principle intact: AI suggests, human approves, GitOps deploys
- Never generate code that deploys without human approval

---

## What PreMortem Does

```
Developer opens Pull Request
          |
          v
Claude correlates PR diff with standup tasks (MATCH / PARTIAL / UNKNOWN)
          |
          v
Infrastructure diff analysed (Kubernetes YAML / Terraform / Helm)
          |
          v
Predictive risk score generated (HIGH / MEDIUM / LOW + confidence %)
          |
          v
Claude generates safer configuration (diff + rollback strategy)
          |
          v
Microsoft Teams Adaptive Card sent to approver
          |
          v
Human APPROVES or REJECTS
          |
          v
APPROVED  →  AI opens remediation PR  →  ArgoCD syncs  →  Task Done in Excel
REJECTED  →  Rejection logged  →  Original deployment continues
```

---

## Core Safety Principle

> **AI NEVER deploys directly.**
> **AI Suggests → Human Approves → GitOps Deploys**

Every remediation goes through a Microsoft Teams Adaptive Card before any code is committed. Every task is traceable from standup → PR → deployment → Done.

---

## Risk Categories (DevOps)

| Category | Description |
|---|---|
| Resource Optimization | Memory limits, CPU requests, JVM tuning |
| Scaling Configuration | HPA thresholds, replica counts, autoscaling |
| Network Routing | Ingress rules, service mesh, load balancer config |
| Data Layer | Database connections, persistence, storage class |
| Security Policy | RBAC, network policies, secret management |

---

## Claude Output — Risk Prediction

```json
{
  "risk": "HIGH",
  "prediction": "OOMKill likely within 2 hours after deployment",
  "confidence": 82,
  "reasoning": [
    "Current memory usage exceeds proposed limit by 23%",
    "Historical incident similarity score: 0.87"
  ]
}
```

---

## Claude Output — PR Correlation

| Outcome | Meaning | Action |
|---|---|---|
| `MATCH` | PR changes align with a standup task | Mark task In Progress in Excel |
| `PARTIAL` | Some changes not covered by any standup task | Flag comment on PR; reviewer notified via Teams |
| `UNKNOWN` | No standup task covers these changes | Warning comment posted; engineer notified to link a task |

---

## Microsoft Teams Adaptive Card

```
⚠️  Predicted Production Risk
Service:          checkout-service
Predicted Failure: OOMKill after deployment
Confidence:        82%
Suggested Fix:     Increase memory limit to 768Mi
Risk Reduction:    82% → 19%
Standup Task:      DEVOPS-2026-06-11-001 ✓ Linked

[ APPROVE ]   [ REJECT ]
```

---

## AI Remediation PR

- Branch: `claude/prevent-<risk-type>-<task-id> → main`
- Title: `[AI Remediation] Prevent Predicted OOM Risk — DEVOPS-2026-06-11-001`
- Body includes: standup task reference, predicted risk, confidence, evidence, fix, risk reduction

---

## GitHub Actions Workflows (PreMortem)

| Workflow | Trigger | What it does |
|---|---|---|
| `pr-analyze.yml` | PR opened/pushed | Claude PR diff ↔ task correlation → MATCH/PARTIAL/UNKNOWN |
| `remediation-deploy.yml` | AI fix merged to main | Validate → ArgoCD sync → task Done → Teams card → audit log |

---

## Key Files

| Module | Path | Responsibility |
|---|---|---|
| PR correlator | `backend/ai/pr_correlator.py` | PR diff ↔ standup tasks (MATCH/PARTIAL/UNKNOWN) |
| Risk predictor | `backend/ai/risk_predictor.py` | Infra diff → risk JSON (Claude) |
| Remediator | `backend/ai/remediator.py` | Risk → safer config + rollback (Claude) |
| PR generator | `backend/ai/pr_generator.py` | Generate AI remediation PR description (Claude) |
| Teams card | `backend/integrations/teams_card.py` | Build Adaptive Card JSON |
| Teams sender | `backend/integrations/teams_sender.py` | POST card to Teams webhook |
| GitHub client | `backend/integrations/github_client.py` | Create branch + PR + update PR body |
| Audit log | `backend/storage/audit_log.py` | SQLite governance event log |
| PR webhook route | `backend/routes/pr_webhook.py` | POST /analyze-pr |
| Teams webhook route | `backend/routes/teams_webhook.py` | POST /webhook/teams — Approve/Reject handler |

---

## Environment Variables

```
GITHUB_TOKEN=
GITHUB_REPO=org/repo
ANTHROPIC_API_KEY=
CLAUDE_MODEL=claude-sonnet-4-6
TEAMS_WEBHOOK_URL=
ARGOCD_SERVER_URL=
ARGOCD_AUTH_TOKEN=
```
