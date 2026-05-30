# Platform Group AI Context

Platform Group Copilot instructions for the [osinfra-io](https://github.com/osinfra-io) platform. This is the group-level layer of the instruction hierarchy and applies universally to every platform team's repositories.

## Overview

This repository is the **Platform Group** level of a three-level GitHub Copilot instruction hierarchy. Instructions here apply universally to every `pt-*` repository across all platform teams.

```none
Platform Group   pt-ai-context                   ← this repo (applies to all pt-* repos)
  └── Platform Team   pt-*-ai-context             ← one per team (applies to that team's repos)
        └── Repository   .github/copilot-instructions.md   ← in every repo (repo-specific only)
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
