# Changelog

Before running `cage-release`, add your notes under `## Unreleased`:

    ## Unreleased

    - feat: description of new feature
    - fix: description of bug fix

---

## Unreleased

- feat: switch from npm to native Anthropic installer for Claude Code
- feat: smart version check in cage-update — skips rebuild when already up-to-date, prompts with `[y/N]`
- feat: `--force` flag on cage-update to skip version prompt

## v0.1.1 (2026-02-28)

- feat: support Homebrew installation path for Dockerfile
- fix: find Dockerfile via relative path from script, not brew --prefix

## v0.1.0 (2026-02-28)

- Initial release
- feat: include repo name in container names
- feat: ensure skipDangerousModePermissionPrompt is set in entrypoint
- fix: resolve symlinks in cage-update to find Dockerfile correctly
