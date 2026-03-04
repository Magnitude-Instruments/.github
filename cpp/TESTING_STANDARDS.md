# C++ Testing Standards

Testing standards and conventions for Magnitude Instruments C++ libraries.

## Framework

All C++ tests use [Google Test](https://github.com/google/googletest) with GMock. .NET binding tests use NUnit where applicable.

## Directory Structure

```
test/
├── CMakeLists.txt              # Defines add_maginsts_test() helper
├── unit/                       # Unit tests (no hardware/external dependencies)
│   └── <module>/
│       ├── CMakeLists.txt
│       ├── test_<component>.cpp
│       └── test_<component>_<aspect>.cpp
├── integration/                # Tests requiring real hardware or external services
│   └── <module>/
│       └── test_<component>_integration.cpp
└── dotnet/                     # .NET binding tests (NUnit), if applicable
```

Unit and integration test directories mirror the source structure under `src/maginsts/<library>/`.

## CMake Setup

Each repo defines a shared `add_maginsts_test()` helper in `test/CMakeLists.txt`:

```cmake
function(add_maginsts_test)
    cmake_parse_arguments(ARG "" "NAME" "SOURCES;LIBRARIES" ${ARGN})

    add_executable(${ARG_NAME} ${ARG_SOURCES})

    target_link_libraries(${ARG_NAME} PRIVATE
        GTest::gtest
        GTest::gtest_main
        GTest::gmock
        ${ARG_LIBRARIES}
    )

    gtest_discover_tests(${ARG_NAME})
endfunction()
```

Usage in subdirectory CMakeLists.txt:

```cmake
add_maginsts_test(
    NAME test_logger
    SOURCES test_logger.cpp
    LIBRARIES MagInsts::Common
)
```

Each test source file produces its own executable. Split large components into multiple files by aspect (e.g., `test_comdevice.cpp`, `test_comdevice_thread_safety.cpp`, `test_comdevice_protocol.cpp`).

## Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Test file | `test_<component>[_<aspect>].cpp` | `test_fwxc.cpp`, `test_comdevice_async.cpp` |
| Test fixture | `ComponentNameTest` (PascalCase) | `LoggerTest`, `CRC16Test`, `ServerMetricsTest` |
| Test case | Descriptive, often `Method_Condition_Expected` | `SetPosition_OutOfRange_ThrowsException` |
| Integration test file | `test_<component>_integration.cpp` | `test_fwxc_integration.cpp` |

Test case names should make the purpose obvious without reading the test body. Common patterns:

```cpp
// Action result
TEST_F(FWxCTest, Connect_Success)

// Method + condition + expected outcome
TEST_F(FWxCTest, SetPosition_OutOfRange_ThrowsException)
TEST_F(FWxCTest, GetPosition_NotConnected_ThrowsException)

// Feature or behavior
TEST_F(FWxCTest, HealthMonitoring_EnableAndDisable)
TEST_F(ServerMetricsTest, ThreadSafety)

// Categorized scenarios
TEST_F(ExceptionTest, RealWorldScenario_FilterWheelError)
TEST_F(ExceptionTest, EdgeCase_SpecialCharacters)
TEST_F(ExceptionTest, Performance_ExceptionCreationTime)
```

## Test Structure

### Fixtures

All tests use fixtures inheriting from `::testing::Test`:

```cpp
class JsonConfigTest : public ::testing::Test
{
protected:
    std::string test_dir;
    std::string test_file;

    void SetUp() override
    {
        // Create unique temp directory
        test_dir = "test_jsonconfig_" + std::to_string(
            std::chrono::system_clock::now().time_since_epoch().count()
        );
        test_file = test_dir + "/test_config.json";
    }

    void TearDown() override
    {
        // Clean up temp files
        if (fs::exists(test_dir))
        {
            std::error_code ec;
            fs::remove_all(test_dir, ec);
        }
    }

    // Helper methods for common operations
    void CreateTestConfig(const std::string& content) { /* ... */ }
};
```

Guidelines:
- `SetUp()` initializes clean state; `TearDown()` guarantees cleanup
- Use unique directory/file names with timestamps to avoid collisions
- Add fixture helper methods for repeated setup logic
- Each test must be independent — no shared mutable state between tests

### Assertions

Prefer `EXPECT_*` for non-fatal checks. Use `ASSERT_*` only when subsequent assertions depend on the result (e.g., checking container size before accessing elements):

```cpp
auto devices = HardwareFactory<IHardware>::ListAvailableDevices();
ASSERT_EQ(devices.size(), 2);   // Must pass before accessing elements
EXPECT_EQ(devices[0].manufacturer, "Mock");
```

Common assertion patterns:

```cpp
// Exceptions
EXPECT_THROW(fw->SetPosition(0), OutOfRangeException);
EXPECT_NO_THROW(fw->SetPosition(1));

// String matching (GMock)
EXPECT_THAT(ex.what(), ::testing::HasSubstr("timeout"));

// JSON state verification
auto state = fw->GetState();
EXPECT_EQ(state["positionCount"], 6);
EXPECT_TRUE(state.contains("position"));

// Floating point
EXPECT_DOUBLE_EQ(metrics["avg_time_ms"], 13.9);
```

## Mocking Strategies

Choose the simplest approach that provides adequate isolation:

### Simple Mock Objects

For testing against interfaces — implement the interface with minimal controllable behavior:

```cpp
class MockFilterWheel : public IFilterWheel
{
public:
    void Connect() override { m_state = DeviceState::Connected; }
    bool IsConnected() const override { return m_state == DeviceState::Connected; }
    int GetPosition() const override { return m_position; }
    void SetPosition(int position) override { m_position = position; }
private:
    DeviceState m_state = DeviceState::Disconnected;
    int m_position = 1;
};
```

### GMock

For verifying call expectations, particularly useful for server/callback testing:

```cpp
class MockServer : public Server
{
    MOCK_METHOD(nlohmann::ordered_json, mock_get_handler, (const nlohmann::ordered_json& body));
};

EXPECT_CALL(*server, mock_get_handler(::testing::_))
    .WillOnce(Return(expected_response));
```

### Function Pointer Replacement

For classes that wrap vendor DLLs — replace function pointers with mock implementations:

```cpp
class TestableFWxC : public FWxC
{
public:
    TestableFWxC()
    {
        m_dll.Open = &MockOpen;
        m_dll.Close = &MockClose;
        // ...
    }

    static void ResetMockState() { /* reset all static mock state */ }

    static int __cdecl MockOpen(char*, int, int) { return s_openResult; }
private:
    static int s_openResult;
};
```

### Test Mode Injection

For classes that call system commands — intercept at the command execution layer:

```cpp
void SetUp() override
{
    PnPDevice::SetTestMode(true);
    PnPDevice::ClearTestResponses();
    PnPDevice::SetTestCommandResponse(
        "pnputil /enum-devices /class Ports",
        mockEnumOutput,
        0  // exit code
    );
}

void TearDown() override
{
    PnPDevice::SetTestMode(false);
    PnPDevice::ClearTestResponses();
}
```

### Testable Subclass

For accessing protected members in tests:

```cpp
class TestableDevice : public PnPDevice
{
public:
    using PnPDevice::PnPDevice;
    using PnPDevice::instanceId;      // Expose protected members
    using PnPDevice::serialNumber;
};
```

## Thread Safety Testing

Test thread safety with multiple threads performing concurrent operations, using `std::atomic` for shared counters:

```cpp
TEST_F(JsonConfigTest, ThreadSafety)
{
    const int num_threads = 10;
    const int operations_per_thread = 100;
    std::atomic<int> success_count{0};
    std::vector<std::thread> threads;

    for (int i = 0; i < num_threads; ++i)
    {
        threads.emplace_back([&, thread_id = i]() {
            for (int j = 0; j < operations_per_thread; ++j)
            {
                config.Set(j, key, "value");
                success_count++;
            }
        });
    }

    for (auto& t : threads) t.join();

    EXPECT_EQ(success_count, num_threads * operations_per_thread);
}
```

## Integration Tests

Integration tests that require hardware or external services must skip gracefully when dependencies are unavailable:

```cpp
class FWxCIntegrationTest : public ::testing::Test
{
protected:
    std::unique_ptr<FWxC> fw;

    void SetUp() override
    {
        auto devices = FWxC::ListAvailableDevices();
        if (devices.empty())
        {
            GTEST_SKIP() << "No FWxC devices found. Skipping hardware tests.";
        }

        fw = std::make_unique<FWxC>(devices[0].deviceId);
        fw->Connect();
    }

    void TearDown() override
    {
        if (fw && fw->IsConnected())
        {
            try { fw->Disconnect(); } catch (...) {}
        }
    }
};
```

Guidelines:
- Always use `GTEST_SKIP()` when hardware or services are unavailable
- Safe teardown with try-catch to ensure cleanup
- Integration tests go in `test/integration/`, never mixed with unit tests

## What to Test

- **Public API surface** — all public methods, including error paths
- **Boundary values** — min, max, off-by-one, empty input
- **Exception handling** — correct exception types thrown for invalid operations
- **Callbacks** — subscription, notification, and removal
- **State transitions** — connected/disconnected, valid/invalid states
- **Thread safety** — concurrent access to shared state where APIs are documented as thread-safe

## Running Tests

```bash
# All tests
ctest --test-dir build/debug -C Debug --output-on-failure

# Filter by test executable name
ctest --test-dir build/debug -C Debug -R "Logger"

# Filter by specific test case
ctest --test-dir build/debug -C Debug -R "CRC16Test.MagIO_ReadInputsCommand"

# Verbose output
ctest --test-dir build/debug -C Debug -V
```
