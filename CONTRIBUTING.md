# Contributing Guidelines

## Common
### Commit Style
We generally follow simple plain-English commit titles and summaries, which encapsulate what a commit has done. We don't use a distinct commit style like that of the Linux Kernel for any commits and they would look extremely out of place in the repository overall.

In addition, we stick to a single objective with one commit albeit ensure that the scope isn't too small so there'll be a huge amount of them or too large so it's a single commit that changes vast swaths of the codebase. Try to find the right balance between committing too less and too much.

### Use line-wrapping
There is no column limit in the codebase, this is so that the line width can adjust to everyone's display size using line-wrap. Do not manually wrap lines unless it can be done in a natural way and is needed at all, let line-wrap handle it for you.

### Use code formatter
Android Studio comes with a code formatter in-built, this can fix minor mistakes in code-style. To reformat code: Right-click on the relevant file/directory -> Reformat. We recommend doing so prior to all commits to ensure the codebase is clean.

This can also be done by using `Ctrl + Alt + L` on Windows, `Ctrl + Shift + Alt + L` on Linux and `Option + Command + L` on macOS. 

## C++
### Include Order
* STD includes
* External library includes
* Includes in a parent directory with `<>` (where otherwise you would need `../`)
* Local includes with `""`

### Naming rules
* Macro: `SCREAMING_SNAKE_CASE`
* Namespaces: `camelCase`
* Enum: `PascalCase`
* Enumerator: `PascalCase` **(1)**
* Union: `PascalCase`
* Classes/Structs: `PascalCase` **(1)**
* Local Variable: `camelCase`
* Member Variables: `camelCase`
* Global Variables: `PascalCase`
* Functions: `PascalCase`
* Parameters: `camelCase`
* Files and Directories: `snake_case` except for when they correspond to a HOS structure (EG: Services, Kernel Objects)

**(1)** Except when the whole name is an abbreviation such as `OS` and `NCE` but not `JVMManager`

### Comments
Use doxygen style comments for:
* Classes/Structs - Use `/**` block comments with a brief
* Class/Struct Variables - Use `//!<` single-line comments on their utility 
* Class/Struct Functions - Use `/**` block comments on their function with a brief, all arguments and the return value (The brief can be skipped if the function's arguments and return value alone explain what the function does)
* Enumerations - Use a `/**` block comment with a brief for the enum itself and a `//!<` single-line comment for all the individual items

Notes: 
* The DeviceState object can be skipped from function argument documentation as well as class members in the constructor
* Any class members don't need to be redundantly documented in the constructor

### Spacing
We generally follow the rule of **"Functional Spacing"**, that being spacing between chunks of code that do something functionally different while functionally similar blocks of code can be closer together.

Here are a few examples of this to help with intution:
* Spacing should generally follow multiple related variable declarations, this applies even more if error checking code needs to follow it
  ```cpp
  auto a = GetSomethingA();
  auto b = GetSomethingB();
  auto c = GetSomethingC();

  auto result = DoSomething(a, b, c);
  if (!result)
    throw exception("DoSomething has failed: {}", result);
  ```
* If a function doesn't require multiple variable declarations, the function call should be right after the variable declaration
  ```cpp
  auto a = GetClassA();
  a.DoSomething();

  auto b = GetClassB();
  b.DoSomething();
  ```
* If a single variable is used by a single-line control flow statement, there can be no spaces after it's declaration 
  ```cpp
  auto a = GetClass();
  if (a.fail)
    throw exception();
  ```
* **Be consistent** (The above examples assume that `a`/`b`/`c` are correlated)
  * Inconsistent
    ```cpp
    auto a = GetClassA();
    a.DoSomething();

    auto b = GetClassB();
    auto c = GetClassC();

    b.DoSomething();
    c.DoSomething();
    ```
  * Consistent #1
    ```cpp
    auto a = GetClassA();
    auto b = GetClassB();
    auto c = GetClassC();

    a.DoSomething();
    b.DoSomething();
    c.DoSomething();
    ```
  * Consistent #2
    ```cpp
    auto a = GetClassA();
    a.DoSomething();

    auto b = GetClassB();
    b.DoSomething();

    auto c = GetClassC();
    c.DoSomething();
    ```

### Control flow statements (if, for and while)
#### If a child control-flow statement has brackets, the parent statement must as well
* Correct
  ```cpp
  if (a) {
    while (b) {
      printf("A");
      printf("B");

      b = false;
    }
  }
  ```
* Incorrect
  ```cpp
  if (a)
    while (b) {
      printf("A");
      printf("B");

      b = false;
    }
  ```

#### If any cases of an if statement have curly brackets all must have curly brackets
* Correct
  ```cpp
  if (a) {
    printf("a");
    
    a = false;
  } else {
    printf("none");
  }
  ```
* Incorrect
  ```cpp
  if (a) {
    printf("a");

    a = false;
  } else
    printf("none");
  ```

### Lambda Usage
We generally support the usage of functional programming and lambda, usage of it for assigning conditional variables is recommended especially if it would otherwise be a nested ternary statement:
* With Lambda (Inlined function)
  ```cpp
  auto a = random();
  auto b = [a] {
    if (a > 1000)
      return 0;
    else if (a > 500)
      return 1;
    else if (a > 250)
      return 2;
    else
      return 3;
  }();
  ```
* With Ternary Operator
  ```cpp
  auto a = random();
  auto b = (a > 1000) ? 0 : ((a > 500) ? 1 : (a > 250 ? 2 : 3)); 
  ```

### References
For passing any parameter which isn't a primitive prefer to use references/const references to pass them into functions or other places as a copy can be avoided that way.  
In addition, always use a const reference rather than a normal reference unless the argument needs to be modified in-place as the compiler knows the intent far better in that case.
```cpp
void DoSomething(const Class& class, u32 primitive);
void ClassConstructor(const Class& class) : class(class) {} // Make a copy directly from a `const reference` for class member initialization
```

### Range-based Iterators
Use C++ range-based iterators for any C++ container iteration unless it can be performed better with functional programming (After C++20 when they are merged with the container). In addition, stick to using references/const references using them

### Usage of auto
Use `auto` unless a variable needs to be a specific type that isn't automatically deducible.
```cpp
u16 a = 20; // Only if `a` is required to specifically be 16-bit, otherwise integers should be auto
Handle b = 0x10; // Handle is a specific type that won't be automatically assigned
```

### Constants
If a variable is constant at compile time use `constexpr`, if it's only used in a local function then place it in the function but if it's used throughout a class then in the corresponding header add the variable to the `skyline::constant` namespace. If a constant is used throughout the codebase, add it to `common.h`.

In addition, try to `constexpr` as much as possible including constructors and functions so that they may be initialized at compile-time and have lesser runtime overhead during usage and certain values can be pre-calculated in advance.

## Kotlin
### Naming rules
* Enumerator: `PascalCase` **(1)**
* Classes: `PascalCase` **(1)**
* Local Variable: `camelCase` 
* Member Variables: `camelCase`
* Global Variables: `PascalCase`
* Functions: `PascalCase`
* Parameters: `camelCase`
* Files: `PascalCase`
* Directories: `camelCase`

**(1)** Except when the whole name is an abbreviation such as `OS` and `NCE` but not `JVMManager`

### Comments
Use KDoc comments (`/**`) for:
* Classes - A comment about the function of a class and any of it's parameters if required, albeit this can be skipped if the information is mundane enough to the point of it being useless which is generally the case with [Activities](https://developer.android.com/reference/android/app/Activity), albeit this isn't always the case. In addition, document any parameters in the primary constructor using `@param`.
* Variables/Functions - A comment about what the variable is used for or what it is depending on what's more applicable. This is mandatory and shouldn't be skipped.
* Functions - A comment about what the function is used for or what it is depending on what's more applicable and is mandatory, just like a variable. All parameters should be documented using `@param`. In addition, if it's an overridden function then it's description should cover more what it specifically does rather than what it is.  
* Constructors - A comment about the constructor can be skipped if it simply calls another constructor or just assigns variables, assuming those variables are documented themselves. If the constructor does anything outside of that, it must be documented in accordance to the function comments.

Exceptions to `@param`:
* If a parameter has already been tagged in the comment using a KDoc link (`[]`), then it doesn't need to have it's own `@param`
* If a parameter is a standard Android class such as `Context` then it's documentation can be skipped entirely

Notes:
* Any class members don't need to be redundantly documented in the constructor
* Use a KDoc link (`[]`) when referencing any item 

### Spacing
The spacing in Kotlin follows the same rule as C++ with "**Functional Spacing**", spacing between chunks of code that do something functionally different while functionally similar blocks of code can be closer together.

Here are a few examples of this to help with intution:
* Spacing should generally follow multiple related variable declarations
  ```kotlin
  val a = GetSomethingA()
  val b = GetSomethingB()
  val c = GetSomethingC()

  val result = DoSomething(a, b, c)
  ```
* If a function doesn't require multiple variable declarations, the function call should be right after the variable declaration
  ```kotlin
  val a = GetClassA()
  a.DoSomething()

  val b = GetClassB()
  b.DoSomething()
  ```
  ```kotlin
  val a = GetClassA()
  DoSomething(a)

  val b = GetClassB()
  DoSomething(b)
  ```
* If a single variable is used by a single-line control flow statement, there can be no spaces after it's declaration 
  ```kotlin
  val a = GetClass()
  if (a.fail)
    throw exception()
  ```
* In `when` statements, if cases generally have multi-line code blocks then have spacing between all the cases
  ```kotlin
  when(num) {
    1 -> {
      a.Something()
      a.SomethingElse()
    }
    
    2 -> {
      b.Something()
      b.SomethingElse()
    }

    else -> c.Something()
  }
  ```
  ```kotlin
  when(num) {
    1 -> a.Something()
    2 -> b.Something()
    else -> c.Something()
  }
  ```
* **Be consistent** (The above examples assume that `a`/`b`/`c` are correlated)
  * Inconsistent
    ```kotlin
    val a = GetClassA()
    a.DoSomething()

    val b = GetClassB()
    val c = GetClassC()

    b.DoSomething()
    c.DoSomething()
    ```
  * Consistent #1
    ```kotlin
    val a = GetClassA()
    val b = GetClassB()
    val c = GetClassC()

    a.DoSomething()
    b.DoSomething()
    c.DoSomething()
    ```
  * Consistent #2
    ```kotlin
    val a = GetClassA()
    a.DoSomething()

    val b = GetClassB()
    b.DoSomething()

    val c = GetClassC()
    c.DoSomething()
    ```

### Control flow statements (if, for and while)
#### If a child control-flow statement has brackets, the parent statement must as well
* Correct
  ```cpp
  if (a) {
    while (b) {
      Something(a)
      Something(b)
    }
  }
  ```
* Incorrect
  ```cpp
  if (a)
    while (b) {
      Something(a)
      Something(b)
    }
  ```

#### If any cases of an if statement have curly brackets all must have curly brackets
* Correct
  ```cpp
  if (a) {
    Something()
    SomethingElse()
  } else {
    SomethingElse()
  }
  ```
* Incorrect
  ```cpp
  if (a) {
    Something()
    SomethingElse()
  } else
    SomethingElse()
  ```