Universal Include for FASM (by Tomasz Grysztar).
## Comprehensive Usage Example

Since the file contains macros for both the kernel (e.g., `vprint`, `isr_wrapper`) and Linux user-space (`SYS_WRITE`, `unisys`), it is impossible to meaningfully use all of them in a single executable context. Below is a user-space program example demonstrating the power of the HLL macros, structure manipulation, memory management, and system calls.

```assembly
include 'universal.inc'

; Defining a custom structure using type macros
struct UserData
    id      uint32_t ?
    active  uint8_t  ?
    buffer  size_t   ?
ends

section '.data' writeable
    dstr msg_start, "Processing started...", 10
    dstr msg_done,  "Processing finished.", 10
    dstr msg_err,   "Memory error!", 10
    
    heap_bottom dq ? ; Required for the mallc macro

section '.text' executable
    global _start

_start:
    ; 1. Using the universal system call macro
    unisys SYS_WRITE, 1, msg_start, 22
    
    ; 2. Preserving the register context
    preserve

    ; 3. Allocating memory via the wrapper (assumes a malloc function exists)
    kmalloc 1024
    
    ; 4. Using HLL branching (.if / .else)
    .if rax = 0
        unisys SYS_WRITE, 1, msg_err, 14
        jmp .exit
    .else
        mov rbx, rax ; Save the memory pointer
    .endif

    ; 5. Working with the structure
    mov [rbx + UserData.id], 100
    mov [rbx + UserData.active], 1

    ; 6. Using the advanced .for loop
    ; Counter initialization, condition, operation (+), step (1)
    mov rsi, 0
    .for rsi, <, 10, +, 1
        ; You can nest other HLL structures inside the loop
        .if [rbx + UserData.active] = 1
            ; Execute payload
            inc [rbx + UserData.id]
        .endif
    .endf

    ; 7. Freeing memory
    kfree rbx
    
    ; 8. Restoring registers
    restore

.exit:
    unisys SYS_WRITE, 1, msg_done, 21
    
    ; Exit system call (SYS_EXIT)
    unisys SYS_EXIT, 0, 0, 0, 0, 0, 0
```

---

## Deep Analysis of `uniinc.inc`

The provided file is a powerful header (include) file for FASM/UASM assemblers that transforms basic assembly into a high-level programming language. The file can be divided into five key subsystems.

### 1. Linux System Calls
*   The file contains a comprehensive list of x86_64 Linux syscall constants, ranging from `SYS_READ` (0) to `SYS_STATX` (332)[cite: 2].
*   The `unisys` macro provides an elegant wrapper for kernel calls: it takes the syscall number and up to six parameters, automatically distributing them across the `rax`, `rdi`, `rsi`, `rdx`, `r10`, `r8`, and `r9` registers before executing the `syscall` instruction.

### 2. High-Level Logic (HLL)
This is the most complex part of the preprocessor in the file, abstracting away routine `cmp` and `jcc` instructions.
*   **Branching:** Implements `.if`, `.elseif`, `.else`, and `.endif` constructs. They support a complex condition parsing tree (the `PARSECOND` macro), allowing the use of operators like `<`, `>`, `==` and compound expressions with `&` (AND) and `|` (OR).
*   **Loops:** Supports pre-condition loops (`.while` / `.endw`), post-condition loops (`.repeat` / `.until`), as well as a specialized `.for` loop.
*   The `.endf` macro (which closes a `.for` loop) contains an automatic optimization: if the loop step is 1, it generates efficient `inc` or `dec` instructions; otherwise, it falls back to `add` or `sub`.

### 3. Advanced Data Structures
The file redefines standard data allocation directives, adding robust support for complex nested structures and unions.
*   Standard C-like type aliases are introduced: `uint8_t` (db), `uint16_t` (dw), `uint32_t` (dd), `uint64_t`, and `size_t` (dq).
*   The `struct` and `ends` macro system tracks virtual offsets (`virtual at`), allowing it to dynamically calculate field sizes (`sizeof.#name`) and create nested data layouts.
*   For string creation, a convenient `dstr` macro is included, which automatically appends a null-terminator to the end of the defined string.

### 4. Memory Management
The code contains built-in logic for heap management operations.
*   It defines a `MemoryBlock` structure containing the block's size, an `is_free` flag, and a pointer to the `next` block in the chain.
*   The `mallc` macro implements a basic algorithm to find a free memory block by traversing a linked list starting from the `[heap_bottom]` address.
*   It includes high-level wrapper macros `kmalloc`, `kfree`, `kcalloc`, and `krealloc`. These prepare the registers (for example, aligning the allocation size to a 16-byte boundary using `add rcx, 15` and `and rcx, -16`) and call the respective underlying procedures.

### 5. Low-Level and Hardware Utilities
The file contains multiple helpers that are specifically useful when writing an OS kernel or bare-metal applications.
*   **Register Management:** Includes `preserve` and `restore` for quickly saving and restoring general-purpose registers to the stack (rax through r11), as well as `pushm` / `popm` for bulk pushing/popping arbitrary arguments.
*   **CPU Control:** Features macros like `halt` (disables interrupts and halts execution), `wait_irq` (enables interrupts and halts), `pcpu` (a repeating `nop` loop for exact delays), and `ULcpu` (controls the interrupt flag based on a passed parameter).
*   **TSC Reading:** The `read_tsc` macro is used for reading the processor's time-stamp counter, smoothly combining the `rdx` and `rax` registers using a bitwise shift.
*   **Procedures and Interrupts:** Macros `proc_start` and `proc_end` help manage the stack frame via the `rbp` register, while `isr_wrapper` is designed to safely wrap interrupt service routines.
*   **I/O Operations:** The `writectrl` macro sends a byte to port `0x20` (the PIC / interrupt controller), and `vprint` allows for direct writing of characters with color attributes to the VGA text mode video memory at address `0xB8000`.*
### 6. LICENSE
GPLv3 License.
