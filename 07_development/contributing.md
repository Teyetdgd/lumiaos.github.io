# Contributing to LumiaOS

Thank you for your interest in contributing to LumiaOS! Because this is a bare-metal project built from scratch without standard libraries, contributing requires a specific mindset and adherence to strict architectural rules.

## Core Philosophy: "Bare Metal, High Signal"
- **No external dependencies:** Do not attempt to `#include <iostream>` or link external C libraries. You must use the kernel's internal implementations (e.g., `print_string()`, `kmalloc()`).
- **No lazy code:** If you build a new feature, you must also build the necessary infrastructure to support it securely and stably.

## Repository Etiquette
1. **Never use manual `git` commands.** The repository history is managed strictly via automated PowerShell scripts.
2. To submit changes, use:
   ```powershell
   ./scripts/powershell/push-to-main.ps1 -Message "Added new feature X"
   ```
   *(Note: This applies primarily to the core maintainers. If you are submitting a PR on GitHub, follow standard fork-and-pull-request workflows, but ensure your commits are squashed logically).*

## Code Style
- **Indentation:** 4 spaces (no tabs).
- **Naming:** 
  - Functions: `snake_case` (e.g., `gui_draw_rect`).
  - Structs/Classes: `PascalCase` or `snake_case_t` (e.g., `LumiaWindow`, `pcb_t`).
- **File Structure:** We are actively transitioning from header-only implementations to a standard `.h` and `.cpp` model. Place declarations in `include/` and definitions in `src/`.

## The "No-Panic" Rule
If you are writing user-facing or GUI code, **never** use a hard `KASSERT` or `while(1) hlt;` loop for predictable errors (like failing to load an image or allocating a small buffer).
- Use proper error handling and return codes.
- Rely on the `setjmp/longjmp` Watchdog mechanism only as a last resort for catastrophic failures.
