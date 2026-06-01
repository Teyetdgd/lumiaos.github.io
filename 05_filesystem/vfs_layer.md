# Virtual File System (VFS)

The Virtual File System (`src/fs/fs.cpp`) acts as an abstraction layer between user-space applications (or kernel subsystems) and the physical storage media.

## Architecture

In LumiaOS, an application never calls disk hardware drivers directly. Instead, it uses standard POSIX-like functions:
- `fs_open()`, `fs_read_file()`, `fs_write_file()`, `fs_change_directory()`.

The VFS layer takes these calls and looks at the **Mount Table** (`src/fs/mount_table.cpp`). The mount table maps logical paths (like `/home/`) to specific storage backends.

## Fallback Mechanism
A unique feature of LumiaOS is its resilient fallback system. 
- If the OS is booted from a Live CD (without an attached hard drive), the VFS automatically routes all file operations to **RamFS** (a volatile, in-memory filesystem).
- If a hard drive is detected and formatted, the VFS routes operations to **LumiaFS**, ensuring data persistence.
- This allows applications (like the Browser saving bookmarks) to use the exact same code regardless of the underlying hardware state.

## Device Abstraction (Dev Layer)
The VFS also manages the `/dev/` namespace (`src/fs/dev_layer.cpp`). Physical hardware devices, such as the ATA disk (`/dev/d0`), are exposed as files, allowing the filesystem drivers themselves to interact with hardware using standard read/write paradigms.
