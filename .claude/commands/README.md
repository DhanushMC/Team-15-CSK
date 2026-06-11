# Claude Code Skills — PreMortem + StandupSync

Two composable skills for AI-Powered Predictive DevOps Governance at DEWA.

---

## Skills

### `/standup` — StandupSync

Captures structured tasks from every department's daily standup, stores them in Microsoft Excel, and enforces them as GitHub PR checklists.

```
/standup departments               List all 10 departments and schemas
/standup standup devops [text]     Extract tasks from a transcript
/standup checklist devops sprint1  Build a PR checklist from pending tasks
/standup generate excel_sync       Write backend/storage/excel_sync.py
```

Canonical skill: [.claude/skills/standup-sync/SKILL.md](../skills/standup-sync/SKILL.md)

---

### `/premortem` — PreMortem

Predicts infrastructure risk before deployment, generates AI remediations, enforces human approval via Microsoft Teams, and deploys fixes through ArgoCD.

**Requires:** `/standup` (standup-intelligence ≥ 1.0.0)

```
/premortem risk categories         List the 5 DevOps risk categories
/premortem build order             Return the 4-phase build sequence
/premortem generate risk_predictor Write backend/ai/risk_predictor.py
/premortem explain layer 6         Explain the Teams Approval Workflow
```

Canonical skill: [.claude/skills/premortem/SKILL.md](../skills/premortem/SKILL.md)

---

## Dependency Graph

```
standup-sync  →  provides: standup-intelligence
                           excel-task-registry
                           pr-checklist-gate

premortem     →  requires: standup-intelligence
              →  provides: risk-predictor
                           teams-approval-workflow
                           pr-correlator
                           audit-trail
```

---

## File Locations

```
.claude/
├── commands/
│   ├── standup.md       /standup entry point
│   ├── premortem.md     /premortem entry point
│   └── README.md        this file
└── skills/
    ├── standup-sync/
    │   └── SKILL.md     canonical StandupSync skill definition
    └── premortem/
        └── SKILL.md     canonical PreMortem skill definition
```

Scope: **project-level** — available only within this repository.
To make a skill global, copy its command file to `~/.claude/commands/`.

---

## Safety Principle

> **AI NEVER deploys directly.**
> AI Suggests → Human Approves → GitOps Deploys
