# The Aero Compositor Engine

LumiaOS features a custom software-based windowing system heavily inspired by modern display servers (such as Wayland or DWM), referred to internally as the **Aero Compositor** (`src/display/aero_compositor.cpp`).

## Architecture

Unlike older operating systems that allowed applications to draw directly to the screen (leading to tearing and graphical artifacts), LumiaOS enforces strict isolation between application rendering and physical screen output.

### 1. The Backbuffer (Double Buffering)
All graphical operations are first drawn to an off-screen memory region known as the **Backbuffer** (allocated dynamically based on the current resolution, e.g., 8MB for 1024x768x32bpp).
- Once the compositor completes a full drawing cycle (background, icons, windows, cursor), it calls `flip_buffer()`.
- `flip_buffer()` copies the entire backbuffer to the physical hardware framebuffer (mapped at `0xFD000000` for BGA) in one rapid block transfer. This eliminates screen flickering and tearing.

### 2. The Render Loop
The `lumia_compositor_start()` function acts as the infinite loop for the GUI thread. In every iteration, it:
1. **Reads Input:** Processes mouse movement and keyboard keystrokes via `lumia_process_input()`.
2. **Executes App Logic:** Iterates through all active windows and calls their `app_tick()` functions, giving them CPU time to process networking, calculations, or layout changes.
3. **Paints:** If `needs_full_repaint` is true, it redraws the wallpaper, desktop icons, and windows (from lowest Z-Order to highest).
4. **Draws Cursor:** Overlays the hardware-independent mouse cursor on top of the final image.

## Visual Effects

### Alpha Blending (Transparency)
The compositor supports real-time software alpha blending. Each pixel in LumiaOS is 32-bit (ARGB). The `blend_colors()` function mathematically combines the background pixel with the foreground pixel based on the `alpha` channel, enabling:
- Semi-transparent window title bars (Aero Glass effect).
- Smooth drop shadows around active windows.
- Context menu overlay dimming.

### Taskbar Live Previews
LumiaOS includes a unique feature where hovering over a taskbar icon provides a real-time, scaled-down snapshot of the running application.
- When an application renders its content, the compositor intercepts the pixel data within the window's clipping bounds.
- It applies a fast nearest-neighbor scaling algorithm to fit the content into a fixed thumbnail size while preserving the aspect ratio.
