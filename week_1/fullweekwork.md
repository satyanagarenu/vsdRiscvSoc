# Task 1: Install & Sanity-Check the Toolchain

## Objective
Install the RISC-V GNU toolchain, add it to PATH, and verify the installation.

## Commands Used
```bash
tar -xzf riscv-toolchain-rv32imac-x86_64-ubuntu.tar.gz
```



## Commands to inspect and view all the files inside the bin folder (binary files)
```bash
cd opt
cd riscv
ls bin
```



## Add PATH 
```bash
export PATH=$HOME/riscv/bin:$PATH
```

## To make PATH addition permanent
```bash
nano ~/.bashrc
export PATH=$HOME/riscv/bin:$PATH
source ~/.bashrc
echo $PATH
```
- 1. First open the ~/.bashrc for editing the Shell config file
- 2. Add the export line at the end of the config file
- 3. Save and exit: (In nano, press Ctrl+O, then Enter to save. Then Ctrl+X to exit.) 
- 4. Apply the changes by using source 
- 5. Verify by running the echo and checking if it includes /home/yourusername/riscv/bin.


## Verification Commands
```bash
riscv32-unknown-elf-gcc --version
riscv32-unknown-elf-gdb --version
riscv32-unknown-elf-objdump --version
```



- ✅ riscv32-unknown-elf-gcc --version → shows GCC 14.2.0 → Compiler is working
- ✅ riscv32-unknown-elf-objdump --version → shows Binutils 2.43.1 → Disassembler is working
- ✅ riscv32-unknown-elf-gdb --version → shows GDB 15.2 → Debugger is working

# Task 2: Compile “Hello, RISC-V”

## Objective
Compile a simple "Hello, RISC-V" program using the cross-compiler.

## C Program: helloworld.c
```bash
gedit helloworld.c
```
```c
#include <stdio.h>

int main() {
    printf("Hello, World!\n");
    return 0;
}

```
Save and close the .c file.


## Compilation Command
```bash
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -o helloworld.elf helloworld.c
```

### Explanation of Flags
| Flag                      | Meaning                                                       |
| ------------------------- | ------------------------------------------------------------- |
| `riscv32-unknown-elf-gcc` | RISC-V GCC compiler for 32-bit bare-metal targets (no OS)     |
| `-march=rv32imc`          | Target **RISC-V 32-bit** architecture with **IMC extensions** |
| `-mabi=ilp32`             | Use **ILP32** ABI (int, long, pointer = 32 bits)              |
| `helloworld.c`            | Your **C source file**                                        |
| `-o helloworld.elf`       | Name of the output **ELF executable**                         |

## ELF Verification
```bash
file helloworld.elf
```
The output confirms:

- The ELF is 32-bit
- It’s for the RISC-V architecture
- Uses the compressed instructions (RVC)
- Uses soft-float ABI(Application Binary Interface) (floating point handled in software)
- It’s statically linked




# Task 3: From C to Assembly

## Objective
Generate the RISC-V assembly (`.s`) file from the C source and understand the prologue/epilogue.

## Command Used
```bash
riscv32-unknown-elf-gcc -S -O0 helloworld.c
```

This creates `hello.s` containing the assembly code in the same directory.
!
## Contents of .s file
```bash
  	.file	"helloworld.c"
	.option nopic
	.attribute arch, "rv32i2p1_m2p0_a2p1_c2p0"
	.attribute unaligned_access, 0
	.attribute stack_align, 16
	.text
	.section	.rodata
	.align	2
.LC0:
	.string	"Hello, World!"
	.text
	.align	1
	.globl	main
	.type	main, @function
main:
	addi	sp,sp,-16
	sw	ra,12(sp)
	sw	s0,8(sp)
	addi	s0,sp,16
	lui	a5,%hi(.LC0)
	addi	a0,a5,%lo(.LC0)
	call	puts
	li	a5,0
	mv	a0,a5
	lw	ra,12(sp)
	lw	s0,8(sp)
	addi	sp,sp,16
	jr	ra
	.size	main, .-main
	.ident	"GCC: (g04696df096) 14.2.0"
	.section	.note.GNU-stack,"",@progbits
```

### What is the prologue and Epilogue?
The function prologue is the set of instructions at the beginning of a function that:
- Saves the return address (so the function can return properly),
- Saves any callee-saved registers it will use,
- Sets up the stack frame (allocates space on the stack),
- Prepares the environment for the function body.

In RISC-V assembly, a typical prologue includes instructions like:
```
    addi sp, sp, -16     # allocate stack space (adjust stack pointer)
    sw ra, 12(sp)        # save return address on stack
    sw s0, 8(sp)         # save frame pointer (s0) on stack
    addi s0, sp, 16      # set frame pointer to current stack pointer + offset
```
The function epilogue is the code at the end of the function that:
- Restores the saved registers,
- Restores the stack pointer to its original value,
- Returns control to the caller.
- Typical epilogue instructions:

```
lw ra, 12(sp)        # restore return address
lw s0, 8(sp)         # restore frame pointer
addi sp, sp, 16      # deallocate stack space (reset stack pointer)
ret                  # return to caller
```

# Task 4: Hex Dump & Disassembly

##  Objective
Convert the compiled ELF file into:
- A raw Intel HEX format file
- A human-readable disassembly using `objdump`

##  Commands Used

```bash
riscv32-unknown-elf-objcopy -O ihex hello.elf hello.hex 
riscv32-unknown-elf-objdump -d hello.elf > hello.asm
```
The first command is used for creating a raw HEX from the ELF file
The second command is to disassemble the HEX file using objdump to a .asm file in the same directory


### Output .asm file created by disassembly by objdump


## Breakdown of the instruction fields in main
| Column           | Meaning                                                     |
| ---------------- | ----------------------------------------------------------- |
| `0:`             | Offset address within the function or section               |
| `10162`          | Actual machine code in hex (the instruction in binary form) |
| `addi sp,sp,-16` | Human-readable disassembly of the machine code              |

Mnemonic
This is the operation name, like:
- addi (add immediate),
- lw (load word),
- sw (store word),
- jal (jump and link).
- It tells what the instruction does, just like function names in C.

Operands
These are the inputs and destination for the instruction:
- Registers (e.g., sp, a0, s0, etc.)
- Immediate values (constants, like -16)
- Addresses (for branches/jumps or memory access)

# Task 5: ABI and Register cheat sheet
## Objective
List all 32 RV32 integer registers with their:
- ABI Names - Application Binary Interface
- typical calling-convention roles

| Register | ABI Name | Description                          | Calling Convention Role |
|----------|----------|--------------------------------------|-------------------------|
| x0       | zero     | Hard-wired zero                      | Immutable constant zero |
| x1       | ra       | Return address                       | Caller-saved            |
| x2       | sp       | Stack pointer                        | Callee-saved            |
| x3       | gp       | Global pointer                       | Unallocatable           |
| x4       | tp       | Thread pointer                       | Unallocatable           |
| x5       | t0       | Temporary register 0                 | Caller-saved            |
| x6       | t1       | Temporary register 1                 | Caller-saved            |
| x7       | t2       | Temporary register 2                 | Caller-saved            |
| x8       | s0/fp    | Saved register 0 / Frame pointer     | Callee-saved            |
| x9       | s1       | Saved register 1                     | Callee-saved            |
| x10      | a0       | Function argument 0 / Return value 0 | Caller-saved            |
| x11      | a1       | Function argument 1 / Return value 1 | Caller-saved            |
| x12      | a2       | Function argument 2                  | Caller-saved            |
| x13      | a3       | Function argument 3                  | Caller-saved            |
| x14      | a4       | Function argument 4                  | Caller-saved            |
| x15      | a5       | Function argument 5                  | Caller-saved            |
| x16      | a6       | Function argument 6                  | Caller-saved            |
| x17      | a7       | Function argument 7                  | Caller-saved            |
| x18      | s2       | Saved register 2                     | Callee-saved            |
| x19      | s3       | Saved register 3                     | Callee-saved            |
| x20      | s4       | Saved register 4                     | Callee-saved            |
| x21      | s5       | Saved register 5                     | Callee-saved            |
| x22      | s6       | Saved register 6                     | Callee-saved            |
| x23      | s7       | Saved register 7                     | Callee-saved            |
| x24      | s8       | Saved register 8                     | Callee-saved            |
| x25      | s9       | Saved register 9                     | Callee-saved            |
| x26      | s10      | Saved register 10                    | Callee-saved            |
| x27      | s11      | Saved register 11                    | Callee-saved            |
| x28      | t3       | Temporary register 3                 | Caller-saved            |
| x29      | t4       | Temporary register 4                 | Caller-saved            |
| x30      | t5       | Temporary register 5                 | Caller-saved            |
| x31      | t6       | Temporary register 6                 | Caller-saved            |

# Task 6: Stepping with GDB 
## Objective
Debug a RISC-V ELF binary (`helloworld.elf`) using GDB, view the register contents, disassemble instructions, and step through the program using QEMU's GDB remote interface.

## Commands Used

### Step 1: Launch QEMU with GDB Server
```bash
qemu-riscv32 -g 1234 helloworld.elf
```
- Launches QEMU and halts execution.
- Enables a GDB server on port 1234, waiting for a debugger to connect.


| Component    | Explanation                                                             |
|--------------|-------------------------------------------------------------------------|
| qemu-riscv32 | This is the QEMU emulator for a generic 32-bit RISC-V CPU.              |
| -g 1234      | This tells QEMU to start in GDB debug mode and listen on TCP port 1234  |
| hello.elf    | This is the compiled RISC-V ELF binary                                  |


### Step 2: Launch GDB in another terminal
```bash
riscv32-unknown-elf-gdb helloworld.elf
```
- Starts the RISC-V version of GDB and loads the ELF with symbols.

### Step 3: Connect GDB to QEMU
```gdb
(gdb) target remote localhost:1234
```
- Establishes connection with QEMU.


### Step 4: View all the avaiable functions
```gdb
(gdb) info functions
```


### Step 5: Set Breakpoint at different functions (_start,main,exit) and Run
```gdb
(gdb) break _start
(gdb) continue
(gdb) break main
(gdb) continue
(gdb) break exit
(gdb) continue
```
- Execution halts at the corresponding function.

### Step 6: Step Through and Inspect
```gdb
(gdb) step
(gdb) info reg a0
(gdb) disas /r
```
- Step through the program.
- Inspect register contents (`a0`, `sp`, `ra`, etc.).
- Disassemble code to view human-readable assembly.



---
## Learnings
- Cross-compilation for RISC-V
- ELF generation with correct architecture
- QEMU emulation
- Remote debugging with GDB
- Proper program execution flow
The "Hello, World!" appearing in Terminal 1 is the definitive proof that the RISC-V program executed successfully. 
 
 
The cross-compilation chain worked perfectly:

- ✅ C source compiled to RV32IMC ELF
- ✅ ELF loads and runs on RISC-V emulation
- ✅ printf() function works correctly
- ✅ Program terminates normally with exit code 0
- ✅ GDB can debug the RISC-V binary remotely

# Task 7: Running Under an Emulator (QEMU)

## Objective
Execute a bare-metal RISC-V ELF binary using emulators (Spike and QEMU) to simulate real hardware execution and observe UART console output in a production-like environment.


## Creating a bare-metal .c file to print "Hello World"
```c
// Working Hello World bare-metal program 
#define UART_BASE 0x10000000
void _start(void) {
    // Initialize stack pointer - this is crucial!
    asm volatile ("li sp, 0x80400000");
    // Direct UART access - no function calls

    volatile unsigned char *uart = (volatile unsigned char *)UART_BASE;
    // Print "Hello, World!\n"
    char *msg1 = "Hello, World!\n";

    while (*msg1) {

        *uart = *msg1;

        msg1++;

    }
    // Print "This is bare-metal RISC-V!\n"

    char *msg2 = "This is bare-metal RISC-V!\n";

    while (*msg2) {

        *uart = *msg2;

        msg2++;

    }

    // Halt execution with infinite loop

    while (1) {
        asm volatile ("wfi");  // Wait for interrupt (saves power)
    }
}
```
## Creating a linker script - .ld

```c
ENTRY(_start)
MEMORY {

    RAM : ORIGIN = 0x80200000, LENGTH = 126M

}
SECTIONS {
    .text : {

        *(.text)

        *(.text.*)

    } > RAM
    .data : {

        *(.data)

        *(.data.*)
    } > RAM
    .bss : {

        *(.bss)

        *(.bss.*)

    } > RAM
}
```
## Why the above approach
A linker script (.ld file) is a plain-text configuration file used by the linker (ld) to control how your program's code and data are laid out in memory.

Why Do We Need It?
In embedded systems (like RISC-V development), we don't have an operating system to manage memory. So we must tell the linker:
- Where the code (like _start, main) should go
- Where to place data, bss, and stack
- How to organize segments in Flash (ROM) and RAM
- Without this, the program won't know where to load and run from.


### Why this .c file and not a general .c file with printf() statements?
- This is a bare metal code that directly writes characters to the memory-mapped UART (at address 0x10000000)
- Uses _start() as the entry point, not main()
- Does not use any standard C libraries
- Runs on hardware (or emulated hardware like QEMU) without an operating system
- However printf version: Needs a richer runtime, not suitable for truly bare-metal

### Why This Approach Matters:
- **Real-world relevance**: Mirrors actual embedded development workflow
- **Hardware understanding**: Demonstrates system-level programming skills
- **Testing methodology**: Shows production verification techniques
- **Progressive learning**: Builds upon previous debugging experience

---
## Commands Used

### Using QEMU System Emulator
```bash
riscv32-unknown-elf-gcc -march=rv32imac -mabi=ilp32 -nostdlib -nostartfiles -ffreestanding -T bm.ld -o hello_bm.elf hello_bm.c //for compiling and getting .elf file
qemu-system-riscv32 -nographic -machine virt -kernel hello_bm.elf

```
- `qemu-system-riscv32`: Full system emulation for RISC-V 32-bit
- `-nographic`: Disables graphical output, redirects console to terminal
- `-kernel hello_bm.elf`: Loads ELF as kernel image for bare-metal execution

---



## Key Learnings and Comparison

| Aspect              | Task 6 (GDB)           | Task 7 (Emulator)        |
|---------------------|------------------------|--------------------------|
| **Purpose**         | Debugging & Analysis   | Execution & Testing      |
| **Control**         | Interactive stepping   | Continuous execution     |
| **Visibility**      | Instruction-level      | Output-level only        |
| **Environment**     | Debug simulator        | Hardware simulation      |
| **Use Case**        | "What's happening?"    | "Does it work?"          |

# Task 8: Exploring GCC Optimisation 

## Objective
Observe the differences that appear in the .s files by using two types of gcc optimisation

## Commands Used
```bash
riscv32-unknown-elf-gcc -S -O0 helloworld.c -o heloworld_1.s
riscv32-unknown-elf-gcc -S -O2 helloworld.c -o heloworld_2.s
```

## Output .s files of -O0 and -O2 optimisation (LHS : -O0; RHS : -O2)


## Explanation of Differences

| Aspect                    | `-O0`                                  | `-O2`                                   |
| ------------------------- | -------------------------------------- | --------------------------------------- |
| **Prologue/Epilogue**     | Creates stack frame, saves/restores RA | No stack frame needed                   |
| **Temporary Variables**   | Stored in memory                       | Eliminated                              |
| **Loads/Stores**          | Multiple unnecessary memory accesses   | Removed via register usage              |
| **Function Overhead**     | Conservative; preserves state          | Aggressive optimization; minimal code   |
| **Dead Code Elimination** | Not done                               | Performed (eliminates unused variables) |

### Why these differences?
-O0 prioritizes debuggability:
  - Keeps variables in memory
  - Avoids instruction reordering
  - Easier to inspect with a debugger


-O2 focuses on execution speed and code size:
  - Eliminates redundant loads/stores
  - Removes unnecessary variables and stack usage
  - Performs constant folding, inlining, strength reduction, etc.

# Task 9: Inline Assembly Basics 

## Objective
Implement a C function to read the RISC-V cycle counter using inline assembly and provide detailed explanation of assembly constraints.

## Commands Used
```bash
riscv32-unknown-elf-gcc -march=rv32imac_zicsr -mabi=ilp32 -nostdlib -nostartfiles -T linker.ld -o inline.elf start.S inline.c
qemu-system-riscv32 -nographic -machine virt -kernel inline.elf
```

## A function that returns the cycle counter by reading CSR 0xC00 using inline asm
```c
static inline uint32_t rdcycle(void) {
    uint32_t c;
    asm volatile ("csrr %0, 0xC00" : "=r"(c));
    return c;
}
```
## A bare metal .c to print the cycle counter value 

```c

// rdcycle_std.c - RISC-V cycle counter with standard headers
#include <stdint.h>

#define UART_BASE 0x10000000

// RISC-V cycle counter function using CSR 0xC00
static inline uint32_t rdcycle(void) {
    uint32_t c;
    asm volatile ("csrr %0, 0xC00" : "=r"(c));
    return c;
}

// Simple UART output function
void uart_putc(char c) {
    volatile uint8_t *uart = (volatile uint8_t *)UART_BASE;
    *uart = c;
}

void uart_puts(const char *str) {
    while (*str) {
        uart_putc(*str);
        str++;
    }
}

void uart_puthex(uint32_t value) {
    char hex[] = "0123456789ABCDEF";
    uart_puts("0x");
    for (int i = 28; i >= 0; i -= 4) {
        uart_putc(hex[(value >> i) & 0xF]);
    }
}

int main(void) {
    uart_puts("Testing cycle counter...\n");
    
    // Read cycle counter using CSR 0xC00
    uint32_t cycle_value = rdcycle();
    
    uart_puts("Cycle read successful!\n");
    uart_puts("Cycle value: ");
    uart_puthex(cycle_value);
    uart_putc('\n');
    
    
    return 0;
}
```

## A startup file .S 
Since the linker expects an entry point symbol _start (by default), we write a minimal assembly start that sets up the stack pointer and calls main.
```c
    .section .text
    .globl _start
_start:
    la sp, _stack_top        # Set stack pointer to top of stack
    call main                # Call main()
    # If main returns, loop forever
1:
    wfi
    j 1b

    .section .bss
    .space 4096              # reserve 4KB stack space (adjust as needed)
_stack_top:

```
## Commands Used
```bash
riscv32-unknown-elf-gcc -march=rv32imac_zicsr -mabi=ilp32 -nostdlib -nostartfiles -T linker.ld -o inline.elf start.S inline.c
qemu-system-riscv32 -machine virt -nographic -kernel inline.elf
```


## Explanation of Components
1. asm volatile(...)
    - This tells the compiler to insert inline assembly into the C code.
    - asm: keyword for GCC-style inline assembly.
    - volatile: tells the compiler not to optimize away or reorder this assembly instruction.
    - Without volatile, the compiler might remove or move this line if it thinks it has no side effects (since the output c is not guaranteed to be used immediately).


2. "csrr %0, cycle"
    - This is the RISC-V assembly instruction that reads the cycle CSR (0xC00).
    - csrr: Control and Status Register Read instruction
    - %0: placeholder for the first output operand (in this case, c)
    - It tells the assembler: “substitute this with a register that will hold the output.”


3. "=r"(c)
    - his is the output operand constraint, which tells the compiler:
    - =: this is write-only (output-only). The value of c is written by the instruction.
    - r: use a general-purpose register to hold the value of cycle.
    - (c): bind this output to the C variable c.


# Task 10:  Memory-Mapped I/O Demo
## Objective
Toggle a GPIO register located at address 0x10012000 using bare-metal C, and prevent the compiler from optimizing away the memory access.

## Commands Used
```bash
riscv32-unknown-elf-gcc -march=rv32imac_zicsr -mabi=ilp32 -nostdlib -nostartfiles -T linker.ld -o tog.elf start.S tog.c
qemu-system-riscv32 -machine virt -nographic -kernel tog.elf
```

## Bare-metal toggle program
```c
#include <stdint.h>
#define UART_BASE 0x10000000
#define UART_THR  (*(volatile uint8_t *)(UART_BASE + 0x00))  // UART transmit register
#define GPIO_ADDR 0x10012000

void uart_putc(char c) {
    UART_THR = c;
}

void uart_puts(const char *s) {
    while (*s) uart_putc(*s++);
}

void delay() {
    for (volatile int i = 0; i < 100000; ++i);
}

int main() {
    volatile uint32_t *gpio = (volatile uint32_t *)GPIO_ADDR;

    uart_puts("GPIO set HIGH\n");
    *gpio = 0x1;        // Set GPIO high
    delay();

    uart_puts("GPIO set LOW\n");
    *gpio = 0x0;        // Set GPIO low

    // Halt the processor using `wfi` (wait for interrupt)
    while (1) {
        __asm__ volatile ("wfi");
    }
}

```

## Startup and linker file
```bash
    .section .text
    .globl _start
_start:
    la sp, _stack_top        # Set stack pointer to top of stack
    call main                # Call main()
    # If main returns, loop forever
1:
    wfi
    j 1b

    .section .bss
    .space 4096              # reserve 4KB stack space (adjust as needed)
_stack_top:
```


```bash
ENTRY(_start)

MEMORY {
    FLASH : ORIGIN = 0x80200000, LENGTH = 4M   /* For code, read+execute */
    RAM   : ORIGIN = 0x80600000, LENGTH = 12M  /* For data, read+write */
}

SECTIONS {
    .text : {
        *(.text*)
    } > FLASH

    .rodata : {
        *(.rodata*)
    } > FLASH

    .data : {
        *(.data*)
    } > RAM

    .bss : {
        *(.bss*)
        *(COMMON)
    } > RAM
}

```


## Key Concepts
### volatile Keyword
- Tells the compiler not to optimize access to the memory location.
- Ensures that every read/write to *gpio actually occurs, even if it looks redundant in the code.
- Without volatile, the compiler might optimize out writes that appear to have no side effects, which is dangerous in embedded I/O.

### Memory Alignment
- uint32_t is 4 bytes, so 0x10012000 must be 4-byte aligned — which it is.
- Misaligned accesses can cause hardware faults or undefined behavior on some systems.

# Task 11: Linker Script 101
## Objective
To create a  minimal linker script that places .text at 0x00000000 and .data at 0x10000000 for RV32IMC.

## Linker Script
```bash
ENTRY(_start)

SECTIONS {
    .text 0x00000000 : {
        *(.text)
        *(.text.*)
    }

    .data 0x10000000 : {
        *(.data)
        *(.data.*)
    }
}

```
## Explanation: Flash vs SRAM
1. Flash (e.g., address 0x00000000):
    - Non-volatile storage.
    - Stores code and sometimes constants.
    - Read-only during runtime unless special controller access is used.


2. SRAM (e.g., address 0x10000000):
    - Volatile memory.
    - Used for read/write data sections like .data and .bss.
    - Loses content on power-off.


## Reason for separation:
- At boot, code runs from Flash.
- Global/static variables need to be writable — placed in SRAM.
- The bootloader or startup code often copies .data from Flash to SRAM before jumping to main().

# Task 12: Start-up Code & crt0 

## Objective
To understand crt0.S role in a bare-metal RISC-V program and finding out where to get one.

## Startup script
```asm
    .section .text
    .globl _start
_start:
    # Set up the stack pointer
    la sp, _stack_top

    # Zero out the .bss section
    la a0, __bss_start
    la a1, __bss_end
zero_bss:
    bgeu a0, a1, bss_done
    sw zero, 0(a0)
    addi a0, a0, 4
    j zero_bss
bss_done:

    # Call main()
    call main

    # Infinite loop if main returns
1:
    wfi
    j 1b

    .section .bss
    .globl __bss_start
    .globl __bss_end
__bss_start:
    .space 0x1000    # Adjust size as needed
__bss_end:

    .space 4096
_stack_top:
```

## Where to get a crt0.S?
1. Newlib: A popular C library for embedded systems, includes a generic crt0.S that you can adapt.


    Examples in SDKs:
    - Microchip’s SoftConsole or SiFive Freedom E SDK.
    - PlatformIO or Zephyr RTOS projects.

2. VSDSquad or GitHub: Many educational projects and RISC-V demos on GitHub (search for riscv crt0.S).

# Task 13: Interrupt Primer
## Objective
To enable the machine-timer interrupt (MTIP) and write a simple handler in C

## What is the Machine Timer Interrupt (MTIP)?

The **Machine Timer Interrupt (MTIP)** is a fundamental interrupt in the RISC-V privileged architecture. It is triggered by the machine timer, which uses two special memory-mapped registers:

- **mtime**: A continuously incrementing 64-bit timer register representing the current time.
- **mtimecmp**: A 64-bit comparator register. When `mtime` equals or exceeds `mtimecmp`, the machine timer interrupt is triggered.

These registers are usually implemented in the **CLINT** (Core Local Interruptor) hardware block of the RISC-V system.

---

## How MTIP Works in this Example

- **Setting up MTIP**:  
  We program `mtimecmp` to `mtime + MTIMECMP_DELAY`. This means the timer interrupt will fire when the current time reaches this new threshold.

- **Interrupt generation**:  
  When `mtime` reaches `mtimecmp`, the hardware sets the MTIP bit in the machine interrupt pending register (`mip`), causing the machine-level timer interrupt.

- **Interrupt handler**:  
  Upon interrupt, control transfers to the trap handler (`trap_handler`), which saves the context, calls the C-level `timer_isr()` handler, then restores context and returns using `mret`.

- **Reprogramming the timer**:  
  Inside `timer_isr()`, we print a message and then set the next interrupt point by updating `mtimecmp` again (`mtime + MTIMECMP_DELAY`). This ensures periodic timer interrupts continue.

- **Interrupt count limit**:  
  We limit the number of interrupts handled to 5 using a `count` variable, stopping after 5 interrupts.

---

## MTIP handler in C
```c
#include <stdint.h>
#include <stddef.h>

#define UART_BASE        0x10000000
#define MSTATUS_MIE      (1 << 3)
#define MIE_MTIE         (1 << 7)
#define MTIMECMP_DELAY   500000

#define CLINT_MTIME      (*(volatile uint64_t *)(0x200bff8))
#define CLINT_MTIMECMP   (*(volatile uint64_t *)(0x2004000))

// UART output helper
void uart_puts(const char *s) {
    volatile char *uart = (volatile char *)UART_BASE;
    while (*s) {
        *uart = *s++;
    }
}
// Timer interrupt handler
volatile int count = 0;

void timer_isr(void) {
    if (count < 5) {
        uart_puts(">> Timer Interrupt Triggered\n");
        count++;
        CLINT_MTIMECMP = CLINT_MTIME + MTIMECMP_DELAY;
    }
}
void main(void) {
    uart_puts("== Timer Interrupt Example ==\n");

    timer_isr();  // Initial test call

    // Set mtvec to point to trap handler
    extern void trap_handler(void);
    uintptr_t trap_addr = (uintptr_t)&trap_handler;
    asm volatile("csrw mtvec, %0" :: "r"(trap_addr));

    // Set first timer interrupt
    CLINT_MTIMECMP = CLINT_MTIME + MTIMECMP_DELAY;

    // Enable machine timer interrupt and global interrupt
    asm volatile("csrs mie, %0" :: "r"(MIE_MTIE));
    asm volatile("csrs mstatus, %0" :: "r"(MSTATUS_MIE));

    while (1) {
        asm volatile("wfi");
    }
}

```
## Linker
```
OUTPUT_ARCH(riscv)
ENTRY(_start)

MEMORY {
  RAM (rwx) : ORIGIN = 0x80000000, LENGTH = 16M
}
SECTIONS {
  . = 0x80000000;
  .text : {
    *(.text*)
  }
  .rodata : {
    *(.rodata*)
  }
  .data : {
    *(.data*)
  }
  .bss : {
    *(.bss*)
    *(COMMON)
  }
  .trap : {
    *(.trap)
  }

  . = ALIGN(4);
  PROVIDE(_stack_top = ORIGIN(RAM) + LENGTH(RAM));
}
```
## Startup code

```
.section .text
.globl _start
_start:
    la sp, _stack_top         // Initialize stack pointer
    call main                 // Call main()
1:  wfi                       // Halt if main returns
    j 1b

.section .trap, "ax"
.globl trap_handler
trap_handler:
    addi sp, sp, -16
    sw ra, 12(sp)
    sw t0, 8(sp)
    sw t1, 4(sp)
    sw t2, 0(sp)

    call timer_isr           // Call C handler

    lw ra, 12(sp)
    lw t0, 8(sp)
    lw t1, 4(sp)
    lw t2, 0(sp)
    addi sp, sp, 16
    mret

```

## Commands Used
```bash
gedit intr.c
gedit intr_link.ld
gedit startup_intr.S

riscv32-unknown-elf-gcc -march=rv32imac_zicsr -mabi=ilp32 -nostdlib -nostartfiles   -T intr_link.ld -o intr.elf startup_intr.S intr.c
qemu-system-riscv32 -machine virt -nographic -bios none -kernel intr.elf
```

## Output of the MTIP handler


- We're setting a machine timer interrupt (MTIP) using mtimecmp = mtime + delay.
- Every time mtime reaches or exceeds mtimecmp, an interrupt triggers. Threfore it causes loop of interrupts every MTIMECMP_DELAY ticks.
- Since we have used a simple for loop to limit it to 5 times, we only see 5 interrupt triggers.

## Key Observations from This Task

- **Interrupts trigger periodically** due to the `mtimecmp` update on each timer ISR call, creating a timer tick interrupt every fixed delay.

- **Enabling interrupts properly is crucial**:
  - Writing the trap vector base address to `mtvec` CSR enables the processor to jump to your trap handler on interrupts.
  - Setting the machine interrupt enable (`MSTATUS_MIE`) and machine timer interrupt enable (`MIE_MTIE`) bits is essential for receiving timer interrupts.

- **Stack and context management**:  
  The trap handler saves and restores key registers (`ra`, `t0`, `t1`, `t2`) to preserve the context of the interrupted code. This ensures your interrupt handler doesn’t corrupt the program state.

- **`wfi` instruction usage**:  
  The main loop uses the `wfi` (wait for interrupt) instruction to reduce power consumption by halting the CPU until the next interrupt.

- **Linker script placement matters**:  
  The `.trap` section is defined to place the trap handler code correctly in memory, matching the `mtvec` address.

---

## Additional Notes

- The **machine timer interrupt is a key facility for OSes and bare-metal applications** to implement periodic scheduling, timekeeping, or timeouts.

- The **CLINT base addresses** (`0x2004000` for `mtimecmp` and `0x200bff8` for `mtime`) are platform specific and depend on the RISC-V board or emulator configuration (here QEMU virt machine).

- The **`volatile` keyword is essential** when accessing these hardware registers to prevent the compiler from optimizing out necessary reads/writes.

---


# Task 14:  rv32imac vs rv32imc – What’s the “A”? 
## Objective
**To explain the ‘A’ (atomic) extension in rv32imac. What instructions are added and why are they useful?**

## Concept  
- The **‘A’ extension** stands for **Atomic instructions** in the RISC-V ISA, making the difference between `rv32imc` (integer, multiply, compressed) and `rv32imac` (integer, multiply, atomic, compressed).  
- It adds **atomic memory operations** such as:  
  - **`lr.w`** (Load-Reserved Word)  
  - **`sc.w`** (Store-Conditional Word)  
  - **`amoadd.w`** (Atomic Memory Operation: Add Word)  
  - And other atomic operations like `amoswap.w`, `amoxor.w`, `amoand.w`, `amoor.w`, `amomin.w`, `amomax.w`, etc.  
- These instructions allow **atomic read-modify-write sequences** on memory locations without interruption, which is crucial for:  
  - Implementing **lock-free data structures** (queues, stacks)  
  - Writing **OS kernels and synchronization primitives** (mutexes, spinlocks, semaphores)  
  - Ensuring **safe concurrency** on multi-core or multi-threaded processors  
- Without these atomic instructions, software would need to disable interrupts or use heavier synchronization methods, which impact performance and scalability.

# Task 15: Atomic Test Program 
## Objective
Demonstrate the use of the RISC-V atomic extension (`A`) to implement a simple spinlock using the Load-Reserved (LR) and Store-Conditional (SC) instructions in a two-thread pseudo-concurrent environment.

## Spinlock Implementation Using LR/SC
The spinlock works by repeatedly attempting to set a lock variable from 0 (unlocked) to 1 (locked). If the lock is already held (value 1), the thread spins until it becomes free.

## Pseudo-threaded C Implementation with Inline Assembly

```c
#include <stdint.h>

#define UART_BASE 0x10000000
volatile char *uart = (volatile char *)UART_BASE;

// UART print helper
void uart_puts(const char *s) {
    while (*s) {
        *uart = *s++;
    }
}

// Simple spinlock using lr/sc
volatile int lock = 0;

void spinlock_acquire(volatile int *lock) {
    int tmp;
    asm volatile(
        "1:\n"
        "lr.w %0, (%1)\n"         // load-reserved from lock
        "bnez %0, 1b\n"           // if lock != 0, spin (busy wait)
        "li %0, 1\n"              // try to set lock = 1
        "sc.w %0, %0, (%1)\n"     // store-conditional to lock
        "bnez %0, 1b\n"           // if store failed, retry
        : "=&r"(tmp)
        : "r"(lock)
        : "memory"
    );
}

void spinlock_release(volatile int *lock) {
    *lock = 0;
}

// Simulate two pseudo-threads trying to acquire lock alternately
void main(void) {
    uart_puts("Starting atomic spinlock test\n");

    for (int i = 0; i < 2; i++) {
        // Pseudo-thread 1
        uart_puts("Thread 1: Waiting for lock...\n");
        spinlock_acquire(&lock);
        uart_puts("Thread 1: Lock acquired!\n");

        // Critical section
        uart_puts("Thread 1: Working...\n");

        spinlock_release(&lock);
        uart_puts("Thread 1: Lock released\n\n");

        // Pseudo-thread 2
        uart_puts("Thread 2: Waiting for lock...\n");
        spinlock_acquire(&lock);
        uart_puts("Thread 2: Lock acquired!\n");

        // Critical section
        uart_puts("Thread 2: Working...\n");

        spinlock_release(&lock);
        uart_puts("Thread 2: Lock released\n\n");
    }

    uart_puts("Test completed.\n");

    while (1) {
        asm volatile("wfi");
    }
}
```

---

## Key Observations

- The use of `lr.w` and `sc.w` allows atomic lock acquisition without disabling interrupts or requiring complex hardware primitives.
- The `spinlock_acquire` function loops until it successfully sets the lock.
- The pseudo-threaded example shows alternating lock acquisition by two threads, ensuring mutual exclusion.
- This approach forms the foundation for OS-level mutexes and other concurrency controls.
- Works well in a simulated environment like QEMU; actual hardware concurrency requires proper thread/task support.

---

## Commands Used
```bash
riscv32-unknown-elf-gcc -march=rv32imac_zicsr -mabi=ilp32 -nostdlib -nostartfiles   -T intr_link.ld -o threads.elf start.S threads.c
qemu-system-riscv32 -machine virt -nographic -bios none -kernel threads.elf
```
### Note: The same Linker and startup scripts have been used here

---


---

# Task 16: Using Newlib printf Without an OS

## Objective
Demonstrate how to use Newlib's printf functionality in a bare-metal RISC-V environment by implementing custom system call stubs and retargeting the `_write` function to output via UART.

## Solution Overview
- Implement `_write(int fd, char* buf, int len)` that loops over bytes to UART_TX
- Provide required syscall stubs for Newlib compatibility
- Link with Newlib while using custom startup code

## Implementation

### Write system call function
```c
// Write system call - this is what printf uses
int _write(int fd, char *buf, int len) {
    // Only handle stdout/stderr
    if (fd == 1 || fd == 2) {
        for (int i = 0; i < len; i++) {
            uart_putc(buf[i]);
        }
        return len;
    }
    return -1;
}
```

### syscalls.c - System Call Implementations
```c
#include <sys/stat.h>
#include <sys/types.h>
#include <sys/fcntl.h>
#include <sys/times.h>
#include <sys/errno.h>
#include <sys/time.h>
#include <stdio.h>

// QEMU virt machine UART base address
#define UART_BASE 0x10000000
#define UART_TX   (*(volatile char*)(UART_BASE + 0))
#define UART_LSR  (*(volatile char*)(UART_BASE + 5))
#define UART_LSR_THRE 0x20  // Transmitter holding register empty

// Simple UART output function
static void uart_putc(char c) {
    // Wait for transmitter to be ready
    while (!(UART_LSR & UART_LSR_THRE));
    UART_TX = c;
}

// Write system call - this is what printf uses
int _write(int fd, char *buf, int len) {
    // Only handle stdout/stderr
    if (fd == 1 || fd == 2) {
        for (int i = 0; i < len; i++) {
            uart_putc(buf[i]);
        }
        return len;
    }
    return -1;
}

// Required syscall stubs for Newlib
int _read(int fd, char *buf, int len) {
    return -1;  // Not implemented
}

int _close(int fd) {
    return -1;  // Not implemented
}

int _lseek(int fd, int offset, int whence) {
    return -1;  // Not implemented
}

int _fstat(int fd, struct stat *st) {
    st->st_mode = S_IFCHR;
    return 0;
}

int _isatty(int fd) {
    return 1;  // Assume all file descriptors are TTY
}

void *_sbrk(int incr) {
    extern char _end;
    static char *heap_end = 0;
    char *prev_heap_end;

    if (heap_end == 0) {
        heap_end = &_end;
    }
    
    prev_heap_end = heap_end;
    heap_end += incr;
    
    return (void *)prev_heap_end;
}

void _exit(int status) {
    while (1) {
        asm volatile("wfi");
    }
}

int _kill(int pid, int sig) {
    return -1;  // Not implemented
}

int _getpid(void) {
    return 1;
}
```

### main.c - Main Program with Printf
```c
#include <stdio.h>
#include <stdint.h>


int main(void) {
    printf("Hello from RISC-V bare metal!\n");
    printf("Testing printf with numbers: %d \n", 42);
    printf("Float test: %.2f\n", 3.14159);
    
    while (1) {
        // Main loop - let interrupts handle the rest
        asm volatile("wfi");
    }
    
    return 0;
}
```

### Updated Linker Script (intr_link.ld)
```ld
OUTPUT_ARCH(riscv)
ENTRY(_start)

MEMORY {
  RAM (rwx) : ORIGIN = 0x80000000, LENGTH = 16M
}

SECTIONS {
  . = 0x80000000;
  
  .text : {
    *(.text.startup)
    *(.text*)
  } > RAM
  
  .rodata : {
    *(.rodata*)
  } > RAM
  
  .data : {
    *(.data*)
  } > RAM
  
  .bss : {
    *(.bss*)
    *(COMMON)
    . = ALIGN(4);
    _end = .;  /* End of BSS for heap */
  } > RAM
  
  .trap : {
    *(.trap)
  } > RAM
  
  . = ALIGN(4);
  PROVIDE(_stack_top = ORIGIN(RAM) + LENGTH(RAM));
}
```


---

## Commands Used
```bash
# Compile command - Key changes from original:
# Kept -nostartfiles since we provide our own _start

riscv32-unknown-elf-gcc -march=rv32imac_zicsr -mabi=ilp32 \
    -nostartfiles -T intr_link.ld \
    -o main.elf start.S main.c syscalls.c

# Run with QEMU
qemu-system-riscv32 -machine virt -nographic -bios none -kernel threads.elf
```
### Note: Uses the same startup assembly (start.S) as the previous tasks

---

## Key Technical Details

### UART Retargeting
- The `_write()` function intercepts all printf output calls
- Uses QEMU virt machine's UART at memory address `0x10000000`
- Implements proper UART status checking before transmitting bytes
- Only handles stdout (fd=1) and stderr (fd=2) file descriptors

### System Call Stubs
- `_sbrk()` provides heap management for printf's internal memory allocation
- `_fstat()` and `_isatty()` ensure proper file descriptor handling
- Other stubs prevent linking errors while indicating non-implementation

### Compilation Changes
- **Removed `-nostdlib` to allow linking with Newlib (required for printf)
- **Added `syscalls.c` to provide system call implementations
- **Kept `-nostartfiles` since custom startup code is provided

---

## How It Works

1. **Printf Call: When `printf()` is called, Newlib formats the string and calls `_write(1, buffer, length)`
2. **Syscall Interception: Our custom `_write()` function intercepts this call
3. **UART Output: Each byte in the buffer is sent to the UART transmit register
4. **Memory Management: The `_sbrk()` function provides heap space for printf's internal operations

---



---

## Key Observations

- Printf functionality works seamlessly in bare-metal environment with proper syscall retargeting
- The UART implementation properly handles transmitter status checking for reliable output
- Heap management via `_sbrk()` enables printf's dynamic memory allocation
- The approach maintains compatibility with existing startup code and interrupt handlers
- Forms the foundation for more complex I/O operations in embedded systems
- Can be extended to support input operations by implementing `_read()` with UART receive functionality


# Task 17: RISC-V Endianness & Struct Packing Analysis

## Objective
Verify RISC-V byte ordering (endianness) using C union tricks and demonstrate struct packing behavior in bare-metal environments.

## Question Answered
"Is RV32 little-endian by default? Show me how to verify byte ordering with a union trick in C."

**Answer: Yes, RISC-V (including RV32) is little-endian by default.**

## Endianness Verification Method

### Union Trick Implementation
The most reliable way to verify endianness is using a union that overlays a 32-bit integer with a 4-byte array:

```c
typedef union {
    uint32_t word;
    uint8_t bytes[4];
} endian_test_t;

void test_endianness(void) {
    endian_test_t test;
    test.word = 0x01020304;  // Store test pattern
    
    printf("Stored value: 0x%08X\n", test.word);
    printf("bytes[0] = 0x%02X (LSB)\n", test.bytes[0]);
    printf("bytes[1] = 0x%02X\n", test.bytes[1]);
    printf("bytes[2] = 0x%02X\n", test.bytes[2]);
    printf("bytes[3] = 0x%02X (MSB)\n", test.bytes[3]);
    
    if (test.bytes[0] == 0x04) {
        printf("Result: LITTLE ENDIAN\n");
        printf("Memory layout: [04][03][02][01]\n");
    }
}
```

### Expected RISC-V Output
```
Stored value: 0x01020304
bytes[0] = 0x04 (LSB)
bytes[1] = 0x03
bytes[2] = 0x02
bytes[3] = 0x01 (MSB)
Result: LITTLE ENDIAN
Memory layout: [04][03][02][01]
```

## Struct Packing Analysis

### Normal vs Packed Structures
```c
// Normal struct (with padding)
struct normal_struct {
    uint8_t  byte1;    // 1 byte
    uint32_t word1;    // 4 bytes (aligned to 4-byte boundary)
    uint16_t half1;    // 2 bytes
    uint8_t  byte2;    // 1 byte
};  // Total: 12 bytes due to padding

// Packed struct (no padding)
struct packed_struct {
    uint8_t  byte1;    // 1 byte
    uint32_t word1;    // 4 bytes
    uint16_t half1;    // 2 bytes
    uint8_t  byte2;    // 1 byte
} __attribute__((packed));  // Total: 8 bytes
```

### Memory Layout Comparison
**Normal Struct (12 bytes):**
```
Offset 0: 0xAA       (byte1)
Offset 1: 0x00       (padding)
Offset 2: 0x00       (padding)
Offset 3: 0x00       (padding)
Offset 4: 0x78       (word1 LSB)
Offset 5: 0x56
Offset 6: 0x34
Offset 7: 0x12       (word1 MSB)
Offset 8: 0xDE       (half1 LSB)
Offset 9: 0xBC       (half1 MSB)
Offset 10: 0xFF      (byte2)
Offset 11: 0x00      (padding)
```

**Packed Struct (8 bytes):**
```
Offset 0: 0xAA       (byte1)
Offset 1: 0x78       (word1 LSB)
Offset 2: 0x56
Offset 3: 0x34
Offset 4: 0x12       (word1 MSB)
Offset 5: 0xDE       (half1 LSB)
Offset 6: 0xBC       (half1 MSB)
Offset 7: 0xFF       (byte2)
```

## Complete Test Program

```c
#include <stdio.h>
#include <stdint.h>

// Union trick to examine byte ordering
typedef union {
    uint32_t word;
    uint8_t bytes[4];
} endian_test_t;

// Struct packing examples
struct packed_struct {
    uint8_t  byte1;
    uint32_t word1;
    uint16_t half1;
    uint8_t  byte2;
} __attribute__((packed));

struct normal_struct {
    uint8_t  byte1;
    uint32_t word1;
    uint16_t half1;
    uint8_t  byte2;
};

// Function to print binary representation
void print_binary(uint32_t value) {
    printf("Binary: ");
    for (int i = 31; i >= 0; i--) {
        printf("%c", (value & (1U << i)) ? '1' : '0');
        if (i % 8 == 0 && i > 0) printf(" ");
    }
    printf("\n");
}

int main(void) {
    printf("RISC-V Endianness & Struct Packing Analysis\n");
    printf("==========================================\n\n");
    
    // === ENDIANNESS TEST ===
    printf("=== RISC-V Endianness Test ===\n");
    
    endian_test_t test;
    test.word = 0x01020304;
    
    printf("Stored value: 0x%08X\n", test.word);
    print_binary(test.word);
    
    printf("Byte-by-byte examination:\n");
    printf("bytes[0] = 0x%02X (LSB)\n", test.bytes[0]);
    printf("bytes[1] = 0x%02X\n", test.bytes[1]);
    printf("bytes[2] = 0x%02X\n", test.bytes[2]);
    printf("bytes[3] = 0x%02X (MSB)\n", test.bytes[3]);
    
    // Determine endianness
    if (test.bytes[0] == 0x04) {
        printf("Result: LITTLE ENDIAN (LSB first)\n");
        printf("Memory layout: [04][03][02][01]\n");
    } else if (test.bytes[0] == 0x01) {
        printf("Result: BIG ENDIAN (MSB first)\n");
        printf("Memory layout: [01][02][03][04]\n");
    } else {
        printf("Result: UNKNOWN ENDIANNESS\n");
    }
    printf("\n");
    
    // === STRUCT PACKING TEST ===
    printf("=== Struct Packing Test ===\n");
    
    struct normal_struct normal = {0xAA, 0x12345678, 0xBCDE, 0xFF};
    struct packed_struct packed = {0xAA, 0x12345678, 0xBCDE, 0xFF};
    
    printf("Normal struct size: %zu bytes\n", sizeof(struct normal_struct));
    printf("Packed struct size: %zu bytes\n", sizeof(struct packed_struct));
    
    // Print memory layout of normal struct
    printf("\nNormal struct memory layout:\n");
    uint8_t *normal_ptr = (uint8_t*)&normal;
    for (size_t i = 0; i < sizeof(normal); i++) {
        printf("Offset %zu: 0x%02X\n", i, normal_ptr[i]);
    }
    
    // Print memory layout of packed struct
    printf("\nPacked struct memory layout:\n");
    uint8_t *packed_ptr = (uint8_t*)&packed;
    for (size_t i = 0; i < sizeof(packed); i++) {
        printf("Offset %zu: 0x%02X\n", i, packed_ptr[i]);
    }
    printf("\n");
    
    // === BIT FIELD TEST ===
    printf("=== Bit Field Test ===\n");
    
    struct bit_field_test {
        uint32_t field1 : 4;   // 4 bits
        uint32_t field2 : 8;   // 8 bits
        uint32_t field3 : 12;  // 12 bits
        uint32_t field4 : 8;   // 8 bits
    } bf;
    
    bf.field1 = 0xF;      // 1111
    bf.field2 = 0xAB;     // 10101011
    bf.field3 = 0x123;    // 000100100011
    bf.field4 = 0xCD;     // 11001101
    
    printf("Bit field struct size: %zu bytes\n", sizeof(bf));
    printf("field1 (4 bits) = 0x%X\n", bf.field1);
    printf("field2 (8 bits) = 0x%02X\n", bf.field2);
    printf("field3 (12 bits) = 0x%03X\n", bf.field3);
    printf("field4 (8 bits) = 0x%02X\n", bf.field4);
    
    // Print raw memory
    printf("Raw memory content:\n");
    uint8_t *bf_ptr = (uint8_t*)&bf;
    for (size_t i = 0; i < sizeof(bf); i++) {
        printf("Byte %zu: 0x%02X\n", i, bf_ptr[i]);
    }
    printf("\n");
    
    // === ALIGNMENT TEST ===
    printf("=== Memory Alignment Test ===\n");
    
    char buffer[16];
    
    // Test different data type alignments
    uint8_t  *ptr8  = (uint8_t*)(buffer + 1);
    uint16_t *ptr16 = (uint16_t*)(buffer + 2);
    uint32_t *ptr32 = (uint32_t*)(buffer + 4);
    
    printf("Buffer base address: %p\n", (void*)buffer);
    printf("uint8_t  pointer: %p (offset: %ld)\n", (void*)ptr8, (char*)ptr8 - buffer);
    printf("uint16_t pointer: %p (offset: %ld)\n", (void*)ptr16, (char*)ptr16 - buffer);
    printf("uint32_t pointer: %p (offset: %ld)\n", (void*)ptr32, (char*)ptr32 - buffer);
    
    // Check alignment
    printf("uint8_t  alignment: %s\n", ((uintptr_t)ptr8 % 1 == 0) ? "OK" : "BAD");
    printf("uint16_t alignment: %s\n", ((uintptr_t)ptr16 % 2 == 0) ? "OK" : "BAD");
    printf("uint32_t alignment: %s\n", ((uintptr_t)ptr32 % 4 == 0) ? "OK" : "BAD");
    printf("\n");
    
    printf("Analysis complete!\n");
    printf("Program will now halt...\n");
    
    // Give time for all output to flush
    for (volatile int i = 0; i < 1000000; i++) {
        // Small delay to ensure output completes
    }
    
    while (1) {
        asm volatile("wfi");
    }
    
    return 0;
}
```
## Commands Used
```bash
# Compile with the syscalls implementation for printf support
riscv32-unknown-elf-gcc -march=rv32imac_zicsr -mabi=ilp32 \
    -nostartfiles -T linker.ld \
    -o endianness.elf start.S endianness.c syscalls.c

# Run with QEMU
qemu-system-riscv32 -machine virt -nographic -bios none -kernel endianness.elf
```

### Note: Uses the same startup assembly (start.S), linker script (intr_link.ld), and syscalls.c from the previous printf example

---



## Key Technical Points

### RISC-V Endianness
- **Default**: Little-endian (LSB stored at lowest memory address)
- **Bi-endian Support**: RISC-V specification allows big-endian implementations, but little-endian is standard
- **Verification**: Union trick reliably detects byte ordering at runtime

### Structure Alignment Rules
- **Natural Alignment**: Data types align to their size boundaries
  - `uint8_t`: 1-byte alignment
  - `uint16_t`: 2-byte alignment  
  - `uint32_t`: 4-byte alignment
- **Padding**: Compiler inserts padding bytes to maintain alignment
- **Packed Attribute**: `__attribute__((packed))` removes padding

### Bit Field Behavior
- Bit fields are allocated in little-endian order within each storage unit
- First declared field occupies least significant bits
- Endianness affects how bit fields map to memory bytes

---

## Key Observations

- **RISC-V is little-endian by default**: The union trick confirms LSB is stored at the lowest memory address
- **Struct padding affects memory efficiency**: Normal structs use 50% more memory due to alignment requirements
- **Packed structs trade performance for space**: Remove padding but may cause unaligned access penalties
- **Endianness impacts multi-byte data interpretation**: Critical for network protocols, file formats, and cross-platform compatibility
- **Union trick is portable**: Works across different architectures and compilers for endianness detection
- **Memory layout visualization**: Direct byte examination reveals how data is actually stored in memory

---
