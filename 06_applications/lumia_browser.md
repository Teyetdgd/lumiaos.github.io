# Lumia Browser

The crown jewel of LumiaOS applications is the **Lumia Browser** (`include/apps/browser.h`). It is a from-scratch web rendering engine embedded directly into the operating system.

## The Rendering Pipeline

Unlike terminal-based browsers (like Lynx), Lumia Browser aims to replicate the modern Box Model of HTML/CSS.

1. **Chunked Decoding:** When the HTTP response arrives, if it uses `Transfer-Encoding: chunked`, the browser decodes it in-place to extract raw HTML.
2. **DOM Parser:** The HTML string is fed into `dom_parse()` (`browser_dom.h`). This builds a tree of `DOMNode` structures (tags, text, attributes) using an arena allocator for speed.
3. **CSS Engine:** `css_collect_from_html()` parses any `<style>` blocks in the document. It generates a cascading stylesheet rule set.
4. **Layout (Box Model):** The `dom_layout_block()` function recursively walks the DOM tree. It calculates the `X`, `Y`, `Width`, and `Height` of every element based on CSS rules (margin, padding, display type).
5. **Painting:** Finally, `dom_paint()` executes hardware framebuffer commands (`draw_rect`, `draw_string`) based on the computed geometry.

## Browser Features

### Smart Download Manager
When a user clicks a link (which is hit-tested via an array of `BrowserLink` bounding boxes generated during the layout phase), the browser checks the extension.
- If it ends in `.bin`, `.zip`, `.exe`, or `.bmp`, the browser prevents navigation.
- It displays a visual Download Bar at the bottom of the window.
- Upon completion, the raw bytes are automatically saved to the VFS path `/LumiaOS/home/user/downloads/`.

### Bookmarks System
Users can save sites via the `*` (Star) button.
- Bookmarks are serialized into a string format and written permanently to `/LumiaOS/config/bookmarks.txt`.
- The `lumia://bookmarks` internal URL generates a dynamic HTML page by reading this file from the disk, demonstrating tight integration between the OS file system and the browser rendering engine.
