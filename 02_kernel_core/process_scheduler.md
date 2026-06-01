# Process Scheduler & Multitasking

LumiaOS supports concurrent execution of multiple tasks through a **Preemptive Round-Robin Scheduler**.

## Process Control Block (PCB)

Every running task in LumiaOS is represented by a `pcb_t` structure (`include/core/process.h`).
The PCB holds critical state information for a process:
- `pid`: Process ID.
- `state`: Status (Ready, Running, Blocked, Sleeping).
- `eip`: Instruction Pointer (where the process is currently executing).
- `esp` / `ebp`: Stack pointers.
- Register states (eax, ebx, ecx, edx, esi, edi).

## Context Switching

Context switching is the mechanism by which the CPU stops executing one process and begins executing another.

1. **The Timer Interrupt**: The PIT fires an interrupt on IRQ0, temporarily halting the current process and jumping into kernel code.
2. **State Saving**: The low-level assembly stub `proc_switch.asm` pushes the current CPU registers onto the stack and saves the stack pointer (`esp`) into the current process's PCB.
3. **Selection**: The `proc_schedule()` function looks at the process table, finds the next process in the `PROC_READY` state, and marks it as `PROC_RUNNING`.
4. **Restoration**: The assembly stub loads the stack pointer of the *new* process, pops its registers, and uses `iret` to jump back into the new process's execution flow.

## The GUI Compositor vs. Processes

It is important to note the distinction between true background processes and GUI applications in the current architecture:

- **True Processes (`.lxe` binaries)**: These are spawned by the Shell or File Explorer and run as distinct entries in the Process Table, taking advantage of preemptive multitasking.
- **GUI Applications (Browser, Task Manager)**: Currently, built-in GUI apps run cooperatively within the Aero Compositor's main loop. The compositor iterates through windows and calls their `app_tick` functions. 

*(Future roadmap items include moving GUI applications into their own isolated preemptive processes using the `.lxe` execution format).*
