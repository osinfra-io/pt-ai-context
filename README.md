# Platform Team AI Context

Platform-level Copilot instructions for the [osinfra-io](https://github.com/osinfra-io) platform teams workspace.

## Overview

This repository is the **platform level** of a three-level GitHub Copilot instruction hierarchy. Instructions here apply universally to every `pt-*` repository across all teams.

```none
Platform   pt-ai-context                   ← this repo (applies to all pt-* repos)
  └── Team   pt-*-ai-context               ← one per team (applies to that team's repos)
        └── Repo   .github/copilot-instructions.md   ← in every repo (repo-specific only)
```

## Setup

`COPILOT_CUSTOM_INSTRUCTIONS_DIRS` tells the GitHub Copilot CLI which directories to load custom instructions from at startup. Set it to a comma-separated list of absolute paths — no spaces around commas.

The workspace is cloned at `~/repositories/osinfra-io/platform-teams/`. Each ai-context repo lives within a team subdirectory:

```none
~/repositories/osinfra-io/platform-teams/
├── pt-ai-context/                        ← platform-level (always include)
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
$HOME/repositories/osinfra-io/platform-teams/pt-ai-context,\
$HOME/repositories/osinfra-io/platform-teams/<team>/pt-<team>-ai-context"
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
$HOME/repositories/osinfra-io/platform-teams/pt-ai-context,\
$HOME/repositories/osinfra-io/platform-teams/arche/pt-arche-ai-context,\
$HOME/repositories/osinfra-io/platform-teams/corpus/pt-corpus-ai-context,\
$HOME/repositories/osinfra-io/platform-teams/ekklesia/pt-ekklesia-ai-context,\
$HOME/repositories/osinfra-io/platform-teams/logos/pt-logos-ai-context,\
$HOME/repositories/osinfra-io/platform-teams/pneuma/pt-pneuma-ai-context,\
$HOME/repositories/osinfra-io/platform-teams/techne/pt-techne-ai-context"
```

After editing your shell profile, reload it:

```bash
source ~/.zshrc
```
