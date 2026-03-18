
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

'''
