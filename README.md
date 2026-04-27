# OLEDFrame — Video → Arduino Bitmap Converter

A self-contained, single-file web app that converts any video into monochrome bitmap frames ready to display on an SSD1306 or SH1106 OLED using the Adafruit library. No installation, no server — just open `oled-video-converter.html` in a browser. Please note all code was generated with Claude code.

---

## Features

- **Drag-and-drop video loading** — MP4, WebM, MOV, AVI
- **Display size selection** — 128×64 or 128×32 (standard SSD1306 / SH1106 sizes)
- **Threshold control** — 0–255 brightness cutoff for black-and-white conversion
- **Floyd-Steinberg dithering** — optional; produces much better results on gradients and faces
- **Colour inversion** — swap black and white pixels
- **Horizontal / vertical flip** — independent toggles, baked into the bitmap data
- **Configurable frame count** — extract 1–128 frames, spread evenly across the selected range
- **Video trim** — dual-handle timeline with a live thumbnail strip; preset shortcuts for first half, second half, and middle
- **OLED preview** — accurate white-on-black pixel simulation with play/pause and adjustable FPS (1–60)
- **Frame scrubber** — click any thumbnail to inspect and select individual frames
- **Arduino code output** — complete `.ino` sketch or raw PROGMEM arrays, ready to copy or download

---

## Usage

### 1. Open the file

Open `oled-video-converter.html` in any modern browser (Chrome, Firefox, Edge, Safari). No internet connection required after the page loads.

### 2. Load a video

Drag a video file onto the drop zone, or click it to browse. Supported formats depend on your browser's codec support — MP4 (H.264) works everywhere.

### 3. Configure

| Setting | Description |
|---|---|
| **Display Size** | Match your physical OLED: 128×64 or 128×32 |
| **Threshold** | Pixels with brightness ≥ threshold become white; below become black |
| **Dithering** | Floyd-Steinberg error diffusion — better for photos, slower to process |
| **Frame Count** | How many frames to extract from the video (or trim range) |
| **Invert** | Swap black and white — useful if your subject is dark on a light background |
| **Flip H / V** | Mirror the image horizontally or vertically before conversion |

### 4. Trim (optional)

Click **ADVANCED OPTIONS** to expand the trim panel.

- Drag the **green IN handle** (left) to set the start of the clip
- Drag the **green OUT handle** (right) to set the end
- Or click anywhere on the timeline bar to snap the nearest handle to that position
- Use the preset buttons — **RESET**, **1ST HALF**, **2ND HALF**, **MIDDLE** — for quick ranges

Frame extraction always spans exactly the trimmed range.

### 5. Process

Click **▶ PROCESS VIDEO**. A progress bar shows extraction status. When complete:

- The OLED preview updates with frame 0
- Thumbnails appear in the frame scrubber strip
- Arduino code is generated automatically

### 6. Preview

Use the OLED preview to check results before copying code:

- **◀ PREV / NEXT ▶** — step through frames one at a time
- **▶ PLAY / ■ STOP** — animate at the selected FPS
- **FPS slider** — 1 to 60 FPS for playback and code output
- Click any thumbnail in the strip to jump to that frame

### 7. Export

Choose your output mode:

| Mode | Description |
|---|---|
| **All frames** | Export every processed frame |
| **Selected frame** | Export only the currently highlighted frame |
| **Animation sketch** | Full `.ino` with `setup()`, `loop()`, frame table, and PROGMEM arrays |
| **Arrays only** | Just the raw `uint8_t PROGMEM` data — for integrating into an existing sketch |

Click **⧉ COPY TO CLIPBOARD** or **↓ DOWNLOAD .ino**.

---

## Arduino Setup

### Required libraries

Install via the Arduino Library Manager:

- **Adafruit SSD1306**
- **Adafruit GFX Library**

### Wiring (I2C)

| OLED Pin | Arduino Uno | Arduino Nano |
|---|---|---|
| VCC | 3.3V or 5V | 3.3V or 5V |
| GND | GND | GND |
| SDA | A4 | A4 |
| SCL | A5 | A5 |

### I2C address

The generated sketch defaults to `0x3C`. If your display doesn't respond, try `0x3D` — change the `SCREEN_ADDR` define at the top of the sketch.

### Memory considerations

Each frame is stored in `PROGMEM` (flash memory), not SRAM. The sizes below are per frame:

| Display | Bytes per frame |
|---|---|
| 128×64 | 1024 bytes (1 KB) |
| 128×32 | 512 bytes (0.5 KB) |

An Arduino Uno has 32 KB of flash. Practical limits after accounting for the sketch itself:

| Display | Safe max frames |
|---|---|
| 128×64 | ~24 frames |
| 128×32 | ~48 frames |

Boards with more flash (Mega, ESP32, RP2040) can hold significantly more.

---

## Generated Sketch Structure

```cpp
// PROGMEM bitmap array for each frame
static const uint8_t PROGMEM frame_0[] = { ... };
static const uint8_t PROGMEM frame_1[] = { ... };

// Pointer table — also in PROGMEM
const uint8_t* const frames[] PROGMEM = { frame_0, frame_1, ... };
const uint16_t FRAME_COUNT = 16;
const uint16_t FRAME_DELAY = 83;  // ms (12 fps)

void loop() {
  for (uint16_t i = 0; i < FRAME_COUNT; i++) {
    display.clearDisplay();
    display.drawBitmap(0, 0,
      (const uint8_t*)pgm_read_ptr(&frames[i]),
      SCREEN_WIDTH, SCREEN_HEIGHT, SSD1306_WHITE);
    display.display();
    delay(FRAME_DELAY);
  }
}
```

Bitmaps are packed **MSB-first, row by row** — the format `Adafruit_GFX::drawBitmap()` expects natively.

---

## Tips

- **High contrast source footage** gives the best results at threshold — talking heads, cartoons, and silhouettes work especially well
- **Dithering** helps with photos and gradients but increases visual noise at small sizes; try both and compare
- **Invert** is useful when your OLED background is white in the original content — it maps better to how OLEDs consume power (dark pixels = off)
- **Flip H** is handy if your display is mounted upside-down or mirrored in an enclosure
- For smooth animation on Uno, keep frame count low (8–12) and FPS around 10–15
- The **Selected frame** export mode is useful for extracting a single logo or splash screen

---

## Browser Compatibility

All processing is done locally using the Canvas API and the browser's built-in video decoder. No data is uploaded anywhere.

| Browser | Status |
|---|---|
| Chrome / Edge 90+ | ✅ Full support |
| Firefox 90+ | ✅ Full support |
| Safari 15+ | ✅ Full support |
| Mobile Chrome / Safari | ✅ Works (touch drag supported on trim handles) |

---

## File Structure

```
oled-video-converter.html   — the entire app, self-contained
README.md                   — this file
```

---

## License

MIT — do whatever you like with it.
