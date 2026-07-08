# C++ Engineering Standard

| | |
|---|---|
| **Document ID** | SPEC-CODING-001 |
| **Title** | C++ Engineering Standard |
| **Version** | 1.0.0 |
| **Status** | Draft |
| **Category** | Coding |
| **Applies to** | All C++ source code in Lingmo Desktop repositories |
| **Supersedes** | None |
| **Cross-ref** | SPEC-CORE-001 (Naming Convention), SPEC-REPO-002 (Header Organization), SPEC-REPO-003 (CMake Organization) |

---

## 1. Language Standard

### 1.1 C++ Version

[REQ-CPP-001] All Lingmo Desktop C++ code MUST conform to the C++20 standard as defined by ISO/IEC 14882:2020.

[REQ-CPP-002] Compiler extensions MUST NOT be used unless guarded by `#ifdef` for the specific compiler and approved by the architecture team.

### 1.2 Allowed Language Features

[REQ-CPP-003] The following C++20 features are REQUIRED for use where applicable:

| Feature | Usage |
|---|---|
| `auto` (type deduction) | Variable declarations and return types where the type is obvious from context |
| `constexpr` / `consteval` | Compile-time evaluation where invariants can be checked at compile time |
| `concepts` and `requires` | Template constraints instead of SFINAE or `static_assert` |
| `std::span` | Non-owning views over contiguous sequences |
| `std::string_view` | Non-owning string parameters (use `QStringView` for Qt APIs) |
| `std::optional` | Optional return values instead of out-parameters or sentinel values |
| `std::variant` | Type-safe unions instead of `enum` + `void*` patterns |
| structured bindings | Decomposing pairs, tuples, and aggregates |
| designated initializers | Aggregate initialization with named fields |
| `= default` and `= delete` | Explicit special member function control |
| `explicit(bool)` | Conditional explicit constructors |
| `noexcept` specifications | Where guaranteed not to throw |
| `inline` namespaces | ABI versioning boundaries |

[REQ-CPP-004] The following C++20 features are FORBIDDEN:

| Feature | Rationale |
|---|---|
| `std::source_location` | Use `Q_FUNC_INFO` or `QDebug` for consistency with Qt |
| `import` / modules | CMake and Qt 6 do not fully support C++20 modules in production. Use traditional `#include`. |
| `coroutines` | Qt 6 co_await support is immature. Re-evaluate at Qt 6.8+. |
| `std::jthread` | Use `QThread` for Qt-integrated threading. |
| `std::stop_token` | Use `QThread::requestInterruption()`. |

### 1.3 Compiler Compatibility

[REQ-CPP-005] All C++ code MUST compile without errors under:

| Compiler | Minimum Version | Flags |
|---|---|---|
| GCC | 12 | `-std=c++20 -Wall -Wextra -Wpedantic -Werror` |
| Clang | 16 | `-std=c++20 -Wall -Wextra -Wpedantic -Werror` |
| MSVC | Visual Studio 17 2022 | `/std:c++20 /W4 /WX /permissive-` |

### 1.4 Forbidden C++98/11 Patterns

[REQ-CPP-006] The following legacy patterns MUST NOT appear in new code and MUST be removed when modifying surrounding code:

- `NULL` or `0` for null pointers — use `nullptr`
- `throw()` exception specifications — use `noexcept`
- `auto_ptr` — use `std::unique_ptr`
- `bind1st` / `bind2nd` — use lambdas
- `dynamic_cast` for non-polymorphic types — use `static_cast`
- C-style casts — use `static_cast`, `reinterpret_cast`, or `qobject_cast`

---

## 2. General Programming Rules

### 2.1 Initialization

[REQ-CPP-007] Brace initialization MUST be preferred for all variable declarations:

```cpp
int value{42};
QString name{"Lingmo"};
std::vector<int> numbers{1, 2, 3};
```

[REQ-CPP-008] Narrowing conversions in brace initialization MUST be avoided. If a narrowing conversion is necessary, an explicit `static_cast` is REQUIRED.

[REQ-CPP-009] Member initializer lists MUST be used instead of assignment in constructors:

```cpp
// CORRECT
Theme::Theme() : d{std::make_unique<Private>()} {}

// FORBIDDEN
Theme::Theme() { d = std::make_unique<Private>(); }
```

### 2.2 Const Correctness

[REQ-CPP-010] Every variable, parameter, and member function that is not modified MUST be declared `const`:

```cpp
const int maxItems{100};
void process(const QString& input) const;
```

[REQ-CPP-011] Member functions that do not modify observable state MUST be declared `const`. Getter functions MUST be `const`.

[REQ-CPP-012] Pointers and references to const-qualified objects MUST be used for function parameters that are read-only:

```cpp
void setTheme(const QString& name);                // CORRECT: read-only reference
void setTheme(QString name);                       // ACCEPTABLE when taking ownership
void setTheme(const QString&& name);               // FORBIDDEN: const rvalue ref has no purpose
```

### 2.3 RAII

[REQ-CPP-013] Every resource MUST be owned by a RAII object. Raw resource acquisition without immediate RAII wrapping is FORBIDDEN:

| Resource | RAII Wrapper |
|---|---|
| Heap memory | `std::unique_ptr`, `std::shared_ptr`, `QScopedPointer` |
| File handles | `QFile`, `std::fstream` |
| Mutex locks | `std::lock_guard`, `std::unique_lock`, `QMutexLocker` |
| Database connections | `QSqlDatabase` wrapper |
| Socket connections | `QTcpSocket`, `QLocalSocket` |

### 2.4 Ownership Rules

[REQ-CPP-014] Every object MUST have a single, clearly defined owner at all times. Ownership MUST be documented when not obvious from the type.

[REQ-CPP-015] The following ownership hierarchy applies:

| Scope | Owner | Mechanism |
|---|---|---|
| Whole program | `QCoreApplication` / `QApplication` | Qt event loop |
| QObject tree | Parent QObject | `QObject` parent-child (Section 3.2) |
| Heap object (single owner) | `std::unique_ptr` | Move-only semantics |
| Heap object (shared) | `std::shared_ptr` / `QSharedPointer` | Reference counting |
| Stack object | Current scope | Automatic destruction |

### 2.5 Lifetime Management

[REQ-CPP-016] A pointer or reference to an object MUST NOT outlive the object. The following lifetime extension mechanisms are REQUIRED where necessary:

| Pattern | Mechanism |
|---|---|
| Prevent use-after-free | `QPointer` for QObject tracking |
| Prevent dangling `this` | `QObject::deleteLater()` |
| Weak reference to shared resource | `std::weak_ptr` |
| Observer pattern | `QObject::connect()` with context objects |

### 2.6 Error Prevention

[REQ-CPP-017] Shadowing of variables (declaring a variable with the same name as an outer-scope variable) MUST be avoided:

```cpp
int value{10};
if (condition) {
    int value{20};    // FORBIDDEN: shadows outer 'value'
}
```

[REQ-CPP-018] Fallthrough in switch statements MUST be annotated:

```cpp
switch (state) {
case State::Active:
    doActive();
    break;
case State::Pending:
    doPending();
    [[fallthrough]]; // Intentional fallthrough
case State::Idle:
    doIdle();
    break;
}
```

[REQ-CPP-019] Unused parameters MUST use the `/*name*/` comment pattern or (C++20) omit the name entirely:

```cpp
void handler(int /*unused*/) { }
void handler(int) { }
```

---

## 3. Qt 6 Integration

### 3.1 QObject Inheritance

[REQ-CPP-020] Every class that uses signals, slots, properties, or `QObject` parent-child lifetime MUST inherit from `QObject` (directly or indirectly).

[REQ-CPP-021] Multiple inheritance with `QObject` is FORBIDDEN. Only one QObject base class is permitted.

[REQ-CPP-022] QObject-derived classes MUST NOT be copyable:

```cpp
class Theme : public QObject {
    Q_OBJECT
public:
    Theme(const Theme&) = delete;
    Theme& operator=(const Theme&) = delete;
};
```

### 3.2 Parent-Child Lifetime

[REQ-CPP-023] Every `QObject` MUST have a parent or be explicitly managed. The parent-child tree determines destruction order.

[REQ-CPP-024] Widgets and QObjects with visual representation MUST be created with a parent:

```cpp
auto* button = new QPushButton{"Click me", parentWidget}; // CORRECT
auto* button = new QPushButton{"Click me"};                // MUST have parent or be top-level
```

[REQ-CPP-025] Objects added to a parent via `setParent()` or constructor argument MUST NOT be manually deleted. The parent handles destruction.

### 3.3 Q_OBJECT Macro

[REQ-CPP-026] The `Q_OBJECT` macro MUST appear in the `private` section of every QObject-derived class declaration:

```cpp
class Theme : public QObject {
    Q_OBJECT
public:
    // ...
};
```

[REQ-CPP-027] The `Q_OBJECT` macro MUST NOT be placed in a `.cpp` file. The `Q_PLUGIN_METADATA` macro is the exception for plugin classes.

### 3.4 Signals and Slots

[REQ-CPP-028] New-style (PMF) `&Class::method` connect syntax MUST be used. String-based connect syntax is FORBIDDEN:

```cpp
// CORRECT
connect(button, &QPushButton::clicked, this, &Controller::handleClick);

// FORBIDDEN
connect(button, SIGNAL(clicked()), this, SLOT(handleClick()));
```

[REQ-CPP-029] Lambda expressions in connect() MUST capture by context object pointer for safe disconnection:

```cpp
connect(manager, &Manager::finished, this, [this](Result r) {
    processResult(r);
});
```

[REQ-CPP-030] The context object (third argument to `connect`) MUST be specified for lambdas that reference `this`:

```cpp
connect(source, &Source::signal, this, [this] { ... });  // CORRECT
connect(source, &Source::signal, [this] { ... });          // FORBIDDEN: no context, may fire after destruction
```

### 3.5 Event Handling

[REQ-CPP-031] Event filters MUST be implemented via `installEventFilter` and `eventFilter()`. Lambdas MUST NOT be used as event filters.

[REQ-CPP-032] Custom events MUST inherit from `QEvent` and use a custom type ID registered with `QEvent::registerEventType()`:

```cpp
class ThemeChangeEvent : public QEvent {
public:
    static constexpr Type EventType = static_cast<Type>(registerEventType());
    ThemeChangeEvent() : QEvent{EventType} {}
};
```

### 3.6 Qt Meta-Object System

[REQ-CPP-033] `Q_PROPERTY` MUST be used for all properties that are:

| Condition | Example |
|---|---|
| Needed in QML | `Q_PROPERTY(QColor accentColor READ accentColor NOTIFY accentColorChanged)` |
| Configurable at runtime | `Q_PROPERTY(int timeout READ timeout WRITE setTimeout NOTIFY timeoutChanged)` |
| Observable from outside | `Q_PROPERTY(bool running READ running NOTIFY runningChanged)` |

[REQ-CPP-034] The `NOTIFY` signal in `Q_PROPERTY` MUST be present for readable properties that can change at runtime. Properties that never change after construction MAY omit `NOTIFY`.

[REQ-CPP-035] `Q_ENUM` MUST be used for enums exposed to the meta-object system. `Q_ENUM_NS` MUST be used for enums in namespaces.

---

## 4. Memory Management

### 4.1 std::unique_ptr

[REQ-CPP-036] `std::unique_ptr` MUST be used for exclusive ownership of heap-allocated non-QObject resources:

```cpp
auto settings = std::make_unique<Settings>(configPath);
```

[REQ-CPP-037] `std::unique_ptr` MUST be returned from factory functions that transfer exclusive ownership:

```cpp
auto createTheme() -> std::unique_ptr<Theme>;
```

### 4.2 std::shared_ptr and QSharedPointer

[REQ-CPP-038] `std::shared_ptr` or `QSharedPointer` MUST be used only when ownership is genuinely shared. Shared ownership MUST be justified by a comment explaining why exclusive ownership is insufficient.

[REQ-CPP-039] `std::shared_ptr` MUST NOT be used:

- For QObject-derived types (use `QPointer` or parent-child)
- Across library boundaries (ABI instability from different allocators)
- As a premature optimization for copy-on-write

[REQ-CPP-040] `QSharedPointer` is preferred over `std::shared_ptr` when the managed object is a QObject, because `QSharedPointer` respects QObject destructor ordering.

### 4.3 QPointer

[REQ-CPP-041] `QPointer` MUST be used to track QObject lifetime across async boundaries:

```cpp
QPointer<Theme> m_theme;
// ...
if (m_theme) {
    m_theme->apply();
}
```

[REQ-CPP-042] `QPointer` MUST be checked before dereference. The tracked object may have been deleted at any time.

### 4.4 QObject Ownership

[REQ-CPP-043] QObject heap allocation MUST use `new`. `std::make_unique` MUST NOT be used for QObject types because QObject destructor ordering requires the Qt meta-object system.

[REQ-CPP-044] QObjects that are not part of the parent-child tree MUST be deleted via `deleteLater()`:

```cpp
m_worker->deleteLater();   // CORRECT: deferred deletion at safe point
delete m_worker;            // FORBIDDEN: may crash if event is pending
```

### 4.5 Stack Allocation

[REQ-CPP-045] Stack allocation MUST be preferred over heap allocation whenever the object fits on the stack and does not require dynamic lifetime:

```cpp
QString result;                          // CORRECT: stack
auto* result = new QString;              // FORBIDDEN: unnecessary heap allocation
```

[REQ-CPP-046] Large objects (≥ 4 KB) or objects with expensive constructors MAY use heap allocation. The decision MUST be documented.

### 4.6 Forbidden Memory Patterns

[REQ-CPP-047] The following patterns are FORBIDDEN:

| Pattern | Replacement |
|---|---|
| `malloc()` / `free()` | `new`/`delete` or `std::make_unique` |
| `new`/`delete` for arrays | `std::vector`, `QVector` |
| Raw `new` without immediate `unique_ptr` | `std::make_unique` |
| `void*` for type erasure | `QVariant`, `std::any`, `std::variant` |
| Manual buffer management | `QByteArray`, `std::vector<char>` |
| `memcpy` for non-trivially-copyable types | Copy constructors or `std::copy` |

---

## 5. Containers and Data Types

### 5.1 Qt Container Selection

[REQ-CPP-048] The following Qt types MUST be preferred over their standard library equivalents for Qt-integrated code:

| Qt Type | Use Case |
|---|---|
| `QString` | All user-visible text and internal text manipulation |
| `QByteArray` | Binary data, serialization, non-UTF8 text |
| `QStringView` | Read-only string parameters (Qt APIs) |
| `QList<T>` | Ordered collections (Qt-compatible, COW semantics) |
| `QVector<T>` | Contiguous storage (alias of `QList` since Qt 6) |
| `QHash<K,V>` | Unordered associative containers (QObject keys) |
| `QSet<T>` | Unordered sets (QObject elements) |
| `QMap<K,V>` | Ordered associative containers |
| `QVariant` | Type-erased values for property system |

[REQ-CPP-049] `QList<T>` is the default Qt container. `std::vector` SHOULD be preferred when C++ interop or ABI stability at the C ABI boundary is required.

### 5.2 Standard Library Container Selection

[REQ-CPP-050] The following standard library types MUST be preferred over Qt equivalents in the indicated contexts:

| `std::` Type | When Preferred |
|---|---|
| `std::vector<T>` | Public API where ABI matters; integration with non-Qt code |
| `std::optional<T>` | Optional return values |
| `std::variant<T...>` | Type-safe discriminated unions |
| `std::expected<T,E>` | Error-returning functions (C++23 polyfill) |
| `std::span<T>` | Non-owning contiguous sequence parameters |
| `std::string_view` | Interop with non-Qt string APIs |

[REQ-CPP-051] `QString` and `std::string` conversions MUST be explicit:

```cpp
auto stdStr = qString.toStdString();       // EXPLICIT
auto qStr = QString::fromStdString(stdStr); // EXPLICIT

QString qStr = stdString;                   // FORBIDDEN: implicit conversion
```

### 5.3 Container Usage Rules

[REQ-CPP-052] `reserve()` MUST be called before populating a container when the final size is known in advance:

```cpp
QList<QString> items;
items.reserve(expectedCount);
for (...) { items.append(value); }
```

[REQ-CPP-053] Range-based `for` loops MUST be used instead of index-based iteration:

```cpp
for (const auto& item : items) { process(item); }  // CORRECT
for (int i = 0; i < items.size(); ++i) {}           // AVOID unless index needed
```

[REQ-CPP-054] `QString` manipulation MUST follow these rules:

| Operation | Method | Rationale |
|---|---|---|
| Concatenation | `QString::arg()` or `QStringLiteral` | `+` creates temporary copies |
| Number formatting | `QString::number()` | Locale-aware |
| Large string building | `QStringBuilder` with `%` | COW optimization |

### 5.4 Type Selection Rules

[REQ-CPP-055] The following type decisions are REQUIRED:

| Situation | Correct Type | Rationale |
|---|---|---|
| Boolean state | `bool` | |
| Fixed-size integer | `int8_t`, `uint32_t`, etc. from `<cstdint>` | Platform-independent size |
| Counter, index | `int` or `qsizetype` (Qt 6) | `qsizetype` matches pointer size |
| File size | `qint64` | Files can exceed 2 GB |
| Timestamp | `QDateTime` | Timezone-aware |
| Duration | `std::chrono::milliseconds` or `std::chrono::duration` | Type-safe time math |
| Network port | `uint16_t` | Valid range 0–65535 |
| Enum bitfield | `QFlags<T>` | Type-safe flags |

---

## 6. ABI Stability Rules

### 6.1 Scope

This section defines rules for maintaining ABI stability. The detailed mechanism (PIMPL, export macros, visibility) is defined in SPEC-REPO-002 §6 and §7. This section specifies the engineering decisions that affect ABI.

### 6.2 Public Class Rules

[REQ-CPP-056] A public class is any class annotated with an export macro (SPEC-REPO-002 §5.3). Public classes MUST follow the Qt polyymorphic class convention:

```cpp
class LINGMOTHEME_EXPORT Theme : public QObject {
    Q_OBJECT
public:
    explicit Theme(QObject* parent = nullptr);
    ~Theme() override;
    // ...
};
```

[REQ-CPP-057] Public classes MUST use PIMPL (SPEC-REPO-002 §7) if the class has any non-trivial member variable.

[REQ-CPP-058] Public classes MUST NOT have inline `Q_DECLARE_METATYPE` in public headers. Meta-type registration belongs in the `.cpp` file or a dedicated registration header.

### 6.3 Virtual Function Rules

[REQ-CPP-059] Virtual functions MUST NOT be added to a public class after its first release. Adding a virtual function changes the vtable layout and breaks ABI.

[REQ-CPP-060] Virtual functions in public classes MUST use the `override` keyword. The `virtual` keyword is OPTIONAL when `override` is present.

[REQ-CPP-061] A public class that is not intended for subclassing MUST be declared `final`:

```cpp
class LINGMOTHEME_EXPORT Theme final : public QObject { ... };
```

### 6.4 Inline Function Restrictions

[REQ-CPP-062] Inline functions in public headers are part of the ABI (SPEC-REPO-002 §6.3). Their implementation MUST NOT be patched in a patch release.

[REQ-CPP-063] Inline function bodies MUST NOT reference private symbols or private headers:

```cpp
// CORRECT inline — only public API
inline int Widget::value() const { return m_value; }

// FORBIDDEN inline — references private type
inline void Widget::setup() { Private::init(); }
```

### 6.5 Template Exposure

[REQ-CPP-064] Templates defined in public headers are fully instantiated by the consumer and are NOT part of the library's ABI (SPEC-REPO-002 §6.4). Template definitions MUST be stable at the source level.

[REQ-CPP-065] Explicit template instantiations in public headers MUST use `extern template`:

```cpp
extern template class LINGMOTHEME_EXPORT std::vector<Theme>;
```

[REQ-CPP-066] Template specializations MUST NOT be added or removed in a minor version.

### 6.6 Binary Compatibility

[REQ-CPP-067] Binary compatibility MUST be maintained within a major version. The following changes are FORBIDDEN in a minor or patch release (SPEC-REPO-002 §6.1):

- Adding, removing, or reordering member variables
- Adding, removing, or reordering virtual functions
- Changing function parameter types
- Changing the size or alignment of a public type
- Changing the inheritance hierarchy
- Removing or reordering enum values

### 6.7 PIMPL Requirements

[REQ-CPP-068] PIMPL (Pointer to Implementation) is REQUIRED for any exported class that meets the criteria in SPEC-REPO-002 §7.1.

[REQ-CPP-069] The `d` pointer MUST be the last member variable in the class to minimize padding changes if it is removed.

---

## 7. Exception and Error Handling

### 7.1 Exception Policy

[REQ-CPP-070] Exceptions MUST NOT propagate across library boundaries. Every public API function MUST catch all exceptions and convert them to error return values:

```cpp
int LingmoTheme::loadConfig() noexcept {
    try {
        return doLoad();
    } catch (const std::exception& e) {
        qCCritical(Logging) << "Config load failed:" << e.what();
        return -1;
    }
}
```

[REQ-CPP-071] Exceptions MAY be used within a single translation unit for exceptional error conditions. Exceptions MUST NOT be used for control flow.

### 7.2 Forbidden Exception Patterns

[REQ-CPP-072] The following exception patterns are FORBIDDEN:

| Pattern | Problem |
|---|---|
| `throw` from a destructor | Undefined behavior during stack unwinding |
| `throw` from a signal handler | Not signal-safe |
| `throw` across a C ABI boundary | Stack unwinding cannot cross C frames |
| `throw` from `QObject::connect` lambda | Qt slot invocation may not propagate exceptions |

### 7.3 Error Return Patterns

[REQ-CPP-073] The following error return mechanisms MUST be used as specified:

| Mechanism | When REQUIRED |
|---|---|
| `bool` return | Simple success/failure with no error details |
| `std::optional<T>` | Function may return no value (not an error) |
| `std::expected<T, E>` | Function returns a value OR an error (C++23 polyfill) |
| `enum class Error` | Error code with known set of failures |
| `QString` error out-parameter | Additional error description (pass by pointer) |

[REQ-CPP-074] Out-parameters for errors MUST use pointer syntax:

```cpp
auto loadConfig(QString* error = nullptr) -> bool;
if (!loadConfig(&error)) { handle(error); }
```

### 7.4 Result Types

[REQ-CPP-075] `std::optional` is the preferred return type for functions that may reasonably produce no result:

```cpp
auto findTheme(const QString& name) -> std::optional<Theme>;
```

[REQ-CPP-076] When a polyfill for `std::expected` is not available, a struct with a union or variant is acceptable for returning value-or-error.

### 7.5 Assertion Rules

[REQ-CPP-077] `Q_ASSERT` MUST be used for internal invariant checks. `Q_ASSERT` is compiled out in Release builds.

[REQ-CPP-078] `Q_ASSERT_X` MUST be used when a descriptive message helps diagnosis:

```cpp
Q_ASSERT_X(!m_theme.isNull(), "Theme::apply", "Theme must be initialized before apply()");
```

[REQ-CPP-079] `Q_ASSERT` MUST NOT be used to validate user input or network data. Use explicit checks with error returns.

[REQ-CPP-080] `static_assert` MUST be used for compile-time invariant checks:

```cpp
static_assert(sizeof(Theme) <= 64, "Theme class should fit in two cache lines");
```

---

## 8. Concurrency

### 8.1 Thread Model

[REQ-CPP-081] Lingmo Desktop uses the Qt thread model. The main thread owns all QObject instances and runs the Qt event loop. Worker threads MUST NOT create QObjects unless they have their own event loop.

[REQ-CPP-082] GUI updates MUST occur on the main (GUI) thread. Worker threads MUST use signals to communicate results back to the main thread.

### 8.2 QThread Usage

[REQ-CPP-083] Worker threads MUST use the worker-object pattern:

```cpp
auto* worker = new Worker;                    // Do NOT set parent — moved to thread
auto* thread = new QThread;
worker->moveToThread(thread);
connect(thread, &QThread::started, worker, &Worker::process);
connect(worker, &Worker::finished, thread, &QThread::quit);
connect(thread, &QThread::finished, worker, &QObject::deleteLater);
thread->start();
```

[REQ-CPP-084] Subclassing `QThread` is FORBIDDEN. The exception is a thread that performs no Qt event processing and only runs a tight loop.

### 8.3 QObject Thread Affinity

[REQ-CPP-085] QObject thread affinity MUST NOT be changed after the object has been connected to signals. Use `moveToThread()` before any connections are established.

[REQ-CPP-086] Thread affinity MUST NOT be changed for QWidget or QWindow instances. Widgets are main-thread-only.

### 8.4 QtConcurrent

[REQ-CPP-087] `QtConcurrent::run` SHOULD be used for fire-and-forget background tasks that do not require a persistent worker thread:

```cpp
QtConcurrent::run([this] {
    auto result = performExpensiveOperation();
    QMetaObject::invokeMethod(this, [this, result] {
        onResult(result);
    }, Qt::QueuedConnection);
});
```

### 8.5 Mutex Protection

[REQ-CPP-088] Data shared between threads MUST be protected by:

| Scenario | Protection |
|---|---|
| QObject property access | Main thread only (no mutex needed) |
| Shared data structure | `QMutex` with `QMutexLocker` |
| Read-mostly data | `QReadWriteLock` with `QReadLocker`/`QWriteLocker` |
| Atomic integer/flag | `std::atomic<T>` |
| Lock-free data (rare) | `std::atomic` with memory ordering documentation |

[REQ-CPP-089] `QMutexLocker` MUST be used instead of manual `lock()`/`unlock()`:

```cpp
// CORRECT
{
    const QMutexLocker locker{&m_mutex};
    m_sharedData = newValue;
}

// FORBIDDEN — not exception-safe
m_mutex.lock();
m_sharedData = newValue;
m_mutex.unlock();
```

### 8.6 Atomic Operations

[REQ-CPP-090] `std::atomic` MUST be used for simple flags and counters shared between threads:

```cpp
std::atomic<bool> m_running{false};
std::atomic<int> m_activeCount{0};
```

[REQ-CPP-091] `QAtomicInt` and related Qt atomics are acceptable but `std::atomic` is preferred for C++ standard compliance.

### 8.7 Thread Pool

[REQ-CPP-092] `QThreadPool::globalInstance()` MUST be used for short-lived tasks:

```cpp
QtConcurrent::run(QThreadPool::globalInstance(), [this] { ... });
```

[REQ-CPP-093] Custom thread pool instances SHOULD be used for workload isolation (e.g., a dedicated pool for thumbnail generation).

---

## 9. Performance Guidelines

### 9.1 Move Semantics

[REQ-CPP-094] Move semantics MUST be used to avoid unnecessary copies:

```cpp
// CORRECT — move large object into container
m_items.push_back(std::move(largeObject));

// CORRECT — return by value (move or RVO)
QList<Item> collectItems();

// CORRECT — accept value and move
void setItems(QList<Item> items) { m_items = std::move(items); }
```

[REQ-CPP-095] Parameters that are stored MUST be passed by value and moved:

```cpp
void setName(QString name) { m_name = std::move(name); }   // CORRECT
void setName(const QString& name) { m_name = name; }         // ACCEPTABLE for small types
```

### 9.2 Allocation Minimization

[REQ-CPP-096] Frequent allocations in hot paths MUST be avoided:

```cpp
// AVOID — allocates in every iteration
for (int i = 0; i < count; ++i) {
    QString result = compute() + suffix;
}

// PREFER — reuse buffer
QString result;
for (int i = 0; i < count; ++i) {
    result.clear();
    result = compute();
    result += suffix;
}
```

[REQ-CPP-097] Small buffer optimization (SBO) types MUST be preferred when the data is typically small:

```cpp
std::string name;              // SBO: small allocation on stack
QString name;                  // COW, may allocate
QSmallString<64> name;         // Fixed-size stack buffer (where available)
```

### 9.3 QString Optimization

[REQ-CPP-098] `QStringLiteral` MUST be used for compile-time string constants:

```cpp
const auto themeFile = QStringLiteral("lingmo-theme.conf");
```

[REQ-CPP-099] `QLatin1String` MUST be used for comparison with Latin-1 literals:

```cpp
if (value == QLatin1String("active")) { ... }
```

### 9.4 reserve() Usage

[REQ-CPP-100] `reserve()` MUST be called on containers when the final size is known:

```cpp
m_files.reserve(fileList.size());
for (const auto& f : fileList) {
    m_files.append(f);
}
```

### 9.5 Unnecessary Copies

[REQ-CPP-101] Range-for with `const auto&` MUST be used for read-only iteration:

```cpp
for (const auto& item : container) { ... }    // CORRECT: no copy
for (auto item : container) { ... }            // FORBIDDEN: copies each element
for (auto&& item : container) { ... }          // ACCEPTABLE: universal ref when modifying
```

### 9.6 Lazy Initialization

[REQ-CPP-102] Expensive initialization MUST be deferred until the result is actually used:

```cpp
mutable std::once_flag m_cacheFlag;
mutable QHash<QString, Theme> m_themeCache;

auto themeFor(const QString& name) -> const Theme& {
    std::call_once(m_cacheFlag, [this] {
        m_themeCache = loadAllThemes();
    });
    return m_themeCache.value(name);
}
```

[REQ-CPP-103] `Q_DECL_CONST_FUNC` and `Q_CONSTINIT` MUST be used for constexpr data:

```cpp
Q_CONSTINIT const std::array<int, 4> defaultSizes = {16, 22, 32, 48};
```

---

## 10. Security

### 10.1 Unsafe APIs

[REQ-CPP-104] The following APIs MUST NOT be used:

| API | Risk | Replacement |
|---|---|---|
| `gets()` | Buffer overflow | `QTextStream::readLine()` |
| `sprintf()` | Buffer overflow | `QString::asprintf()` or `QString::arg()` |
| `strcpy()` / `strcat()` | Buffer overflow | `QString` / `QByteArray` |
| `system()` | Command injection | `QProcess` |
| `popen()` | Command injection | `QProcess` |
| `alloca()` | Stack overflow | `std::vector` or `QVarLengthArray` |
| `setuid()` / `setgid()` | Privilege escalation risk | systemd unit isolation |

### 10.2 Input Validation

[REQ-CPP-105] All external input (files, network, D-Bus, environment variables) MUST be validated before use:

```
Input → Validation → Trusted processing → Output
```

[REQ-CPP-106] String inputs MUST be validated for:

- Length (upper and lower bounds)
- Character encoding
- Allowed character set
- Path traversal patterns (`../`, symlink attacks)

[REQ-CPP-107] Numeric inputs MUST be validated for:

- Range (upper and lower bounds)
- Type bounds (no integer overflow)
- Sign (unsigned vs signed expectations)

### 10.3 Memory Safety

[REQ-CPP-108] The Qt container and string classes are memory-safe by design. Code that uses them correctly is immune to buffer overflows and use-after-free from the C++ standard library.

[REQ-CPP-109] Any use of raw pointer arithmetic or `reinterpret_cast` MUST be isolated in a `detail/` header with a documented safety justification.

### 10.4 Filesystem Handling

[REQ-CPP-110] File operations MUST use `QDir`, `QFile`, `QFileInfo`, or `QSaveFile`. Raw POSIX `open()`/`read()`/`write()` MUST NOT be used:

```cpp
QSaveFile file{path};
if (file.open(QIODevice::WriteOnly)) {
    file.write(data);
    file.commit();          // Atomic write
}
```

[REQ-CPP-111] Temporary files MUST be created with `QTemporaryFile` or `QTemporaryDir`:

```cpp
QTemporaryFile tmp;
if (tmp.open()) {
    // tmp.fileName() is unique and securely created
}
```

### 10.5 Privilege Separation

[REQ-CPP-112] Code that requires elevated privileges MUST be isolated in a separate process (daemon) that exposes a D-Bus API with PolicyKit authorization (SPEC-PLATFORM-001 §3.2).

---

## 11. Logging and Diagnostics

### 11.1 Logging Categories

[REQ-CPP-113] Every component MUST declare a named logging category using `Q_LOGGING_CATEGORY`:

```cpp
// In a .cpp file:
Q_LOGGING_CATEGORY(lingmoThemeLog, "lingmo.theme")

// Usage:
qCDebug(lingmoThemeLog) << "Theme loaded:" << name;
qCWarning(lingmoThemeLog) << "Theme file not found:" << path;
qCCritical(lingmoThemeLog) << "Theme system corrupted";
```

[REQ-CPP-114] Logging category names MUST follow the pattern `lingmo.<component>.<subsystem>`.

### 11.2 Debug Output Rules

[REQ-CPP-115] `qDebug()` MUST be used for developer-facing diagnostics. It is stripped in Release builds if `QT_NO_DEBUG_OUTPUT` is defined.

[REQ-CPP-116] `qWarning()` MUST be used for recoverable runtime issues that should be investigated.

[REQ-CPP-117] `qCritical()` MUST be used for severe errors that may cause data loss or process termination.

### 11.3 Production Logging

[REQ-CPP-118] Log messages MUST NOT contain personally identifiable information (PII):

```cpp
// CORRECT
qCWarning(log) << "Login failed for user" << hashedUserId;

// FORBIDDEN
qCWarning(log) << "Login failed for user" << plaintextUserId;
```

[REQ-CPP-119] Log messages MUST be structured for machine parsing:

```cpp
qCWarning(log) << "Config load failed:" << path << "error:" << errorString;
```

### 11.4 Crash Reporting Hooks

[REQ-CPP-120] Every Lingmo process MUST install a crash handler via `qInstallMessageHandler` or the Lingmo crash handler framework (SPEC-APP-005).

[REQ-CPP-121] The crash handler MUST generate a crash report containing:

- Signal/exception type
- Stack trace (backtrace)
- Application version
- Loaded libraries
- Environment variables (excluding secrets)

---

## 12. Testing Requirements

### 12.1 Unit Test Expectations

[REQ-CPP-122] Every public class MUST have a corresponding unit test file:

| Source File | Test File |
|---|---|
| `Theme.cpp` | `tst_Theme.cpp` |
| `Config.cpp` | `tst_Config.cpp` |

[REQ-CPP-123] Every public function MUST have at least one test case covering its expected behavior.

[REQ-CPP-124] Test names MUST describe the scenario and expected outcome:

```cpp
void tst_Theme::testLoadValidTheme_returnsTrue() { ... }
void tst_Theme::testLoadMissingTheme_returnsFalse() { ... }
```

### 12.2 Testable Design

[REQ-CPP-125] Dependencies SHOULD be injectable to enable testing:

```cpp
// Testable — accepts dependency
class ThemeLoader {
public:
    explicit ThemeLoader(QObject* storage) : m_storage{storage} {}
};
```

[REQ-CPP-126] Global state MUST NOT be required for unit testing. Singletons MUST support replacement in tests:

```cpp
class Theme {
public:
    static Theme* instance();
    static void setInstance(Theme* theme);   // For testing only
};
```

### 12.3 Mock Requirements

[REQ-CPP-127] Interfaces that are expected to have multiple implementations (backends, providers, storage) SHOULD be defined as abstract classes suitable for mocking:

```cpp
class ThemeStorage {
public:
    virtual ~ThemeStorage() = default;
    virtual auto load(const QString& name) -> std::optional<Theme> = 0;
    virtual auto save(const Theme& theme) -> bool = 0;
};
```

[REQ-CPP-128] Mock implementations MUST use Qt Test's mock support or a lightweight hand-written mock:

```cpp
class MockThemeStorage : public ThemeStorage {
    auto load(const QString& name) -> std::optional<Theme> override {
        return m_themes.contains(name) ? m_themes[name] : std::nullopt;
    }
    QHash<QString, Theme> m_themes;
};
```

### 12.4 Deterministic Behavior

[REQ-CPP-129] Tests MUST be deterministic. Randomness, wall-clock time, and real I/O MUST be replaced with seeded generators, mocked clocks, and in-memory backends.

---

## 13. Deprecated Practices

[REQ-CPP-130] The following patterns are DEPRECATED and MUST NOT appear in new code. Existing occurrences MUST be flagged in code reviews and migrated:

### 13.1 Raw Owning Pointers

[REQ-CPP-131] Raw pointers MUST NOT own resources. Every owning pointer MUST use a smart pointer or QObject parent ownership.

### 13.2 C-Style Casts

[REQ-CPP-132] C-style casts (`(Type)value`) are FORBIDDEN:

| Intent | Correct Cast |
|---|---|
| Numeric conversion | `static_cast<Type>(value)` |
| Remove const | `const_cast<Type>(value)` |
| Bitwise reinterpret | `reinterpret_cast<Type>(value)` |
| QObject cast | `qobject_cast<Type>(value)` |

### 13.3 C-Style Arrays

[REQ-CPP-133] C-style arrays are FORBIDDEN:

```cpp
int data[100];                    // FORBIDDEN
std::array<int, 100> data;        // CORRECT
QVarLengthArray<int> data(100);   // CORRECT (stack-allocated with heap fallback)
```

### 13.4 Global Mutable State

[REQ-CPP-134] Mutable global variables are FORBIDDEN. Use `Q_GLOBAL_STATIC` for lazy-initialized globals:

```cpp
Q_GLOBAL_STATIC(ThemeManager, s_themeManager)
```

### 13.5 Singleton Abuse

[REQ-CPP-135] The singleton pattern MUST be used ONLY when:

1. The resource is inherently singular (display, audio output)
2. The singleton can be replaced in unit tests

### 13.6 QObject Copying

[REQ-CPP-136] QObject-derived classes MUST be non-copyable (REQ-CPP-022).

### 13.7 Exposing Private Implementation Details

[REQ-CPP-137] Private class definitions (`_p.h`) MUST NOT be included from any public header (SPEC-REPO-002 §3.4).

[REQ-CPP-138] The `d` pointer MUST be in the `private` section of the public class.

### 13.8 Signal-Slot Overloads

[REQ-CPP-139] Signal and slot overloads (same name, different parameters) are DISCOURAGED. When unavoidable, PMF casts must be documented:

```cpp
connect(obj, qOverload<int>(&Widget::valueChanged), this, &Handler::onValueChanged);
```

---

## 14. Compliance Checklist

### 14.1 Language Standard

| ID | Requirement | Verification |
|---|---|---|
| REQ-CPP-001 | C++20 standard | CMake scan |
| REQ-CPP-002 | No compiler extensions | Compiler audit |
| REQ-CPP-004 | No modules, coroutines, jthread | Automated grep |
| REQ-CPP-005 | Compiles on GCC 12, Clang 16, MSVC | CI matrix |
| REQ-CPP-006 | No legacy patterns (NULL, auto_ptr, etc.) | Automated grep |

### 14.2 General Programming

| ID | Requirement | Verification |
|---|---|---|
| REQ-CPP-007 | Brace initialization preferred | Code review |
| REQ-CPP-010 | Const correctness | Compiler warnings |
| REQ-CPP-013 | RAII for all resources | Code review |
| REQ-CPP-014 | Single owner per object | Code review |
| REQ-CPP-016 | QPointer for QObject tracking | Code review |
| REQ-CPP-017 | No variable shadowing | Compiler `-Wshadow` |

### 14.3 Qt Integration

| ID | Requirement | Verification |
|---|---|---|
| REQ-CPP-020 | QObject for signals/slots | Compiler check |
| REQ-CPP-022 | QObject non-copyable | Compiler check |
| REQ-CPP-023 | Parent-child lifetime | Code review |
| REQ-CPP-026 | Q_OBJECT in class | Compiler MOC check |
| REQ-CPP-028 | PMF connect syntax | Automated grep |
| REQ-CPP-030 | Context object in connect | Automated lint |
| REQ-CPP-033 | Q_PROPERTY for observable properties | Code review |

### 14.4 Memory Management

| ID | Requirement | Verification |
|---|---|---|
| REQ-CPP-036 | unique_ptr for exclusive ownership | Code review |
| REQ-CPP-038 | shared_ptr justified | Code review |
| REQ-CPP-041 | QPointer for async QObject tracking | Code review |
| REQ-CPP-043 | QObjects via new, not make_unique | Automated scan |
| REQ-CPP-044 | deleteLater() for deferred deletion | Code review |
| REQ-CPP-047 | No malloc, raw new/delete | Automated grep |

### 14.5 Containers

| ID | Requirement | Verification |
|---|---|---|
| REQ-CPP-048 | QString for text, QByteArray for binary | Code review |
| REQ-CPP-049 | QList default, std::vector for ABI | Code review |
| REQ-CPP-050 | std::optional for optional returns | Code review |
| REQ-CPP-052 | reserve() when size known | Code review |
| REQ-CPP-054 | QString::arg() for concatenation | Code review |

### 14.6 ABI Stability

| ID | Requirement | Verification |
|---|---|---|
| REQ-CPP-057 | PIMPL for public classes | Code review |
| REQ-CPP-059 | No virtual added after release | API diff |
| REQ-CPP-067 | No ABI breaks in minor versions | abidiff |

### 14.7 Error Handling

| ID | Requirement | Verification |
|---|---|---|
| REQ-CPP-070 | Exceptions caught at library boundary | Code review |
| REQ-CPP-072 | No throw from destructor | Code review |
| REQ-CPP-073 | std::optional or std::expected | Code review |
| REQ-CPP-077 | Q_ASSERT for invariants | Code review |

### 14.8 Concurrency

| ID | Requirement | Verification |
|---|---|---|
| REQ-CPP-081 | Qt thread model | Code review |
| REQ-CPP-083 | Worker-object pattern | Code review |
| REQ-CPP-084 | No QThread subclassing | Automated grep |
| REQ-CPP-088 | Mutex protection for shared data | Code review |
| REQ-CPP-089 | QMutexLocker pattern | Automated grep |

### 14.9 Performance

| ID | Requirement | Verification |
|---|---|---|
| REQ-CPP-094 | Move semantics for stored params | Code review |
| REQ-CPP-096 | Avoid hot-path allocations | Profiling |
| REQ-CPP-098 | QStringLiteral for constants | Automated grep |
| REQ-CPP-100 | reserve() calls | Code review |

### 14.10 Security

| ID | Requirement | Verification |
|---|---|---|
| REQ-CPP-104 | No unsafe APIs | Automated grep |
| REQ-CPP-105 | Input validation | Code review |
| REQ-CPP-110 | QFile, not raw POSIX | Automated grep |
| REQ-CPP-111 | QTemporaryFile for temp files | Automated grep |

### 14.11 Testing

| ID | Requirement | Verification |
|---|---|---|
| REQ-CPP-122 | Test per public class | File existence check |
| REQ-CPP-125 | Injectable dependencies | Code review |
| REQ-CPP-126 | Replaceable singletons | Code review |
| REQ-CPP-129 | Deterministic tests | CI flakiness check |

### 14.12 Deprecated Practices

| ID | Requirement | Verification |
|---|---|---|
| REQ-CPP-131 | No raw owning pointers | Automated grep |
| REQ-CPP-132 | No C-style casts | Compiler `-Wold-style-cast` |
| REQ-CPP-133 | No C-style arrays | Automated grep |
| REQ-CPP-134 | No mutable globals | Automated grep |
| REQ-CPP-136 | QObject non-copyable | Compiler check |

---

*Cross-ref*: SPEC-CORE-001 (Naming Convention), SPEC-REPO-002 (Header Organization), SPEC-REPO-003 (CMake Organization), SPEC-CORE-002 (Authoring Standard), SPEC-CORE-003 (Glossary)
