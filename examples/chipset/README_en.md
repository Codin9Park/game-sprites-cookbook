# Chipset Spritesheet Cookbook

This README summarizes the end-to-end steps used in `examples/chipset` to go from an AI-generated video to a 3x4 spritesheet.

![Chipset original image](01_sources/chipset_200x200.png)

> **Chipset**: A robot parrot character from the game Golden Quest by Coding Park who shows Cody where the treasures are. Deep in his damaged circuits, he carries an ancestral know-how: the ability to code without AI...

## Prerequisites

- FFmpeg
- ImageMagick (commands: `magick`, `mogrify`)
- TexturePacker Free

## Folder layout

```
examples/chipset/
├── 01_sources/         # Original image to animate (input)
├── 02_ai_videos/       # AI-generated videos
├── 03_frames_raw/      # extracted frames (raw)
├── 04_frames_resized/  # resized frames
├── 05_frames_clean/    # background removed frames
├── 06_frames_selected/ # final selected frames
└── 07_spritesheets/    # exported spritesheets
```

## 0) AI prompt used to generate the animation (01 -> 02)

```
Animate this parrot hovering in place in a cartoonish style. He floats gently as if flying in place moving his wings up and down, as if stabilizing himself in the air. Keep it fun, playful, and cartoon-style. Make the loop seamless for reuse as an idle flying animation.
```

## 1) Extract frames from the AI video (02 -> 03)

```bash
ffmpeg -i examples/chipset/02_ai_videos/chipset_fly.mp4 -vf "fps=12" examples/chipset/03_frames_raw/frame_%04d.png
```

Tip: 8-12 fps is usually enough for a loop.

## 2) Resize frames (03 -> 04)

```bash
mogrify -path examples/chipset/04_frames_resized -resize x240 examples/chipset/03_frames_raw/frame_*.png
```

## 3) Remove background (04 -> 05)

```bash
mogrify -path examples/chipset/05_frames_clean -fuzz 10% -transparent white examples/chipset/04_frames_resized/frame_*.png
```

If you see a white halo, smooth the edges:

```bash
mogrify -path examples/chipset/05_frames_clean -fuzz 10% -transparent white -morphology Erode Disk:1 examples/chipset/04_frames_resized/frame_*.png
```

## 4) Select best frames in TexturePacker (05 -> 06)

Manual step:
- Import `examples/chipset/05_frames_clean/*.png`
- Preview the animation
- Keep the best 12 frames
- Export individual PNGs into `examples/chipset/06_frames_selected`

## 5) Export a 3x4 spritesheet (06 -> 07)

```bash
magick montage examples/chipset/06_frames_selected/*.png \
  -background none -alpha set \
  -tile 3x4 -geometry +0+0 \
  examples/chipset/07_spritesheets/spritesheet_3x4.png
```

## Notes

- Keep the character size and camera distance constant in the AI video.
- If frames have inconsistent sizes, normalize them before the montage.

## Copyright

- Video generated using Demo AI.
- Original character design: Coding Park.
