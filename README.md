# Platform Group AI Context

Platform Group Copilot instructions for the [osinfra-io](https://github.com/osinfra-io) platform. This is the group-level layer of the instruction hierarchy and applies universally to every `pt-*` repository across all platform teams.

## Overview

This repository is the **Platform Group** level of a three-level GitHub Copilot instruction hierarchy. Instructions here apply universally to every `pt-*` repository across all platform teams.

```none
Platform Group   pt-ai-context                   ← this repo (applies to all pt-* repos)
  └── Platform Team   pt-*-ai-context             ← one per team (applies to that team's repos)
        └── Repository   .github/copilot-instructions.md   ← in every repo (repo-specific only)
```

## Workspace repository tree

This is the on-demand reference copy of the current multi-root `platform-group/` workspace inventory. Keep it up to date when repositories are added or removed.

```none
platform-group/
├── pt-ai-context/                      # platform group instructions (COPILOT_CUSTOM_INSTRUCTIONS_DIRS)
├── pt-ai-plugins/                      # platform group Copilot CLI plugins + marketplace
├── arche/
│   ├── pt-arche-ai-context/            # team instructions (COPILOT_CUSTOM_INSTRUCTIONS_DIRS)
│   ├── pt-arche-child-module-template/
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
│   └── pt-ekklesia-docs/
├── logos/
│   ├── pt-logos-ai-context/            # team instructions (COPILOT_CUSTOM_INSTRUCTIONS_DIRS)
│   └── pt-logos/
├── pneuma/
│   ├── pt-pneuma-ai-context/           # team instructions (COPILOT_CUSTOM_INSTRUCTIONS_DIRS)
│   ├── pt-pneuma/
│   └── pt-pneuma-istio-test/
└── techne/
    ├── pt-techne-agents/
    ├── pt-techne-ai-context/           # team instructions (COPILOT_CUSTOM_INSTRUCTIONS_DIRS)
    ├── pt-techne-development-setup/
    ├── pt-techne-mcp-server/
    ├── pt-techne-misc-workflows/
    ├── pt-techne-opentofu-codespace/
    ├── pt-techne-opentofu-workflows/
    └── pt-techne-pre-commit-hooks/
```

## Setup

`COPILOT_CUSTOM_INSTRUCTIONS_DIRS` tells the GitHub Copilot CLI which directories to load custom instructions from at startup. Set it to a comma-separated list of absolute paths — no spaces around commas.

The workspace is cloned at `~/repositories/osinfra-io/platform-group/`. Each ai-context repo lives within a team subdirectory:

```none
~/repositories/osinfra-io/platform-group/
├── pt-ai-context/                        ← platform group (always include)
├── arche/pt-arche-ai-context/
├── corpus/pt-corpus-ai-context/
├── ekklesia/pt-ekklesia-ai-context/
├── logos/pt-logos-ai-context/
├── pneuma/pt-pneuma-ai-context/
└── techne/pt-techne-ai-context/
```

Always include `pt-ai-context` plus the ai-context repo for your team. If you work across multiple teams, include all relevant team repos.

### Single team (most common)

```bash
# ~/.zshrc — replace <team> with your team name
export COPILOT_CUSTOM_INSTRUCTIONS_DIRS="\
$HOME/repositories/osinfra-io/platform-group/pt-ai-context,\
$HOME/repositories/osinfra-io/platform-group/<team>/pt-<team>-ai-context"
```

| Team | Path segment |
| --- | --- |
| arche | `arche/pt-arche-ai-context` |
| corpus | `corpus/pt-corpus-ai-context` |
| ekklesia | `ekklesia/pt-ekklesia-ai-context` |
| logos | `logos/pt-logos-ai-context` |
| pneuma | `pneuma/pt-pneuma-ai-context` |
| techne | `techne/pt-techne-ai-context` |

### All teams

```bash
# ~/.zshrc
export COPILOT_CUSTOM_INSTRUCTIONS_DIRS="\
$HOME/repositories/osinfra-io/platform-group/pt-ai-context,\
$HOME/repositories/osinfra-io/platform-group/arche/pt-arche-ai-context,\
$HOME/repositories/osinfra-io/platform-group/corpus/pt-corpus-ai-context,\
$HOME/repositories/osinfra-io/platform-group/ekklesia/pt-ekklesia-ai-context,\
$HOME/repositories/osinfra-io/platform-group/logos/pt-logos-ai-context,\
$HOME/repositories/osinfra-io/platform-group/pneuma/pt-pneuma-ai-context,\
$HOME/repositories/osinfra-io/platform-group/techne/pt-techne-ai-context"
```

After editing your shell profile, reload it:

```bash
source ~/.zshrc
```

> **Note:** `COPILOT_CUSTOM_INSTRUCTIONS_DIRS` is shell-scoped. It only exists in shells that
> have sourced the profile where it is exported (e.g. `~/.zshrc`). If Copilot is launched from
> a context that does not load that profile — a different shell (bash), a GUI launcher, or an
> automation/CI step — the variable is unset and **none** of these custom instructions load,
> which looks like Copilot "ignoring" them. Export it from a profile that every shell you use
> to launch Copilot will source.

## Plugins and marketplace

Custom instructions are the **always-on** layer. Installable Copilot CLI **capabilities** — skills, agents, and MCP servers — are distributed separately through the complementary [`pt-ai-plugins`](https://github.com/osinfra-io/pt-ai-plugins) repository and its `osinfra-io` marketplace.

Plugins do not replace instructions: the `plugin.json` manifest has no field for `copilot-instructions.md` or `*.instructions.md`, so the hierarchy described above is unaffected. Use `COPILOT_CUSTOM_INSTRUCTIONS_DIRS` for instructions and the marketplace for capabilities:

```bash
copilot plugin marketplace add osinfra-io/pt-ai-plugins
copilot plugin marketplace browse osinfra-io
```
