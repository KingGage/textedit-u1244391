R1
Version 1.0.0 (week 1)
Core Features
File Operations

Create new files
Open existing files
Save files
Save As with custom file path
Auto-detect file encoding
Track file modification status

Text Editing

Full text editing capabilities
Undo/Redo support
Cut, Copy, Paste operations
Select All functionality
Configurable tab size
Auto-indentation

Search & Replace

Find text with next/previous navigation
Replace single or all occurrences
Case-sensitive search option
Whole word matching
Regular expression support

User Interface

Menu bar with File, Edit, View, and Help menus
Toolbar with quick-access buttons
Status bar showing line/column position, encoding, and modification status
Cross-platform native look and feel

Keyboard Shortcuts

Standard shortcuts for all common operations
Customizable keybindings
Platform-appropriate defaults (Cmd on macOS, Ctrl on Windows/Linux)

Settings

Persistent user preferences
Font customization
Tab size configuration
Auto-indent toggle


R2
Version 2.0.0 (week 2)
Added additional features

Dark Mode support: 
- Enable/Disable dark mode 
Inverts the colors for a more comfortable viewing experience. Had to continually remind the AI to make sure things were dark mode compatible because it sometimes forgot. 

File explorer
- Navigate through files and directories
- Open files for editing
- Create new files and directories
- Delete files and directories
- Can open and close projects from the file
This one was surprisingly easy and pretty much was done in one shot by the AI. It still only has basic functionality, but it's a good start. can be removed from the window and used all by itself in its own window which I thought was a cool idea. 

Split View
- Open multiple files side by side
- Navigate between files using tabs
- Split view can be toggled on and off
This one was harder as files would not be opened in the split view. The AI had to be reminded to open the files from the file explorer in the selected split view. It was a bit of a pain but it was worth it and it worked out fine. still cant figure out how to drag between split views which I think would be good but its not working yet. 


R3
Version 3.0.0 (Week 3)

    Find and replace 
    - updated the find function to highlight found matches instead of giving the number of matches in a popup
    - added functionality across open files for the find and replace. (only files that are open in the editor)
    - Gives a warning if replacing with an empty string.
    - Gives how many matches across how many files
    - Does not work with Undo or redo 
    Did not initially test the functionality of just the find until presenting day and thought the AI made a terrible design choice but my fault. it is now updated to be much more user friendly by highlighting the matches across files. In just the find option the replace text was still there originally but not used so I needed the replace stuff to be removed from just the find option. 

    Dark Mode
    - added dark mode compatibility to the pop up windows. 
    Like Ive stated in previous notes the dark mode compatibility is always something I need to mention to the AI for it to do it otherwise just thinks in standard light mode. 

    Indentation, quote, and bracket matching. 
    - added indentation, quote, and bracket matching to the editor. 
    - matching quotes and brackets highlight when cursor is next to one of them. 
    - adds a second automatically if you add one. 
    - tab spacing is saved when you press enter. 
    - backspace by tabs on new lines. 
    These didnt take too long to implement but the bracket and spacing took a couple tries to get correct. When pressing enter between brackets the bracket would go down one but then tab to the right which looked off because its different from other editors. after a brief description of how it should work it was able to fix it easy. The highlighted matching brackets would sometimes just stop working between prompts somehow but got that all fixed after reminding the AI.


# Frame Timer Documentation

## Overview

The Frame Timer is a performance monitoring widget that tracks the time taken to process individual frames in the text editor. It measures user-triggered rendering performance by monitoring the time between when a user input event is detected and when the corresponding frame update completes.

## Features

- **Last Frame Time**: Shows the duration (in milliseconds) of the most recent user-triggered frame
- **Average Frame Time**: Displays the average duration across all recorded frames
- **Max Frame Time**: Tracks the longest frame duration since the timer was enabled
- **Idle Frame Exclusion**: Only measures frames triggered by user input; idle redraws (cursor blink, OS repaints) are not counted
- **Toggle Control**: Show/hide the timer with **Ctrl+P**

## How It Works

### Frame Timing Mechanism

1. **User Input Detection**: When the user performs an action (keypress, mouse click, scroll), the editor triggers the `start_frame()` method and sets a flag indicating user input
2. **Frame Measurement**: The elapsed time between frame start and end is measured using Python's `time.time()`
3. **Recording**: The frame time is recorded (in milliseconds) and used to calculate statistics
4. **Idle Frames Ignored**: If a frame renders without user input, it is not counted

### Supported Input Events

- **Keyboard**: Any key press (including text input, navigation, special keys)
- **Mouse**: Click, drag, and selection operations
- **Scroll**: Wheel scroll events

### Statistics Tracking

- **Last Frame Time**: The duration of the most recent measured frame
- **Average Frame Time**: Sum of all frame times divided by the number of frames
- **Max Frame Time**: The longest recorded frame duration since enabling the timer

## Usage

### Toggle Frame Timer (Ctrl+P)

Press **Ctrl+P** to toggle the frame timer visibility:
- **Hidden State**: Timer widget is not displayed, and no frame timings are recorded
- **Visible State**: Timer widget is displayed at the bottom of the editor window, and frame timings are actively recorded

### Reading the Display

```
Last: 2.35 ms    Avg: 1.89 ms    Max: 5.43 ms
```

- **Last**: Duration of the most recent user-triggered frame
- **Avg**: Average duration across all recorded frames since the timer was enabled
- **Max**: Maximum duration of any frame since the timer was enabled

### When to Use

Use the Frame Timer to:
- Identify slow frame processing during specific interactions
- Monitor performance of text editing operations (typing, pasting, scrolling)
- Detect performance regressions after code changes
- Verify that idle operations (cursor blink) are not being counted

## Expected Performance

### Typical Values

On a modern system with an empty file and no user interaction:
- **Idle Frames (not measured)**: ~500ms between redraws (cursor blink)
- **User Input Frames**: 2-5ms for text input
- **Complex Operations**: May range 5-20ms (e.g., large paste, complex selection)

### Verification

To verify that idle frames are being excluded:
1. Enable the frame timer (Ctrl+P)
2. Open an empty file
3. Leave the file untouched with the timer visible
4. The displayed timings should remain near zero or show very small values (< 5ms)
5. When you press a key or click, you'll see the frame time update with the user-triggered event

If you see persistent ~500ms values without user interaction, the idle frame exclusion is not working correctly.

## Implementation Details

### FrameTimer Class

Located in `textEditor.py`, the `FrameTimer` class manages all timing logic:

```python
class FrameTimer:
    def start_frame() -> None       # Mark frame start (if user input triggered)
    def end_frame() -> None         # Record frame end time
    def set_user_input_triggered()  # Flag that user input triggered this frame
    def get_average_frame_time()    # Calculate average of recorded frames
    def reset()                     # Clear all timing data
    def set_enabled(enabled: bool)  # Enable/disable recording
```

### FrameTimerWidget Class

Provides the UI display component:
- Updates every 100ms
- Shows last, average, and max frame times
- Respects dark/light mode theming
- Automatically stops updating timer when hidden

### Integration Points

The frame timer is integrated into:
- **TextEditor.keyPressEvent()**: Records key-triggered frames
- **TextEditor.mousePressEvent()**: Records mouse click frames
- **TextEditor.wheelEvent()**: Records scroll frames
- **MainWindow.keyPressEvent()**: Handles Ctrl+P toggle

## Troubleshooting

### Timer Shows No Data

**Problem**: Frame timer is visible but shows all zeros
- **Solution**: Perform a user action (type a character, click, scroll) to trigger frame measurement

### Timing Values Are Too High

**Problem**: Seeing consistent 50ms+ values on simple operations
- **Solution**: Check if system is under high load, or if heavy operations are being performed

### Idle Frames Are Being Counted

**Problem**: Seeing persistent 400-500ms values without user interaction
- **Solution**: This indicates cursor blink or OS-triggered redraws are being measured. Verify that `set_user_input_triggered()` is only called in response to actual user input events, not internal Qt signals.