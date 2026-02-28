---
allowed-tools: Bash(git *), Bash(gh *), Bash(curl *), Bash(shasum *), Bash(date *), Bash(rm *), Read, Edit, Write
---

# Release cage

Perform a full release of the cage project: stamp CHANGES.md, tag and push, create a GitHub release, and update the homebrew-tap formula.

The cage repo is at the current working directory. The homebrew-tap is at `../homebrew-tap` relative to the cage repo root.

---

## Step 1: Gather current state (run in parallel)

- `git describe --tags --abbrev=0` — last version tag
- `git status --porcelain` — uncommitted changes in cage repo
- `git -C ../homebrew-tap status --porcelain` — uncommitted changes in tap
- `date +%Y-%m-%d` — today's date
- Read `CHANGES.md`

## Step 2: Verify clean working trees

If either `git status` shows output, stop and tell the user to commit or stash those changes first.

## Step 3: Show recent commits and propose a version

Run `git log <LAST_TAG>..HEAD --oneline`.

Analyze the commit messages:
- If any line starts with `feat:` or `feat(` → suggest a **minor** bump
- Otherwise → suggest a **patch** bump

Compute the suggested version by splitting the last tag on `.` and incrementing the appropriate component. Strip any leading `v` before splitting.

Present the commits and your suggested version, then ask the user to confirm or enter a different version. Wait for their response before continuing.

## Step 4: Verify CHANGES.md has release notes

Re-read `CHANGES.md` if needed. There must be an `## Unreleased` section with at least one non-blank line of content beneath it.

If the section is missing or empty, stop and explain the required format:

```
## Unreleased

- feat: description of change
- fix: description of bug fix
```

Do not proceed until the user has added notes and asked you to continue.

## Step 5: Stamp the version in CHANGES.md

Use the Edit tool to replace `## Unreleased` with `## v{VERSION} ({TODAY})` in `CHANGES.md`.

## Step 6: Commit, tag, and push

Run each command separately and in order:

```
git add CHANGES.md
git commit -m "chore: release v{VERSION}"
git tag v{VERSION}
git push origin main
git push origin v{VERSION}
```

## Step 7: Create GitHub release

Read `CHANGES.md`. Extract all lines between the `## v{VERSION}` heading and the next `## v` heading — these are the release notes for this version.

Write the extracted lines to `/tmp/cage-release-notes.md` using the Write tool.

Then run:
```
gh release create v{VERSION} --repo sschlesier/cage --title v{VERSION} --notes-file /tmp/cage-release-notes.md
```

Then remove the temp file:
```
rm /tmp/cage-release-notes.md
```

## Step 8: Compute sha256 of the release tarball

Run these in order:
```
curl -fsSL -o /tmp/cage-release.tar.gz https://github.com/sschlesier/cage/archive/refs/tags/v{VERSION}.tar.gz
shasum -a 256 /tmp/cage-release.tar.gz
rm /tmp/cage-release.tar.gz
```

The sha256 is the first field of the `shasum` output.

## Step 9: Update the homebrew formula

Read `../homebrew-tap/Formula/cage.rb`.

Use the Edit tool to:
1. Replace the `url` line with:
   `  url "https://github.com/sschlesier/cage/archive/refs/tags/v{VERSION}.tar.gz"`
2. Replace the `sha256` line with:
   `  sha256 "{SHA256}"`

## Step 10: Commit and push the homebrew tap

Run each command separately:
```
git -C ../homebrew-tap add Formula/cage.rb
git -C ../homebrew-tap commit -m "feat: bump cage to v{VERSION}"
git -C ../homebrew-tap push origin main
```

Report that the release is complete, showing the version and the GitHub release URL returned by `gh`.
