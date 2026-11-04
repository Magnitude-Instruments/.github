# Contributing to Magnitude Instruments

## Branching Strategy

We work trunk-based. `main` is the only long-lived branch, and it is always releasable.

```
main                        ← The trunk: every merged change, always releasable
<type>/<issue>-<slug>       ← Short-lived work branch, cut from main, squash-merged by PR
release/<major>.<minor>.x   ← Only when a shipped version needs a fix it can't take from main
```

There is no `develop` branch.

### Day-to-Day Development

1. Branch from `main` by name, not from whatever is checked out:
   ```bash
   git fetch origin
   git switch -c fix/42-usb-reconnect-timeout origin/main
   ```
   The type is one of the commit types below. A branch with no issue is `<type>/<slug>`.
2. Make your changes, build, and run the tests.
3. Commit with a clear message (see below).
4. Open a pull request targeting `main`. It is reviewed, must be green on its last push, and is squash-merged, so the PR title becomes the commit on `main`.

Keep branches short-lived: merge within a day or two rather than letting a branch drift from `main`.

### Releasing a Version

1. On `main`, bump the version (for example in `CMakeLists.txt`) and update `CHANGELOG.md`, through a normal PR.
2. Tag the merged commit with an annotated tag:
   ```bash
   git switch main && git pull
   git tag -a v1.2.0 -m "v1.2.0"
   git push origin v1.2.0
   ```
3. Publish updated packages (maintainers only).

### Fixing a Released Version

Fix it on `main` first and release a new version from there. Only when a shipped version needs a fix it can't take from `main` (for example, `main` has moved on to an incompatible change):

1. Cut a maintenance branch from the release tag, once per minor version:
   ```bash
   git switch -c release/1.2.x v1.2.0
   ```
2. Fix through a PR into `release/1.2.x`, bump the patch version, and tag `v1.2.1` on that branch.
3. Make sure `main` has the same fix.

## Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/). With squash merges, the PR title is the commit, so write the PR title this way. Put the issue number in the scope when there is one:

```
fix(#42): Handle timeout on USB reconnect
feat: Add wavelength calibration support
fix: Handle timeout on USB reconnect
docs: Update COMDevice API reference
refactor: Simplify PnP device enumeration
test: Add edge case tests for CRC validation
chore: Update vcpkg baseline
```

## Code Standards

- **Testing** — New functionality requires unit tests
- **Thread safety** — Public APIs should be thread-safe where applicable
- **Naming** — Follow existing conventions in each repo (see `CLAUDE.md`)

## Language-Specific Standards

- [C++ Conventions](cpp/) — CMake targets, vcpkg patterns, documentation templates
