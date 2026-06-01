# Memory Management

LumiaOS features a custom memory management subsystem that handles physical memory allocation, virtual memory mapping (Paging), and dynamic heap allocation (`kmalloc`/`kfree`).

## Physical Memory (RAM)

Physical memory is managed by `src/core/ram.cpp`. 

### Boot Information
When the kernel boots, it reads the memory map provided by GRUB via the Multiboot specification (`multiboot_info_t`). The kernel determines the total available RAM and initializes its internal tracking structures.

### Page Frame Allocator
Physical memory is divided into **4KB frames**. The kernel uses a bitmap array (`ram_bitmap`) to keep track of free and used frames.
- `ram_alloc_frame()`: Finds the first free bit in the bitmap, marks it as used, and returns the physical address.
- `ram_free_frame()`: Clears the bit corresponding to a physical address, returning it to the free pool.

## Virtual Memory (Paging)

Virtual memory is managed by `src/core/paging.cpp`. LumiaOS uses a 32-bit, two-level paging structure consisting of a **Page Directory** and **Page Tables**.

### Identity Mapping
Currently, LumiaOS uses a simplified **Identity Paging** model. This means that virtual addresses map directly to the same physical addresses (e.g., Virtual `0x100000` -> Physical `0x100000`).
- The kernel identity-maps the first 256MB of RAM during boot.
- This approach simplifies DMA (Direct Memory Access) for drivers (like Network and Disk) but means user-space applications do not currently have isolated address spaces.

### Framebuffer Mapping
The high-resolution graphical framebuffer (provided by the BGA driver, typically at physical address `0xFD000000`) is mapped dynamically by the kernel so the `Aero Compositor` can draw pixels to the screen.

### Page Faults
If code attempts to access unmapped memory or violates protection rules, a Page Fault exception (Interrupt 14) is triggered. 
- The `page_fault_handler()` decodes the error and prints a diagnostic message (BSOD).
- **Watchdog Integration:** If a GUI application causes a Page Fault, the kernel uses `setjmp`/`longjmp` to recover, forcefully closing the offending application without crashing the entire operating system.

## Dynamic Memory (Heap)

Heap allocation is handled by a custom implementation of `kmalloc` and `kfree` located in `src/core/mem.cpp`.

### Allocation Strategy
The heap starts at a predefined boundary after the kernel code.
- It uses a hybrid approach: A fast **bump allocator** for the initial pass, coupled with a slot-reuse mechanism (an array of `allocation_t` structures tracking ptr, size, and used status) to recycle freed memory.
- `kfree()` marks a slot as unused, making it available for subsequent `kmalloc()` calls of similar or smaller sizes.

### Process Limits (OOM Protection)
To prevent poorly written or malicious applications from exhausting system RAM and crashing the kernel, LumiaOS enforces strict memory limits.
- **4MB Limit:** Every process (tracked via `_alloc_owner_slot`) is limited to 4 Megabytes of heap allocation.
- If an application requests more than 4MB, `kmalloc` returns `0` (NULL), which usually triggers an application-level assert, allowing the Watchdog to safely terminate the app.
