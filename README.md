# Magic Hand — Gesture-Controlled Air Canvas

Draw in the air with your hand and watch your sketches turn into 3D objects — no mouse, no touchscreen, just a webcam.

Point your webcam at your hand and draw in mid-air. When you pause, the app recognises what you drew: a circle, triangle, square, rectangle, pentagon or hexagon becomes a clean 3D shape you can move, rotate and zoom; handwriting is recognised as text. Everything runs live on your camera feed with an on-screen HUD.

## How it works

1. **Hand tracking** — MediaPipe's Gesture Recognizer (`models/gesture_recognizer.task`, 2 hands) reads 21 hand landmarks per hand from the webcam.
2. **Gestures** — `hand_signals.py` turns landmarks into poses (pointing, fist, open hand, thumb up/down, V sign, 3-finger circle sign) and orientation signals (roll / yaw / pitch).
3. **Smoothing** — `filters.py` applies a One Euro filter so the cursor doesn't jitter.
4. **Canvas** — `canvas.py` holds strokes and the 3D objects they become.
5. **Recognition** — `sketch_recognition.py` classifies finished strokes as shapes, and recognises handwriting (Tesseract when available, otherwise a built-in template matcher). Slow handwriting recognition runs off the video loop in a worker thread (`async_recognition.py`).
6. **3D engine** — `mesh3d.py` rotates, projects, shades and casts shadows for the objects.
7. **Zoom lens** — `magnifier.py` magnifies the area inside the thumb/index circle (1.2×–4.5×).
8. **HUD** — `hud.py` paints the help text, status, cursor trail and debug info on top of the frame.

## Gestures

| Gesture | Action |
|---|---|
| Index finger, held | Draw (pause and the stroke becomes an object) |
| Closed fist | Move the selected object |
| Open hand over an object | Rotate it in 3D |
| Move your hand away | Deselect |
| Circle sign (3 fingers) | Zoom — raise the fingers to make the object bigger, lower them to make it smaller |
| Thumb up | Next colour |
| Thumb down | Delete the last drawing |
| V sign | Undo |
| Both hands open, then remove them | Clear everything |

## Keyboard shortcuts

| Key | Action |
|---|---|
| `q` | Quit |
| `h` | Toggle help |
| `o` | Toggle shadows |
| `c` | Next colour |
| `z` | Undo |
| `s` | Save a snapshot to `captures/pinceau_YYYYMMDD_HHMMSS.png` |
| `d` | Toggle debug view |

## Setup

**Requirements:** Python, a webcam, and (optionally) the Tesseract binary for better handwriting recognition.

```bash
pip install -r requirements.txt
```

Download the MediaPipe gesture model into `models/` (it is not included in the repo):

```powershell
Invoke-WebRequest -Uri "https://storage.googleapis.com/mediapipe-models/gesture_recognizer/gesture_recognizer/float16/1/gesture_recognizer.task" -OutFile "models\gesture_recognizer.task"
```

> `opencv-contrib-python` is required instead of plain `opencv-python` — the offline letter recogniser uses `cv2.ximgproc.thinning`, which the base package does not ship. Installing both makes them fight over the same `cv2` module, so install only the contrib build.

## Run

```bash
python gesture_canvas_manipulation.py
```

The app tries webcam indexes 0, 1 and 2, and prefers 1088×612, falling back to 1024×576, 960×540 and 640×480.

## Tech stack

- **Python** — OpenCV (`opencv-contrib-python` ≥ 4.8), MediaPipe (≥ 0.10), NumPy (≥ 1.24)
- **MediaPipe Gesture Recognizer** — hand landmarks + gesture classification, 2 hands, video mode
- **pytesseract** (optional) — handwriting recognition; without the Tesseract binary the app falls back to a built-in template matcher and keeps working

## Project structure

```
magic_hand/
├── gesture_canvas_manipulation.py   # Entry point: gestures, state machines, camera loop
├── hand_signals.py                  # Step 1 — read the hand from MediaPipe's 21 landmarks
├── filters.py                       # Step 2 — One Euro smoothing for noisy landmark signals
├── canvas.py                        # Step 3 — strokes, 3D objects, and the layer they live on
├── sketch_recognition.py            # Step 4 — a finished drawing becomes a shape, a letter, or stays a drawing
├── async_recognition.py             # Step 5 — handwriting recognition off the video loop (worker thread)
├── mesh3d.py                        # Step 6 — minimal 3D engine: rotate, project, shade, cast a shadow
├── magnifier.py                     # Step 7 — the zoom lens
├── hud.py                           # Step 8 — everything painted on top of the frame
├── requirements.txt
└── captures/                        # Created on first save (key `s`)
```

> Note: the on-screen labels and status text are in French (the window is titled "Main Pinceau").
