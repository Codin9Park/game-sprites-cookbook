# Cookbook — Créer son personnage animé (vidéo IA → frames → spritesheet)

Ce guide explique une méthode **de bout en bout** pour créer un spritesheet à partir :
1) d’images d’un personnage (front / back / left / right)  
2) d’une vidéo générée par IA (DemoAI / DomoAI)  
3) d’un pipeline de post-process (FFmpeg + ImageMagick + outils optionnels)  
4) d’une sélection/visualisation dans **TexturePacker Free**  
5) d’un export final (spritesheet + aperçu)

> Objectif : qu’un élève puisse **appliquer la recette** étape par étape pour intégrer ses animations dans un jeu.

---

## 0) Pré-requis

### Outils à installer
- **FFmpeg**
- **ImageMagick** (commande `magick`, `mogrify`, `montage`)
- **TexturePacker Free** (pour visualiser / choisir les frames / exporter)
- Un éditeur d’image (au choix) :
  - conseillé : **Photopea** (web) ou **GIMP** (local)

### Installation (macOS)
```bash
brew install ffmpeg imagemagick
```

### Installation (Windows)
	•	FFmpeg : installer + ajouter au PATH (ou utiliser “winget” si dispo)
	•	ImageMagick : installer la version qui inclut magick.exe

### Vérifier l’installation
```bash
ffmpeg -version
magick -version
```

## 1) Structure du dépôt

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

Si tu crées un nouveau personnage, suis la même structure numérotée que `examples/chipset`.

## 2) Étape IA : générer une vidéo utilisable pour un spritesheet

### 2.1 Conseils importants (pour éviter les vidéos “inexploitables”)

	•	Le personnage doit rester à taille constante (pas d’approche caméra / pas de zoom).
	•	Mouvement “sur place” : walk/run doit être un cycle, pas un déplacement dans le décor.
	•	Style : cartoon / cohérent avec l’image source.
	•	Pas de morphologie qui change : garder proportions, volumes, couleurs, shading.

### 2.2 Prompts de base (exemples)

Walk cycle (sur place)

 Animate this character doing a looping walk cycle in place.
 Keep the exact same body proportions, size, camera angle and lighting.
 Do NOT move toward the camera, do NOT scale up or down.
 The character stays centered, feet animate naturally like a game walk loop.
 Cartoon style, smooth but simple, loopable motion.

Idle (respiration légère)

 Animate this character in an idle loop.
 No walking, no talking. Only subtle breathing (tiny up/down), occasional eye blink.
 Keep the exact same size, proportions, and camera angle. Loopable.

Action (throw / punch / jump)

 Animate a short action that can loop cleanly (start → action → return to idle pose).
 Keep constant size and camera distance; never move toward the camera.
 Cartoon style, no deformation of the character design.


### 2.3 Export vidéo

	•	Télécharger la vidéo en MP4 (si possible sans watermark)
	•	Mettre le fichier dans videos/ :
	    •	walk_front.mp4, walk_back.mp4, idle_right.mp4, etc.


## 3) Extraire les frames d’une vidéo (FFmpeg)

### 3.1 Extraction à FPS constant (recommandé)

Exemple : 12 images/seconde

```bash
mkdir -p frames
ffmpeg -i videos/walk_front.mp4 -vf "fps=12" frames/walk_front_%04d.png
```

> Astuce : 8–12 fps est souvent suffisant pour des spritesheets “jeu vidéo”.

### 3.2 Réduire le nombre de frames (1 sur 2, 1 sur 3)

1 frame sur 2

```bash
mkdir -p frames
ffmpeg -i videos/walk_front.mp4 -vf "select=not(mod(n\,2))" -vsync vfr frames/walk_front_%04d.png
```

## 4) Redimensionner les frames (ImageMagick)

### 4.1 Redimensionner en pourcentage (garde le ratio)

```bash
mkdir -p frames_resized
mogrify -path frames_resized -resize 85% frames/walk_front_*.png
```

### 4.2 Redimensionner à une taille fixe (force le ratio)

Si tu veux une hauteur cible (ex : 240 px de haut) et tu acceptes de forcer :

```bash
mkdir -p frames_resized
mogrify -path frames_resized -resize x240 frames/walk_front_*.png
```

> -resize x240 = fixe la hauteur à 240, largeur ajustée automatiquement.


## 5) Supprimer le background (si nécessaire)

> ⚠️ Important : beaucoup de vidéos IA sont exportées sans alpha (fond blanc/noir).

Dans ce cas, il faut “détourer” :
	•	soit avec un outil IA (remove.bg, etc.)
	•	soit avec ImageMagick quand le fond est quasi uniforme (mais attention aux halos)

### 5.1 Fond blanc → transparent (avec fuzz)

```bash
mkdir -p frames_clean
mogrify -path frames_clean -fuzz 10% -transparent white frames_resized/walk_front_*.png
```

### 5.2 Réduire un halo (optionnel)

```bash
mkdir -p frames_clean
mogrify -path frames_clean -fuzz 10% -transparent white -morphology Erode Disk:1 frames_resized/walk_front_*.png
```

> À utiliser seulement si les bords gardent un liseré blanc.

## 6) Flipper une animation (Left → Right)

### 6.1 Flipper toutes les frames

```bash
mkdir -p frames_flipped
mogrify -path frames_flipped -flop frames_clean/walk_left_*.png
```

> -flop = miroir horizontal (gauche ↔ droite)


## 7) Créer un spritesheet (ImageMagick) — assemblage

### 7.1 Assembler une grille NxM

Exemple : 4 colonnes × 3 lignes

```bash
mkdir -p spritesheets
magick montage frames_selected/*.png \
  -background none -alpha set \
  -tile 4x3 -geometry +0+0 \
  spritesheets/walk_front.png
```

> -background none évite le retour d’un fond blanc lors de l’assemblage.


## 8) TexturePacker Free : sélectionner & exporter proprement

### 8.1 Pourquoi TexturePacker
	•	Visualiser l’animation (preview)
	•	Supprimer les frames “moches”
	•	Vérifier la boucle
	•	Export final stable

### 8.2 Workflow recommandé
	1.	Importer les images (frames_clean ou frames_resized)
	2.	Preview animation
	3.	Garder seulement les frames utiles (ex : 8–12)
	4.	Exporter en spritesheet (grid / ou pack) selon le besoin du moteur

> Pour Golden Quest / Pygame : privilégier une grille régulière.


## 9) Recettes prêtes à copier-coller

### 9.1 Vidéo → frames → resize → spritesheet (sans remove bg)

```bash
mkdir -p frames frames_resized frames_selected spritesheets

ffmpeg -i videos/idle_right.mp4 -vf "fps=12" frames/idle_right_%04d.png
mogrify -path frames_resized -resize x240 frames/idle_right_*.png

# (sélection manuelle) copier quelques frames dans frames_selected/
magick montage frames_selected/*.png -background none -alpha set -tile 4x3 -geometry +0+0 spritesheets/idle_right.png
```

### 9.2 Vidéo → frames → resize → remove bg → spritesheet

```bash
mkdir -p frames frames_resized frames_clean frames_selected spritesheets

ffmpeg -i videos/walk_front.mp4 -vf "fps=12" frames/walk_front_%04d.png
mogrify -path frames_resized -resize x240 frames/walk_front_*.png
mogrify -path frames_clean -fuzz 10% -transparent white frames_resized/walk_front_*.png

# (sélection) frames_clean → frames_selected
magick montage frames_selected/*.png -background none -alpha set -tile 4x3 -geometry +0+0 spritesheets/walk_front.png
```

### 9.3 Spritesheet → explode → flip → reassemble

Exemple (frames 118×210, grille 4×3)

```bash
mkdir -p tmp_left tmp_right spritesheets

magick input_left.png -crop 118x210 +repage +adjoin tmp_left/frame_%02d.png
mogrify -path tmp_right -flop tmp_left/*.png

magick montage tmp_right/*.png -background none -alpha set -tile 4x3 -geometry +0+0 spritesheets/output_right.png
```

## 10) Problèmes fréquents & solutions

### A) Le perso grossit / avance vers la caméra

→ Prompt à renforcer :
	•	“in place”
	•	“no camera movement”
	•	“constant size”
	•	“do not move toward the camera”

### B) Fond blanc qui revient lors de l’assemblage

→ Ajouter :
	•	-background none -alpha set dans montage

### C) Halo blanc autour du personnage après remove bg
	•	baisser ou ajuster -fuzz (ex : 6–12%)
	•	essayer -morphology Erode Disk:1 (avec prudence)

### D) Les frames ne sont pas toutes identiques en dimensions

→ Normaliser :

```bash
mogrify -path frames_resized -resize x240 -extent 240x240 -gravity center frames/*.png
```
(à utiliser si tu dois forcer un canevas commun)


## 11) Naming recommandé (pour ne pas se perdre)

	•	idle_front.png, idle_back.png, idle_left.png, idle_right.png
	•	walk_front.png, walk_back.png, walk_left.png, walk_right.png
	•	throw_left.png, throw_right.png
	•	etc.

## Annexes (à venir)
	•	Tutoriel vidéo (pas à pas)
	•	Check-list “avant intégration dans le jeu”
	•	Templates de prompts par animation (idle / walk / talk / throw / hit / die)
