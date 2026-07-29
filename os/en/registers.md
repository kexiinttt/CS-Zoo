# What Is a Register?

A register is a small, extremely fast storage unit inside the CPU. Registers usually have very limited capacity, but they are much faster to access than memory.

In operating systems, registers matter because:

- They record which instruction the CPU is executing.
- They record the current function-call stack position.
- Intermediate computation results are often kept in registers.
- Context switching requires saving and restoring register state.

---

# Why Does the OS Care About Registers?

The OS manages multiple threads and processes, while a CPU can execute only one thread at a time on a single core.

The execution object may change when:

- A time slice expires
- A thread blocks
- An interrupt arrives
- A system call enters the kernel
- The scheduler chooses another thread

The CPU cannot forget where the current thread stopped. It must save the execution context, whose most important part is the values of the registers.

Register state is therefore a core part of a context switch.

---

# Register Categories

Common categories include:

1. Registers controlling execution flow
   - PC (Program Counter)
   - SP (Stack Pointer)
2. Registers holding computation data
   - General-purpose registers
3. Registers holding CPU state
   - Status or flags registers
4. Memory-management registers
   - Page-table base registers and similar registers
5. Registers for floating-point and vector computation
   - FPU, SIMD, and vector registers

---

# Program Counter

The PC stores the address of the next instruction. It tells the CPU where to fetch the next instruction.

For a paused thread to resume, the CPU must know where execution stopped. The PC records that position. Without it, the thread would not know which instruction to execute next.

The PC does not simply increase by one; it changes with control flow:

- Sequential execution: points to the next instruction.
- `if`, `while`, and `for`: jumps to different code locations.
- Function call: jumps to the function entry.
- Function return: resumes after the call site.
- Interrupt or exception: jumps to the interrupt or exception handler.
- System call: enters kernel handling logic.

The PC therefore determines the program's control flow.

---

# Stack Pointer

The SP stores the location of the top of the current stack. Threads generally have independent stacks, making SP a core part of a thread's execution context.

A stack commonly stores:

- Function return addresses
- Local variables
- Some function arguments
- Saved registers
- Call-chain stack frames

Calling a function normally creates a stack frame. When a thread resumes, it must know not only where execution stopped, but also its call depth, local-variable locations, return address, and where to restore the frame. SP determines which stack state to continue from.

---

# General-Purpose Registers

General-purpose registers hold temporary data and intermediate results:
- additions, subtractions, and temporary variables
- function arguments
- return values
- memory addresses
- loop counters

Many intermediate values are in these registers while a thread runs. If the thread is switched out without saving them, its calculation will be wrong when it returns. The OS therefore saves the necessary registers during a context switch and restores them when resuming the thread.

---

# Status / Flags Register

The status register stores CPU state information:

- Zero flag
- Carry flag
- Sign flag
- Overflow flag
- Interrupt-enable bit
- Privilege-related state bits

These flags are part of the execution context. If they are not saved, conditional branches, arithmetic state, and interrupt state can be corrupted after the thread resumes.

---

# Base Pointer / Frame Pointer

Many architectures and compilers use a register to point to the base of the current stack frame. A frame usually contains:

- Return address
- Saved old frame pointer
- Local variables
- Temporary space

Using BP/FP as a fixed reference makes local variables and arguments easier to access, for example through offsets relative to FP.

---

# Memory-Management Registers

Modern operating systems use virtual memory. A process accesses virtual addresses rather than physical addresses directly. The CPU needs a way to know which page tables to use for translation.

Page tables are generally part of a process's address-space resources and are associated with the PCB or process-memory descriptor. The hardware actually performs translation using the current page-table base information stored in registers.

---

# Privilege-Level Registers

CPUs normally support privilege levels such as:

- User mode
- Kernel mode

Some registers or state bits represent the current privilege level or participate in switching between modes.

---

# Floating-Point / SIMD / Vector Registers

These registers support high-performance numerical computation, image processing, scientific computing, machine learning, and similar workloads.
