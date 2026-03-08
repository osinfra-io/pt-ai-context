# pt-ai-context

Platform-level Copilot instructions for the [osinfra-io](https://github.com/osinfra-io) platform teams workspace.

## Overview

This repository is the **platform level** of a three-level GitHub Copilot instruction hierarchy. Instructions here apply universally to every `pt-*` repository across all teams.

```none
Platform   pt-ai-context                   ← this repo (applies to all pt-* repos)
  └── Team   pt-*-ai-context               ← one per team (applies to that team's repos)
        └── Repo   .github/copilot-instructions.md   ← in every repo (repo-specific only)
```

## What's in this repo

`.github/instructions/team.instructions.md` — loaded via `COPILOT_CUSTOM_INSTRUCTIONS_DIRS`. Contains conventions shared across the entire platform:

- Workspace structure and repository tree
- Platform architecture and team descriptions
- Code quality principles
- OpenTofu file structure and ordering rules
- Pre-commit workflow
- Key conventions (module pinning, conditional resources, variables/outputs, GitHub Actions SHAs, Mermaid diagrams)
- GitHub Flow
- Commit and PR conventions

## Setup

Add this repo to your `COPILOT_CUSTOM_INSTRUCTIONS_DIRS` environment variable alongside your team's ai-context repo:

```bash
# ~/.zshrc or ~/.bashrc
export COPILOT_CUSTOM_INSTRUCTIONS_DIRS="\
$HOME/repositories/osinfra-io/platform-teams/pt-ai-context,\
$HOME/repositories/osinfra-io/platform-teams/<team>/pt-<team>-ai-context"
```

See the team-level ai-context repos for the path specific to your team.
