# Documentation Navigation Templates

Standard navigation patterns for MagInsts C++ library documentation. All library repos (Common, Network, Hardware) follow these conventions.

## Documentation Structure

Every library repo has a `docs/` folder. The structure depends on the repo's complexity.

### Simple repos (e.g., Common)

Repos with a single component use a flat layout:

```
docs/
├── README.md                          # Docs home
├── api/
│   ├── README.md                      # API reference index
│   └── ComponentName.md               # Per-class API docs
└── guides/
    ├── implementation/
    │   └── component-implementation.md
    └── testing/
        └── component-testing.md
```

### Multi-section repos (e.g., Hardware, Network)

Repos with multiple distinct sections group docs by section, each with its own `api/` and `guides/`:

```
docs/
├── README.md                          # Docs home (links to all sections)
├── section-a/
│   ├── README.md                      # Section overview
│   ├── api/
│   │   ├── README.md
│   │   └── Component.md
│   └── guides/
│       ├── implementation/
│       └── testing/
└── section-b/
    ├── README.md
    ├── api/
    └── guides/
```

The key rule: **each section mirrors the same `api/` + `guides/` pattern**. Sections should match the code architecture (e.g., Hardware uses `core/`, `transport/`, `devices/`).

## Header Navigation

Every documentation file starts with a breadcrumb navigation header showing its location in the hierarchy. The header always starts with `[Docs Home]` linking to the root `docs/README.md`.

### Docs Home (`docs/README.md`)

```markdown
# MagInsts-LibraryName Documentation

Brief description.
```

No navigation header needed — this IS the home.

### Section Home (`docs/section/README.md`)

```markdown
[Docs Home](../README.md) | **Section Name**

---

# Section Name
```

### API Index (`docs/[section/]api/README.md`)

```markdown
[Docs Home](../../README.md) | [Section](../README.md) | **API Reference**

---

# API Reference
```

For simple repos without sections, omit the section breadcrumb:

```markdown
[Docs Home](../README.md) | **API Reference**

---

# API Reference
```

### API Page (`docs/[section/]api/Component.md`)

```markdown
[Docs Home](../../README.md) | [Section](../README.md) | [API Reference](README.md) | **Component**

---

# Component API Reference
```

### Guide Page (`docs/[section/]guides/implementation/component.md`)

```markdown
[Docs Home](../../../README.md) | [Section](../../README.md) | **Implementation Guide**

---

# Component Implementation Guide
```

For simple repos without sections:

```markdown
[Docs Home](../../README.md) | **Implementation Guide**

---

# Component Implementation Guide
```

## Footer Navigation

Every documentation file ends with a See Also section and back-navigation.

### Standard Footer

```markdown
---

## See Also

- [Related Component](./Related.md) - Brief description
- [Implementation Guide](../guides/implementation/component-implementation.md)

---

[Back to API Reference](README.md) | [Docs Home](../../README.md)
```

### Minimal Footer

```markdown
---

[Back to Docs Home](../README.md)
```

Adjust relative paths based on the file's depth. The footer should always include a link back to the parent section and to Docs Home.

## Path Reference

The `[Docs Home]` link always points to `docs/README.md`. Calculate the relative path based on depth:

| Depth from `docs/` | `[Docs Home]` path |
|---------------------|-------------------|
| `docs/` | `README.md` |
| `docs/api/` or `docs/section/` | `../README.md` |
| `docs/section/api/` or `docs/guides/impl/` | `../../README.md` |
| `docs/section/guides/impl/` | `../../../README.md` |

## Include Path Convention in Examples

Use the correct include style in all code examples:

```cpp
// External dependency (via vcpkg) - angle brackets
#include <maginsts/common/Logger.hpp>

// Own project headers - quotes
#include "maginsts/network/server/Server.hpp"

// Relative within same module
#include "ServerMetrics.hpp"
```

## Guidelines

- **Breadcrumbs match structure**: Header nav should reflect the actual directory hierarchy
- Every page links back to at least its parent section
- README.md files serve as navigation hubs for their section
- Use descriptive link text, not "click here"
- Update parent README.md when adding new documentation
- Keep navigation breadcrumbs to 4 levels max (Home | Section | SubSection | **Page**)
