# LumiaFS and Disk Operations

While LumiaOS supports FAT32 parsing for USB drives (`fat32.cpp`), its primary persistent storage mechanism is a custom filesystem known as **LumiaFS**.

## Block Device Layer (ATA/IDE)
To read from the hard drive, LumiaOS uses the Block IO Layer (`src/fs/disk_io.cpp`).
- It communicates with the IDE controller via Port I/O (`0x1F0` primary bus).
- Uses **LBA28** (Logical Block Addressing) to read and write specific 512-byte sectors.
- Provides a simple interface: `disk_read_sectors()` and `disk_write_sectors()`.

## LumiaFS Architecture
LumiaFS (`src/fs/lumiafs.cpp`, `diskfs_core.cpp`) is a simple, robust file system designed specifically for the needs of this kernel.

### The Superblock
Located at a fixed LBA (Sector 5050), the Superblock contains the filesystem metadata:
- Magic Number (`0x4C554D49` - "LUMI").
- Total Sectors and Free Sectors.
- Pointer to the Root Directory.

### Directory Structure
Directories are stored as linked lists of structures on disk. Each entry contains:
- Filename (up to 32 characters).
- Attributes (Is Directory, Is Hidden).
- `start_sector`: The LBA where the file's data begins.
- `file_size`: The exact size of the file in bytes.

### The Installer
When the `installer_app.cpp` formats a new disk, it completely overwrites the Superblock, clears the root directory sectors, and creates the default OS hierarchy (`/LumiaOS/home`, `/LumiaOS/system`).
