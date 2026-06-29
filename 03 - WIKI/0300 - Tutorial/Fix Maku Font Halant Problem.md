---
type:
  - tutorial
date: 2025-12-29
timestamp: 2025-12-29 23:44:21
updated_at: 2025-12-29T23:45:36.541+05:30
edited_seconds: 30

tags:
  - zz
---

# Tutorial: Repairing Maku Fonts for Devanagari

This guide documents how we fixed the Devanagari rendering issues in the Maku font family.

## The Problem
The Maku fonts were failing to render Devanagari conjuncts correctly.
- **Symptom**: Words like `क्त` (Kta) or `स्त` (Sta) showed visible Halant/Virama marks (`क्` + `त`) instead of forming a single composite glyph.
- **Cause**: The font contained **legacy Apple Advanced Typography (AAT)** tables (`morx`, `feat`, etc.).
  - On Linux/HarfBuzz, if AAT tables are found, the text engine attempts to use them **instead** of standard OpenType (`GSUB`) tables.
  - The AAT tables in Maku were incomplete or broken for Devanagari, causing the engine to skip the critical "Half Form" substitutions defined in the perfectly valid `GSUB` table.

## The Solution
The fix was to **remove the interfering AAT tables**. This forces the text engine to fall back to the OpenType `GSUB` table, which was correctly implemented all along.

## The Repair Script
We used the Python `fontTools` library to surgically remove the AAT tables while preserving the rest of the font.

### Prerequisites
You need Python installed with the `fontTools` library:
```bash
pip install fonttools
```

### Script: `repair_maku.py`
Save the following code as `repair_maku.py`:

```python
#!/usr/bin/env python3
import sys
import os
from fontTools.ttLib import TTFont

def remove_aat_tables(font_path):
    """
    Removes standard AAT tables (morx, mort, etc.) from an OTF/TTF font
    to force engines to use OpenType (GSUB/GPOS) tables instead.
    """
    output_path = font_path.replace('.otf', '_fixed.otf').replace('.ttf', '_fixed.ttf')
    
    print(f"Processing: {font_path}")
    font = TTFont(font_path)
    
    # List of AAT-specific tables to remove
    aat_tables = [
        'morx', # Extended Glyph Metamorphosis (The check culprit)
        'mort', # Older Glyph Metamorphosis
        'kerx', # Extended Kerning
        'ankr', # Anchor points
        'feat', # Feature Name table
        'lcar', # Ligature Caret
        'prop', # Properties
        'trak', # Tracking
        'opbd'  # Optical Bounds
    ]
    
    removed_count = 0
    for tag in aat_tables:
        if tag in font:
            del font[tag]
            print(f"  - Removed table: {tag}")
            removed_count += 1
            
    if removed_count > 0:
        font.save(output_path)
        print(f"  ✓ Saved fixed font to: {output_path}")
    else:
        print("  i No AAT tables found. Font might already be clean.")

def main():
    if len(sys.argv) < 2:
        print("Usage: python3 repair_maku.py <font-file1> [font-file2 ...]")
        sys.exit(1)
        
    for font_file in sys.argv[1:]:
        if os.path.exists(font_file):
            remove_aat_tables(font_file)
        else:
            print(f"Error: File not found: {font_file}")

if __name__ == "__main__":
    main()
```

## How to Run
1. Place the script in the same folder as your fonts.
2. Run it on all Maku font files:
   ```bash
   python3 repair_maku.py Maku-Regular.otf Maku-Bold.otf Maku-Medium.otf Maku-ExtraBold.otf
   ```
3. The script will generate new files ending in `_fixed.otf` (e.g., `Maku-Regular_fixed.otf`).
4. These new files are the repaired versions ready for installation.
