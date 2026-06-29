---
type:
  - tutorial
date: 2025-12-26
timestamp: 2025-12-26 01:57:38
updated_at: 2025-12-27T22:39:23.668+05:30
edited_seconds: 220

tags:
  - zz
---
# Kobo Fonts Extraction and Setup

## Step 1 : Find and Copy Fonts folder in koreader into your Computer

The font folder location is `/usr/local/Trolltech/QTEmbedded-x.x.x-arm/lib/fonts`.

## Step 2 : De-obfuscate  fonts and Organise them in Prober organised Folders

```zsh
# 1. CREATE THE MASTER SCRIPT
cat << 'EOF' > complete_fix.py
#!/usr/bin/env python3
import os
import shutil
import zlib
from fontTools import ttLib

# Configuration
BASE_DIR = os.getcwd()
OUT_DIR = os.path.join(BASE_DIR, "Kobo Fonts")  # Changed to "Kobo Fonts"
PREFIX = "Ko "

def decrypt_qtd_zlib(filepath):
    try:
        with open(filepath, 'rb') as f: data = f.read()
    except: return None
    for offset in range(32):
        if len(data) <= offset: continue
        try:
            dec = zlib.decompress(data[offset:])
            if dec.startswith((b'\x00\x01\x00\x00', b'OTTO', b'ttcf')): return dec
        except: continue
    return None

def process_font(font_data, filename, is_ttc=False):
    tmp_in = "temp.tmp"
    with open(tmp_in, 'wb') as f: f.write(font_data)
    try:
        font = ttLib.TTFont(tmp_in, fontNumber=0 if is_ttc else -1)
        
        # 1. Identify Family & Style
        fam, style = None, None
        name_table = font['name']
        for record in name_table.names:
            if record.nameID == 16: fam = record.toUnicode()
            if record.nameID == 17: style = record.toUnicode()
        if not fam:
            for r in name_table.names:
                if r.nameID == 1: fam = r.toUnicode()
        if not style:
             for r in name_table.names:
                if r.nameID == 2: style = r.toUnicode()

        # Fallbacks
        if not fam: fam = os.path.splitext(filename)[0].split('-')[0]
        if not style: style = "Regular"

        # Kobo Nickel logic
        if "Nickel" in fam and "Kobo" not in fam: fam = "Kobo Nickel"

        # Apply Prefix
        clean_fam = fam.replace("Ko ", "").replace(" Ko", "").strip()
        final_fam = f"{PREFIX.strip()} {clean_fam}"

        # 2. Update Metadata
        for record in name_table.names:
            if record.nameID in (1, 16, 21):
                record.string = final_fam.encode(record.getEncoding())
            elif record.nameID == 4:
                new_full = f"{final_fam} {style}"
                record.string = new_full.encode(record.getEncoding())

        # 3. Save
        fam_dir = os.path.join(OUT_DIR, final_fam)
        if not os.path.exists(fam_dir): os.makedirs(fam_dir)
        new_name = f"{final_fam}-{style}.ttf"
        font.save(os.path.join(fam_dir, new_name))
        print(f"[OK] {filename} -> {final_fam}/{new_name}")
        return True
    except: return False
    finally:
        if os.path.exists(tmp_in): os.remove(tmp_in)

def main():
    if os.path.exists(OUT_DIR): shutil.rmtree(OUT_DIR)
    os.makedirs(OUT_DIR)
    
    # Process
    processed = []
    for f in os.listdir(BASE_DIR):
        if not os.path.isfile(f) or f.endswith('.py'): continue
        
        with open(f, 'rb') as fin: head = fin.read(4)
        data, is_ttc = None, False
        
        if head in (b'\x00\x01\x00\x00', b'OTTO'):
            with open(f, 'rb') as fin: data = fin.read()
        elif head == b'ttcf':
            is_ttc = True
            with open(f, 'rb') as fin: data = fin.read()
        else:
            data = decrypt_qtd_zlib(f) # Try decrypt
            if data and data.startswith(b'ttcf'): is_ttc = True
        
        if data and process_font(data, f, is_ttc):
            processed.append(f)

    # Cleanup
    for f in processed: 
        try: os.remove(f)
        except: pass
    
    # Trash
    for t in ["Ryumin.otf", "Gothic MB101.otf", "ub_arudjingxihei.ttf", "KBJ-TsukuMinPr6N-RB.ttf", "KBJ-UDKakugoPr6N-M.ttf"]:
        if os.path.exists(t): os.remove(t)
    
    # Remove old "Organized_Fonts" if it exists from previous runs
    if os.path.exists("Organized_Fonts"):
        shutil.rmtree("Organized_Fonts")
    
    print("\n✅ Done! Only 'Kobo Fonts' remains.")

    # SELF DESTRUCT
    try:
        os.remove(__file__)
    except:
        pass

if __name__ == "__main__":
    main()
EOF

# RUN IT
python3 complete_fix.py
```
