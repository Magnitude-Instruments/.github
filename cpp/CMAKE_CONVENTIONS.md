# CMake Conventions

Standard CMake patterns for MagInsts C++ libraries.

## Target Naming

### Pattern

```
Physical target:  MagInsts_<Repo>_<Component>
Alias target:     MagInsts::<Repo>::<Component>
Export name:       <Component>
Namespace:         MagInsts::<Repo>::
```

### Current Targets

| Repo | Physical Target | Consumer Uses |
|------|----------------|---------------|
| Common | `MagInsts_Common` | `MagInsts::Common` |
| Network | `MagInsts_Network_Server` | `MagInsts::Network::Server` |
| Network | `MagInsts_Network_Discovery` | `MagInsts::Network::Discovery` |
| Hardware | `MagInsts_Hardware_PnP` | `MagInsts::Hardware::PnP` |
| Hardware | `MagInsts_Hardware_USB` | `MagInsts::Hardware::USB` |
| Hardware | `MagInsts_Hardware_COM` | `MagInsts::Hardware::COM` |

### Setting Up a Target

```cmake
add_library(MagInsts_Repo_Component STATIC source.cpp)
add_library(MagInsts::Repo::Component ALIAS MagInsts_Repo_Component)
set_target_properties(MagInsts_Repo_Component PROPERTIES EXPORT_NAME Component)
```

## Component Support

Libraries with multiple targets support CMake components so consumers only pull what they need.

### Consumer Usage

```cmake
# Load specific components
find_package(MagInsts-Network REQUIRED COMPONENTS Server)
find_package(MagInsts-Hardware REQUIRED COMPONENTS PnP)

# Load everything (backward compatible)
find_package(MagInsts-Network REQUIRED)
find_package(MagInsts-Hardware REQUIRED)

# Single-target libraries don't need components
find_package(MagInsts-Common REQUIRED)
```

### Install Pattern

Each component gets its own export set:

```cmake
install(TARGETS MagInsts_Repo_ComponentA
    EXPORT MagInsts-Repo-ComponentATargets
    ...)

install(TARGETS MagInsts_Repo_ComponentB
    EXPORT MagInsts-Repo-ComponentBTargets
    ...)

install(EXPORT MagInsts-Repo-ComponentATargets
    FILE MagInsts-Repo-ComponentATargets.cmake
    NAMESPACE MagInsts::Repo::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/MagInsts-Repo)
```

### Config File Pattern

The `Config.cmake.in` handles component loading:

```cmake
@PACKAGE_INIT@
include(CMakeFindDependencyMacro)

# Always-needed dependencies
find_dependency(MagInsts-Common REQUIRED)

set(_supported_components ComponentA ComponentB)

if(NOT PackageName_FIND_COMPONENTS)
    set(PackageName_FIND_COMPONENTS ${_supported_components})
endif()

foreach(_comp IN LISTS PackageName_FIND_COMPONENTS)
    if(_comp STREQUAL "ComponentA")
        find_dependency(SomeDep REQUIRED)  # Component-specific deps
        if(NOT TARGET MagInsts::Repo::ComponentA)
            include("${CMAKE_CURRENT_LIST_DIR}/MagInsts-Repo-ComponentATargets.cmake")
        endif()
        set(PackageName_ComponentA_FOUND TRUE)
    else()
        ...
    endif()
endforeach()

check_required_components(PackageName)
```

## Versioning

- Version defined in root `CMakeLists.txt`: `project(MagInsts-Repo VERSION x.y.z)`
- Follows [Semantic Versioning](https://semver.org/)
- Published to vcpkg registry on each tagged release

| Change Type | Version Bump |
|-------------|-------------|
| Breaking API change | MAJOR |
| New feature (backward compatible) | MINOR |
| Bug fix | PATCH |

## vcpkg Integration

MagInsts libraries are distributed through a vcpkg registry. Consuming projects need a `vcpkg-configuration.json` that references the registry:

```json
{
  "registries": [
    {
      "kind": "git",
      "repository": "<registry-url>",
      "baseline": "<baseline-commit-sha>",
      "packages": [
        "maginsts-common-cpp",
        "maginsts-network-cpp",
        "maginsts-hardware-cpp"
      ]
    }
  ]
}
```

See internal documentation for registry URL and baseline values.

## Build Presets

Standard CMake presets across all repos:

```json
{
  "configurePresets": [
    {
      "name": "debug",
      "generator": "Visual Studio 18 2026",
      "toolset": "v145",
      "binaryDir": "${sourceDir}/build/debug",
      "cacheVariables": {
        "CMAKE_BUILD_TYPE": "Debug"
      }
    }
  ]
}
```

Build commands:
```bash
cmake --preset debug
cmake --build build/debug --config Debug
ctest --test-dir build/debug -C Debug --output-on-failure
```
