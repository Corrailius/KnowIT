# C & C++ — From First Program to Confident

> **Who is this for?** Everyone, from someone writing their first line of code to developers who have been shipping C for decades. Every section opens with an **"In plain English"** box that explains the idea without jargon. The **"Deep dive"** notes carry the precise details (undefined behavior, differences between standards, platform quirks) that experienced developers care about. Skim what you already know, slow down where you don't.

> **About the code:** every complete program in this guide (the ones with a `main`) was compiled with `-Wall -Wextra -Wpedantic` and run, with AddressSanitizer and UndefinedBehaviorSanitizer enabled. C samples use `-std=c17`, C++ samples use `-std=c++20`. Two examples intentionally trigger a compiler warning, and the text says so. Where output depends on your platform, the text says so too.

---

## Table of Contents

1. [What Are C and C++?](#1-what-are-c-and-c)
2. [From Source Code to Executable](#2-from-source-code-to-executable)
3. [Your First Program](#3-your-first-program)
4. [The Building Blocks](#4-the-building-blocks)
5. [Functions](#5-functions)
6. [Memory and Pointers — The Heart of C](#6-memory-and-pointers--the-heart-of-c)
7. [Arrays and Strings](#7-arrays-and-strings)
8. [Structs, Enums, and Unions](#8-structs-enums-and-unions)
9. [Dynamic Memory](#9-dynamic-memory)
10. [Headers, Multiple Files, and Build Tools](#10-headers-multiple-files-and-build-tools)
11. [C++ Essentials — What Changes](#11-c-essentials--what-changes)
12. [C or C++? Side by Side](#12-c-or-c-side-by-side)
13. [Undefined Behavior and Classic Pitfalls](#13-undefined-behavior-and-classic-pitfalls)
14. [Debugging and Tooling](#14-debugging-and-tooling)
15. [Language Standards: Which One to Use?](#15-language-standards-which-one-to-use)
16. [Quick Reference and Glossary](#16-quick-reference-and-glossary)

---

## 1. What Are C and C++?

> **In plain English:** C is a small, fast language that sits very close to the hardware: you decide exactly how memory is used, and almost nothing happens behind your back. C++ began as "C with classes" and grew into a much larger language with tools for managing complexity (classes, templates, a rich standard library), while still letting you drop down to C's level whenever you want. Today they are two separate languages that share a common core.

| | **C** | **C++** |
|---|---|---|
| Created by | Dennis Ritchie, Bell Labs (early 1970s) | Bjarne Stroustrup, Bell Labs (started 1979 as "C with Classes", renamed C++ in 1983) |
| Style | Procedural | Multi-paradigm: procedural, object-oriented, generic, functional-style |
| Memory management | Manual (`malloc` / `free`) | Manual **or** automatic via RAII and smart pointers |
| Standard library | Small (`stdio.h`, `string.h`, `stdlib.h`, ...) | Large (containers, algorithms, strings, threads, ...) |
| Typical uses | Operating systems, drivers, embedded, language runtimes, libraries with a stable ABI | Games, browsers, databases, desktop apps, high-performance and real-time systems |

### C++ is not a strict superset of C

You will often read that "C++ is C plus extras". It is *mostly* true, but not always. Some valid C does not compile as C++, and some code compiles in both but **behaves differently**:

| Construct | In C | In C++ |
|---|---|---|
| `int *p = malloc(4);` | Valid (`void *` converts implicitly) | **Error**: needs a cast (or, better, `new` / a container) |
| `sizeof('a')` | `sizeof(int)`, usually 4 (a character literal has type `int`) | `1` (a character literal has type `char`) |
| Calling an undeclared function | Invalid since C99 (many compilers only warn; GCC 14+ errors) | Always an error |
| Variable-length arrays `int a[n];` | Valid since C99 (optional since C11) | Not standard C++ (some compilers allow it as an extension) |
| `class`, `new`, `delete`, `template` as variable names | Allowed (not keywords in C) | Error (reserved words) |
| `restrict` pointers | Supported (C99) | Not standard (compilers offer `__restrict`) |

> **Deep dive:** because of differences like these, the two languages have separate standards committees (ISO/IEC JTC 1/SC 22 working groups WG14 for C and WG21 for C++). Mixing them in one project is common and works well, as long as you compile each file with the right compiler mode and use `extern "C"` at the boundary (see [section 10](#10-headers-multiple-files-and-build-tools)).

### Which one should you learn?

- **Learn C first** if you want to understand how computers really work: memory, pointers and the machine model are laid bare. Everything you learn carries over to C++.
- **Learn C++** if you want to build large, fast applications with less risk of memory bugs. Modern C++ (C++11 and later) is a very different, much safer experience than the C++ of the 1990s.
- **Many developers learn both**, and this guide is organized so you can: the C chapters come first, then [section 11](#11-c-essentials--what-changes) shows what C++ changes.

---

## 2. From Source Code to Executable

> **In plain English:** a computer can't run the text you type. A set of tools called the **toolchain** translates your source files into machine code in several steps. Knowing these steps makes error messages far less mysterious: "compile error" and "link error" are two different failures in two different stages.

```
 main.c ──► Preprocessor ──► Compiler ──► Assembler ──► main.o ──┐
 util.c ──► Preprocessor ──► Compiler ──► Assembler ──► util.o ──┼─► Linker ──► program
                                              libraries (.a/.lib/.so/.dll) ───┘
```

| Stage | What happens | GCC flag to stop here | Output |
|---|---|---|---|
| 1. **Preprocessor** | Handles lines starting with `#`: pastes in `#include` files, expands `#define` macros, removes comments | `-E` | `.i` (C) / `.ii` (C++) |
| 2. **Compiler** | Checks syntax and types, translates C/C++ into assembly | `-S` | `.s` |
| 3. **Assembler** | Turns assembly into machine code in an *object file* | `-c` | `.o` / `.obj` |
| 4. **Linker** | Joins your object files and libraries, resolves function references between them | (default) | executable |

You can run each stage by hand to see what it produces:

```bash
gcc -E main.c -o main.i     # 1. preprocess  → expanded source
gcc -S main.i -o main.s     # 2. compile     → assembly
gcc -c main.s -o main.o     # 3. assemble    → object file
gcc main.o -o main          # 4. link        → executable
```

On Linux the result is an **ELF** file; on Windows it's a **PE** file, the format dissected in [The Windows .exe Format](The%20Windows%20Executable%20%28.exe%29%20Format.md). The section names you'll meet in [section 6](#6-memory-and-pointers--the-heart-of-c) (`.text`, `.data`, `.rdata`, `.bss`) are exactly the ones described there.

### The common compilers

| Compiler | Commands | Platform |
|---|---|---|
| **GCC** | `gcc` (C), `g++` (C++) | Linux, macOS (via Homebrew), Windows (MinGW / MSYS2) |
| **Clang / LLVM** | `clang`, `clang++` | Linux, macOS, Windows |
| **MSVC** | `cl.exe` | Windows (Visual Studio / Build Tools) |

> **Deep dive:** each `.c` / `.cpp` file, after preprocessing, is called a **translation unit**, and it is compiled completely independently of the others. That is why a header must *declare* what a file uses, and why the linker, not the compiler, reports `undefined reference to 'foo'`: only at link time does anyone check that `foo` exists somewhere. In C++, the **One Definition Rule (ODR)** says every function and non-inline variable must be defined exactly once across the whole program.

---

## 3. Your First Program

**In C:**

```c
#include <stdio.h>

int main(void)
{
    printf("Hello, World!\n");
    return 0;
}
```

**In C++:**

```cpp
#include <iostream>

int main()
{
    std::cout << "Hello, World!\n";
    return 0;
}
```

**Compile and run:**

```bash
gcc -std=c17   -Wall -Wextra -Wpedantic hello.c   -o hello    # C
g++ -std=c++20 -Wall -Wextra -Wpedantic hello.cpp -o hello    # C++

./hello          # Linux / macOS
hello.exe        # Windows
```

> **✔ Tip:** always compile with `-Wall -Wextra -Wpedantic` (or `/W4` on MSVC). Warnings are free bug reports, and most real-world C/C++ bugs would have been flagged by one.

| Piece | Meaning |
|---|---|
| `#include <stdio.h>` | Pastes in the declarations for the standard I/O functions (`printf`, ...). `<...>` searches the system headers. |
| `int main(void)` | The entry point. The `int` it returns is the **exit code** (`0` = success). |
| `printf(...)` / `std::cout <<` | Prints text. `\n` is a newline. |
| `return 0;` | Exit code. In `main` only, this line is optional since C99 and C++98: reaching the closing brace returns `0`. |
| `std::` | The **namespace** of the C++ standard library. It prevents name clashes. |

> **Deep dive:** in C before C23, `int main()` means "unspecified parameters" while `int main(void)` means "no parameters", so `(void)` is the correct, explicit form. C23 makes the two equivalent, as C++ has always done.

### Reading command-line arguments

```c
#include <stdio.h>

int main(int argc, char *argv[])
{
    printf("program name: %s\n", argv[0]);
    for (int i = 1; i < argc; i++)
        printf("argument %d: %s\n", i, argv[i]);
    return 0;
}
```

`argc` is the number of arguments (including the program name), `argv` is an array of C strings.

---

## 4. The Building Blocks

### 4.1 Variables and fundamental types

> **In plain English:** a variable is a named box in memory. Its **type** tells the compiler how big the box is and how to interpret what's inside: whole number, decimal number, character...

| Type | Typical size | Guaranteed by the standard |
|---|---|---|
| `char` | 1 byte | Exactly 1 byte by definition (`sizeof(char) == 1`), at least 8 bits |
| `short` | 2 bytes | At least 16 bits |
| `int` | 4 bytes | At least 16 bits |
| `long` | 4 or 8 bytes | At least 32 bits (**4 on 64-bit Windows, 8 on 64-bit Linux/macOS**) |
| `long long` | 8 bytes | At least 64 bits |
| `float` | 4 bytes | Usually IEEE 754 single precision |
| `double` | 8 bytes | Usually IEEE 754 double precision |
| `bool` | 1 byte | `<stdbool.h>` in C (a keyword in C++ and in C23) |

Each integer type also has an `unsigned` variant (no negative values, twice the positive range).

```c
#include <stdio.h>
#include <stdint.h>

int main(void)
{
    printf("char      : %zu\n", sizeof(char));
    printf("short     : %zu\n", sizeof(short));
    printf("int       : %zu\n", sizeof(int));
    printf("long      : %zu\n", sizeof(long));
    printf("long long : %zu\n", sizeof(long long));
    printf("float     : %zu\n", sizeof(float));
    printf("double    : %zu\n", sizeof(double));
    printf("void *    : %zu\n", sizeof(void *));
    printf("int32_t   : %zu\n", sizeof(int32_t));
    return 0;
}
```

Run it on your machine: the sizes of `long` and `void *` reveal your platform.

> **✔ Tip:** when the exact size matters (file formats, network protocols, hardware registers), use the **fixed-width types** from `<stdint.h>` / `<cstdint>`: `int8_t`, `uint8_t`, `int16_t`, `uint16_t`, `int32_t`, `uint32_t`, `int64_t`, `uint64_t`. For sizes and array indices, use `size_t` (an unsigned type, printed with `%zu`).

> **Deep dive:**
> - **Signed integer overflow is undefined behavior**; **unsigned overflow wraps** modulo 2<sup>N</sup> (fully defined). The compiler may assume signed overflow never happens and optimize accordingly.
> - Whether plain `char` is signed or unsigned is **implementation-defined** (signed on x86 Linux, unsigned on ARM Linux).
> - When signed and unsigned operands meet, the signed one is converted to unsigned. This surprises almost everyone once:

```c
#include <stdio.h>

int main(void)
{
    int a = -1;
    unsigned int b = 1;

    if (a < b)
        printf("-1 < 1\n");
    else
        printf("surprise: -1 is NOT less than 1u\n");   /* this branch runs */
    return 0;
}
```

`-1` becomes `4294967295` before the comparison. GCC and Clang warn about it with `-Wextra` (`-Wsign-compare`), which is one more reason to turn warnings on.

### 4.2 Operators

| Category | Operators | Notes |
|---|---|---|
| Arithmetic | `+ - * / %` | Integer `/` **truncates toward zero** (`7 / 2 == 3`, `-7 / 2 == -3`). `%` is the remainder. |
| Comparison | `== != < > <= >=` | Give `1` (true) or `0` (false) in C, `bool` in C++. |
| Logical | `&& \|\| !` | **Short-circuit**: in `a && b`, `b` is not evaluated if `a` is false. |
| Bitwise | `& \| ^ ~ << >>` | Work on individual bits. |
| Assignment | `= += -= *= /= %= &= \|= ^= <<= >>=` | `x += 2` means `x = x + 2`. |
| Increment | `++ --` | `i++` returns the old value, `++i` the new one. |
| Other | `?:` `sizeof` `,` `&` `*` `->` `.` | Ternary, size in bytes, address-of, dereference, member access. |

> **⚠ Classic trap:** `=` assigns, `==` compares. Writing `if (x = 5)` assigns 5 to `x` and is always true. Compilers warn about it with `-Wall`.

Bitwise operators are how flags are stored compactly. Hardware registers, file formats and the PE `Characteristics` fields in the [`.exe` guide](The%20Windows%20Executable%20%28.exe%29%20Format.md) all use this technique:

```c
#include <stdio.h>

#define FLAG_READ  (1u << 0)   /* 0001 */
#define FLAG_WRITE (1u << 1)   /* 0010 */
#define FLAG_EXEC  (1u << 2)   /* 0100 */

int main(void)
{
    unsigned int perms = FLAG_READ | FLAG_WRITE;   /* set two flags    */
    perms |= FLAG_EXEC;                            /* add a flag       */
    perms &= ~FLAG_WRITE;                          /* remove a flag    */

    if (perms & FLAG_EXEC)                         /* test a flag      */
        printf("can execute\n");
    printf("perms = %u\n", perms);                 /* 5 (binary 0101)  */
    return 0;
}
```

> **✔ Tip:** don't memorize the full precedence table. Add parentheses whenever an expression mixes operator families: `(a & MASK) == 0`, not `a & MASK == 0` (which parses as `a & (MASK == 0)`).

### 4.3 Control flow

```c
#include <stdio.h>

int main(void)
{
    for (int i = 1; i <= 15; i++) {          /* for: counted loop */
        if (i % 15 == 0)      printf("FizzBuzz\n");
        else if (i % 3 == 0)  printf("Fizz\n");
        else if (i % 5 == 0)  printf("Buzz\n");
        else                  printf("%d\n", i);
    }

    int n = 3;
    while (n > 0) {                          /* while: test first */
        printf("%d... ", n);
        n--;
    }
    printf("liftoff\n");

    do {                                     /* do-while: body runs at least once */
        n++;
    } while (n < 3);
    printf("n = %d\n", n);
    return 0;
}
```

`switch` selects between constant values. **Each case falls through to the next unless you `break`:**

```c
switch (command) {
    case 'a':
        add_item();
        break;                 /* without this, execution would continue into 'd' */
    case 'd':
        delete_item();
        break;
    default:
        puts("unknown command");
}
```

`break` leaves the innermost loop or switch, `continue` jumps to the next iteration. `goto` exists and has a few legitimate uses (such as jumping to a single cleanup block at the end of a C function), but it is rare.

### 4.4 Input and output

| `printf` format | Prints | Argument type |
|---|---|---|
| `%d` / `%i` | signed decimal | `int` |
| `%u` | unsigned decimal | `unsigned int` |
| `%ld` / `%lld` | signed decimal | `long` / `long long` |
| `%zu` | unsigned decimal | `size_t` |
| `%f` | decimal number | `double` (a `float` is promoted) |
| `%c` | one character | `int` / `char` |
| `%s` | string | `const char *` |
| `%x` / `%X` | hexadecimal | `unsigned int` |
| `%p` | pointer | `void *` (cast to it) |
| `%%` | a literal `%` | — |

> **⚠ Important:** `printf` cannot check that your format matches your arguments at run time. A mismatch such as `%d` with a `double` is undefined behavior. `-Wall` makes GCC and Clang check literal format strings for you.

Reading input in C: prefer `fgets` over `scanf("%s", ...)`, which has no length limit and is a classic buffer-overflow source.

```c
#include <stdio.h>
#include <string.h>

int main(void)
{
    char name[32];

    printf("Your name: ");
    if (fgets(name, sizeof name, stdin) != NULL) {
        name[strcspn(name, "\n")] = '\0';        /* strip the trailing newline */
        printf("Hello, %s!\n", name);
    }
    return 0;
}
```

The C++ equivalent needs no size management at all:

```cpp
#include <iostream>
#include <string>

int main()
{
    std::string name;
    std::cout << "Your name: ";
    std::getline(std::cin, name);
    std::cout << "Hello, " << name << "!\n";
}
```

---

## 5. Functions

> **In plain English:** a function is a named, reusable block of code. You give it inputs (**parameters**), it may give back one output (**return value**). Splitting a program into small functions is the single most effective way to keep it understandable.

```c
int add(int a, int b);        /* declaration (prototype): promises the function exists */

int add(int a, int b)         /* definition: the actual code */
{
    return a + b;
}
```

A declaration tells the compiler *how to call* a function; the definition *is* the function. A function must be declared before it is used.

### Parameters are copies

C always passes arguments **by value**: the function works on a copy. To let a function modify the caller's variable, pass its **address** (a pointer, [section 6](#6-memory-and-pointers--the-heart-of-c)):

```c
#include <stdio.h>

void swap_wrong(int a, int b) { int t = a; a = b; b = t; }     /* swaps the copies only */
void swap_ok(int *a, int *b)  { int t = *a; *a = *b; *b = t; } /* swaps the originals   */

int main(void)
{
    int x = 1, y = 2;

    swap_wrong(x, y);
    printf("after swap_wrong: x=%d y=%d\n", x, y);   /* x=1 y=2 */

    swap_ok(&x, &y);
    printf("after swap_ok:    x=%d y=%d\n", x, y);   /* x=2 y=1 */
    return 0;
}
```

C++ adds **references**, which do the same job with cleaner syntax:

```cpp
#include <iostream>

void swap_ref(int &a, int &b) { int t = a; a = b; b = t; }

int main()
{
    int x = 1, y = 2;
    swap_ref(x, y);                       // no & at the call site
    std::cout << x << ' ' << y << '\n';   // 2 1
}
```

C++ also supports **function overloading** (several functions with the same name and different parameters) and **default arguments**. C has neither.

### Function pointers

A function has an address too, and you can pass it around. The classic use is a callback, such as the comparison function for `qsort`:

```c
#include <stdio.h>
#include <stdlib.h>

static int cmp_int(const void *pa, const void *pb)
{
    int a = *(const int *)pa;
    int b = *(const int *)pb;
    return (a > b) - (a < b);          /* -1, 0 or 1; never overflows, unlike a - b */
}

int main(void)
{
    int v[] = {42, 7, 19, -3, 7};
    size_t n = sizeof v / sizeof v[0];

    qsort(v, n, sizeof v[0], cmp_int);

    for (size_t i = 0; i < n; i++)
        printf("%d ", v[i]);
    putchar('\n');                      /* -3 7 7 19 42 */
    return 0;
}
```

> **Deep dive:** `static` on a function (or a global variable) gives it **internal linkage**: it is visible only inside its own file, which avoids name collisions at link time. `static` on a *local* variable gives it **static storage duration**: it keeps its value between calls. The same keyword, two meanings; C++ adds a third (class members shared by all objects).

---

## 6. Memory and Pointers — The Heart of C

> **In plain English:** picture memory as a very long street of numbered mailboxes. Each mailbox holds one byte and has an **address** (its number). A **pointer** is simply a variable that stores an address, so it "points at" another variable. Pointers are what make C powerful, and also what make it dangerous: the language trusts you completely.

### Where do variables live?

```
 High addresses
┌──────────────────────────────────────┐
│ Stack                                │  local variables, function calls
│   ↓ grows (usually) downward         │  created/destroyed automatically
│                                      │
│   ↑ grows upward                     │
│ Heap                                 │  malloc / new: YOU decide when it dies
├──────────────────────────────────────┤
│ .bss     uninitialized globals       │
│ .data    initialized globals         │
│ .rodata  string literals, constants  │
│ .text    your machine code           │
└──────────────────────────────────────┘
 Low addresses
```

| Region | Holds | Lifetime |
|---|---|---|
| **Stack** | Local variables, function arguments, return addresses | Until the function returns |
| **Heap** | Memory you request with `malloc` / `new` | Until you release it |
| **`.data` / `.bss`** | Global and `static` variables | The whole program |
| **`.rodata`** | String literals and constants | The whole program (read-only) |
| **`.text`** | The compiled code | The whole program (read-only, executable) |

The names `.text`, `.data`, `.rdata` (the Windows name for `.rodata`) and `.bss` are the same section names stored in an executable file: see [Standard Sections Reference](The%20Windows%20Executable%20%28.exe%29%20Format.md#standard-sections-reference).

### Pointer basics

```c
#include <stdio.h>

int main(void)
{
    int x = 10;
    int *p = &x;                      /* p holds the address of x */

    printf("x     = %d\n", x);
    printf("&x    = %p\n", (void *)&x);
    printf("p     = %p\n", (void *)p);   /* same address as &x */
    printf("*p    = %d\n", *p);          /* follow the pointer: 10 */

    *p = 25;                             /* write through the pointer */
    printf("x now = %d\n", x);           /* 25 */

    int arr[3] = {10, 20, 30};
    int *q = arr;                        /* an array decays to a pointer to its first element */
    printf("%d %d %d\n", *q, *(q + 1), *(q + 2));   /* q + 1 moves by one whole int */
    return 0;
}
```

| Syntax | Meaning |
|---|---|
| `int *p;` | Declare `p` as a pointer to `int` |
| `&x` | **Address of** `x` |
| `*p` | **Dereference**: the object `p` points to |
| `p->field` | Shorthand for `(*p).field` (struct access through a pointer) |
| `p + 1` | Address of the *next element*: moves by `sizeof(*p)` bytes, not 1 |
| `NULL` (`nullptr` in C++11, and C23) | "Points to nothing". Dereferencing it is undefined behavior. |

### `const` and pointers: read right to left

| Declaration | Reads as | You can change... |
|---|---|---|
| `const int *p` | "p is a pointer to a const int" | the pointer, not the pointed-to value |
| `int *const p` | "p is a const pointer to an int" | the pointed-to value, not the pointer |
| `const int *const p` | "const pointer to a const int" | neither |

### The classic pointer bugs

```c
int *dangling(void)
{
    int local = 42;
    return &local;        /* BUG: local is destroyed when the function returns */
}

void uninitialized(void)
{
    int *p;               /* BUG: p holds garbage */
    *p = 1;               /* writes to a random address */
}
```

> **⚠ Important:** a **dangling pointer** (to something that no longer exists), an **uninitialized pointer** and a **null pointer** are the three ways to hold a bad address. The fix is a habit: initialize every pointer (to a valid address or to `NULL`), and never return the address of a local variable.

> **Deep dive:**
> - **Strict aliasing:** accessing an object through a pointer of an incompatible type (for example reading a `float` through an `int *`) is undefined behavior, and optimizers rely on it. To reinterpret bytes legally, use `memcpy`, a `union` (C only) or `std::bit_cast` (C++20), or access through `char *`, which is always permitted.
> - **Alignment:** most types must live at addresses that are a multiple of their alignment (often their size). Misaligned access is undefined behavior and crashes on some CPUs.
> - Pointer arithmetic is only defined inside an array (or one element past its end). Computing a pointer farther away is undefined behavior even if you never dereference it.

---

## 7. Arrays and Strings

### Arrays

> **In plain English:** an array is a row of same-type boxes sitting next to each other in memory. You reach the *n*-th box with `a[n]`, counting from **0**. C does **not** check that `n` is in range.

```c
#include <stdio.h>

int main(void)
{
    int scores[5] = {90, 85, 70};                      /* the rest are zero-initialized */
    size_t count = sizeof scores / sizeof scores[0];   /* 5: total bytes / bytes per element */

    for (size_t i = 0; i < count; i++)
        printf("scores[%zu] = %d\n", i, scores[i]);
    return 0;
}
```

Two facts to keep in mind:

- **Valid indices are `0` to `size - 1`.** Writing `scores[5]` compiles fine and corrupts whatever sits after the array (see [section 13](#13-undefined-behavior-and-classic-pitfalls)).
- **Arrays decay to pointers when passed to functions.** Inside the function, `sizeof` gives the size of a *pointer*, not of the array, so you must pass the length separately:

```c
void print_all(const int *values, size_t count)   /* "int values[]" means exactly the same */
{
    for (size_t i = 0; i < count; i++)
        printf("%d\n", values[i]);
}
```

### Strings in C

> **In plain English:** C has no string type. A "string" is an array of `char` whose end is marked by a zero byte, `'\0'` (the **null terminator**). `"Hi"` takes **3** bytes: `'H'`, `'i'`, `'\0'`.

```c
char  a[] = "Hi";          /* a COPY in modifiable memory: 3 bytes, you may change them */
const char *b = "Hi";      /* points at a read-only string literal: never modify it     */
```

> **⚠ Important:** modifying a string literal (`char *s = "Hi"; s[0] = 'h';`) is undefined behavior and usually crashes. Declare such pointers `const char *` so the compiler stops you.

| Function (`<string.h>`) | Does | Caution |
|---|---|---|
| `strlen(s)` | Length, not counting `'\0'` | Walks the string: O(n) |
| `strcmp(a, b)` | `0` if equal, negative / positive otherwise | Never compare strings with `==` (that compares addresses) |
| `strcpy(dst, src)` | Copies a string | **Unsafe**: no size check |
| `strncpy(dst, src, n)` | Copies at most `n` chars | **Does not always add `'\0'`**; pads with zeros: easy to misuse |
| `strcat(dst, src)` | Appends | **Unsafe**: no size check |
| `snprintf(dst, size, fmt, ...)` | Formatted copy, **never overflows**, always terminates | The safe default |

```c
#include <stdio.h>
#include <string.h>

int main(void)
{
    char buf[8];
    const char *src = "Hello, World!";               /* 13 characters + '\0' */

    snprintf(buf, sizeof buf, "%s", src);            /* truncates safely to 7 characters + '\0' */
    printf("buf = \"%s\" (length %zu)\n", buf, strlen(buf));
    return 0;
}
```

The output is `buf = "Hello, " (length 7)`. The buffer is never overrun and always terminated. GCC even notices at compile time that the text won't fit and warns (`-Wformat-truncation`): the function protects your memory, the warning protects your intent.

> **✔ Tip:** in C++, use `std::string`. It grows automatically, knows its length, compares with `==`, and cannot overflow through normal use.

---

## 8. Structs, Enums, and Unions

### Structs: grouping related data

```c
#include <stdio.h>

typedef struct {
    char name[20];
    int  age;
} Person;

static void print_person(const Person *p)
{
    printf("%s (%d)\n", p->name, p->age);     /* p->age is shorthand for (*p).age */
}

int main(void)
{
    Person alice = { "Alice", 30 };
    print_person(&alice);                      /* pass the address: no big copy */
    return 0;
}
```

### Padding and alignment

> **In plain English:** the compiler may leave empty gaps inside a struct so each member sits at an address its type likes. That means a struct can be **bigger than the sum of its members**, and the *order* of the members matters.

```c
#include <stdio.h>

struct Bad  { char a; int b; char c; };   /* a, 3 bytes padding, b, c, 3 bytes padding */
struct Good { int b; char a; char c; };   /* b, a, c, 2 bytes padding                 */

int main(void)
{
    printf("Bad : %zu bytes\n", sizeof(struct Bad));    /* typically 12 */
    printf("Good: %zu bytes\n", sizeof(struct Good));   /* typically 8  */
    return 0;
}
```

The same idea, alignment of data to a boundary, is behind `FileAlignment` and `SectionAlignment` in the [`.exe` guide](The%20Windows%20Executable%20%28.exe%29%20Format.md). Rule of thumb: declare members from largest to smallest.

> **Deep dive:** never assume a struct's layout matches a file format or network packet. Padding, endianness and member order are the compiler's and the CPU's business. Serialize field by field, or use explicit, fixed-width types plus compiler-specific packing (`#pragma pack`, `__attribute__((packed))`) and accept the portability and speed trade-offs.

### Enums: named constants

```c
enum Color { RED, GREEN, BLUE };          /* RED = 0, GREEN = 1, BLUE = 2 */
enum Color c = GREEN;
```

In C, an enum is essentially an `int` with friendly names, and its constants leak into the surrounding scope. C++11 adds **`enum class`**, which is scoped (`Color::Green`) and does not convert implicitly to `int`. Prefer it in C++.

### Unions: one storage, several views

```c
#include <stdio.h>
#include <stdint.h>

union Bits { float f; uint32_t u; };       /* both members share the same 4 bytes */

int main(void)
{
    union Bits b = { .f = 1.0f };
    printf("1.0f as hex: 0x%08X\n", (unsigned)b.u);   /* 0x3F800000 with IEEE 754 floats */
    return 0;
}
```

> **Deep dive:** reading a union member other than the one last written ("type punning") is allowed in C but **undefined behavior in C++**. The portable way in both languages is `memcpy`:

```cpp
#include <cstdint>
#include <cstdio>
#include <cstring>

int main()
{
    float f = 1.0f;
    std::uint32_t u;
    static_assert(sizeof f == sizeof u, "sizes must match");
    std::memcpy(&u, &f, sizeof u);                      // well-defined in C and C++
    std::printf("0x%08X\n", static_cast<unsigned>(u));  // 0x3F800000
}
```

---

## 9. Dynamic Memory

> **In plain English:** stack variables disappear when their function ends. When data must outlive the function, or its size is only known at run time, you ask the system for memory from the **heap**, and **you are responsible for giving it back**.

### In C: `malloc` and `free`

| Function (`<stdlib.h>`) | Does |
|---|---|
| `malloc(n)` | Reserves `n` bytes. Contents are **uninitialized**. Returns `NULL` on failure. |
| `calloc(count, size)` | Reserves `count × size` bytes, **zeroed**. |
| `realloc(ptr, n)` | Resizes a block, possibly moving it. Returns `NULL` on failure (the old block is still valid). |
| `free(ptr)` | Releases a block. `free(NULL)` is allowed and does nothing. |

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    size_t capacity = 2, size = 0;
    int *data = malloc(capacity * sizeof *data);        /* sizeof *data survives a type change */
    if (data == NULL) { perror("malloc"); return 1; }   /* ALWAYS check */

    for (int i = 1; i <= 5; i++) {
        if (size == capacity) {                         /* full: grow the array */
            size_t new_cap = capacity * 2;
            int *tmp = realloc(data, new_cap * sizeof *data);
            if (tmp == NULL) { free(data); perror("realloc"); return 1; }
            data = tmp;                  /* overwrite 'data' only after realloc succeeded */
            capacity = new_cap;
        }
        data[size++] = i * i;
    }

    for (size_t i = 0; i < size; i++)
        printf("%d ", data[i]);
    putchar('\n');                                      /* 1 4 9 16 25 */

    free(data);
    data = NULL;                                        /* no dangling pointer left behind */
    return 0;
}
```

### The five classic heap bugs

| Bug | What happens | Fix |
|---|---|---|
| **Memory leak** | You never call `free`: memory use grows until the program ends | Every `malloc` needs exactly one matching `free` |
| **Double free** | `free(p); free(p);`: corrupts the allocator, often exploitable | Set `p = NULL` after freeing |
| **Use after free** | Reading or writing a block you already released | Same: nullify, and keep ownership clear |
| **Buffer overflow** | Writing past the end of the block | Track sizes, use bounds-checked operations |
| **Uninitialized read** | `malloc` memory has random contents | Use `calloc`, or initialize before reading |

### In C++: let the objects clean up (RAII)

> **In plain English:** C++ ties a resource's lifetime to an object's lifetime. When the object goes out of scope its **destructor** runs automatically and releases the resource. This idea is called **RAII** (*Resource Acquisition Is Initialization*), and it is the single most important C++ idiom: it removes most manual `free` / `delete` calls and with them most leaks.

C++ has `new` / `delete` (and `new[]` / `delete[]` for arrays; mixing them up is undefined behavior), but modern code rarely calls them directly. It uses **smart pointers** instead:

| Smart pointer (`<memory>`) | Ownership | Use when |
|---|---|---|
| `std::unique_ptr<T>` | **Single** owner, not copyable (movable), zero overhead | The default choice |
| `std::shared_ptr<T>` | **Shared** ownership via reference counting | Several owners genuinely need the object |
| `std::weak_ptr<T>` | Non-owning observer of a `shared_ptr` | Breaking ownership cycles |

```cpp
#include <iostream>
#include <memory>

struct Widget {
    explicit Widget(int id) : id_(id) { std::cout << "Widget " << id_ << " created\n"; }
    ~Widget()                         { std::cout << "Widget " << id_ << " destroyed\n"; }
    int id_;
};

int main()
{
    auto w = std::make_unique<Widget>(1);     // freed automatically at the end of main
    {
        auto s1 = std::make_shared<Widget>(2);
        auto s2 = s1;                         // reference count is now 2
        std::cout << "use_count = " << s1.use_count() << '\n';
    }                                         // last owner gone: Widget 2 destroyed here
    std::cout << "end of main\n";
}                                             // Widget 1 destroyed here
```

> **✔ Tip:** in modern C++, if you type `new` or `delete` yourself, stop and ask whether `std::vector`, `std::string`, `std::make_unique` or `std::make_shared` would do the job. Almost always, one of them does.

---

## 10. Headers, Multiple Files, and Build Tools

> **In plain English:** real programs are split across many files. A **header** (`.h` / `.hpp`) lists *what exists* (declarations); a **source file** (`.c` / `.cpp`) contains *how it works* (definitions). Other files `#include` the header to learn how to call your code, and the linker joins everything.

**math_utils.h**

```c
#ifndef MATH_UTILS_H        /* include guard: prevents double inclusion */
#define MATH_UTILS_H

int add(int a, int b);      /* declarations only */
int square(int x);

#endif /* MATH_UTILS_H */
```

**math_utils.c**

```c
#include "math_utils.h"

int add(int a, int b) { return a + b; }       /* definitions */
int square(int x)     { return x * x; }
```

**main.c**

```c
#include <stdio.h>
#include "math_utils.h"

int main(void)
{
    printf("%d\n", add(2, 3));       /* 5  */
    printf("%d\n", square(4));       /* 16 */
    return 0;
}
```

Build by hand, one translation unit at a time, then link:

```bash
gcc -std=c17 -Wall -Wextra -Wpedantic -c main.c
gcc -std=c17 -Wall -Wextra -Wpedantic -c math_utils.c
gcc main.o math_utils.o -o app
```

| Rule | Why |
|---|---|
| `#include <file>` for system / library headers, `#include "file"` for your own | `<>` searches the system paths; `""` searches your project first |
| Every header gets an **include guard** (`#ifndef` / `#define` / `#endif`, or `#pragma once`) | Prevents "redefinition" errors when a header is included twice. `#pragma once` is not in the standard but supported by every major compiler. |
| Put **declarations** in headers, **definitions** in source files | A function defined in a header and included in two files is defined twice, and the linker complains |
| Mark file-private helpers `static` | Avoids name clashes between files |
| Link extra libraries with `-l` (e.g. `-lm` for `<math.h>` on Linux) | The standard math library is a separate library on many systems |

### Automating the build: Make

Typing compiler commands doesn't scale. **Make** rebuilds only what changed. Save this as `Makefile` (**recipe lines must start with a Tab character, not spaces**):

```make
CC      = gcc
CFLAGS  = -std=c17 -Wall -Wextra -Wpedantic -g

app: main.o math_utils.o
	$(CC) $(CFLAGS) $^ -o $@

%.o: %.c math_utils.h
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f app *.o
```

Run `make` to build and `make clean` to remove the outputs.

### Automating the build: CMake

For larger or cross-platform projects, **CMake** generates the native build files (Makefiles, Ninja, Visual Studio projects) from one description:

```cmake
cmake_minimum_required(VERSION 3.16)
project(MyApp C)

set(CMAKE_C_STANDARD 17)
set(CMAKE_C_STANDARD_REQUIRED ON)

add_executable(app main.c math_utils.c)

if(MSVC)
  target_compile_options(app PRIVATE /W4)
else()
  target_compile_options(app PRIVATE -Wall -Wextra -Wpedantic)
endif()
```

```bash
cmake -S . -B build
cmake --build build
```

### Mixing C and C++: `extern "C"`

C++ encodes parameter types into function names (**name mangling**) so overloads can coexist; C does not. To call C code from C++ (or expose C++ code to C), the declarations must be marked with C linkage. The standard header pattern is:

```c
#ifndef MATH_UTILS_H
#define MATH_UTILS_H

#ifdef __cplusplus
extern "C" {                /* only the C++ compiler sees this */
#endif

int add(int a, int b);
int square(int x);

#ifdef __cplusplus
}
#endif

#endif /* MATH_UTILS_H */
```

---

## 11. C++ Essentials — What Changes

> **In plain English:** everything in sections 1-10 still works in C++. This section covers what C++ *adds* so that you write less code, make fewer memory mistakes, and still run just as fast. Compile with `g++ -std=c++20 -Wall -Wextra -Wpedantic`.

### 11.1 References

A **reference** is an alias for an existing variable. Unlike a pointer, it can't be null and can't be re-pointed elsewhere after initialization.

```cpp
int x = 10;
int &r = x;        // r is another name for x
r = 20;            // x is now 20

void print(const std::string &s);   // pass big objects by const reference:
                                    // no copy, and the function can't modify them
```

### 11.2 Classes and RAII

A **class** bundles data with the functions that work on it. **Constructors** set an object up; the **destructor** cleans it up. Combined with RAII ([section 9](#9-dynamic-memory)), this makes resource handling automatic and exception-safe:

```cpp
#include <cstdio>
#include <stdexcept>

class File {
public:
    File(const char *path, const char *mode) : fp_(std::fopen(path, mode))
    {
        if (!fp_) throw std::runtime_error("cannot open file");
    }
    ~File() { if (fp_) std::fclose(fp_); }            // always runs when File goes out of scope

    File(const File &) = delete;                      // forbid copying: two owners would
    File &operator=(const File &) = delete;           // close the same FILE twice

    std::FILE *get() const { return fp_; }

private:
    std::FILE *fp_;
};

int main()
{
    File out("hello.txt", "w");
    std::fputs("written safely\n", out.get());
}   // file closed here, even if an exception had been thrown above
```

| Keyword | Meaning |
|---|---|
| `public` / `private` / `protected` | Who may access a member (`struct` defaults to public, `class` to private) |
| `explicit` | A one-argument constructor can't be used for silent implicit conversions |
| `const` after a method | The method promises not to modify the object |
| `= delete` / `= default` | Forbid / explicitly request a compiler-generated special function |
| `override` | Tells the compiler this method must override a virtual one (catches typos) |
| `virtual` | Enables run-time polymorphism through base-class pointers/references |

### 11.3 The standard library: containers, strings, algorithms

You rarely need to write your own dynamic array, string or sorted map. The standard library gives you tested, fast ones:

| Need | Use | Header |
|---|---|---|
| Text | `std::string` | `<string>` |
| Growable array | `std::vector<T>` | `<vector>` |
| Fixed-size array | `std::array<T, N>` | `<array>` |
| Sorted key → value | `std::map<K, V>` | `<map>` |
| Hashed key → value | `std::unordered_map<K, V>` | `<unordered_map>` |
| Sort, search, count, transform... | `std::sort`, `std::find`, `std::count_if`, ... | `<algorithm>` |

```cpp
#include <algorithm>
#include <iostream>
#include <map>
#include <string>
#include <utility>
#include <vector>

int main()
{
    std::vector<std::string> words = {"pear", "apple", "fig", "apple", "pear", "apple"};

    std::map<std::string, int> counts;                 // sorted by key
    for (const auto &w : words)                        // range-based for: no manual indices
        ++counts[w];                                   // a missing key starts at 0

    std::vector<std::pair<std::string, int>> ranked(counts.begin(), counts.end());
    std::sort(ranked.begin(), ranked.end(),
              [](const auto &a, const auto &b) { return a.second > b.second; });   // lambda

    for (const auto &[word, n] : ranked)               // structured bindings (C++17)
        std::cout << word << ": " << n << '\n';
}
```

The same program in C would need a hand-written hash table or sorted array, string comparison, and manual memory management. That difference is the core reason C++ exists.

> **⚠ Important:** `vector[i]` and `string[i]` do **not** check bounds (just like C arrays). `.at(i)` does, and throws `std::out_of_range` when `i` is invalid.

### 11.4 Templates: code that works for any type

A **template** is a recipe the compiler instantiates for each type you use. It is how the standard containers are written, and it costs nothing at run time:

```cpp
#include <iostream>
#include <string>

template <typename T>
T maximum(const T &a, const T &b) { return (a < b) ? b : a; }

int main()
{
    std::cout << maximum(3, 7) << '\n';                        // T deduced as int
    std::cout << maximum(2.5, 1.5) << '\n';                    // T deduced as double
    std::cout << maximum<std::string>("abc", "abd") << '\n';   // T given explicitly
}
```

> **Deep dive:** templates are checked when instantiated, which historically produced pages of error messages. **Concepts** (C++20) let you state requirements (`template <std::totally_ordered T>`) and get readable diagnostics instead.

### 11.5 Exceptions

```cpp
#include <iostream>
#include <stdexcept>
#include <vector>

int main()
{
    std::vector<int> v = {1, 2, 3};
    try {
        std::cout << v.at(10) << '\n';               // at() checks the index and throws
    } catch (const std::out_of_range &e) {
        std::cout << "caught: " << e.what() << '\n';
    }
}
```

When an exception is thrown, C++ **unwinds the stack**, running the destructor of every object on the way. This is exactly why RAII makes cleanup reliable. Never let an exception escape from a destructor.

### 11.6 Copying, moving, and the Rule of Zero / Three / Five

Copying a large object is expensive. C++11 introduced **move semantics**: if the source is a temporary (or you mark it with `std::move`), the destination can *steal* its resources instead of copying them, as with returning a `std::vector` from a function at no real cost.

| Rule | Statement |
|---|---|
| **Rule of Zero** | Build your classes from members that already manage themselves (`std::string`, `std::vector`, smart pointers) and write **none** of the special functions below. This is the goal. |
| **Rule of Three** | If you write a destructor, copy constructor, or copy assignment, you almost certainly need all three. |
| **Rule of Five** | Add the move constructor and move assignment to the three above. |

### 11.7 Modern C++ at a glance

| Feature | Since | What it gives you |
|---|---|---|
| `auto`, range-based `for`, lambdas, `nullptr`, `constexpr`, `enum class`, `override`, smart pointers, move semantics, `std::thread` | **C++11** | The "modern C++" baseline |
| `std::make_unique`, generic lambdas, digit separators (`1'000'000`) | **C++14** | Polish on C++11 |
| Structured bindings, `std::optional`, `std::variant`, `std::string_view`, `std::filesystem`, `if constexpr`, inline variables | **C++17** | Everyday conveniences |
| Concepts, ranges, `std::span`, `<=>`, coroutines, modules, `std::format`, `std::bit_cast` | **C++20** | Major language additions |
| `std::expected`, `std::print`, `std::mdspan`, `import std;` | **C++23** | Error handling and I/O refinements |

---

## 12. C or C++? Side by Side

| Task | The C way | The C++ way |
|---|---|---|
| Print | `printf("%d\n", n);` | `std::cout << n << '\n';` |
| Text | `char buf[N]` + `snprintf` / `strlen` | `std::string` |
| Growable array | `malloc` + `realloc` + `free` + a size variable | `std::vector<T>` |
| Single object on the heap | `malloc(sizeof *p)` ... `free(p)` | `std::make_unique<T>()` |
| Modify caller's variable | Pointer parameter | Reference parameter |
| Generic code | `void *` + callbacks, or macros, or `_Generic` | Templates |
| Error handling | Return codes, `errno` | Exceptions (or return codes / `std::expected`) |
| Avoid name clashes | Prefix every name (`mylib_init`) | `namespace` |
| Release resources | Manual, ideally one cleanup block | Destructors (RAII) |
| Constant | `#define N 10` or `const int` | `constexpr int N = 10;` |
| Null pointer | `NULL` (`nullptr` in C23) | `nullptr` |

> **✔ Tip:** choosing C is rarely about speed (both compile to the same kind of machine code). It is about **simplicity, a tiny runtime, a stable ABI, or a target where only C is available**. Choose C++ when program size and complexity make abstractions and automatic cleanup worth having.

---

## 13. Undefined Behavior and Classic Pitfalls

> **In plain English:** most mistakes in other languages produce an error message. In C and C++, many mistakes produce **undefined behavior (UB)**: the language makes *no promise whatsoever* about what happens. The program might crash, might print garbage, might appear to work perfectly for months, and may behave differently when you switch the optimization level or the compiler. UB is the main reason C and C++ are considered dangerous, so it deserves its own section.

Three kinds of "unspecified" are worth telling apart:

| Term | Meaning | Example |
|---|---|---|
| **Implementation-defined** | Allowed to vary, but the compiler must document its choice | Size of `int`; whether `char` is signed |
| **Unspecified** | Several outcomes are allowed; no documentation required | Order in which function arguments are evaluated |
| **Undefined** | No requirements at all, anything may happen | Everything in the table below |

### Common sources of undefined behavior

| Cause | Example |
|---|---|
| Out-of-bounds array/pointer access | `int a[3]; a[3] = 0;` |
| Dereferencing `NULL` or a dangling pointer | `*p` after `free(p)`; returning `&local` |
| Signed integer overflow | `INT_MAX + 1` |
| Reading an uninitialized variable | `int x; printf("%d", x);` |
| Modifying a string literal | `char *s = "hi"; s[0] = 'H';` |
| Shifting by a negative amount or by ≥ the type's width | `1 << 32` for a 32-bit `int` |
| Violating strict aliasing | Reading a `float` through an `int *` |
| Data race (two threads, one write, no synchronization) | Unprotected shared counter |
| Falling off the end of a non-void function | A missing `return` (UB in C++ always; in C only if the caller uses the value) |
| Modifying a variable twice with no sequencing between | Rules differ between languages and standards: just don't |

> **Deep dive:** because UB "cannot happen" in a correct program, optimizers are allowed to *assume* it never does. A check like `if (x + 1 < x)` meant to detect signed overflow may be deleted entirely, since for signed `x` it is "always false". Code can seem to work in a debug build and break at `-O2`. This is not a compiler bug: the program was already wrong.

### Everyday pitfalls

| Pitfall | Why it bites | Fix |
|---|---|---|
| `if (x = 5)` | Assigns instead of comparing | `==`; compile with `-Wall` |
| `if (x == 5);` | The stray `;` makes the `if` body empty | No semicolon; `-Wall` warns |
| Missing `break` in `switch` | Execution falls through to the next case | Add `break`; use `-Wimplicit-fallthrough` |
| `sizeof(pointer)` instead of the array's size | Gives 4 or 8, not the array size | Pass the length explicitly |
| Off-by-one (`i <= n` instead of `i < n`) | Reads or writes one element too many | Prefer `for (i = 0; i < n; i++)` |
| `7 / 2` expecting `3.5` | Integer division truncates | `7 / 2.0` or `(double)7 / 2` |
| `0.1 + 0.2 == 0.3` | Binary floating point can't represent 0.1 exactly | Compare with a tolerance |
| `a == b` on two C strings | Compares addresses, not contents | `strcmp(a, b) == 0` |
| Macro arguments with side effects or operators | Macros do text substitution, not calls | Parenthesize everything, or use `static inline` / `constexpr` |

The macro trap deserves a demonstration:

```c
#include <stdio.h>

#define SQUARE_BAD(x)  x * x
#define SQUARE_OK(x)   ((x) * (x))

int main(void)
{
    printf("%d\n", SQUARE_BAD(1 + 2));   /* expands to 1 + 2 * 1 + 2 = 5, not 9 */
    printf("%d\n", SQUARE_OK(1 + 2));    /* 9 */
    return 0;
}
```

---

## 14. Debugging and Tooling

> **In plain English:** you don't have to find memory bugs by staring at code. Compilers and tools can catch most of them for you, usually in seconds. Use them from day one.

### Step 1: let the compiler warn you

```bash
gcc -std=c17   -Wall -Wextra -Wpedantic -g file.c        # GCC / Clang, C
g++ -std=c++20 -Wall -Wextra -Wpedantic -g file.cpp      # GCC / Clang, C++
cl /W4 /Zi file.cpp                                      # MSVC
```

`-g` adds debug information so debuggers can show your source lines and variable names.

### Step 2: use the sanitizers

**Sanitizers** instrument your program so that bugs are detected *at the moment they happen* and reported with a precise location. Take this deliberately broken program:

<!-- no-test -->
```c
#include <stdlib.h>

int main(void)
{
    int *a = malloc(3 * sizeof *a);
    a[3] = 1;                /* writes one element past the end */
    free(a);
    return 0;
}
```

```bash
gcc -g -fsanitize=address,undefined bug.c -o bug && ./bug
```

```
==12345==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x... 
WRITE of size 4 at 0x... thread T0
    #0 0x... in main bug.c:6
```

The report names the kind of bug, the operation, and the exact file and line.

| Flag | Detects |
|---|---|
| `-fsanitize=address` | Out-of-bounds access, use-after-free, double free, (on Linux) memory leaks |
| `-fsanitize=undefined` | Signed overflow, bad shifts, misaligned access, null dereference... |
| `-fsanitize=thread` | Data races (cannot be combined with `address`) |

MSVC supports AddressSanitizer too (`/fsanitize=address`). **Valgrind** (`valgrind ./app`, Linux/macOS) finds similar memory errors without recompiling, at the price of speed.

### Step 3: a debugger when you need to look inside

| `gdb` command | Does |
|---|---|
| `gdb ./app` | Start the debugger on a program built with `-g` |
| `break main` / `break file.c:42` | Set a breakpoint |
| `run` | Start the program |
| `next` / `step` | Run one line (`step` enters functions, `next` steps over) |
| `print x` | Show a variable's value |
| `backtrace` | Show the chain of function calls that led here |
| `continue` | Run until the next breakpoint |
| `quit` | Exit |

### Step 4: static analysis and formatting

- **clang-tidy** and **cppcheck** analyze your code without running it and flag suspicious patterns.
- **clang-format** applies one consistent code style automatically.

---

## 15. Language Standards: Which One to Use?

Both languages are defined by ISO standards and revised regularly. You choose one with the `-std=` flag. **Always set it explicitly**: compilers' defaults change between versions.

| C standard | Year | Headline additions | GCC / Clang flag |
|---|---|---|---|
| **C89 / C90** | 1989/90 | The original ANSI/ISO standard | `-std=c89` |
| **C99** | 1999 | `//` comments, declarations anywhere, `long long`, `<stdint.h>`, `<stdbool.h>`, VLAs, designated initializers, `inline` | `-std=c99` |
| **C11** | 2011 | `_Static_assert`, `_Generic`, `_Atomic`, optional threads, anonymous structs/unions | `-std=c11` |
| **C17** | 2018 | Defect fixes only, no new features | `-std=c17` |
| **C23** | 2024 | `nullptr`, `bool`/`true`/`false` keywords, `constexpr`, `typeof`, `#embed`, `[[attributes]]`, binary literals (`0b101`), `()` means `(void)` | `-std=c23` (`-std=c2x` on older compilers) |

| C++ standard | Year | Flag |
|---|---|---|
| **C++98 / C++03** | 1998 / 2003 | `-std=c++98` / `-std=c++03` |
| **C++11** | 2011 | `-std=c++11` |
| **C++14** | 2014 | `-std=c++14` |
| **C++17** | 2017 | `-std=c++17` |
| **C++20** | 2020 | `-std=c++20` |
| **C++23** | 2023 (published 2024) | `-std=c++23` |

(What each C++ version added is listed in [section 11.7](#117-modern-c-at-a-glance). New C++ revisions follow a roughly three-year cycle.)

**What to pick:** for a new C project use **C17**, or **C23** if all your compilers support it. For a new C++ project use **C++20**, or **C++17** when a toolchain is older. Stay on an older standard only when forced to by the platform.

To see which standard your compiler uses by default:

```bash
echo | gcc -dM -E - | grep __STDC_VERSION__              # e.g. 201710L = C17
echo | g++ -dM -E -x c++ - | grep __cplusplus             # e.g. 201703L = C++17
```

---

## 16. Quick Reference and Glossary

### Cheat sheet

```bash
# ── COMPILE ──────────────────────────────────────────────────────
gcc -std=c17   -Wall -Wextra -Wpedantic -g a.c -o a      # C
g++ -std=c++20 -Wall -Wextra -Wpedantic -g a.cpp -o a    # C++
gcc -c a.c                    # compile only → a.o
gcc a.o b.o -o app            # link
gcc a.c -lm -o a              # link the math library
gcc -g -fsanitize=address,undefined a.c -o a   # debug build with sanitizers
gcc -O2 a.c -o a              # optimize (release build)
```

```c
/* ── MEMORY (C) ─────────────────────────────────────────────── */
int *p = malloc(n * sizeof *p);  if (!p) { /* handle error */ }
free(p);  p = NULL;

/* ── MEMORY (C++) ───────────────────────────────────────────── */
auto up = std::make_unique<T>(args);        // one owner
auto sp = std::make_shared<T>(args);        // shared owners
std::vector<int> v(n);                      // dynamic array

/* ── SIZES ──────────────────────────────────────────────────── */
size_t n = sizeof arr / sizeof arr[0];      // elements in a true array
```

### Glossary

| Term | Definition |
|---|---|
| **ABI** | Application Binary Interface: the binary-level contract (calling convention, type layout, name mangling) between compiled pieces of code |
| **Address** | The number identifying a byte in memory |
| **Declaration / definition** | A declaration says a name exists and its type; a definition creates it (code or storage) |
| **Dangling pointer** | A pointer to memory that has been freed or has gone out of scope |
| **Heap / stack** | Memory you manage by hand / memory managed automatically per function call |
| **Header** | A file of declarations meant to be `#include`d |
| **Linker** | The tool that joins object files and libraries into one executable |
| **Macro** | A name replaced by text *before* compilation (`#define`) |
| **Namespace** | A named scope that prevents C++ names from clashing |
| **Null pointer** | A pointer that points to nothing (`NULL` / `nullptr`) |
| **Object file** | The compiled but not yet linked output for one source file (`.o` / `.obj`) |
| **ODR** | One Definition Rule: every entity may be defined only once in the whole program |
| **Pointer** | A variable that holds an address |
| **RAII** | Tying a resource's lifetime to an object's scope so cleanup is automatic |
| **Reference** | A C++ alias for an existing variable |
| **STL** | The Standard Template Library: C++ containers, iterators and algorithms |
| **Template** | A C++ recipe the compiler instantiates for each type used |
| **Translation unit** | One source file after preprocessing; the unit the compiler works on |
| **UB** | Undefined behavior: the language makes no guarantees at all |

---

*This guide covers C (up to C23) and C++ (up to C++23) as defined by the ISO standards and implemented by GCC, Clang and MSVC. When in doubt, the final authority is the language standard, and the most practical teacher is a compiler with warnings turned on.*
