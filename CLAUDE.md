# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

**Cage** runs Claude Code inside an isolated Linux VM using Apple's Virtualization Framework (`container` CLI). It mounts only the current project directory into the container, keeping the rest of the host filesystem inaccessible.

## Setup & Usage

```bash
brew install container
container system start
./cage-update    # Build the Docker image
./cage-login     # Authenticate with Anthropic
./cage           # Run Claude Code in isolation
./cage-shell     # Open a shell into a running cage container
```

## Scripts

| Script | Purpose |
|--------|---------|
| `cage` | Main entry point — starts a container and runs Claude Code |
| `cage-shell` | Shells into an existing container for the current branch |
| `cage-login` | Creates/uses `claude-creds` volume and runs `claude login` |
| `cage-update` | Rebuilds the Docker image, installing latest `@anthropic-ai/claude-code` |
| `entrypoint.sh` | Container startup: syncs host config (read-only) into the container |

## Architecture

**Container naming**: `cage-{sanitized-branch-name}[-N]` — derived from git branch, with numeric suffix to avoid collisions.

**Directory mounting**: The current working directory mounts at `/home/claude/workspace`. For git worktrees, both the worktree directory and its common `.git` dir are mounted.

**Config sync**: On startup, `entrypoint.sh` copies `CLAUDE.md`, `settings.json`, `commands/`, `skills/`, and `plugins/` from the host's `~/.claude` (mounted read-only at `/tmp/.claude-host`) into the container's `~/.claude/`.

**Credentials**: Stored in a persistent named volume `claude-creds`, mounted at `~/.claude` before config sync. Login state is preserved across container restarts.

**Git identity**: `GIT_AUTHOR_NAME`, `GIT_AUTHOR_EMAIL`, `GIT_COMMITTER_NAME`, `GIT_COMMITTER_EMAIL` are passed from the host into the container.

## Docker Image

Base: `node:22-slim`. Installs `git`, `curl`, `ripgrep`, and `@anthropic-ai/claude-code` globally. Runs as non-root user `claude`.

## No Tests or Linter

This project has no test suite or linter. Changes are validated by running the scripts manually.
