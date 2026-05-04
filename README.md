# Fasm-Universal-Include-
Universal Include for FASM (by Tomasz Grysztar)
# UniInc: Advanced Assembly Logic & Syscall Framework

**UniInc** is a high-performance macro library designed to bridge the gap between low-level assembly and high-level language ergonomics. Built for **FASM** and **UASM**, it provides a robust set of tools for Linux x86_64 development, focusing on code readability, memory safety, and rapid syscall invocation[cite: 1].

## Key Components

### 1. Unified Linux Syscall Interface
Contains a complete and updated database of **x86_64 Linux syscalls**[cite: 1]. 
*   **Verbosity**: Uses clear naming conventions like `SYS_READ` and `SYS_MMAP`[cite: 1].
*   **Universal Wrapper**: The `unisys` macro automatically handles register clobbering and argument mapping (RDI, RSI, RDX, R10, R8, R9), allowing you to call the kernel in one line[cite: 1].

### 2. HLL Control Flow (High-Level Logic)
Writing complex logic in assembly often leads to "spaghetti code." UniInc solves this with preprocessor-level control structures[cite: 1]:
*   **Conditionals**: `.if`, `.elseif`, `.else` blocks with support for nested logic and complex expressions[cite: 1].
*   **Loops**: Familiar `.while` and `.repeat` loops[cite: 1].
*   **Optimized `.for`**: A specialized loop macro that automatically chooses between `inc/dec` or `add/sub` for the most efficient machine code generation[cite: 1].

### 3. Dynamic Memory & Heap Management
A built-in memory management sub-framework that implements essential allocation primitives[cite: 1]:
*   **Macros**: `kmalloc`, `kfree`, `kcalloc`, and `krealloc`[cite: 1].
*   **Safety**: Includes 16-byte alignment logic and meta-data tracking via `MemoryBlock` structures[cite: 1].

### 4. Advanced Data Modeling
*   **Typed Structures**: Support for `uint8_t` through `uint64_t` and `size_t`[cite: 1].
*   **Powerful Structs**: Advanced `struct` and `union` macros that support nesting and virtual offsets[cite: 1].

---

## Code Examples

### Structured Logic vs Raw Assembly
Instead of manual `cmp` and `jcc` jumps, you can write:
```assembly
.if eax > 100 & ebx == 0
    unisys SYS_WRITE, 1, msg_success, 12
.else
    unisys SYS_WRITE, 1, msg_error, 10
.endif
```
[cite: 1]

### Efficient Loops
The `.for` macro manages your counters and exit conditions automatically:
```assembly
; This generates optimized INC/DEC instructions for step 1
.for rsi, <, rdi, +, 1
    lodsb
    ; ... process byte ...
.endf
```
[cite: 1]

### Register Preservation
Quickly save and restore the entire 64-bit register state:
```assembly
preserve    ; Pushes RAX through R11 to the stack
call heavy_function
restore     ; Pops them back in reverse order
```
[cite: 1]

---

## Technical Details
*   **Platform**: Linux x86_64[cite: 1].
*   **Assembler Support**: FASM (Flat Assembler), UASM[cite: 1].
*   **File Entry Point**: `universal.inc`[cite: 1].

## Why Use This?
UniInc reduces the boilerplate code in assembly projects by up to 40% while maintaining 100% control over the generated binary. It’s ideal for developers writing performance-critical tools, compilers, or minimalist system utilities[cite: 1].

---

### License
GPLv3 License.
