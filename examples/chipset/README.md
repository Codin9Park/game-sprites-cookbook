# Cookbook Spritesheet Chipset

Ce README résume les étapes de bout en bout utilisées dans `examples/chipset` pour passer d'une vidéo IA à un spritesheet 3x4.

![Image originale de Chipset](01_sources/chipset_200x200.png)

> **Chipset** : Un perroquet robot personnage du jeu Golden Quest de Coding Park, qui indique à Cody les endroits où sont les trésors. Au plus profond de ses circuits endommagés, il détient un savoir-faire ancestral : la capacité de coder sans IA...

## Pré-requis

- FFmpeg
- ImageMagick (commandes : `magick`, `mogrify`)
- TexturePacker Free

## Organisation des dossiers

```
examples/chipset/
├── 01_sources/         # Image source à animer (input)
├── 02_ai_videos/       # Vidéos générées par IA
├── 03_frames_raw/      # frames extraites (brutes)
├── 04_frames_resized/  # frames redimensionnées
├── 05_frames_clean/    # frames avec background supprimé
├── 06_frames_selected/ # frames finales sélectionnées
└── 07_spritesheets/    # spritesheets exportés
```

## 0) Prompt IA utilisé pour générer l’animation (01 -> 02)

```
Animate this parrot hovering in place in a cartoonish style. He floats gently as if flying in place moving his wings up and down, as if stabilizing himself in the air. Keep it fun, playful, and cartoon-style. Make the loop seamless for reuse as an idle flying animation.
```

## 1) Extraire les frames de la vidéo IA (02 -> 03)

```bash
ffmpeg -i examples/chipset/02_ai_videos/chipset_fly.mp4 -vf "fps=12" examples/chipset/03_frames_raw/frame_%04d.png
```

Astuce : 8-12 fps suffisent souvent pour une boucle.

## 2) Redimensionner les frames (03 -> 04)

```bash
mogrify -path examples/chipset/04_frames_resized -resize x240 examples/chipset/03_frames_raw/frame_*.png
```

## 3) Supprimer le background (04 -> 05)

```bash
mogrify -path examples/chipset/05_frames_clean -fuzz 10% -transparent white examples/chipset/04_frames_resized/frame_*.png
```

Si vous voyez un halo blanc, lisser les bords :

```bash
mogrify -path examples/chipset/05_frames_clean -fuzz 10% -transparent white -morphology Erode Disk:1 examples/chipset/04_frames_resized/frame_*.png
```

## 4) Sélectionner les meilleures frames dans TexturePacker (05 -> 06)

Étape manuelle :
- Importer `examples/chipset/05_frames_clean/*.png`
- Prévisualiser l’animation
- Garder les 12 meilleures frames
- Exporter les PNG individuels dans `examples/chipset/06_frames_selected`

## 5) Exporter un spritesheet 3x4 (06 -> 07)

```bash
magick montage examples/chipset/06_frames_selected/*.png \
  -background none -alpha set \
  -tile 3x4 -geometry +0+0 \
  examples/chipset/07_spritesheets/spritesheet_3x4.png
```

## Notes

- Garder la taille du personnage et la distance caméra constantes dans la vidéo IA.
- Si les frames ont des tailles différentes, normaliser avant le montage.

## Copyright

- Vidéo générée avec Demo AI.
- Design original du personnage : Coding Park.
