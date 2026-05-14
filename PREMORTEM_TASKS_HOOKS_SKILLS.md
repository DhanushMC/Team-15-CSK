# PreMortem — Tasks, Hooks & Skills Reference

> AI-Powered Predictive DevOps Governance Platform
> Hackathon MVP Build Guide

---

## Table of Contents

1. [Build Tasks](#build-tasks)
2. [Claude Code Hooks](#claude-code-hooks)
3. [Claude Code Skills](#claude-code-skills)
4. [Environment Variables](#environment-variables)
5. [Setup Checklist](#setup-checklist)

---

## Build Tasks

Tasks are ordered by dependency. Complete each phase before moving to the next.

---

### Phase 1 — Backend Foundation

| # | Task | File/Module | Output |
|---|---|---|---|
| 1.1 | Scaffold FastAPI app | `backend/main.py` | Running server on `localhost:8000` |
| 1.2 | Define task data model | `backend/models/task.py` | Pydantic model: `engineer, service, planned_task, risk_category, status` |
| 1.3 | Implement Excel/SQLite task store | `backend/storage/task_store.py` | CRUD operations for tasks |
| 1.4 | Add `/tasks` REST endpoint | `backend/routes/tasks.py` | GET, POST, PATCH `/tasks` |
| 1.5 | Microsoft Graph API client | `backend/integrations/graph_client.py` | Fetch Teams meeting transcripts by meeting ID |

**Phase 1 Done When:** You can POST a task and retrieve it. Graph client returns a raw transcript string.

---

### Phase 2 — AI Intelligence Core

| # | Task | File/Module | Output |
|---|---|---|---|
| 2.1 | Standup transcript extraction | `backend/ai/standup_extractor.py` | Claude call → structured JSON: `[{engineer, service, task, risk_category}]` |
| 2.2 | Persist extracted tasks | `backend/routes/standup.py` | `POST /standup/process` → saves tasks to store |
| 2.3 | GitHub webhook receiver | `backend/routes/pr_webhook.py` | `POST /analyze-pr` receives PR payload |
| 2.4 | PR diff correlation engine | `backend/ai/pr_correlator.py` | Claude call → classifies PR as `MATCH / PARTIAL / UNKNOWN` with explanation |
| 2.5 | Infrastructure risk predictor | `backend/ai/risk_predictor.py` | Claude call → returns `{risk, prediction, confidence, reasoning[]}` |
| 2.6 | AI remediation generator | `backend/ai/remediator.py` | Claude call → returns safer config diff + rollback strategy + risk delta |

**Phase 2 Done When:** Given a PR diff and stored tasks, the system outputs a risk JSON with confidence score and a proposed remediation config.

---

### Phase 3 — Approval & GitOps

| # | Task | File/Module | Output |
|---|---|---|---|
| 3.1 | Teams Adaptive Card builder | `backend/integrations/teams_card.py` | Renders approval card JSON from risk + remediation data |
| 3.2 | Teams card sender | `backend/integrations/teams_sender.py` | POSTs card to Teams incoming webhook URL |
| 3.3 | Approval callback handler | `backend/routes/teams_webhook.py` | `POST /webhook/teams` receives Approve/Reject |
| 3.4 | GitHub branch + commit creator | `backend/integrations/github_client.py` | Creates `claude/prevent-<risk>` branch, commits remediation files |
| 3.5 | AI remediation PR generator | `backend/ai/pr_generator.py` | Generates PR title + description via Claude, opens PR via GitHub API |
| 3.6 | Audit trail logger | `backend/storage/audit_log.py` | Records every approval, rejection, deployment event |

**Phase 3 Done When:** Approving a Teams card automatically creates a GitHub PR with AI-generated description and remediation config.

---

### Phase 4 — Demo Polish (Optional MVP+)

| # | Task | Notes |
|---|---|---|
| 4.1 | Mock Prometheus metrics endpoint | Simulate `peak_memory`, `cpu_usage`, `restart_count` for demo |
| 4.2 | Mock incident history JSON | Seed historical OOMKill, scaling failure incidents |
| 4.3 | Dashboard UI (React or Streamlit) | Show task list, risk predictions, approval history |
| 4.4 | End-to-end demo script | `scripts/demo.sh` — runs full flow from standup to PR in one command |

---

## Claude Code Hooks

Hooks are shell commands that Claude Code executes automatically at specific events during a session. Configure them in `.claude/settings.json`.

---

### Hook: Auto-Format Python on Save

**Trigger:** `PostToolUse` — fires after every `Write` or `Edit` tool call on a `.py` file

**Purpose:** Keeps code consistently formatted without manual intervention. Ruff is used for linting and Black for formatting.

**Command:**
```bash
ruff check --fix "$FILE" && black "$FILE"
```

**Settings entry:**
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "ruff check --fix \"$FILE\" && black \"$FILE\""
          }
        ]
      }
    ]
  }
}
```

**When it fires:** After Claude writes `backend/ai/risk_predictor.py`, it automatically lints and formats the file before showing you the result.

---

### Hook: Validate YAML on Save

**Trigger:** `PostToolUse` — fires after every `Write` or `Edit` tool call on a `.yaml` or `.yml` file

**Purpose:** Kubernetes and Terraform configs must be valid YAML. This catches malformed configs before they reach the risk predictor or GitHub.

**Command:**
```bash
python -c "import yaml, sys; yaml.safe_load(open('$FILE'))" || echo "YAML INVALID: $FILE"
```

**Settings entry:**
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "python -c \"import yaml, sys; yaml.safe_load(open('$FILE'))\" || echo \"YAML INVALID: $FILE\""
          }
        ]
      }
    ]
  }
}
```

**When it fires:** After Claude generates a remediation `deployment.yaml`, it immediately validates the YAML structure and reports any syntax errors.

---

### Hook: Run Tests After Code Changes

**Trigger:** `PostToolUse` — fires after `Write` or `Edit` on any `*.py` file inside `backend/`

**Purpose:** Keeps the test suite green after every AI-generated code change. Catches regressions immediately.

**Command:**
```bash
pytest tests/ -q --tb=short 2>&1 | tail -20
```

**Settings entry:**
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "pytest tests/ -q --tb=short 2>&1 | tail -20"
          }
        ]
      }
    ]
  }
}
```

**When it fires:** After Claude adds the `pr_correlator.py` module, pytest runs automatically and reports any broken imports or failing assertions.

---

### Hook: Validate GitHub Webhook Signature

**Trigger:** `PostToolUse` — fires after editing `backend/routes/pr_webhook.py`

**Purpose:** The GitHub webhook handler must verify `X-Hub-Signature-256` on every request. This hook warns if the signature check is missing.

**Command:**
```bash
grep -q "X-Hub-Signature-256\|hmac\|sha256" backend/routes/pr_webhook.py || echo "WARNING: Missing GitHub webhook signature verification"
```

**When it fires:** If Claude edits the webhook route and accidentally removes HMAC validation, this hook immediately surfaces the security gap.

---

### Hook: Show Changed Files on Stop

**Trigger:** `Stop` — fires when Claude finishes a work session

**Purpose:** Gives a clean summary of every file touched during the session so you know exactly what changed before reviewing or committing.

**Command:**
```bash
git diff --name-only HEAD 2>/dev/null || find . -newer .git/index -name "*.py" -o -name "*.yaml" -o -name "*.json" 2>/dev/null | grep -v __pycache__
```

**Settings entry:**
```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo '--- Files changed this session ---' && git diff --name-only HEAD 2>/dev/null"
          }
        ]
      }
    ]
  }
}
```

**When it fires:** When you say "stop" or Claude completes a task, it prints a concise list of every modified file.

---

### Complete `.claude/settings.json`

```json
{
  "permissions": {
    "allow": [
      "Bash(ruff:*)",
      "Bash(black:*)",
      "Bash(pytest:*)",
      "Bash(python:*)",
      "Bash(git diff:*)",
      "Bash(grep:*)"
    ]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "ruff check --fix \"$FILE\" 2>/dev/null; black \"$FILE\" 2>/dev/null; true"
          },
          {
            "type": "command",
            "command": "case \"$FILE\" in *.yaml|*.yml) python -c \"import yaml; yaml.safe_load(open('$FILE'))\" && echo 'YAML OK' || echo 'YAML INVALID';; esac"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo '=== Session Summary ===' && git diff --name-only HEAD 2>/dev/null || echo 'No git repo'"
          }
        ]
      }
    ]
  }
}
```

---

## Claude Code Skills

Skills are invoked with `/skill-name` inside Claude Code. Each skill brings specialized behavior and domain knowledge.

---

### Skill: `claude-api`

**Invoke:** `/claude-api`

**When to use:** Any time you are writing, debugging, or optimizing code that calls the Anthropic SDK.

**Why it matters for PreMortem:**
Every AI operation in this project goes through the Claude API:
- `standup_extractor.py` — transcript → structured tasks
- `pr_correlator.py` — PR diff + tasks → MATCH/PARTIAL/UNKNOWN classification
- `risk_predictor.py` — infra diffs + metrics → risk JSON
- `remediator.py` — risk data → safer config + rollback strategy
- `pr_generator.py` — risk + remediation → PR title and description

**What the skill handles automatically:**
- Selects the right Claude model (`claude-sonnet-4-6` for fast inference, `claude-opus-4-7` for complex risk analysis)
- Implements **prompt caching** on large system prompts (the risk predictor's system prompt will be large — caching saves ~80% of token cost per call)
- Structures tool use correctly for structured JSON output
- Handles API errors, retries, and rate limiting

**Example trigger:**
```python
# Opening this file triggers the claude-api skill automatically
import anthropic

client = anthropic.Anthropic()
```

---

### Skill: `update-config`

**Invoke:** `/update-config`

**When to use:** Setting up hooks, adding permissions, configuring environment variables, or modifying `.claude/settings.json` and `.claude/settings.local.json`.

**Why it matters for PreMortem:**
PreMortem requires several sensitive credentials and automated behaviors:
- `ANTHROPIC_API_KEY` — Claude API access
- `GITHUB_TOKEN` — create branches and PRs
- `MS_GRAPH_TOKEN` — fetch Teams transcripts
- `TEAMS_WEBHOOK_URL` — send Adaptive Cards
- All the hooks defined above need to be wired into settings

**Use this skill to:**
- Add the auto-format, YAML validation, and test-runner hooks
- Allowlist `ruff`, `black`, `pytest`, `git` commands so hooks don't prompt for permission
- Set environment variables that the backend reads at startup
- Move sensitive tokens to `settings.local.json` (not committed to git)

**Example: what to say after invoking:**
```
Add permission for pytest, ruff, and black.
Set ANTHROPIC_API_KEY and GITHUB_TOKEN as env vars in settings.local.json.
Add the PostToolUse hook to run ruff after every Python file edit.
```

---

### Skill: `security-review`

**Invoke:** `/security-review`

**When to use:** Before finalizing Phase 3 (approval and GitOps layer). Run it on the webhook handlers and GitHub integration code.

**Why it matters for PreMortem:**
PreMortem has three public-facing attack surfaces that need review:

| Endpoint | Risk |
|---|---|
| `POST /analyze-pr` (GitHub webhook) | Must verify `X-Hub-Signature-256` HMAC or anyone can trigger PR analysis |
| `POST /webhook/teams` (Teams approval callback) | Must validate Teams request origin or an attacker can fake approvals |
| GitHub API client | Token scope must be minimal — only `repo` write, not `admin` |

**What the skill checks:**
- Missing or bypassable signature verification
- Secrets leaked into logs or PR descriptions
- Command injection via PR diff content passed to shell
- Overly permissive GitHub token scopes
- Sensitive data in Adaptive Card payloads

**When to run it:** After completing Task 3.3 (approval callback handler) and Task 3.4 (GitHub client).

---

### Skill: `review`

**Invoke:** `/review`

**When to use:** Reviewing the AI-generated remediation PRs before they are merged into the GitOps pipeline.

**Why it matters for PreMortem:**
The entire value proposition of PreMortem rests on the quality of its AI-generated remediation PRs. A PR that increases memory from `256Mi` to `768Mi` must be reviewed for:
- Does the config change actually match the risk prediction?
- Is the rollback strategy viable?
- Are there downstream resource quota implications?

**Also use it for:** Reviewing your own implementation PRs (Phases 1–3) before merging to `main`.

---

### Skill: `init`

**Invoke:** `/init`

**When to use:** At the very start of the project, before writing any code.

**Why it matters for PreMortem:**
Generates a `CLAUDE.md` file that documents:
- Project structure and module responsibilities
- Which Claude models to use for each AI call
- Coding conventions (async FastAPI patterns, error handling)
- How to run tests and start the server

This ensures every future Claude Code session starts with full project context without re-explaining the architecture.

**When to run it:** Once Phase 1 skeleton exists (even empty files) — `/init` will scan the structure and write accurate documentation.

---

## Environment Variables

Store secrets in `.claude/settings.local.json` (gitignored). Reference them in code via `os.getenv()`.

| Variable | Used By | How to Get |
|---|---|---|
| `ANTHROPIC_API_KEY` | All Claude API calls | console.anthropic.com |
| `GITHUB_TOKEN` | Branch creation, PR creation | GitHub → Settings → Tokens (scope: `repo`) |
| `MS_GRAPH_CLIENT_ID` | Graph API auth | Azure Portal → App Registration |
| `MS_GRAPH_CLIENT_SECRET` | Graph API auth | Azure Portal → App Registration |
| `MS_GRAPH_TENANT_ID` | Graph API auth | Azure Portal → Directory ID |
| `TEAMS_WEBHOOK_URL` | Adaptive Card sender | Teams channel → Connectors → Incoming Webhook |
| `GITHUB_WEBHOOK_SECRET` | Webhook signature verification | Set when creating GitHub webhook |

---

## Setup Checklist

Run these in order at the start of the project:

```
[ ] 1. /init                  — Generate CLAUDE.md with project structure
[ ] 2. /update-config         — Add hooks, permissions, env vars to settings.json
[ ] 3. Build Phase 1          — Backend foundation (Tasks 1.1–1.5)
[ ] 4. /claude-api            — Scaffold all Claude API calls (Tasks 2.1, 2.4, 2.5, 2.6)
[ ] 5. Build Phase 2          — AI core (Tasks 2.2, 2.3)
[ ] 6. Build Phase 3          — Approval + GitOps (Tasks 3.1–3.6)
[ ] 7. /security-review       — Audit webhook handlers and GitHub integration
[ ] 8. /review                — Review all PRs before merging to main
[ ] 9. Build Phase 4 (optional) — Demo polish
```

---

## Quick Reference

| You want to... | Use |
|---|---|
| Write a Claude API call with caching | `/claude-api` |
| Add a hook or env var to settings | `/update-config` |
| Check webhook security before demo | `/security-review` |
| Review a PR (yours or AI-generated) | `/review` |
| Document the codebase for future sessions | `/init` |
| Run the Python formatter | Hook fires automatically on file save |
| Validate a Kubernetes YAML | Hook fires automatically on file save |
| See what changed this session | Hook fires on Stop |
