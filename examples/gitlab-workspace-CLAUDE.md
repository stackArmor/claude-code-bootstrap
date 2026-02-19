# [Your Org]-gitlab Workspace

> **Where this file lives matters.** See [Where to Put This File](#where-to-put-this-file) below.
>
> This is based on the real workspace-level `CLAUDE.md` used in the StackArmor `armory-gitlab`
> workspace. Copy it, fill in your values, and drop it at the root of your GitLab workspace directory.

This file provides workspace-specific configuration for Claude Code. Values here
supplement the global `~/.claude/CLAUDE.md` and take effect whenever Claude is
invoked from within this directory tree.

---

## GitLab Plugin Configuration

These values are used by the `sa-gitlab` Claude Code plugin skills (e.g., `/sa-gitlab-create-mr`,
`/sa-gitlab-my-issues`, `/sa-gitlab-pipelines`).

| Key | Value |
|-----|-------|
| `GITLAB_PAT_CMD` | `export GITLAB_PAT=$(echo $MY_GITLAB_PAT)` |
| `GITLAB_HOST` | `gitlab.your-org.example.com` |
| `GITLAB_IAP_SA` | `gitlab-iap-api-access@your-gcp-project.iam.gserviceaccount.com` |
| `GITLAB_IAP_AUDIENCE` | `your-iap-client-id.apps.googleusercontent.com` |
| `GITLAB_PRIMARY_PROJECT` | `your-group/your-project` |
| `GITLAB_PRIMARY_PROJECT_ID` | `1` |
| `GITLAB_USERNAME` | `your-gitlab-username` |

### About `GITLAB_PAT_CMD`

`GITLAB_PAT_CMD` is a shell command that sets the `GITLAB_PAT` environment variable. Claude runs it
lazily — only when a GitLab API call is needed and `GITLAB_PAT` is not already set in the session.

The example above reads from `MY_GITLAB_PAT`, which you would export in your shell profile (`~/.zshrc`
or `~/.bashrc`):

```bash
# ~/.zshrc — set once, available in every terminal session
export MY_GITLAB_PAT="glpat-xxxxxxxxxxxxxxxxxxxx"
```

Other common patterns for `GITLAB_PAT_CMD`:

```bash
# macOS Keychain
export GITLAB_PAT=$(security find-generic-password -a $USER -s gitlab-pat -w)

# GCP Secret Manager (if your PAT is stored there)
export GITLAB_PAT=$(gcloud secrets versions access latest --secret=gitlab-pat)

# 1Password CLI
export GITLAB_PAT=$(op read "op://Personal/GitLab PAT/credential")
```

Pick whatever matches how your team manages secrets. The key requirement is that after `GITLAB_PAT_CMD`
runs, the `GITLAB_PAT` environment variable is populated in the current shell session.

---

## Terraform Environment Switching (`fetch-stage-info.sh`)

All stage directories and tenant directories use a `fetch-stage-info.sh` script to switch between
`prod` and `staging` environments. This script downloads environment-specific provider files and
tfvars from GCS, then runs `terraform init -reconfigure`. The provider files enforce hard isolation —
each environment has a dedicated backend bucket and impersonation SA.

### Workflow

1. Set the environment variable:
   ```bash
   export TF_VAR_environment=staging  # or prod
   ```
2. Run the script from within the stage/tenant directory:
   ```bash
   ./fetch-stage-info.sh
   ```
3. The script downloads the correct providers file, tfvars, and runs `terraform init -reconfigure`.

### Rules

- **Never run `terraform init` directly** — always use `fetch-stage-info.sh`
- **Always re-run the script when switching between `prod` and `staging`** — the providers file, backend, and SA change per environment
- **Always target staging first** and validate before running against production
- If `TF_VAR_environment` is not set, the script defaults to `staging` as a safety measure

---

## Where to Put This File

The GitLab plugin values are **workspace-specific** — they differ between your Armory GitLab workspace
and your Gecko GitLab workspace (different host, different IAP SA, different primary project). This
makes the placement decision straightforward:

### Recommended: Workspace Root (one file per org)

Drop a `CLAUDE.md` at the root folder where all repos for that GitLab org live. Claude Code picks it
up for any session started anywhere inside that tree.

```
~/
├── armory-gitlab/          ← CLAUDE.md here (Armory GitLab config)
│   ├── clarity-tenant/
│   ├── example-tenant/
│   └── ...
└── govgecko-gitlab/        ← CLAUDE.md here (Gecko GitLab config, different values)
    ├── some-repo/
    └── ...
```

This is the cleanest approach. You can `cd` freely between repos in that workspace and Claude always
has the right context without you thinking about it. When you switch to `govgecko-gitlab/`, Claude
automatically picks up that workspace's config instead.

### Alternative: Per-Repo

If only a subset of repos in your workspace use GitLab heavily, put the plugin config in each repo's
own `CLAUDE.md` instead of the workspace root.

```
~/armory-gitlab/
├── CLAUDE.md               ← workspace-level (Terraform rules, general context)
└── clarity-tenant/
    └── CLAUDE.md           ← repo-level (adds GitLab plugin config + repo-specific rules)
```

Repo-level `CLAUDE.md` files are committed to version control, so they benefit the whole team.

### Why Not the Global `~/.claude/CLAUDE.md`?

Don't put GitLab plugin config in your global CLAUDE.md. If you work across multiple GitLab instances
(Armory and Gecko, for example), each has a different host, IAP SA, and primary project. A single
global config means you'd have to manually update it every time you switch contexts — and if you
forget, Claude will call the wrong GitLab instance.

The workspace root approach solves this automatically: Claude reads the right config based on where
you are, with no manual switching required.
