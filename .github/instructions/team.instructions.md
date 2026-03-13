---
applyTo: "**/pt-*/**"
---

# GitHub Copilot Workspace Instructions

## Ignored Directories

**Ignore all `.terraform/` directories entirely.** These are downloaded artifacts from `tofu init` and contain cached copies of OpenTofu child modules. Any instructions, skills files, or `copilot-instructions.md` files found inside `.terraform/` belong to the source module repos — they are not applicable in the consuming repo context and may contain outdated or conflicting guidance. Never read, search, or use content from `.terraform/` directories.

## Workspace Overview

This VS Code workspace aggregates all platform team repositories into a single multi-root workspace. Each top-level directory is a collection of related repositories. **Keep this tree up to date when repositories are added or removed.**

```
platform-teams/
├── pt-ai-context/                      # platform instructions (COPILOT_CUSTOM_INSTRUCTIONS_DIRS)
├── arche/
│   ├── pt-arche-ai-context/            # team instructions (COPILOT_CUSTOM_INSTRUCTIONS_DIRS)
│   ├── pt-arche-core-helpers/
│   ├── pt-arche-datadog-google-integration/
│   ├── pt-arche-google-cloud-sql/
│   ├── pt-arche-google-kubernetes-engine/
│   ├── pt-arche-google-network/
│   ├── pt-arche-google-project/
│   ├── pt-arche-google-storage-bucket/
│   ├── pt-arche-kubernetes-cert-manager/
│   ├── pt-arche-kubernetes-datadog-operator/
│   ├── pt-arche-kubernetes-istio/
│   └── pt-arche-kubernetes-opa-gatekeeper/
├── corpus/
│   ├── pt-corpus-ai-context/           # team instructions (COPILOT_CUSTOM_INSTRUCTIONS_DIRS)
│   └── pt-corpus/
├── ekklesia/
│   ├── pt-ekklesia-ai-context/         # team instructions (COPILOT_CUSTOM_INSTRUCTIONS_DIRS)
│   ├── pt-ekklesia/
│   ├── pt-ekklesia-docs/
│   └── pt-ekklesia-repository-templates/
├── logos/
│   ├── pt-logos-ai-context/            # team instructions (COPILOT_CUSTOM_INSTRUCTIONS_DIRS)
│   └── pt-logos/
├── pneuma/
│   ├── pt-pneuma-ai-context/           # team instructions (COPILOT_CUSTOM_INSTRUCTIONS_DIRS)
│   ├── pt-pneuma/
│   └── pt-pneuma-istio-test/
└── techne/
    ├── pt-techne-ai-context/           # team instructions (COPILOT_CUSTOM_INSTRUCTIONS_DIRS)
    ├── pt-techne-development-setup/
    ├── pt-techne-misc-workflows/
    ├── pt-techne-opentofu-codespace/
    ├── pt-techne-opentofu-workflows/
    └── pt-techne-pre-commit-hooks/
```

## Platform Architecture

The platform is organized into teams (a domain of concern that may map to individual teams, repositories, and infrastructure boundaries — or in the case of a small platform team, all of them), each with a distinct role. Deployment dependencies between the infrastructure layers are enforced at the GitHub Actions workflow level.

- **Logos** — Creates GCP folder hierarchy, Google Identity groups, GitHub teams and repositories, and Datadog teams. Everything downstream depends on its outputs.
- **Corpus** — Translates Logos structure into real infrastructure: GCP projects with CIS compliance, shared VPC and subnets, DNS zones, Artifact Registry, GitHub Actions service accounts, workload identity pools, and encrypted state buckets.
- **Pneuma** — Animates Corpus projects into workload environments: GKE clusters across multiple zones, cert-manager, Istio service mesh, Datadog cluster monitoring, and OPA Gatekeeper policy enforcement.
- **Arche** — Reusable OpenTofu child modules consumed by all three infrastructure layers. Covers GCP projects, GKE, networking, storage, Datadog integration, and all Kubernetes add-ons. The `pt-arche-core-helpers` module is foundational — providing environment detection, labels, and team naming. Check `helpers.tofu` before hardcoding any of those values.
- **Ekklesia** — Developer portal and documentation: platform docs, Backstage running on GKE, and repository templates for new IaC projects.
- **Techne** — Shared tooling: reusable GitHub Actions called workflows for OpenTofu deployments (OIDC auth, state encryption, job summaries), pre-commit hooks for IaC validation, and a GitHub Codespace for standardized developer environments.

## Code Quality Principles

- **Keep it simple** — Favor straightforward solutions over clever ones.
- **Less is more** — Write only the code necessary to solve the problem at hand. Every line is a liability that must be maintained and understood.
- **Avoid over-engineering** — Don't add abstraction or flexibility for hypothetical future needs. Solve today's problems today.
- **Value clarity over brevity** — Explicit, readable code over terse cleverness.
- **Prefer explicit over implicit** — Make dependencies, transformations, and logic flows obvious.
- **Write code for humans first** — Code is read far more often than it's written.

## OpenTofu File Structure

Every module follows this standard file layout:

| File | Purpose |
| --- | --- |
| `main.tofu` | Resource and module declarations |
| `variables.tofu` | Input variables with descriptions and validation |
| `outputs.tofu` | Output values |
| `locals.tofu` | Complex data transformations |
| `helpers.tofu` | Invokes `pt-arche-core-helpers`; provides env, labels, project naming |
| `data.tofu` | Data source definitions |
| `providers.tofu` | Provider configuration |
| `backend.tofu` | State backend configuration |
| `moved.tofu` | Resource renames and moves |

The following files begin with a two-line file header:

```hcl
# <Display Name>
# <URL>
```

| File | Header |
| --- | --- |
| `variables.tofu` | `# Input Variables` / `https://opentofu.org/docs/language/values/variables` |
| `outputs.tofu` | `# Output Values` / `https://opentofu.org/docs/language/values/outputs` |
| `locals.tofu` | `# Local Values` / `https://opentofu.org/docs/language/values/locals` |
| `backend.tofu` | `# Backend Configuration` / `https://opentofu.org/docs/language/settings/backends/configuration` |
| `moved.tofu` | `# Moved Blocks` / `https://opentofu.org/docs/language/moved` |
| `helpers.tofu` | `# OpenTofu Core Helpers Module (osinfra.io)` / `https://github.com/osinfra-io/pt-arche-core-helpers` |
| `providers.tofu` | begins directly with a `terraform {}` block — no file header |
| `main.tofu` | no file-level header — begins directly with the first resource or module block comment |
| `data.tofu` | no file-level header — begins directly with the first data source block comment |

A blank line after the header is followed optionally by additional `#` comment lines describing the file's specific purpose (e.g. what the locals transform, what the outputs expose).

Every `resource`, `data`, and `module` block in `main.tofu` and `data.tofu` is preceded by its own comment block:

```hcl
# <Resource or Module Display Name>
# <URL to provider docs or GitHub repo>
# <optional: additional context lines>

resource "google_project" "this" {
```

Use the provider's documentation URL (e.g. `https://search.opentofu.org/provider/...`) for resource and data blocks. Use the GitHub repo URL (e.g. `https://github.com/osinfra-io/pt-arche-...`) for module blocks.

**Ordering rules (strictly enforced by pre-commit):**
- Variables, outputs, locals, `.tfvars` entries: alphabetical
- Modules and resources/data sources: alphabetical by their label (the identifier after the type keyword, e.g. `resource "google_project" "this"` sorts by `this`)
- All arguments within a block: alphabetical
- Meta-arguments (`count`, `depends_on`, `for_each`, `lifecycle`, `provider`) come first, alphabetically among themselves
- Exception: logical grouping is allowed for team membership variables when annotated with a comment

**Formatting rules:**
- Lists and maps: empty newline before and after, unless they are the first or last argument in the block
- Functions: single-line for simple calls; multi-line for complex nested functions
- Indentation: 2 spaces (enforced by `tofu fmt`)

```hcl
resource "example" "this" {
  description = "Example resource"

  labels = {
    env  = "production"
    team = "platform"
  }

  name = "example"

  tags = [
    "platform",
    "production",
  ]
}
```

## Pre-Commit Workflow (Mandatory)

**`pre-commit run -a` must be run after ANY change in this workspace.** Do not wait to be asked.

Run `pre-commit autoupdate --freeze` once at the start of a session before the first `pre-commit run -a`.

```bash
pre-commit autoupdate --freeze   # run once per session to update hooks and pin to commit SHAs
pre-commit run -a                # run after every change; enforces fmt, validate, docs, security
```

Pre-commit hooks enforce: `tofu fmt`, `tofu validate`, `tofu test`, YAML validation, documentation generation, and trailing whitespace fixes.

## Key Conventions

### Logic in Locals

All conditional logic, string transformations, and computed values must live in `locals.tofu` — never inline in resource or module arguments. Module and resource blocks should reference `local.*` values only.

```hcl
# ✅ Correct
locals {
  dns_name = var.env == "prod" ? "example.com." : "${var.env}.example.com."
}
module "dns" {
  dns_name = local.dns_name
}

# ❌ Wrong
module "dns" {
  dns_name = var.env == "prod" ? "example.com." : "${var.env}.example.com."
}
```

### Module Source Pinning

Always pin module sources to a full 40-character commit SHA with an inline version comment:

```hcl
module "helpers" {
  source = "github.com/osinfra-io/pt-arche-core-helpers//child?ref=<commit_sha>"  # v1.2.3
}
```

Never use branch names (unless explicitly required for testing) or semver tags as `ref` values.

### Lifecycle

- Use `prevent_destroy = true` for critical infrastructure (KMS keys, state buckets, admin accounts)
- Use `ignore_changes` for externally managed fields
- Prefer `enabled` (OpenTofu v1.11+) over `count` for conditional resource creation:

```hcl
# Preferred
resource "example" "this" {
  lifecycle {
    enabled = local.should_create
  }
  name = "example"
}
```

### Variables and Outputs

- Every variable must have `description` and `type`; use `sensitive = true` for secrets; add `validation` for constrained values
- Every output must have a `description`; name outputs after the attribute they expose (`id`, `name`, `self_link`, `email`)

### GitHub Actions

- All actions must use full 40-character commit SHAs with an inline version comment: `uses: action/name@<sha>  # v1.2.3`
- All OpenTofu deployments use reusable called workflows from `osinfra-io/pt-techne-opentofu-workflows`
- OpenTofu state is managed exclusively in GitHub Actions — local development does not have access to state
- The `shared/` directory contains canonical backend and provider configurations symlinked into every workspace directory. When adding new workspace directories, symlink from `shared/`.

Infrastructure teams follow a three-tier deployment workflow unless team-level instructions say otherwise:

- **`sandbox.yml`** — runs on pull requests; deploys to sandbox
- **`non-production.yml`** — runs on merge to `main`; deploys to non-production
- **`production.yml`** — runs automatically after non-production succeeds

Each workflow deploys the main workspace first, then if required, regional jobs (which `needs: main`) run after it completes. Team-level instructions document additional job ordering for subdirectory workspaces.

### Remote State

`terraform_remote_state` is only used within the same repository (e.g. a regional workspace reading its own main state). Cross-repo state is never accessed via `terraform_remote_state` — it is consumed exclusively through `module.helpers`.

### Environments

Three environments are used across all deployments:

| Short (`module.helpers.env`) | Long (`module.helpers.environment`) |
| --- | --- |
| `sb` | `sandbox` |
| `np` | `non-production` |
| `prod` | `production` |

### Workspace Naming

Workspace naming patterns vary by repo type:

- **pt-logos**: `pt-logos-main-{env}` (e.g. `pt-logos-main-production`)
- **pt-corpus**: main workspace `main-{env}`; regional workspace `{region}-{env}` (e.g. `us-east1-production`)
- **pt-pneuma**: main workspace `main-{env}`; zonal workspace `{zone}-{env}` (e.g. `us-east1-b-production`); subdirectory workspace `{zone}-{subdirectory}-{env}` (e.g. `us-east1-b-cert-manager-production`)

### Core Helpers Module

The `pt-arche-core-helpers` module is invoked from every root module's `helpers.tofu`. Always use its outputs rather than hardcoding environment strings, labels, or team data. Available outputs (not all present in every repo type — child modules using `//child` get only `env`, `environment`, `region`, `zone`, `labels`; root modules using `//` get the full set):

| Output | Description |
| --- | --- |
| `module.helpers.env` | Short environment name: `sb`, `np`, `prod` |
| `module.helpers.environment` | Full environment name: `sandbox`, `non-production`, `production` |
| `module.helpers.environment_folder_id` | GCP folder ID for the current environment |
| `module.helpers.labels` | Standard resource labels map — apply to all GCP resources |
| `module.helpers.project_naming` | Struct with `.prefix` and `.description` for project creation |
| `module.helpers.region` | Current deployment region |
| `module.helpers.team` | Current team identifier (e.g. `pt-pneuma`) |
| `module.helpers.teams` | Map of all team data from pt-logos (folders, identity groups, GitHub repos, etc.) |
| `module.helpers.zone` | Current deployment zone (zonal subdirectories only) |

### Creating Releases

Tags follow [Semantic Versioning](https://semver.org/): `MAJOR.MINOR.PATCH` — increment MAJOR for breaking changes, MINOR for backwards-compatible additions, PATCH for backwards-compatible fixes.

Push a tag — the GitHub Actions release workflow generates notes and publishes automatically:

```none
git tag vX.Y.Z
git push origin vX.Y.Z
```

### Arche Module Upgrade Order

The `pt-arche-*` modules have an internal dependency on `pt-arche-core-helpers`. When releasing new versions across the platform, always follow this order to ensure every module is pinned to post-merge commit SHAs (not pre-merge branch SHAs):

1. **Release `pt-arche-core-helpers`** — merge PR, push tag.
2. **Update all other `pt-arche-*` modules** — update their `helpers.tofu` to the new `pt-arche-core-helpers` SHA, open PRs, merge, push tags.
3. **Update consumers** (`pt-corpus`, `pt-ekklesia`, `pt-pneuma`) — update their `main.tofu` / `helpers.tofu` files to the new post-merge SHAs of every `pt-arche-*` module they reference, then open/update PRs and merge.

> **Important:** Module source `ref=` values must always point to a post-merge commit SHA on `main`, not to the SHA of the branch tip before the squash merge. After merging a PR, fetch `main` and use that SHA when updating downstream consumers.

### README Badges

All README files include status badges immediately after the title (before any other content). Use `style=for-the-badge` on all badges for visual consistency.

Every repo includes a Dependabot badge linking to its `dependabot.yml` workflow run:

```markdown
[![Dependabot](https://img.shields.io/github/actions/workflow/status/osinfra-io/<repo>/dependabot.yml?style=for-the-badge&logo=github&color=2088FF&label=Dependabot)](https://github.com/osinfra-io/<repo>/actions/workflows/dependabot.yml)
```

Repos containing IaC (OpenTofu) also include a Datadog Security Enabled badge:

```markdown
[![Datadog Security Enabled](https://img.shields.io/badge/Datadog%20Security-Enabled-632CA6?style=for-the-badge&logo=datadog)](https://app.datadoghq.com/security/code-security/repositories?repository_id=<repo>)
```

### Markdown

Follow [markdownlint](https://github.com/DavidAnson/markdownlint) rules when creating or editing any Markdown file. The following rules are disabled:

| Rule | Reason |
| --- | --- |
| `MD013` (line length) | Long lines are acceptable |
| `MD033` (no inline HTML) | Inline HTML is permitted |
| `MD045` (no empty alt text) | Alt text is not required on images |

## GitHub Flow

All work follows [GitHub Flow](https://docs.github.com/en/get-started/using-github/github-flow) — a lightweight, branch-based workflow:

1. **Branch** off `main` with a short, descriptive name: `git checkout main && git pull && git checkout -b <branch-name>`
2. **Commit** isolated, complete changes with descriptive messages (see Commit and PR Conventions below). Push early and often to back up work.
3. **Open a pull request** to request review. Use a draft PR for early feedback before the work is complete.
4. **Address review comments** with follow-up commits on the same branch.
5. **Squash and merge** once approved. Every PR lands as a single commit on `main`.
6. **Delete the branch** locally and remotely after merging.

## Commit and PR Conventions

- **No Conventional Commits** — do not use `feat:`, `fix:`, `chore:`, `refactor:` prefixes
- Write commit messages and PR titles in sentence case natural language
- Use GitHub labels to categorize PRs: `bug`, `chore`, `documentation`, `enhancement`, `security`, `tech-debt`

✅ `Improve metadata validation for GKE handler`
❌ `feat: add metadata endpoint`

## Workspace Workflow

Delete local branches after PRs are merged. When performing bulk operations across repos, verify each repo's unique structure before applying changes.

**Instruction file structure:** Instructions are organized at three levels:
- **Platform-level** — `pt-ai-context/.github/instructions/team.instructions.md` (this file)
- **Team-level** — `pt-*-ai-context/.github/instructions/team.instructions.md` (one per team)
- **Repo-level** — `.github/copilot-instructions.md` in each repository

When adding a new repository, create `.github/copilot-instructions.md` with a brief description of what the repo does.
