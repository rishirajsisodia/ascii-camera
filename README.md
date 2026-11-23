# ASCII Cam

A browser app that turns your webcam into live ASCII art, inside a terminal-style window. It's a single static `index.html` with no build step, framework or install. Video is processed locally in your browser and never leaves your machine.

Its signature feature is the **hand frame**. You keep the normal camera view and pinch with both hands to form a rectangle in the air, and only the area inside that rectangle turns into ASCII.

## Features

- **Live ASCII video:** each frame is downscaled to a character grid, and every pixel's brightness picks a character from a dark-to-light ramp.
- **Camera picker:** lists every camera the browser can see (built-in, USB, or an iPhone via Continuity Camera) and switches between them live.
- **Hand frame:** pinch thumb and index finger together on **both** hands. The two pinch points become opposite corners of a glowing frame. ASCII shows only inside it, and the normal camera stays visible outside. Release a pinch and the frame disappears.
- **Text color:** pick any color for the characters.
- **Charset:** Classic (` .:-=+*#%@`), Detailed (a long 70-character ramp), Simple, or Blocks (` ░▒▓█`).
- **Width:** 40–200 characters across. A higher value gives more detail but is slower.
- **Invert:** flips the brightness mapping.
- **Mirror:** flips the picture like a mirror (on by default).
- **Use camera colors:** draws each character in the color of the pixel it replaces.
- **Save .txt:** saves the current ASCII frame as a text file.

## Quick start

Camera access only works on `localhost` or HTTPS, not when you open the file directly (`file://`). So serve the folder:

```bash
git clone https://github.com/rishirajsisodia/ascii-camera.git
cd ascii-camera
python3 -m http.server 8000
```

Then open **http://localhost:8000** and click **Allow** when the browser asks for camera access.

Any static server works (`npx serve`, `php -S localhost:8000`, etc.). To use it on other devices or share it, host it over HTTPS, for example on GitHub Pages or Netlify.

## How to use

1. Allow camera access. The camera dropdown fills with your devices.
2. With **Hand frame** on (the default), you see your normal camera view. Hold both hands up and pinch thumb to index finger on each. A glowing frame appears between your two pinch points, and ASCII renders inside it.
3. Move your hands apart or together to resize the frame, or move them to reposition it.
4. Untick **Hand frame** to turn the whole picture into ASCII instead.
5. Change the color, charset, width, invert and camera-colors options as you like. Use **Save .txt** to keep a frame.

**Tips**
- Good, even lighting and hands facing the camera improve hand tracking.
- Pinch with the fingertips visible, not hidden behind other fingers.
- If it lags, lower the width. "Use camera colors" is the slowest mode.

## How it works

Everything runs in one `requestAnimationFrame` loop in `index.html`:

1. A hidden `<video>` plays the camera stream (`getUserMedia`).
2. Each frame is drawn into a small `<canvas>` of `cols × rows`. Rows are derived from the video aspect ratio and a 0.6 glyph-width constant, so characters look correctly proportioned.
3. Each pixel's luminance (`0.299R + 0.587G + 0.114B`), contrast-stretched, indexes into the selected charset.
4. The result is written into a `<pre>` element, whose font size is recomputed so the text fills the window.
5. **Hand frame:** [MediaPipe Hands](https://developers.google.com/mediapipe) detects up to two hands and finds a pinch when thumb tip and index tip are close relative to palm size. The two pinch points form the frame rectangle, which is smoothed and held for about 250 ms to avoid flicker. Only cells inside the rectangle are drawn, over an opaque panel so the camera doesn't show through.

## Troubleshooting

| Problem | Fix |
|---|---|
| Nothing happens or the camera API is unavailable | Serve over `http://localhost` or HTTPS, not `file://`. |
| "Camera permission denied" | Allow the camera in the address bar's site settings, then reload. |
| "Camera is in use by another app" | Close Zoom, FaceTime or other apps using the camera. |
| Camera names look generic | Names only appear after permission is granted. |
| "Hand tracking model could not be loaded" | The hand model downloads from the jsDelivr CDN, so you need an internet connection. The Hand frame box unticks itself and the full picture is ASCII. |

## Requirements

- A modern browser with `getUserMedia` support (Chrome, Edge, Safari, Firefox).
- A camera.
- Internet access for the hand-tracking model (loaded from jsDelivr on first use).

## Project structure

```
index.html   the whole app (HTML, CSS and JavaScript)
CLAUDE.md    notes for AI coding assistants working on the repo
```
