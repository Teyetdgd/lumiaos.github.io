# Building Applications (SDK)

While LumiaOS has built-in applications (like the Browser and Explorer) compiled directly into the kernel, it also supports executing external, dynamically loaded programs using the `.lxe` (Lumia Executable) format.

## The Lumia SDK
Located in the `sdk/` directory, the SDK provides the necessary headers and a custom linker script to compile standard C code into a format LumiaOS can run.

### Why not ELF?
While standard ELF files are powerful, writing a full ELF loader and dynamic linker in a minimal kernel is complex. The `.lxe` format is a streamlined, flat-binary format with a specific header that the LumiaOS `bin_loader.h` can easily parse, map into memory, and execute.

## Writing a "Hello World" App

1. Create a C file (`hello.c`):
   ```c
   #include "lumia.h"

   void main() {
       sys_print("Hello from Lumia SDK!\n");
       sys_exit(0);
   }
   ```

2. Notice that we include `"lumia.h"`. This header provides wrappers around the `int 0x80` system calls, allowing you to interact with the kernel (printing to the terminal, opening files, etc.) without knowing the underlying interrupt mechanics.

## Compiling
The SDK uses a custom toolchain process:
1. `gcc` compiles the C code into an object file.
2. `ld` links it using `sdk/toolchain/linker.ld` to ensure the code expects to be loaded at the correct virtual address (typically `0x40000000`).
3. A custom tool (`elf2lxe.c`) strips the unnecessary ELF metadata and prepends the `.lxe` header.

```bash
cd sdk/examples
make
```

## Running Your App
Once compiled, you will get a file like `hello.lxe`. 
1. Copy this file to your LumiaOS hard drive (e.g., into `/LumiaOS/apps/`).
2. Boot LumiaOS.
3. Open the Terminal and type:
   ```bash
   run /LumiaOS/apps/hello.lxe
   ```
