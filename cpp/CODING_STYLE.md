# C++ Coding Style

Coding style conventions for Magnitude Instruments C++ libraries.

## Naming

| Element | Convention | Example |
|---------|-----------|---------|
| Classes / Structs | PascalCase | `ServerMetrics`, `HardwareDeviceInfo` |
| Interfaces | `I` prefix + PascalCase | `IHardware`, `IFilterWheel`, `IAutoEndpoint` |
| Methods | PascalCase | `GetPosition()`, `SetWavelength()`, `IsConnected()` |
| Member variables | `m_` prefix + camelCase | `m_deviceInfo`, `m_state`, `m_isMoving` |
| Static members | `s_` prefix + camelCase | `s_dllModule`, `s_systemMutex` |
| Local variables | camelCase | `deviceCount`, `serialNumber` |
| Constants | `UPPER_SNAKE_CASE` or `constexpr` camelCase | `CRC16_CCITT_POLY`, `MDNS_PORT` |
| Enums | `enum class`, PascalCase values | `DeviceState::Connected` |
| Namespaces | PascalCase, hierarchical | `MagInsts::Hardware::PnP` |
| Template params | `T` prefix + descriptive | `TInterface` |
| File names | PascalCase `.hpp`/`.cpp` | `HardwareFactory.hpp`, `COMDevice.cpp` |

### Method Prefixes

- `Get*` — accessor (const)
- `Set*` — mutator
- `Is*` / `Has*` — boolean query (const)
- `Enable*` / `Disable*` — feature toggle
- `On*` — callback registration (returns subscription ID)
- `Remove*Callback` — callback removal

### Parameter Names

Include units in parameter names when the type doesn't convey them:

```cpp
void SetWavelength(double wavelength_nm);
void SetSlitWidth(double width_um);
void SetReadTimeout(int milliseconds);
```

## Formatting

- **Indentation**: Tabs for indentation, spaces for alignment
- **Brace style**: Allman — opening brace on its own line for classes, functions, and control flow

```cpp
class Server
{
public:
    void Start()
    {
        if (m_isRunning)
        {
            return;
        }
    }
};
```

- **Trivial single-line accessors**: Allowed inline when the body is a single return or assignment

```cpp
// Single-line OK for trivial accessors
const std::string& Name() const { return m_name; }
bool IsEmpty() const { return m_items.empty(); }

// Non-trivial methods must use Allman
void SetName(const std::string& name)
{
    std::lock_guard lock(m_mutex);
    m_name = name;
    NotifyChanged();
}
```

- **Lambdas**: Inline lambdas passed as function arguments may use K&R braces. Standalone lambda variables use Allman.

```cpp
// Inline lambda argument — K&R OK
list.Add([](int val) {
    DoSomething(val);
});

// Standalone lambda variable — Allman
auto handler = [](int val)
{
    DoSomething(val);
};
```

- **Constructor initializer lists**: One member per line, colon/comma aligned

```cpp
JsonConfig::JsonConfig(JsonConfig&& other) noexcept
    : m_filepath(std::move(other.m_filepath))
    , m_defaults(std::move(other.m_defaults))
    , m_enableBackups(other.m_enableBackups)
{
```

- **Namespace closing**: Always comment with full namespace path

```cpp
} // namespace MagInsts::Hardware::PnP
```

## Header Files

### Guards

Use `#pragma once` (not `#ifndef` guards):

```cpp
#pragma once
```

### Include Order

**In header files (`.hpp`)**:

1. Project headers (angle brackets for dependencies, quotes for local)
2. Standard library headers
3. Third-party headers

```cpp
#pragma once

#include <maginsts/common/Logger.hpp>
#include <maginsts/common/Exceptions.hpp>
#include "ServerMetrics.hpp"

#include <string>
#include <mutex>
#include <atomic>

#include <nlohmann/json.hpp>
#include <boost/asio.hpp>
```

**In source files (`.cpp`)**: The corresponding header is always first. This verifies the header is self-contained (includes everything it needs).

```cpp
#include "JsonConfig.hpp"  // Corresponding header FIRST

#include "Logger.hpp"      // Other project headers
#include "Exceptions.hpp"

#include <string>          // Standard library
#include <mutex>

#include <nlohmann/json.hpp>  // Third-party
```

Separate groups with a blank line. Within each group, order is flexible but keep related headers together.

### Windows Headers

Guard Windows includes to avoid macro pollution:

```cpp
#ifndef WIN32_LEAN_AND_MEAN
#define WIN32_LEAN_AND_MEAN
#endif
#include <windows.h>
```

## Namespaces

Mirror directory structure. Use C++17 nested namespace syntax:

```cpp
namespace MagInsts::Hardware::FilterWheel::Thorlabs
{
    // ...
} // namespace MagInsts::Hardware::FilterWheel::Thorlabs
```

Namespace aliases for convenience in implementation files:

```cpp
namespace fs = std::filesystem;
namespace beast = boost::beast;
```

## Class Layout

### Access Specifier Order

1. `protected:` — member variables (when derived classes need access)
2. `public:` — constructors, destructor, public API
3. `protected:` — helper methods for derived classes
4. `private:` — implementation details

> **Rationale**: Protected data members are listed first so that derived class authors (common in hardware abstraction layers) immediately see the available state. For classes with no protected members, start with `public:` directly.

```cpp
class IHardware
{
protected:
    HardwareDeviceInfo m_deviceInfo;
    bool m_autoReconnectEnabled = false;

public:
    virtual ~IHardware() = default;

    virtual std::string GetHardwareType() const = 0;
    virtual void Connect() = 0;

protected:
    void NotifyConnectionLost();

private:
    void ResetInternalState();
};
```

### Section Comments

Use `=====` separators for major API sections within a class:

```cpp
public:
    // ===== Device Identification =====
    virtual std::string GetHardwareType() const = 0;
    std::string GetManufacturer() const;

    // ===== Connection Management =====
    virtual void Connect() = 0;
    virtual void Disconnect() = 0;

    // ===== Health Monitoring =====
    virtual void EnableHealthMonitoring() {}
```

## Enums

Always use `enum class` (scoped enums). PascalCase for both the type and values. Include Doxygen comments:

```cpp
enum class DeviceState
{
    Disconnected,   ///< Not connected to hardware
    Connecting,     ///< Connection in progress
    Connected,      ///< Connected and operational
    Error,          ///< Connected but in error state
    Disconnecting   ///< Disconnection in progress
};
```

## Structs

Use `struct` for simple data aggregates with public members. Initialize members at declaration:

```cpp
struct SerialPortConfig
{
    unsigned int baudRate = 9600;
    unsigned int characterSize = 8;
    COMParity parity = COMParity::NONE;
    COMStopBits stopBits = COMStopBits::ONE;
};
```

## Modern C++ Features

### Keywords

- `override` — always use on virtual overrides
- `const` — on all methods that don't modify state
- `noexcept` — on move operations and destructors
- `= default` / `= delete` — for explicitly defaulted or deleted special members
- `auto` — use sparingly; prefer explicit types, but fine in range-based for loops and when the type is obvious

```cpp
void Connect() override;
int GetPosition() const override;
~FWxC() noexcept override;

JsonConfig(const JsonConfig&) = delete;
JsonConfig& operator=(const JsonConfig&) = delete;
JsonConfig(JsonConfig&&) noexcept = default;
```

### Smart Pointers

- `std::unique_ptr` — default choice for ownership
- `std::shared_ptr` — only when shared ownership is genuinely needed (e.g., WebSocket connections)
- No raw owning pointers in public APIs

### String Formatting

Use `fmt::format()` instead of string streams or concatenation:

```cpp
throw MagInsts::Common::HardwareConnectionException(
    fmt::format("Failed to load {} (error {})", dllName, error),
    static_cast<int>(error));
```

### Type Aliases

Prefer `using` over `typedef`:

```cpp
using RouteHandler = std::function<ordered_json(const ordered_json& request)>;
```

## Error Handling

Exception-based — no error code returns. Use the custom exception hierarchy from `maginsts-common-cpp`:

| Exception | When to use |
|-----------|------------|
| `HardwareConnectionException` | Device connection failures |
| `HardwareCommunicationException` | Communication errors (with optional error code) |
| `HardwareTimeoutException` | Operation timeouts |
| `ParameterException` | Invalid parameters |
| `OutOfRangeException` | Values outside valid range |
| `ConfigurationException` | Configuration errors |
| `InvalidStateException` | Invalid state transitions |
| `OperationNotAllowedException` | Disallowed operations |

Destructors must never throw:

```cpp
~FWxC() noexcept
{
    try
    {
        if (IsConnected()) Disconnect();
    }
    catch (...) {}
}
```

## Thread Safety

- `std::mutex` / `std::recursive_mutex` with `std::lock_guard` for simple locking
- `std::shared_mutex` with `std::shared_lock` / `std::unique_lock` for reader-writer patterns
- `std::atomic` for flags and counters
- Snapshot pattern for callback invocation (copy under lock, invoke outside lock):

```cpp
void Notify(Args... args) const
{
    std::map<int, Callback> snapshot;
    {
        std::lock_guard<std::mutex> lock(m_mutex);
        snapshot = m_callbacks;
    }
    for (const auto& [id, callback] : snapshot)
    {
        callback(args...);
    }
}
```

## Documentation

Doxygen `/** */` style for public APIs. Use `@brief`, `@param`, `@return`, `@throw`, `@note`, `@code`/`@endcode`:

```cpp
/**
 * @brief Register a device model for creation
 *
 * @param manufacturer Manufacturer name (e.g., "Thorlabs")
 * @param model Model name (e.g., "FW102C")
 * @param creator Factory function that creates the device from a connection ID
 *
 * @note Thread-safe. Can be called during static initialization.
 */
static void Register(const std::string& manufacturer,
                     const std::string& model,
                     CreatorFunc creator);
```

File-level documentation at the top of each header:

```cpp
/**
 * @file MDNSAdvertiser.hpp
 * @brief mDNS/Zeroconf service advertising for server discovery
 */
```

## Logging

Use the macro-based logging from `maginsts-common-cpp`. Tag hardware operations with a component identifier:

```cpp
LOG_INFO("Server started on port {}", port);
LOG_HW_INFO("FWxC", "Connecting to filter wheel {}", serialNumber);
LOG_HW_ERROR("SPMono", "Failed to read serial number: {}", e.what());
```
