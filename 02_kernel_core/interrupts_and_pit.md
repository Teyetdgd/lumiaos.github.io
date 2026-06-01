# Interrupts and Timing (IDT & PIT)

Interrupts are the foundation of hardware interaction and multitasking in LumiaOS. The system relies on the Interrupt Descriptor Table (IDT) to catch exceptions and hardware signals, and the Programmable Interval Timer (PIT) to drive the scheduler.

## Interrupt Descriptor Table (IDT)

The IDT (`src/core/idt.cpp`) tells the CPU where to jump when an interrupt or exception occurs.

### Exceptions (0-31)
The first 32 interrupts are reserved by Intel for CPU exceptions.
- **ISR 13 (General Protection Fault):** Triggered by privilege violations.
- **ISR 14 (Page Fault):** Handled by `page_fault_handler()` to catch bad memory accesses.
- If an unhandled exception occurs, the system will halt with a Kernel Panic.

### Hardware Interrupts (IRQs)
Hardware devices (like the Keyboard, Mouse, and Disk) send signals via the Programmable Interrupt Controller (PIC).
- The Master and Slave PICs are remapped during boot so IRQs 0-15 map to IDT vectors 32-47 (avoiding collision with CPU exceptions).
- `irq_register_handler(int irq, irq_handler_t handler)` allows device drivers to attach their specific C++ functions to hardware events.

## The Programmable Interval Timer (PIT)

The PIT (`src/core/pit.cpp`) is the heartbeat of LumiaOS. It is connected to IRQ0 (Interrupt 32) and fires thousands of times per second.

### Tick Management
Every time the PIT fires, `pit_timer_handler()` increments a global `timer_ticks` counter. This counter is used throughout the OS for:
- `sleep()` functions.
- Animations in the GUI.
- Network timeouts.

### The Watchdog Mechanism
To prevent the OS from freezing, LumiaOS employs a Watchdog timer inside the PIT handler.
- When a GUI application's event loop (`app_tick`) begins, a timestamp is recorded.
- If the PIT handler detects that the application has been executing for more than 2 seconds (e.g., stuck in a `while(1)` loop), it triggers the Watchdog.
- The Watchdog forcefully breaks out of the application's execution context using `setjmp/longjmp`, rescuing the kernel from a total freeze.

## System Calls (Ring 3 to Ring 0)

LumiaOS reserves Interrupt `0x80` for System Calls. 
- When an application (or the `.lxe` runtime) needs a kernel service (like reading a file or creating a window), it triggers `int 0x80`.
- The `syscall_handler()` in `src/core/syscall_handler.cpp` decodes the requested function number in the `eax` register and dispatches the call to the appropriate kernel subsystem.
