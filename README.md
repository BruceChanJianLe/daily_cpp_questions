> A quick refresher and daily reading!

## Table of Contents

- [auto and type deduction](#auto-and-type-deduction)
- [static keyword](#static-keyword)
- [Polymorphism, inheritance, virtual functions](#polymorphism-inheritance-virtual-functions)
- [Lambda functions](#lambda-functions)
- [const qualifier](#const-qualifier)
- [Best practices in modern C++](#best-practices-in-modern-c)
- [Smart pointers](#smart-pointers)
- [References and move semantics](#references-and-move-semantics)
- [C++20](#c20)
- [Special functions](#special-functions)
- [Object-oriented design](#object-oriented-design)
- [Observable behaviors](#observable-behaviors)
- [Standard Template Library](#standard-template-library)
- [Miscellaneous](#miscellaneous)
- [C++ and algorithmic complexities](#c-and-algorithmic-complexities)

---

## auto and type deduction

<details>
<summary>1. Explain auto type deduction</summary>

`auto` deduces the type of a variable from its initializer at compile time, following the same rules as template type deduction. References and cv-qualifiers (`const`/`volatile`) are stripped unless explicitly specified.

```cpp
int x = 5;
auto a = x;        // int
auto& b = x;       // int&
const auto c = x;  // const int
```

</details>

<details>
<summary>
2. When can `auto` deduce undesired types?
</summary>

- With `std::initializer_list`: `auto x = {1};` deduces `std::initializer_list<int>`, not `int`.
- With proxy types: `auto bit = std::vector<bool>{true, false}[0];` deduces a proxy object, not `bool`.
- When iterating: `auto val = map[key];` copies instead of referencing if you forget `&`.
- `const char*` vs `std::string`: `auto s = "hello";` gives `const char*`, not `std::string`.

</details>

<details>
<summary>3. What are the advantages of using `auto`?</summary>

- Avoids redundant type repetition (less verbose).
- Prevents unintentional implicit conversions.
- Adapts automatically to type changes during refactoring.
- Ensures variables are always initialized (no default-init pitfalls).
- Makes code more generic and easier to maintain.

</details>

<details>
<summary>4. What is the type of `myCollection` after the following declaration?</summary>

```cpp
std::map<std::string, std::vector<int>> myCollection;
auto myCollection2 = myCollection;
```

`myCollection2` is `std::map<std::string, std::vector<int>>` — a full copy. `auto` strips references and cv-qualifiers but preserves the full type.

</details>

<details>
<summary>5. What are trailing return types?</summary>

A syntax where the return type is specified after the parameter list using `->`, introduced in C++11. Useful when the return type depends on parameter types.

```cpp
auto add(int a, int b) -> int { return a + b; }

template<typename T, typename U>
auto multiply(T a, U b) -> decltype(a * b) { return a * b; }
```

</details>

<details>
<summary>6. Explain `decltype`!</summary>

`decltype` queries the type of an expression without evaluating it. Unlike `auto`, it preserves references and cv-qualifiers exactly.

```cpp
int x = 5;
int& ref = x;
decltype(x)   a = x;    // int
decltype(ref) b = x;    // int&
decltype((x)) c = x;    // int& (parenthesized expression → lvalue ref)
```

</details>

<details>
<summary>7. When to use `decltype(auto)`?</summary>

Use `decltype(auto)` when you want to deduce the return type of a function while preserving references and cv-qualifiers — something `auto` alone would strip.

```cpp
int x = 42;
int& getRef() { return x; }

auto           f() { return getRef(); }  // returns int (copy)
decltype(auto) g() { return getRef(); }  // returns int& (reference preserved)
```

</details>

<details>
<summary>8. Which data type do you get when you add two `bool`s?</summary>

`int`. Due to integral promotion, `bool` values are promoted to `int` before arithmetic operations. `true + true` yields `2` of type `int`.

```cpp
bool a = true, b = true;
auto c = a + b;  // int, value = 2
```

</details>

---

## static keyword

<details>
<summary>9. What does a `static` member variable mean?</summary>

A `static` member variable is shared across all instances of a class — there is only one copy regardless of how many objects exist. It must be defined outside the class (except `inline static` in C++17).

```cpp
struct Counter {
    static int count;  // declaration
};
int Counter::count = 0;  // definition
```

</details>

<details>
<summary>10. What does a `static` member function mean?</summary>

A `static` member function belongs to the class, not to any instance. It has no `this` pointer and can only access `static` members. It can be called without an object.

```cpp
struct Foo {
    static void greet() { std::cout << "Hello\n"; }
};
Foo::greet();  // no object needed
```

</details>

<details>
<summary>11. What is the `static` initialization order fiasco?</summary>

When two static objects in different translation units have a dependency, the order of their initialization is undefined. If object A (in TU1) depends on object B (in TU2), B might not be initialized yet when A's constructor runs.

```cpp
// tu1.cpp
extern int b;
int a = b + 1;  // undefined: b may not be initialized yet

// tu2.cpp
int b = 42;
```

</details>

<details>
<summary>12. How to solve the static initialization order fiasco?</summary>

Use the **construct-on-first-use** idiom: wrap the static variable inside a function. Local static variables are guaranteed to be initialized on first call (since C++11, this is also thread-safe).

```cpp
int& getB() {
    static int b = 42;
    return b;
}
int a = getB() + 1;  // safe: getB() ensures b is initialized first
```

</details>

---

## Polymorphism, inheritance, virtual functions

<details>
<summary>13. Difference between function overloading and overriding?</summary>

- **Overloading**: multiple functions with the same name but different parameter lists in the same scope. Resolved at compile time (static dispatch).
- **Overriding**: a derived class provides its own implementation of a `virtual` function from the base class. Resolved at runtime (dynamic dispatch).

```cpp
void foo(int);    // overload
void foo(double); // overload

struct Base { virtual void bar(); };
struct Derived : Base { void bar() override; }; // override
```

</details>

<details>
<summary>14. What is a `virtual` function?</summary>

A member function declared with `virtual` in a base class that can be overridden in derived classes. When called through a pointer or reference to the base class, the most-derived override is called at runtime via the vtable.

```cpp
struct Animal {
    virtual void speak() { std::cout << "..."; }
};
struct Dog : Animal {
    void speak() override { std::cout << "Woof"; }
};
Animal* a = new Dog;
a->speak(); // prints "Woof"
```

</details>

<details>
<summary>15. What is the `override` keyword and its advantages?</summary>

`override` (C++11) explicitly marks a virtual function as overriding a base class function. The compiler emits an error if no matching virtual function exists in the base — catching typos, signature mismatches, and accidental hiding.

```cpp
struct Base { virtual void foo(int); };
struct Derived : Base {
    void foo(int) override;    // OK
    // void foo(float) override; // error: no matching base function
};
```

</details>

<details>
<summary>16. Explain covariant return types and use-cases</summary>

A covariant return type allows an overriding virtual function to return a pointer or reference to a more-derived type than the base function's return type.

```cpp
struct Base {
    virtual Base* clone() { return new Base(*this); }
};
struct Derived : Base {
    Derived* clone() override { return new Derived(*this); } // covariant
};
```

Useful in the **prototype pattern** — callers holding a `Derived*` get back a `Derived*` without casting.

</details>

<details>
<summary>17. What is virtual inheritance and when to use it?</summary>

Virtual inheritance ensures only one shared instance of a base class exists when multiple paths lead to the same base (the diamond problem). Use it when multiple classes inherit from a common base and you want to avoid ambiguity and duplication.

```cpp
struct A { int x; };
struct B : virtual A {};
struct C : virtual A {};
struct D : B, C {};  // only one A::x exists in D
```

</details>

<details>
<summary>18. Should we always use virtual inheritance?</summary>

No. It adds overhead: a virtual base pointer (vbptr) per class in the hierarchy, larger object size, and more complex construction order. Only use it when you truly have the diamond problem and need a single shared base subobject.

</details>

<details>
<summary>19. Output and expectations of a sample program?</summary>

*(Context-dependent — see book example.)* Generally, virtual dispatch through a base pointer calls the most-derived override. Without `virtual`, the base version is called based on the static type of the pointer.

</details>

<details>
<summary>20. Can you access public/protected members with private inheritance?</summary>

Within the derived class itself, yes — you can access public and protected members of the private base. But outside the derived class, the inherited members are inaccessible (they become `private` from the outside world's perspective).

```cpp
struct Base { void pub() {} };
struct Derived : private Base {
    void foo() { pub(); }  // OK inside Derived
};
Derived d;
// d.pub(); // error: inaccessible
```

</details>

<details>
<summary>21. What is private inheritance used for?</summary>

"Implemented-in-terms-of" relationships — reusing a base class's implementation without exposing its interface. It is an alternative to composition when you need access to `protected` members or need to override `virtual` functions of the base.

```cpp
struct Timer { virtual void onTick(); };
struct Widget : private Timer {   // Widget uses Timer's machinery
    void onTick() override { /* ... */ }
};
```

</details>

<details>
<summary>22. Can you call a `virtual` function from a constructor/destructor?</summary>

You can call it, but virtual dispatch does **not** work as expected. During construction/destruction, the dynamic type is the type currently being constructed/destroyed, so the base version (not an override) is called.

```cpp
struct Base {
    Base() { foo(); }        // calls Base::foo, NOT Derived::foo
    virtual void foo() { std::cout << "Base\n"; }
};
struct Derived : Base {
    void foo() override { std::cout << "Derived\n"; }
};
Derived d; // prints "Base"
```

</details>

<details>
<summary>23. What role does a `virtual` destructor play?</summary>

It ensures that when a derived object is deleted through a base class pointer, the derived destructor is called first (then base). Without it, only the base destructor runs, causing resource leaks.

```cpp
struct Base { virtual ~Base() {} };
struct Derived : Base { ~Derived() { /* cleanup */ } };
Base* p = new Derived;
delete p; // calls ~Derived then ~Base — correct
```

</details>

<details>
<summary>24. Can we inherit from standard containers like `std::vector`?</summary>

Technically yes, but it is strongly discouraged. Standard containers have no virtual destructors, so deleting via a base pointer is undefined behavior. Prefer composition or use private inheritance if you must reuse internals.

</details>

<details>
<summary>25. What does a strong type mean and its advantages?</summary>

A strong type is a distinct type that wraps a primitive to prevent accidental interchange with other types of the same underlying representation.

```cpp
struct Meters  { explicit Meters(double v)  : value(v) {} double value; };
struct Seconds { explicit Seconds(double v) : value(v) {} double value; };
// Meters m = Seconds{5.0}; // compile error — no accidental mix-up
```

Advantages: catches unit/logic errors at compile time, improves readability.

</details>

<details>
<summary>26. Explain short-circuit evaluation</summary>

In `&&` and `||`, the right operand is only evaluated if the left operand does not determine the result:
- `a && b`: if `a` is `false`, `b` is not evaluated.
- `a || b`: if `a` is `true`, `b` is not evaluated.

```cpp
int* p = nullptr;
if (p != nullptr && *p > 0) { }  // safe: *p not reached if p is null
```

</details>

<details>
<summary>27. What is a destructor and how can we overload it?</summary>

A destructor is a special member function called when an object's lifetime ends, used to release resources. It takes no parameters and returns nothing — **it cannot be overloaded**. A class can have only one destructor.

```cpp
struct Foo {
    ~Foo() { std::cout << "destroyed\n"; }
};
```

</details>

<details>
<summary>28. Output of a code sample and why?</summary>

*(Context-dependent — see book example.)* Key concepts usually tested: object slicing, virtual dispatch, order of construction/destruction, or copy semantics.

</details>

<details>
<summary>29. How to use the `= delete` specifier?</summary>

`= delete` explicitly disables a function. The compiler emits an error if that function is called or selected by overload resolution.

```cpp
struct NonCopyable {
    NonCopyable(const NonCopyable&) = delete;
    NonCopyable& operator=(const NonCopyable&) = delete;
};

void process(double) = delete; // prevent implicit conversion from int
void process(int x) { /* ... */ }
```

</details>

---

## Lambda functions

<details>
<summary>30. What are immediately invoked lambda functions?</summary>

A lambda that is defined and called in the same expression — no name needed. Useful for initializing `const` variables with complex logic.

```cpp
const int result = [](int x) { return x * x; }(5); // result = 25

const std::string msg = []() {
    if (condition) return "yes";
    return "no";
}();
```

</details>

<details>
<summary>31. What kind of captures are available for lambdas?</summary>

| Capture | Meaning |
|---------|---------|
| `[]` | No capture |
| `[=]` | Capture all by value |
| `[&]` | Capture all by reference |
| `[x]` | Capture `x` by value |
| `[&x]` | Capture `x` by reference |
| `[=, &x]` | All by value, `x` by reference |
| `[&, x]` | All by reference, `x` by value |
| `[this]` | Capture `this` pointer |
| `[*this]` | Capture `*this` by value (C++17) |

</details>

---

## const qualifier

<details>
<summary>32. Output of code sample and why?</summary>

*(Context-dependent — see book example.)* Common trap: `const` on a pointer vs. pointer-to-const. `const int* p` — the pointed-to value is const. `int* const p` — the pointer itself is const.

</details>

<details>
<summary>33. Advantages of using `const` local variables?</summary>

- Communicates intent: value will not change.
- Enables compiler optimizations.
- Catches accidental mutation at compile time.
- Extends lifetime of temporaries when binding a `const` reference.

</details>

<details>
<summary>34. Is it good to have `const` members in a class?</summary>

Generally no. `const` data members prevent the compiler from generating a useful copy/move assignment operator (the class becomes non-assignable), which breaks many standard library requirements and container usage. Prefer getter functions returning `const` references instead.

</details>

<details>
<summary>35. Does it make sense to return `const` objects by value?</summary>

No. In modern C++ (C++11+), returning `const` by value inhibits move semantics and NRVO (Named Return Value Optimization), causing unnecessary copies. Avoid it.

```cpp
const std::string foo();  // bad: prevents move/NRVO
std::string bar();        // good
```

</details>

<details>
<summary>36. How should you return `const` pointers from functions?</summary>

Return `const T*` (pointer to const) to prevent callers from modifying the pointed-to data through the returned pointer. `T* const` (const pointer) is useless by value since the caller gets their own copy of the pointer.

```cpp
const char* getName() { return "Alice"; }  // caller can't modify the string
```

</details>

<details>
<summary>37. Should functions return `const` references?</summary>

Only when returning a reference to a member or something that outlives the function call. Never return a `const` reference to a local variable (dangling reference). Returning `const&` avoids copying large objects but ties caller lifetime to the object's lifetime.

</details>

<details>
<summary>38. Should you take plain old data types by `const` reference?</summary>

No. For cheap-to-copy types (`int`, `double`, `bool`, pointers), taking by value is equally or more efficient — no indirection overhead, and the compiler can optimize better. `const&` adds a pointer dereference for no benefit.

</details>

<details>
<summary>39. Should you pass objects by `const` reference?</summary>

Yes, for large or non-trivially-copyable objects. `const T&` avoids a copy while preventing modification. The guideline: pass by value if ≤ pointer-size or cheaply copyable; pass by `const&` otherwise.

</details>

<details>
<summary>40. Does function declaration signature match definition?</summary>

A `const` at the top level of a by-value parameter is ignored in the declaration but is meaningful in the definition (it prevents modification of the local copy). These are the same signature:

```cpp
void foo(int x);          // declaration
void foo(const int x) {}  // definition — same function, const is local detail
```

</details>

<details>
<summary>41. Explain `consteval` and `constinit`</summary>

- **`consteval`** (C++20): declares an **immediate function** — it *must* be evaluated at compile time. Unlike `constexpr`, calling it at runtime is a compile error.
- **`constinit`** (C++20): asserts that a variable has **static initialization** (not dynamic). Prevents the static initialization order fiasco. The variable is not necessarily `const` — it can still be modified at runtime.

```cpp
consteval int square(int n) { return n * n; }
constinit int x = square(5); // x = 25, initialized at compile time
x = 10; // OK: constinit doesn't mean const
```

</details>

---

## Best practices in modern C++

<details>
<summary>42. What is aggregate initialization?</summary>

Direct initialization of aggregates (arrays, structs/classes with no user-provided constructors, no private/protected members, no base classes in C++11) using brace-enclosed lists.

```cpp
struct Point { int x; int y; };
Point p = {1, 2};  // aggregate initialization
Point q{3, 4};     // same (brace-init)
```

</details>

<details>
<summary>43. What are explicit constructors and their advantages?</summary>

`explicit` prevents a constructor from being used for implicit conversions or copy-initialization. Avoids surprises where a single-argument constructor silently converts unrelated types.

```cpp
struct Wrapper {
    explicit Wrapper(int x) {}
};
Wrapper w = 5;  // error with explicit
Wrapper w{5};   // OK
```

</details>

<details>
<summary>44. What are user-defined literals?</summary>

Suffixes on literals that invoke a user-defined operator to produce a typed value. Defined with `operator""`.

```cpp
long double operator"" _km(long double d) { return d * 1000.0; }
auto dist = 1.5_km; // 1500.0

// Standard library examples:
using namespace std::literals;
auto s = "hello"s;  // std::string
auto d = 100ms;     // std::chrono::milliseconds
```

</details>

<details>
<summary>45. Why use `nullptr` instead of `NULL` or `0`?</summary>

`nullptr` is a typed null pointer constant (`std::nullptr_t`) that only converts to pointer types. `NULL`/`0` are integers and can cause ambiguous overload resolution.

```cpp
void foo(int);
void foo(int*);
foo(NULL);    // ambiguous or calls foo(int) — surprising
foo(nullptr); // unambiguously calls foo(int*)
```

</details>

<details>
<summary>46. What advantages does `alias` have over `typedef`?</summary>

`using` (alias declaration, C++11) supports templates directly, is more readable, and has consistent syntax:

```cpp
// typedef cannot template directly
template<typename T>
using Vec = std::vector<T>;   // alias template — not possible with typedef

using Fn = void(*)(int);      // clearer than: typedef void(*Fn)(int);
```

</details>

<details>
<summary>47. Advantages of scoped `enum`s over unscoped?</summary>

`enum class` (C++11):
- Enumerators do not leak into the enclosing scope — no name collisions.
- No implicit conversion to `int` — type-safe.
- Can be forward-declared.

```cpp
enum class Color { Red, Green, Blue };
Color c = Color::Red;  // must qualify
// int x = Color::Red; // error: no implicit conversion
```

</details>

<details>
<summary>48. Should you explicitly delete unused special functions?</summary>

Yes — it documents intent and prevents surprising implicit generation. If a class manages a resource, deleting copy operations makes misuse a compile error rather than a runtime bug.

```cpp
struct FileHandle {
    FileHandle(const FileHandle&) = delete;
    FileHandle& operator=(const FileHandle&) = delete;
};
```

</details>

<details>
<summary>49. How to use the `= delete` specifier?</summary>

Apply `= delete` to any function declaration to make calling it a compile error. Common uses: disabling copy/move, preventing implicit conversions, and removing overloads.

```cpp
struct NoCopy {
    NoCopy(const NoCopy&) = delete;
};
void foo(int) {}
void foo(double) = delete; // prevent double argument
```

</details>

<details>
<summary>50. What is a trivial class?</summary>

A class is trivial if it has:
- A trivial default constructor (does nothing / compiler-generated).
- Trivial copy/move constructors and assignment operators.
- A trivial destructor.
- No virtual functions or virtual base classes.

Trivial types can be safely copied with `memcpy` and have C-compatible layout. Checked with `std::is_trivial<T>`.

</details>

---

## Smart pointers

<details>
<summary>51. Explain the RAII idiom</summary>

**Resource Acquisition Is Initialization** — bind a resource's lifetime to an object's lifetime. Acquire in the constructor, release in the destructor. Guarantees cleanup even when exceptions are thrown.

```cpp
struct FileGuard {
    FILE* f;
    FileGuard(const char* path) : f(fopen(path, "r")) {}
    ~FileGuard() { if (f) fclose(f); }
};
// fclose is called automatically when FileGuard goes out of scope
```

</details>

<details>
<summary>52. When should we use unique pointers?</summary>

`std::unique_ptr` expresses **exclusive ownership**. Use it when:
- One owner manages the resource lifetime.
- You need heap allocation with automatic cleanup.
- Transferring ownership (via `std::move`) is needed but sharing is not.

It has zero overhead compared to a raw pointer.

</details>

<details>
<summary>53. Reasons to use shared pointers?</summary>

`std::shared_ptr` expresses **shared ownership** via reference counting. Use it when:
- Multiple owners need the same object to stay alive.
- Ownership is transferred to callbacks or stored in containers where lifetime is unclear.

Be aware: reference counting overhead, potential for cycles (use `weak_ptr` to break them).

</details>

<details>
<summary>54. When to use a weak pointer?</summary>

`std::weak_ptr` holds a non-owning reference to a `shared_ptr`-managed object. Use it to:
- Break cyclic references (e.g., parent/child graphs).
- Observe an object without extending its lifetime.

```cpp
std::weak_ptr<Foo> wp = sharedFoo;
if (auto sp = wp.lock()) { // check if still alive
    sp->doSomething();
}
```

</details>

<details>
<summary>55. Advantages of `std::make_shared` and `std::make_unique`?</summary>

- **Exception safety**: no raw `new` — avoids leaks if constructor throws.
- **`make_shared` efficiency**: allocates control block and object in a single allocation (faster, better cache locality).
- **Cleaner syntax**: no need to repeat the type.

```cpp
auto p = std::make_unique<Foo>(args);  // preferred over: unique_ptr<Foo>(new Foo(args))
auto s = std::make_shared<Bar>(args);
```

</details>

<details>
<summary>56. Should you use smart pointers over raw pointers always?</summary>

For owning pointers, yes. For non-owning/observing pointers (function parameters, iterating), raw pointers or references are appropriate and clearer. The guideline: **raw pointers should never own resources**.

</details>

<details>
<summary>57. When and why initialize pointers to `nullptr`?</summary>

Always initialize pointers that are not immediately assigned. Uninitialized pointers have indeterminate values — reading or dereferencing them is undefined behavior. A `nullptr` pointer is safe to check and compare.

```cpp
int* p = nullptr;
if (p) { *p = 5; }  // safe: checked before use
```

</details>

---

## References and move semantics

<details>
<summary>58. What does `std::move` move?</summary>

`std::move` does not move anything — it is an **unconditional cast to an rvalue reference** (`T&&`). It signals to the compiler that the object may be "moved from," enabling the move constructor/assignment operator to steal resources rather than copy them.

```cpp
std::string a = "hello";
std::string b = std::move(a); // a is now in a valid but unspecified state
```

</details>

<details>
<summary>59. What does `std::forward` forward?</summary>

`std::forward` performs **conditional casting**: it casts to an rvalue reference only if the argument was originally an rvalue. Used in perfect forwarding to preserve the value category of template arguments.

```cpp
template<typename T>
void wrapper(T&& arg) {
    target(std::forward<T>(arg)); // forwards lvalue as lvalue, rvalue as rvalue
}
```

</details>

<details>
<summary>60. Difference between universal and rvalue references?</summary>

- **Rvalue reference** (`T&&` where `T` is concrete): binds only to rvalues.
- **Universal reference / forwarding reference** (`T&&` in a deduced context): binds to both lvalues and rvalues. When an lvalue is passed, `T` deduces to `X&`, giving `X& &&` which collapses to `X&`.

```cpp
void foo(std::string&& s);  // rvalue reference only
template<typename T>
void bar(T&& t);            // universal reference
```

</details>

<details>
<summary>61. What is reference collapsing?</summary>

When references to references arise (e.g., via templates or `typedef`), C++ collapses them:
- `& &` → `&`
- `& &&` → `&`
- `&& &` → `&`
- `&& &&` → `&&`

This is the mechanism that makes universal references and `std::forward` work.

</details>

<details>
<summary>62. When are `constexpr` functions evaluated?</summary>

A `constexpr` function *may* be evaluated at compile time when called with constant expressions and the result is needed in a constant context (e.g., array size, template argument). Otherwise it is evaluated at runtime like a regular function.

```cpp
constexpr int square(int n) { return n * n; }
int arr[square(5)];          // compile-time
int x = square(runtimeVal);  // runtime
```

</details>

<details>
<summary>63. When should you declare functions as `noexcept`?</summary>

Declare `noexcept` when a function genuinely cannot throw (or you are willing for `std::terminate` to be called if it does). Key cases:
- Move constructors and move assignment operators (enables STL optimizations — e.g., `std::vector` reallocation uses move only if `noexcept`).
- Destructors (implicitly `noexcept`).
- Swap functions.

</details>

---

## C++20

<details>
<summary>64. What are concepts in C++?</summary>

Concepts (C++20) are named compile-time predicates that constrain template parameters. They replace SFINAE and `enable_if` with readable, expressive constraints.

```cpp
template<typename T>
concept Addable = requires(T a, T b) { a + b; };

template<Addable T>
T add(T a, T b) { return a + b; }
```

</details>

<details>
<summary>65. What are the available standard attributes?</summary>

Standard attributes are placed in `[[...]]`. Common ones:

| Attribute | Purpose |
|-----------|---------|
| `[[nodiscard]]` | Warn if return value is ignored |
| `[[maybe_unused]]` | Suppress unused variable/function warnings |
| `[[deprecated]]` | Mark as deprecated with optional message |
| `[[fallthrough]]` | Suppress warning for intentional switch fallthrough |
| `[[likely]]` / `[[unlikely]]` | Optimization hints for branch prediction (C++20) |
| `[[noreturn]]` | Function never returns |

</details>

<details>
<summary>66. What is 3-way comparison?</summary>

The spaceship operator `<=>` (C++20) computes the ordering relationship between two values in a single call, returning a comparison category type:
- `std::strong_ordering` (for integers)
- `std::weak_ordering`
- `std::partial_ordering` (for floats, NaN)

```cpp
auto result = 3 <=> 5; // std::strong_ordering::less
// Defaulting it generates all 6 comparison operators:
auto operator<=>(const Foo&) const = default;
```

</details>

<details>
<summary>67. Explain `consteval` and `constinit`</summary>

*(See Q41 for full answer.)*
- `consteval`: immediate function — must run at compile time.
- `constinit`: variable must have static/constant initialization — prevents dynamic init order issues while still allowing runtime modification.

</details>

<details>
<summary>68. What are modules and their advantages?</summary>

Modules (C++20) replace textual `#include` with a compiled, importable unit:

```cpp
export module math;
export int add(int a, int b) { return a + b; }

// consumer:
import math;
```

Advantages:
- Faster compilation (no repeated parsing of headers).
- No macro leakage between modules.
- No include-order dependencies.
- Cleaner separation of interface and implementation.

</details>

---

## Special functions

<details>
<summary>69. Explain the rule of three</summary>

If a class needs a custom **destructor**, **copy constructor**, or **copy assignment operator**, it almost certainly needs all three. Typically applies when the class manually manages a resource (raw pointer, file handle, etc.).

```cpp
struct Buffer {
    Buffer(const Buffer&);             // copy constructor
    Buffer& operator=(const Buffer&);  // copy assignment
    ~Buffer();                         // destructor
};
```

</details>

<details>
<summary>70. Explain the rule of five</summary>

Extends the rule of three to include **move constructor** and **move assignment operator** (C++11). If you define any of the five, consider defining all five for correct and efficient resource management.

```cpp
struct Buffer {
    Buffer(const Buffer&);
    Buffer& operator=(const Buffer&);
    Buffer(Buffer&&) noexcept;
    Buffer& operator=(Buffer&&) noexcept;
    ~Buffer();
};
```

</details>

<details>
<summary>71. Explain the rule of zero</summary>

The preferred modern approach: design classes so they need **none** of the five special functions. Delegate resource management to RAII types (`std::unique_ptr`, `std::string`, etc.). The compiler-generated defaults are then correct.

```cpp
struct Person {
    std::string name;           // manages its own memory
    std::unique_ptr<int> data;  // manages its own resource
    // No need for custom destructor, copy/move — defaults are correct
};
```

</details>

<details>
<summary>72. What does `std::move` move?</summary>

*(See Q58.)* It is a cast to `T&&`. The actual resource transfer happens in the move constructor/assignment operator of the target type.

</details>

<details>
<summary>73. What is a destructor and how can we overload it?</summary>

*(See Q27.)* A destructor cannot be overloaded — a class has exactly one destructor, taking no parameters.

</details>

<details>
<summary>74. Should you explicitly delete unused special functions?</summary>

Yes, when the default behavior would be incorrect or dangerous. Deleting them makes misuse a compile error and documents the design decision. If a class should not be copied (e.g., it wraps a unique OS handle), explicitly deleting the copy operations is clearer than leaving them implicitly disabled.

</details>

<details>
<summary>75. What is a trivial class?</summary>

*(See Q50.)* A class where all special member functions are trivial (compiler-generated, do nothing meaningful). Trivial classes can be copied with `memcpy` and are compatible with C-style interfaces.

</details>

<details>
<summary>76. Advantages of having a default constructor?</summary>

- Allows creation of objects without arguments.
- Required for use in standard containers like `std::vector` (for resize) and `std::array`.
- Enables default member initialization patterns.
- Necessary for certain template metaprogramming and library requirements.

</details>

---

## Object-oriented design

<details>
<summary>77. Differences between a class and a struct?</summary>

In C++, the only difference is **default access**:
- `struct`: members and inheritance are `public` by default.
- `class`: members and inheritance are `private` by default.

Convention: use `struct` for passive data holders; `class` for types with invariants and behavior.

</details>

<details>
<summary>78. What is constructor delegation?</summary>

A constructor calling another constructor of the same class (C++11). Avoids duplicating initialization logic.

```cpp
struct Foo {
    Foo(int x, int y) : x_(x), y_(y) {}
    Foo(int x) : Foo(x, 0) {}  // delegates to Foo(int, int)
    int x_, y_;
};
```

</details>

<details>
<summary>79. Explain covariant return types</summary>

*(See Q16.)* An override may return a pointer/reference to a more-derived type than the base virtual function. Useful for clone/factory patterns where callers can avoid casts.

</details>

<details>
<summary>80. Difference between overloading and overriding?</summary>

*(See Q13.)* Overloading: same name, different parameters, same scope, resolved at compile time. Overriding: same name and signature, derived class, resolved at runtime via vtable.

</details>

<details>
<summary>81. What is the `override` keyword?</summary>

*(See Q15.)* Compiler-checked annotation that a virtual function is intentionally overriding a base class function. Makes errors visible at compile time.

</details>

<details>
<summary>82. Explain friend classes or functions</summary>

A `friend` declaration grants a non-member function or another class access to `private` and `protected` members. Friendship is not inherited or transitive.

```cpp
class Box {
    int width;
    friend void printWidth(const Box& b);  // can access width
};
void printWidth(const Box& b) { std::cout << b.width; }
```

</details>

<details>
<summary>83. What are default arguments?</summary>

Values specified in a function declaration that are used when the caller omits the corresponding argument. They must be at the trailing end of the parameter list and are evaluated at the call site.

```cpp
void log(std::string msg, int level = 0, bool timestamp = true);
log("hello");     // level=0, timestamp=true
log("hello", 2);  // level=2, timestamp=true
```

</details>

<details>
<summary>84. What is `this` pointer and can we delete it?</summary>

`this` is an implicit pointer to the current object inside non-static member functions. Technically you can call `delete this` if the object was heap-allocated and you ensure no further use, but it is extremely dangerous and almost never correct.

</details>

<details>
<summary>85. What is virtual inheritance?</summary>

*(See Q17.)* Ensures a single shared instance of a base class in diamond inheritance hierarchies.

</details>

<details>
<summary>86. Should we always use virtual inheritance?</summary>

*(See Q18.)* No — only when diamond inheritance is intentional and you need a single shared base. It adds overhead and complexity.

</details>

<details>
<summary>87. What does a strong type mean?</summary>

*(See Q25.)* A wrapper type that creates a distinct type from a primitive, preventing accidental interchange of semantically different values with the same representation.

</details>

<details>
<summary>88. What are user-defined literals?</summary>

*(See Q44.)* Suffixes on literals calling `operator""` to produce typed values. Improves readability and type safety for units, strings, durations, etc.

</details>

<details>
<summary>89. Why shouldn't we use boolean arguments?</summary>

Boolean parameters make call sites unreadable — `process(true, false)` tells the reader nothing. Prefer:
- Named enums or strong types.
- Separate functions with descriptive names.
- Named parameters via structs.

```cpp
// Bad:
render(true, false, true);

// Good:
render({.antialiasing = true, .wireframe = false, .shadows = true});
```

</details>

<details>
<summary>90. Distinguish between shallow and deep copy</summary>

- **Shallow copy**: copies the pointer/handle value — both original and copy point to the same underlying data. Modifications through one affect the other.
- **Deep copy**: duplicates the underlying data — each object owns its own independent copy.

The compiler-generated copy constructor does a memberwise shallow copy. If a class owns resources, a deep copy must be implemented manually (rule of three/five).

</details>

<details>
<summary>91. Are class functions part of object size?</summary>

No. Member functions (including virtual ones) are not stored per object. `sizeof(MyClass)` reflects only data members plus the vpointer (one pointer) if there are virtual functions. All instances share the same compiled function code.

</details>

<details>
<summary>92. What does dynamic dispatch mean?</summary>

Selecting which function implementation to call at **runtime** based on the actual type of the object, rather than its static (compile-time) type. Implemented via the vtable in C++. Only applies to `virtual` functions called through pointers or references.

</details>

<details>
<summary>93. What are vtable and vpointer?</summary>

- **vtable** (virtual table): a per-class array of function pointers to the most-derived virtual function implementations.
- **vpointer** (vptr): a hidden pointer added to each object with virtual functions, pointing to that class's vtable. The compiler uses it to resolve virtual calls at runtime.

</details>

<details>
<summary>94. Should base class destructors be virtual?</summary>

Yes, if objects are deleted through base class pointers. Without a virtual destructor, `delete basePtr` calls only the base destructor — derived resources are leaked and behavior is undefined.

Exception: non-polymorphic base classes (no virtual functions, not intended to be deleted via base pointer) — e.g., policy/mixin classes.

</details>

<details>
<summary>95. What is an abstract class?</summary>

A class with at least one **pure virtual function** (`= 0`). It cannot be instantiated directly and serves as an interface or base that derived classes must implement.

```cpp
struct Shape {
    virtual double area() const = 0;  // pure virtual
    virtual ~Shape() = default;
};
```

</details>

<details>
<summary>96. Is polymorphism possible without virtual functions?</summary>

Yes:
- **Static polymorphism** via templates (compile-time dispatch).
- **CRTP** (Curiously Recurring Template Pattern) for zero-overhead static dispatch.
- **`std::variant` + `std::visit`** for type-safe sum types.
- **Function overloading** and template specialization.

</details>

<details>
<summary>97. How to use the Curiously Recurring Template Pattern (CRTP)?</summary>

A derived class passes itself as a template argument to the base class. Enables the base to call derived methods without virtual dispatch — zero-overhead static polymorphism.

```cpp
template<typename Derived>
struct Base {
    void interface() {
        static_cast<Derived*>(this)->implementation();
    }
};
struct Derived : Base<Derived> {
    void implementation() { std::cout << "Derived\n"; }
};
```

</details>

<details>
<summary>98. Good reasons to use init() functions?</summary>

Generally avoid them — prefer constructors. But `init()` can be justified when:
- Initialization can fail and you can't use exceptions.
- Two-phase construction is required (e.g., circular dependencies).
- The object is created before the context needed to initialize it is available.

Risk: objects can exist in an uninitialized (invalid) state between construction and `init()`.

</details>

---

## Observable behaviors

<details>
<summary>99. What is observable behavior of code?</summary>

The C++ standard defines observable behavior as: reads/writes to `volatile` objects, I/O operations, and file operations. The compiler may reorder or eliminate any computation as long as observable behavior is preserved (the "as-if" rule).

</details>

<details>
<summary>100. Characteristics of an ill-formed C++ program?</summary>

An ill-formed program violates C++ syntax or semantic rules. The compiler **must** diagnose it (error or warning). Examples: missing semicolons, calling undefined functions, violating access specifiers. The program should not be expected to compile or run.

</details>

<details>
<summary>101. What is unspecified behavior?</summary>

Behavior where the standard allows multiple valid outcomes but does not require the implementation to document which one it chooses. The program is still valid, but results are unpredictable across implementations.

Example: order of evaluation of function arguments.
```cpp
foo(bar(), baz()); // unspecified: bar() or baz() may be called first
```

</details>

<details>
<summary>102. What is implementation-defined behavior?</summary>

Behavior that varies between implementations but each implementation must **document** its choice. The program is well-formed, but results are platform/compiler specific.

Examples: size of `int`, result of right-shifting a negative signed integer on a given platform.

</details>

<details>
<summary>103. What is undefined behavior?</summary>

Behavior for which the standard imposes **no requirements**. The compiler may assume it never happens — enabling optimizations that can produce surprising results. Common examples: signed integer overflow, null pointer dereference, out-of-bounds array access, data races.

</details>

<details>
<summary>104. Reasons behind undefined behavior's existence?</summary>

- Enables aggressive compiler optimizations (assume no UB → faster code).
- Allows portability across hardware with different semantics (e.g., integer overflow varies on different CPUs).
- Avoids mandating runtime checks that would cost performance.
- Reflects the historical C heritage where low-level behavior was left to the platform.

</details>

<details>
<summary>105. Approaches to avoid undefined behavior?</summary>

- Use sanitizers: AddressSanitizer (ASan), UBSan, ThreadSanitizer.
- Enable compiler warnings (`-Wall -Wextra -Wpedantic`).
- Use `std::array` with `.at()` instead of raw arrays.
- Avoid manual memory management — use RAII and smart pointers.
- Use `std::optional`, `std::variant` to avoid invalid states.
- Static analysis tools (clang-tidy, cppcheck, Coverity).

</details>

<details>
<summary>106. What is iterator invalidation?</summary>

Certain container operations invalidate existing iterators, pointers, or references — using them afterward is undefined behavior.

```cpp
std::vector<int> v = {1, 2, 3};
auto it = v.begin();
v.push_back(4);  // may reallocate — it is now invalid
*it;             // undefined behavior
```

Each container has specific invalidation rules (e.g., `vector::push_back` may invalidate all iterators; `list::insert` does not).

</details>

---

## Standard Template Library

<details>
<summary>107. What is the STL?</summary>

The Standard Template Library is part of the C++ standard library providing:
- **Containers**: `vector`, `list`, `map`, `set`, `unordered_map`, etc.
- **Algorithms**: `sort`, `find`, `transform`, `accumulate`, etc.
- **Iterators**: abstractions connecting containers and algorithms.
- **Functors and utilities**: `std::function`, `std::pair`, `std::tuple`, etc.

</details>

<details>
<summary>108. Advantages of algorithms over raw loops?</summary>

- Express **intent** clearly (`std::sort` vs a manual sort loop).
- Less error-prone — boundary conditions handled internally.
- Potentially optimized (e.g., `std::sort` uses introsort).
- Composable and reusable.
- Easier to parallelize (C++17 parallel execution policies).

```cpp
// raw loop
for (int i = 0; i < n-1; ++i) for (int j = ...) { ... }

// algorithm
std::sort(v.begin(), v.end());
```

</details>

<details>
<summary>109. Do algorithms validate ranges?</summary>

No. Standard library algorithms do not validate that iterators form a valid range or that `first <= last`. Passing invalid ranges (reversed iterators, past-end iterators) is undefined behavior. Use sanitizers or safe wrappers if validation is needed.

</details>

<details>
<summary>110. Can you combine containers of different sizes?</summary>

Algorithms operating on two ranges (e.g., `std::transform`, `std::copy`) take a start and end for the first range, but only a start for the second — no size check is performed. If the second range is shorter, it is undefined behavior. You must ensure sizes are compatible.

</details>

<details>
<summary>111. How is a `vector`'s memory layout organized?</summary>

`std::vector` stores elements in a **contiguous block** of heap memory. It maintains:
- `size`: number of elements currently stored.
- `capacity`: total allocated space (may be larger than size).

When `size == capacity` and a new element is pushed, it reallocates (typically doubling capacity), copying/moving all elements. This gives amortized O(1) `push_back`.

</details>

<details>
<summary>112. Can we inherit from standard containers?</summary>

*(See Q24.)* Technically yes, but strongly discouraged — standard containers have no virtual destructors. Deleting via a base pointer is UB. Prefer composition.

</details>

<details>
<summary>113. What is the type of myCollection after declaration?</summary>

```cpp
std::map<std::string, std::vector<int>> myCollection;
auto myCollection2 = myCollection;
```

`myCollection2` is `std::map<std::string, std::vector<int>>` — deduced by `auto` from the right-hand side.

</details>

<details>
<summary>114. Advantages of `const_iterator`s over iterators?</summary>

- Signals read-only intent — the element cannot be modified through the iterator.
- Works on `const` containers (where only `const_iterator` is available).
- Communicates to readers that you are only inspecting, not mutating.

```cpp
const std::vector<int> v = {1, 2, 3};
auto it = v.cbegin(); // const_iterator
```

</details>

<details>
<summary>115. Binary search an element with algorithms</summary>

```cpp
std::vector<int> v = {1, 3, 5, 7, 9}; // must be sorted

// Check existence:
bool found = std::binary_search(v.begin(), v.end(), 5);

// Find position:
auto it = std::lower_bound(v.begin(), v.end(), 5);
if (it != v.end() && *it == 5) { /* found */ }
```

`std::binary_search` requires a sorted range and runs in O(log n).

</details>

<details>
<summary>116. What is an Iterator class?</summary>

An iterator is an object that abstracts traversal over a sequence. It provides at minimum:
- `operator*` — dereference (access element).
- `operator++` — advance.
- `operator!=` / `operator==` — comparison.

Iterator categories (from weakest to strongest): Input, Output, Forward, Bidirectional, Random Access, Contiguous. Algorithms require specific categories.

</details>

---

## Miscellaneous

<details>
<summary>117. Can you call a `virtual` function from constructor/destructor?</summary>

*(See Q22.)* Yes, but virtual dispatch does not work — the base class version is called because the derived part is not yet constructed (or already destroyed).

</details>

<details>
<summary>118. What are default arguments?</summary>

*(See Q83.)* Values provided in the function declaration used when the caller omits trailing arguments. Evaluated at the call site each time.

</details>

<details>
<summary>119. Can virtual functions have default arguments?</summary>

Technically yes, but it is a bad idea. Default arguments are resolved based on the **static type** of the pointer/reference, not the dynamic type. This means the base's default is used even when the derived override is called.

```cpp
struct Base   { virtual void foo(int x = 1); };
struct Derived : Base { void foo(int x = 2) override; };
Base* p = new Derived;
p->foo(); // calls Derived::foo but x = 1 (Base's default)!
```

</details>

<details>
<summary>120. Should base class destructors be virtual?</summary>

*(See Q94.)* Yes for polymorphic base classes. No for non-polymorphic bases not intended for deletion via base pointer.

</details>

<details>
<summary>121. Function of the `mutable` keyword?</summary>

`mutable` allows a member variable to be modified even in a `const` member function. Used for caching/lazy evaluation where logical constness is preserved but internal state changes.

```cpp
struct Cache {
    mutable int cachedValue = -1;
    int compute() const {
        if (cachedValue == -1) cachedValue = expensiveOp();
        return cachedValue;
    }
};
```

</details>

<details>
<summary>122. Function of the `volatile` keyword?</summary>

`volatile` tells the compiler that a variable may change outside the program's control (hardware register, signal handler, another thread at the hardware level). It prevents the compiler from caching the value in a register or reordering accesses to it.

Note: `volatile` does **not** provide thread safety or memory ordering guarantees — use `std::atomic` for that.

</details>

<details>
<summary>123. What is an inline function?</summary>

`inline` suggests to the compiler to replace the function call with the function body. In practice, modern compilers inline regardless of the keyword. The real purpose today: `inline` allows a function to be **defined in multiple translation units** (e.g., in a header) without violating the One Definition Rule.

</details>

<details>
<summary>124. What do we catch?</summary>

In a `try/catch` block you catch **exceptions by type**. Best practices:
- Catch by `const` reference to avoid slicing and unnecessary copies.
- Catch specific types before general ones.
- `catch(...)` catches everything (use as last resort).

```cpp
try { riskyOp(); }
catch (const std::out_of_range& e) { /* specific */ }
catch (const std::exception& e)    { /* general  */ }
catch (...)                        { /* everything */ }
```

</details>

<details>
<summary>125. Differences between references and pointers?</summary>

| | Reference | Pointer |
|---|---|---|
| Null | Cannot be null | Can be null |
| Rebinding | Cannot be rebound after init | Can point to different objects |
| Syntax | `ref.member` | `ptr->member` |
| Arithmetic | Not allowed | Allowed |
| Initialization | Must be initialized | Not required (dangerous) |
| Indirection | Transparent | Explicit dereference |

</details>

<details>
<summary>126. Which variable declarations compile?</summary>

Common tricky cases:
```cpp
int& r;            // error: reference must be initialized
const int& r = 5;  // OK: const ref extends lifetime of temporary
int* p;            // OK (warning): uninitialized pointer — dangerous
int* const p2;     // error: const pointer must be initialized
```

</details>

<details>
<summary>127. What will the code print out and why?</summary>

*(Context-dependent — see book example.)* Common traps: integer promotion, implicit conversions, object slicing, operator precedence, or initialization order.

</details>

<details>
<summary>128. Difference between pre- and post-increment/decrement?</summary>

- **Pre-increment** (`++i`): increments and returns the new value. No temporary created.
- **Post-increment** (`i++`): saves the old value, increments, returns the old value. Creates a temporary copy.

For non-trivial types (iterators), prefer pre-increment — post-increment may be more expensive.

```cpp
int i = 5;
std::cout << ++i; // 6
std::cout << i++; // 6 (prints 6, then i becomes 7)
```

</details>

<details>
<summary>129. Final values of variables?</summary>

*(Context-dependent — see book example.)* Typically tests understanding of pre/post increment, operator precedence, or sequence points.

</details>

<details>
<summary>130. Does this string declaration compile?</summary>

*(Context-dependent — see book example.)* Common cases:
```cpp
std::string s1 = "hello";    // OK
std::string s2{"world"};     // OK
const char* s3 = "literal";  // OK
char s4[] = "array";         // OK
```

</details>

<details>
<summary>131. What are Default Member Initializers?</summary>

Values provided directly in the class definition for non-static data members (C++11). Used when no constructor explicitly initializes that member.

```cpp
struct Config {
    int timeout = 30;
    bool verbose = false;
    std::string name = "default";
};
Config c; // timeout=30, verbose=false, name="default"
```

</details>

<details>
<summary>132. What is the most vexing parse?</summary>

An ambiguity in C++ grammar where a declaration that looks like object creation is parsed as a function declaration.

```cpp
Widget w();   // most vexing parse: declares a function w() returning Widget
Widget w{};   // OK: value-initializes a Widget object
```

</details>

<details>
<summary>133. Does this code compile?</summary>

*(Context-dependent — see book example.)* Common gotchas: narrowing conversions with brace-init, accessing private members, missing includes, redeclaring in wrong scope.

</details>

<details>
<summary>134. What is `std::string_view`?</summary>

A non-owning, read-only reference to a character sequence (C++17). It stores a pointer and a length — no heap allocation. Useful as a function parameter to accept `std::string`, `const char*`, and string literals without copying.

```cpp
void log(std::string_view msg) { std::cout << msg; }
log("literal");         // no copy
log(std::string{"s"});  // no copy
```

Caution: the underlying data must outlive the `string_view`.

</details>

<details>
<summary>135. How to check if string starts or ends with substring?</summary>

C++20 added `starts_with` and `ends_with` to `std::string` and `std::string_view`:

```cpp
std::string s = "Hello, World";
s.starts_with("Hello");  // true
s.ends_with("World");    // true

// Pre-C++20:
s.rfind("Hello", 0) == 0;  // starts_with
s.size() >= 5 && s.substr(s.size() - 5) == "World";  // ends_with
```

</details>

<details>
<summary>136. What is RVO?</summary>

**Return Value Optimization** — the compiler constructs the return value directly in the caller's storage, eliminating the copy/move of the returned object. A form of copy elision guaranteed by the standard (C++17, mandatory for prvalues).

```cpp
std::string makeString() {
    return std::string("hello"); // constructed directly in caller's location
}
```

</details>

<details>
<summary>137. How to ensure compiler performs RVO?</summary>

- Return a single, local named variable (NRVO — Named RVO, not guaranteed but common).
- Return a temporary/prvalue directly (guaranteed copy elision in C++17).
- Avoid `std::move` on the return expression — it inhibits NRVO by converting to an rvalue reference.

```cpp
std::string good() {
    std::string s = "hello";
    return s;             // NRVO likely applied
}
std::string bad() {
    std::string s = "hello";
    return std::move(s);  // inhibits NRVO; forces move instead
}
```

</details>

<details>
<summary>138. Primary and mixed value categories in C++?</summary>

Primary categories:
- **lvalue**: has identity, can take address (`x`, `obj.member`).
- **prvalue**: no identity, temporary result of an expression (`42`, `a + b`, `T()`).
- **xvalue**: expiring value — has identity but resources can be moved from (`std::move(x)`).

Mixed categories:
- **glvalue** = lvalue + xvalue (has identity).
- **rvalue** = prvalue + xvalue (can be moved from).

</details>

<details>
<summary>139. Can you safely compare signed and unsigned integers?</summary>

Not directly. When a signed and unsigned integer are compared, the signed value is implicitly converted to unsigned — negative values wrap around to large positive numbers.

```cpp
int s = -1;
unsigned u = 1;
s < u;  // false! -1 converted to a huge unsigned value
```

Use explicit casts or `std::cmp_less` (C++20) for safe comparison.

</details>

<details>
<summary>140. Return value of main and available signatures?</summary>

`main` must return `int`. Two standard signatures:

```cpp
int main();
int main(int argc, char* argv[]);
// Some implementations also accept:
int main(int argc, char* argv[], char* envp[]); // platform-specific
```

Returning `0` or `EXIT_SUCCESS` signals success; `EXIT_FAILURE` signals failure. Falling off the end of `main` implicitly returns `0`.

</details>

<details>
<summary>141. Prefer default arguments or overloading?</summary>

- **Default arguments**: simpler when the extra parameters are truly optional and have sensible defaults. Avoids code duplication.
- **Overloading**: better when different argument sets need different implementations, or when the set of parameters is genuinely different.

Avoid default arguments on virtual functions (see Q119 — defaults are resolved statically).

</details>

<details>
<summary>142. How many variables should you declare on a line?</summary>

One variable per line. Multiple declarations on one line reduce readability and can cause confusion with pointer/reference syntax:

```cpp
int* a, b;  // a is int*, b is int — confusing
int* a;
int  b;     // clear
```

</details>

<details>
<summary>143. Prefer switch statement or chained if?</summary>

Prefer `switch` when branching on a single integral or enum value with multiple cases — it is more readable and compilers can optimize it with jump tables. Use chained `if`/`else if` for ranges, multiple conditions per branch, or non-integral types.

```cpp
switch (status) {    // clear and fast
    case OK:  break;
    case ERR: break;
}
```

</details>

<details>
<summary>144. What are include guards?</summary>

Preprocessor directives that prevent a header from being included more than once in a single translation unit, avoiding redefinition errors.

```cpp
#ifndef MY_HEADER_H
#define MY_HEADER_H
// ... header content ...
#endif
```

Modern alternative: `#pragma once` (not standard, but universally supported and simpler).

</details>

<details>
<summary>145. Use angle brackets or double quotes for includes?</summary>

- `#include <header>`: for **system and standard library headers**. The preprocessor searches standard/system include paths.
- `#include "header.h"`: for **project-local headers**. The preprocessor searches relative to the current file first, then system paths.

</details>

<details>
<summary>146. How many return statements should a function have?</summary>

No strict rule, but **early returns** (guard clauses) reduce nesting and can make code much cleaner:

```cpp
Result process(Input in) {
    if (!in.valid()) return Error;
    if (in.empty()) return Empty;
    return compute(in);
}
```

Single exit point is useful when cleanup before returning is needed, but modern RAII usually eliminates that concern.

</details>

---

## C++ and algorithmic complexities

<details>
<summary>147. Differences between `std::map` and `std::unordered_map`?</summary>

| | `std::map` | `std::unordered_map` |
|---|---|---|
| Implementation | Red-black tree | Hash table |
| Key order | Sorted | Unordered |
| Lookup | O(log n) | O(1) average, O(n) worst |
| Insert | O(log n) | O(1) average |
| Key requirement | `operator<` | `std::hash` + `operator==` |
| Iterator stability | Stable on insert/erase | May invalidate on rehash |
| Memory | Higher (tree nodes) | Higher (hash buckets) |

Use `map` when you need sorted iteration or range queries. Use `unordered_map` for faster average-case lookups.

</details>

<details>
<summary>148. When to use a list over a vector?</summary>

Prefer `std::list` over `std::vector` only when:
- Frequent **insertion/deletion in the middle** at known iterator positions is needed (O(1) for list, O(n) for vector).
- Iterator/pointer stability is required (list iterators are never invalidated by insert/erase elsewhere).

In practice, `std::vector` wins for most use cases due to cache locality. Even sequential insertion is often faster with a vector due to CPU prefetching.

</details>

<details>
<summary>149. Algorithmic complexities of important algorithms?</summary>

| Algorithm | Complexity | Notes |
|---|---|---|
| `std::sort` | O(n log n) | Introsort (quicksort + heapsort + insertion) |
| `std::stable_sort` | O(n log² n) | Preserves relative order |
| `std::find` | O(n) | Linear search |
| `std::binary_search` | O(log n) | Requires sorted range |
| `std::lower_bound` | O(log n) | Requires sorted range |
| `std::nth_element` | O(n) | Partial sort |
| `std::partition` | O(n) | |
| `std::merge` | O(n + m) | |
| `std::min_element` | O(n) | |
| `std::count` | O(n) | |
| `std::accumulate` | O(n) | |

</details>

---

# Reference

Below are the reference links to where questions are consolidated.

- [Daily C++ Interview Questions](https://leanpub.com/cppinterview)
