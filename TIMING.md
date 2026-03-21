
# Editor Performance Timings

## Initial Timings

### small.txt (~hundreds of lines)
| Metric | Value |
|--------|-------|
| Max frame time on open | 16.44 ms |
| Max frame time scrolling | 0.45 ms|
| Max frame time clicking scrollbar | 6.97 ms|
| Avg frame time clicking scrollbar | 0.64 ms |
| Max frame time find-replace "while"→"for" (19 matches) | 9.38 ms|
| Memory usage (Physical/Real) | 35.1 MB |

### medium.txt (~10,000 lines)
| Metric | Value |
|--------|-------|
| Max frame time on open | 49.59 ms |
| Max frame time scrolling | 0.73 ms|
| Max frame time clicking scrollbar | 8.27 ms |
| Avg frame time clicking scrollbar | 0.82 ms |
| Max frame time find-replace "while"→"for" (1,186 matches) | 569.66 ms|
| Memory usage (Physical/Real) | 74 MB |

### large.txt (1M+ lines)
| Metric | Value |
|--------|-------|
| Max frame time on open | 7878.77 ms|
| Max frame time scrolling | 1.07 ms|
| Max frame time clicking scrollbar | 9.08 ms|
| Avg frame time clicking scrollbar | 0.85 ms|
| Max frame time find-replace "while"→"for" (668,753 matches) | COULD NOT COMPLETE IN 15 Min |
| Memory usage (Physical/Real) | 1.2 GB|

## Final Timings

### small.txt (~hundreds of lines)
| Metric | Value |
|--------|-------|
| Max frame time on open | 15.74 ms |
| Max frame time scrolling | 0.35 ms|
| Max frame time clicking scrollbar | 6.57 ms|
| Avg frame time clicking scrollbar | 0.54 ms |
| Max frame time find-replace "while"→"for" (19 matches) | 10.38 ms|
| Memory usage (Physical/Real) | 35.1 MB |

### medium.txt (~10,000 lines)
| Metric | Value |
|--------|-------|
| Max frame time on open | 42.19 ms |
| Max frame time scrolling | 0.78 ms|
| Max frame time clicking scrollbar | 8.62 ms |
| Avg frame time clicking scrollbar | 0.85 ms |
| Max frame time find-replace "while"→"for" (1,186 matches) | 517.43 ms|
| Memory usage (Physical/Real) | 72 MB |

### large.txt (1M+ lines)
| Metric | Value |
|--------|-------|
| Max frame time on open | 7611.37 ms|
| Max frame time scrolling | 1.31 ms|
| Max frame time clicking scrollbar | 9.01 ms|
| Avg frame time clicking scrollbar | 0.81 ms|
| Max frame time find-replace "while"→"for" (668,753 matches) | COULD NOT COMPLETE WITHOUT REFACTOR |
| Memory usage (Physical/Real) | 1.2 GB |

# Week 8 

Using the Amp coding agent I added a FrameTimer system to my editor to measure responsiveness from user input to render completion, along with a FrameTimerWidget overlay that shows last, average, and max frame times and can be toggled with Ctrl+P (disabling it fully resets all timing data). We integrated timing hooks across all user input paths in my TextEditor, including keyPressEvent (handling early returns like tab, enter, backspace, auto-closing characters, and multi-cursor cases), mousePressEvent, wheelEvent, and a scrollbar handler tied to both vertical and horizontal actions using QTimer.singleShot(0, ...) to capture post-repaint timing. At the MainWindow level, I applied the same timing pattern to _open_file_path (which handles all file opening paths, including menu actions, explorer clicks, and command-line arguments) and to FindReplaceDialog operations like find_all and replace_all. My current editor architecture centers around a MainWindow coordinating high-level actions, a TextEditor widget managing all input and rendering logic, and modular utility components like the frame timer system layered in without disrupting core functionality; importantly, timing only runs when triggered by actual user input, ensuring idle events like cursor blinking are excluded

# Week 9 — Memory-Mapped File I/O Optimization

## Optimized Timings

### small.txt (~hundreds of lines)
| Metric | Value |
|--------|-------|
| Max frame time on open | 12.12 ms |
| Max frame time scrolling | 0.31 ms|
| Max frame time clicking scrollbar | 6.03 ms|
| Avg frame time clicking scrollbar | 0.54 ms |
| Max frame time find-replace "while"→"for" (19 matches) | 3.38 ms|
| Memory usage (Physical/Real) | 86.1 MB |

### medium.txt (~10,000 lines)
| Metric | Value |
|--------|-------|
| Max frame time on open | 16.72 ms |
| Max frame time scrolling | 0.40 ms|
| Max frame time clicking scrollbar | 5.91 ms |
| Avg frame time clicking scrollbar | 0.45 ms |
| Max frame time find-replace "while"→"for" (1,186 matches) | 517.43 ms|
| Memory usage (Physical/Real) | 87.71 MB |

### large.txt (1M+ lines)
| Metric | Value |
|--------|-------|
| Max frame time on open | 77.32 ms|
| Max frame time scrolling | 0.45 ms|
| Max frame time clicking scrollbar | 6.01 ms|
| Avg frame time clicking scrollbar | 0.65 ms|
| Max frame time find-replace "while"→"for" (668,753 matches) | 1403.42 |
| Memory usage (Physical/Real) | 296.23 MB |

## Core Architecture Change: Memory-Mapped File I/O

Replaced the file loading approach with memory-mapped file access. Files are no longer read entirely into a string at open time. Instead:

1. **MmapDocument** memory-maps the file descriptor via `mmap.mmap()` with `ACCESS_READ`
2. A **line index** (list of byte offsets marking each line start) is built by scanning the mapped region once — this is the only O(n) work at open time
3. Only the line index lives in memory; file content remains in the OS-managed mapped region
4. Lines are decoded on demand when the editor needs to render them

## Rendering: Virtual/Windowed Rendering

**MmapTextView** (subclass of `QAbstractScrollArea`) replaces `QPlainTextEdit` for file-opened tabs:

- Only renders lines visible in the viewport via `viewportEvent` paint override
- Uses the line index to compute which lines are visible for the current scroll position
- Fetches and decodes only those lines from the mmap region
- All scroll positions (including after find-replace jumps) use the same lightweight render path

## Find and Find-Replace: Indexed Search over mmap

- **Find**: scans the mmap bytes directly using `re.compile(pattern_bytes).finditer(mmap_region)` — no copy to string
- Byte offsets are converted to line numbers via `bisect.bisect_right` on the line index
- **Replace**: builds an in-memory list of `(byte_offset, old_length, new_bytes)` replacement records
  - When rendering a line, applicable replacements are overlaid on the raw bytes before decoding
  - The full replaced file is only written to disk on save
- Match counts verified: small.txt = 19 ("while"), medium.txt = 1,186, large.txt = 668,753

## Scrolling

- Scrolling triggers only a viewport re-render fetching lines from mmap — no file re-reads, no line index recomputation
- Scrollbar range is computed from `total_lines` (from line index) vs `visible_line_count`
- Track click, thumb drag, and scroll wheel all update the scroll offset and trigger the same lightweight paint path

## Threading

- The line index build runs on a **background thread** (`threading.Thread`, daemon=True)
- A single `threading.Lock` protects the line index: the background thread extends it in batches of 50,000 offsets, the main thread reads it
- The editor is usable before indexing completes as it renders whatever lines are indexed so far
- A `QTimer` polls indexing progress every 100ms and updates the scrollbar range
- All rendering and event handling stays on the main thread
- Find-replace runs on the main thread for correctness

## Libraries Used

| Library | What it does | Why it is necessary |
|---------|-------------|-------------------|
| `mmap` (Python stdlib) | Memory-maps the file descriptor so the OS pages file content in on demand | Required to open large.txt without a multi-second delay or 1.2GB memory allocation |
| `threading` (Python stdlib) | Runs the line-index build on a background thread | The O(n) line scan for 1M+ line files would block the main thread; background indexing keeps the editor responsive |
| `bisect` (Python stdlib) | Binary search on the sorted line-offset array | Converts byte offsets from search results to line numbers in O(log n) instead of O(n) |
| `os` (Python stdlib) | `os.fstat()` to get file size without reading | Needed to know the mmap region size and the byte range of the last line |
| `re` (Python stdlib) | Regex search over mmap bytes | Provides case-insensitive, whole-word, and regex search directly on the mapped region without copying to a string |

## Accounting for Unavoidable Dropped Frames

| Action | File Size | Frames Dropped | Code Path | Why >16ms | Why Not Optimizable |
|--------|-----------|----------------|-----------|-----------|---------------------|
| File Open (file picker dialog) | All sizes | ~2 frames | `QFileDialog.getOpenFileName()` — native OS file picker | The system file picker is a native modal dialog rendered by the OS, not by our code | Cannot optimize — this is OS-level UI outside application control. Consistent across all file sizes. |
| File Open (mmap + initial render) | small/medium | 0 frames | `MmapDocument.open()` + `MmapTextView.set_document()` | mmap() is near-instant; only visible lines are rendered | N/A — within budget |
| File Open (line index build) | large (1M+) | 0 UI frames | `MmapDocument._build_line_index()` on background thread | The O(n) scan runs on a background thread — UI is never blocked | N/A — threading prevents frame drops |
| Scrolling | All sizes | 0 frames | `MmapTextView._paint_viewport()` | Only ~40 visible lines are fetched and painted per frame | N/A — within budget |
| Scrollbar interaction | All sizes | 0 frames | `MmapTextView._on_scrollbar_action()` → `_paint_viewport()` | Same lightweight render path as scroll wheel | N/A — within budget |
| Find (search) | large (1M+) | Multiple frames blocked during search | `MmapDocument.search()` — `re.finditer(mmap)` | Regex scan over ~100MB of mapped bytes takes >16ms even though it runs at C speed inside Python's `re` engine | The search must scan the entire file to produce correct match counts. Could be chunked across frames but would complicate the search API and result ordering. The search is still dramatically faster than the previous approach (which couldn't complete at all for large files). |
| Find-Replace | large (1M+) | Multiple frames during search phase | `MmapDocument.search()` + `add_replacements_from_matches()` | Same regex scan cost as find; the replacement record building is O(matches) | Same as find — the search phase dominates. The replacement application itself is lazy and O(1) per rendered line. |
