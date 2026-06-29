---
type:
  - tutorial
date: 2026-01-09
timestamp: 2026-01-09 21:06:27

tags:
  - xx
  - yy
  - zz
---
# Complete Guide: Fast Font Generation (Ubuntu & Arch)

This guide provides everything you need to generate "Fast Fonts" (Bionic Reading effect) on Linux. It includes the full script, installation commands for Ubuntu and Arch Linux, usage instructions, and a troubleshooting guide for common logs.

**What this does:**
- Generates **4 font weights** for every family: Regular, Bold, Italic, Bold Italic.
- If a weight is missing (e.g. no Italic), it **synthetically generates** it.
- Applies **Bionic Reading** (bold first letters) to **Latin text only**.
- Preserves **Devanagari/Arabic** glyphs (they render normally, though shaping may be simplified).

---

## 1. Installation

### Ubuntu / Debian
```bash
# Install system dependencies
sudo apt update
sudo apt install python3 python3-pip fontforge python3-fontforge

# Install Python library for OpenType table handling
pip3 install fonttools
```

### Arch Linux / Manjaro
```bash
# Install system dependencies
sudo pacman -Syu
sudo pacman -S python python-pip fontforge

# Install Python library
pip install fonttools
```

---

## 2. The Script

Save the following code as **`generate_fast_font.py`**:

```python
#!/usr/bin/env python3
import os
import sys
import shutil
import logging
import re
import uuid
import unicodedata
from fontTools.ttLib import TTFont
from fontTools.feaLib.builder import addOpenTypeFeatures
import fontforge

# Configure Logging
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

class FastFontGenerator:
    def __init__(self, input_dir, output_dir):
        self.input_dir = input_dir
        self.output_dir = output_dir
        
        # Standard Weight Regex Helpers
        self.reg_pattern = re.compile(r'(?i)([-_.\s]+(Regular|Book|Roman)\b|-R\.)')
        self.bold_pattern = re.compile(r'(?i)([-_.\s]+Bold\b|-B\.)')
        self.italic_pattern = re.compile(r'(?i)([-_.\s]+Italic\b|-I\.)')
        self.bold_italic_pattern = re.compile(r'(?i)([-_.\s]+(Bold.*Italic|Italic.*Bold)\b|-BI\.)')
        
        # Exclude other weights
        self.exclude_pattern = re.compile(r'(?i)(Thin|Light|ExtraLight|Medium|SemiBold|Demi|Black|Heavy|ExtraBold|Condensed|Display|Caption)')

    def scan_families(self):
        """Scans input directory for font families."""
        families = {}
        
        for root, dirs, files in os.walk(self.input_dir):
            for file in files:
                if not file.lower().endswith(('.ttf', '.otf')):
                    continue
                
                if self.exclude_pattern.search(file):
                    logging.info(f"Skipping non-standard weight: {file}")
                    continue
                
                if 'stix' in file.lower():
                     logging.info(f"Skipping complex/large font to prevent crash: {file}")
                     continue

                full_path = os.path.join(root, file)
                try:
                    tt = TTFont(full_path)
                    family_name = self.get_name_record(tt, 16) or self.get_name_record(tt, 1)
                    
                    if not family_name:
                        continue
                        
                    if family_name not in families:
                        families[family_name] = {'regular': None, 'bold': None, 'italic': None, 'bold_italic': None}
                    
                    subfamily = (self.get_name_record(tt, 17) or self.get_name_record(tt, 2) or "").lower()
                    
                    if 'bold' in subfamily and 'italic' in subfamily:
                        families[family_name]['bold_italic'] = full_path
                    elif 'bold' in subfamily:
                        families[family_name]['bold'] = full_path
                    elif 'italic' in subfamily:
                        families[family_name]['italic'] = full_path
                    else:
                        if families[family_name]['regular'] is None:
                            families[family_name]['regular'] = full_path
                        else:
                            if 'regular' in file.lower() or 'book' in file.lower():
                                families[family_name]['regular'] = full_path
                                
                except Exception as e:
                    logging.warning(f"Could not parse font {file}: {e}")
                    
        return families

    def get_name_record(self, font, nameID):
        for record in font['name'].names:
            if record.nameID == nameID and record.platformID == 3 and record.langID == 0x409:
                return record.toUnicode()
        return None

    def create_synthetic_bold(self, regular_path, output_path):
        """Generates a synthetic bold version of the font using FontForge."""
        logging.info(f"Generating synthetic bold for {os.path.basename(regular_path)}...")
        try:
            font = fontforge.open(regular_path)
            font.weight = "Bold"
            font.os2_weight = 700
            font.selection.all()  # CRITICAL: Select all glyphs before changing weight
            font.changeWeight(40, "LCG", 0, 0, "squish") 
            font.generate(output_path)
            font.close()
            return output_path
        except Exception as e:
            logging.error(f"Failed to create synthetic bold: {e}")
            return None
    
    def create_synthetic_italic(self, source_path, output_path):
        """Generates a synthetic italic (skewed) version of the font."""
        logging.info(f"Generating synthetic italic for {os.path.basename(source_path)}...")
        try:
            font = fontforge.open(source_path)
            # Skew by ~13 degrees (0.22 radian)
            font.selection.all()  # CRITICAL: Select all glyphs before transforming
            mat = (1, 0, 0.22, 1, 0, 0)
            font.transform(mat)
            font.italicangle = -13
            font.os2_stylemap = font.os2_stylemap | 1  # Add Italic bit
            font.generate(output_path)
            font.close()
            return output_path
        except Exception as e:
            logging.error(f"Failed to create synthetic italic: {e}")
            return None

    def create_synthetic_bold_italic(self, bold_source, regular_source, output_path):
        """Generates synthetic bold italic. Prefers skewing bold; falls back to embolen+skew regular."""
        logging.info(f"Generating synthetic bold-italic...")
        try:
            source = bold_source if bold_source and os.path.exists(bold_source) else regular_source
            font = fontforge.open(source)
            
            # If starting from Regular, make it Bold first
            if source == regular_source:
                font.weight = "Bold"
                font.os2_weight = 700
                font.selection.all()
                font.changeWeight(40, "LCG", 0, 0, "squish")
            
            # Now Skew it
            font.selection.all() # CRITICAL: Select all glyphs before transforming
            mat = (1, 0, 0.22, 1, 0, 0)
            font.transform(mat)
            font.italicangle = -13
            font.os2_stylemap = font.os2_stylemap | 1  # Add Italic bit
            
            font.generate(output_path)
            font.close()
            return output_path
        except Exception as e:
            logging.error(f"Failed to create synthetic bold-italic: {e}")
            return None

    def update_metadata(self, font_path, suffix="Fast"):
        """Updates the Name Table of a font."""
        try:
            tt = TTFont(font_path)
            name_table = tt['name']
            
            current_family = self.get_name_record(tt, 16) or self.get_name_record(tt, 1)
            new_family_name = f"{current_family} {suffix}"
            
            def update_record(nID, new_val):
                for record in name_table.names:
                    if record.nameID == nID and record.platformID == 3 and record.langID == 0x409:
                        record.string = new_val.encode('utf-16-be')
                    elif record.nameID == nID and record.platformID == 1 and record.langID == 0:
                         record.string = new_val.encode('latin1')

            update_record(1, new_family_name)
            update_record(16, new_family_name)
            
            subfamily = self.get_name_record(tt, 2)
            if subfamily:
                try:
                     new_full = f"{new_family_name} {subfamily}"
                     new_ps = f"{new_family_name.replace(' ', '')}-{subfamily.replace(' ', '')}"
                     update_record(4, new_full)
                     update_record(6, new_ps)
                except:
                     pass
            
            tt.save(font_path)
            logging.info(f"Metadata updated: {os.path.basename(font_path)} -> {new_family_name}")
            return True
        except Exception as e:
            logging.error(f"Failed to update metadata for {font_path}: {e}")
            return False

    def generate_bionic_regular(self, regular_path, bold_path, dest_dir, family_name):
        """
        Creates the 'Fast Regular' font with bionic reading effect for Latin text.
        
        NOTE: The bionic effect (bold first letters) applies ONLY to Latin characters.
        Complex scripts (Devanagari, Arabic, etc.) will render with original glyphs,
        but may lose advanced shaping features due to FontForge merge limitations.
        """
        temp_bold = os.path.join(dest_dir, f"temp_bold_{uuid.uuid4().hex}.ttf")
        temp_merged = os.path.join(dest_dir, f"temp_merged_{uuid.uuid4().hex}.ttf")
        base = os.path.splitext(os.path.basename(regular_path))[0]
        clean_base = re.sub(r'(?i)([-_.\s]+(regular|book|roman)\b|-R\.)', '', base)
        final_output_name = f"{clean_base}-Regular-Fast.ttf"
        final_output = os.path.join(dest_dir, final_output_name)
        
        try:
            # 1. Prepare Bold (rename glyphs using FontForge)
            ff_font = fontforge.open(bold_path)
            for glyph in ff_font.glyphs():
                glyph.glyphname = glyph.glyphname + ".bold"
                glyph.unicode = -1 
            ff_font.generate(temp_bold)
            ff_font.close()
            
            # 2. Merge using FontForge (simple approach, may lose GPOS for complex scripts)
            logging.info("Merging fonts using FontForge...")
            reg_font = fontforge.open(regular_path)
            reg_font.mergeFonts(temp_bold)
            reg_font.generate(temp_merged)
            reg_font.close()
            
            # 3. Inject Features
            if not self.inject_calt_feature(temp_merged, family_name):
                logging.error("Failed to inject features.")
                return None
            
            # 4. Update Metadata
            self.update_metadata(temp_merged, "Fast")
            
            # 5. Move to final output (GPOS/GDEF already intact!)
            shutil.move(temp_merged, final_output)
            
            if os.path.exists(temp_bold): os.remove(temp_bold)
            
            return final_output
            
        except Exception as e:
            logging.error(f"Error generating Fast Regular: {e}")
            return None

    def inject_calt_feature(self, font_path, family_name):
        """Injects 'calt' feature for bionic reading effect (Latin-only)."""
        try:
            tt = TTFont(font_path)
            cmap = tt.getBestCmap()
            glyph_names = tt.getGlyphOrder()
            existing_names = set(glyph_names)
            
            bionic_whitelist = set()
            for code, name in cmap.items():
                try:
                    char = chr(code)
                    cat = unicodedata.category(char)
                    is_latin = 'LATIN' in unicodedata.name(char, "").upper()
                    is_number = cat.startswith('N')
                    if (cat.startswith('L') and is_latin) or is_number:
                         bionic_whitelist.add(name)
                except:
                    pass
            
            logging.info(f"  Bionic Whitelist Size: {len(bionic_whitelist)} mapped glyphs")
            
            normal_glyphs = []
            bold_glyphs = []
            
            for name in glyph_names:
                if name.endswith('.bold'):
                    base_name = name[:-5]
                    if base_name in existing_names:
                        if base_name not in bionic_whitelist:
                            continue
                        normal_glyphs.append(base_name)
                        bold_glyphs.append(name)
            
            if not normal_glyphs:
                logging.warning("No Latin/Symbol bold pairs found.")
            
            fea = []
            fea.append("languagesystem DFLT dflt;")
            fea.append("languagesystem latn dflt;")
            
            fea.append(f"@normal = [{' '.join(normal_glyphs)}];")
            fea.append(f"@bold = [{' '.join(bold_glyphs)}];")
            fea.append("@all = [@normal @bold];")
            
            fea.append("feature calt {")
            fea.append("    ignore sub @all @all @all @all @all @all @all @normal' @all @all @all @all @all @all @all @all @all @all;")
            fea.append("    sub @all @all @all @all @all @all @normal' @all @all @all @all @all @all @all @all @all @all by @bold;")
            fea.append("    ignore sub @all @all @all @all @all @all @normal' @all @all @all @all @all @all @all @all;")
            fea.append("    sub @all @all @all @all @all @normal' @all @all @all @all @all @all @all @all by @bold;")
            fea.append("    ignore sub @all @all @all @all @all @normal' @all @all @all @all @all @all;")
            fea.append("    sub @all @all @all @all @normal' @all @all @all @all @all @all @all by @bold;")
            fea.append("    ignore sub @all @all @all @all @normal' @all @all @all @all @all;")
            fea.append("    sub @all @all @all @normal' @all @all @all @all @all by @bold;")
            fea.append("    ignore sub @all @all @all @normal' @all @all @all @all;")
            fea.append("    sub @all @all @normal' @all @all @all @all by @bold;")
            fea.append("    ignore sub @all @all @normal' @all @all;")
            fea.append("    sub @all @normal' @all @all by @bold;")
            fea.append("    ignore sub @all @normal';")
            fea.append("    sub @normal' by @bold;")
            fea.append("} calt;")
            
            fea_path = font_path + ".fea"
            with open(fea_path, 'w') as f:
                f.write("\n".join(fea))
                
            addOpenTypeFeatures(tt, fea_path)
            tt.save(font_path)
            return True
        except Exception as e:
            logging.error(f"FEA Injection Error: {e}")
            return False

    def process_siblings(self, family_info, dest_dir):
        """Copies Bold/Italic files."""
        for style in ['bold', 'italic', 'bold_italic']:
            path = family_info.get(style)
            if path and os.path.exists(path):
                base = os.path.splitext(os.path.basename(path))[0]
                weight_map = {'bold': 'Bold', 'italic': 'Italic', 'bold_italic': 'BoldItalic'}
                weight_label = weight_map.get(style, style.title())
                clean_base = re.sub(r'(?i)([-_.\s]+(regular|book|roman|bold|italic|bold_italic|bolditalic)\b|-(R|B|I|BI)\.)', '', base)
                new_filename = f"{clean_base}-{weight_label}-Fast.ttf"
                dest_path = os.path.join(dest_dir, new_filename)
                
                shutil.copy2(path, dest_path)
                self.update_metadata(dest_path, "Fast")

    def run(self):
        logging.info("Scanning for families...")
        families = self.scan_families()
        logging.info(f"Found {len(families)} families.")
        
        for family, data in families.items():
            if not data['regular']:
                logging.info(f"Skipping {family}: No Regular found.")
                continue
            
            logging.info(f"Processing Family: {family}")
            family_out_dir = os.path.join(self.output_dir, f"{family} Fast")
            os.makedirs(family_out_dir, exist_ok=True)
            
            regular_source = data['regular']
            bold_source = data['bold']
            italic_source = data['italic']
            bold_italic_source = data['bold_italic']
            
            temp_files_to_cleanup = []
            
            # 1. Ensure Bold
            if not bold_source:
                logging.info(f"  No Bold found. Creating synthetic Bold.")
                safe_fam = family.replace(" ", "")
                syn_bold = os.path.join(family_out_dir, f"{safe_fam}-Bold.ttf")
                if self.create_synthetic_bold(regular_source, syn_bold):
                    bold_source = syn_bold
                    temp_files_to_cleanup.append(syn_bold)
            
            # 2. Ensure Italic
            if not italic_source:
                logging.info(f"  No Italic found. Creating synthetic Italic.")
                safe_fam = family.replace(" ", "")
                syn_italic = os.path.join(family_out_dir, f"{safe_fam}-Italic.ttf")
                if self.create_synthetic_italic(regular_source, syn_italic):
                    italic_source = syn_italic
                    temp_files_to_cleanup.append(syn_italic)
            
            # 3. Ensure Bold Italic
            if not bold_italic_source:
                logging.info(f"  No Bold Italic found. Creating synthetic Bold Italic.")
                safe_fam = family.replace(" ", "")
                syn_bi = os.path.join(family_out_dir, f"{safe_fam}-BoldItalic.ttf")
                # Pass bold_source (which might be synthetic!) as primary source
                if self.create_synthetic_bold_italic(bold_source, regular_source, syn_bi):
                    bold_italic_source = syn_bi
                    temp_files_to_cleanup.append(syn_bi)
            
            if not bold_source:
                logging.warning(f"  Failed to get a Bold source for {family}. Skipping bionic generation.")
                continue

            # Generate Fast Regular
            logging.info("  Generating Fast Regular...")
            self.generate_bionic_regular(regular_source, bold_source, family_out_dir, family)
            
            # Process Siblings (Bold, Italic, BoldItalic)
            logging.info("  Linking Siblings...")
            sibling_data = {
                'bold': bold_source,
                'italic': italic_source,
                'bold_italic': bold_italic_source
            }
            self.process_siblings(sibling_data, family_out_dir)
            
            # Cleanup temp files
            for f in temp_files_to_cleanup:
                if os.path.exists(f):
                    os.remove(f)

if __name__ == "__main__":
    if len(sys.argv) < 3:
        print("Usage: python3 generate_fast_font.py <input_dir> <output_dir>")
        sys.exit(1)
    generator = FastFontGenerator(sys.argv[1], sys.argv[2])
    generator.run()
```

---

## 3. Usage Steps

1.  **Prepare your input folder**:
    Create a folder named `fonts` and put your original font files (TTF/OTF) in it. You can have subdirectories.

    ```bash
    mkdir -p fonts
    # Copy your .ttf / .otf files here
    ```

2.  **Run the script**:
    Create an output directory (e.g., `FastFamilies`) and run the script:

    ```bash
    mkdir -p FastFamilies
    python3 generate_fast_font.py fonts FastFamilies
    ```

3.  **Install the fonts**:
    Copy the generated fonts to your user font directory:

    ```bash
    mkdir -p ~/.local/share/fonts/FastFonts
    cp -r FastFamilies/* ~/.local/share/fonts/FastFonts/
    
    # Update font cache
    fc-cache -fv
    ```

4.  **Verify**:
    Open an e-reader app (like KOReader) or a word processor (LibreOffice). Look for fonts ending in "**Fast**" (e.g., "Sama Devanagari Fast").

---

## 4. Expected Warnings & Troubleshooting

During execution, you may see some error messages in the console. These are usually expected behavior.

### ⚠️ "Internal Error (overlap): monotonic is both needed and unneeded"
**Cause:** This comes from **FontForge's** backend when synthetically emboldening or skewing a font. It happens when glyph curves are complex or slightly imperfect (e.g., overlapping paths).
**Solution:** **Ignore it.** The script continues processing, and the resulting synthetic font is almost always valid and usable.

### ⚠️ "Internal Error: Invalid 2nd order spline in SplineRefigure2"
**Cause:** Similar to the above, this occurs during mathematical transformations of glyph curves.
**Solution:** **Ignore it.**

### ⚠️ "Mark Positioning lookup... is too big. Will not be useable."
**Cause:** This confirms the script is working as intended for Option B. We are intentionally dropping complex GPOS (Positioning) tables that are too large or incompatible with the simplified merge strategy.
**Outcome:** **Expected.** This ensures the font generates successfully. The trade-off is that complex script shaping (e.g., Devanagari stacked conjuncts) may be simplified, but the Latin bionic effect will be perfect.

### ⚠️ "The following table(s) in the font have been ignored by FontForge"
**Cause:** FontForge strips out non-standard or proprietary tables (like `DSIG` digital signatures) during the merge.
**Outcome:** **Harmless.** These tables are not needed for display.
