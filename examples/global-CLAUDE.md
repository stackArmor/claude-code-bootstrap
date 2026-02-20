# Global Claude Code Instructions

> **This file lives at `~/.claude/CLAUDE.md` and applies to every Claude Code session on your laptop.**
>
> It is deeply personal. Treat it like your `.zshrc` or `.gitconfig` — unique to you and your machine.
> Do not commit this file to any shared repository. Do not copy someone else's verbatim.
>
> This example borrows patterns from team members who have been using Claude Code in production.
> Replace every placeholder with your own information.

---

## Brand Names and Terminology

- The company name is **stackArmor** — lowercase 's', uppercase 'A'. Never write "StackArmor".

---

## Identity

- GitHub username: `YOUR_GITHUB_USERNAME`
- GitLab username: `YOUR_GITLAB_USERNAME`
- Primary email: `you@example.com`

---

## Model Selection

- Use **Sonnet** (`claude-sonnet-4-6`) for all routine tasks — it handles the vast majority of engineering work well.
- Use **Opus** (`claude-opus-4-6`) only when a task genuinely requires deeper reasoning (complex architecture decisions, intricate multi-system debugging). It is significantly more expensive.
- When in doubt, start with Sonnet. Switch to Opus for a specific session only if needed: `claude --model claude-opus-4-6`.
- Use `/cost` inside a session to check token consumption. Use `/clear` to reset a long-running context.

> Formal controls around model usage and token budgets are coming. Use responsibly in the interim.

---

## Communication Style

- Be concise. Prefer short, direct answers over long explanations.
- Do not use emojis unless I ask for them.
- When you are uncertain, say so — do not make things up.
- When there are multiple valid approaches, summarize the trade-offs and ask which I prefer before proceeding.

---

## Plugin and Skill Usage

- **Always prefer installed plugins and skills** over ad-hoc bash commands or manual API calls when a relevant skill exists.
- Check available slash commands with `/` before constructing raw curl or API calls from scratch.

---

## Git Safety

- **Never force push, reset --hard, or run other destructive git commands without explicit approval.**
- **Never push directly to the main or default branch** (`main`, `master`, `armory-prod`, etc.). Always:
  1. Create a new branch from the current default branch
  2. Commit changes to that branch
  3. Push the branch and open a PR or MR
- Write commit messages that explain *why*, not just *what*.
- Always run against a staging or development environment first before touching production.

---

## General Operational Rules

- **Always run `terraform plan` before `terraform apply`.** Never apply without reviewing the plan first.
- **Never make ad-hoc changes directly on production systems.** All changes go through version-controlled automation (Terraform, Ansible, etc.).
- When targeting infrastructure, start with the lowest-risk environment (dev → staging → prod).

---

## Permissions Philosophy

I have configured `settings.json` to auto-approve read-only operations and ask before writes. The following are always gated behind my explicit approval:

- Any `terraform apply` or `terraform destroy`
- Any Ansible playbook runs
- `kubectl apply`, `kubectl delete`, `kubectl create`
- `helm install`, `helm upgrade`, `helm uninstall`
- Any cloud provider delete operations
- Any HTTP POST requests to external services

If you are unsure whether an operation has side effects, ask before running it.

---

## Workspace-Specific Context

> This section is optional. Use it when you work across multiple Git workspaces that each have their own CLAUDE.md.
> You can leave this blank and rely on the workspace-level CLAUDE.md files instead.

| Working directory | Context |
|-------------------|---------|
| `~/work/infra/*`  | Production infrastructure — extra caution, always confirm before any changes |
| `~/work/sandbox/*`| Sandbox environment — more latitude, lower risk |
