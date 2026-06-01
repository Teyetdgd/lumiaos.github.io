# Architectural Overview

LumiaOS is designed as a **Monolithic Kernel** with a strong emphasis on isolating complex subsystems into manageable modules. It eschews the use of existing libraries (no `libc`, no Linux kernel headers) to ensure a pure, from-scratch learning and execution environment.

## Design Philosophy
1. **Bare Metal, High Signal**: Every line of code exists for a reason. There are no bloated wrappers or unnecessary abstractions.
2. **Fail-Safe execution**: Applications should be able to crash without taking down the entire kernel.
3. **Modularity**: The transition from a "Header-Only" architecture to a standard `.h` / `.cpp` structure ensures clean compilation boundaries and maintainability.

## System Layers

### 1. Bootloader (GRUB2 & Multiboot)
LumiaOS relies on the **Multiboot Specification**. The `boot.asm` file acts as the entry point, setting up a minimal stack and pushing the magic number and multiboot info structure to the C++ kernel entry point `kernel_main()`.

### 2. The Core Kernel (`src/core/`)
This is the heart of the operating system. It runs in Ring 0 (highest privilege) and manages:
- **GDT / IDT**: Global Descriptor Table and Interrupt Descriptor Table for hardware and software interrupts.
- **Memory Management (`mem.cpp`, `paging.cpp`)**: Physical page allocation, virtual memory mapping (Identity Paging), and the `kmalloc` heap allocator.
- **Process Scheduler (`process.cpp`, `pit.cpp`)**: A preemptive Round-Robin scheduler driven by the Programmable Interval Timer (PIT).

### 3. Hardware Drivers (`src/drivers/`)
LumiaOS includes custom drivers to interface with specific hardware:
- **PS/2 Mouse & Keyboard**: Event-driven input handling.
- **BGA (Bochs Graphics Adapter)**: High-resolution (1024x768), 32-bpp framebuffer graphics.
- **E1000 & PCnet**: Network Interface Card (NIC) drivers for packet transmission.
- **ATA/IDE**: Block device drivers for reading/writing to hard disks.

### 4. Virtual File System (`src/fs/`)
The VFS layer abstracts the underlying storage media.
- Applications interact with generic functions like `fs_read_file()` and `fs_write_file()`.
- Depending on the boot state, these calls are routed either to **RamFS** (in-memory, volatile) or **LumiaFS** (custom block-based disk filesystem, persistent).

### 5. Network Stack (`src/net/`)
A completely custom TCP/IP stack implementation.
- **Link Layer**: Ethernet frame parsing and ARP resolution.
- **Internet Layer**: IPv4 and ICMP (Ping).
- **Transport Layer**: UDP and a robust TCP state machine.
- **Application Layer**: DNS resolution and HTTP/1.1 client implementation.
- **Security**: A native, from-scratch TLS 1.2 implementation (`tls.cpp`, `crypto/`) for secure HTTPS communication.

### 6. Aero Compositor & GUI (`src/display/`)
The graphical layer of LumiaOS.
- Implements a **software compositor** inspired by modern display servers (like Wayland/DWM).
- Manages an off-screen backbuffer to prevent screen tearing (Double Buffering).
- Handles Z-Order, Alpha Blending (transparency), and window event dispatching (clicks, drags, keyboard focus).

### 7. Applications (`include/apps/`)
User-facing software integrated into the OS.
- **Lumia Browser**: A tabbed web browser with a custom HTML/CSS rendering engine and smart download capabilities.
- **File Explorer**: Graphical navigation of the LumiaFS directory structure.
- **Task Manager**: Real-time monitoring of CPU usage, heap allocations, and process states.

## The C++ Transition
Historically, LumiaOS used a monolithic "Header-Only" design where implementation details were written inside `.h` files. The project is actively transitioning to a standard C++ model (`.h` for declarations, `.cpp` for definitions). The `build.sh` script automatically discovers and compiles all `.cpp` files in the `src/` directory.
