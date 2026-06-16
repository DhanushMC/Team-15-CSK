# PreMortem + StandupSync — Plugin Overview

> **AI Suggests → Human Approves → GitOps Deploys**
> Version 1.3.0 · DEWA DevOps / AI Platform Team · dhanush.mc@dewa.gov.ae

---

## What is this plugin?

Most engineering teams are **reactive** — they discuss work in daily standups, write it down nowhere structured, open PRs with no context, and discover failures only after a production incident.

**PreMortem + StandupSync** makes teams **predictive**. It listens to every department's daily standup, extracts structured tasks using Claude AI, stores them in Microsoft Excel, enforces them as PR checklists on GitHub, predicts infrastructure risk before deployment, and routes every AI-generated fix through a mandatory human approval step in Microsoft Teams before anything reaches production.

```
Daily Standup (Teams)
        |
        v
Claude extracts tasks  -->  Microsoft Excel (one tab per department)
        |
        v
Developer opens PR  -->  Checklist injected into PR body
        |
        v
Claude correlates PR diff with standup tasks  (MATCH / PARTIAL / UNKNOWN)
        |
        v
Infrastructure risk predicted  -->  AI generates safer config
        |
        v
Teams Adaptive Card  -->  Human APPROVE / REJECT
        |
        v
ArgoCD deploys  -->  Task marked Done in Excel  -->  Audit log written
```

The whole pipeline runs automatically. The only human action required is the approval card in Teams.

---

## The Big Idea

| Old Way | With PreMortem + StandupSync |
|---|---|
| Standup notes get lost | Tasks extracted and stored in Excel automatically |
| PRs have no context | Every PR has a linked standup task checklist |
| Undocumented changes reach prod | PARTIAL/UNKNOWN changes flagged before merge |
| Risk found after deployment | Risk predicted before deployment with confidence score |
| AI changes deployed blindly | Every AI fix requires human approval in Teams |
| No audit trail | Full trace: standup → Excel → PR → deploy → Done |

---

## Plugin Structure

```
premortem-standupsync/
|
+-- CLAUDE.md                               Project context loaded by Claude Code
+-- OVERVIEW.md                             This file
+-- CHANGELOG.md                            Full version history (semver)
+-- plugin.json                             Manifest: version, deps, provides, checksums
+-- README.md                               User-facing documentation
|
+-- .claude/                                CLAUDE CODE LAYER
|   |
|   +-- settings.json                       All hooks defined here
|   |
|   +-- commands/
|   |   +-- standup.md                      /standup slash command (intelligence layer)
|   |   +-- premortem.md                    /premortem slash command (governance layer)
|   |   +-- README.md                       Command usage reference
|   |
|   +-- skills/
|   |   +-- standup-sync/
|   |   |   +-- SKILL.md                    Canonical StandupSync skill
|   |   +-- premortem/
|   |       +-- SKILL.md                    Canonical PreMortem skill
|   |
|   +-- skill-activity.log                  Auto-written: every file edit timestamped (gitignored)
|   +-- plugin-audit.log                    Auto-written: SHA-256 on tracked file changes (gitignored)
|
+-- .github/                                GITHUB ACTIONS LAYER
    +-- workflows/
        +-- standup-checklist.yml           Hook: PR opened/edited
        +-- pr-analyze.yml                  Hook: PR diff correlation
        +-- standup-sync.yml                Hook: Scheduled standup sync
        +-- remediation-deploy.yml          Hook: AI fix merge gate
```

---

## Skills

### `/standup` — StandupSync Skill

**File:** `.claude/commands/standup.md` (entry) → `.claude/skills/standup-sync/SKILL.md` (canonical)

Standup intelligence layer. Invoke with `/standup [command]`.

| Command | What it does |
|---|---|
| `/standup departments` | List all 10 departments and their Excel column schemas |
| `/standup standup [dept] [transcript]` | Extract structured tasks from a transcript using Claude |
| `/standup checklist [dept] [sprint]` | Generate a GitHub PR checklist from pending tasks |
| `/standup sheets sync [dept]` | Write the full `excel_sync.py` module for a department |
| `/standup department schema [dept]` | Return the column schema for a specific department |
| `/standup generate [module]` | Write complete Python/FastAPI code for any StandupSync module |
| `/standup explain [component]` | Explain any StandupSync architecture component |

---

### `/premortem` — PreMortem Skill

**File:** `.claude/commands/premortem.md` (entry) → `.claude/skills/premortem/SKILL.md` (canonical)

**Requires:** `/standup` (standup-intelligence ≥ 1.0.0)

Predictive governance layer. Invoke with `/premortem [command]`.

| Command | What it does |
|---|---|
| `/premortem risk categories` | Return the 5 DevOps risk categories |
| `/premortem build order` | Return the 4-phase build sequence |
| `/premortem generate [module]` | Write complete Python/FastAPI code for any PreMortem module |
| `/premortem explain [layer]` | Explain any PreMortem architecture layer |
| `/premortem write prompt for [module]` | Generate a Claude API system prompt + schema for any AI module |
| `/premortem generate checksums` | Compute SHA-256 hashes for all plugin files in plugin.json |

**Supported Departments (10 total):**

| Department | Excel Tab | Key Schema Fields |
|---|---|---|
| DevOps | `devops` | engineer, service, task, risk_category, environment |
| AI Deployment | `ai-deployment` | engineer, model_name, service, task, risk_category |
| Testing / QA | `testing` | tester, test_type, component, task, priority |
| INTF | `intf` | engineer, integration_point, system_a, system_b |
| AI Adoption | `ai-adoption` | owner, use_case, business_unit, adoption_stage |
| Finance | `finance` | owner, process, system, impact_area, deadline |
| Security | `security` | engineer, control_domain, asset, severity, compliance_tag |
| Data Governance | `data-governance` | owner, dataset, domain, governance_tier, regulation |
| SAP | `sap` | consultant, module, transaction_code, change_type |
| Business | `business` | owner, initiative, stakeholder, priority, kpi |

---

## Hooks

All hooks are defined in `.claude/settings.json`. They fire automatically — no manual invocation needed.

### Hook 1 — `SessionStart`

**When:** Every time a Claude Code session opens in this project.

**What it does:** Prints a welcome message listing available `/premortem` commands so the team always knows what's available.

```
PreMortem + StandupSync v1.2.0 loaded.
Try: /premortem departments | /premortem build order | /premortem generate [module]
```

---

### Hook 2 — `PostToolUse: Write|Edit` → Activity Log

**When:** Every time Claude writes or edits any file.

**What it does:** Appends a timestamped line to `.claude/skill-activity.log`:

```
2026-06-11 09:32:01 | EDIT | backend/ai/standup_extractor.py
2026-06-11 09:32:45 | EDIT | .github/workflows/standup-checklist.yml
```

Gives the team a full audit trail of every file Claude touched during a session.

---

### Hook 3 — `PostToolUse: Write` → Auto-Stage

**When:** Every time Claude writes a new file.

**What it does:** Automatically runs `git add` on any `.py`, `.yml`, `.yaml`, `.md`, or `.json` file Claude generates. Generated code is staged to git immediately — no manual `git add` needed.

---

### Hook 4 — `PostToolUse: Write|Edit` → Plugin Integrity

**When:** Every time one of the four tracked plugin files is written or edited:
- `.claude/settings.json`
- `.claude/skills/premortem-standupsync/SKILL.md`
- `.claude/commands/premortem.md`
- `plugin.json`

**What it does:** Computes the SHA-256 hash of the changed file and writes a tamper-evidence entry to `.claude/plugin-audit.log`:

```
2026-06-11 09:45:12 | INTEGRITY | .claude/skills/.../SKILL.md | SHA256:3cb7ddd8...
```

If anyone edits a skill or hook outside Claude Code, the hash in the audit log will no longer match the checksums in `plugin.json` — drift is immediately detectable.

---

### Hook 5 — `Stop`

**When:** Every time a Claude Code session ends.

**What it does:** Prints a session-ready message with the audit log location so the team knows where to find the activity trail.

```
PreMortem + StandupSync ready.
Run /premortem [command] to continue. Audit log: .claude/plugin-audit.log
```

---

## GitHub Actions Connectors (Plugin Layer)

The GitHub Actions workflows are the automation connectors — they bridge GitHub events to the PreMortem backend, Microsoft Excel, Microsoft Teams, and ArgoCD.

---

### Connector 1 — `standup-checklist.yml`

**File:** `.github/workflows/standup-checklist.yml`

**Purpose:** Every PR gets a live checklist of pending standup tasks. Merge is never blocked — this is visibility only.

**Triggers (pre-hooks):**
- `pull_request` — opened, edited, synchronize, reopened
- `pull_request_review` — submitted (re-checks after reviewer acts)
- `workflow_dispatch` — manual ops re-run

**What it does (step by step):**
1. Calls `POST /pr-checklist` on the PreMortem backend — Claude injects the checklist into the PR body
2. Reads the PR body, counts `- [ ]` (unchecked) and `- [x]` (checked) items
3. Posts or updates a single bot comment on the PR — never spams with duplicates
4. Writes a summary table to the GitHub Actions job summary page

**Post-hooks:**
- Comment updated on every sync push — always shows current task state
- `if: always()` job summary — written even if backend is unreachable

**Output example:**
```
⚠️ PreMortem Standup Reminder — 2 of 4 task(s) still pending.
| ✅ Checked | 2 |
| 🔲 Pending | 2 |
You may always merge — this is awareness, not a gate.
```

---

### Connector 2 — `pr-analyze.yml`

**File:** `.github/workflows/pr-analyze.yml`

**Purpose:** When code is pushed to a PR, Claude reads the diff and determines whether the changes align with standup tasks — returning MATCH, PARTIAL, or UNKNOWN.

**Triggers (pre-hooks):**
- `pull_request` — opened, synchronize, reopened, ready_for_review
- `workflow_dispatch` — manual analysis

**What it does (step by step):**
1. Checks out the repo and extracts the PR diff (changed files + patch)
2. Sends the diff to `POST /analyze-pr` — Claude correlates against pending standup tasks
3. Receives one of three outcomes:
   - `MATCH` — all changes align with a standup task (green)
   - `PARTIAL` — some changes not covered by any standup task (orange)
   - `UNKNOWN` — no standup task covers these changes at all (red)
4. Posts or updates a single correlation comment on the PR
5. If PARTIAL or UNKNOWN — sends a Teams Adaptive Card alerting the reviewer

**Post-hooks:**
- Teams alert card — only fires on PARTIAL or UNKNOWN
- Job summary — always written

**Output example:**
```
⚠️ PreMortem PR Analysis — PARTIAL
| Correlation  | PARTIAL |
| Risk Level   | MEDIUM  |
| Matched Tasks | 1      |
This PR contains changes not covered by any standup task.
A reviewer has been notified via Teams.
```

---

### Connector 3 — `standup-sync.yml`

**File:** `.github/workflows/standup-sync.yml`

**Purpose:** After every daily standup, automatically fetch the Teams meeting transcript, let Claude extract structured tasks, and write them to Microsoft Excel.

**Triggers (pre-hooks):**
- `schedule` — cron `15 5 * * 0-4` (09:15 GST, Sun–Thu, after standup window)
- `push` to `main` on `backend/**`, `k8s/**`, `helm/**` paths
- `workflow_dispatch` — manual run with department selector + optional transcript paste + dry-run toggle

**What it does (step by step):**
1. Calls `POST /standup` with the department name and optional transcript
2. Backend fetches the Teams transcript via Microsoft Graph API (if no transcript pasted)
3. Claude extracts structured tasks using the department-specific schema
4. Tasks are upserted to the department's Excel tab via MS Graph API
5. Sends a Teams summary card with task counts and a link to the Excel sheet

**Post-hooks:**
- Job summary — always written (`if: always()`)
- Teams success card — only if sync succeeded and not a dry run
- Teams failure card — `if: failure()` — ops alerted immediately

**Output example (Teams card):**
```
✅ Standup Sync Complete — devops
Department:      devops
Tasks extracted: 5
Tasks written:   5
[View Excel Sheet]  [View Run]
```

---

### Connector 4 — `remediation-deploy.yml`

**File:** `.github/workflows/remediation-deploy.yml`

**Purpose:** When an AI-generated remediation PR is merged, validate it, trigger ArgoCD, mark the standup task Done in Excel, write an audit log entry, and notify Teams. The gate ensures AI never deploys without a human having approved first.

**Triggers (pre-hooks):**
- `push` to `main` — gate job checks commit message for `AI Remediation` or `claude/prevent-*`
- `workflow_dispatch` — break-glass manual deploy

**What it does (step by step):**
1. **Gate job** — reads the commit message; skips the deploy job entirely if this is not a remediation merge
2. **`environment: production`** — GitHub environment protection rules apply (optional manual approval gate)
3. Triggers `POST /api/v1/applications/{app}/sync` on ArgoCD
4. Calls `PATCH /tasks/{task_id}` on the PreMortem backend — marks the standup task `Done` in Excel
5. Writes a `REMEDIATION_DEPLOYED` event to the PreMortem audit log
6. Sends a green Teams confirmation card with full traceability (task ID, commit SHA, ArgoCD status)

**Post-hooks:**
- Audit log write — always, even if ArgoCD sync failed
- Teams confirmation card — always after audit log
- Job summary — `if: always()`
- Teams failure card — `if: failure()` — manual intervention required

**Output example (Teams card):**
```
🚀 AI Remediation Deployed — DEVOPS-2026-06-11-001
Task ID:     DEVOPS-2026-06-11-001
Department:  devops
ArgoCD App:  checkout-service
Sync Status: ✅ TRIGGERED
Commit:      a3f9c12
Deployed by: GitHub Actions (human-approved)
[View Run]  [View PR]
```

---

## Pre and Post Hooks — Full Reference

The plugin has two hook layers: **Claude Code hooks** (in `settings.json`) and **GitHub Actions hooks** (in each workflow). Together they cover the full lifecycle — from a developer opening Claude Code, to a standup being processed, to a fix being deployed.

---

### Layer 1 — Claude Code Hooks (`settings.json`)

These fire inside the developer's local Claude Code session.

| # | Event | Type | When it fires |
|---|---|---|---|
| 1 | `SessionStart` | Pre-hook | Before the developer types anything — project opens |
| 2 | `PostToolUse: Write\|Edit` | Post-hook | After Claude writes or edits any file |
| 3 | `PostToolUse: Write` | Post-hook | After Claude creates a new file |
| 4 | `PostToolUse: Write\|Edit` | Post-hook | After Claude touches a tracked plugin file |
| 5 | `Stop` | Post-hook | After the session ends |

---

#### Pre-hook — `SessionStart`

**Fires:** When Claude Code opens the project, before any user input.

**Role:** Bootstraps context. Tells the developer what commands are available so they never have to guess.

```
PreMortem + StandupSync v1.2.0 loaded.
Try: /premortem departments | /premortem build order | /premortem generate [module]
```

---

#### Post-hook 1 — Activity Log (`PostToolUse: Write|Edit`)

**Fires:** After every file write or edit Claude makes, across all files.

**Role:** Passive audit trail. Every file Claude touched is timestamped and recorded.

```
2026-06-11 09:32:01 | EDIT | backend/ai/standup_extractor.py
2026-06-11 09:45:20 | EDIT | .github/workflows/standup-checklist.yml
```

Log location: `.claude/skill-activity.log`

---

#### Post-hook 2 — Auto-Stage (`PostToolUse: Write`)

**Fires:** After Claude creates a new file (Write, not Edit).

**Role:** Removes friction. Automatically runs `git add` on any `.py`, `.yml`, `.yaml`, `.md`, or `.json` file Claude generates. The developer never needs to manually stage generated code.

---

#### Post-hook 3 — Plugin Integrity (`PostToolUse: Write|Edit`)

**Fires:** After Claude writes or edits one of the tracked plugin files:
- `.claude/skills/standup-sync/SKILL.md`
- `.claude/skills/premortem/SKILL.md`
- `.claude/commands/standup.md`
- `.claude/commands/premortem.md`
- `.claude/settings.json`
- `plugin.json`

**Role:** Tamper detection. Computes the SHA-256 hash of the changed file and appends a signed entry to the audit log. If anyone edits a skill or hook outside Claude Code, the hash will no longer match the reference checksums in `plugin.json`.

```
2026-06-11 09:45:12 | INTEGRITY | .claude/skills/.../SKILL.md | SHA256:3cb7ddd8...
2026-06-11 09:50:33 | INTEGRITY | .claude/settings.json        | SHA256:db53a6e1...
```

Log location: `.claude/plugin-audit.log`

---

#### Post-hook 4 — `Stop`

**Fires:** When the Claude Code session ends.

**Role:** Closes the loop. Confirms the session is over and points the developer to the audit log.

```
PreMortem + StandupSync ready.
Run /premortem [command] to continue. Audit log: .claude/plugin-audit.log
```

---

### Layer 2 — GitHub Actions Hooks (Workflow-level)

These fire inside GitHub CI/CD, triggered by repository events.

---

#### `standup-checklist.yml` — Pre and Post Hooks

```
EVENT                                ACTION
-------                              ------

PRE  pull_request (open/edit/sync)   Workflow starts
PRE  pull_request_review submitted   Workflow re-runs (reviewer acted)
PRE  workflow_dispatch               Manual ops re-run
PRE  concurrency guard               Cancels stale run on same PR

PRE  Step: backend injection         POST /pr-checklist
                                     Claude injects task list into PR body
                                     exit 0 even on failure (non-blocking)

MAIN Step: count items               Read PR body
                                     Count - [ ] unchecked, - [x] checked

POST Step: upsert comment            Create or update single bot comment
                                     Never posts duplicates (MARKER pattern)

POST Step: job summary               Always written (if: always() implicit)
                                     Table: PR / Total / Checked / Pending
```

---

#### `pr-analyze.yml` — Pre and Post Hooks

```
EVENT                                ACTION
-------                              ------

PRE  pull_request (open/sync/ready)  Workflow starts
PRE  workflow_dispatch               Manual re-analysis
PRE  concurrency guard               Cancels stale run on same PR

PRE  Step: checkout                  git fetch-depth: 0
                                     Full history for accurate diff

PRE  Step: extract diff              git diff --name-only + --stat + --unified
                                     Writes patch to /tmp/diff_patch.txt

MAIN Step: POST /analyze-pr          Sends diff + files to PreMortem backend
                                     Claude returns MATCH / PARTIAL / UNKNOWN
                                     3 retries, exponential backoff
                                     Falls back to UNKNOWN if backend down

POST Step: upsert PR comment         Create or update single correlation comment
                                     Icon: MATCH=green / PARTIAL=orange / UNKNOWN=red

POST Step: Teams alert               ONLY fires if PARTIAL or UNKNOWN
           (if: PARTIAL|UNKNOWN)     Human reviewer notified immediately

POST Step: job summary               Always written
                                     Table: PR / Correlation / Risk / Matched Tasks
```

---

#### `standup-sync.yml` — Pre and Post Hooks

```
EVENT                                ACTION
-------                              ------

PRE  schedule cron 09:15 GST         Fires Sun-Thu after standup window
PRE  push to main (infra paths)      Fires when backend/k8s/helm changes
PRE  workflow_dispatch               Manual: dept picker + transcript + dry-run

PRE  concurrency: in-progress=false  Never cancels a sync mid-write
                                     (data integrity: don't cut an Excel write)

PRE  Step: log context               Print dept / trigger / dry-run / ref
                                     Before any API call

MAIN Step: POST /standup             Send dept + transcript to backend
                                     Claude extracts tasks
                                     Excel upsert via MS Graph API
                                     3 retries, 15s backoff

POST Step: job summary               if: always()
                                     Written even if sync failed
                                     Table: dept / status / extracted / written

POST Step: Teams success card        if: STATUS==SUCCESS && DRY_RUN==false
                                     Task counts + Excel sheet link

POST Step: Teams failure card        if: failure()
                                     Red card — manual intervention required
                                     Tasks NOT written to Excel
```

---

#### `remediation-deploy.yml` — Pre and Post Hooks

```
EVENT                                ACTION
-------                              ------

PRE  push to main                    Workflow starts on every merge
PRE  workflow_dispatch               Break-glass manual deploy

PRE  Job: gate                       Reads commit message
                                     Checks for "AI Remediation" / "claude/prevent-"
                                     Sets is_remediation=true/false
                                     If false: deploy job is SKIPPED entirely

PRE  Job: deploy (needs: gate)       Only runs if gate passes
PRE  environment: production         GitHub environment protection rules
                                     Optional: require a human approver before deploy

MAIN Step: ArgoCD sync               POST /api/v1/applications/{app}/sync
                                     3 retries, 10s backoff
                                     Non-fatal if fails (still closes task + notifies)

MAIN Step: close task                PATCH /tasks/{task_id}
                                     Status → Done in Microsoft Excel
                                     3 retries, 5s backoff

POST Step: audit log entry           Always runs (no if: condition)
                                     Writes REMEDIATION_DEPLOYED event
                                     Includes: task_id / dept / commit SHA / actor

POST Step: Teams confirmation card   Always runs after audit log
                                     Green card with full traceability
                                     ArgoCD status / task ID / commit / run link

POST Step: job summary               if: always()
                                     Written even on failure

POST Step: Teams failure card        if: failure()
                                     Red card — manual intervention required
                                     Task NOT marked Done, ArgoCD may not have synced
```

---

### Complete Hook Execution Timeline

```
Developer opens Claude Code
  |
  +-- PRE (SessionStart)         Print available commands

Developer runs /premortem generate backend/ai/standup_extractor.py
  |
  +-- [Claude writes file]
  +-- POST (Activity Log)        Log: EDIT | standup_extractor.py
  +-- POST (Auto-Stage)          git add standup_extractor.py
  +-- POST (Integrity)           SKIP (not a tracked plugin file)

Developer edits SKILL.md
  |
  +-- [Claude edits file]
  +-- POST (Activity Log)        Log: EDIT | SKILL.md
  +-- POST (Auto-Stage)          git add SKILL.md
  +-- POST (Integrity)           SHA-256 → plugin-audit.log

Developer closes Claude Code
  |
  +-- POST (Stop)                Print session-ready + audit log path

Developer pushes code, opens PR
  |
  +-- PRE  (standup-checklist)   pull_request: opened fires
  +-- PRE  (pr-analyze)          pull_request: opened fires
  |
  +-- PRE  (checklist)           POST /pr-checklist → checklist injected
  +-- POST (checklist)           Bot comment posted on PR
  |
  +-- PRE  (analyze)             git checkout + extract diff
  +-- MAIN (analyze)             POST /analyze-pr → MATCH/PARTIAL/UNKNOWN
  +-- POST (analyze)             Correlation comment posted
  +-- POST (analyze, if PARTIAL) Teams alert card sent

09:15 GST — standup window closes
  |
  +-- PRE  (standup-sync)        schedule cron fires
  +-- MAIN (standup-sync)        POST /standup → tasks → Excel
  +-- POST (standup-sync)        Teams success card + job summary

AI remediation PR merged to main
  |
  +-- PRE  (remediation-deploy)  gate job validates commit message
  +-- PRE  (remediation-deploy)  environment: production gate
  +-- MAIN (remediation-deploy)  ArgoCD sync + task Done in Excel
  +-- POST (remediation-deploy)  Audit log + Teams card + job summary
```

---

## Version Control & Integrity

### plugin.json

The plugin manifest. Records:
- `version` — semantic version (`major.minor.patch`)
- `createdAt` / `updatedAt` — dates
- `dependencies` — what the plugin requires
- `provides` — capabilities exposed to child plugins
- `extends` — parent plugin name (`null` = root plugin)
- `integrity.checksums` — SHA-256 hash of every plugin file at last release

### plugin-audit.log

Auto-written by Hook 4 every time a tracked file changes:

```
2026-06-11 09:45:12 | INTEGRITY | .claude/skills/.../SKILL.md   | SHA256:3cb7ddd8...
2026-06-11 09:50:33 | INTEGRITY | .claude/settings.json          | SHA256:db53a6e1...
```

To verify integrity at any time:

```bash
python -c "
import hashlib
files = [
    '.claude/skills/standup-sync/SKILL.md',
    '.claude/skills/premortem/SKILL.md',
    '.claude/commands/standup.md',
    '.claude/commands/premortem.md',
    '.claude/settings.json',
    'plugin.json'
]
for f in files:
    h = hashlib.sha256(open(f,'rb').read()).hexdigest()
    print('sha256:' + h, f)
"
```

Or from Claude Code: `/premortem generate checksums`

### CHANGELOG.md

Full version history. Every release documents what was added, changed, or fixed.

---

## Dependency Graph

```
                    premortem-standupsync v1.2.0
                    extends: null  (root plugin)

      REQUIRES                              PROVIDES

microsoft-graph-api  ---+              +--- standup-intelligence
microsoft-teams       ---+  THIS       +--- pr-checklist-gate
anthropic-api         ---+  PLUGIN     +--- risk-predictor
argocd >= 2.0         ---+             +--- teams-approval-workflow
claude-code >= 1.0.0  ---+             +--- excel-task-registry
github-actions >= 3.0 ---+             +--- audit-trail


Child plugin example:
  "extends": "premortem-standupsync"
  "consumes": ["standup-intelligence", "pr-checklist-gate"]
```

---

## Required Secrets

Configure these in GitHub → Settings → Secrets and Variables → Actions:

| Secret | Used By |
|---|---|
| `PREMORTEM_BACKEND_URL` | All 4 workflows |
| `PREMORTEM_API_TOKEN` | All 4 workflows |
| `TEAMS_WEBHOOK_URL` | All 4 workflows |
| `ARGOCD_SERVER_URL` | remediation-deploy only |
| `ARGOCD_AUTH_TOKEN` | remediation-deploy only |

Optional repo variable: `PREMORTEM_DEPARTMENT` (defaults to `devops`)

---

## File Formats at a Glance

| Layer | File | Format |
|---|---|---|
| Skill (StandupSync) | `.claude/skills/standup-sync/SKILL.md` | Markdown |
| Skill (PreMortem) | `.claude/skills/premortem/SKILL.md` | Markdown |
| Command entry | `.claude/commands/standup.md`, `.claude/commands/premortem.md` | Markdown |
| Hooks | `.claude/settings.json` | JSON |
| Manifest | `plugin.json` | JSON |
| Connectors | `.github/workflows/*.yml` | YAML |
| Context | `CLAUDE.md` | Markdown |
| Versions | `CHANGELOG.md` | Markdown |

---

## Testing Guide

### Tier 1 — Instant (no backend, no secrets)

Open Claude Code in this repo. These commands work immediately:

```
/standup departments
```
Lists all 10 supported departments and their schemas. No API calls.

```
/standup standup devops Ahmed: we need to reduce memory on checkout-service, it keeps OOMKilling. Sara: updating autoscaling for payment-api, thresholds are too aggressive.
```
Claude extracts structured tasks from the inline transcript in real time.

```
/standup checklist devops 2026-06-16
```
Generates a formatted GitHub PR checklist from DevOps pending tasks.

```
/premortem risk categories
```
Returns the 5 DevOps risk categories with descriptions.

```
/premortem build order
```
Returns the 4-phase platform build sequence.

```
/premortem generate risk_predictor
```
Writes `backend/ai/risk_predictor.py` — complete FastAPI module.

```
/premortem write prompt for standup extractor
```
Generates a production Claude API system prompt + JSON output schema.

---

### Tier 2 — GitHub Actions (demo mode, no secrets needed)

All 4 workflows exit cleanly when `PREMORTEM_BACKEND_URL` is not set.
They skip the backend call with a `notice` annotation and show green.

**Steps:**
1. Go to **Actions** tab on GitHub
2. Select **PreMortem | Standup Sync** → **Run workflow**
3. Choose any department → click **Run workflow**
4. Run completes green — job summary shows `STATUS=DEMO`

Repeat with **PreMortem | PR Analysis & Task Correlation** via workflow_dispatch.

---

### Tier 3 — Full integration (backend deployed)

Configure these in **GitHub → Settings → Secrets and variables → Actions**:

| Secret | Required by |
|---|---|
| `PREMORTEM_BACKEND_URL` | All 4 workflows |
| `PREMORTEM_API_TOKEN` | All 4 workflows |
| `TEAMS_WEBHOOK_URL` | All 4 workflows |
| `ARGOCD_SERVER_URL` | `remediation-deploy.yml` only |
| `ARGOCD_AUTH_TOKEN` | `remediation-deploy.yml` only |

Optional repo variable: `PREMORTEM_DEPARTMENT` (defaults to `devops`)

With secrets set:
- `standup-sync.yml` — fetches the real Teams transcript, writes tasks to Excel
- `standup-checklist.yml` — injects live pending tasks into PRs
- `pr-analyze.yml` — Claude correlates the PR diff, posts result + Teams alert
- `remediation-deploy.yml` — ArgoCD sync → task Done → Teams confirmation card

---

*PreMortem + StandupSync · DEWA DevOps / AI Platform Team · v1.3.0*
