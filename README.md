# Fasm-Universal-Include-
Universal Include for FASM (by Tomasz Grysztar)
# UniInc: Advanced Assembly Logic & Syscall Framework

**UniInc** is a high-performance macro library designed to bridge the gap between low-level assembly and high-level language ergonomics. Built for **FASM** and **UASM**, it provides a robust set of tools for Linux x86_64 development, focusing on code readability, memory safety, and rapid syscall invocation.

## Key Components

### 1. Unified Linux Syscall Interface
Contains a complete and updated database of **x86_64 Linux syscalls**]. 
*   **Verbosity**: Uses clear naming conventions like `SYS_READ` and `SYS_MMAP`.
*   **Universal Wrapper**: The `unisys` macro automatically handles register clobbering and argument mapping (RDI, RSI, RDX, R10, R8, R9), allowing you to call the kernel in one line.

### 2. HLL Control Flow (High-Level Logic)
Writing complex logic in assembly often leads to "spaghetti code." UniInc solves this with preprocessor-level control structures:
*   **Conditionals**: `.if`, `.elseif`, `.else` blocks with support for nested logic and complex expressions.
*   **Loops**: Familiar `.while` and `.repeat` loops.
*   **Optimized `.for`**: A specialized loop macro that automatically chooses between `inc/dec` or `add/sub` for the most efficient machine code generation.

### 3. Dynamic Memory & Heap Management
A built-in memory management sub-framework that implements essential allocation primitives:
*   **Macros**: `kmalloc`, `kfree`, `kcalloc`, and `krealloc`.
*   **Safety**: Includes 16-byte alignment logic and meta-data tracking via `MemoryBlock` structures.

### 4. Advanced Data Modeling
*   **Typed Structures**: Support for `uint8_t` through `uint64_t` and `size_t`.
*   **Powerful Structs**: Advanced `struct` and `union` macros that support nesting and virtual offsets.

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


### Efficient Loops
The `.for` macro manages your counters and exit conditions automatically:
```assembly
; This generates optimized INC/DEC instructions for step 1
.for rsi, <, rdi, +, 1
    lodsb
    ; ... process byte ...
.endf
```


### Register Preservation
Quickly save and restore the entire 64-bit register state:
```assembly
preserve    ; Pushes RAX through R11 to the stack
call heavy_function
restore     ; Pops them back in reverse order
```


---

## Technical Details
*   **Platform**: Linux x86_64.
*   **Assembler Support**: FASM (Flat Assembler), UASM.
*   **File Entry Point**: `universal.inc`.

## Why Use This?
UniInc reduces the boilerplate code in assembly projects by up to 40% while maintaining 100% control over the generated binary. It’s ideal for developers writing performance-critical tools, compilers, or minimalist system utilities.

---

### License
GPLv3 License.
