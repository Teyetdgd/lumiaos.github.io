# Built-in Applications

LumiaOS ships with a suite of built-in GUI applications, serving as both utilities for the user and reference implementations for developers.

### File Explorer
- **Location:** `include/apps/file_explorer.h`
- A graphical interface for the Virtual File System. It allows users to navigate directories, click on files, and automatically launch associated programs (e.g., opening `.bmp` files in the Image Viewer).

### Task Manager
- **Location:** `include/apps/taskmgr.h`
- Reads directly from the kernel's process table and heap tracking structures. It displays real-time statistics including CPU usage, total RAM, and per-process memory consumption (respecting the 4MB Watchdog limit).

### LumiaVim
- **Location:** `include/editor/lumiavim.h`
- A modal text editor inspired by Vim. It supports basic Normal and Insert modes, allowing users to edit configuration files or scripts directly from the command line or GUI terminal.

### Games
- **Location:** `include/apps/games_app.h`
- To demonstrate the capabilities of the Aero Compositor, the OS includes several classic games:
  - **Minesweeper:** Demonstrates complex grid rendering and mouse-click logic.
  - **Snake & Tetris:** Demonstrate keyboard event handling and game-loop timing tied to the PIT.
