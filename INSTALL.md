# Installation

## Required tools
- FFmpeg
- ImageMagick
- GIMP
- TexturePacker (free edition)

## macOS (brew)
brew install ffmpeg imagemagick

## Windows (PC)

### Option A: winget (recommended if available)
```powershell
winget install Gyan.FFmpeg
winget install ImageMagick.ImageMagick
winget install GIMP.GIMP
winget install CodeAndWeb.TexturePacker
```

### Option B: manual installers
1. Download and install FFmpeg, then add it to PATH.
2. Install ImageMagick (make sure `magick.exe` is included).
3. Install GIMP.
4. Install TexturePacker Free.

### Verify installation
```powershell
ffmpeg -version
magick -version
```
