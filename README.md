# .github

Organization-wide standards, templates, and development conventions for [Magnitude Instruments](https://magnitudeinstruments.com).

## Org-Wide Defaults

These files apply to all repositories in the organization unless overridden by a repo's own version:

- [CONTRIBUTING.md](CONTRIBUTING.md) — Git Flow branching strategy, commit conventions, development workflow
- [PULL_REQUEST_TEMPLATE.md](.github/PULL_REQUEST_TEMPLATE.md) — Default pull request template
- [ISSUE_TEMPLATE/](.github/ISSUE_TEMPLATE/) — Bug report and feature request templates

## C++ Standards

- [Coding Style](cpp/CODING_STYLE.md) — Naming, formatting, class layout, modern C++ conventions
- [CMake Conventions](cpp/CMAKE_CONVENTIONS.md) — Target naming, components, versioning, build presets
- [Testing Standards](cpp/TESTING_STANDARDS.md) — Google Test conventions, mocking strategies, test organization
- [Navigation Templates](cpp/NAVIGATION_TEMPLATES.md) — Documentation structure and navigation patterns for library repos

## Tech Stack

- **C++23** with MSVC (Visual Studio 2026, v145 toolset)
- **CMake** with presets and vcpkg integration
- **Google Test** for unit testing
- **Semantic Versioning** (MAJOR.MINOR.PATCH)

## Standard Build Pattern

All C++ library repos follow the same build commands:

```bash
cmake --preset debug
cmake --build build/debug --config Debug
ctest --test-dir build/debug -C Debug --output-on-failure
```
