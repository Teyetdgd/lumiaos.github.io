# Window Management

The Window Manager in LumiaOS handles the lifecycle, geometry, and user interactions for all GUI applications.

## Window Structure (`LumiaWindow`)

Every window is defined by a `LumiaWindow` struct (`include/gui/aero/aero_types.h`), which contains:
- **Geometry:** `x`, `y`, `width`, `height`.
- **State Flags:** `visible`, `minimized`, `maximized`, `active`.
- **Identity:** `id`, `pid`, `title`, `type` (used to route events to specific app handlers).
- **Callbacks:** Function pointers for `app_tick` (logic) and `app_cleanup` (destruction).

## Event Dispatching & Input Handling

All raw PS/2 mouse and keyboard interrupts are enqueued into circular buffers. The compositor drains these buffers in `lumia_process_input()` (`include/gui/aero/aero_input.h`).

### Hit Testing and Focus
When the user clicks the mouse:
1. The system performs a reverse-iteration (highest Z-Order to lowest) to find the first window bounding box that contains the `(x, y)` coordinates.
2. If a window is hit, it is pulled to the front (highest Z-Order) via `lumia_focus_window()`.
3. The click coordinates are then converted from *Screen Space* to *Window Space* (relative coordinates) and passed to the specific application's click handler (e.g., `browser_handle_click`).

### Dragging and Resizing
- **Dragging:** If a click lands on the title bar (and is not consumed by custom app controls like Browser Tabs), the compositor locks into dragging mode. Mouse delta movements directly update the window's `(x, y)` coordinates.
- **Resizing:** Intrusive edge-resizing has been intentionally disabled for stability. Instead, windows feature a dedicated **Resize Handle** in the bottom-right corner. Clicking and dragging this specific area updates the `width` and `height`, triggering a `layout_dirty` flag that forces the internal application (like the DOM engine) to recalculate its element positions.

## Application Sandboxing (Watchdog)

GUI applications run cooperatively within the compositor thread. To prevent a single poorly-written application from freezing the entire Desktop Environment:
- The compositor wraps the execution of every `app_tick` in a `setjmp` environment.
- The hardware PIT (Timer) monitors execution time. If an application hangs, the kernel triggers a `longjmp` back to the compositor, forcefully closing the misbehaving window and maintaining OS stability.
