# Arduino Music Visualizer

A desktop application that plays MP3 audio through your computer speakers, analyzes songs for beats using librosa, displays synchronized .lrc lyrics, and streams real-time display data to an Arduino UNO with a parallel LCD and LED beat visualization.

The laptop handles **all** audio playback, timing, beat detection, and lyric scheduling. The Arduino is purely a display and LED controller — it receives pre-computed packets and never calculates timing itself.

## Hardware Requirements

- Arduino UNO
- 16x2 LCD (HD44780, parallel interface, **NOT** I2C)
- 4 LEDs (for beat visualization)
- Resistors (220Ω for LEDs, 10K potentiometer for LCD contrast)
- Breadboard and jumper wires

### LCD Wiring

| LCD Pin | Arduino Pin |
|---------|-------------|
| RS      | 12          |
| E       | 11          |
| D4      | 5           |
| D5      | 4           |
| D6      | 3           |
| D7      | 2           |
| VSS     | GND         |
| VDD     | 5V          |
| V0      | Potentiometer (contrast) |
| A       | 5V (backlight) |
| K       | GND (backlight) |

### LED Wiring

| LED | Arduino Pin |
|-----|-------------|
| 1   | 6           |
| 2   | 7           |
| 3   | 8           |
| 4   | 9           |

Each LED should be connected through a 220Ω resistor to GND.

## Software Requirements

- Python 3.10+
- Arduino IDE (to upload the firmware)

## Installation

1.  **Clone or download** this repository.
2.  **Upload firmware**: Open `firmware/music_visualizer/music_visualizer.ino` in the Arduino IDE and upload to your Arduino UNO.
3.  **Start the application** using the batch script:
    ```
    start.bat
    ```

    The batch script will automatically:
    1. Create a Python virtual environment (if missing).
    2. Activate it.
    3. Upgrade pip.
    4. Install required dependencies.
    5. Launch the desktop application.

## Usage

### Song Files

Place MP3 files and optional synchronized .lrc lyric files in the `songs/` folder:

```
songs/
    Believer.mp3
    Believer.lrc
    Numb.mp3
    Numb.lrc
```

The .lrc file must have the **exact same filename** (case-insensitive) as the MP3 and be in the **same folder**. When you select an MP3, the application automatically:

1. **Searches for a matching .lrc** — If `Believer.lrc` exists next to `Believer.mp3`, it is loaded. If not, playback continues with "No synchronized lyrics found."
2. **Checks the beat cache** — Previously analyzed songs load instantly from `.cache/beat_analysis/`. Unchanged songs are never re-analyzed.
3. **Analyzes only when needed** — If no cache exists, librosa analyzes the song once, saves the cache, and subsequent loads reuse it.

### Selecting a Song

- **Library**: Double-click any MP3 in the library list (auto-scanned from `songs/`).
- **Open MP3...**: Use the file picker to select any MP3 from anywhere on your computer.

The .lrc search and beat cache work the same way for both methods.

### Playback

| Control | Behavior |
|---------|----------|
| Play | Starts or resumes playback. Lyrics and beats sync to the current position. |
| Pause | Pauses audio. Lyrics and beat timing freeze. |
| Stop | Stops playback, clears the Arduino LCD. |
| Seek | Drag the progress bar. Lyrics and beats immediately resync to the new position. |

### Synchronization

The laptop's audio playback position is the **single master clock**:

- **Computer speakers**: Play audio.
- **LCD lyrics**: Display the current and next lyric line based on playback position.
- **LEDs**: Flash on beat events scheduled from cached beat timestamps.

Pausing, resuming, seeking, or changing songs instantly resynchronizes all three outputs.

## How It Works

### Audio Playback

MP3 files play through your **computer speakers using pygame**. The Arduino never plays audio — it is only a display and LED controller.

### Beat Detection

When an MP3 is selected for the first time, librosa analyzes the audio waveform and detects beat timestamps. These timestamps are cached as a JSON file in `.cache/beat_analysis/` (keyed by file path + modification time). Subsequent loads reuse the cache.

During playback, a BeatScheduler compares the current playback position against the cached beat timestamps and fires beat events at the correct moments.

### Lyrics

The application loads `.lrc` files with the same base name as the selected MP3. It supports:
- **Line-level timestamps** (`[00:12.34]Lyric text`) — Per-word timing is estimated by dividing the line duration evenly across words.
- **Word-level timestamps** (`<00:12.50>word`) — Precise timing is used directly.

If no `.lrc` file is found, the lyric display shows "No synchronized lyrics found." during playback.

### Arduino Role

The Arduino **only**:
- Receives serial packets (`LYRIC`, `BEAT`, `CLR`, `PING`)
- Displays text on the 16x2 LCD
- Animates 4 LEDs on beat events

It does **no** MP3 decoding, timing calculation, beat detection, or audio playback. All intelligence is in the Python application.

## Configuration

Edit `config.yaml` to customize:

| Key | Default | Description |
|-----|---------|-------------|
| `audio.cache_dir` | `.cache/beat_analysis` | Beat analysis cache location |
| `audio.sample_rate` | `22050` | Sample rate for librosa analysis |
| `arduino.baud_rate` | `115200` | Serial baud rate (must match firmware) |
| `arduino.reconnect_interval` | `2` | Seconds between reconnection attempts |
| `ui.window_width` | `800` | Initial window width |
| `ui.window_height` | `600` | Initial window height |
| `directories.songs` | `./songs` | Song library folder |

## Project Structure

```
├── config.yaml              # Application configuration
├── requirements.txt         # Python dependencies
├── start.bat                # Windows launcher
├── .gitignore
├── README.md
├── songs/                   # Place your MP3 and .lrc files here
├── src/
│   ├── main.py              # Entry point
│   ├── config.py            # Configuration loader
│   ├── logger_config.py     # Logging setup
│   ├── audio/
│   │   ├── cache.py         # Beat analysis cache manager
│   │   ├── analyzer.py      # Librosa beat detection
│   │   └── player.py        # MP3 playback + beat scheduling
│   ├── lyrics/
│   │   ├── parser.py        # .lrc file parser
│   │   └── renderer.py      # Timed lyric rendering
│   ├── arduino/
│   │   ├── protocol.py      # Serial packet encoding
│   │   ├── detector.py      # Port scanning and detection
│   │   └── communicator.py  # Serial I/O with auto-reconnect
│   └── ui/
│       ├── app.py           # Main application window
│       ├── song_browser.py  # MP3 file browser
│       ├── player_controls.py  # Play/pause/seek controls
│       ├── lyric_display.py # Lyric text display
│       └── status_bar.py    # Status indicators
└── firmware/
    └── music_visualizer/
        └── music_visualizer.ino  # Arduino sketch
```
"# LCD-LYRICS-VISUALIZER" 
"# LCD-LYRICS-VISUALIZER" 
