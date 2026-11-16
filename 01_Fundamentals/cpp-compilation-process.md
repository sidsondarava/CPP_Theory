# C++ Compilation Process: From `.cpp` File to Executable

## 1. Objective

Understand the complete journey of a C++ source file (`.cpp`) until it becomes an executable file.

We will understand:

- How C++ source code is converted into machine code.
- What happens during preprocessing, compilation, assembly and linking.
- What an object file contains.
- How the linker resolves symbols and relocates addresses.
- Difference between static and dynamic linking.
- How the operating system loads and executes an executable.
- What happens internally when we execute a program in Linux.

**Environment:** Ubuntu 24 + GCC/G++ + x86-64 architecture.

---

# 2. Overview of the Compilation Process

When we write a C++ program, the compiler does not directly convert the `.cpp` file into an executable.

The process consists of multiple stages:

1. Preprocessing
2. Compilation
3. Assembly
4. Linking
5. Loading
6. Execution

### Overall Flow

```text
                  main.cpp
                     |
                     |
              PREPROCESSOR
                     |
                     v
                  main.i
             (Expanded source)
                     |
                     |
                  COMPILER
                     |
                     v
                  main.s
              (Assembly code)
                     |
                     |
                  ASSEMBLER
                     |
                     v
                  main.o
                (Object file)
                     |
                     |
                   LINKER
                     |
                     v
                 application
                 (Executable)
                     |
                     |
                   LOADER
                     |
                     v
                  EXECUTION
```

### Generated Files

| Stage | Input | Output | Description |
|---|---|---|---|
| Preprocessing | `.cpp` | `.i` | Expanded source code |
| Compilation | `.i` | `.s` | Assembly code |
| Assembly | `.s` | `.o` | Object file |
| Linking | `.o` + libraries | Executable | Final executable |
| Loading | Executable | Process | Program loaded into memory |
| Execution | Process | Output | Instructions executed by CPU |

**Important:** Preprocessing, compilation, assembly and linking are build-time activities. Loading and execution happen at runtime.

---

# 3. Stage 1: Preprocessing

**Tool:** C++ Preprocessor (`cpp`)

The preprocessor handles all preprocessor directives before actual C++ compilation begins.

Preprocessor directives start with `#`.

Examples:

```cpp
#include <iostream>
#define MAX_SIZE 100

#ifdef DEBUG
#endif
```

### Main Responsibilities

#### 3.1 Header File Inclusion

Example:

```cpp
#include <iostream>
```

The preprocessor finds the `iostream` header and processes its contents into the translation unit.

Conceptually:

```cpp
// Declarations and other contents from iostream
// Plus its included dependencies

int main()
{
    std::cout << "Hello World";
}
```

The actual expanded content is much larger because standard library headers include many other headers and declarations.

**Why is this required?**

The compiler needs declarations of functions, classes, templates and other entities used in our source code.

#### 3.2 Macro Expansion

Example:

```cpp
#define MAX_SIZE 100

int main()
{
    int arr[MAX_SIZE];
}
```

After preprocessing:

```cpp
int main()
{
    int arr[100];
}
```

The preprocessor replaces macro occurrences with their definitions.

#### 3.3 Conditional Compilation

Example:

```cpp
#define DEBUG

#ifdef DEBUG
    // Debugging code
    printf("Debug mode");
#endif
```

If `DEBUG` is defined, the code inside `#ifdef` is retained.

Otherwise, it is excluded from the preprocessed output.

### Practical Demonstration

```bash
g++ -E main.cpp -o main.i
```

Explanation:

- `g++`: GNU C++ compiler driver.
- `-E`: Stop after preprocessing.
- `main.cpp`: Input source file.
- `-o main.i`: Save preprocessed output.

Inspect the output:

```bash
less main.i
```

**Generated file:** `main.i`

### Key Takeaways

- Preprocessing happens before actual compilation.
- Header files are processed and included.
- Macros are expanded.
- Conditional compilation directives are evaluated.
- The resulting source is used for compilation.

---

# 4. Stage 2: Compilation

**Tool:** C++ Compiler (GCC uses `cc1plus`)

The compiler converts preprocessed C++ source code into assembly code.

Compilation involves several internal stages.

## 4.1 Lexical Analysis (Tokenization)

The compiler reads the source code and breaks it into individual tokens.

Example:

```cpp
int sum = a + b;
```

Tokens:

| Token | Category |
|---|---|
| `int` | Keyword |
| `sum` | Identifier |
| `=` | Operator |
| `a` | Identifier |
| `+` | Operator |
| `b` | Identifier |
| `;` | Punctuation |

The compiler identifies individual components of the source code.

## 4.2 Syntax Analysis (Parsing)

The compiler checks whether the tokens follow C++ grammar rules.

Example:

Valid:

```cpp
int main()
{
    int a = 10;
    return a;
}
```

Invalid:

```cpp
int main()
{
    int a = 10
    return a;
}
```

The second example contains a missing semicolon.

During parsing, the compiler constructs an Abstract Syntax Tree (AST).

Example:

```cpp
int result = a + b;
```

Simplified AST:

```text
       Assignment
        /      \
   result       +
               / \
              a   b
```

The AST represents the structural relationship between different elements of the program.

## 4.3 Semantic Analysis

The compiler checks whether the program makes sense according to C++ language rules.

Example:

```cpp
int a = 10;
int* ptr = a;
```

The syntax is valid, but the assignment is semantically invalid.

Semantic analysis includes:

- Type checking.
- Scope resolution.
- Name lookup.
- Access control.
- Function overload resolution.
- Template instantiation.

## 4.4 Intermediate Representation (IR)

The compiler converts the program into an intermediate representation.

GCC uses:

- GIMPLE
- RTL (Register Transfer Language)

Intermediate representations make optimization easier.

They separate the original C++ source syntax from the target machine architecture.

## 4.5 Optimization

The compiler optimizes the intermediate representation based on optimization flags.

Example:

```cpp
int calculate()
{
    int a = 10;
    int b = 20;

    return a + b;
}
```

The compiler may simplify this to effectively return the constant `30`.

Common optimization flags:

| Flag | Description |
|---|---|
| `-O0` | No optimization |
| `-O1` | Basic optimization |
| `-O2` | More extensive optimization |
| `-O3` | More aggressive optimization |
| `-Os` | Optimize for code size |

## 4.6 Code Generation

Finally, the compiler backend generates assembly instructions for the target architecture.

For example, if the target is x86-64, the compiler generates x86-64 assembly.

Simplified example:

```asm
main:
    mov eax, 0
    ret
```

Actual assembly depends on the source code, compiler version, optimization level and target architecture.

### Practical Demonstration

```bash
g++ -S main.cpp -o main.s
```

Explanation:

- `-S`: Compile but stop before assembly.
- `main.cpp`: Input source.
- `main.s`: Generated assembly file.

Inspect:

```bash
less main.s
```

**Generated file:** `main.s`

### Key Takeaways

- Compiler checks syntax and semantics.
- AST represents the program structure.
- Intermediate representations help perform optimizations.
- Compiler generates architecture-specific assembly.
- Compilation errors are detected during this stage.

---

# 5. Stage 3: Assembly

**Tool:** Assembler (`as`)

The assembler converts assembly instructions into machine code.

Difference:

- Assembly language: Human-readable representation of machine instructions.
- Machine code: Binary instructions understood by the processor.

Example assembly:

```asm
mov eax, 10
add eax, 20
```

The assembler converts these instructions into corresponding machine-code bytes.

## 5.1 What is an Object File?

The assembler generates an object file.

Linux:

```text
main.o
```

Windows:

```text
main.obj
```

An object file contains more than machine code.

It generally contains:

1. Machine code.
2. Data sections.
3. Symbol table.
4. Relocation information.
5. Debugging information (if generated).

### A. Machine Code

Contains the actual instructions generated for functions.

### B. Symbol Table

Contains information about symbols defined or referenced in the object file.

Example:

```cpp
void display()
{
    // Implementation
}
```

The object file contains a symbol corresponding to `display`.

The linker uses symbol information to identify functions and variables across object files.

### C. Relocation Information

Consider:

```cpp
// main.cpp

void display();

int main()
{
    display();
}
```

At compilation time, the compiler knows that `display()` is declared.

However, it may not know its final memory address.

The object file therefore contains relocation information that helps the linker resolve the address later.

### Practical Demonstration

```bash
g++ -c main.cpp -o main.o
```

Explanation:

- `-c`: Compile and assemble, but don't link.
- `main.cpp`: Input source.
- `main.o`: Generated object file.

Inspect:

```bash
file main.o
```

Example output:

```text
main.o: ELF 64-bit LSB relocatable, x86-64
```

Inspect symbols:

```bash
nm main.o
```

Inspect sections:

```bash
readelf -S main.o
```

**Important:** An object file is not a complete executable. It is a relocatable compiled file.

### Key Takeaways

- Assembler converts assembly into machine code.
- Object files contain machine code, symbols and relocation information.
- Multiple object files can be linked together.
- An object file is not necessarily independently executable.

---

# 6. Stage 4: Linking

**Tool:** Linker (`ld`)

The linker combines object files and libraries to generate the final executable.

## 6.1 Why Do We Need a Linker?

Consider a project containing two source files.

### main.cpp

```cpp
#include <iostream>

void display();

int main()
{
    display();
    return 0;
}
```

### display.cpp

```cpp
#include <iostream>

void display()
{
    std::cout << "Hello from display()" << std::endl;
}
```

Here:

- `main.cpp` calls `display()`.
- `display.cpp` contains the implementation of `display()`.

When compiling `main.cpp`, the compiler knows the declaration of `display()`.

But its implementation is in another source file.

The linker connects these separately compiled parts.

## 6.2 Compile Source Files Separately

```bash
g++ -c main.cpp -o main.o

g++ -c display.cpp -o display.o
```

Generated files:

```text
main.o
display.o
```

## 6.3 Link Object Files

```bash
g++ main.o display.o -o application
```

The linker performs several important operations.

### A. Symbol Resolution

The linker identifies symbols that are defined and symbols that are referenced.

Example:

| Object file | Symbol | Status |
|---|---|---|
| main.o | main | Defined |
| main.o | display | Undefined reference |
| display.o | display | Defined |

The linker matches the reference to `display()` in `main.o` with its definition in `display.o`.

If a required symbol cannot be found, linking fails.

Example:

```text
undefined reference to `display()'
```

This is a linker error, not a compiler syntax error.

### B. Relocation

During compilation, the compiler may not know the final address of a function or variable.

During linking, the linker determines appropriate addresses and updates relevant references.

This process is called relocation.

### C. Library Linking

C++ programs commonly depend on libraries.

For example:

```cpp
std::cout << "Hello World";
```

The C++ standard library provides the implementation of these facilities.

Common GCC runtime libraries include:

| Library | Purpose |
|---|---|
| libstdc++ | C++ standard library |
| libc | C standard library |
| libgcc | GCC runtime support |

Some library dependencies are linked statically, while others are dynamically linked.

### D. Executable Generation

After symbol resolution and relocation, the linker generates the final executable.

Linux:

```text
application
```

Windows:

```text
application.exe
```

### Practical Demonstration

```bash
./application
```

Output:

```text
Hello from display()
```

### Key Takeaways

- Linker combines object files and libraries.
- Symbol resolution connects declarations/references with definitions.
- Relocation adjusts addresses.
- Linker errors occur when required symbols cannot be resolved.
- The linker generates the final executable.

---

# 7. Static Linking vs Dynamic Linking

When a program depends on external libraries, they can be linked in different ways.

## 7.1 Static Linking

In static linking, the required library code is incorporated into the executable during linking.

Conceptually:

```text
          application
               |
       -----------------
       |       |       |
     main.o display.o Library Code
```

Advantages:

- Fewer runtime library dependencies.
- Easier deployment when required shared libraries aren't installed.
- Library implementation is included in the executable.

Disadvantages:

- Larger executable size.
- Library updates generally require rebuilding the executable.

## 7.2 Dynamic Linking

In dynamic linking, the executable refers to shared libraries that are loaded at runtime.

Example:

```text
              application
                   |
          Dynamic Dependencies
                   |
          ---------------------
          |         |         |
       libstdc++   libc    libother.so
```

The executable does not contain all the implementation code of these shared libraries.

Instead, it contains dependency information and references to symbols.

Advantages:

- Smaller executable.
- Multiple applications can share libraries in memory.
- Libraries can be updated independently, subject to compatibility.

Disadvantages:

- Required shared libraries must be available at runtime.
- Version incompatibilities can cause runtime failures.

### Practical Demonstration

Check dependencies:

```bash
ldd application
```

Example output:

```text
linux-vdso.so.1
libstdc++.so.6
libc.so.6
libm.so.6
libgcc_s.so.1
```

Actual output depends on your system and build configuration.

Inspect ELF dynamic information:

```bash
readelf -d application
```

Look for:

```text
NEEDED
```

These entries indicate required shared libraries.

### Key Difference

| Static Linking | Dynamic Linking |
|---|---|
| Library code incorporated into executable | Library code loaded from shared libraries |
| Usually larger executable | Usually smaller executable |
| Fewer runtime library dependencies | Requires compatible shared libraries |
| Library updates generally require relinking | Libraries can be updated independently |

---

# 8. Stage 5: Loading

**Component:** Operating System Loader

After linking, we have an executable.

But how does the operating system execute it?

When we execute:

```bash
./application
```

The shell asks the operating system to execute the program.

On Linux, this involves the `execve()` system call.

Conceptually:

```cpp
execve("./application", argv, envp);
```

## 8.1 What is ELF?

ELF stands for Executable and Linkable Format.

It is the standard binary format used by Linux for:

- Executable files.
- Object files.
- Shared libraries.

An ELF executable contains:

- ELF header.
- Program headers.
- Sections and metadata.
- Machine instructions.
- Dynamic linking information.
- Symbol information, when present.

Inspect the ELF header:

```bash
readelf -h application
```

Example information:

```text
Class:   ELF64
Data:    2's complement, little endian
Type:    DYN
Machine: Advanced Micro Devices X86-64
```

Note: Modern Linux executables are often PIE (Position Independent Executables), which typically appear as `DYN` rather than `EXEC`.

## 8.2 What Happens During Loading?

The Linux kernel:

1. Receives the execution request through `execve()`.
2. Validates the executable format.
3. Creates a new process execution environment.
4. Maps executable loadable segments into virtual memory.
5. Sets up the initial stack with arguments and environment information.
6. Identifies the program's entry point.
7. Transfers control to the program's startup code.

## 8.3 What About Shared Libraries?

If the executable depends on shared libraries, the kernel identifies the dynamic linker specified in the ELF interpreter section.

For a typical x86-64 Ubuntu system:

```text
/lib64/ld-linux-x86-64.so.2
```

The dynamic linker:

1. Loads required shared libraries.
2. Resolves dynamic symbols.
3. Performs required relocations.
4. Transfers control to the program's startup code.

Inspect the interpreter:

```bash
readelf -l application
```

Look for:

```text
INTERP
```

### Key Takeaways

- Linux uses ELF as its executable format.
- `execve()` requests execution of a program.
- The kernel maps executable segments into virtual memory.
- The dynamic linker loads required shared libraries.
- Execution starts from the executable's entry point.

---

# 9. Stage 6: Execution

Now the program is loaded into memory.

Does execution start directly from `main()`?

**No. Execution generally starts before `main()`.**

## 9.1 Entry Point

The ELF executable contains an entry-point address.

Inspect it:

```bash
readelf -h application
```

Look for:

```text
Entry point address
```

This address typically points to startup code provided by the C runtime, not directly to the `main()` function.

## 9.2 Startup Code

On a typical GCC/Linux application, the execution flow is conceptually:

```text
       Operating System
               |
               v
             _start
               |
               v
      Runtime initialization
               |
               v
      C++ initialization
               |
               v
             main()
               |
               v
        Return value
               |
               v
        Exit handling
               |
               v
       Process termination
```

Startup code performs operations such as:

- Setting up the runtime environment.
- Initializing global and static objects.
- Preparing arguments and environment.
- Calling `main()` through the C runtime startup machinery.

## 9.3 Main Function Execution

Now our application code executes:

```cpp
int main()
{
    std::cout << "Hello World" << std::endl;
    return 0;
}
```

The CPU executes the machine instructions corresponding to our program.

## 9.4 Process Termination

When `main()` returns:

```cpp
return 0;
```

The C runtime performs normal termination.

This includes:

- Calling registered termination handlers.
- Destroying static-duration objects.
- Flushing standard I/O streams.
- Terminating the process with the return status.

The operating system then reclaims the process's resources.

### Key Takeaways

- Execution does not normally start directly from `main()`.
- `_start` is the typical ELF entry point for a Linux C++ application.
- Runtime startup code initializes the application.
- `main()` executes after initialization.
- Returning from `main()` leads to normal program termination.

---

# 10. What Does G++ Actually Do?

`g++` is a compiler driver.

It coordinates multiple tools to complete the build process.

When we execute:

```bash
g++ main.cpp -o application
```

G++ internally coordinates approximately the following sequence:

```text
                  g++
                   |
                   v
              Preprocessor
                 (cpp)
                   |
                   v
             C++ Compiler
               (cc1plus)
                   |
                   v
               Assembler
                  (as)
                   |
                   v
              Object File
                (main.o)
                   |
                   v
                Linker
                  (ld)
                   |
                   v
               application
```

The actual toolchain may involve additional tools and intermediate steps.

### Important GCC Tools

| Tool | Responsibility |
|---|---|
| cpp | Preprocessing |
| cc1plus | C++ compilation |
| as | Assembly |
| ld | Linking |
| collect2 | GCC linker support, commonly used to coordinate linking |

### Inspect Internal Commands

```bash
g++ -v main.cpp -o application
```

This displays detailed information about the build process.

Another useful command:

```bash
g++ -### main.cpp -o application
```

The `-###` option displays the commands without executing them.

---

# 11. Understanding Translation Units

This is a fundamental C++ concept.

Consider this project:

```text
MyProject/
|
|-- main.cpp
|-- calculator.cpp
|-- calculator.h
```

### calculator.h

```cpp
#ifndef CALCULATOR_H
#define CALCULATOR_H

int add(int a, int b);

#endif
```

### calculator.cpp

```cpp
#include "calculator.h"

int add(int a, int b)
{
    return a + b;
}
```

### main.cpp

```cpp
#include <iostream>
#include "calculator.h"

int main()
{
    std::cout << add(10, 20);
    return 0;
}
```

## What Happens Internally?

Each `.cpp` file is independently preprocessed and compiled.

This creates two translation units.

Translation Unit 1:

```text
main.cpp
   |
   +-- calculator.h
   |
   v
Preprocessing
   |
   v
Compilation
   |
   v
main.o
```

Translation Unit 2:

```text
calculator.cpp
   |
   +-- calculator.h
   |
   v
Preprocessing
   |
   v
Compilation
   |
   v
calculator.o
```

Finally:

```text
main.o + calculator.o
          |
          v
        Linker
          |
          v
      application
```

**Definition:**

A translation unit is essentially the source file after preprocessing, including all the content introduced through header inclusion.

### Why Is This Important?

C++ supports separate compilation.

If we modify `calculator.cpp`, we generally only need to recompile that translation unit and link the resulting object file with the other object files.

This is fundamental to large C++ projects and build systems such as CMake.

---

# 12. Complete Practical Experiment on Ubuntu

Let's perform all stages practically.

## Step 1: Create Project

```bash
mkdir cpp-build-demo
cd cpp-build-demo
```

Create:

```text
cpp-build-demo/
|
|-- main.cpp
|-- calculator.cpp
|-- calculator.h
```

Use the source code from Section 11.

## Step 2: Preprocessing

```bash
g++ -E main.cpp -o main.i
```

Output:

```text
main.i
```

## Step 3: Compilation

```bash
g++ -S main.cpp -o main.s
```

Output:

```text
main.s
```

## Step 4: Assembly

```bash
g++ -c main.cpp -o main.o
```

Output:

```text
main.o
```

Compile the second source file:

```bash
g++ -c calculator.cpp -o calculator.o
```

Output:

```text
calculator.o
```

## Step 5: Linking

```bash
g++ main.o calculator.o -o application
```

Output:

```text
application
```

## Step 6: Inspect Executable

```bash
file application
```

```bash
ldd application
```

```bash
readelf -h application
```

## Step 7: Execute

```bash
./application
```

Expected output:

```text
30
```

---

# 13. Compilation Error vs Linker Error vs Runtime Error

These three types of errors are frequently confused.

| Error Type | Example | Detected During |
|---|---|---|
| Compilation error | Missing semicolon, invalid type conversion | Compilation |
| Linker error | Undefined reference to `display()` | Linking |
| Runtime error | Segmentation fault | Execution |

### Example: Compilation Error

```cpp
int main()
{
    int a = 10
    return a;
}
```

Missing semicolon.

### Example: Linker Error

```cpp
void display();

int main()
{
    display();
}
```

If no definition of `display()` is provided, linking fails.

### Example: Runtime Error

```cpp
int main()
{
    int* ptr = nullptr;
    *ptr = 10;
}
```

Dereferencing a null pointer causes undefined behavior and commonly results in a segmentation fault.

---

# 14. Interview Questions

Try answering these without referring to the theory.

## Basic Understanding

1. What happens when we compile a C++ program?
2. What are the different stages of the compilation process?
3. What is the difference between a compiler and a linker?
4. What is preprocessing?
5. Why do we need header files?

## Compiler Internals

6. What is lexical analysis?
7. What is syntax analysis?
8. What is semantic analysis?
9. What is an Abstract Syntax Tree?
10. What is Intermediate Representation?
11. What is the difference between compilation and assembly?

## Object Files and Linking

12. What is an object file?
13. What information does an object file contain?
14. What is a translation unit?
15. What is symbol resolution?
16. What is relocation?
17. What happens when the linker encounters an undefined symbol?
18. What is name mangling?
19. What is the One Definition Rule?

## Linking and Loading

20. What is the difference between static and dynamic linking?
21. What is a shared library?
22. What is the difference between `.a` and `.so`?
23. What is ELF?
24. What happens when `execve()` is called?
25. What is the role of the dynamic linker?
26. What is the difference between ELF sections and segments?

## Execution

27. Does execution start directly from `main()`?
28. What is `_start`?
29. What happens before `main()` executes?
30. How are global and static objects initialized?
31. What happens when `main()` returns?

---

# 15. Key Takeaways

Remember these points:

1. **Preprocessing:** Handles header inclusion, macros and conditional compilation.

2. **Compilation:** Performs lexical analysis, parsing, semantic analysis, optimization and code generation.

3. **Assembly:** Converts assembly instructions into machine code and generates object files.

4. **Linking:** Combines object files and libraries, resolves symbols and performs relocation.

5. **Loading:** The operating system maps the executable and its required dependencies into the process address space.

6. **Execution:** Runtime startup code initializes the application, calls `main()`, and handles normal termination.

### Most Important Build Pipeline

```text
.cpp
 |
 v
Preprocessor
 |
 v
.i
 |
 v
Compiler
 |
 v
.s
 |
 v
Assembler
 |
 v
.o
 |
 v
Linker
 |
 v
Executable
 |
 v
OS Loader
 |
 v
Runtime Initialization
 |
 v
main()
 |
 v
Program Execution
```

**Remember:** The compiler translates C++ code into machine-specific instructions, the linker combines the required compiled components, and the operating system loads and executes the resulting program.

---

# 16. Further Practical Learning

Recommended practical exercises:

- Generate and inspect preprocessed output.
- Generate assembly and understand basic instructions.
- Inspect object files using `nm` and `readelf`.
- Observe undefined symbols in object files.
- Create a static library and link it to an application.
- Create a shared library and link it dynamically.
- Inspect shared library dependencies.
- Experiment with `-O0`, `-O2` and `-O3`.
- Explore ELF sections and program headers.
- Observe the entry point of an executable.

### Recommended Tools

| Tool | Purpose |
|---|---|
| GCC/G++ | Compilation |
| GDB | Debugging |
| GDB/objdump | Inspecting generated code |
| nm | Symbol inspection |
| readelf | ELF inspection |
| objdump | Disassembly |
| ldd | Shared library dependencies |
| CMake | Build system |

### References

1. GCC Documentation: https://gcc.gnu.org/onlinedocs/
2. ELF Specification: https://refspecs.linuxfoundation.org/elf/
3. Linux `execve()` Manual: https://man7.org/linux/man-pages/man2/execve.2.html
4. GNU Binutils Documentation: https://sourceware.org/binutils/docs/

---

**End of Notes**