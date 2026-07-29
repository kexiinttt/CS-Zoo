# User Mode and Kernel Mode

Operating systems usually divide CPU execution privileges into levels:
- **User mode** &rarr; lower privilege; ordinary applications run here.
- **Kernel mode** &rarr; higher privilege; the operating-system kernel runs here.

---

# Why Separate User and Kernel Mode?

Privilege isolation makes the system safer and more stable. Without it, an ordinary program could:

- Read or write arbitrary physical memory
- Access a disk controller directly
- Operate a network card freely
- Disable interrupts
- Modify page tables
- Corrupt another process's data

---

# Characteristics

| Mode | Characteristics |
|---|---|
| User mode | Restricted permissions; cannot directly execute privileged instructions or operate hardware; cannot freely access kernel memory; must request services through system calls |
| Kernel mode | Highest privilege; can execute privileged instructions, access hardware, and manage processes, threads, memory, and other system resources |

---

# What Runs in Each Mode?

Ordinary C++ or Python code normally runs in user mode:
- Browsers
- Applications
- User logic in a database
- Command-line programs

Kernel mode normally runs operating-system code:
- Process scheduling
- Memory management
- File systems
- Network protocol stacks
- Interrupt handling
- Device drivers

---

# Privileged Instructions

Some CPU instructions cannot be executed freely by ordinary applications. They are **privileged instructions** and generally may run only in kernel mode:

- Modifying page tables or address-translation registers
- Enabling or disabling interrupts
- Performing I/O-port operations
- Changing CPU control registers
- Triggering low-level hardware control operations

If a user-mode program attempts one, the CPU normally raises an exception and the OS takes control.

---

# System Calls

Application code runs in user mode, but operations requiring OS assistance enter kernel mode through a system call:

- Read or write a file: `read` / `write`
- Create a process: `fork`
- Exit a process: `exit`
- Allocate memory: `mmap`
- Establish a network connection: `connect`

---

# When Does a Mode Switch Happen?

## System Call

When a program needs an operating-system operation, it makes a system call.

## Exception

An exceptional condition during execution also enters kernel mode:

- Page fault
- Division by zero
- Illegal instruction
- Invalid-address access
- User-mode execution of a privileged instruction

## Interrupt

External hardware events can also switch the CPU into kernel mode. The CPU pauses the current execution flow and runs the kernel's interrupt handler:

- Timer interrupt
- Keyboard input
- Packet arrival at the network card
- Disk I/O completion
- Other device interrupts

---

# What Does the CPU Do During the Switch?

- Change privilege level from user mode to kernel mode.
- Save the current context, including the program counter, some registers, and status registers, so execution can later resume.
- Jump to the kernel entry selected by the interrupt or trap table.
- Switch to the kernel stack. User programs have user stacks; the CPU and OS use the current thread's kernel stack for system calls and interrupts.
- Execute kernel code.
- Restore the context and return. The kernel restores saved registers and the execution position, then executes a return instruction to resume user mode.

After entering kernel mode, the original user stack is usually no longer used. The CPU switches to the thread's kernel stack, so context-related registers must be saved to memory before entry.

## What Is Saved and Where?

Because scheduling may switch threads, the state is generally saved in the TCB. See [PCB and TCB](./processes-and-threads.md#control-block).

---

# User Stack and Kernel Stack

Each thread normally has at least two stacks:
- **User stack**: used while the thread runs in user mode.
- **Kernel stack**: used after the thread enters kernel mode.

The kernel cannot fully trust user-space data or the user stack. Continuing to use it could create security and stability problems, so a system call, interrupt, or exception switches to the kernel stack.

---

# User/Kernel Mode Switch Is Not a Context Switch

A user/kernel mode switch changes privilege level and may still continue executing the same thread.

A context switch changes the thread, but it does not necessarily involve a system call, exception, or interrupt.

---

# Does Every System Call Switch Threads?

No. A non-blocking operation such as `getpid()` may simply enter kernel mode, do a small amount of work, and immediately return. The thread may never be switched out.

Some system calls block, however:

- `read` waiting for data
- `accept` waiting for a connection
- `sleep`
- `futex` waiting for a lock

The thread may then be suspended and the scheduler may run another thread.

---

# Why Is Reducing System Calls Important?

Every system call crosses the user/kernel boundary. Common optimizations therefore include:

- Batch reads and writes to reduce small I/O operations.
- Use buffering.
- Use `epoll` instead of inefficient polling.
- Use `mmap` to reduce explicit `read` and `write` calls.
- Use zero-copy techniques.
- Use more efficient asynchronous I/O mechanisms.
