# OpenTofu & README Templates (on-demand reference)

Verbatim templates referenced by `platform-group.instructions.md`. This file is **not** auto-loaded — read it only when you actually need the exact markup (writing OpenTofu file headers, resource/module comment headings, or README badges). The rules that govern these templates live in the always-on platform instructions; this file holds only the copy-paste content.

## OpenTofu file headers

Files that begin with a two-line file header (`# <Display Name>` / `# <URL>`):

| File | Header |
| --- | --- |
| `variables.tofu` | `# Input Variables` / `# https://opentofu.org/docs/language/values/variables` |
| `outputs.tofu` | `# Output Values` / `# https://opentofu.org/docs/language/values/outputs` |
| `locals.tofu` | `# Local Values` / `# https://opentofu.org/docs/language/values/locals` |
| `backend.tofu` | `# Backend Configuration` / `# https://opentofu.org/docs/language/settings/backends/configuration` |
| `moved.tofu` | `# Moved Blocks` / `# https://opentofu.org/docs/language/moved` |
| `helpers.tofu` | `# OpenTofu Core Helpers Module (osinfra.io)` / `# https://github.com/osinfra-io/pt-arche-core-helpers` |
| `providers.tofu` | begins directly with a `terraform {}` block — no file header |
| `main.tofu` | no file-level header — begins directly with the first resource or module block comment |
| `data.tofu` | no file-level header — begins directly with the first data source block comment |

## Resource, data, and module comment headings

Each group of blocks sharing the same type (or same module source) is preceded by one comment block:

```hcl
# <Resource, Data Source, or Module Display Name>
# <URL to provider docs or GitHub repo>
# <optional: additional context lines>

resource "google_project" "this" {
```

Multiple `kubernetes_manifest` resources share one heading:

```hcl
# Kubernetes Manifest Resource
# https://search.opentofu.org/provider/hashicorp/kubernetes/latest/docs/resources/manifest

resource "kubernetes_manifest" "istio_gateway" { ... }

resource "kubernetes_manifest" "istio_peer_authentication" { ... }
```

Multiple modules consuming the same source share one heading:

```hcl
# Datadog Google Cloud Platform Integration Module (osinfra.io)
# https://github.com/osinfra-io/pt-arche-datadog-google-integration

module "datadog_google_integration" { ... }

module "datadog_google_integration_team_kubernetes_projects" { ... }

module "datadog_google_integration_team_projects" { ... }
```

Use the provider's documentation URL for resource and data blocks; use the GitHub repo URL for module blocks. Always validate that comment URLs resolve correctly.

## Block formatting example

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

## README badge markdown

Badges appear immediately after the title, ordered as below. Use `style=for-the-badge` on all badges, and include a badge only when the repo has the corresponding workflow or feature.

1. **Copilot Agent** — repos containing `.github/agents/`:

```markdown
[![Copilot Agent](https://img.shields.io/badge/Copilot%20Agent-Enabled-6E40C9?style=for-the-badge&logo=githubcopilot&logoColor=white)](https://github.com/osinfra-io/<repo>/tree/main/.github/agents)
```

2. **OpenTofu Tests** — repos with a `test.yml` workflow:

```markdown
[![OpenTofu Tests](https://img.shields.io/github/actions/workflow/status/osinfra-io/<repo>/test.yml?style=for-the-badge&logo=opentofu&color=FEDA15&label=OpenTofu%20Tests)](https://github.com/osinfra-io/<repo>/actions/workflows/test.yml)
```

3. **Dependabot** — repos with a `dependabot.yml` workflow:

```markdown
[![Dependabot](https://img.shields.io/github/actions/workflow/status/osinfra-io/<repo>/dependabot.yml?style=for-the-badge&logo=github&color=2088FF&label=Dependabot)](https://github.com/osinfra-io/<repo>/actions/workflows/dependabot.yml)
```

4. **Datadog Security** — repos containing IaC (OpenTofu):

```markdown
[![Datadog Security Enabled](https://img.shields.io/badge/Datadog%20Security-Enabled-632CA6?style=for-the-badge&logo=datadog)](https://app.datadoghq.com/security/code-security/repositories?repository_id=<repo>)
```
