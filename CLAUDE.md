# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is the `.github` org-level repository for **Magnitude Instruments**. It contains organization-wide standards, templates, and conventions that apply across all Magnitude Instruments GitHub repositories. There is no buildable code here — only documentation and templates.

## Structure

- `CONTRIBUTING.md` — Git Flow branching strategy, commit conventions, development workflow
- `.github/PULL_REQUEST_TEMPLATE.md` — Default PR template for all org repos
- `.github/ISSUE_TEMPLATE/` — Bug report and feature request templates
- `cpp/CODING_STYLE.md` — Naming, formatting, class layout, modern C++ conventions
- `cpp/CMAKE_CONVENTIONS.md` — CMake target naming, components, versioning, build presets
- `cpp/TESTING_STANDARDS.md` — Google Test conventions, mocking strategies, test organization
- `cpp/NAVIGATION_TEMPLATES.md` — Documentation structure and navigation patterns for library repos

## Org-Wide Conventions

### Branching (Git Flow)
- **main** — Tagged releases only
- **develop** — Integration branch (default PR target)
- **feature/**, **hotfix/**, **release/** — Standard Git Flow branches

### Commit Messages
Conventional Commits format: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`

### C++ Standards
- C++23, MSVC (Visual Studio 2026, v145 toolset)
- CMake with presets, Google Test, vcpkg
- Target naming: `MagInsts::<Repo>::<Component>` (e.g., `MagInsts::Hardware::USB`)
- Semantic versioning (MAJOR.MINOR.PATCH)

### Library Dependency Graph
```
maginsts-hardware-cpp  → maginsts-common-cpp
maginsts-network-cpp   → maginsts-common-cpp
```

### Standard Build Commands (for library repos, not this repo)
```bash
cmake --preset debug
cmake --build build/debug --config Debug
ctest --test-dir build/debug -C Debug --output-on-failure
```

## Editing Guidelines

When modifying files in this repo, maintain consistency with:
- Existing markdown formatting and heading styles
- Breadcrumb navigation patterns in `cpp/NAVIGATION_TEMPLATES.md`
- Conventional Commits for any commits to this repo
- The include style convention: `<maginsts/...>` for external deps, `"maginsts/..."` for own project, `"Relative.hpp"` for same module
