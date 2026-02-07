## 🔧 Developer Details & Architecture

### System Overview
**OMIKUJI // MASTER_CORE** is a single-file, zero-dependency (runtime) web audio workstation. It relies entirely on the HTML5 **Web Audio API** for sound generation and scheduling. The architecture is designed to be "hackable," operating without build tools (Webpack/Vite) or frameworks (React/Vue).

### 🛠 Tech Stack
* **Core:** Vanilla JavaScript (ES2020+), HTML5, CSS3.
* **Audio Engine:** Web Audio API (`AudioContext`).
* **File Handling:** `JSZip` (v3.10.1 via CDN) for session packing.
* **Styling:** CSS Custom Properties (Variables) for theming.

### 🧠 Core Logic & Signal Flow

#### 1. The Audio Scheduler (`sch()`)
The sequencer uses a **Lookahead Scheduling** technique (similar to the metronome logic in standard web audio tutorials).
* **`sch()`**: A recursive function driven by `setTimeout`. It fires every 25ms.
* **`nxt` (Next Note Time)**: Tracks the precise audio time for the next 16th note.
* **`ctx.currentTime`**: If `nxt` falls within the lookahead window (0.1s), the system schedules the oscillator/buffer immediately.
* **Swing Calculation**: The `swingEngine.getOffset()` modifies the `time` variable before scheduling, pushing even-numbered steps forward in time without altering the grid index.

#### 2. Signal Chain
The audio graph is reconstructed dynamically per trigger:
```mermaid
[Source (Oscillator/Buffer)] 
      ⬇
[Track Gain (vV)] 
      ⬇
[Route Selector (Switch)] ➡ (If FX) ➡ [Plugin Input] ➡ [Plugin Output]
      ⬇                                      ⬇

OMIKUJI // MASTER_CORE_v32 // SUB_ZERO
OPERATIONAL MANUAL & REFERENCE GUIDE
System Overview: The Master Core is a web-based, 16-step sequencing environment designed for sample manipulation and melodic arrangement. It features a "Sub-Zero" aesthetic, dynamic track management, Javascript-based FX injection, and a song-mode arranger.
1. INITIALIZATION
Upon loading the application, the system enters a "Safety Lock" state (black screen).
Boot System: Click anywhere on the >> INITIALIZE_SYSTEM << screen to start the audio engine.
Audio Context: This action unlocks the browser's AudioContext and allows sound output.
2. TRANSPORT & GLOBAL CONTROLS
Located at the top of the Chassis.
PLAY: Toggles the sequencer Start/Stop.
Note: Stopping the engine performs a "Soft Kill" on all active audio buffers to prevent hanging notes.
BPM (Tempo):
Adjust: Click ▲ or ▼ to change tempo by 1 BPM.
Fast Adjust: Hold SHIFT while clicking arrows to change by 5 BPM.
Manual Entry: You can type directly into the number box.
CPY / PST:
CPY: Copies the currently selected pattern (1-8) into the clipboard.
PST: Pastes the clipboard data into the currently active pattern.
EXP (Export Session): Downloads the entire project (samples + sequencing data + settings) as a .omikuji file (ZIP archive).
IMP (Import Session): Loads a previously saved .omikuji file.
MODE: Switches the sequencer logic:
MODE: PAT: Loops the currently selected single pattern indefinitely.
MODE: SONG: Follows the "Arranger Matrix" sequence (see Section 5).
3. THE RACK (Track & Sequencer)
The main workspace where audio is manipulated.
A. Track Controls (Left Side)
T#: Track Indicator.
P (Piano Toggle): Opens/Closes the Piano Drawer for melodic programming (see below).
X (Delete): Removes the track from the rack entirely.
L (Load Sample): Opens the file browser to load an audio file (.wav, .mp3) into this track.
Behavior: This replaces the current sample buffer. The filename will appear below the controls.
Route Selector: A dropdown menu (defaulting to "MASTER").
Use this to route audio audio into specific FX Machines (see Section 6).
Volume Slider: Horizontal slider at the bottom of the control block. Adjusts individual track gain.
B. Step Grid (Top Row)
Toggle Steps: Click any of the 16 boxes to enable/disable a trigger.
Visual Feedback:
Cyan: Active step, currently playing.
White: Active step, waiting to play.
Dim Blue: Inactive step.
Mono-Trigger Logic: Samples on the same track never overlap. Triggering a step cuts off the previous sound on that track immediately.
C. Velocity Grid (Bottom Row)
Adjust Velocity: Below every step is a vertical bar (The V-Box).
Click & Drag: Click inside the box or drag up/down to set the volume (Velocity) for that specific step.
Full Height: 100% Volume.
Bottom: 0% Volume.
D. Piano Drawer (Melodic Mode)
Accessible by clicking the "P" button on a track.
The Matrix: A vertical representation of notes (C to C).
Set Pitch: Click a note to assign that pitch to the specific step.
Clear Pitch: Clicking an active (glowing) note removes the trigger.
OCTAVE Slider: Adjusts the base octave of the track (0-6). Changing this shifts the pitch of all notes in the track immediately.
4. TRACK MANAGEMENT
Located in the tab bar (SEQ / FX / +TRK).
+TRK: Adds a new empty track to the bottom of the rack.
Default State: New tracks are initialized with a default sawtooth "beep" until a sample is loaded.
Scrolling: If you add many tracks, the Rack area becomes scrollable.
5. ARRANGER MATRIX (Song Mode)
Located at the top of the SEQ page.
Concept: This allows you to chain patterns (1-8) together to create a full composition.
+ Button: Adds a new slot to the end of the song sequence.
- Button: Removes the last slot from the sequence.
Changing Patterns: Click the Block (P1, P2...) to cycle through pattern numbers 1 through 8.
Playback: Ensure MODE is set to SONG. The engine will play the pattern in the first block, then move to the second block, etc.
Pattern Navigation Bar: Located directly below the Arranger.
Buttons 1-8: Manually switches the currently viewed/edited pattern.
Note: You can edit Pattern 5 while the Song Mode is playing Pattern 1.
6. FX SYSTEM (Inject Machine)
Click the FX tab to access the hardware bay.
+ INJECT MACHINE: Loads a Javascript file as an audio effect processor.
Workflow:
Click + INJECT MACHINE.
Select a valid .js effect file (Formatted as a Web Audio Class).
The UI for the FX appears in the FX slot.
Routing: Go back to the SEQ tab. On any track, change the Route Selector from "MASTER" to the ID of the new FX (e.g., FX_A1B2).
Audio from that track now flows through the FX code before hitting the Master output.
DEL: Removes the FX unit and automatically reroutes all connected tracks back to MASTER.
7. MASTER & SWING
Bottom Left Panel.
Master Volume: Controls the final output gain of the entire system.
Swing Amount: A slider from 0% to 100%.
Note: This requires a Swing Engine to be loaded.
LOAD_SWING: Loads a Javascript file defining the groove/micro-timing logic.
Default: The system starts with "STRAIGHT" timing (mathematically perfect 16ths).
8. RECORDER (Resampling)
Bottom Right Panel.
ARM REC:
Click to "Arm" the tape deck. The button turns Cyan.
Press PLAY. The system begins capturing the Master Output.
Press PLAY (Stop) to end recording.
SAVE .WAV:
Appears only after a recording is finished.
Click to download the recording as REC.wav.
Preview Player: A standard audio player appears to audition the recording before downloading.
9. KEYBOARD SHORTCUTS / HIDDEN FEATURES
Shift + BPM Arrows: Adjust tempo in increments of 5.
File Drag & Drop: (Partial support depending on browser) Most operations rely on the file picker buttons (L, IMP, INJECT).
System Monitor: The scrolling text log at the bottom displays system status, errors, and confirmations.
10. TECHNICAL SPECIFICATIONS
Audio Engine: HTML5 Web Audio API.
Sample Rate: Dependent on host system (usually 44.1kHz or 48kHz).
Project File: .omikuji (JSON data + Binary Blobs zipped).
Pattern Length: Hardcoded to 16 steps per pattern.
Polyphony:
Per Track: Monophonic (New triggers choke old ones).
System: Unlimited (Bounded only by CPU).
[Master Gain (mG)] ⬅⬅⬅⬅⬅⬅⬅⬅⬅⬅⬅⬅⬅⬅⬅
      ⬇
[Audio Destination] & [MediaStreamDestination (Recorder)]
