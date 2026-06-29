---
type:
  - tutorial
date: 2025-12-30
timestamp: 2025-12-30 19:30:55
updated_at: 2025-12-31T10:57:13.841+05:30
edited_seconds: 210

tags:
  - zz
---
# Tutorial: Build a full font family (Regular/Bold/Italic/Bold Italic) on Linux using FontForge + FontTools

This guide shows how to start from a single `*-Regular.ttf`, generate a synthetic **Bold**, optionally “borrow” Italic/Bold Italic from another font by _rebranding metadata_, and then verify/fix everything so apps (and e-readers) treat it as one clean family.[fontforge+1](https://fontforge.org/docs/scripting/scripting.html)​

## Prerequisites

- Tools:
    
    - **FontForge** (CLI + Python scripting) for generating/editing fonts.[fontforge+1](https://fontforge.org/docs/scripting/python/fontforge.html)​
        
    - **fontconfig** tools (`fc-scan`, `fc-cache`) for Linux-side inspection/caching.[man.archlinux](https://man.archlinux.org/man/fc-scan.1.en)​
        
    - **fonttools** (Python package; includes `TTFont` and `ttx`) for low-level table verification.[pypi+2](https://pypi.org/project/fonttools/)​
        
- Directory layout (example):

```bash
mkdir -p ~/Z/Goudy_Bookletter_1911 cd ~/Z/Goudy_Bookletter_1911 ls # GoudyBookletter1911-Regular.ttf
```
  

## Create Bold (from Regular)

Run one command to embolden outlines and set the metadata so it links as **Bold** inside the same family.[fontforge+1](https://fontforge.org/docs/scripting/scripting.html)​

bash

`
```bash
fontforge -lang=py -c "import fontforge; f=fontforge.open('GoudyBookletter1911-Regular.ttf'); f.selection.all(); f.changeWeight(25, 'LCG'); f.fontname='GoudyBookletter1911-Bold'; f.familyname='Goudy Bookletter 1911'; f.fullname='Goudy Bookletter 1911 Bold'; f.weight='Bold'; f.os2_weight=700; f.appendSFNTName(0x409, 2, 'Bold'); f.appendSFNTName(0x409, 17, 'Bold'); f.generate('GoudyBookletter1911-Bold.ttf')"
```

Notes:

- `changeWeight(25, "LCG")` is a synthetic embolden; adjust `25` up/down to taste.[fontforge](https://fontforge.org/docs/scripting/python/fontforge.html)​
    
- The `name` table entries (like Subfamily) are part of OpenType’s naming system and affect how menus group/style fonts.[learn.microsoft](https://learn.microsoft.com/en-us/typography/opentype/spec/name)​
    

## Add Italic + Bold Italic (metadata rebrand)

If your original family has no true Italic, you can take an existing Italic and Bold Italic font and **rebrand** them to appear as the Italic members of your target family (this changes naming/flags, not the outlines).[learn.microsoft+1](https://learn.microsoft.com/en-us/typography/opentype/spec/name)​

## Rebrand an Italic source → `GoudyBookletter1911-Italic.ttf`

```bash
fontforge -lang=py -c "import fontforge; f=fontforge.open('KingsCaslon-Italic.ttf'); f.fontname='GoudyBookletter1911-Italic'; f.familyname='Goudy Bookletter 1911'; f.fullname='Goudy Bookletter 1911 Italic'; f.weight='Italic'; f.os2_stylemap=0x01; f.macstyle=2; f.appendSFNTName(0x409, 2, 'Italic'); f.appendSFNTName(0x409, 17, 'Italic'); f.generate('GoudyBookletter1911-Italic.ttf')"
```

## Rebrand a Bold Italic source → `GoudyBookletter1911-BoldItalic.ttf`


```bash
fontforge -lang=py -c "import fontforge; f=fontforge.open('KingsCaslon-BoldItalic.ttf'); f.fontname='GoudyBookletter1911-BoldItalic'; f.familyname='Goudy Bookletter 1911'; f.fullname='Goudy Bookletter 1911 Bold Italic'; f.weight='Bold Italic'; f.os2_weight=700; f.os2_stylemap=0x21; f.macstyle=3; f.appendSFNTName(0x409, 2, 'Bold Italic'); f.appendSFNTName(0x409, 17, 'Bold Italic'); f.generate('GoudyBookletter1911-BoldItalic.ttf')"
```

Why these fields matter:

- OpenType “name” records include both “Family/Subfamily” and “Typographic Family/Subfamily” concepts; some software prefers typographic names if present.[learn.microsoft](https://learn.microsoft.com/en-us/typography/opentype/spec/name)​
    

## Verify + fix deeper naming issues (FontTools)

## Quick Linux-side check (`fc-scan`)

This tells you what fontconfig thinks the family/style/weight are.[man.archlinux](https://man.archlinux.org/man/fc-scan.1.en)​

```bash
for f in GoudyBookletter1911-*.ttf; do   fc-scan "$f" --format 'File: %{file}\n  Family: %{family}\n  Style: %{style}\n  Weight: %{weight}\n\n' done
```

## Binary-level truth check (FontTools, no temp files)

This checks OpenType Name ID 1/2 and OS/2 `usWeightClass` directly from the font tables.[fonttools.readthedocs+2](https://fonttools.readthedocs.io/en/latest/ttLib/ttFont.html)​

bash

`python3 - <<'EOF' import glob from fontTools.ttLib import TTFont print(f"{'FILENAME':<35} {'FAMILY (ID 1)':<25} {'SUBFAMILY (ID 2)':<20} {'US WEIGHT'}") print("-" * 95) for path in sorted(glob.glob("GoudyBookletter1911-*.ttf")):     tt = TTFont(path)    name = tt["name"]     def get_name(name_id):        # Prefer Windows platform + English (US)        for r in name.names:            if r.nameID == name_id and r.platformID == 3 and r.langID == 0x409:                return r.toUnicode()        for r in name.names:            if r.nameID == name_id:                return r.toUnicode()        return "N/A"     family = get_name(1)      # Family    subfam = get_name(2)      # Subfamily (Regular/Bold/Italic/...)    weight = tt["OS/2"].usWeightClass     print(f"{path:<35} {family:<25} {subfam:<20} {weight}") EOF`

Expected output pattern:

- Same **Family (ID 1)** for all four
    
- Subfamily (ID 2): `Regular`, `Bold`, `Italic`, `Bold Italic`
    
- `US WEIGHT`: 400 for Regular/Italic, 700 for Bold/Bold Italic
    

## If an e-reader still shows “Kings Caslon”: remove Name IDs 16/17

Many readers/apps will display **Typographic Family/Subfamily** (Name IDs 16/17) if present, instead of Name IDs 1/2.[learn.microsoft](https://learn.microsoft.com/en-us/typography/opentype/spec/name)​  
So if the rebranded fonts still carry old 16/17 values, strip them:

bash

`python3 - <<'EOF' import glob from fontTools.ttLib import TTFont for path in glob.glob("GoudyBookletter1911-*.ttf"):     tt = TTFont(path)    name = tt["name"]    before = len(name.names)    name.names = [r for r in name.names if r.nameID not in (16, 17)]    after = len(name.names)    if after != before:        tt.save(path)        print("Removed ID 16/17 from", path) EOF`

## Install (Linux) + refresh cache

bash

`mkdir -p ~/.local/share/fonts cp GoudyBookletter1911-*.ttf ~/.local/share/fonts/ fc-cache -fv`
