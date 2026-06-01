# Graphics Drivers (VGA & BGA)

LumiaOS bypasses the BIOS (which is inaccessible in 32-bit Protected Mode) and communicates directly with graphics hardware via MMIO (Memory Mapped I/O) and I/O ports.

## Standard VGA (Legacy Mode)

During early boot or in environments without advanced graphics support, LumiaOS can fallback to standard VGA mode `0x13` (320x200 with 256 colors).
- Initialization is done by writing specific byte sequences directly to the VGA CRTC, Sequencer, and Graphics Controller ports (`0x3C0` - `0x3D5`).
- While functional, this mode is deprecated for the primary desktop experience and is primarily used for testing or ultra-low-resource environments.

## Bochs Graphics Adapter (BGA)

The primary display driver for LumiaOS is the **BGA (Bochs Graphics Adapter)** driver (`src/display/vga.cpp`). BGA is a widely supported paravirtualized graphics interface used by QEMU, VirtualBox, and Bochs.

### PCI Detection
During boot, the PCI bus is scanned (`src/net/pci.cpp`) for the specific Vendor ID (`0x1234`) and Device ID (`0x1111`) associated with the BGA device.

### Initialization Sequence
1. **Version Verification:** The driver reads the VBE `INDEX` register (`0x01CE`) to ensure the emulator supports at least BGA version 5.
2. **Resolution Request:** It writes the desired resolution (e.g., `1024` to the X-Res register and `768` to the Y-Res register) and sets the bit depth to `32` (ARGB).
3. **Enable VBE:** Finally, it writes the `VBE_DISPI_ENABLED` and `VBE_DISPI_LFB_ENABLED` flags to the Enable register.

### The Linear Framebuffer (LFB)
Once enabled, the BGA device exposes a Linear Framebuffer—a continuous block of physical memory (usually mapped at `0xFD000000`) representing the screen pixels.
- The memory manager (`paging.cpp`) maps this large physical chunk (e.g., 16MB) into the kernel's virtual address space.
- The `Aero Compositor` uses this mapped pointer to execute its rapid `flip_buffer()` block transfers.
