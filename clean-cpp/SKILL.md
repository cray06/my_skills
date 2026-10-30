---
name: clean-cpp
description: Rules for writing and reviewing clean, simple, readable C++ that is understood on first read. Use whenever writing, refactoring or reviewing C++ code (.cpp, .hpp), or when the user runs /clean-cpp on a file.
argument-hint: "[file or folder to review]"
---

# Clean C++

The goal is code a classmate understands on the first read: classes with clear names,
short functions, common standard library only, no clever tricks. When `$ARGUMENTS`
names files, review them against this checklist, list the violations, then fix them.

## Files and classes
- Sources live in `src/`, grouped by theme in sub-folders (for example `src/chess/`,
  `src/network/`, `src/training/`, `src/programs/`). Project includes are written from
  `src/`: `#include "chess/Position.hpp"`.
- Every `.hpp` declares exactly one class, and its `.cpp` implements it. Same name for the
  class and the files: `NeuralNetwork.hpp` / `NeuralNetwork.cpp`.
- No header made only of free functions, constants, structs or enums. Small data structs
  used by one class are nested inside that class.
- No namespaces at all: no project namespace, no anonymous namespace, never
  `using namespace std;`. Helpers are private member functions (`static` when they use no
  member).
- Headers start with `#pragma once` and contain declarations only.
- Include order: own header, project headers, standard headers, as one block with no
  blank line between them. Include what you use.
- `main` files only build the program object, run it and return the exit code.

## Naming
- Classes in `PascalCase`.
- Functions, methods, variables and parameters in `snake_case`: `load_network`,
  `learning_rate`, `batch_size`.
- Private members in `snake_case` with a leading underscore: `_weights`, `_learning_rate`.
- Constants are `static const` class members in `UPPER_SNAKE_CASE`, declared in the
  header and defined in the `.cpp`: `const double Trainer::DEFAULT_LEARNING_RATE = 0.001;`.
  No `constexpr`, no `k` prefix, no magic numbers in the code.
- Names describe the meaning: `random_engine`, `best_accuracy`, `square_count`.
  No abbreviations, no single letters except `i` / `j` in a short loop.

## Standard library: only the common parts
- Allowed: `string`, `vector`, `map`, `fstream`, `sstream`, `iostream`, `iomanip`,
  `cmath`, `cstdlib`, `algorithm`, `random`, `stdexcept` / `exception`, `csignal`.
- Avoid what a student would not recognise at first glance: `string_view`, `span`,
  `optional`, `variant`, `charconv` / `from_chars`, `format`, `filesystem`,
  `function`, `numeric` algorithms, structured bindings, designated initializers,
  attributes such as `[[nodiscard]]`.
- Random numbers: one `std::default_random_engine` member named `_random_engine`,
  used through `std::shuffle` and `std::normal_distribution`.
- Use explicit types; `auto` only for iterators.

## Functions
- One responsibility per function, ideally under 20 lines, never above 30.
- At most 4 parameters; beyond that, group values in a nested struct.
- Early returns instead of nested `if`; nesting depth at most 3.
- No boolean parameters that switch behaviour: write two functions.
- Loops: `for` and `while` only. No `do ... while`, no `goto`, no recursion unless the
  data is recursive.
- No ternary operator (`a ? b : c`): write an `if`.
- No lambdas when a named private method does the job.

## One action per line
- A line does one thing: one function call, or one calculation, or one comparison.
  Store intermediate results in named variables, one per line.
  ```cpp
  const std::size_t first_digit = fullmove.find_first_not_of('0');
  if (first_digit == std::string::npos) {
      throw Error("fullmove number must be a positive integer");
  }
  ```
  instead of `const bool is_zero = fullmove.find_first_not_of('0') == std::string::npos;`.
- Never store a comparison in a boolean variable to test it later: write the `if` where
  the decision is taken.
- An `if` condition compares simple values, or calls one function that returns a `bool`.
  Call other functions before the `if` and test the stored result. At most two simple
  conditions joined by `&&` / `||`; beyond that, use successive `if` statements.
  Plain accessors such as `size()` and `empty()` count as simple values.
- No nested calls such as `f(g(x))`: compute `g(x)` on its own line first.
- A `for` header only compares a counter with a limit; any other stop condition is an
  `if (...) { break; }` inside the loop. A `while` header may test one call that returns
  a `bool`, such as `while (std::getline(file, line))`.

## Types
- `const` by default for variables, parameters and methods that do not modify the object.
- Pass `std::string`, `std::vector` and objects by `const &`; return them by value.
- `std::size_t` for sizes and indices, `double` for maths.
- No enum unless it names a real concept of the subject that the reader already knows;
  never an enum that only exists to make a generic helper work.
- No raw `new` / `delete`, no owning raw pointers.

## Formatting
- 4 spaces, no tabs, lines under 100 characters.
- Function braces on their own line; control-statement braces on the same line, always
  present, even for one-line bodies.
- Never align continuation lines on an opening parenthesis: introduce named intermediate
  variables instead; if a call still does not fit, one 4-space hanging indent.
- One statement per line, one declaration per line, a blank line between functions.

## No comments
- No comments and no doc blocks. Names, small functions and types explain the code.
  If something needs a comment, rename or extract a method instead.

## Errors
- One `Error` class deriving from `std::exception`, thrown with a clear message.
- Never `exit()` outside `main`. `main` catches, prints to `std::cerr`, returns the code.

## Never
- Macros (except `#pragma once`), global variables (a `static` member is allowed only
  when the language requires it, such as a signal handler flag), C-style casts
  (use `static_cast`), copy-pasted blocks, dead code, commented-out code,
  `printf` / `malloc`.

## Build and check before finishing
- Compile with `-std=c++20 -Wall -Wextra -Werror -O2`: zero warnings.
- Plain Makefile: name variables, compiler and flags, one explicit `SRC` list per binary
  (one file per line), `OBJ = $(SRC:.cpp=.o)`, and the rules `all`, `clean`, `fclean`,
  `re`. No dependency-generation flags (`-MMD`, `-MP`, `DEPS`), no `$(dir ...)` tricks,
  no build directory: objects sit next to their sources and `clean` removes them.
- Run the program on a real case, then reread every file against this checklist and fix
  what does not match before reporting.
