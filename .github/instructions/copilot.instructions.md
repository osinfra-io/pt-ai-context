# GitHub Copilot Workspace Instructions

## Workspace Overview

This VS Code workspace aggregates all platform team repositories into a single multi-root workspace. Each top-level directory is a collection of related repositories. **Keep this tree up to date when repositories are added or removed.**

```
platform-teams/
├── pt-ai-context/                      # platform instructions (COPILOT_CUSTOM_INSTRUCTIONS_DIRS)
├── arche/
│   ├── pt-arche-ai-context/            # team instructions (COPILOT_CUSTOM_INSTRUCTIONS_DIRS)
│   ├── pt-arche-core-helpers/
│   ├── pt-arche-datadog-google-integration/
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
    ├── pt-techne-misc-workflows/
    ├── pt-techne-opentofu-codespace/
    ├── pt-techne-opentofu-workflows/
    └── pt-techne-pre-commit-hooks/
```

Every non-`ai-context` repository has a `.github/copilot-instructions.md` file with repo-specific context, loaded automatically by Copilot.

## Platform Architecture

The platform is organized into teams (a domain of concern that may map to individual teams, repositories, and infrastructure boundaries — or in the case of a small platform team, all of them), each with a distinct role. Deployment dependencies between the infrastructure layers are enforced at the GitHub Actions workflow level.

- **Logos** — *"The foundational principle of order across systems, integrating multi-provider infrastructure, establishing boundaries, governance, and stable standards for teams to operate autonomously."* Creates GCP folder hierarchy, Google Identity groups, GitHub teams and repositories, and Datadog teams. Everything downstream depends on its outputs.

- **Corpus** — *"The embodiment of that order — the structural form where networks, shared services, and core infrastructure take shape, preparing the body that Pneuma will animate."* Translates Logos structure into real infrastructure: GCP projects with CIS compliance, shared VPC and subnets, DNS zones, Artifact Registry, GitHub Actions service accounts, workload identity pools, and encrypted state buckets.

- **Pneuma** — *"The breath of life animating the platform via Kubernetes, orchestrating dynamic, self-healing, and scalable services atop the Logos foundation."* Animates Corpus projects into workload environments: GKE clusters across multiple zones, cert-manager, Istio service mesh, Datadog cluster monitoring, and OPA Gatekeeper policy enforcement.

- **Arche** — *"The origin and first cause — the primordial source from which all platform foundations draw their initial form and essential nature."* Reusable OpenTofu child modules consumed by all three infrastructure layers. Covers GCP projects, GKE, networking, storage, Datadog integration, and all Kubernetes add-ons. The `pt-arche-core-helpers` module is foundational — providing environment detection, labels, and team naming. Check `helpers.tofu` before hardcoding any of those values.

- **Ekklesia** — *"The assembly of the called-out — where distinct capabilities are gathered into a unified body, deliberating and acting in concert toward shared platform purpose."* The developer portal and documentation layer: platform docs, Backstage running on GKE, and repository templates for new IaC projects.

- **Techne** — *"The practiced art of making — the disciplined craft through which raw materials of infrastructure are shaped into purposeful, refined platform instruments."* Shared tooling: reusable GitHub Actions called workflows for OpenTofu deployments (OIDC auth, state encryption, job summaries), pre-commit hooks for IaC validation, and a GitHub Codespace for standardized developer environments.

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
|---|---|
| `main.tofu` | Resource and module declarations |
| `variables.tofu` | Input variables with descriptions and validation |
| `outputs.tofu` | Output values |
| `locals.tofu` | Complex data transformations |
| `helpers.tofu` | Invokes `pt-arche-core-helpers`; provides env, labels, project naming |
| `data.tofu` | Data source definitions |
| `providers.tofu` | Provider configuration |
| `backend.tofu` | State backend configuration |
| `moved.tofu` | Resource renames and moves |

Every `.tofu` file begins with a comment block:

```hcl
# Output Values
# https://opentofu.org/docs/language/values/outputs
```

**Ordering rules (strictly enforced by pre-commit):**
- Variables, outputs, locals, `.tfvars` entries: alphabetical
- Resources and data sources: alphabetical by resource type
- All arguments within a block: alphabetical
- Meta-arguments (`count`, `depends_on`, `for_each`, `lifecycle`, `provider`) come first, alphabetically among themselves

## Pre-Commit Workflow (Mandatory)

**`pre-commit run -a` must be run after ANY change in this workspace.** Do not wait to be asked.

Run `pre-commit autoupdate --freeze` once at the start of a session before the first `pre-commit run -a`.

```bash
pre-commit autoupdate --freeze   # run once per session to update hooks and pin to commit SHAs
pre-commit run -a                # run after every change; enforces fmt, validate, docs, security
```

Pre-commit hooks enforce: `tofu fmt`, `tofu validate`, `tofu test`, YAML validation, documentation generation, and trailing whitespace fixes.

## Key Conventions

### Module Source Pinning

Always pin module sources to a full 40-character commit SHA with an inline version comment:

```hcl
module "helpers" {
  source = "github.com/osinfra-io/pt-arche-core-helpers//child?ref=<commit_sha>"  # v1.2.3
}
```

Never use branch names or semver tags as `ref` values.

### Conditional Resources

Prefer `enabled` (OpenTofu v1.11+) over `count` for conditional resource creation:

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

### Mermaid Diagrams

- Use `graph LR` (left-to-right) layout
- Always include `color:#000` for text readability on colored backgrounds
- Color palette (use consistently; refer to the existing diagram in each repo's README for how they're applied):
  `#fff4e6`, `#d4edda`, `#e6d9f5`, `#d1ecf1`, `#fff3cd`, `#f8d7e5`, `#ffdab9`
- Keep diagrams in sync with actual workflow job structure when modifying workflows

## GitHub Flow

All work follows [GitHub Flow](https://docs.github.com/en/get-started/using-github/github-flow) — a lightweight, branch-based workflow:

1. **Branch** off `main` with a short, descriptive name: `git checkout main && git pull && git checkout -b <branch-name>`
2. **Commit** isolated, complete changes with descriptive messages (see Commit and PR Conventions below). Push early and often to back up work.
3. **Open a pull request** to request review. Use a draft PR for early feedback before the work is complete.
4. **Address review comments** with follow-up commits on the same branch.
5. **Squash and merge** once approved. Every PR lands as a single commit on `main`.
6. **Delete the branch** locally and remotely after merging.

Never commit directly to `main`.

## Commit and PR Conventions

- **No Conventional Commits** — do not use `feat:`, `fix:`, `chore:`, `refactor:` prefixes
- Write commit messages and PR titles in sentence case natural language
- Use GitHub labels (`enhancement`, `bug`, `refactor`, `docs`) to categorize PRs

✅ `Improve metadata validation for GKE handler`
❌ `feat: add metadata endpoint`

## Workspace Workflow

Always sync before branching:

```bash
git checkout main && git pull && git checkout -b <branch-name>
```

Delete local branches after PRs are merged. When performing bulk operations across repos, verify each repo's unique structure before applying changes.

**Instruction file structure:** Instructions are organized at three levels:
- **Platform-level** — `pt-ai-context/.github/instructions/copilot.instructions.md` (this file)
- **Team-level** — `pt-*-ai-context/.github/instructions/copilot.instructions.md` (one per team)
- **Repo-level** — `.github/copilot-instructions.md` in each repository

Platform and team instructions are loaded via `COPILOT_CUSTOM_INSTRUCTIONS_DIRS`. Set this env var in your shell profile to a comma-separated list of the platform context repo and your team's ai-context repo:

```bash
# In ~/.zshrc or ~/.bashrc — adjust the team path to match your team
export COPILOT_CUSTOM_INSTRUCTIONS_DIRS="\
$HOME/repositories/osinfra-io/platform-teams/pt-ai-context,\
$HOME/repositories/osinfra-io/platform-teams/logos/pt-logos-ai-context"
```

Team paths by team:
- `arche` → `arche/pt-arche-ai-context`
- `logos` → `logos/pt-logos-ai-context`
- `corpus` → `corpus/pt-corpus-ai-context`
- `pneuma` → `pneuma/pt-pneuma-ai-context`
- `ekklesia` → `ekklesia/pt-ekklesia-ai-context`
- `techne` → `techne/pt-techne-ai-context`

Repo-level instructions (`.github/copilot-instructions.md`) are loaded automatically by Copilot for each repository. When adding a new repository, create this file with a brief description of what the repo does.

## References

- [OpenTofu Documentation](https://opentofu.org/docs/)
- [OpenTofu conventions for this workspace](./../arche/pt-arche-core-helpers/.github/skills/opentofu.md)
- [Repository instructions documentation](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-custom-instructions)
