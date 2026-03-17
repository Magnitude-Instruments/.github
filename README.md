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

## AWS Standards

- [SAM Conventions](aws/SAM_CONVENTIONS.md) — Project structure, naming, template patterns, deployment workflow
- [Python Lambda Style](aws/PYTHON_LAMBDA_STYLE.md) — Handler patterns, logging, error handling, SES emails

## React Standards

- [React Conventions](react/REACT_CONVENTIONS.md) — Component patterns, state management, styling, accessibility, API integration

## Tech Stacks

### C++
- **C++23** with MSVC (Visual Studio 2026, v145 toolset)
- **CMake** with presets and vcpkg integration
- **Google Test** for unit testing

### AWS / Backend
- **Python 3.12** Lambda functions via AWS SAM
- **API Gateway** with Cognito JWT authorization
- **SES** for transactional emails
- **S3**, **DynamoDB**, **Secrets Manager**

### Frontend
- **React 18** (JavaScript) with Vite
- **AWS Amplify v6** for Auth and API modules
- **MUI 5** for UI components
- **AWS Amplify Hosting** for CI/CD

## Standard Build Patterns

### C++ Libraries
```bash
cmake --preset debug
cmake --build build/debug --config Debug
ctest --test-dir build/debug -C Debug --output-on-failure
```

### SAM Projects
```bash
sam build --use-container
sam deploy
```

### React Apps
```bash
npm start        # Dev server
npm run build    # Production build
```
