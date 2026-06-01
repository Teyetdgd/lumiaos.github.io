# Getting Started with LumiaOS

Welcome to **LumiaOS**, a 32-bit (x86) bare-metal operating system built entirely from scratch. This guide will help you understand how to build the project and run it in an emulator.

## Prerequisites

LumiaOS requires a specific cross-compiled toolchain to ensure the generated code does not link against your host operating system's libraries (like Linux `glibc` or Windows `msvcrt`).

You will need the following tools:
1. **`i686-elf-gcc` and `i686-elf-ld`**: The standard cross-compiler for 32-bit x86 OS development.
2. **`nasm`**: The Netwide Assembler, used for compiling bootloaders, ISR stubs, and low-level context switching.
3. **`grub-mkrescue` & `xorriso`**: Required to pack the compiled kernel into a bootable `.iso` image using GRUB2.
4. **`qemu-system-i386`**: The recommended emulator for running and debugging the OS.

*(If you are on Windows, we highly recommend using WSL - Windows Subsystem for Linux - to set up these tools).*

## Building the OS

LumiaOS uses a custom bash script (`build.sh`) to handle the entire compilation, linking, and ISO generation process. 

To build the kernel, open your terminal and run:
```bash
./build.sh
```

If successful, the script will output a bootable image located at `build/lumia.iso`.

## Running in QEMU

You can boot the generated ISO directly in QEMU. However, LumiaOS supports advanced features like Networking and Persistent Storage, which require additional QEMU flags.

### Basic Boot (Live Mode)
```bash
qemu-system-i386 -cdrom build/lumia.iso -m 256M -vga std
```
*Note: In Live Mode, changes to the file system are written to RamFS and will be lost upon reboot.*

### Full Boot (With Hard Disk and Network)
To enable persistent storage and network connectivity:

1. **Create a virtual hard disk** (only needed the first time):
   ```bash
   dd if=/dev/zero of=build/disk.img bs=1M count=64
   ```
2. **Run QEMU**:
   ```bash
   qemu-system-i386 -cdrom build/lumia.iso -hda build/disk.img -m 256M -vga std \
   -netdev user,id=net0 -device e1000,netdev=net0
   ```

### First Boot Experience
When you boot with an empty `disk.img`, LumiaOS will recognize the unformatted drive and launch the **LumiaOS Setup Wizard**. Follow the on-screen prompts to format the disk (using LumiaFS) and create your user account. 

On subsequent boots, the system will start in **Installed Mode**, asking for your credentials before dropping you into the terminal or GUI.

## Starting the Desktop Environment
Once logged in via the command line interface (CLI), type the following command to launch the Aero Compositor and enter the graphical desktop:
```bash
user@lumia-pc:/$ gui
```
