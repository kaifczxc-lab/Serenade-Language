# The Serenade Programming Language Specification

Version 0.2-Alpha | February 2026

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

Conditional execution:

```
if_stmt = "if" expression block .
block   = "{" { statement } "}" .
```

Example:

```serenade
if hp < 50 {
  emit "low health"
}
```

No `else` or `elif` is currently supported. Use multiple `if` statements:

```serenade
if hp < 30 {
  emit "critical"
}
if hp >= 30 && hp < 70 {
  emit "damaged"
}
if hp >= 70 {
  emit "healthy"
}
```

### Cycle Statements

Loops in Serenade use the `cycle` keyword.

#### Count Loop

```
cycle_stmt = "cycle" expression [ "as" identifier ] block .
```

Example:

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

#### Range Loop

```
range_cycle = "cycle" expression ".." expression [ "as" identifier ] block .
```

Example:

```serenade
cycle 0..9 as n {
  emit "n={n}"
}
```

Ranges are **inclusive** on both ends. The loop `0..9` iterates from 0 to 9 (10 iterations).

#### Break Statement

Exit a loop early:

```serenade
cycle 100 as i {
  if i == 50 {
    break
  }
}
```

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

### Task Declarations

Tasks are lambda-style functions:

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

```
Program       = { Statement } .
Statement     = LetStmt | EmitStmt | IfStmt | CycleStmt | TaskStmt |
                ReturnStmt | BreakStmt | WaitStmt | SpawnStmt |
                NativeBlock | AsmStmt | ExprStmt .

LetStmt       = "let" identifier "=" Expr .
EmitStmt      = "emit" Expr .
IfStmt        = "if" Expr Block .
CycleStmt     = "cycle" ( CountCycle | RangeCycle ) .
CountCycle    = Expr [ "as" identifier ] Block .
RangeCycle    = Expr ".." Expr [ "as" identifier ] Block .
TaskStmt      = "task" identifier "(" [ ParamList ] ")" Block .
ReturnStmt    = "return" Expr .
BreakStmt     = "break" .
WaitStmt      = ( "wait" | "sleep" ) Expr .
SpawnStmt     = "spawn" Expr .
NativeBlock   = "native" [ "{" ] { CppLine } "endnative" .
AsmStmt       = "asm" AsmOp AsmArgs .
ExprStmt      = Expr .

Block         = "{" { Statement } "}" .
ParamList     = identifier { "," identifier } .

Expr          = PipeExpr .
PipeExpr      = LogicalExpr { ">>" LogicalExpr } .
LogicalExpr   = CompareExpr { ( "&&" | "||" ) CompareExpr } .
CompareExpr   = AddExpr { ( "==" | "!=" | "<" | ">" | "<=" | ">=" ) AddExpr } .
AddExpr       = MulExpr { ( "+" | "-" ) MulExpr } .
MulExpr       = UnaryExpr { ( "*" | "/" | "%" ) UnaryExpr } .
UnaryExpr     = PrimaryExpr | "!" UnaryExpr | "-" UnaryExpr | "move" UnaryExpr .
PrimaryExpr   = Literal | identifier | FuncCall | MemberAccess |
                ArenaAlloc | PointerIndex | SafeAccess |
                RegistryLit | "(" Expr ")" .

FuncCall      = identifier "(" [ ExprList ] ")" .
MemberAccess  = Expr "." identifier .
ArenaAlloc    = "@f32" "[" Expr "]" .
PointerIndex  = Expr "^" "[" Expr "]" .
SafeAccess    = Expr "?." identifier .
RegistryLit   = "registry" "{" [ KeyValueList ] "}" .

KeyValueList  = KeyValue { "," KeyValue } .
KeyValue      = identifier ":" Expr .
ExprList      = Expr { "," Expr } .

Literal       = IntLit | FloatLit | StringLit | BoolLit .
BoolLit       = "true" | "false" .
```

### Reserved Words

```
let cycle registry emit task scan if return break
spawn await native atomic move using as endnative
true false
```

### Operator Precedence

From highest to lowest:

1. Primary (literals, identifiers, function calls)
2. Unary (`!`, `-`, `move`)
3. Multiplicative (`*`, `/`, `%`)
4. Additive (`+`, `-`)
5. Comparison (`==`, `!=`, `<`, `>`, `<=`, `>=`)
6. Logical AND (`&&`)
7. Logical OR (`||`)
8. Pipe (`>>`)

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

## Frequently Asked Questions

**Q: Why no semicolons?**

A: Serenade is line-based. One statement per line makes the code cleaner and easier to read.

**Q: Can I use C++ features directly?**

A: Yes, through `native` blocks you have full C++ access.

**Q: Is Serenade interpreted or compiled?**

A: Compiled. Serenade transpiles to C++/Go/ASM and compiles to native code.

**Q: What's the performance compared to C++?**

A: Generated C++ code is as fast as hand-written C++. Assembly paths can exceed C++ performance.

**Q: Can I link against existing libraries?**

A: Yes, modify `serena.build.conf` to add linker flags.

**Q: How do I debug Serenade code?**

A: Examine the generated C++ code in the temp build directory. Use `srcheckproc()` for runtime diagnostics.

**Q: Can I use Serenade in production?**

A: Serenade is alpha software. Use for prototyping and experimentation.

**Q: How do I contribute?**

A: See the project repository for contribution guidelines.

**Q: What's the license?**

A: MIT

---

## Acknowledgments

Serenade draws inspiration from:

- **Go** - simplicity and tooling
- **Rust** - memory safety concepts
- **Zig** - compile-time execution
- **C++** - performance and control

Special thanks to the open-source community for tools and libraries that make Serenade possible.

---

*End of Serenade Language Specification v0.2-Alpha*
