---
type:
  - notes
  - tutorial
date: 2025-12-25
timestamp: 2025-12-25 17:19:53
updated_at: 2025-12-26T01:57:38.217+05:30
edited_seconds: 380

tags:
  - zz
---
## Prerequisites

Get the required firmware which is comparable to your device like Kindle Basic is comparable to Kobo Clara BW in terms of screen size:
```
https://www.amazon.com/gp/help/customer/display.html?nodeId=GKMQC26VQQMM8XSW
```

## Prerequisites

1. Get the required firmware:
```
https://www.amazon.com/gp/help/customer/display.html?nodeId=GKMQC26VQQMM8XSW
```

2. Before starting, install the required tools:

```bash
# Install KindleTool from AUR (for firmware extraction)
yay -S kindletool

# Install p7zip and squashfs-tools (for extraction)
sudo pacman -S p7zip squashfs-tools
```

## Overview

This tutorial extracts fonts from Kindle firmware using a **non-mounting approach** that:

- ✅ Requires **no root privileges** for filesystem access
- ✅ Uses standard archive tools (7zip)
- ✅ Avoids mounting/unmounting complexity
- ✅ Works on any Linux system with the required tools

## Step-by-Step Guide

### Step 1: Extract the Firmware Package

Navigate to your firmware directory and extract the `.bin` file:

```bash
# Navigate to your firmware directory
# cd /path/to/your/firmware/

# Extract the Kindle firmware .bin file
kindletool extract update_kindle_11th_2024_5.18.6.bin extracted_firmware

# Navigate to the extracted firmware
cd extracted_firmware/
```

**Expected output:** KindleTool will extract multiple files including `rootfs.img.gz`, `data.stgz`, `metadata.txt`, and device-specific files.

### Step 2: Decompress the Root Filesystem

The root filesystem is compressed with gzip and must be decompressed first:

```bash
# Decompress the rootfs.img.gz to get rootfs.img
gunzip rootfs.img.gz

# Verify the decompressed file exists
ls -la rootfs.img
```

### Step 3: Extract the Root Filesystem

Use 7zip to extract the ext4 filesystem without mounting:

```bash
# Create a directory for the extracted rootfs
mkdir rootfs_extracted

# Extract the rootfs.img using 7zip (no root required!)
7z x rootfs.img -orootfs_extracted/

# Verify the extraction worked
ls -la rootfs_extracted/
```
**NOTE**: the command `-orootfs_extracted/` is right dont seperate `-o`
### Step 4: Locate the Font Archive

The fonts are stored in a compressed SquashFS filesystem:

```bash
# The font archive is located at:
# rootfs_extracted/usr/java/lib/fonts.sqsh

# Verify the file exists and check its details
file rootfs_extracted/usr/java/lib/fonts.sqsh
```

**Expected output:** Should show a SquashFS filesystem (~83MB) containing the fonts.
### Step 5: Extract the Font Files

Extract the SquashFS archive containing all Kindle fonts:

```bash
# Create a directory for the extracted fonts
mkdir kindle_fonts_extracted

# Extract the SquashFS filesystem containing all fonts
unsquashfs -d kindle_fonts_extracted rootfs_extracted/usr/java/lib/fonts.sqsh
```

### Step 6: Explore and Organize

Browse and organize your extracted fonts:

```bash
# List the contents of the extracted fonts
ls -la kindle_fonts_extracted/

# Find all font files by extension
find kindle_fonts_extracted/ -type f \( -iname "*.ttf" -o -iname "*.otf" -o -iname "*.woff" -o -iname "*.woff2" \)

# Get detailed information about font files
find kindle_fonts_extracted/ -type f -exec file {} \; | grep -i -E "(font|truetype|opentype)"

# Optional: Copy fonts to a clean directory
mkdir ~/kindle_fonts
find kindle_fonts_extracted/ -type f \( -iname "*.ttf" -o -iname "*.otf" -o -iname "*.woff" -o -iname "*.woff2" \) -exec cp {} ~/kindle_fonts/ \;
```

### Step 7: View Your Fonts

```bash
# List your final extracted fonts
ls -la ~/kindle_fonts/

# Get font information (optional, requires fontconfig)
find ~/kindle_fonts/ -name "*.ttf" -exec fc-query {} \; | head -20
```

## File Structure

After completing the extraction, your directory structure will look like:

```
extracted_firmware/
├── rootfs.img                    # Decompressed filesystem image
├── rootfs_extracted/             # Extracted filesystem contents
│   ├── usr/
│   │   └── java/
│   │       └── lib/
│   │           └── fonts.sqsh    # Font archive
│   ├── opt/
│   ├── etc/
│   └── ...                       # Other system directories
├── kindle_fonts_extracted/       # All extracted fonts
│   ├── font1.ttf
│   ├── font2.otf
│   └── ...
├── data.stgz                     # Other firmware components
├── metadata.txt
└── mt8110_bellatrix4/            # Device-specific files
```


## What You'll Find

The extracted fonts typically include:

- **System fonts** - Interface elements and menus
- **Reading fonts** - Serif, sans-serif, and other reading typefaces
- **Language-specific fonts** - Support for international characters
- **Symbol fonts** - Special characters, icons, and symbols

Common font families you might find:

- Amazon Ember (Amazon's custom font)
- Bookerly (Amazon's reading font)
- Various serif and sans-serif options
- CJK (Chinese, Japanese, Korean) font support
- Arabic and other script support


## Troubleshooting

### gunzip Command Fails

```bash
# Check if the .gz file exists and is valid
file rootfs.img.gz
ls -la rootfs.img.gz

# If file is corrupted, re-extract firmware
cd .. && rm -rf extracted_firmware
kindletool extract update_kindle_all_new_paperwhite_12th_5.18.4.0.1.bin extracted_firmware
```


### 7zip Extraction Fails

```bash
# Check if the rootfs.img is valid after decompression
file rootfs.img

# Try listing contents first to verify integrity
7z l rootfs.img

# If that works, try extracting to current directory
7z x rootfs.img
```


### SquashFS Extraction Fails

```bash
# Ensure squashfs-tools is installed
sudo pacman -S squashfs-tools

# Check the SquashFS file integrity
file rootfs_extracted/usr/java/lib/fonts.sqsh

# Try with verbose output to see what's happening
unsquashfs -v -d kindle_fonts_extracted rootfs_extracted/usr/java/lib/fonts.sqsh
```


### No Font Files Found

```bash
# Search more broadly for font-related files
find kindle_fonts_extracted/ -type f -name "*" -exec file {} \; | grep -i font

# Look for files by size (fonts are typically larger)
find kindle_fonts_extracted/ -type f -size +10k

# Check for alternative font formats
find kindle_fonts_extracted/ -type f \( -iname "*.bdf" -o -iname "*.pcf" -o -iname "*.pfb" \)
```


## 🏆 The Definitive Kindle Font Master Tutorial

#### 🛠️ Phase 1: Preparation

```bash
## 1. Enter your work directory
cd "kindle_fonts_extracted" || exit 1

## 2. Safety backup
mkdir -p ../master_backup
cp -a . ../master_backup/

## 3. Ensure fonttools is ready
# python3 -m pip install --user fonttools

```

---

#### 📂 Phase 2: Structural Organization & Orphan Sweep

This script is aggressive. It moves system fonts first, then processes reading fonts, and finally **forces** any leftover `.ttf` files into their correct folders.

```zsh
cat <<'MASTER_SCRIPT' | zsh
set -euo pipefail
setopt null_glob

# ==============================================================================
# MASTER KINDLE FONT ORGANIZER FOR KOBO/NICKELMENU
# ==============================================================================

echo ">>> [0/5] Checking dependencies..."
if ! command -v python &> /dev/null; then
    echo "Error: Python is required."
    exit 1
fi
if ! python -c "import fontTools" 2>/dev/null; then
    echo "Installing 'fonttools' library..."
    pip install fonttools
fi

# --- STEP 1: RESET ENVIRONMENT ---
echo ">>> [1/5] Resetting directory..."
# Move all fonts from subfolders back to root
mv */*.(ttf|otf) . 2>/dev/null || true
mv */*/*.{ttf,otf} . 2>/dev/null || true # In case of deep nesting
# Remove old directories
rm -rf K\ * System_* Code2000 Kindle "Noto Fonts" _trash 2>/dev/null || true

# --- STEP 2: PROTECT SYSTEM FONTS ---
echo ">>> [2/5] Isolating System Fonts..."
mkdir -p "System_Kindle" "System_Noto" "System_Code2000"

# Move Kindle UI/Symbol fonts
mv Kindle*.(ttf|otf) "System_Kindle/" 2>/dev/null || true
mv *MTChinese*.(ttf|otf) "System_Kindle/" 2>/dev/null || true

# Move Noto (International) fonts
mv Noto*.(ttf|otf) "System_Noto/" 2>/dev/null || true

# Move Code2000
mv code2000*.(ttf|otf) "System_Code2000/" 2>/dev/null || true


# --- STEP 3: RENAME & ORGANIZE FILES ---
echo ">>> [3/5] Organizing Reading Fonts (File Renaming)..."

process_font_file() {
  local f="$1"
  local bn="${f:t}"
  local ext="${bn:e}"
  local base="${bn:r}"
  local fam="" 
  local style=""
  
  # --- A. FAMILY DETECTION LOGIC ---
  if [[ "$base" == AmazonEmberBold* || "$base" == "K Amazon Ember Bold"* ]]; then
    fam="Amazon Ember Bold"
    style="${base#*Bold}"; style="${style#*-}"
  elif [[ "$base" == AmazonEmber* || "$base" == Amazon-Ember* || "$base" == "K Amazon Ember"* ]]; then
    fam="Amazon Ember"
    style="${base#*Ember}"; style="${style#-}"
  elif [[ "$base" == Futura* ]]; then 
    fam="Futura"; style="${base#Futura}"
  elif [[ "$base" == Helvetica_LT* ]]; then 
    fam="Helvetica LT"; style="${base#Helvetica_LT}"
  elif [[ "$base" == Caecilia_LT* ]]; then
    if [[ "$base" == *"Cond"* ]]; then fam="PMN Caecilia LT Condensed"; else fam="PMN Caecilia LT"; fi
    style="${base#Caecilia_LT}"
  elif [[ "$base" == STHeiti* ]]; then 
    if [[ "$base" == *TC* ]]; then fam="STHeiti TC"; else fam="STHeiti"; fi
    style="${base#STHeiti}"
  elif [[ "$base" == STSong* ]]; then 
    if [[ "$base" == *TC* ]]; then fam="STSong TC"; else fam="STSong"; fi
    style="${base#STSong}"
  elif [[ "$base" == TBGothic* ]]; then 
    fam="TBGothic"; style="${base#TBGothic}"
  elif [[ "$base" == TBMincho* ]]; then 
    fam="TBMincho"; style="${base#TBMincho}"
  else
    # Fallback
    if [[ "$base" == *-* ]]; then fam="${base%-*}"; style="${base#*-}"
    else fam="$base"; style="Regular"; fi
  fi

  # --- B. STYLE CLEANING LOGIC ---
  # 1. Clean delimiters
  style="${style//_/ }"; style="${style//-/ }"; style="${style// /}"
  
  # 2. Fix RegularItalic / Oblique -> Italic
  style="${style/RegularItalic/Italic}"
  style="${style/Oblique/Italic}"
  style="${style/MediumItalic/Italic}"

  # 3. Remove Numbers
  style="${style//[0-9]/}"

  # 4. Remove redundant family tags from style
  if [[ "$fam" == *"Condensed"* ]]; then style="${style/Condensed/}"; style="${style/Cond/}"; fi
  if [[ "$fam" == *" TC"* ]]; then style="${style/TC/}"; fi

  # 5. Fix Medium/Med -> Regular (Crucial for Kobo)
  if [[ "$style" == "Medium" || "$style" == "Med" ]]; then style="Regular"; fi

  # 6. Safety Defaults
  if [[ -z "$style" ]]; then style="Regular"; fi
  style="${style/RegularRegular/Regular}"

  # --- C. MOVE FILE ---
  local out="K ${fam}-${style}.${ext}"
  local target_dir="K ${fam}"
  
  mkdir -p "$target_dir"
  mv -f "$f" "$target_dir/$out"
}

# Run loop on remaining fonts
for f in ./*.(ttf|otf); do
  # Avoid processing the system folders if glob caught them weirdly
  [[ -d "$f" ]] && continue
  process_font_file "$f"
done


# --- STEP 4: FIX INTERNAL METADATA (PYTHON) ---
echo ">>> [4/5] Updating Internal Metadata & Flags..."

cat <<'PYTHON_SCRIPT' > fix_metadata_final.py
import os
import glob
import sys
try:
    from fontTools.ttLib import TTFont
except ImportError:
    print("Error: fonttools not installed.")
    sys.exit(1)

def set_name(name_table, nameID, string_val):
    # Mac
    name_table.setName(string_val, nameID, platformID=1, platEncID=0, langID=0)
    # Windows
    name_table.setName(string_val, nameID, platformID=3, platEncID=1, langID=1033)

def update_flags(font, is_bold, is_italic):
    # 1. OS/2 Table (fsSelection)
    if 'OS/2' in font:
        os2 = font['OS/2']
        # Clear bits 0(Italic), 5(Bold), 6(Regular)
        os2.fsSelection &= ~((1<<0)|(1<<5)|(1<<6))
        
        if is_bold: os2.fsSelection |= (1 << 5)
        if is_italic: os2.fsSelection |= (1 << 0)
        if not is_bold and not is_italic: os2.fsSelection |= (1 << 6)
            
    # 2. Head Table (macStyle)
    if 'head' in font:
        head = font['head']
        # Clear bits 0(Bold), 1(Italic)
        head.macStyle &= ~((1<<0)|(1<<1))
        
        if is_bold: head.macStyle |= (1 << 0)
        if is_italic: head.macStyle |= (1 << 1)

def process():
    files = glob.glob("K */*.ttf") + glob.glob("K */*.otf")
    print(f"   Processing {len(files)} files...")

    for path in files:
        try:
            filename = os.path.basename(path)
            name_part = os.path.splitext(filename)[0]
            
            # Format: "K Family-Style"
            if "-" not in name_part: continue

            family_name = name_part.rsplit("-", 1)[0]
            style_name = name_part.rsplit("-", 1)[1]
            
            is_bold = "Bold" in style_name
            is_italic = "Italic" in style_name
            
            full_name = f"{family_name} {style_name}"
            if style_name == "Regular": full_name = family_name

            # PostScript Name (Strict ASCII, no spaces)
            ps_name = f"{family_name.replace(' ', '')}-{style_name}"

            font = TTFont(path)
            name_table = font['name']

            # Wipe old IDs
            name_table.names = [n for n in name_table.names if n.nameID not in [1, 2, 4, 6, 16, 17]]

            # Write new IDs
            set_name(name_table, 1, family_name)       # Family
            set_name(name_table, 2, style_name)        # Subfamily
            set_name(name_table, 4, full_name)         # Full Name
            set_name(name_table, 6, ps_name)           # PostScript
            set_name(name_table, 16, family_name)      # Typo Family
            set_name(name_table, 17, style_name)       # Typo Subfamily

            update_flags(font, is_bold, is_italic)

            font.save(path)
            
        except Exception as e:
            print(f"   Error on {filename}: {e}")

if __name__ == "__main__":
    process()
PYTHON_SCRIPT

python fix_metadata_final.py
rm fix_metadata_final.py

# --- STEP 5: CLEANUP ---
echo ">>> [5/5] Final Cleanup..."
find . -type d -empty -delete
echo "========================================================"
echo "✔ SUCCESS"
echo "  - System Fonts: stored in System_* folders"
echo "  - Reading Fonts: stored in 'K [Family]' folders"
echo "  - Metadata: Updated (Names + Bold/Italic Flags)"
echo "========================================================"
MASTER_SCRIPT

```

#### 🏁 Final Verification

The root directory should now be completely empty of files.

```bash
ls -F

```

**What you have now:**

1. **Zero Orphans:** Amazon Ember and the `.dir` files are gone or moved.
2. **Clean Family Menu:** Asian fonts (STSong, STHeiti) are merged into single folders.
3. **Perfect Rendering:** The metadata bits are "clean," meaning the Kindle will switch from Regular to Bold/Italic smoothly.
