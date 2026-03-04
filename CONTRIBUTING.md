# Contributing to Magnitude Instruments

## Branching Strategy

We follow [Git Flow](https://nvie.com/posts/a-successful-git-branching-model/):

```
main          ← Tagged releases only
develop       ← Integration branch (default)
feature/*     ← New features, branch from develop
hotfix/*      ← Urgent fixes, branch from main, merge into both main and develop
release/*     ← Release prep, branch from develop, merge into both main and develop
```

### Day-to-Day Development

1. Branch from `develop`:
   ```bash
   git checkout develop
   git pull
   git checkout -b feature/my-feature
   ```
2. Make your changes, build, and run tests
3. Commit with a clear message (see below)
4. Open a pull request targeting `develop`

### Releasing a Version

1. Branch from `develop`:
   ```bash
   git checkout -b release/v1.2.0 develop
   ```
2. Bump version in `CMakeLists.txt`, update `CHANGELOG.md`
3. PR into `main`, merge, and tag:
   ```bash
   git tag v1.2.0
   ```
4. Merge `main` back into `develop` to pick up the version bump
5. Publish updated packages (maintainers only)

### Hotfixes

For urgent fixes to a released version:

1. Branch from `main`:
   ```bash
   git checkout -b hotfix/fix-description main
   ```
2. Fix the issue, bump patch version
3. PR into `main`, merge, and tag
4. Merge `main` back into `develop`

## Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
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
