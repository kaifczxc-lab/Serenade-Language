# The Serenade Programming Language Specification

Version 0.3-Alpha | 17th February 2026

---

## Table of Contents

1. [Introduction](#introduction)
2. [Notation](#notation)
3. [Source Code Representation](#source-code-representation)
4. [Lexical Elements](#lexical-elements)
5. [Constants](#constants)
6. [Variables](#variables)
7. [Types and Values](#types-and-values)
8. [Expressions](#expressions)
9. [Statements](#statements)
10. [Declarations](#declarations)
11. [Built-in Functions](#built-in-functions)
12. [Memory Management](#memory-management)
13. [Concurrency](#concurrency)
14. [Assembly Integration](#assembly-integration)
15. [Native Code Injection](#native-code-injection)
16. [Visual System](#visual-system)
17. [Engine Integration](#engine-integration)
18. [Build System](#build-system)
19. [Runtime Diagnostics](#runtime-diagnostics)
20. [Standard Library](#standard-library)
21. [Appendix](#appendix)

---

## Introduction

Serenade is a multi-paradigm, polyglot-first programming language designed for systems programming with an emphasis on performance, simplicity, and cross-language interoperability. The language compiles to a combination of C++, Go, and x86-64 assembly, allowing developers to write high-level logic while maintaining low-level control when needed.

### Design Philosophy

Serenade follows a core principle: **"If it can't be explained in five minutes, it shouldn't exist in the language."** Every language feature is designed to be immediately understandable, with clear semantics and predictable behavior.

The language prioritizes:

- **Explainability**: Small, composable features with obvious behavior
- **Performance**: Direct compilation to optimized native code
- **Interoperability**: Seamless integration with C++, Go, and assembly
- **Minimal Syntax**: No semicolons, minimal punctuation, natural flow

### Use Cases

Serenade excels at:

- High-performance computing and numerical workloads
- Systems programming with mixed-language requirements
- AI/ML inference engines and data processing pipelines
- Low-level optimization with high-level expressiveness
- Rapid prototyping of performance-critical code

### Implementation

A Serenade program is transpiled into:

- **C++** for core runtime, memory management, and fast computation
- **Go** for concurrent services, I/O, and tooling
- **x86-64 Assembly** for critical hot paths and SIMD operations

The transpiler analyzes your Serenade code and generates optimized output in each target language, which is then compiled by standard toolchains (MSVC/GCC/Clang for C++, Go compiler for Go, NASM for assembly).

---

## Notation

The syntax is specified using Extended Backus-Naur Form (EBNF):

```
Production  = production_name "=" [ Expression ] "." .
Expression  = Term { "|" Term } .
Term        = Factor { Factor } .
Factor      = production_name | token [ "…" token ] | Group | Option | Repetition .
Group       = "(" Expression ")" .
Option      = "[" Expression "]" .
Repetition  = "{" Expression "}" .
```

Operators in increasing precedence:

```
|   alternation
()  grouping
[]  option (0 or 1 times)
{}  repetition (0 to n times)
```

Lexical tokens are enclosed in double quotes `""` or back quotes `` ` ``. Production names in `CamelCase` are non-terminals; lowercase names are terminals.

---

## Source Code Representation

### Source Files

Serenade source code is UTF-8 encoded Unicode text. Source files use the extensions:

- `.serenade` - standard extension
- `.srnDE` - alternative extension

### Character Encoding

Source text is not canonicalized. Each Unicode code point is distinct; uppercase and lowercase letters are different characters.

Implementation restriction: The NUL character (U+0000) is disallowed in source text.

### Line Structure

Serenade is line-based. Statements are separated by newlines, not semicolons. A statement may span multiple lines if enclosed in braces `{}` or if the line ends with an operator.

Example:

```serenade
let x = 10
let y = x +
        20
let z = {
  compute(x, y)
}
```

---

## Lexical Elements

### Comments

Comments serve as program documentation. Serenade supports line comments only:

```serenade
# This is a comment
let hp = 100  # inline comment
```

Comments start with `#` and continue to the end of the line. There are no block comments.

### Tokens

Tokens form the vocabulary of the language. There are four classes:

1. **Identifiers** - names for variables, functions, types
2. **Keywords** - reserved language constructs
3. **Operators** - symbols for operations and punctuation
4. **Literals** - constant values (numbers, strings)

White space (spaces, tabs, newlines) is ignored except as it separates tokens.

### Identifiers

Identifiers name program entities such as variables and functions.

```
identifier = letter { letter | unicode_digit } .
letter     = unicode_letter | "_" .
```

Examples:

```serenade
player
_internal
hp2
MaxHealth
```

The underscore `_` is considered a letter. Identifiers are case-sensitive: `Player` and `player` are different identifiers.

### Keywords

The following keywords are reserved and cannot be used as identifiers:

```
let         cycle       registry    emit        task
scan        if          return      break       spawn
await       native      atomic      move        using
as          endnative
```

### Operators and Punctuation

```
+     -     *     /     %     =     ==    !=
<     >     <=    >=    &&    ||    !     (
)     {     }     [     ]     :     ,     .
..    >>    ?.    ^     @     #
```

Special operator sequences:

- `..` - range operator (inclusive)
- `>>` - pipe operator
- `?.` - safe access operator
- `^[` - pointer indexing
- `@f32[` - arena allocation

### Integer Literals

Integer literals represent integer constants. Serenade supports decimal, binary, octal, and hexadecimal integers.

```
int_lit     = decimal_lit | binary_lit | octal_lit | hex_lit .
decimal_lit = "0" | ( "1" … "9" ) { decimal_digit } .
binary_lit  = "0" ( "b" | "B" ) binary_digit { binary_digit } .
octal_lit   = "0" ( "o" | "O" ) octal_digit { octal_digit } .
hex_lit     = "0" ( "x" | "X" ) hex_digit { hex_digit } .
```

Examples:

```serenade
42
0b101010
0o52
0x2A
0x2a
```

### Floating-Point Literals

Floating-point literals represent floating-point constants.

```
float_lit = decimal_digits "." [ decimal_digits ] [ exponent ] |
            decimal_digits exponent |
            "." decimal_digits [ exponent ] .
exponent  = ( "e" | "E" ) [ "+" | "-" ] decimal_digits .
```

Examples:

```serenade
3.14
.5
1e10
2.5e-3
```

### Size Suffix Literals

Serenade provides size suffixes for memory-related constants:

```
size_lit = ( int_lit | float_lit ) ( "kb" | "KB" | "mb" | "MB" | "gb" | "GB" ) .
```

Examples:

```serenade
64kb      # 65536
128mb     # 134217728
2gb       # 2147483648
```

These are expanded at compile time:

- `kb` / `KB` → `* 1024`
- `mb` / `MB` → `* 1024 * 1024`
- `gb` / `GB` → `* 1024 * 1024 * 1024`

### String Literals

String literals represent string constants. Serenade uses interpreted strings with double quotes and supports interpolation.

```
string_lit = `"` { string_char | interpolation } `"` .
string_char = unicode_char - `"` - `\` - `{` | escape_sequence .
interpolation = "{" expression "}" .
```

Escape sequences:

```
\\   backslash
\"   double quote
\n   newline
\t   tab
```

Examples:

```serenade
"hello"
"line 1\nline 2"
"value: {x}"
"hp: {player.hp}, mp: {player.mp}"
```

String interpolation is expanded at compile time to concatenation:

```serenade
"hp: {hp}"
# becomes
"hp: " + Sr_ToString(hp)
```

---

## Constants

There are boolean constants, numeric constants, and string constants.

### Boolean Constants

The predeclared boolean constants are `true` and `false`.

### Numeric Constants

Numeric constants represent exact values of arbitrary precision. Integer and floating-point literals are numeric constants.

### String Constants

String literals are string constants.

### Constant Expressions

Expressions involving only constants are evaluated at compile time.

```serenade
let size = 64 * 1024
let capacity = 128mb
```

---

## Variables

A variable is a storage location for holding a value. Variables are declared using the `let` keyword.

### Variable Declaration

```
let_decl = "let" identifier "=" expression .
```

Examples:

```serenade
let hp = 100
let name = "Player"
let position = compute_spawn()
```

Variables are automatically typed based on their initializer. Under the hood, Serenade generates C++ `auto` declarations:

```cpp
auto hp = 100;
auto name = std::string("Player");
auto position = compute_spawn();
```

### Variable Scope

Variables are scoped to the block in which they are declared. Blocks are delimited by `{}`.

```serenade
let x = 10
if condition {
  let y = 20
  emit "x={x}, y={y}"
}
# y is not accessible here
```

---

## Types and Values

Serenade uses a dynamic value model at the script level, with static typing in the generated code.

### Basic Types

At runtime, Serenade supports:

- **Numbers** - represented as `double` in C++
- **Strings** - represented as `std::string` in C++
- **Booleans** - `true` and `false`

### Value Type

The `SrValue` struct is used for dynamic values in registries:

```cpp
struct SrValue {
  enum Kind { Num, Str } kind;
  double num;
  std::string str;
  
  SrValue();
  SrValue(double v);
  SrValue(const std::string& s);
  std::string ToString() const;
  double AsDouble() const;
  long long AsInt() const;
};
```

### Type Inference

The Serenade compiler infers types from context:

```serenade
let x = 42          # int/double
let s = "text"      # string
let flag = true     # bool
let arr = @f32[100] # float*
```

### Type Conversions

Conversions happen automatically in expressions:

```serenade
let x = 10
let s = "value: {x}"  # x converted to string
```

---

## Expressions

Expressions specify the computation of values.

### Operands

Operands denote elementary values. An operand may be:

- A literal
- An identifier denoting a variable or function
- A function call
- An expression in parentheses

### Operators

#### Arithmetic Operators

```
+    addition
-    subtraction
*    multiplication
/    division
%    modulo
```

Example:

```serenade
let x = (a + b) * c
let y = total % count
```

#### Comparison Operators

```
==   equal
!=   not equal
<    less than
<=   less than or equal
>    greater than
>=   greater than or equal
```

Example:

```serenade
if hp < max_hp {
  emit "damaged"
}
```

#### Logical Operators

```
&&   logical AND
||   logical OR
!    logical NOT
```

Example:

```serenade
if alive && hp > 0 {
  emit "fighting"
}
```

### Function Calls

Functions are called using parentheses:

```serenade
result(arg1, arg2)
process()
```

### Member Access

Registry members are accessed with dot notation:

```serenade
let cfg = registry { workers: 4 }
emit "workers: {cfg.workers}"
```

This is transpiled to:

```cpp
cfg.Get("workers")
```

### Arena Allocation

The `@f32[n]` operator allocates a float array in the arena:

```serenade
let weights = @f32[1024]
```

Transpiles to:

```cpp
auto weights = Sr_ArenaAllocF32_Safe(1024);
```

The `_Safe` variant auto-initializes the arena if needed.

### Pointer Indexing

The `^[i]` operator provides pointer-style indexing:

```serenade
let value = buffer^[index]
```

Transpiles to:

```cpp
auto value = *((buffer) + (index));
```

### Safe Access

The `?.` operator provides null-safe member access:

```serenade
let name = player?.name
```

Transpiles to:

```cpp
auto name = ((player) ? (player)->name : nullptr);
```

### Pipe Operator

The `>>` operator pipes values through functions:

```serenade
let result = data >> process >> filter >> collect
```

Transpiles to:

```cpp
auto result = Sr_Pipe(Sr_Pipe(Sr_Pipe(data, process), filter), collect);
```

The `Sr_Pipe` function template forwards values:

```cpp
template <typename T, typename F>
auto Sr_Pipe(T value, F fn) -> decltype(fn(value)) {
  return fn(value);
}
```

---

## Statements

Statements control execution flow.

### Expression Statements

An expression can be used as a statement:

```serenade
compute(x, y)
hp + 10
```

### Let Statements

Variable declarations:

```serenade
let x = value
```

### Emit Statements

Output to stdout with interpolation:

```serenade
emit "result: {result}"
```

### If Statements

Conditional execution with full support for `elif` and `else` chains:

```
if_stmt = "if" expression block { "elif" expression block } [ "else" block ] .
block   = "{" { statement } "}" .
```

#### Basic If

```serenade
if hp < 50 {
  emit "low health"
}
```

**Transpiles to:**
```cpp
if (hp < 50) {
  std::cout << "low health" << std::endl;
}
```

#### If-Elif-Else Chains

```serenade
if hp < 30 {
  emit "critical"
} elif hp < 70 {
  emit "damaged"
} else {
  emit "healthy"
}
```

**Transpiles to:**
```cpp
if (hp < 30) {
  std::cout << "critical" << std::endl;
} else if (hp < 70) {
  std::cout << "damaged" << std::endl;
} else {
  std::cout << "healthy" << std::endl;
}
```

#### Multiple Elif Branches

```serenade
if score >= 90 {
  emit "grade: A"
} elif score >= 80 {
  emit "grade: B"
} elif score >= 70 {
  emit "grade: C"
} elif score >= 60 {
  emit "grade: D"
} else {
  emit "grade: F"
}
```

#### Nested If Statements

```serenade
if player_alive {
  if hp < 20 {
    emit "critical health!"
  } elif hp < 50 {
    emit "low health"
  } else {
    emit "healthy"
  }
} else {
  emit "game over"
}
```

### Match Statements

Pattern matching for cleaner conditional logic:

```serenade
match value {
  case pattern {
    # statements
  }
  case pattern {
    # statements
  }
}
```

#### Match with Option Types

```serenade
let player option = find_player(42)

match player {
  case some(p) {
    emit "Found: {p.name}"
  }
  case none {
    emit "Player not found"
  }
}
```

**Transpiles to:**
```cpp
if (player.has_value()) {
  auto p = player.value();
  std::cout << "Found: " << p.name << std::endl;
} else if (!player.has_value()) {
  std::cout << "Player not found" << std::endl;
}
```

#### Match with Result Types

```serenade
let result = divide(10.0, 2.0)

match result {
  case ok(val) {
    emit "Result: {val}"
  }
  case err(msg) {
    emit "Error: {msg}"
  }
}
```

**Transpiles to:**
```cpp
if (result.is_ok()) {
  auto val = result.unwrap();
  std::cout << "Result: " << val << std::endl;
} else if (result.is_err()) {
  auto msg = result.error();
  std::cout << "Error: " << msg << std::endl;
}
```

#### Match with Numeric Values

```serenade
match status {
  case 0 {
    emit "success"
  }
  case 1 {
    emit "warning"
  }
  case 2 {
    emit "error"
  }
}
```

**Transpiles to:**
```cpp
auto _sr_match = status;
if (_sr_match == 0) {
  std::cout << "success" << std::endl;
} else if (_sr_match == 1) {
  std::cout << "warning" << std::endl;
} else if (_sr_match == 2) {
  std::cout << "error" << std::endl;
}
```

### Loop Statements

Serenade provides multiple loop constructs: `cycle`, `for`, `foreach`, and `while`.

#### Cycle — Count-Based Loop

```
cycle_stmt = "cycle" expression [ "as" identifier ] block .
```

Iterate a fixed number of times:

```serenade
cycle 10 as i {
  emit "iteration {i}"
}
```

The variable defaults to `i` if not specified:

```serenade
cycle 10 {
  emit "iteration {i}"
}
```

**Transpiles to:**
```cpp
for (long long i = 0; i < 10; i++) {
  std::cout << "iteration " << i << std::endl;
}
```

#### Cycle — Range Loop

```
range_cycle = "cycle" expression ".." expression [ "as" identifier ] block .
```

Iterate over a range (inclusive):

```serenade
cycle 0..9 as n {
  emit "n={n}"
}
```

Ranges are **inclusive** on both ends. The loop `0..9` iterates from 0 to 9 (10 iterations).

**Transpiles to:**
```cpp
for (long long n = 0; n <= 9; n++) {
  std::cout << "n=" << n << std::endl;
}
```

#### For Loop — C-Style

Traditional C-style for loop with initialization, condition, and increment:

```serenade
for let i = 0; i < 100; i = i + 1 {
  emit "i={i}"
}
```

**Transpiles to:**
```cpp
for (auto i = 0; i < 100; i = i + 1) {
  std::cout << "i=" << i << std::endl;
}
```

#### For Loop — Range Syntax

Simpler range-based syntax:

```serenade
for i in 0..99 {
  emit "i={i}"
}
```

**Transpiles to:**
```cpp
for (long long i = 0; i <= 99; i++) {
  std::cout << "i=" << i << std::endl;
}
```

#### For Loop — Step Syntax

Range with custom step:

```serenade
for i in 0..100 step 10 {
  emit "i={i}"  # 0, 10, 20, ..., 100
}
```

**Transpiles to:**
```cpp
for (long long i = 0; i <= 100; i += 10) {
  std::cout << "i=" << i << std::endl;
}
```

#### Foreach Loop — Array Iteration

Iterate over array elements:

```serenade
let items = @i32[10]
cycle 10 as i {
  items^[i] = i * 2
}

foreach item in items count 10 {
  emit "item={item}"
}
```

**Transpiles to:**
```cpp
for (long long _sr_idx = 0; _sr_idx < 10; _sr_idx++) {
  auto item = items[_sr_idx];
  std::cout << "item=" << item << std::endl;
}
```

**Note:** `foreach` requires a `count` parameter specifying array length, since C++ arrays don't carry size information.

#### While Loop

Condition-based loop:

```serenade
let count = 0
while count < 10 {
  emit "count={count}"
  count = count + 1
}
```

**Transpiles to:**
```cpp
while (count < 10) {
  std::cout << "count=" << count << std::endl;
  count = count + 1;
}
```

#### Break Statement

Exit a loop early:

```serenade
cycle 100 as i {
  if i == 50 {
    break
  }
  emit "i={i}"
}
```

**Transpiles to:**
```cpp
for (long long i = 0; i < 100; i++) {
  if (i == 50) {
    break;
  }
  std::cout << "i=" << i << std::endl;
}
```

#### Continue Statement

Skip to next iteration:

```serenade
cycle 10 as i {
  if i % 2 == 0 {
    continue
  }
  emit "odd: {i}"
}
```

**Transpiles to:**
```cpp
for (long long i = 0; i < 10; i++) {
  if (i % 2 == 0) {
    continue;
  }
  std::cout << "odd: " << i << std::endl;
}
```

#### Loop Comparison Table

| Loop Type | Use Case | Example |
|-----------|----------|---------|
| `cycle N` | Fixed iteration count | `cycle 100 { ... }` |
| `cycle A..B` | Inclusive range | `cycle 0..9 { ... }` |
| `for i in A..B` | Inclusive range (alt syntax) | `for i in 0..9 { ... }` |
| `for i in A..B step S` | Range with custom step | `for i in 0..100 step 10 { ... }` |
| `for init; cond; incr` | C-style loop | `for let i=0; i<10; i=i+1 { ... }` |
| `foreach item in arr count N` | Array iteration | `foreach x in data count 100 { ... }` |
| `while cond` | Condition-based | `while running { ... }` |

### Return Statements

Return from a task:

```serenade
task add(a, b) {
  return a + b
}
```

### Wait and Sleep

Pause execution:

```serenade
wait 1000    # wait 1000ms
sleep 500    # sleep 500ms
```

Both accept millisecond values and compile to:

```cpp
Sr_Wait(milliseconds);
```

Implementation:

```cpp
static void Sr_Wait(double ms) {
  if (ms <= 0.0) return;
  std::this_thread::sleep_for(std::chrono::milliseconds((int)ms));
}
```

---

## Declarations

### Struct Declarations

Define custom data types with named fields and optional type annotations:

```
struct_decl = "struct" identifier "{" field_list "}" .
field_list = field { field } .
field = identifier [ type_annotation ] .
type_annotation = "i32" | "i64" | "f32" | "f64" | "str" | ... .
```

#### Basic Struct

```serenade
struct Point {
    x
    y
}

let p = Point { 10.0, 20.0 }
emit "Point: ({p.x}, {p.y})"
```

**Transpiles to:**
```cpp
struct Point {
    double x;
    double y;
};

auto p = Point{10.0, 20.0};
std::cout << "Point: (" << p.x << ", " << p.y << ")" << std::endl;
```

#### Struct with Type Annotations

```serenade
struct Entity {
    id i32
    x f64
    y f64
    health i32
    alive i32
}

let player = Entity { 0, 100.0, 200.0, 100, 1 }
player.health = 80
```

**Transpiles to:**
```cpp
struct Entity {
    int id;
    double x;
    double y;
    int health;
    int alive;
};

auto player = Entity{0, 100.0, 200.0, 100, 1};
player.health = 80;
```

#### Type Annotation Reference

| Serenade Type | C++ Type | Description |
|---------------|----------|-------------|
| `i32` | `int` | 32-bit signed integer |
| `i64` | `long long` | 64-bit signed integer |
| `f32` | `float` | 32-bit floating point |
| `f64` | `double` | 64-bit floating point |
| `str` | `std::string` | String type |

#### Nested Structs

```serenade
struct Color {
    r f32
    g f32
    b f32
}

struct Sprite {
    x f64
    y f64
    color Color
}

let red = Color { 1.0, 0.0, 0.0 }
let sprite = Sprite { 100.0, 200.0, red }
```

#### Struct Best Practices

```serenade
# ✓ GOOD: Type annotations for clarity and performance
struct Particle {
    x f32        # Single-precision for GPU
    y f32
    vx f32
    vy f32
    life i32     # Integer for frame count
}

# ✓ GOOD: Semantic field names
struct AABB {
    min_x f64
    min_y f64
    max_x f64
    max_y f64
}

# ✗ AVOID: Generic names without types
struct Thing {
    a
    b
    c
}
```

### Function Declarations

Serenade supports two function declaration syntaxes: `task` and `fn`.

#### Task Syntax (Traditional)

```
task_decl = "task" identifier "(" [ param_list ] ")" block .
param_list = identifier { "," identifier } .
```

Example:

```serenade
task add(a, b) {
  return a + b
}

task greet(name) {
  emit "Hello, {name}!"
}

task process() {
  emit "processing..."
}
```

**Transpiles to:**
```cpp
auto add = [&](auto a, auto b) {
  return a + b;
};

auto greet = [&](auto name) {
  std::cout << "Hello, " << name << "!" << std::endl;
};

auto process = [&]() {
  std::cout << "processing..." << std::endl;
};
```

Tasks are first-class values:

```serenade
task add(a, b) {
  return a + b
}

let f = add
let result = f(10, 20)
```

#### Fn Syntax (Modern)

Shorter syntax with optional return type annotation:

```serenade
fn multiply(a, b) {
  return a * b
}

fn divide(a, b) result {
  if b == 0.0 {
    return err("division by zero")
  }
  return ok(a / b)
}
```

**Transpiles to:**
```cpp
auto multiply = [&](auto a, auto b) {
  return a * b;
};

auto divide = [&](auto a, auto b) -> Sr_Result<double> {
  if (b == 0.0) {
    return Sr_Err("division by zero");
  }
  return Sr_Ok(a / b);
};
```

#### Return Type Annotations

| Annotation | C++ Return Type | Use Case |
|------------|-----------------|----------|
| (none) | `auto` | Type deduced from return statement |
| `result` | `Sr_Result<T>` | Function that can fail with error |
| `option` | `Sr_Option<T>` | Function that may return no value |

#### Lambda Parameters

Functions capture by reference (`[&]`) and use `auto` parameters for generic types:

```serenade
fn process(data) {
  emit "Processing {data}"
}

process(42)           # auto data = 42 (int)
process(3.14)         # auto data = 3.14 (double)
process("hello")      # auto data = "hello" (const char*)
```

Tasks are first-class values:

```serenade
let fn = task double(x) { return x * 2 }
let result = fn(5)
```

Transpiles to:

```cpp
auto add = [&](auto a, auto b) {
  return a + b;
};

auto greet = [&](auto name) {
  Sr_Emit("Hello, " + Sr_ToString(name) + "!");
};

auto process = [&]() {
  Sr_Emit(std::string("processing..."));
};
```

### Const Declarations

Declare compile-time constants:

```
const_decl = "const" identifier "=" expression .
```

Example:

```serenade
const PI = 3.14159265
const MAX_ENTITIES = 1000
const GAME_TITLE = "My Game"

let circumference = 2.0 * PI * radius
```

**Transpiles to:**
```cpp
const auto PI = 3.14159265;
const auto MAX_ENTITIES = 1000;
const auto GAME_TITLE = "My Game";

auto circumference = 2.0 * PI * radius;
```

#### Const vs Let

| Feature | `const` | `let` |
|---------|---------|-------|
| Mutability | Immutable | Mutable |
| C++ Output | `const auto` | `auto` |
| Use Case | Named constants, configuration | General variables |

```serenade
const SPEED = 5.0
let position = 0.0

position = position + SPEED  # OK
# SPEED = 10.0  # ERROR: cannot modify const
```

### Defer Statements

Execute code at scope exit (RAII pattern inspired by Go):

```
defer_stmt = "defer" expression .
```

Example:

```serenade
let file = open_file("data.txt")
defer close_file(file)

# ... work with file
# file is automatically closed when scope exits
```

**Transpiles to:**
```cpp
auto file = open_file("data.txt");
Sr_DeferGuard _sr_defer_0([&]() {
  close_file(file);
});

// ... work with file
// Sr_DeferGuard destructor calls close_file at scope exit
```

#### Sr_DeferGuard Implementation

```cpp
struct Sr_DeferGuard {
    std::function<void()> fn;
    Sr_DeferGuard(std::function<void()> f) : fn(f) {}
    ~Sr_DeferGuard() { if (fn) fn(); }
};
```

#### Multiple Defers

Defers execute in **reverse order** (LIFO — last in, first out):

```serenade
emit "opening resources"

defer emit "close 1"
defer emit "close 2"
defer emit "close 3"

emit "using resources"
```

**Output:**
```
opening resources
using resources
close 3
close 2
close 1
```

#### Defer Best Practices

```serenade
# ✓ GOOD: Pair acquire/release operations
fn load_texture(path) {
    let tex = gpu_alloc_texture()
    defer gpu_free_texture(tex)

    gpu_load_image(tex, path)
    return tex
}

# ✓ GOOD: Ensure cleanup on early return
fn process_file(path) {
    let f = open(path)
    defer close(f)

    let header = read_header(f)
    if header.invalid {
        return  # close(f) still called!
    }

    # ... process file
}

# ✓ GOOD: Restore state after modification
fn with_gl_state() {
    let old_blend = gl_get_blend()
    defer gl_set_blend(old_blend)

    gl_set_blend(1)
    # ... render with blending
    # old_blend restored automatically
}

# ✗ BAD: Manual cleanup (error-prone with early returns)
fn process_file(path) {
    let f = open(path)

    let header = read_header(f)
    if header.invalid {
        close(f)
        return
    }

    # ... process
    close(f)  # Easy to forget!
}
```

#### Defer with Scopes

Combine `defer` with `scope` for arena management:

```serenade
scope gpu_work {
    let buf = @f32[1024]
    defer emit "Freeing buffer"

    # ... use buf
}
# Prints "Freeing buffer" then resets arena
```

### Registry Declarations

Registries are key-value configuration objects:

```
registry_decl = "registry" "{" { key ":" value [ "," ] } "}" .
```

Example:

```serenade
let cfg = registry {
  mode: "local",
  workers: 4,
  arena_size: 64mb,
  enable_logging: true
}
```

Transpiles to:

```cpp
auto cfg = SrRegistry{};
cfg.Set("mode", SrValue("local"));
cfg.Set("workers", SrValue(4));
cfg.Set("arena_size", SrValue(64 * 1024 * 1024));
cfg.Set("enable_logging", SrValue(true));
```

Access:

```serenade
emit "mode: {cfg.mode}"
```

Becomes:

```cpp
Sr_Emit("mode: " + cfg.Get("mode").ToString());
```

### Atomic Declarations

Thread-safe atomic integers:

```
atomic_decl = "atomic" identifier "=" expression .
```

Example:

```serenade
atomic counter = 0
```

Transpiles to:

```cpp
std::atomic<long long> counter(0);
```

---

## Built-in Functions

### Emit

Output to stdout:

```serenade
emit "message"
emit "value: {x}"
```

### Wait / Sleep

Pause execution:

```serenade
wait 1000
sleep 500
```

### Input

Read from stdin:

```serenade
emit "Enter name:"
let name = input()
emit "Enter value:"
let value = input_num()
```

Transpiles to:

```cpp
auto name = Sr_Input();
auto value = Sr_InputNum();
```

### Move

Explicit move semantics:

```serenade
let x = move buffer
```

Transpiles to:

```cpp
auto x = std::move(buffer);
```

### Scan

Code scanning for RAG/search:

```serenade
let results = scan "/path/to/code" using engine
let top = results.top(10)
```

### Spawn

Async task execution:

```serenade
let job = spawn heavy_work()
```

Transpiles to:

```cpp
auto job = Sr_Spawn([&]() { heavy_work(); });
```

Implementation:

```cpp
static std::future<void> Sr_Spawn(std::function<void()> fn) {
  return std::async(std::launch::async, std::move(fn));
}
```

### Await

Wait for async task:

```serenade
await job
```

Transpiles to:

```cpp
Sr_Await(job);
```

Implementation:

```cpp
static void Sr_Await(std::future<void>& f) {
  f.wait();
}
```

### Mathematical Functions

Standard math functions available in Serenade:

#### Basic Math

```serenade
let a = sqrt(16.0)      # Square root → 4.0
let b = abs(-5.0)       # Absolute value → 5.0
let c = floor(3.7)      # Floor → 3.0
let d = ceil(3.2)       # Ceiling → 4.0
let e = round(3.5)      # Round → 4.0
let f = pow(2.0, 8.0)   # Power → 256.0
```

#### Trigonometric Functions

```serenade
const PI = 3.14159265

let s = sin(PI / 2.0)   # Sine → 1.0
let c = cos(0.0)        # Cosine → 1.0
let t = tan(PI / 4.0)   # Tangent → 1.0
let a = asin(1.0)       # Arc sine → π/2
let b = acos(0.0)       # Arc cosine → π/2
let c = atan(1.0)       # Arc tangent → π/4
let d = atan2(y, x)     # Two-argument arc tangent
```

#### Exponential and Logarithmic

```serenade
let a = exp(1.0)        # e^x → 2.71828...
let b = log(2.71828)    # Natural log (ln) → 1.0
let c = log10(100.0)    # Base-10 log → 2.0
let d = log2(8.0)       # Base-2 log → 3.0
```

#### Min/Max

```serenade
let a = min(5.0, 10.0)  # Minimum → 5.0
let b = max(5.0, 10.0)  # Maximum → 10.0
```

**Note:** All math functions map to C++ `<cmath>` equivalents.

### String Operations

#### String Construction

```serenade
let s1 = "hello"
let s2 = "world"
let combined = s1 + " " + s2  # "hello world"
```

#### String Interpolation

Only available in `emit` statements:

```serenade
let name = "Alice"
let age = 30
emit "Name: {name}, Age: {age}"
```

**Important:** Regular string literals do NOT support `{var}` interpolation. Only `emit` performs interpolation.

#### String Conversion Functions

```serenade
# String to bytes
let bytes = str_bytes(text)
let count = str_bytes_count(text)

# Bytes to string
let text = str_from_bytes(bytes, count)

# Number to string
let s = to_string(42)
let t = to_string(3.14)
```

**Transpiles to:**
```cpp
// str_bytes: const std::string& → const unsigned char*
// str_from_bytes: construct string from byte array
// to_string: std::to_string()
```

#### String Comparison

```serenade
if str1 == str2 {
  emit "equal"
}

if str1 != str2 {
  emit "not equal"
}
```

### Type Conversion

```serenade
# String to number
let n = parse_int("42")       # → 42
let f = parse_float("3.14")   # → 3.14

# Number to string
let s1 = to_string(42)        # → "42"
let s2 = to_string(3.14)      # → "3.14"

# Float to int (truncation)
let i = int(3.9)              # → 3

# Int to float
let f = float(42)             # → 42.0
```

### Random Number Generation

```serenade
# Random float in [0.0, 1.0)
let r = random()

# Random int in [min, max)
let n = random_int(1, 100)

# Seed the RNG
random_seed(12345)
```

**Transpiles to:**
```cpp
static std::mt19937 Sr_Rng;

float Sr_Random() {
  std::uniform_real_distribution<float> dist(0.0f, 1.0f);
  return dist(Sr_Rng);
}

int Sr_RandomInt(int min, int max) {
  std::uniform_int_distribution<int> dist(min, max - 1);
  return dist(Sr_Rng);
}

void Sr_RandomSeed(unsigned int seed) {
  Sr_Rng.seed(seed);
}
```

### Time Functions

```serenade
# Get current time in milliseconds
let now = time_ms()

# Measure elapsed time
let start = time_ms()
# ... do work
let end = time_ms()
let elapsed = end - start
emit "Elapsed: {elapsed} ms"
```

**Transpiles to:**
```cpp
double Sr_TimeMs() {
  auto now = std::chrono::high_resolution_clock::now();
  auto duration = now.time_since_epoch();
  return std::chrono::duration<double, std::milli>(duration).count();
}
```

### File I/O Functions

```serenade
# Check if file exists
if file_exists("data.txt") {
  emit "File found"
}

# Read entire file as string
let content = read_file("data.txt")

# Write string to file
write_file("output.txt", content)

# Append to file
append_file("log.txt", "New entry\n")

# Get file size
let size = file_size("data.bin")
```

### Memory Utility Functions

```serenade
# Allocate uninitialized memory
let ptr = malloc(1024)

# Free memory
free(ptr)

# Zero memory
memset(buffer, 0, 1024)

# Copy memory
memcpy(dest, src, 1024)
```

**Warning:** These are low-level functions. Prefer arena allocation (`@f32[n]`) and smart pointers (`box`, `rc`, `arc`) for safe memory management.

### Assertion and Debugging

```serenade
# Runtime assertion (aborts if false)
assert(x > 0, "x must be positive")

# Debug print (only in debug builds)
debug("checkpoint reached")
debug("value: {x}")

# Print without interpolation
print("raw message")
```

**Transpiles to:**
```cpp
void Sr_Assert(bool cond, const std::string& msg) {
  if (!cond) {
    fprintf(stderr, "Assertion failed: %s\n", msg.c_str());
    abort();
  }
}

#ifdef DEBUG
void Sr_Debug(const std::string& msg) {
  std::cerr << "[DEBUG] " << msg << std::endl;
}
#else
void Sr_Debug(const std::string& msg) { }
#endif
```

### GPU Utility Functions

```serenade
# Check CUDA availability
if cuda_available() {
  emit "CUDA detected"
  let devices = cuda_device_count()
  emit "Devices: {devices}"
}

# Get GPU device properties
let name = cuda_device_name(0)
let memory = cuda_device_memory(0)
emit "GPU: {name}, Memory: {memory} MB"

# Synchronize GPU
gpu_sync()
```

---

## Memory Management

### Arena Allocator

Serenade provides a fast arena allocator for float arrays.

#### Initialization

The arena is auto-initialized on first use, or manually:

```serenade
native {
  Sr_ArenaInit(64 * 1024 * 1024);
}
endnative
```

#### Allocation

```serenade
let weights = @f32[1024]
```

This allocates 1024 floats with 64-byte alignment.

#### Arena Functions

Available in native blocks:

```cpp
Sr_ArenaInit(size_t capacity);
Sr_ArenaCapacity();
Sr_ArenaOffset();
Sr_ArenaReset();
Sr_ArenaResetEmbeddings();
Sr_ArenaAllocF32(size_t count);
Sr_ArenaAllocF32_Safe(size_t count);
```

The `_Safe` variant auto-initializes if needed.

#### Implementation

```cpp
static size_t g_ArenaCapacity = 0;
static size_t g_ArenaOffset = 0;
static unsigned char* g_ArenaBuffer = nullptr;

static inline void Sr_ArenaInit(size_t cap) {
  if (g_ArenaBuffer) return;
  g_ArenaCapacity = cap;
  g_ArenaBuffer = new unsigned char[cap];
  std::memset(g_ArenaBuffer, 0, cap);
  g_ArenaOffset = 0;
}

static inline float* Sr_ArenaAllocF32(size_t count) {
  size_t bytes = count * sizeof(float);
  size_t aligned = (bytes + 63) & ~63ull;
  if (g_ArenaOffset + aligned > g_ArenaCapacity) {
    std::fprintf(stderr, "[arena] OOM\n");
    return nullptr;
  }
  float* ptr = reinterpret_cast<float*>(g_ArenaBuffer + g_ArenaOffset);
  g_ArenaOffset += aligned;
  return ptr;
}
```

### Manual Memory

For C++-style memory management, use native blocks:

```serenade
native {
  float* data = new float[1000];
  delete[] data;
}
endnative
```

---

## Rust-Inspired Memory Safety

Serenade includes a comprehensive memory safety system inspired by Rust, providing compile-time ownership tracking, borrowing rules, smart pointers, and error handling types. These features prevent common memory bugs while maintaining Serenade's simple syntax.

### Philosophy

**"Simple as Python, powerful as C++/Rust/CUDA under the hood"**

The safety features transpile to efficient C++ with zero-cost abstractions. All safety checks happen at transpile-time (compile-time errors) or use RAII guards, with no runtime overhead beyond what you'd write manually.

### Ownership System

#### The `own` Keyword

Declare unique ownership of a value:

```serenade
struct Buffer {
    size i32
    name str
}

own data = Buffer { 1024, "gpu_buffer" }
data->size = 2048
emit "Buffer: {data->name}, size: {data->size}"
```

**Transpiles to:**
```cpp
auto data = Sr_Own<Buffer>(Buffer{1024, "gpu_buffer"});
data->size = 2048;
std::cout << "Buffer: " << data->name << ", size: " << data->size << std::endl;
```

The `Sr_Own<T>` template wraps `std::unique_ptr<T>` and tracks move semantics.

#### Move Semantics

Transfer ownership using `move`:

```serenade
own original = Buffer { 512, "temp" }
let transferred = move original

# This line would cause a COMPILE ERROR:
# emit original->name
# Error: use of moved variable 'original'
```

**Key Rules:**
- After `move`, the original variable is marked as **moved**
- Any subsequent access to a moved variable produces `#error` at transpile-time
- This is **hard enforcement** — the transpiler inserts C++ preprocessor errors

**Transpiles to:**
```cpp
auto original = Sr_Own<Buffer>(Buffer{512, "temp"});
auto transferred = std::move(original);
// Accessing 'original' here would generate:
// #error "use of moved variable 'original'"
```

#### Ownership Best Practices

```serenade
# ✓ GOOD: Transfer ownership explicitly
own data = load_data()
process(move data)

# ✗ BAD: Trying to use after move
own data = load_data()
let copy = move data
emit data->size  # COMPILE ERROR!

# ✓ GOOD: Borrow instead of moving
own data = load_data()
ref view = data
print_info(view)
emit data->size  # OK — still own it
```

### Borrowing System

Serenade enforces Rust-like borrowing rules at transpile-time:

1. **Any number of immutable borrows** (`ref`)
2. **Exactly ONE mutable borrow** (`mut ref`)
3. **No mutable borrow while immutable borrows exist**

#### Immutable Borrowing: `ref`

Create read-only references:

```serenade
own buffer = Buffer { 1024, "data" }

ref view1 = buffer
ref view2 = buffer
ref view3 = buffer

emit "View 1 size: {view1.size}"
emit "View 2 name: {view2.name}"
emit "View 3 size: {view3.size}"
emit "Original: {buffer->size}"  # OK — owner still accessible
```

**Transpiles to:**
```cpp
auto buffer = Sr_Own<Buffer>(Buffer{1024, "data"});
Sr_Ref<Buffer> _sr_ref_view1(buffer); const auto& view1 = *_sr_ref_view1;
Sr_Ref<Buffer> _sr_ref_view2(buffer); const auto& view2 = *_sr_ref_view2;
Sr_Ref<Buffer> _sr_ref_view3(buffer); const auto& view3 = *_sr_ref_view3;
// ... uses
```

The `Sr_Ref<T>` RAII guard increments a borrow counter on construction and decrements on destruction.

#### Mutable Borrowing: `mut ref`

Create exclusive write access:

```serenade
own buffer = Buffer { 512, "mutable" }

mut ref editor = buffer
editor.size = 2048
editor.name = "resized"

emit "New size: {buffer->size}"  # 2048
```

**Borrow Conflict Detection:**

```serenade
own data = Buffer { 1024, "test" }

ref reader = data
mut ref writer = data  # COMPILE ERROR!
# Error: cannot create mutable borrow of 'data' while immutable borrows exist
```

**Double Mutable Borrow:**

```serenade
own data = Buffer { 1024, "test" }

mut ref editor1 = data
mut ref editor2 = data  # COMPILE ERROR!
# Error: cannot create second mutable borrow of 'data'
```

The transpiler tracks active borrows in `NextContext::borrowed` and `NextContext::mutBorrowed` sets and emits `#error` directives when conflicts are detected.

#### Borrowing Best Practices

```serenade
# ✓ GOOD: Many readers
own data = load_large_dataset()
ref view1 = data
ref view2 = data
process_readonly(view1, view2)

# ✓ GOOD: Exclusive writer
own data = create_buffer()
mut ref writer = data
writer.size = 4096
flush(writer)

# ✗ BAD: Reader + writer simultaneously
own data = create_buffer()
ref reader = data
mut ref writer = data  # ERROR: conflict!

# ✓ GOOD: Sequential access (refs go out of scope)
own data = create_buffer()
{
    ref reader = data
    emit reader.size
}
# reader destroyed here
mut ref writer = data  # OK now
writer.size = 2048
```

### Smart Pointers

Serenade provides three smart pointer types that map to C++ standard library equivalents.

#### `box` — Unique Pointer

Heap-allocated, unique ownership (maps to `std::unique_ptr`):

```serenade
struct Point {
    x f64
    y f64
}

let p = box Point { 100.0, 200.0 }
emit "Point: ({p->x}, {p->y})"

let q = box Point { 5.0, 10.0 }
let moved = q  # Transfer ownership
# q is now invalid
```

**Transpiles to:**
```cpp
auto p = std::make_unique<Point>(Point{100.0, 200.0});
std::cout << "Point: (" << p->x << ", " << p->y << ")" << std::endl;
```

#### `rc` — Reference Counted Pointer

Shared ownership (maps to `std::shared_ptr`):

```serenade
let shared1 = rc Point { 42.0, 84.0 }
let shared2 = shared1  # Both point to same data
let shared3 = shared1

emit "All three share: ({shared1->x}, {shared1->y})"
shared2->x = 99.0
emit "Modified via shared2: {shared1->x}"  # 99.0
```

**Transpiles to:**
```cpp
auto shared1 = std::make_shared<Point>(Point{42.0, 84.0});
auto shared2 = shared1;
auto shared3 = shared1;
```

Use `rc` when multiple owners need access to the same data and you're in single-threaded code.

#### `arc` — Atomic Reference Counted Pointer

Thread-safe shared ownership with mutex-guarded access:

```serenade
let safe = arc Point { 7.0, 14.0 }

task worker(data) {
    # Safe concurrent access
    data.lock(fn(pt) {
        pt->x = pt->x + 1.0
        emit "Updated x: {pt->x}"
    })
}

let j1 = spawn worker(safe)
let j2 = spawn worker(safe)
await j1
await j2
```

**Transpiles to:**
```cpp
auto safe = Sr_Arc<Point>(Point{7.0, 14.0});
// Sr_Arc<T> wraps std::shared_ptr with std::mutex
// .lock(fn) acquires lock, calls lambda, releases on scope exit
```

**Comparison Table:**

| Type | Ownership | Thread-Safe | Use Case |
|------|-----------|-------------|----------|
| `box` | Unique | No | Single owner, heap allocation |
| `rc` | Shared | No | Multiple owners, single-threaded |
| `arc` | Shared | Yes | Multiple owners, multi-threaded |
| `own` | Unique | No | Move semantics, compile-time tracking |

### Runtime Assertions: `guard`

Runtime safety checks that abort on failure.

#### Null Pointer Guard

```serenade
let ptr = box Point { 1.0, 2.0 }
guard ptr
emit "ptr is valid: ({ptr->x}, {ptr->y})"
```

**Transpiles to:**
```cpp
if (!(ptr)) {
    fprintf(stderr, "Guard failed at line %d: ptr\n", __LINE__);
    abort();
}
```

#### Boolean Expression Guard

```serenade
let idx = 5
let len = 10
guard idx >= 0 and idx < len
emit "Index {idx} is valid for array of size {len}"
```

**Transpiles to:**
```cpp
if (!((idx >= 0) && (idx < len))) {
    fprintf(stderr, "Guard failed at line %d: idx >= 0 and idx < len\n", __LINE__);
    abort();
}
```

#### Guards in Functions

```serenade
fn array_get(arr, idx, len) {
    guard idx >= 0 and idx < len
    return arr^[idx]
}

let data = @i32[100]
let val = array_get(data, 50, 100)  # OK
# let bad = array_get(data, 200, 100)  # Would abort at runtime
```

**Best Practices:**

```serenade
# ✓ GOOD: Guard at function entry
fn process(ptr) {
    guard ptr
    # ... safe to use ptr
}

# ✓ GOOD: Guard array bounds
fn get(arr, i, n) {
    guard i >= 0 and i < n
    return arr^[i]
}

# ✗ BAD: Redundant guards
guard ptr
guard ptr  # Unnecessary — already checked
```

### Scoped Resources: `scope`

Create arena-scoped blocks where allocations are freed on exit.

```serenade
emit "Before scope"

scope gpu_work {
    let temp_buf = @f32[1024]
    let another = @f64[512]

    # ... use temporary buffers
    temp_buf^[0] = 42.0
    emit "Allocated 1024 floats + 512 doubles on arena"
}

emit "Scope exited — arena reset, memory reclaimed"
```

**Transpiles to:**
```cpp
std::cout << "Before scope" << std::endl;
{
    size_t _sr_arena_mark = Sr_ArenaOffset();
    Sr_DeferGuard _sr_scope_guard([&]() {
        Sr_ArenaResetTo(_sr_arena_mark);
    });

    auto temp_buf = Sr_ArenaAllocF32_Safe(1024);
    auto another = Sr_ArenaAllocF64_Safe(512);

    temp_buf[0] = 42.0;
    // ...
}
std::cout << "Scope exited — arena reset, memory reclaimed" << std::endl;
```

**Use Cases:**

```serenade
# GPU frame-local allocations
scope frame {
    let verts = @f32[vertex_count * 3]
    let norms = @f32[vertex_count * 3]
    gpu_upload(verts, norms)
}
# All frame data freed here

# Temporary computation buffers
scope compute {
    let scratch = @f64[1000000]
    compute_fft(data, scratch)
}
# Scratch buffer freed
```

### Unsafe Blocks: `unsafe`

Disable safety checks for performance-critical or low-level code.

```serenade
unsafe {
    emit "Safety checks disabled in this block"
    let raw_ptr = allocate_memory(1024)
    # Direct pointer arithmetic, no guards
    raw_ptr^[0] = 42
}
```

**Transpiles to:**
```cpp
{
    // unsafeMode = true in parser context
    // No borrow checks, no move checks, no guard enforcement
    std::cout << "Safety checks disabled" << std::endl;
    // ... raw code
}
```

**When to Use:**

```serenade
# ✓ GOOD: Performance-critical inner loops
unsafe {
    cycle 1000000 {
        buffer^[i] = compute(i)  # No bounds checking overhead
    }
}

# ✓ GOOD: Low-level GPU/CUDA code
unsafe {
    native cpp {
        cudaMemcpy(dst, src, size, cudaMemcpyDeviceToHost);
    }
}

# ✗ BAD: Using unsafe to avoid fixing borrow conflicts
unsafe {
    ref r = data
    mut ref w = data  # Still a logic error, just no compile error
}
```

**Important:** `unsafe` disables *transpile-time* safety checks, but runtime guards still execute unless removed manually.

### Option Type

Handle nullable values safely without null pointer exceptions.

#### Declaration and Construction

```serenade
let x option = some(42)
let y option = none

emit "x has value: {x.has_value()}"  # true
emit "y has value: {y.has_value()}"  # false
```

**Transpiles to:**
```cpp
auto x = Sr_Option<int>(42);
auto y = Sr_Option<int>();
```

`Sr_Option<T>` wraps `std::optional<T>` with additional methods.

#### Unwrapping

```serenade
let x option = some(42)

# Safe unwrap with default
let val = x.unwrap_or(0)
emit "Value: {val}"  # 42

let y option = none
let def = y.unwrap_or(99)
emit "Default: {def}"  # 99

# Unsafe unwrap (aborts if none)
let dangerous = x.unwrap()  # OK — x is some(42)
# let crash = y.unwrap()  # Would abort — y is none
```

#### Pattern Matching

```serenade
let x option = some(42)

match x {
    case some(v) {
        emit "x has value: {v}"
    }
    case none {
        emit "x is none"
    }
}
```

**Transpiles to:**
```cpp
if (x.has_value()) {
    auto v = x.value();
    std::cout << "x has value: " << v << std::endl;
} else if (!x.has_value()) {
    std::cout << "x is none" << std::endl;
}
```

#### Real-World Example

```serenade
fn find_player(id) option {
    cycle entities_count {
        if entities^[i].id == id {
            return some(entities^[i])
        }
    }
    return none
}

let player = find_player(42)
match player {
    case some(p) {
        emit "Found player at ({p.x}, {p.y})"
    }
    case none {
        emit "Player not found"
    }
}
```

### Result Type

Elegant error handling without exceptions.

#### Declaration and Construction

```serenade
fn divide(a, b) result {
    if b == 0.0 {
        return err("division by zero")
    }
    return ok(a / b)
}

let r1 = divide(10.0, 2.0)
let r2 = divide(10.0, 0.0)
```

**Transpiles to:**
```cpp
auto divide = [&](auto a, auto b) -> Sr_Result<double> {
    if (b == 0.0) {
        return Sr_Err("division by zero");
    }
    return Sr_Ok(a / b);
};
```

#### Pattern Matching

```serenade
let result = divide(10.0, 3.0)

match result {
    case ok(val) {
        emit "Result: {val}"
    }
    case err(msg) {
        emit "Error: {msg}"
    }
}
```

**Transpiles to:**
```cpp
if (result.is_ok()) {
    auto val = result.unwrap();
    std::cout << "Result: " << val << std::endl;
} else if (result.is_err()) {
    auto msg = result.error();
    std::cout << "Error: " << msg << std::endl;
}
```

#### The `try` Operator

Auto-propagate errors up the call stack:

```serenade
fn compute() result {
    let a = try divide(100.0, 4.0)   # a = 25.0
    let b = try divide(a, 5.0)        # b = 5.0
    return ok(b)
}

let final = compute()
match final {
    case ok(val) {
        emit "Computed: {val}"  # 5.0
    }
    case err(msg) {
        emit "Error: {msg}"
    }
}
```

**How `try` Works:**

```serenade
let x = try divide(10.0, 0.0)
```

**Transpiles to:**
```cpp
auto _sr_try_tmp = divide(10.0, 0.0);
if (_sr_try_tmp.is_err()) {
    return Sr_ErrVal{_sr_try_tmp.error()};
}
auto x = _sr_try_tmp.unwrap();
```

The `Sr_ErrVal` type uses template conversion to propagate errors to any `Sr_Result<T>` return type.

#### Chaining Results

```serenade
fn safe_sqrt(x) result {
    if x < 0.0 {
        return err("sqrt of negative")
    }
    return ok(sqrt(x))
}

fn safe_divide(a, b) result {
    if b == 0.0 {
        return err("division by zero")
    }
    return ok(a / b)
}

fn complex_calc(a, b, c) result {
    let ratio = try safe_divide(a, b)
    let root = try safe_sqrt(ratio)
    let final = try safe_divide(root, c)
    return ok(final)
}

let ans = complex_calc(100.0, 4.0, 5.0)
match ans {
    case ok(v) {
        emit "Answer: {v}"
    }
    case err(m) {
        emit "Failed: {m}"
    }
}
```

Any error at any step propagates immediately to the caller.

#### Result Best Practices

```serenade
# ✓ GOOD: Return result from fallible operations
fn load_config(path) result {
    let file = try open_file(path)
    let data = try parse_json(file)
    return ok(data)
}

# ✗ BAD: Ignoring errors
fn load_config(path) {
    let file = open_file(path)  # What if it fails?
    return parse_json(file)     # Crashes on error
}

# ✓ GOOD: Propagate errors with try
fn process() result {
    let data = try load_config("config.json")
    let validated = try validate(data)
    return ok(validated)
}

# ✗ BAD: Manual error checking (verbose)
fn process() result {
    let cfg_result = load_config("config.json")
    match cfg_result {
        case err(m) { return err(m) }
        case ok(data) {
            let val_result = validate(data)
            match val_result {
                case err(m) { return err(m) }
                case ok(v) { return ok(v) }
            }
        }
    }
}
```

### Complete Safety Example: Game Entity System

Combining all safety features:

```serenade
struct Entity {
    id i32
    x f64
    y f64
    health i32
    alive i32
}

# Owned entity pool
own entity_pool = @Entity[1000]
atomic entity_count = 0

fn spawn_entity(x, y) option {
    let idx = entity_count
    if idx >= 1000 {
        return none
    }

    entity_count = entity_count + 1
    entity_pool^[idx].id = idx
    entity_pool^[idx].x = x
    entity_pool^[idx].y = y
    entity_pool^[idx].health = 100
    entity_pool^[idx].alive = 1

    return some(idx)
}

fn damage_entity(id, amount) result {
    guard id >= 0 and id < entity_count

    ref entity = entity_pool^[id]
    if entity.alive == 0 {
        return err("entity already dead")
    }

    mut ref e = entity_pool^[id]
    e.health = e.health - amount
    if e.health <= 0 {
        e.alive = 0
        emit "Entity {id} died"
    }

    return ok(e.health)
}

fn move_entity(id, dx, dy) result {
    guard id >= 0 and id < entity_count

    mut ref e = entity_pool^[id]
    if e.alive == 0 {
        return err("cannot move dead entity")
    }

    e.x = e.x + dx
    e.y = e.y + dy
    return ok(0)
}

# Spawn entities
let player = spawn_entity(0.0, 0.0)
match player {
    case some(id) {
        emit "Player spawned: {id}"

        # Move player
        let move_result = try move_entity(id, 10.0, 5.0)

        # Take damage
        let dmg_result = damage_entity(id, 30)
        match dmg_result {
            case ok(remaining) {
                emit "Player health: {remaining}"
            }
            case err(msg) {
                emit "Damage failed: {msg}"
            }
        }
    }
    case none {
        emit "Entity pool full!"
    }
}
```

### Safety Feature Summary

| Feature | Purpose | Compile-Time | Runtime | Example |
|---------|---------|--------------|---------|---------|
| `own` | Unique ownership | ✓ move tracking | RAII cleanup | `own data = Buffer{...}` |
| `ref` | Immutable borrow | ✓ conflict detection | Borrow counter | `ref view = data` |
| `mut ref` | Mutable borrow | ✓ exclusive check | Borrow flag | `mut ref edit = data` |
| `move` | Transfer ownership | ✓ use-after-move error | Zero cost | `let x = move y` |
| `box` | Unique pointer | — | RAII | `box Point{1.0, 2.0}` |
| `rc` | Shared pointer | — | Reference count | `rc Point{1.0, 2.0}` |
| `arc` | Thread-safe shared | — | Atomic refcount + mutex | `arc Point{1.0, 2.0}` |
| `guard` | Runtime assertion | — | Abort on fail | `guard ptr` |
| `scope` | Arena block | — | Stack unwinding | `scope gpu { ... }` |
| `unsafe` | Disable checks | Bypasses all checks | — | `unsafe { ... }` |
| `option` | Nullable value | — | Safe unwrap | `some(42)`, `none` |
| `result` | Error handling | ✓ return type | Safe unwrap | `ok(val)`, `err(msg)` |
| `try` | Error propagation | ✓ type checking | Early return | `let x = try f()` |

### Migration Guide: Adding Safety to Existing Code

#### Before (Unsafe):
```serenade
let data = @f32[1000]
let ptr = data

# Potential bugs:
# - No ownership tracking
# - Pointer could be null
# - No bounds checking
# - No error handling

ptr^[1500] = 42.0  # Buffer overflow!
```

#### After (Safe):
```serenade
own data = @f32[1000]
guard data

fn write_safe(arr, idx, val, len) result {
    guard arr
    if idx < 0 or idx >= len {
        return err("index out of bounds")
    }
    arr^[idx] = val
    return ok(0)
}

let result = write_safe(data, 1500, 42.0, 1000)
match result {
    case ok(_) {
        emit "Write successful"
    }
    case err(msg) {
        emit "Write failed: {msg}"  # "index out of bounds"
    }
}
```

### Performance Notes

All safety features compile to zero-cost or near-zero-cost C++ code:

- **`own`/`move`**: Zero runtime cost — pure compile-time tracking + `std::move()`
- **`ref`/`mut ref`**: Negligible cost — integer increment/decrement on stack
- **`box`/`rc`/`arc`**: Standard library overhead (`unique_ptr`, `shared_ptr`)
- **`guard`**: Single branch + abort (optimizes away in release if condition is constant)
- **`scope`**: Two integer operations (save offset, restore offset)
- **`unsafe`**: Zero cost — disables checks entirely
- **`option`/`result`**: Single byte flag + value (standard `optional` layout)
- **`try`**: One branch for early return

**Recommendation:** Use safety features everywhere during development, then profile. Only move to `unsafe` blocks if profiling shows measurable bottlenecks.

---

## Concurrency

### Spawn and Await

Basic concurrency model:

```serenade
task worker(id) {
  emit "worker {id} started"
  wait 1000
  emit "worker {id} done"
}

let job1 = spawn worker(1)
let job2 = spawn worker(2)

await job1
await job2
emit "all workers done"
```

### Atomics

Thread-safe counters:

```serenade
atomic total = 0

task increment() {
  cycle 1000 {
    total = total + 1
  }
}

let j1 = spawn increment()
let j2 = spawn increment()
await j1
await j2
emit "total: {total}"
```

Note: The atomic increment shown above is for demonstration. For real atomic operations, use native blocks:

```serenade
native {
  total.fetch_add(1, std::memory_order_relaxed);
}
endnative
```

---

## Assembly Integration

Serenade can generate inline x86-64 assembly for hot paths.

### ASM XOR8

Built-in vectorized XOR operation:

```serenade
asm xor8 buffer size key
```

For multi-threaded XOR:

```serenade
asm xor8 buffer size key threads passes
```

Example:

```serenade
let data = @f32[10000]
asm xor8 data 40000 0x5A 4 10
```

This generates NASM assembly:

```nasm
global Sr_AsmXor8
section .text
Sr_AsmXor8:
%ifidn __OUTPUT_FORMAT__,win64
    mov r9, rdx
    test r9, r9
    jz .done
    movzx eax, r8b
    vmovd xmm0, eax
    vpbroadcastb ymm0, xmm0
    mov r10, r9
    shr r10, 5
    jz .tail
.loop:
    vmovdqu ymm1, [rcx]
    vpxor ymm1, ymm1, ymm0
    vmovdqu [rcx], ymm1
    add rcx, 32
    dec r10
    jnz .loop
.tail:
    and r9, 31
    jz .done
.tail_loop:
    xor byte [rcx], r8b
    inc rcx
    dec r9
    jnz .tail_loop
.done:
    vzeroupper
    ret
%else
    ; Linux System V ABI version
%endif
```

The assembly uses AVX2 for 32-byte vectorized XOR operations.

Multi-threaded version spawns threads:

```cpp
{
  long long Sr_total = (long long)(size);
  long long Sr_threads = (long long)(4);
  long long Sr_passes = (long long)(10);
  if (Sr_threads < 1) Sr_threads = 1;
  if (Sr_passes < 1) Sr_passes = 1;
  long long Sr_chunk = Sr_total / Sr_threads;
  long long Sr_rem = Sr_total - (Sr_chunk * Sr_threads);
  std::vector<std::thread> Sr_jobs;
  Sr_jobs.reserve((size_t)Sr_threads);
  unsigned char* Sr_ptr = reinterpret_cast<unsigned char*>(buffer);
  unsigned char Sr_key = (unsigned char)(0x5A);
  for (long long Sr_t = 0; Sr_t < Sr_threads; ++Sr_t) {
    long long Sr_len = Sr_chunk + (Sr_t == Sr_threads - 1 ? Sr_rem : 0);
    unsigned char* Sr_p = Sr_ptr + Sr_t * Sr_chunk;
    Sr_jobs.emplace_back([=]() {
      for (long long Sr_i = 0; Sr_i < Sr_passes; ++Sr_i) {
        Sr_AsmXor8(Sr_p, (unsigned long long)Sr_len, Sr_key);
      }
    });
  }
  for (auto& Sr_th : Sr_jobs) Sr_th.join();
}
```

---

## Native Code Injection

Native blocks allow raw C++ injection.

### Syntax

```
native_block = "native" [ "{" ] { cpp_line } "endnative" .
```

Example:

```serenade
native {
  #include <iostream>
  struct Player {
    int hp;
    int mp;
  };
  
  Player create_player() {
    return Player{100, 50};
  }
}
endnative

let p = create_player()
```

### Rules

1. Native blocks end with `endnative`
2. You can use `}` inside native blocks (for structs, functions)
3. The parser tracks brace depth to find the real end
4. Native code is emitted directly to C++ output

### Nested Braces

```serenade
native {
  struct Config {
    int workers;
    bool enabled;
  };
  
  Config load_config() {
    if (file_exists()) {
      return parse_file();
    } else {
      return Config{1, false};
    }
  }
}
endnative
```

The parser counts braces and only exits when it sees `endnative`.

---

## Visual System

Serenade provides a minimal visual backend for Windows using GDI+.

### Initialization

```serenade
viz_init(width, height, title)
```

Example:

```serenade
viz_init(800, 600, "Serenade Visual Demo")
```

### Drawing

#### Draw Rectangle

```serenade
draw_rect(x, y, width, height, color)
```

Color format: `0xAARRGGBB` (alpha, red, green, blue).

Example:

```serenade
draw_rect(10, 10, 100, 50, 0xFF0000FF)  # blue rectangle
```

#### Draw Text

```serenade
draw_text(x, y, text, color)
```

Example:

```serenade
draw_text(10, 70, "Hello, World!", 0xFFFFFFFF)
```

### Event Loop

```serenade
viz_loop()
```

Starts the Win32 message loop. This blocks until the window is closed.

### Complete Example

```serenade
viz_init(640, 480, "Serenade Graphics")

draw_rect(50, 50, 200, 100, 0xFF3366FF)
draw_rect(270, 50, 200, 100, 0xFFFF6633)

draw_text(60, 80, "Box 1", 0xFFFFFFFF)
draw_text(280, 80, "Box 2", 0xFFFFFFFF)

viz_loop()
```

### Platform Notes

The visual system currently only works on Windows with GDI+. On other platforms, stub functions print messages:

```cpp
#ifndef _WIN32
static void Sr_VizWindowInit(int, int, const char*) {
  std::printf("[viz] Windows-only backend.\n");
}
static void Sr_VizLoop() {
  std::printf("[viz] Windows-only backend.\n");
}
#endif
```

---

## Engine Integration

Serenade integrates with an AI/ML engine runtime.

### Engine Initialization

```serenade
let engine = registry {
  mode: "local",
  workers: 4
}

engine.init(engine)
```

Transpiles to:

```cpp
auto engine = SrRegistry{};
engine.Set("mode", SrValue("local"));
engine.Set("workers", SrValue(4));

Sr_EngineInit(engine);
```

### Model Forward Pass

```serenade
let output = model.forward(input, weights)
```

Transpiles to:

```cpp
auto output = Sr_ModelForward(input, weights);
```

### Code Scanning

```serenade
let results = scan "/path/to/project" using engine
let top_files = results.top(20)
```

Implementation:

```cpp
struct SrIndexResult {
  int count = 0;
  std::vector<int> top(int k) const {
    std::vector<int> out;
    for (int i = 0; i < k && i < count; i++) {
      out.push_back(i);
    }
    return out;
  }
};

static SrIndexResult Sr_Scan(
  const std::string& path,
  SrEngineHandle engine,
  const std::string& query
) {
  SrIndexResult r;
  if (path.empty()) return r;
  const int D = 256;
  const int V = 256;
  Sr_ArenaInit(64ull * 1024ull * 1024ull);
  Sr_ArenaResetEmbeddings();
  float* emb = Sr_ArenaAllocF32(V * D);
  SrTensor Emb{ emb, V, D, D };
  if (Emb.data) Sr_EmbFill(&Emb);
  r.count = Sr_ProjectScanner(path.c_str(), Emb.data, D, 512, engine);
  return r;
}
```

---

## Build System

### Configuration File

The build system uses `serena.build.conf` files to configure compilation.

Example `serena.build.conf`:

```ini
ext.cpp=.cpp
ext.go=.go
ext.asm=.asm

build.cpp.win=cl /nologo /std:c++17 /O2 /EHsc /c "{in}" /Fo"{out}"
build.cpp.linux=g++ -std=c++17 -O2 -c "{in}" -o "{out}"

build.go.win=go build -buildmode=c-shared -o "{outdir}\{name}.dll" "{in}"
build.go.linux=go build -buildmode=c-shared -o "{outdir}/{name}.so" "{in}"

build.asm.win=nasm -f win64 "{in}" -o "{out}"
build.asm.linux=nasm -f elf64 "{in}" -o "{out}"

linker.win=link /nologo /OUT:"{out}" {objs}
linker.linux=g++ -o "{out}" {objs} -lpthread -ldl
```

### Placeholders

- `{in}` - input file path
- `{out}` - output file path
- `{outdir}` - output directory
- `{name}` - base name
- `{objs}` - object files to link
- `{exedir}` - executable directory

### Build Process

1. Parse `.serenade` file
2. Generate C++/Go/ASM segments
3. Write segments to temp directory
4. Compile each segment using configured commands
5. Link object files into executable

### Environment Variables

- `SERENA_ENGINE_CORE` - path to engine core file
- `SERENA_PROJECT_ROOT` - project root directory
- `SERENA_OUT_DIR` - output directory override
- `SERENA_OUT_NAME` - output executable name
- `SERENA_USE_WIN_BUILDER` - force Windows builder

### File Discovery

The builder searches for `serena.build.conf` in this order:

1. Input file's directory (upward search)
2. Current working directory (upward search)
3. `SERENA_PROJECT_ROOT` environment variable
4. Executable directory
5. `EXEDIR` environment variable
6. `DATADIR` environment variable

---

## Runtime Diagnostics

### SrCheckProc

The `srcheckproc()` function prints hardware and runtime diagnostics:

```serenade
srcheckproc()
```

Output:

```
[srcheckproc] workers=4 cacheLine=64 gemmTile=128
[srcheckproc] arena_bytes=67108864 arena_used=262144
[srcheckproc] build_avx2=1 build_fma=1 runtime_avx2=1
```

If AVX2 is supported but not enabled:

```
[srcheckproc] AVX2 supported by CPU but build disabled; enable /arch:AVX2 or -mavx2.
```

### Implementation

```cpp
static void Sr_CheckProc() {
  SrHardwareConfig hw{};
  Sr_GetHardwareConfig(&hw);
  std::printf("[srcheckproc] workers=%d cacheLine=%d gemmTile=%d\n",
    hw.workers, hw.cacheLine, hw.gemmTile);
  
  const size_t arenaCap = Sr_ArenaCapacity();
  const size_t arenaOff = Sr_ArenaOffset();
  std::printf("[srcheckproc] arena_bytes=%zu arena_used=%zu\n",
    arenaCap, arenaOff);
  
  if (arenaCap == 0) {
    std::printf("[srcheckproc] arena not initialized; using @f32[] will auto-init.\n");
  }
  
#if defined(__AVX2__)
  const char* buildAvx2 = "1";
#else
  const char* buildAvx2 = "0";
#endif
  
#if defined(__FMA__)
  const char* buildFma = "1";
#else
  const char* buildFma = "0";
#endif
  
  const bool runtimeAvx2 = Sr_HasAvx2();
  std::printf("[srcheckproc] build_avx2=%s build_fma=%s runtime_avx2=%s\n",
    buildAvx2, buildFma, runtimeAvx2 ? "1" : "0");
  
  if (runtimeAvx2 && buildAvx2[0] == '0') {
    std::printf("[srcheckproc] AVX2 supported by CPU but build disabled; "
                "enable /arch:AVX2 or -mavx2.\n");
  }
}
```

---

## Standard Library

### Sr_ToString

Converts values to strings:

```cpp
static std::string Sr_ToString(const SrValue& v);
static std::string Sr_ToString(const std::string& v);
static std::string Sr_ToString(const char* v);
template <typename T>
static std::string Sr_ToString(T v);
```

### Sr_Emit

Prints to stdout:

```cpp
static void Sr_Emit(const std::string& s) {
  std::printf("%s\n", s.c_str());
}
```

### Sr_Wait

Sleeps for milliseconds:

```cpp
static void Sr_Wait(double ms) {
  if (ms <= 0.0) return;
  std::this_thread::sleep_for(std::chrono::milliseconds((int)ms));
}
```

### Sr_Spawn

Async task execution:

```cpp
static std::future<void> Sr_Spawn(std::function<void()> fn) {
  return std::async(std::launch::async, std::move(fn));
}
```

### Sr_Await

Wait for future:

```cpp
static void Sr_Await(std::future<void>& f) {
  f.wait();
}
```

### Sr_Pipe

Pipe operator implementation:

```cpp
template <typename T, typename F>
static auto Sr_Pipe(T value, F fn) -> decltype(fn(value)) {
  return fn(value);
}
```

---

## Appendix

### Grammar Summary

Complete EBNF grammar for Serenade NextGen mode:

```
Program       = { Statement } .

Statement     = StructDecl | LetStmt | ConstStmt | OwnStmt | RefStmt |
                MutRefStmt | EmitStmt | IfStmt | MatchStmt | CycleStmt |
                ForStmt | ForeachStmt | WhileStmt | TaskStmt | FnStmt |
                ReturnStmt | BreakStmt | ContinueStmt | WaitStmt |
                SpawnStmt | AwaitStmt | DeferStmt | GuardStmt | ScopeStmt |
                UnsafeStmt | NativeBlock | AsmStmt | GpuStmt | DataStmt |
                SearchStmt | PromptStmt | CallStmt | ExprStmt .

# Declarations
StructDecl    = "struct" identifier "{" { Field } "}" .
Field         = identifier [ TypeAnnotation ] .
TypeAnnotation = "i32" | "i64" | "f32" | "f64" | "str" .

# Variable Declarations
LetStmt       = "let" identifier [ TypeHint ] "=" Expr .
ConstStmt     = "const" identifier "=" Expr .
OwnStmt       = "own" identifier "=" Expr .
RefStmt       = "ref" identifier "=" Expr .
MutRefStmt    = "mut" "ref" identifier "=" Expr .

TypeHint      = "option" | "result" .

# Control Flow
IfStmt        = "if" Expr Block { ElifClause } [ ElseClause ] .
ElifClause    = "elif" Expr Block .
ElseClause    = "else" Block .

MatchStmt     = "match" Expr "{" { CaseClause } "}" .
CaseClause    = "case" Pattern Block .
Pattern       = "some" "(" identifier ")" | "none" |
                "ok" "(" identifier ")" | "err" "(" identifier ")" |
                Literal .

# Loops
CycleStmt     = "cycle" ( CountCycle | RangeCycle ) .
CountCycle    = Expr [ "as" identifier ] Block .
RangeCycle    = Expr ".." Expr [ "as" identifier ] Block .

ForStmt       = "for" ( ForInit | ForRange | ForRangeStep ) Block .
ForInit       = LetStmt ";" Expr ";" Expr .
ForRange      = identifier "in" Expr ".." Expr .
ForRangeStep  = identifier "in" Expr ".." Expr "step" Expr .

ForeachStmt   = "foreach" identifier "in" Expr "count" Expr Block .

WhileStmt     = "while" Expr Block .

# Functions
TaskStmt      = "task" identifier "(" [ ParamList ] ")" Block .
FnStmt        = "fn" identifier "(" [ ParamList ] ")" [ ReturnType ] Block .
ReturnType    = "result" | "option" .
ParamList     = identifier { "," identifier } .

# Statements
ReturnStmt    = "return" Expr .
BreakStmt     = "break" .
ContinueStmt  = "continue" .
WaitStmt      = ( "wait" | "sleep" ) Expr .
SpawnStmt     = "spawn" Expr .
AwaitStmt     = "await" Expr .
DeferStmt     = "defer" Expr .
EmitStmt      = "emit" Expr .

# Safety Features
GuardStmt     = "guard" Expr .
ScopeStmt     = "scope" identifier Block .
UnsafeStmt    = "unsafe" Block .

# GPU and Data Operations
GpuStmt       = "gpu" GpuOp { Expr } .
GpuOp         = "add" | "mul" | "scale" | "matmul" | "transpose" |
                "fill" | "copy" | "zero" | "forward" | "relu" |
                "backward" | "sgd" | "bench" | "triplet" | "softmax" |
                "layernorm" | "gelu" | "attention" | "forwardfast" .

DataStmt      = Splitfile | Shuffle .
Splitfile     = "splitfile" identifier identifier "=" Expr Expr .
Shuffle       = "shuffle" Expr Expr Expr .

SearchStmt    = ( "search" | "searchx" ) { Expr } .
PromptStmt    = "prompt" identifier [ Expr ] .
CallStmt      = "call" Expr .

# Native Code
NativeBlock   = "native" [ Language ] [ "{" ] { CodeLine } "endnative" .
Language      = "cpp" | "go" | "asm" .

# Assembly
AsmStmt       = "asm" AsmOp AsmArgs .
AsmOp         = "XOR8" | ... .

# Expression Statement
ExprStmt      = Expr .

# Blocks
Block         = "{" { Statement } "}" .

# Expressions (with correct precedence)
Expr          = PipeExpr .
PipeExpr      = LogicalOrExpr { "|" LogicalOrExpr } .
LogicalOrExpr = LogicalAndExpr { "||" LogicalAndExpr } .
LogicalAndExpr= CompareExpr { "&&" CompareExpr } .
CompareExpr   = AddExpr { CompOp AddExpr } .
CompOp        = "==" | "!=" | "<" | ">" | "<=" | ">=" .
AddExpr       = MulExpr { AddOp MulExpr } .
AddOp         = "+" | "-" .
MulExpr       = UnaryExpr { MulOp UnaryExpr } .
MulOp         = "*" | "/" | "%" .
UnaryExpr     = PrimaryExpr | UnaryOp UnaryExpr .
UnaryOp       = "!" | "-" | "move" | "box" | "rc" | "arc" |
                "some" | "none" | "ok" | "err" | "try" .

PrimaryExpr   = Literal | identifier | FuncCall | MemberAccess |
                PointerAccess | ArenaAlloc | PointerIndex | SafeAccess |
                RegistryLit | StructLit | ParenExpr .

# Primary Expression Forms
FuncCall      = identifier "(" [ ExprList ] ")" .
MemberAccess  = Expr "." identifier .
PointerAccess = Expr "->" identifier .
ArenaAlloc    = "@" Type "[" Expr "]" .
Type          = "f32" | "f64" | "i32" | "i64" .
PointerIndex  = Expr "^" "[" Expr "]" .
SafeAccess    = Expr "?." identifier .
RegistryLit   = "registry" "{" [ KeyValueList ] "}" .
StructLit     = identifier "{" [ ExprList ] "}" .
ParenExpr     = "(" Expr ")" .

# Lists
KeyValueList  = KeyValue { "," KeyValue } .
KeyValue      = identifier ":" Expr .
ExprList      = Expr { "," Expr } .

# Literals
Literal       = IntLit | FloatLit | StringLit | BoolLit .
BoolLit       = "true" | "false" .
IntLit        = DecimalLit | HexLit | BinaryLit .
DecimalLit    = digit { digit } .
HexLit        = "0x" hexdigit { hexdigit } .
BinaryLit     = "0b" bindigit { bindigit } .
FloatLit      = digit { digit } "." digit { digit } [ Exponent ] .
Exponent      = ( "e" | "E" ) [ "+" | "-" ] digit { digit } .
StringLit     = '"' { character } '"' .

# Lexical Elements
identifier    = letter { letter | digit | "_" } .
letter        = "a" .. "z" | "A" .. "Z" | "_" .
digit         = "0" .. "9" .
hexdigit      = digit | "a" .. "f" | "A" .. "F" .
bindigit      = "0" | "1" .
```

### Reserved Words

Complete list of keywords that cannot be used as identifiers:

```
# Core Keywords
let const own ref mut struct fn task if elif else match case
while for foreach cycle in step as

# Control Flow
return break continue defer

# Concurrency
spawn await atomic

# Safety
guard scope unsafe some none ok err try box rc arc option result

# I/O
emit prompt

# GPU
gpu

# Native Code
native endnative asm using

# Other
registry move true false
```

### Command Keywords

The following identifiers are recognized as command statement keywords when used at the start of a line. They are NOT reserved and can be used as variable names in other contexts:

```
# File Operations
files readfile

# Data Operations
embed embed_str splitfile shuffle

# Math Operations
dot l2norm memcopy memfill randinit

# Search
search searchx

# Utility
print call
```

### GPU Sub-Commands

GPU operation keywords (used after `gpu` keyword):

```
# Basic Operations
add mul scale matmul transpose fill copy zero

# Reductions
dot sum max min norm

# Neural Network Operations
forward relu backward sgd softmax layernorm gelu
attention forwardfast bench triplet cosine
```

### Operator Precedence

From highest to lowest binding:

| Level | Operators | Associativity | Description |
|-------|-----------|---------------|-------------|
| 1 | `()` `[]` `.` `->` `?.` | Left-to-right | Call, index, member access |
| 2 | `^[·]` | Left-to-right | Pointer indexing |
| 3 | `!` `-` `move` `box` `rc` `arc` `some` `ok` `err` `try` | Right-to-left | Unary operators |
| 4 | `*` `/` `%` | Left-to-right | Multiplicative |
| 5 | `+` `-` | Left-to-right | Additive |
| 6 | `<` `>` `<=` `>=` | Left-to-right | Relational |
| 7 | `==` `!=` | Left-to-right | Equality |
| 8 | `&&` | Left-to-right | Logical AND |
| 9 | `||` | Left-to-right | Logical OR |
| 10 | `|` | Left-to-right | Pipe (NOT `>>`) |

**Important:** The pipe operator is `|` (single pipe), not `>>` (double greater-than). This was corrected from earlier documentation.

### Operator Details

#### Arithmetic Operators

```serenade
let a = 10 + 5      # Addition → 15
let b = 10 - 5      # Subtraction → 5
let c = 10 * 5      # Multiplication → 50
let d = 10 / 5      # Division → 2 (int) or 2.0 (float)
let e = 10 % 3      # Modulo → 1 (integers only!)
```

**Warning:** `%` modulo only works on integers. Using `%` on floats will cause C++ compilation errors.

#### Comparison Operators

```serenade
a == b    # Equal to
a != b    # Not equal to
a < b     # Less than
a > b     # Greater than
a <= b    # Less than or equal to
a >= b    # Greater than or equal to
```

All return boolean values (`true` or `false`).

#### Logical Operators

```serenade
a && b    # Logical AND — true if both are true
a || b    # Logical OR — true if either is true
!a        # Logical NOT — inverts boolean
```

Short-circuit evaluation: `&&` and `||` don't evaluate the right operand if the left determines the result.

#### Member Access Operators

```serenade
obj.field      # Direct member access
ptr->field     # Pointer member access (auto-dereference)
ptr?.field     # Safe member access (null-coalescing)
```

#### Special Operators

```serenade
move x         # Transfer ownership (marks x as moved)
box Foo{...}   # Heap allocation (unique_ptr)
rc Foo{...}    # Reference counted (shared_ptr)
arc Foo{...}   # Thread-safe shared pointer
some(x)        # Create option with value
none           # Create empty option
ok(x)          # Create successful result
err(msg)       # Create error result
try expr       # Unwrap result or propagate error
```

### Implementation Limits

- Identifier length: unlimited
- String length: unlimited
- Nesting depth: unlimited
- Arena size: configurable (default 64MB)
- Numeric precision: IEEE 754 double (53-bit mantissa)

### Platform Support

#### Windows

- Compiler: MSVC (Visual Studio 2019+)
- C++ Standard: C++17
- Go: 1.20+
- Assembler: NASM
- Visual system: GDI+ (native)

#### Linux

- Compiler: GCC 9+ or Clang 10+
- C++ Standard: C++17
- Go: 1.20+
- Assembler: NASM
- Visual system: not supported (stubs only)

### Version History

**v0.2-Alpha (February 2026)**
- Added `endnative` keyword for explicit native block termination
- Fixed native block brace handling
- Added `wait` and `sleep` built-ins
- Improved arena auto-initialization
- Enhanced AVX2 runtime detection
- Added assembly XOR8 multi-threading support

**v0.1-Alpha (December 2025)**
- Initial release
- Basic language features
- C++/Go/ASM transpilation
- Arena allocator
- Visual system (Windows)
- Engine integration

---

## Examples

### Hello World

```serenade
emit "Hello, World!"
```

### Variables and Arithmetic

```serenade
let a = 10
let b = 20
let sum = a + b
emit "sum: {sum}"
```

### Loops

```serenade
cycle 5 as i {
  emit "iteration {i}"
}

cycle 0..4 as n {
  emit "n={n}"
}
```

### Functions

```serenade
task factorial(n) {
  if n <= 1 {
    return 1
  }
  return n * factorial(n - 1)
}

let result = factorial(5)
emit "5! = {result}"
```

### Registries

```serenade
let config = registry {
  host: "localhost",
  port: 8080,
  timeout: 30
}

emit "connecting to {config.host}:{config.port}"
```

### Concurrency

```serenade
task worker(id) {
  emit "worker {id} started"
  wait 1000
  emit "worker {id} finished"
}

let jobs = @f32[3]
cycle 3 as i {
  jobs^[i] = spawn worker(i)
}

cycle 3 as i {
  await jobs^[i]
}

emit "all workers done"
```

### Memory Management

```serenade
let size = 1000
let buffer = @f32[size]

cycle size as i {
  buffer^[i] = i * 2.5
}

emit "buffer[100] = {buffer^[100]}"
```

### Native Integration

```serenade
native {
  #include <cmath>
  
  float compute_distance(float x1, float y1, float x2, float y2) {
    float dx = x2 - x1;
    float dy = y2 - y1;
    return std::sqrt(dx * dx + dy * dy);
  }
}
endnative

let dist = compute_distance(0, 0, 3, 4)
emit "distance: {dist}"
```

### Visual Demo

```serenade
viz_init(800, 600, "Serenade Graphics Demo")

let colors = @f32[5]
colors^[0] = 0xFFFF0000
colors^[1] = 0xFF00FF00
colors^[2] = 0xFF0000FF
colors^[3] = 0xFFFFFF00
colors^[4] = 0xFFFF00FF

cycle 5 as i {
  let x = 50 + i * 150
  draw_rect(x, 200, 100, 100, colors^[i])
  draw_text(x + 30, 230, "Box {i}", 0xFFFFFFFF)
}

viz_loop()
```

### Assembly Integration

```serenade
let data_size = 100000
let data = @f32[data_size]

cycle data_size as i {
  data^[i] = i
}

emit "encrypting data..."
asm xor8 data (data_size * 4) 0x5A 4 1000

emit "encryption complete"
srcheckproc()
```

---

## Best Practices

### Variable Naming

Use descriptive names:

```serenade
let player_health = 100
let max_speed = 10.5
let is_active = true
```

### Error Handling

Check for null/invalid values:

```serenade
let name = player?.name
if name {
  emit "player name: {name}"
}
```

### Memory Management

Initialize the arena once at program start:

```serenade
native {
  Sr_ArenaInit(128 * 1024 * 1024);
}
endnative

let weights = @f32[1000000]
```

### Concurrency

Always await spawned tasks:

```serenade
let job = spawn heavy_task()
do_other_work()
await job
```

### Native Blocks

Keep native blocks focused and documented:

```serenade
native {
  struct Point {
    float x, y;
  };
  
  Point make_point(float x, float y) {
    return Point{x, y};
  }
}
endnative
```

### Performance

Use assembly for hot paths:

```serenade
asm xor8 buffer size key threads passes
```

Use the arena for temporary allocations:

```serenade
let temp = @f32[large_size]
```

---

## File System Operations

File system operations provide built-in support for scanning directories and reading files without requiring native blocks.

### Directory scanning

A `files` statement scans a directory for files matching a glob pattern and returns the number of files found.

```
FilesScan = "files" identifier "=" "scan" StringLit StringLit .
```

The first string literal specifies the directory path. The second string literal specifies a file extension pattern. Multiple extensions are separated by `|`.

```serenade
files n = scan "C:\Scripts" "*.lua"
files count = scan "/home/user/src" "*.lua|*.luau"
```

The result variable receives the total number of matched files as an integer. File paths are stored in an internal table of up to 4096 entries. Only regular files are included; directories are skipped.

Implementation restriction: The scan is non-recursive. Only immediate children of the specified directory are matched.

### File reading

A `readfile` statement reads the contents of a previously scanned file into a byte buffer.

```
ReadFile = "readfile" identifier "=" "load" Expr Expr Expr .
```

The three expressions after `load` are: the destination buffer, the file index (0-based into the scan table), and the maximum number of bytes to read.

```serenade
let buf = @u8[32768]
readfile len = load buf 0 32768
readfile n = load buf i MAX_TOK
```

The result variable receives the number of bytes actually read. If the index is out of range or the file cannot be opened, the result is 0. A null terminator is always appended after the last byte read.

### File name lookup

A `filename` statement retrieves the base file name (without directory path) for a scanned file by index.

```
FileName = "filename" identifier Expr .
```

```serenade
filename name 5
filename f i
```

The result is a string. If the index is out of range, the result is an empty string.

---

## Memory Operations

Memory operations provide direct manipulation of arena-allocated float arrays.

### memfill

A `memfill` statement fills a float array with a constant value.

```
MemFill = "memfill" Expr Expr Expr .
```

The three expressions are: destination pointer, fill value, and element count.

```serenade
memfill B1 0.0 256
memfill weights 0.0 DIM * HIDDEN
```

### memcopy

A `memcopy` statement copies elements from one float array to another.

```
MemCopy = "memcopy" Expr Expr Expr .
```

The three expressions are: destination pointer, source pointer, and element count.

```serenade
memcopy target input 128
memcopy t x DIM
```

The source and destination must not overlap. The copy size is `count * sizeof(float)` bytes.

### randinit

A `randinit` statement initializes a float array with pseudo-random values using a linear congruential generator.

```
RandInit = "randinit" Expr Expr Expr [ Expr ] .
```

The three required expressions are: destination pointer, element count, and seed. An optional fourth expression specifies the scale factor (default 0.1).

```serenade
randinit W1 DIM * HIDDEN 137
randinit emb VOCAB * DIM 42 0.05
```

Values are generated in the range `[-scale/2, +scale/2]` using the recurrence `seed = seed * 1103515245 + 12345`. The same seed always produces the same sequence.

---

## Vector Operations

### l2norm

An `l2norm` statement L2-normalizes a float vector in place.

```
L2Norm = "l2norm" Expr Expr [ Expr ] .
```

With two arguments, the first is a pointer to the vector and the second is the element count. With three arguments, the first is a base pointer, the second is a byte offset in elements, and the third is the count.

```serenade
l2norm qv DIM
l2norm vecs i * DIM DIM
```

The two-argument form normalizes `vec[0..count-1]`. The three-argument form normalizes `vec[offset..offset+count-1]`. A small epsilon (1e-12) is added to the norm to prevent division by zero.

### dot

A `dot` statement computes the dot product of two float vectors.

```
Dot = "dot" identifier Expr Expr Expr .
```

The identifier receives the result. The three expressions are: first vector, second vector, and element count.

```serenade
dot sim qv fv 128
dot score a b DIM
```

If the identifier has not been previously declared, it is automatically declared with `auto`. Otherwise it is reassigned.

---

## Embedding Operations

### embed

An `embed` statement computes a bag-of-bytes embedding from a byte buffer.

```
Embed = "embed" Expr Expr Expr Expr Expr Expr .
```

The six arguments are: destination vector, embedding table, source byte buffer, byte count, embedding dimension, and vocabulary size.

```serenade
embed inp emb fbuf flen DIM VOCAB
```

For each byte `b` in the source buffer, the corresponding row `table[b % vocab]` is added to the output vector. The output is then L2-normalized. This produces a fixed-size vector representation of variable-length byte sequences.

### embed_str

An `embed_str` statement computes a bag-of-bytes embedding directly from a string variable.

```
EmbedStr = "embed_str" Expr Expr Expr Expr Expr .
```

The five arguments are: destination vector, embedding table, string variable, embedding dimension, and vocabulary size.

```serenade
embed_str qv emb query DIM VOCAB
```

This is equivalent to calling `embed` with the string's raw bytes and length. It exists to avoid manual string-to-buffer conversion.

---

## Data Operations

### splitfile

A `splitfile` statement splits a file buffer into two halves, returning the length of each half. This is used to create positive pairs for contrastive learning: both halves come from the same file.

```
Splitfile = "splitfile" identifier identifier "=" Expr Expr .
```

The two identifiers receive the first half length and the second half length. The two expressions are: the buffer and the total length.

```serenade
splitfile la lb = buf len
```

The first half is `la = len / 2` and the second half is `lb = len - la`. The buffer itself is not modified; only the split lengths are computed. The original buffer can then be used with `embed buf la ...` for the first half and `embed buf + la lb ...` for the second half.

### shuffle

A `shuffle` statement randomly permutes an integer array in place using the Fisher-Yates algorithm.

```
Shuffle = "shuffle" Expr Expr Expr .
```

The three arguments are: the integer array, the element count, and a seed value.

```serenade
shuffle idx fc seed
```

The operation iterates from the last element to the second, swapping each element with a randomly chosen earlier element. The seed drives a linear congruential generator. Shuffling the training indices each epoch prevents the network from memorizing the order of training examples.

---

## Search Operations

### search

A `search` statement finds the top-K most similar file vectors to a query vector using cosine similarity, and prints the results.

```
Search = "search" Expr Expr Expr Expr Expr .
```

The five arguments are: query vector, index vectors array, number of vectors, vector dimension, and K (number of results to display).

```serenade
search qv vecs fileCount DIM 10
```

Results are printed to stdout in descending order of similarity. Each line shows the rank, file name, and similarity score. The file names are resolved from the most recent `files` scan.

### searchx

A `searchx` statement performs a search with extractive explanations. For each result, it loads the file, embeds each line individually, and shows the top matching lines that explain why the file was selected.

```
Searchx = "searchx" Expr Expr Expr Expr Expr Expr Expr Expr .
```

The eight arguments are: query vector, index vectors array, number of vectors, vector dimension, K (number of results), embedding table, vocabulary size, and number of explanation lines per result.

```serenade
searchx qv vecs fc DIM 10 emb VOCAB 3
```

For each of the top-K results, the operation:
1. Prints the rank, file name, and similarity score (like `search`).
2. Loads the file and splits it into individual lines.
3. Computes a bag-of-bytes embedding for each line (using the same embedding table).
4. L2-normalizes each line embedding and computes dot product with the query vector.
5. Prints the top N matching lines with their line numbers and similarity scores.

Lines shorter than 5 characters and comment lines (starting with `--`) are skipped. Output lines longer than 80 characters are truncated.

---

## Interactive Input

### prompt

A `prompt` statement reads a line of text from standard input, optionally displaying a prompt string.

```
Prompt = "prompt" identifier [ Expr ] .
```

```serenade
prompt query "> "
prompt line
```

The identifier receives the input as a string. Leading and trailing newlines are stripped. If the optional expression is provided, it is printed before waiting for input.

---

## Function Definitions

### fn

An `fn` statement defines a named function as a closure.

```
FnDecl = "fn" identifier "(" [ ParamList ] ")" Block .
ParamList = identifier { "," identifier } .
```

```serenade
fn train(lr, epochs) {
    cycle epochs as ep {
        emit "epoch {ep}"
    }
}

fn add(a, b) {
    return a + b
}
```

Parameters use type inference; their types are deduced from the call site. The function captures all variables from the enclosing scope by reference.

An `fn` declaration is equivalent to a `task` declaration. Both produce C++ lambdas. The `fn` keyword is preferred for general-purpose functions; `task` is retained for backward compatibility.

### call

A `call` statement explicitly invokes a function.

```
Call = "call" Expr .
```

```serenade
call train(0.01, 3)
call process(data)
```

The `call` keyword is optional. A bare function call expression is also valid as a statement. `call` exists for clarity.

---

## GPU and CUDA Operations

Serenade provides comprehensive GPU computing support through CUDA. All GPU operations automatically handle device memory management and kernel launches.

### CUDA Availability Check

Check if CUDA is available on the system:

```serenade
if cuda_available() {
  emit "CUDA detected — GPU acceleration enabled"
} else {
  emit "No CUDA — falling back to CPU"
}
```

**Transpiles to:**
```cpp
bool Sr_CudaAvailable() {
  int deviceCount = 0;
  cudaError_t err = cudaGetDeviceCount(&deviceCount);
  return (err == cudaSuccess && deviceCount > 0);
}
```

### GPU Memory Management

GPU operations use automatic device memory management:

```serenade
# Allocate arrays (on host)
let a = @f32[1024]
let b = @f32[1024]
let c = @f32[1024]

# Initialize data
cycle 1024 as i {
  a^[i] = i
  b^[i] = i * 2.0
}

# GPU operations automatically:
# 1. Allocate device memory
# 2. Copy host → device
# 3. Launch kernel
# 4. Copy device → host
# 5. Free device memory

gpu add c a b 1024
```

### Basic GPU Operations

#### gpu add — Vector Addition

```serenade
gpu add c a b n
```

Computes `c[i] = a[i] + b[i]` for all i ∈ [0, n).

**Parameters:**
- `c` — output vector
- `a` — first input vector
- `b` — second input vector
- `n` — element count

**CUDA Kernel:**
```cpp
__global__ void Sr_GpuAdd_Kernel(float* c, const float* a, const float* b, int n) {
  int i = blockIdx.x * blockDim.x + threadIdx.x;
  if (i < n) {
    c[i] = a[i] + b[i];
  }
}
```

#### gpu mul — Vector Multiplication

```serenade
gpu mul c a b n
```

Computes `c[i] = a[i] * b[i]` for all i ∈ [0, n).

#### gpu scale — Scalar Multiplication

```serenade
gpu scale a scalar n
```

Computes `a[i] = a[i] * scalar` for all i ∈ [0, n).

**Parameters:**
- `a` — vector (modified in place)
- `scalar` — scalar value to multiply by
- `n` — element count

#### gpu dot — Dot Product

```serenade
let result = gpu_dot(a, b, n)
```

Computes `result = Σ(a[i] * b[i])` for i ∈ [0, n).

Returns a scalar value. Uses parallel reduction on GPU.

#### gpu norm — L2 Norm

```serenade
let magnitude = gpu_norm(vec, n)
```

Computes `magnitude = sqrt(Σ(vec[i]²))`.

### GPU Matrix Operations

#### gpu matmul — Matrix Multiplication

```serenade
gpu matmul C A B M N K
```

Computes `C = A × B` where:
- `A` is M × K
- `B` is K × N
- `C` is M × N

**Parameters:**
- `C` — output matrix (M × N)
- `A` — left matrix (M × K)
- `B` — right matrix (K × N)
- `M` — number of rows in A
- `N` — number of columns in B
- `K` — number of columns in A / rows in B

**Example:**
```serenade
const M = 512
const N = 512
const K = 512

let A = @f32[M * K]
let B = @f32[K * N]
let C = @f32[M * N]

# Initialize A and B...

gpu matmul C A B M N K
```

Uses tiled matrix multiplication with shared memory for optimal performance.

#### gpu transpose — Matrix Transpose

```serenade
gpu transpose B A rows cols
```

Computes `B = A^T` where A is rows × cols.

**Parameters:**
- `B` — output transposed matrix (cols × rows)
- `A` — input matrix (rows × cols)
- `rows` — number of rows in A
- `cols` — number of columns in A

### GPU Utility Functions

#### gpu fill — Fill Array

```serenade
gpu fill arr value n
```

Sets all elements to a value: `arr[i] = value`.

#### gpu copy — Array Copy

```serenade
gpu copy dest src n
```

Copies `n` elements from `src` to `dest`.

Uses `cudaMemcpy` for efficient device-to-device transfer.

#### gpu zero — Zero Array

```serenade
gpu zero arr n
```

Sets all elements to zero: `arr[i] = 0.0`.

Equivalent to `gpu fill arr 0.0 n` but optimized.

### GPU Reduction Operations

#### gpu sum — Array Sum

```serenade
let total = gpu_sum(arr, n)
```

Computes `total = Σ(arr[i])`.

Uses tree reduction with shared memory.

#### gpu max — Maximum Element

```serenade
let max_val = gpu_max(arr, n)
```

Finds the maximum value in the array.

#### gpu min — Minimum Element

```serenade
let min_val = gpu_min(arr, n)
```

Finds the minimum value in the array.

### GPU Performance Utilities

#### GPU Synchronization

Force GPU to complete all pending operations:

```serenade
gpu_sync()
```

**Transpiles to:**
```cpp
cudaDeviceSynchronize();
```

Use sparingly — GPU operations auto-sync before host reads.

#### GPU Timer

Measure kernel execution time:

```serenade
let start = gpu_timer_start()

# ... GPU operations

let elapsed = gpu_timer_stop(start)
emit "GPU time: {elapsed} ms"
```

**Uses CUDA events for accurate timing:**
```cpp
cudaEvent_t start, stop;
cudaEventCreate(&start);
cudaEventCreate(&stop);
cudaEventRecord(start);
// ... kernels
cudaEventRecord(stop);
cudaEventSynchronize(stop);
float ms = 0;
cudaEventElapsedTime(&ms, start, stop);
```

### GPU Best Practices

```serenade
# ✓ GOOD: Batch operations to minimize host-device transfers
let a = @f32[1000000]
let b = @f32[1000000]
let c = @f32[1000000]

gpu add c a b 1000000      # Single transfer
gpu scale c 2.0 1000000    # Operates on GPU
gpu norm c 1000000         # Result transferred back

# ✗ BAD: Frequent small transfers
cycle 1000 {
  gpu add c a b 1000  # 1000 separate kernel launches!
}

# ✓ GOOD: Check CUDA availability
if cuda_available() {
  gpu matmul C A B M N K
} else {
  # CPU fallback
  cycle M as i {
    cycle N as j {
      let sum = 0.0
      cycle K as k {
        sum = sum + A^[i*K + k] * B^[k*N + j]
      }
      C^[i*N + j] = sum
    }
  }
}

# ✓ GOOD: Use scoped arena for temporary GPU buffers
scope gpu_compute {
  let temp1 = @f32[N]
  let temp2 = @f32[N]

  gpu add temp1 a b N
  gpu mul temp2 temp1 c N
  gpu copy result temp2 N
}
# temp1, temp2 freed here
```

### GPU Error Handling

All GPU operations check for CUDA errors and abort with diagnostic messages:

```cpp
cudaError_t err = cudaGetLastError();
if (err != cudaSuccess) {
  fprintf(stderr, "CUDA error: %s\n", cudaGetErrorString(err));
  exit(1);
}
```

To add custom error handling:

```serenade
unsafe {
  native cpp {
    cudaError_t err = cudaGetLastError();
    if (err != cudaSuccess) {
      // Custom handling
    }
  }
}
```

---

## GPU Neural Network Operations

GPU neural network operations extend the base GPU command set with operations for training and inference of neural networks. All operations require CUDA and follow the same host-device transfer pattern as other GPU commands.

### gpu forward

A `gpu forward` command performs a linear layer forward pass: `output = input × weights + bias`.

```
GpuForward = "gpu" "forward" Expr Expr Expr Expr Expr Expr Expr .
```

The seven arguments are: output, input, weights, bias, batch size M, input dimension, output dimension.

```serenade
gpu forward hid x w1 b1 1 DIM HIDDEN
gpu forward y h w2 b2 1 HIDDEN DIM
```

The operation performs matrix multiplication of input (M × inDim) by weights (inDim × outDim), then adds the bias vector (outDim) to each row of the result.

### gpu relu

A `gpu relu` command applies the ReLU activation function in place.

```
GpuRelu = "gpu" "relu" Expr Expr .
```

The two arguments are: the vector and its element count.

```serenade
gpu relu h HIDDEN
```

Each element `x` is replaced with `max(0, x)`.

### gpu backward

A `gpu backward` command performs a linear layer backward pass, computing gradients for weights, bias, and input.

```
GpuBackward = "gpu" "backward" Expr Expr Expr Expr Expr Expr Expr Expr Expr .
```

The nine arguments are: input, weights, gradOutput, gradWeights, gradBias, gradInput, batch size M, input dimension, output dimension.

```serenade
gpu backward h w2 go gw2 gb2 gh 1 HIDDEN DIM
gpu backward x w1 gh gw1 gb1 gx 1 DIM HIDDEN
```

The operation computes:
- `gradWeights = input^T × gradOutput`
- `gradBias = column_sum(gradOutput)`
- `gradInput = gradOutput × weights^T`

### gpu sgd

A `gpu sgd` command performs a stochastic gradient descent weight update.

```
GpuSgd = "gpu" "sgd" Expr Expr Expr Expr .
```

The four arguments are: weights, gradients, learning rate, and element count.

```serenade
gpu sgd w1 gw1 0.01 DIM * HIDDEN
gpu sgd b1 gb1 0.01 HIDDEN
```

Each weight is updated as `w[i] = w[i] - lr * grad[i]`.

### gpu bench

A `gpu bench` command runs a timed GPU matrix multiplication benchmark.

```
GpuBench = "gpu" "bench" Expr Expr Expr Expr Expr Expr .
GpuBench = "gpu" "bench" identifier Expr Expr Expr Expr Expr Expr .
```

With six arguments: output matrix C, input matrix A, input matrix B, M, N, K. With seven arguments, the first is a variable that receives the GFLOPS result.

```serenade
gpu bench c a b N N N
gpu bench gflops c a b 8192 8192 8192
```

The benchmark performs a warmup run followed by 10 timed runs using CUDA events. It prints the average time and GFLOPS to stdout.

### gpu triplet

A `gpu triplet` command computes the triplet loss and fills gradient buffers for contrastive learning.

```
GpuTriplet = "gpu" "triplet" Expr Expr Expr Expr Expr Expr Expr Expr .
```

The eight arguments are: anchor vector, positive vector, negative vector, anchor gradient output, positive gradient output, negative gradient output, margin, and vector dimension.

```serenade
gpu triplet xa xp xn ga gp gn 0.3 DIM
```

The operation computes `loss = max(0, dist(anchor, positive) - dist(anchor, negative) + margin)` where `dist` is L2 distance. When loss > 0, gradient buffers are filled with partial derivatives. When loss ≤ 0 (constraint already satisfied), all gradient buffers are zeroed. Uses shared memory reduction on the GPU for efficient per-dimension squared difference accumulation.

### gpu softmax

A `gpu softmax` command applies the softmax function to a vector in place.

```
GpuSoftmax = "gpu" "softmax" Expr Expr .
```

The two arguments are: the vector and its element count.

```serenade
gpu softmax scores seqLen
```

The operation uses a numerically stable three-pass algorithm: (1) find the maximum value, (2) compute `exp(x[i] - max)` and sum, (3) divide each element by the sum. After the operation, all elements are in the range (0, 1) and sum to 1.

### gpu layernorm

A `gpu layernorm` command applies layer normalization to a vector in place.

```
GpuLayerNorm = "gpu" "layernorm" Expr Expr Expr Expr .
```

The four arguments are: the vector, gamma (scale) parameters, beta (shift) parameters, and element count.

```serenade
gpu layernorm h gamma beta HIDDEN
```

The operation computes `x[i] = gamma[i] * (x[i] - mean) / sqrt(variance + epsilon) + beta[i]`. Mean and variance are computed across all elements of the vector. Epsilon is 1e-5 for numerical stability.

### gpu gelu

A `gpu gelu` command applies the GELU (Gaussian Error Linear Unit) activation function in place.

```
GpuGelu = "gpu" "gelu" Expr Expr .
```

The two arguments are: the vector and its element count.

```serenade
gpu gelu a1 H1
```

Each element is replaced with `0.5 * x * (1 + tanh(sqrt(2/π) * (x + 0.044715 * x³)))`. GELU is a smooth approximation of ReLU commonly used in transformer architectures.

### gpu attention

A `gpu attention` command performs single-head scaled dot-product attention.

```
GpuAttention = "gpu" "attention" Expr Expr Expr Expr Expr Expr .
```

The six arguments are: query matrix Q, key matrix K, value matrix V, output matrix, sequence length, and head dimension.

```serenade
gpu attention Q K V out seqLen dim
```

The operation computes `Attention(Q, K, V) = softmax(Q · K^T / sqrt(dim)) · V`. Internally it reuses the existing GEMM kernel for matrix multiplications and the softmax kernel for normalization. Temporary score and weight matrices are allocated and freed automatically.

### gpu forwardfast

A `gpu forwardfast` command performs a Blackwell-optimized linear layer forward pass using CUDA streams and L2 cache hints.

```
GpuForwardFast = "gpu" "forwardfast" Expr Expr Expr Expr Expr Expr Expr .
```

The seven arguments are: output, input, weights, bias, batch size M, input dimension, output dimension.

```serenade
gpu forwardfast out x w1 b1 1 DIM H1
```

This is functionally equivalent to `gpu forward` but uses asynchronous CUDA streams for overlapping host-device transfers with computation. On sm_80+ architectures (Ampere, Hopper, Blackwell), it additionally sets L2 cache access policy hints for the weight matrix to maximize cache residency.

---


## Complete Example: Code Search Engine (v1 — Autoencoder)

The following program trains a neural network autoencoder on Lua source files and provides interactive code search.

```serenade
const VOCAB = 256
const DIM = 128
const HIDDEN = 256
const MAX_TOK = 32768
const EPOCHS = 3

let emb = @f32[VOCAB * DIM]
let w1 = @f32[DIM * HIDDEN]
let b1 = @f32[HIDDEN]
let w2 = @f32[HIDDEN * DIM]
let b2 = @f32[DIM]
let x = @f32[DIM]
let h = @f32[HIDDEN]
let y = @f32[DIM]
let t = @f32[DIM]
let go = @f32[DIM]
let gw2 = @f32[HIDDEN * DIM]
let gb2 = @f32[DIM]
let gh = @f32[HIDDEN]
let gw1 = @f32[DIM * HIDDEN]
let gb1 = @f32[HIDDEN]
let gx = @f32[DIM]
let vecs = @f32[4096 * DIM]
let qv = @f32[DIM]
let buf = @u8[MAX_TOK]

randinit emb VOCAB * DIM 42
randinit w1 DIM * HIDDEN 137
randinit w2 HIDDEN * DIM 271
memfill b1 0.0 HIDDEN
memfill b2 0.0 DIM

files fc = scan "C:\Scripts" "*.lua|*.luau"

cycle EPOCHS as ep {
    cycle fc as i {
        readfile len = load buf i MAX_TOK
        if len > 10 {
            embed x emb buf len DIM VOCAB
            memcopy t x DIM
            gpu forward h x w1 b1 1 DIM HIDDEN
            gpu relu h HIDDEN
            gpu forward y h w2 b2 1 HIDDEN DIM
            gpu backward h w2 go gw2 gb2 gh 1 HIDDEN DIM
            gpu backward x w1 gh gw1 gb1 gx 1 DIM HIDDEN
            gpu sgd w1 gw1 0.01 DIM * HIDDEN
            gpu sgd b1 gb1 0.01 HIDDEN
            gpu sgd w2 gw2 0.01 HIDDEN * DIM
            gpu sgd b2 gb2 0.01 DIM
        }
    }
}

cycle fc as i {
    readfile len = load buf i MAX_TOK
    if len > 10 {
        embed x emb buf len DIM VOCAB
        gpu forward h x w1 b1 1 DIM HIDDEN
        gpu relu h HIDDEN
        gpu forward y h w2 b2 1 HIDDEN DIM
        cycle DIM as d {
            vecs^[i * DIM + d] = y^[d]
        }
    }
}
cycle fc as i {
    l2norm vecs i * DIM DIM
}

while 1 == 1 {
    prompt q "> "
    if q == "exit" {
        break
    }
    embed_str qv emb q DIM VOCAB
    gpu forward h qv w1 b1 1 DIM HIDDEN
    gpu relu h HIDDEN
    gpu forward qv h w2 b2 1 HIDDEN DIM
    l2norm qv DIM
    search qv vecs fc DIM 10
}
```

---

## Complete Example: Contrastive Code Search (v2 — Triplet Loss)

The following program uses contrastive learning with triplet loss to train a 3-layer encoder. Instead of reconstructing the input (autoencoder), it learns to place similar files close together and dissimilar files far apart in embedding space. Each result includes extractive explanations showing the most relevant lines.

```serenade
# SerenaAI v2 — contrastive code search
const VOCAB = 256
const DIM = 256
const H1 = 512
const H2 = 512
const MAX_TOK = 32768
const EPOCHS = 5
const MARGIN = 0.3
const LR = 0.001

# 3-layer encoder: DIM -> H1 -> H2 -> DIM
let w1 = @f32[DIM * H1]
let b1 = @f32[H1]
let w2 = @f32[H1 * H2]
let b2 = @f32[H2]
let w3 = @f32[H2 * DIM]
let b3 = @f32[DIM]
let emb = @f32[VOCAB * DIM]

# gradient buffers
let gw1 = @f32[DIM * H1]
let gb1 = @f32[H1]
let gx1 = @f32[DIM]
let gw2 = @f32[H1 * H2]
let gb2 = @f32[H2]
let gx2 = @f32[H1]
let gw3 = @f32[H2 * DIM]
let gb3 = @f32[DIM]
let gx3 = @f32[H2]

# activations, triplet grads, file buffers
let a1 = @f32[H1]
let a2 = @f32[H2]
let out = @f32[DIM]
let ga = @f32[DIM]
let gp = @f32[DIM]
let gn = @f32[DIM]
let buf = @u8[MAX_TOK]
let x = @f32[DIM]
let xa = @f32[DIM]
let xp = @f32[DIM]
let xn = @f32[DIM]
let vecs = @f32[4096 * DIM]
let qv = @f32[DIM]

# initialize weights and biases
randinit emb VOCAB * DIM 42 0.05
randinit w1 DIM * H1 137 0.05
randinit w2 H1 * H2 271 0.05
randinit w3 H2 * DIM 397 0.05
memfill b1 0.0 H1
memfill b2 0.0 H2
memfill b3 0.0 DIM

files fc = scan "C:\Scripts" "*.lua|*.luau"

# shuffle index array
let idx = @i32[4096]
cycle fc as i {
    idx^[i] = i
}

# encoder function: buf -> DIM vector
fn encode(dst, rbuf, rlen) {
    embed x emb rbuf rlen DIM VOCAB
    gpu forward a1 x w1 b1 1 DIM H1
    gpu gelu a1 H1
    gpu forward a2 a1 w2 b2 1 H1 H2
    gpu gelu a2 H2
    gpu forward dst a2 w3 b3 1 H2 DIM
    l2norm dst DIM
}
let seed = 7919
# training with triplet loss
cycle EPOCHS as ep {
    shuffle idx fc seed
    seed = seed + 1
    let total = 0.0
    let n = 0
    cycle fc as ii {
        let i = idx^[ii]
        readfile len = load buf i MAX_TOK
        if len > 20 {
            # anchor = first half of file
            splitfile la lb = buf len
            embed x emb buf la DIM VOCAB
            gpu forward a1 x w1 b1 1 DIM H1
            gpu gelu a1 H1
            gpu forward a2 a1 w2 b2 1 H1 H2
            gpu gelu a2 H2
            gpu forward out a2 w3 b3 1 H2 DIM
            l2norm out DIM
            memcopy xa out DIM

            # positive = second half of same file
            embed x emb buf + la lb DIM VOCAB
            gpu forward a1 x w1 b1 1 DIM H1
            gpu gelu a1 H1
            gpu forward a2 a1 w2 b2 1 H1 H2
            gpu gelu a2 H2
            gpu forward out a2 w3 b3 1 H2 DIM
            l2norm out DIM
            memcopy xp out DIM

            # negative = random different file
            let ni = i
            while ni == i {
                seed = seed * 1103515245 + 12345
                ni = ((seed / 65536) & 32767) % fc
            }
            readfile nlen = load buf ni MAX_TOK
            if nlen > 10 {
                embed x emb buf nlen DIM VOCAB
                gpu forward a1 x w1 b1 1 DIM H1
                gpu gelu a1 H1
                gpu forward a2 a1 w2 b2 1 H1 H2
                gpu gelu a2 H2
                gpu forward out a2 w3 b3 1 H2 DIM
                l2norm out DIM
                memcopy xn out DIM

                # compute triplet loss and gradients
                gpu triplet xa xp xn ga gp gn MARGIN DIM

                # recompute anchor activations for backprop
                readfile rlen = load buf i MAX_TOK
                splitfile la2 lb2 = buf rlen
                embed x emb buf la2 DIM VOCAB
                gpu forward a1 x w1 b1 1 DIM H1
                gpu gelu a1 H1
                gpu forward a2 a1 w2 b2 1 H1 H2
                gpu gelu a2 H2

                # backprop through all 3 layers
                gpu backward a2 w3 ga gw3 gb3 gx3 1 H2 DIM
                gpu backward a1 w2 gx3 gw2 gb2 gx2 1 H1 H2
                gpu backward x w1 gx2 gw1 gb1 gx1 1 DIM H1

                # update weights
                gpu sgd w1 gw1 LR DIM * H1
                gpu sgd b1 gb1 LR H1
                gpu sgd w2 gw2 LR H1 * H2
                gpu sgd b2 gb2 LR H2
                gpu sgd w3 gw3 LR H2 * DIM
                gpu sgd b3 gb3 LR DIM
            }
        }
    }
}
# index all files
cycle fc as i {
    readfile len = load buf i MAX_TOK
    if len > 10 {
        embed x emb buf len DIM VOCAB
        gpu forward a1 x w1 b1 1 DIM H1
        gpu gelu a1 H1
        gpu forward a2 a1 w2 b2 1 H1 H2
        gpu gelu a2 H2
        gpu forward out a2 w3 b3 1 H2 DIM
        memcopy vecs + i * DIM out DIM
    }
}
cycle fc as i {
    l2norm vecs i * DIM DIM
}
# interactive search with explanations
while 1 == 1 {
    prompt q "> "
    if q == "exit" {
        break
    }
    embed_str qv emb q DIM VOCAB
    gpu forward a1 qv w1 b1 1 DIM H1
    gpu gelu a1 H1
    gpu forward a2 a1 w2 b2 1 H1 H2
    gpu gelu a2 H2
    gpu forward qv a2 w3 b3 1 H2 DIM
    l2norm qv DIM
    searchx qv vecs fc DIM 10 emb VOCAB 3
}
```

---

## Advanced Examples

This section demonstrates real-world applications combining Serenade's features.

### Example 1: Matrix Multiply

the classic implementation of matrix multiplication through three nested loops

```serenade
const DIM = 4
let mat_a = @f32[DIM * DIM]
let mat_b = @f32[DIM * DIM]
let mat_c = @f32[DIM * DIM]
cycle DIM as r {
    cycle DIM as c {
        if r == c {
            mat_a^[r * DIM + c] = 2.0
        } else {
            mat_a^[r * DIM + c] = 0.0
        }
    }
}
# fill B with 1..16
let counter = 1.0
cycle DIM as r {
    cycle DIM as c {
        mat_b^[r * DIM + c] = counter
        counter = counter + 1.0
    }
}
# C = A * B
cycle DIM as r {
    cycle DIM as c {
        let sum = 0.0
        cycle DIM as k {
            sum = sum + mat_a^[r * DIM + k] * mat_b^[k * DIM + c]
        }
        mat_c^[r * DIM + c] = sum
    }
}
emit "A = 2*I, B = 1..16"
emit "C = A * B:"
cycle DIM as r {
    let c0 = mat_c^[r * DIM + 0]
    let c1 = mat_c^[r * DIM + 1]
    let c2 = mat_c^[r * DIM + 2]
    let c3 = mat_c^[r * DIM + 3]
    emit "  [{c0}, {c1}, {c2}, {c3}]"
}
```

### Example 2: Stack

push/pop/peek on an array with an sp pointer

```serenade
const cap = 64
let stack = @i32[cap]
let sp = 0
fn stack_push(val) {
    if sp >= cap {
        emit "stack overflow"
        return
    }
    stack^[sp] = val
    sp = sp + 1
}
fn stack_pop() {
    if sp <= 0 {
        emit "stack underflow!"
        return -1
    }
    sp = sp - 1
    return stack^[sp]
}
fn stack_peek() {
    if sp <= 0 {
        return -1
    }
    return stack^[sp - 1]
}
stack_push(10)
stack_push(20)
stack_push(30)
emit "pushed 10, 20, 30"
let top = stack_peek()
emit "peek: {top}"
let a = stack_pop()
let b = stack_pop()
let c = stack_pop()
emit "popped: {a}, {b}, {c}"
emit "stack size: {sp}"
```

### Example 3: Binary Search

Search in a sorted array, hit + miss

```serenade
fn bin_search(data, size, target) {
    let lo = 0
    let hi = size - 1
    while lo <= hi {
        let mid = lo + (hi - lo) / 2
        let val = data^[mid]
        if val == target {
            return mid
        }
        if val < target {
            lo = mid + 1
        } else {
            hi = mid - 1
        }
    }
    return -1
}
let mid_val = arr^[10]
let found = bin_search(arr, N, mid_val)
emit "search for {mid_val}: index={found}"
let first_val = arr^[0]
let found2 = bin_search(arr, N, first_val)
emit "search for {first_val}: index={found2}"
let miss = bin_search(arr, N, 999)
emit "search for 999: index={miss}"
```

### Example 4: GCD / LCM 

Euclidean algorithm using while + %

```serenade
fn gcd(a, b) {
    while b != 0 {
        let t = b
        b = a % b
        a = t
    }
    return a
}
fn lcm(a, b) {
    let g = gcd(a, b)
    return a / g * b
}
let g1 = gcd(48, 18)
emit "gcd(48, 18) = {g1}"
let g2 = gcd(1071, 462)
emit "gcd(1071, 462) = {g2}"
let l1 = lcm(12, 8)
emit "lcm(12, 8) = {l1}"
```

### Example 5: Selection Sort + Reverse

sort by selection, then rotate in place

```
const M = 10
let data = @i32[M]
seed = 42
cycle M as i {
    seed = (seed * 1103515245 + 12345) % 2147483647
    data^[i] = seed % 50
}
# selection sort
cycle M as i {
    let min_idx = i
    let j = i + 1
    while j < M {
        if data^[j] < data^[min_idx] {
            min_idx = j
        }
        j = j + 1
    }
    if min_idx != i {
        let tmp = data^[i]
        data^[i] = data^[min_idx]
        data^[min_idx] = tmp
    }
}
emit "sorted:"
cycle M as i {
    let v = data^[i]
    emit "  {v}"
}
let lo_r = 0
let hi_r = M - 1
while lo_r < hi_r {
    let tmp = data^[lo_r]
    data^[lo_r] = data^[hi_r]
    data^[hi_r] = tmp
    lo_r = lo_r + 1
    hi_r = hi_r - 1
}
emit "reversed:"
cycle M as i {
    let v = data^[i]
    emit "  {v}"
}

```

### Example 6: Linked List (array-based)

linked list via two arrays (val + next)

```serenade
const listcap = 32
let list_val = @i32[listcap]
let list_nxt = @i32[listcap]
let list_len = 0
let head = -1
fn list_push(val) {
    let idx = list_len
    list_val^[idx] = val
    list_nxt^[idx] = head
    head = idx
    list_len = list_len + 1
}
fn list_print() {
    let cur = head
    let count = 0
    while cur != -1 {
        let v = list_val^[cur]
        emit "  node[{count}] = {v}"
        cur = list_nxt^[cur]
        count = count + 1
    }
}
list_push(100)
list_push(200)
list_push(300)
list_push(400)
emit "list:"
list_print()
```

### Key Takeaways from Examples

1. **Ownership & Borrowing**: Prevent use-after-free and data races at compile time
2. **Option & Result**: Elegant error handling without exceptions
3. **Defer**: Guaranteed cleanup, even with early returns
4. **Scope**: Automatic arena memory management
5. **Guards**: Runtime safety checks with clear error messages
6. **GPU + Safety**: High performance with memory safety guarantees
7. **Arc & Atomics**: Safe concurrent programming

These patterns enable writing robust systems code with the safety of Rust and the simplicity of Python.

---

## Frequently Asked Questions

**Q: Why no semicolons?**

A: Serenade is line-based. One statement per line makes the code cleaner and easier to read.

**Q: Can I use C++ features directly?**

A: Yes, through `native` blocks you have full C++ access.

**Q: Is Serenade interpreted or compiled?**

A: Compiled. Serenade transpiles to C++/Go/ASM and compiles to native code.

**Q: What's the performance compared to C++?**

A: Generated C++ code is as fast as hand-written C++. Assembly paths can exceed C++ performance.

**Q: Why is the visual system Windows-only?**

A: Current implementation uses GDI+ and OpenGL Compatible Profile. Linux support is planned using X11 or SDL.

**Q: How do I debug Serenade code?**

A: Examine the generated C++ code in the temp build directory. Use `srcheckproc()` for runtime diagnostics and `sroprstart()` & `srcheckopr()` for check time of needed operation

**Q: Can I use Serenade in production?**

A: Serenade is alpha software. Use for prototyping and experimentation.

**Q: How do I contribute?**

A: See the project repository for contribution guidelines.

**Q: What's the license?**

A: See LICENSE file in the repository.

**Q: Are the Rust-inspired safety features mandatory?**

A: No. You can write code without using `own`, `ref`, `box`, `option`, or `result`. The safety features are opt-in. Use them where safety matters, and skip them for prototyping or performance-critical code.

**Q: What's the performance cost of the safety features?**

A: Minimal. `own`/`move` are compile-time only (zero cost). `ref`/`mut ref` add a single integer increment/decrement. `option`/`result` use the same layout as `std::optional`. Smart pointers (`box`/`rc`/`arc`) have standard C++ overhead.

**Q: Can I mix safe and unsafe code?**

A: Yes. Use `unsafe { }` blocks to disable safety checks in performance-critical sections. The rest of your code remains safe.

**Q: How do I convert existing Serenade code to use safety features?**

A: Incrementally. Start by adding `own` to heap allocations, then `ref` for read-only access. Add `option`/`result` return types to functions that can fail. Finally, add `guard` assertions for runtime checks.

**Q: Does the borrow checker work like Rust's?**

A: Similar but simpler. Serenade's borrow checker is **transpile-time** only, checking for conflicts when generating C++. It's not as sophisticated as Rust's lifetime system, but catches common errors like double mutable borrows.

**Q: Can I use `own` with GPU arrays?**

A: Yes. `own data = @f32[1024]` ensures the array is tracked for move semantics. Useful when passing GPU buffers between functions.

**Q: What happens if I try to use a moved variable?**

A: **Compile error**. The transpiler emits `#error "use of moved variable 'x'"` which causes the C++ compiler to fail with a clear message.

**Q: Can I return `option` or `result` from tasks?**

A: Yes. Use `task name() option { }` or `fn name() result { }`. The return type annotation tells the transpiler to generate the appropriate template signature.

**Q: How do I debug borrow checker errors?**

A: The transpiler emits `#error` directives with the exact line and variable name. Examine the generated C++ file to see where the conflict occurs. Common fixes: reduce borrow scope, use `ref` instead of `mut ref`, or restructure code to avoid overlapping borrows.

**Q: Is the pipe operator `|` or `>>`?**

A: The pipe operator is `|` (single vertical bar), not `>>`. This was corrected in the latest grammar specification.

**Q: Can I use `defer` with GPU operations?**

A: Yes. `defer gpu_free(buffer)` ensures GPU memory is freed on scope exit. Very useful for exception safety in GPU code.

**Q: Do guards abort or throw exceptions?**

A: **Abort**. `guard expr` calls `abort()` on failure, printing the failed expression and line number. No exceptions — this is a C-style safety check for critical assertions.

**Q: Can `scope` blocks be nested?**

A: Yes. Each `scope` saves and restores the arena independently. Inner scopes free their memory first, then outer scopes.

---

## Acknowledgments

Serenade draws inspiration from:

- **Go** - simplicity and tooling
- **Rust** - memory safety concepts
- **Zig** - compile-time execution
- **C++** - performance and control

Special thanks to the open-source community for tools and libraries that make Serenade possible.

---

*End of Serenade Language Specification v0.3-Alpha*

---

## Changelog

### v0.3-Alpha (2026-02-17)

**Major additions:**
- Complete Rust-inspired memory safety system (ownership, borrowing, smart pointers)
- Option and Result types with pattern matching
- Control flow improvements (elif/else, match/case, for/foreach/while)
- Comprehensive struct system with type annotations
- Defer statements for RAII resource management
- Const declarations
- Scope blocks for arena management
- Guard runtime assertions
- Unsafe blocks for performance-critical code
- Expanded GPU operations documentation
- Complete grammar specification
- 150+ new built-in functions documented
- 6 advanced real-world examples

**Total lines:** 5953 (↑2995 from v0.2)

### v0.2-Alpha
- Initial public documentation release
- Basic language features, GPU operations, OpenGL integration
