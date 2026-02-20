# Claude Code Bootstrap

Getting started with [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) at stackArmor.

> **Note for the Slack canvas:** See the `#armory-technical-team` channel canvas for additional team-specific tips and screencasts.

---

## stackArmor Setup: Vertex AI Backend

At stackArmor, Claude Code runs via **Vertex AI** — not the public Anthropic API. This means:

- No personal Anthropic account or API key required
- All usage is authenticated through your GCP identity (`gcloud` ADC)
- Every API call is auditable per-user — no shared credentials
- Access is gated by IAM group membership in `armoryd2v-gss-prod`

**Before anything else, follow the Vertex AI setup guide:**

**[docs/vertex-ai-setup.md](docs/vertex-ai-setup.md)**

It covers setup for both VS Code and the terminal CLI. The short version:

```bash
# 1. Authenticate gcloud (sets Application Default Credentials)
gcloud auth login --update-adc

# 2. Add to ~/.claude/settings.json
{
  "env": {
    "CLAUDE_CODE_USE_VERTEX": "1",
    "CLOUD_ML_REGION": "global",
    "ANTHROPIC_VERTEX_PROJECT_ID": "armoryd2v-gss-prod"
  }
}

# 3. Launch
claude
```

If you get a `PERMISSION_DENIED` error, ask your manager to add your GCP identity to the Vertex AI access group.

---

## Model Selection & Responsible Use

> **Governance notice:** Formal controls around model selection, token budgets, and API rate limiting
> are coming. In the interim, please use responsibly — costs are real and shared across the team.

### Which Model to Use

| Model | When to use |
|-------|------------|
| `claude-sonnet-4-6` | **Default for everything.** Fast, capable, and cost-effective. Handles the vast majority of engineering tasks well. |
| `claude-opus-4-6` | **Complex tasks only.** Large-scale architecture decisions, intricate multi-system debugging, tasks where Sonnet has demonstrably struggled. |

**Start every session with Sonnet.** If you find yourself hitting the limits of what it can reason through, switch to Opus for that specific task. Do not default to Opus out of habit — the cost difference is significant.

Set your default in `~/.claude/settings.json`:

```json
{
  "model": "claude-sonnet-4-6"
}
```

To switch models for a single session without changing your config:

```bash
claude --model claude-opus-4-6
```

### Checking Your Usage

Use the `/cost` command inside any Claude Code session to see token consumption for the current conversation. If a session is running long, `/clear` resets the context window and reduces ongoing cost.

### What Counts as a "Complex Task"?

Opus is worth considering when the task genuinely requires deeper reasoning — for example:

- Designing the architecture for a new multi-tenant system from scratch
- Debugging a subtle race condition or security issue across many interacting components
- Reviewing a large, unfamiliar codebase to produce a detailed remediation plan

Most day-to-day work — writing Terraform, fixing a bug, drafting runbook updates, creating MRs — is well within Sonnet's capability.

---

## What is Claude Code?

Claude Code is an agentic CLI tool built by Anthropic that runs directly in your terminal. Unlike a chat interface, Claude Code can read your files, run commands, edit code, search the web, and orchestrate complex multi-step tasks — all from your working directory.

If you are new to agentic AI development, here is what that means in practice:

- You describe a task in natural language ("add a Cloud SQL instance to this Terraform module")
- Claude reads the relevant files, forms a plan, and executes it — asking for confirmation before anything dangerous
- You review, approve, or redirect as it works

This is different from asking ChatGPT a question. Claude Code has **agency**: it takes actions on your behalf. That is powerful, which is why the configuration below matters.

---

## Installation

```bash
# WSL / Ubuntu / macOS
curl -fsSL https://claude.ai/install.sh | bash

# Verify
claude --version
```

At stackArmor you do **not** need an Anthropic account or API key. Authentication is handled entirely through GCP. See [docs/vertex-ai-setup.md](docs/vertex-ai-setup.md) for the full setup walkthrough.

---

## How Configuration Works

Claude Code uses two main configuration mechanisms:

### 1. `CLAUDE.md` — Natural Language Instructions

`CLAUDE.md` files contain instructions written in plain English (or Markdown). Claude reads them at startup and follows them throughout the session.

**They are hierarchical and additive.** Claude reads all of them and combines the instructions:

```
~/.claude/CLAUDE.md           ← Global: applies to EVERY session on your laptop
~/your-workspace/CLAUDE.md    ← Workspace: applies when working in that directory tree
~/your-workspace/repo/CLAUDE.md  ← Repo: applies only in that specific repository
```

Each layer adds to (not replaces) the layers above it. A repo-level CLAUDE.md can refine or extend your global instructions for that specific project.

**This file is deeply personal.** Your `~/.claude/CLAUDE.md` should reflect:

- Your usernames and credentials patterns
- Your preferred workflow and communication style
- The tools you use (Terraform, Ansible, Kubernetes, etc.) and your safety rules for them
- Which operations should always require your approval

Copy `examples/global-CLAUDE.md` as a starting point, but treat it as a living document you refine over time. **Do not commit your global CLAUDE.md to any shared repository** — it contains personal identity and workflow details that are unique to your laptop.

Repo-level `CLAUDE.md` files *can* and *should* be committed to the repository — they document how Claude should work in that specific codebase and are useful for the whole team.

### 2. `~/.claude/settings.json` — Permissions and Model Config

`settings.json` controls:

- **Which tools Claude can use without asking** (`allow` rules)
- **Which operations require your explicit approval** (`ask` rules)
- **Environment variables** passed to every session
- **Which plugins are enabled**
- **The default model**

See the [examples](#example-files) section for two configurations: a safe starter and an advanced "yolo" mode.

---

## Getting Started: Safe Mode (Recommended)

When you are new to Claude Code, **let it prompt you for every command.** This builds your intuition for what Claude does and does not need permission for.

The default out-of-the-box behavior already asks before running shell commands. Start there. As you gain confidence, you can whitelist specific commands you trust.

A common progression:

1. **Week 1–2:** Default settings. Claude asks before every bash command. You see exactly what it wants to run.
2. **Week 3+:** Whitelist read-only operations you trust (file reads, `git status`, `terraform plan`).
3. **Later:** Whitelist broader patterns for commands you fully understand and are comfortable with.

The sample configs in `examples/` reflect this progression.

---

## Example Files

| File | Purpose |
|------|---------|
| `examples/global-CLAUDE.md` | Sample `~/.claude/CLAUDE.md` — copy and personalize |
| `examples/gitlab-workspace-CLAUDE.md` | Workspace-level `CLAUDE.md` for GitLab orgs — `sa-gitlab` plugin config with placement guidance |
| `examples/repo-CLAUDE.md` | Sample repo-level `CLAUDE.md` — commit this to your repositories |
| `examples/settings-starter.json` | Safe defaults — Claude asks before any shell command |
| `examples/settings-yolo.json` | Advanced "quasi-yolo" — auto-approves most commands, asks for destructive ops |

---

## Plugins

Plugins extend Claude Code with team-specific skills and slash commands. They are installed from the Claude Code marketplace or from private plugin repositories.

### Installing a Plugin

```
# Inside a Claude Code session, type:
/plugins
```

This opens the plugin browser. Search for a plugin by name and install it.

Alternatively, if your team publishes a private plugin:

```
# Install from a GitHub org
/plugins install <org>/<plugin-name>
```

### What Plugins Provide

Plugins add **skills** — slash commands you can invoke by typing `/skill-name`. Examples:

- `/create-mr` — open a GitLab MR from the current branch
- `/my-issues` — list GitLab issues assigned to you
- `/change-request` — file a change request issue with a template

### Recommended Plugins for stackArmor

| Plugin | What it does |
|--------|-------------|
| `sa-gitlab@stackArmor` | GitLab operations (MRs, issues, pipelines) scoped to our instance |
| `claude-md-management@claude-plugins-official` | Helps audit and improve your CLAUDE.md files |
| `claude-code-setup@claude-plugins-official` | Analyzes a repo and recommends Claude Code automations |

To enable a plugin, add it to the `enabledPlugins` section of `~/.claude/settings.json` (see `examples/settings-yolo.json` for the format).

### Configuring a Plugin

Plugins read configuration from the `CLAUDE.md` nearest to your working directory. The `sa-gitlab`
plugin, for example, reads a configuration table like this from your workspace or repo-level `CLAUDE.md`:

```markdown
## GitLab Plugin Configuration

| Key | Value |
|-----|-------|
| `GITLAB_PAT_CMD`            | `export GITLAB_PAT=$(echo $MY_GITLAB_PAT)` |
| `GITLAB_HOST`               | `gitlab.your-org.example.com` |
| `GITLAB_IAP_SA`             | `gitlab-iap-api-access@your-project.iam.gserviceaccount.com` |
| `GITLAB_IAP_AUDIENCE`       | `your-iap-client-id.apps.googleusercontent.com` |
| `GITLAB_PRIMARY_PROJECT`    | `your-group/your-project` |
| `GITLAB_PRIMARY_PROJECT_ID` | `1` |
| `GITLAB_USERNAME`           | `your-gitlab-username` |
```

A full annotated example — including where to place this file, how to configure `GITLAB_PAT_CMD`,
and why you should not put this in your global `CLAUDE.md` — is in:

**[examples/gitlab-workspace-CLAUDE.md](examples/gitlab-workspace-CLAUDE.md)**

Check each plugin's documentation for the full list of keys it recognizes.

---

## Tips for Effective Use

### Be Specific About What You Want

Claude Code works best when you describe the *outcome*, not just the action:

- Instead of: "edit the Terraform"
- Try: "Add a Cloud SQL instance to the staging environment. Use the same pattern as the existing postgres instance in `db.tf`. Target `armory-example-staging`."

### Use Slash Commands

Type `/` inside a Claude Code session to see available slash commands. These include skills from your installed plugins plus built-ins like `/clear`, `/help`, and `/cost`.

### Review Before Approving

When Claude asks to run a command, read it. This is the most important habit. You will quickly develop a sense for what is reasonable and what looks wrong.

### CLAUDE.md Is a Living Document

Add a rule to your global CLAUDE.md every time you notice Claude doing something you didn't want. Over time it becomes a precise description of how you like to work.

### Use Repo-Level CLAUDE.md for Team Knowledge

The repo-level `CLAUDE.md` is a great place to document:

- Which environments exist and how to target them
- Which operations are safe to run without approval (read-only)
- Which files are generated and should not be edited
- Key runbooks or documentation to consult first

This helps every team member who uses Claude Code in that repo get up to speed faster.

---

## Further Reading

- [stackArmor Vertex AI Setup](docs/vertex-ai-setup.md) — how auth works and how to configure it
- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code/overview)
- [CLAUDE.md Best Practices](https://docs.anthropic.com/en/docs/claude-code/memory)
- [Settings Reference](https://docs.anthropic.com/en/docs/claude-code/settings)
- `#armory-technical-team` Slack channel canvas — team-specific setup notes, screencasts, and tips
