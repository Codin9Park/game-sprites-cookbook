# Tools & Notes

This file collects practical notes about the tools used in this cookbook.
You do NOT need to understand everything at once.
Follow the steps, copy the commands, and adjust if needed.

---

## FFmpeg

Used to:
- extract frames from AI-generated videos
- resize frames
- reassemble animations (if needed)

### Common issues
- Transparency is NOT preserved if the video format does not support alpha.
- Most AI-generated videos have a white or black background → background removal is a separate step.

---

## ImageMagick (mogrify / convert)

Used to:
- resize many images at once (batch)
- flip images horizontally (left ↔ right)
- crop frames to exact sprite size
- build spritesheets (`magick montage`)

### Important
- Always work on copies of your frames.
- Prefer `-filter Lanczos` or `-resize` for sprites.

---

## Background Removal

There is NO perfect automatic solution.

Options:
- Online tools (remove.bg, Canva, etc.) → fast but limited
- Manual cleanup with GIMP → slower but precise
- AI background removal → results may vary

### Known issues
- White halos around characters
- Missing pixels near edges
- Color bleeding

These are normal problems in sprite workflows.

---

## GIMP

Used for:
- manual cleanup
- checking transparency
- selecting best frames
- fixing artifacts

### Important settings
- Always check that the image has an alpha channel
- When resizing: try "NoHalo" or "LoHalo"
- Zoom in to inspect edges before exporting

---

## TexturePacker (Free Edition)

Used to:
- preview animations
- choose which frames to keep
- export sprite sheets

### Notes
- Free edition is enough for learning and workshops
- Final sprite size must match your game engine expectations

---

## General Advice

- Do not try to automate everything.
- Always validate visually.
- Fewer good frames > many bad frames.
- Small imperfections are acceptable in games.
