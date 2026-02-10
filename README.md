# Cookbook - Create Your Animated Character (AI video -> frames -> spritesheet)

This guide explains an end-to-end method to create a spritesheet from:
1) images of a character (front / back / left / right)
2) an AI-generated video (DemoAI / DomoAI)
3) a post-process pipeline (FFmpeg + ImageMagick + optional tools)
4) selection/preview in TexturePacker Free
5) a final export (spritesheet + preview)

> Goal: a student can apply the recipe step by step to integrate animations into a game.

---

## 0) Prerequisites

### Tools to install
- FFmpeg
- ImageMagick (commands: `magick`, `mogrify`, `montage`)
- TexturePacker Free (to preview / choose frames / export)
- An image editor (your choice):
  - recommended: Photopea (web) or GIMP (local)

### Install (macOS)
```bash
brew install ffmpeg imagemagick
```

### Install (Windows)
- FFmpeg: install + add to PATH (or use "winget" if available)
- ImageMagick: install the version that includes magick.exe

### Verify installation
```bash
ffmpeg -version
magick -version
```

## 1) Repository layout

```
.
├── assets/
├── examples/
│   └── chipset/
│       ├── 01_sources/
│       ├── 02_ai_videos/
│       ├── 03_frames_raw/
│       ├── 04_frames_resized/
│       ├── 05_frames_clean/
│       ├── 06_frames_selected/
│       ├── 07_spritesheets/
│       ├── README.md
│       └── README_fr.md
├── tools/
├── INSTALL.md
├── README.md
└── README_fr.md
```

If you create a new character, follow the same numbered folder structure as `examples/chipset`.

## 2) AI step: generate a usable video for a spritesheet

### 2.1 Important tips (to avoid unusable videos)

- The character must stay the same size (no camera push / no zoom).
- Movement should be "in place": walk/run is a loop, not moving across the scene.
- Style: cartoon / consistent with the source image.
- No shape changes: keep proportions, volumes, colors, shading.

### 2.2 Base prompts (examples)

Walk cycle (in place)

 Animate this character doing a looping walk cycle in place.
 Keep the exact same body proportions, size, camera angle and lighting.
 Do NOT move toward the camera, do NOT scale up or down.
 The character stays centered, feet animate naturally like a game walk loop.
 Cartoon style, smooth but simple, loopable motion.

Idle (light breathing)

 Animate this character in an idle loop.
 No walking, no talking. Only subtle breathing (tiny up/down), occasional eye blink.
 Keep the exact same size, proportions, and camera angle. Loopable.

Action (throw / punch / jump)

 Animate a short action that can loop cleanly (start -> action -> return to idle pose).
 Keep constant size and camera distance; never move toward the camera.
 Cartoon style, no deformation of the character design.


### 2.3 Video export

- Download the video as MP4 (if possible without watermark)
- Put the file in videos/:
  - walk_front.mp4, walk_back.mp4, idle_right.mp4, etc.


## 3) Extract frames from a video (FFmpeg)

### 3.1 Extract at constant FPS (recommended)

Example: 12 frames/second

```bash
mkdir -p frames
ffmpeg -i videos/walk_front.mp4 -vf "fps=12" frames/walk_front_%04d.png
```

> Tip: 8-12 fps is often enough for "video game" spritesheets.

### 3.2 Reduce the number of frames (1 of 2, 1 of 3)

1 out of 2 frames

```bash
mkdir -p frames
ffmpeg -i videos/walk_front.mp4 -vf "select=not(mod(n\,2))" -vsync vfr frames/walk_front_%04d.png
```

## 4) Resize frames (ImageMagick)

### 4.1 Resize by percentage (keeps ratio)

```bash
mkdir -p frames_resized
mogrify -path frames_resized -resize 85% frames/walk_front_*.png
```

### 4.2 Resize to a fixed size (forces ratio)

If you want a target height (ex: 240 px tall) and accept forcing:

```bash
mkdir -p frames_resized
mogrify -path frames_resized -resize x240 frames/walk_front_*.png
```

> -resize x240 = fixes height to 240, width adjusts automatically.


## 5) Remove background (if needed)

> Important: many AI videos export without alpha (white/black background).

In that case, you need to cut out the background:
- either with an AI tool (remove.bg, etc.)
- or with ImageMagick when the background is almost uniform (beware of halos)

### 5.1 White background -> transparent (with fuzz)

```bash
mkdir -p frames_clean
mogrify -path frames_clean -fuzz 10% -transparent white frames_resized/walk_front_*.png
```

### 5.2 Reduce a halo (optional)

```bash
mkdir -p frames_clean
mogrify -path frames_clean -fuzz 10% -transparent white -morphology Erode Disk:1 frames_resized/walk_front_*.png
```

> Use only if edges keep a white outline.

## 6) Flip an animation (Left -> Right)

### 6.1 Flip all frames

```bash
mkdir -p frames_flipped
mogrify -path frames_flipped -flop frames_clean/walk_left_*.png
```

> -flop = horizontal mirror (left <-> right)


## 7) Create a spritesheet (ImageMagick) - assembly

### 7.1 Assemble an NxM grid

Example: 4 columns x 3 rows

```bash
mkdir -p spritesheets
magick montage frames_selected/*.png \
  -background none -alpha set \
  -tile 4x3 -geometry +0+0 \
  spritesheets/walk_front.png
```

> -background none prevents a white background from reappearing during assembly.


## 8) TexturePacker Free: select & export cleanly

### 8.1 Why TexturePacker
- Preview the animation
- Remove ugly frames
- Check the loop
- Stable final export

### 8.2 Recommended workflow
1. Import images (frames_clean or frames_resized)
2. Preview animation
3. Keep only useful frames (ex: 8-12)
4. Export as spritesheet (grid or pack) depending on the engine

> For Golden Quest / Pygame: prefer a regular grid.


## 9) Copy-paste recipes

### 9.1 Video -> frames -> resize -> spritesheet (no remove bg)

```bash
mkdir -p frames frames_resized frames_selected spritesheets

ffmpeg -i videos/idle_right.mp4 -vf "fps=12" frames/idle_right_%04d.png
mogrify -path frames_resized -resize x240 frames/idle_right_*.png

# (manual selection) copy a few frames into frames_selected/
magick montage frames_selected/*.png -background none -alpha set -tile 4x3 -geometry +0+0 spritesheets/idle_right.png
```

### 9.2 Video -> frames -> resize -> remove bg -> spritesheet

```bash
mkdir -p frames frames_resized frames_clean frames_selected spritesheets

ffmpeg -i videos/walk_front.mp4 -vf "fps=12" frames/walk_front_%04d.png
mogrify -path frames_resized -resize x240 frames/walk_front_*.png
mogrify -path frames_clean -fuzz 10% -transparent white frames_resized/walk_front_*.png

# (selection) frames_clean -> frames_selected
magick montage frames_selected/*.png -background none -alpha set -tile 4x3 -geometry +0+0 spritesheets/walk_front.png
```

### 9.3 Spritesheet -> explode -> flip -> reassemble

Example (frames 118x210, grid 4x3)

```bash
mkdir -p tmp_left tmp_right spritesheets

magick input_left.png -crop 118x210 +repage +adjoin tmp_left/frame_%02d.png
mogrify -path tmp_right -flop tmp_left/*.png

magick montage tmp_right/*.png -background none -alpha set -tile 4x3 -geometry +0+0 spritesheets/output_right.png
```

## 10) Common problems & solutions

### A) The character gets bigger / moves toward the camera

-> Strengthen the prompt:
- "in place"
- "no camera movement"
- "constant size"
- "do not move toward the camera"

### B) White background returns during assembly

-> Add:
- -background none -alpha set in montage

### C) White halo around the character after remove bg
- lower or adjust -fuzz (ex: 6-12%)
- try -morphology Erode Disk:1 (carefully)

### D) Frames do not all have identical dimensions

-> Normalize:

```bash
mogrify -path frames_resized -resize x240 -extent 240x240 -gravity center frames/*.png
```
(Use if you must force a common canvas)


## 11) Recommended naming (to avoid getting lost)

- idle_front.png, idle_back.png, idle_left.png, idle_right.png
- walk_front.png, walk_back.png, walk_left.png, walk_right.png
- throw_left.png, throw_right.png
- etc.

## Appendices (coming soon)
- Video tutorial (step by step)
- "before integrating into the game" checklist
- Prompt templates per animation (idle / walk / talk / throw / hit / die)
