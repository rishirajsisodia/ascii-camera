# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A browser app that turns a live webcam feed into ASCII art, inside a terminal-styled window. The whole app is a single static file, `index.html` (inline CSS and JS), with no build step, dependencies, linter or tests.

## Running

Camera access (`getUserMedia`) only works on `localhost` or HTTPS, not `file://`. Serve the folder:

```
python3 -m http.server 8000   # then open http://localhost:8000
```

## Architecture

The JS runs in one pipeline, driven by a `requestAnimationFrame` loop (`frame()`):

1. A hidden `<video>` plays the camera stream, started by `start(deviceId)`. `listCams()` runs only after permission is granted, because device labels are empty before that. `camSel.onchange` restarts the stream with the chosen `deviceId`.
2. Each frame is drawn into a hidden `<canvas>` that is downscaled to `cols × rows`. `rows` is `cols * (videoH/videoW) * 0.6`, which compensates for glyphs being taller than they are wide. Mirroring is done with a canvas transform.
3. Each cell maps luminance (`0.299R + 0.587G + 0.114B`, contrast-stretched; flipped by the "Invert" checkbox) to a glyph in the selected charset, which is ordered dark to light. Cells outside the hand frame (when active) are blank.
4. The output goes to `<pre id="out">`. The default mode sets `textContent` and takes its color from the CSS variable `--ascii`, which the color picker updates. "Use camera colors" mode builds a `<span>` per character instead, which is much slower at high widths.
5. The `<pre>` font size is recomputed each frame so the text fits the container width (assumes a monospace glyph is about 0.6em wide). Changing the font or line-height requires revisiting the 0.6 glyph-width constant (used for both `rows` and the font fit), since the camera `<video>` is stretched underneath the `<pre>` and the two must line up.

`lastText` holds the most recent plain-text frame for the "Save .txt" button.

## Hand frame

With "Hand frame" on, MediaPipe Hands (also from jsdelivr, loaded lazily by `initHands()`) looks for a thumb–index pinch on both hands. The two pinch points are opposite corners of `rect` (normalized, already mirror-adjusted, smoothed and held for 250 ms). On by default. The normal camera `<video>` in `.stage` is visible (`.stage.framed`) beneath the `<pre>`, and ASCII is drawn only for cells inside `rect`; `#frameBox` outlines it and has an opaque background (z-index 0, under the `<pre>`) so only ASCII shows inside the frame, not the video. With the box unticked the video is hidden and the whole frame is ASCII.
