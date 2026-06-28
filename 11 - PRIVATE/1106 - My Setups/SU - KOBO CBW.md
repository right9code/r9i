---
type:
  - my_setup
date: 2026-06-27
timestamp: 2026-06-27 09:55:29
tags:
  - my_setup
---
## 1. Initial Setup

1. Setup KOBO Account
```
https://kobo.com/activate
```
- sign in with `right9postbox` after entering the code.

## 2. Install KOReader
1. Go to [KoboPatch Web UI](https://kp.nicoverbruggen.be/) using chrome
2. Click on `Connect my Kobo` and connect the KOBOeReader.
3. Select `Tweak my device with NickelMenu` 
4. Select `Install with preset (and customize)`
	1. **INTERFACE TWEAKS**
		- Set up NickelMenu preset (required)
			- Click Toggle 
				- Tab label : `R9K`
				- Setup the icon: You can find [here](https://icons8.com/icons/)
		- Tick `Simplify navigation tabs`
		- Tick `Hide home screen recommendations`
		- Tick `Hide suggestions next to My Books`
		- Tick `Hide home screen notices`
	2. TEXT AND TYPOGRAPHY
		- Tick `Install additional fonts`
		- Tick `Enable better typography`
	3. READING APPS
		- Tick `Install KOReader`
	4. ADVANCED
		- Tick `Install NickelClock`
		- Tick `Enable Sideload Mode
	5. LEGACY
		- Tick `Copy sample screensaver`
		- Tick `Prevent Calibre books from appearing in My Books`
5. Select `Don't make a backup` or `Back up important files`
6. Select `Write to Kobo`
7. Re-Do the same steps again for NickelMenu to propagate

## 3. Setup Fonts

- Fonts are stored in `fonts` directory
- Create directories: `CORE` and `TEMP`.
	- `CORE` (For Permanent fonts)
		- Kindle Fonts
			- K Bookerly
			- K Futura
			- K Palatino
			- K PMN Caecilia LT
		- Kobo Fonts
			- Ko Atkinson Hyperlegible
			- Ko Bitter
			- Ko Kobo Nickel
			- Ko Noto Sans
			- Ko OpenDyslexic
			- Ko Rakuten Sans
			- Ko Rakuten Serif
			- Ko Ubuntu
			- Ko Vollkorn
		- Devanagari
			- Mangal
			- Devanagari
		- Others
	- `TEMP` (For Testing)
		- [NV Fonts](https://github.com/nicoverbruggen/ebook-fonts)
		- 

## 3. Setup KOReader
### 1. Setup Folder Structure
```bash
mkdir -p "00 - Unsorted" "01 - ACADEMICS/01 - VAULTS" "02 - LIBRARY"/{'00 - SLEEP ROOM','01 - READING ROOM','02 - Fiction','03 - Non Fiction','04 - Light Novel','05 - GraphicRead','06 - Hindi'}
```
1. Folder Structure
	- `fonts` - to store fonts (already there)
	- `00 - Unsorted`
	- `01 - ACADEMICS`
	- `02 - LIBRARY`
		-  `00 - SLEEP ROOM` 
		- `01 - READING ROOM` (HOME)
		- `02 - Fiction`
		- `03 - Non Fiction`
		- `04 - Light Novel`
		- `05 - GraphicRead`
		- `06 - Hindi`
- Set "READING ROOM" as "HOME" Folder by long-pressing on it.
### 2.  Setup Interface fallback emoji fonts
1. Download `NotoEmoji-Regular.ttf`:
	- [NotoEmoji](https://fonts.google.com/noto/specimen/Noto+Emoji)
	- Unzip and copy to `fonts/CORE/`
2. Now run this script in `media/right9zzz/KOBOeReader`
```bash
cat << 'EOF' > /tmp/patch_kobo.sh
#!/bin/bash
KOBO_ROOT="$(pwd)"
EMOJI_SRC="$KOBO_ROOT/fonts/CORE/Noto_Emoji/static"
TARGET_FONT_DIR="$KOBO_ROOT/.adds/koreader/fonts/noto"
LUA_FILE="$KOBO_ROOT/.adds/koreader/frontend/ui/font.lua"

echo "⚙️ RUNNING LIVE KOREADER FONTS PATCHER ⚙️"
echo "------------------------------------------------"

# --- STEP 1: Font Management ---
if [ -f "$TARGET_FONT_DIR/NotoEmoji-Regular.ttf" ]; then
    read -p "❓ Fonts already exist in system target folder. Replace them? [y/N]: " choice
    choice="${choice,,}" # Convert to lowercase
    case "$choice" in 
        y|yes ) do_copy=true ;;
        * ) do_copy=false; echo "⏭️  Skipping font copying." ;;
    esac
else
    read -p "❓ Noto Emoji fonts are missing from system target folder. Copy them now? [y/N]: " choice
    choice="${choice,,}"
    case "$choice" in 
        y|yes ) do_copy=true ;;
        * ) do_copy=false; echo "⏭️  Skipping font copying." ;;
    esac
fi

if [ "$do_copy" = true ]; then
    echo "📁 Copying Noto Emoji static family into place..."
    if cp "$EMOJI_SRC"/*.ttf "$TARGET_FONT_DIR/"; then
        echo "   ✅ Fonts successfully placed in system folder."
    else
        echo "   ❌ Error: Failed to copy fonts from $EMOJI_SRC"
    fi
fi

echo ""

# --- STEP 2: Lua File Patching ---
if [ -f "$LUA_FILE" ]; then
    if grep -q "NotoEmoji-Regular.ttf" "$LUA_FILE"; then
        echo "✅ Checked font.lua: 'noto/NotoEmoji-Regular.ttf' is already present in the fallback matrix."
    else
        echo "⚠️  Checked font.lua: Missing NotoEmoji patch."
        read -p "❓ Would you like to patch font.lua with fallback entry index [9]? [y/N]: " choice
        choice="${choice,,}"
        case "$choice" in 
            y|yes )
                echo "📝 Applying precise Lua table injection..."
                
                awk '
                /\[8\] = "freefont\/FreeSerif.ttf",/ {
                    print $0;
                    print "        [9] = \"noto/NotoEmoji-Regular.ttf\",";
                    next;
                }
                { print }
                ' "$LUA_FILE" > /tmp/font.lua.tmp
                
                # Timestamped backup for safety
                cp "$LUA_FILE" "$LUA_FILE.$(date +%Y%m%d_%H%M%S).bak"
                mv /tmp/font.lua.tmp "$LUA_FILE"
                echo "   🎉 font.lua updated successfully!"
                ;;
            * )
                echo "⏭️  Leaving font.lua untouched."
                ;;
        esac
    fi
else
    echo "❌ Error: Could not find font.lua file path at: $LUA_FILE"
fi

# Force system write buffers to disk
echo "💾 Flushing file cache buffers to USB hardware storage..."
sync
EOF
chmod +x /tmp/patch_kobo.sh && /tmp/patch_kobo.sh && rm /tmp/patch_kobo.sh
echo "------------------------------------------------"
echo "✨ Execution complete. Storage synced. Safe to eject Kobo!"
```
### 3. Setup Plugins
Copy the plugins in `.adds/koreader/plugins` directory.
Plugins folders always end with `.koplgin` suffix
1. [Appstore Plugin](https://github.com/omer-faruq/appstore.koplugin) (Download and install rest of the plugins are setup through this)
	- Tools -> App Store
	- Search and Setup The Rest Of the Plugins
2. [Crash Log Viewer](https://github.com/Billiam/crashlog.koplugin)
	- Tools -> More Tools -> Crash Log Viewer
3. [Github Browser Plugin for KOReader](https://github.com/omer-faruq/webbrowser.koplugin)
	- Search -> GitNotes
		- Create a [Create a PAT](https://github.com/settings/personal-access-tokens) KOReader
		- Setup relevant permissions for token

| **Permission**      | **Access Level**   | **Why it's needed**                                          |
| ------------------- | ------------------ | ------------------------------------------------------------ |
| **Contents**        | **Read and Write** | To push/pull your actual notes and images                    |
| **Metadata**        | **Read-only**      | **Mandatory**; allows the plugin to "see" your repo.         |
| **Commit statuses** | **Read and Write** | Required for the plugin to verify successful syncs.          |
| **Workflows**       | **Read and Write** | **Only** if you keep your `.github/workflows` in your vault. |
```secret-lock
Zz2Sf2qqFdZZzbqUABUMFa88+3z/lOom//p6RfZ4Vu1BsC9CNWsKDIUIGy/BklxXjj3j2u2wfyjtLa400IjIZhY67DaeLAwm45kUa0AcMBG1FvtPVaWVcpgPH4m1fChsu8ae+FBQBrJFGFv6X6mctCC9LQ3Mj9p8Fvs3D9kWGqepGTXqbm410+g=
```
- Create a `token.txt` file with the tokens one in a line in root and import in the plugin menu
- Settings
	- Download Folder: `01 - ACADEMICS/00 - Unsorted`
	- Git workspace: Select `01 - ACADEMICS/01 - VAULTS`
	- Device Name: `R9CBW`
	- Auto-sync on open: `OFF`
- Setup `right9code/r9i`, `right9code/r9lp` repo 
	- Attach to device
4. [FilebrowserPlus](https://github.com/patelneeraj/filebrowserplus.koplugin)
	- Settings -> Network -> FilebrowserPlus
	- login: `admin` / pass: `admin12345678`
	- Check `Login without password`
5. [Localsend](https://github.com/kaikozlov/localsend.koplugin)(**armv7**)
	- Settings -> Network -> Localsend
	- Setup the Download Directory `00 - Unsorted`
6. [Assistant AI ](https://github.com/omer-faruq/assistant.koplugin)
	- Tools -> AI Assistant
	- Set api-keys in a new `configuration.lua` file in plugin folder copied from `configuration.sample.lua`
```secret-lock
E4CKqzj/UNvDUpiCeqpiFk0c/utVEmQypYmzfQeB7Hmbusc0cGei6ZGx/Y3T+SaWokWKhOuHtqxvATm6o8ptaEML/yE+562Gg5qFcD6eeypL73En87pLgMZrEmqK5X0ZBWaoRmk8vj8nlujjHfHOa2gWMHyAmY9nJYkxaKTwsFedP1kq8H9JRw4XZFeN8DRMN8ZM33TiUXvAAmacBv+y/LLO4AS6xpVyMzHaETBdtrjwjXJZ/ssR6eXb320yzEtKNgUdPv6ocNlSuRVn0qK7lXBQWrfdr9L8b6SQVXV806DKqIjZ2T6G1k4RhewzO47pqXHMRbDXJFMl63NGGqDEaauyfqui8ya8X7HWAfs0mnP37cr75BDXhcggfb7GH/K9LAuUWHesNbCYGrnfnOX78Yp/OA1zArg67MwU4uSTmVSrRqw4bqT84aS26IYB0KAB5/q9dr+ZBEBdK+0j7Mri5GksN8ItXVXniC9X+9/BZbX3c6EKBcfXKwtXD23bBmH9VHBcwPK1WIKoS30n8qQAMom4iPMej+0g7l/FmeJhBta0L3U8YNggLkbH+r2bVD+TB8tY+lkzOw==
```
6. [Anna's Fetch](https://github.com/right9code/annas-fetch.koplugin)
	- Search -> Annas Fetch
7.  [Z-Library(ZlibraryKO)](https://github.com/ZlibraryKO/zlibrary.koplugin)
	- Search -> Z-Library
	- Set Base url and credentials
Credentials:
```secret-lock
NJv8N1T5bebY/uE9JKJpRUpjb3Og7C1GoSjw9TGjQyMt4h2O3C0QgOJKRT3OG+sId4aGm6TYrm26RDlrHU1HxTSB
```

```secret-lock
I0WmGe58zYiir7V5BC0awvHRnBBCLWScc/sRL4KwjIoQrT0tG1vmTa/WMw9wGrZNhFo1m/egSFw=
```
8. [FileSync Plugin](https://github.com/abrahamnm/filesync.koplugin)
	- Settings -> Network -> FileSync
9. [Simple UI](https://github.com/doctorhetfield-cmd/simpleui.koplugin)
	- SimpleUI -> Settings
		- Home Screen
			- Edit Layout
				- Page 1 (Home)
					- Clock
						- Tick `Show Clock`
						- Tick `Show Date`
						- Tick `Show Battery`
						- Scale: `80`%
						- Clock Style: `Digital`
						- Alignment: `Center`
						- Date Spacing: `100`%
						- Battery Spacing: `100`%
					- Currently Reading
						- Items
							- Title
							- Author
							- Progress bar
						- Scale: `90`%
						- Text Size: `100`%
						- Cover: `100`%
						- Cover Spacing: `100`%
						- Tick `Show section label`
						- Tick `Frame`
					- Quick Action Row 2
						- Quick Actions`
							- 001 - Class VII
							- 002 - Class VIII
							- 003 - Class IX
							- 004 - Class X
						- Scale: `90`%
						- Label
							- Size: `80`%
							- Untick `Hide Label`
						- Button Type: `Bare`
					- Quick Action Row 3
						- Quick Actions
							- CLASS 07
							- CLASS 08
							- Ask AI
							- CLASS 09
							- CLASS 10
						- Scale: `70`%
						- Label
							- Size: `100`%
							- Untick `Hide Label`
						- Button Type: `Rounded Square`
						- Button Background: `Flat`
				- Page 2
					- Recent Books
						- Scale: `100`%
						- Text Size: `100`%
						- Cover size: `100`%
						- Tick `Show section label`
						- Tick `Frame`
						- Untick `Solid Background`
						- Tick `Progress bar`
						- Tick `Percentage text`
						- Untick `Percentage overlay on cover`
						- Tick `Show finished books`
						- Top Margin: `100`%
					- Quote of the Day
						- Source: `Quotes + My Highlights`
						- Scale: `100`%
						- Alignment: `Center`
					- Reading Stats
						- Items
							- Today - Time
							- Today - Pages
						- Scale: `100`%
						- Text Size: `100`%
						- Style: `List`
						- Alignment: `Center`
						- Top Margin: `100`%
					- Quick Action Row 1
						- Quick Actions
							- Anna's
							- USB
							- Cloud
							- Z-library
							- Github
						- Scale: `70`%
						- Label
							- SIze: `100`%
							- Untick `Hide Label`
						- Button Type: `Rounded Square`
						- Button Background: `Flat`
			- Wallpaper
				- Untick `Enable Wallpaper`
				- 
			- Presets
			- Behaviour
				- Tick `Start with Home Screen`
				- Scale
					- Untick `Lock Scale`
					- Module: `100`%
					- Labels: `100`%
				- Tick `Return to Book Folder`
				- Closing Book Notice: `Never`
				- Tick `Statistics Loading Notice`
				- Tick `Overflow Warning`
				- Untick `Settings on Long Tap`
				- Tick `Preserve Deleted Books in Statistics`
		- Bars
			- Status Bar
				- Tick `Enable Status Bar`
				- Items 
					- Left
						- Custom Text: `ABHI's CBW`
					- Center
					- Right
						- Wifi
						- Battery
				- Size: `100`%
				- Tick `Swipe Indicator`
				- Tick`Hide Wi-Fi Icon When Off`
				- Untick `Settings on Long Tap`
			- Navigation Bar
				- Tick `Enable Navigation Bar`
				- Tabs
					- Library
					- Home
					- Dictionary
				- Tab Style: `Icons only`
				- Bar Style: `Bare`
				- Bar Size: `80`%
				- Icons Size: `90%`
				- Label Size: `100`%
				- Bottom Margin: `100`%
				- Untick `Setting on Long Tap`
			- Title Bar
				- Tick `Enable Title Bar`
				- Library Buttons
					- Tick `Title - On`
					- Left
						- Back
						- Search
					- Right
						- Browse
						- Menu
				- Sub-page Buttons
					- Left
						- Back
					- Right
						- Menu
				- Button Size
					- Button: `Default`
			- Pagination Bar
				- Mode: `Default`
				- Home Screen Style: `Dot Pager`
				- Bar Size: `Default`
				- Untick `Show Page Count in Title Bar`
			- Quick Settings Bar
				- Tick `Enable Quick Settings Bar`
				- Quick Actions
					- Wi-Fi
					- Night Mode
					- Filebrowser
					- FileSync
					- LocalSend
					- Settings
				- Untick `Frontlight Slider`
				- Untick `Warmth Slider`
				- Label
					- SIze: `100`%
					- Untick `Hide Label`
				- Button Type: `Round` 
				- Button Background: `Flat`
				- Untick `Settings on Long Tap`
		- Library
			- Untick `Enable Library Custom Covers`
			- Tick `Enable Browse by Author/Series/Tags`			
			- Display
				- Display mode: `Mosiac with cover images`
				- Mosaic and detailed list settings
					- Items per page in portrait mosaic mode: `3x3`
					- Items per page in landscape mosaic mode: `4x2`
					- Items per page in portrait list mode: `10`
		- Style
			- Icons
				- System Icons
				- Quick Actions Icons
				- Icon Packs
				- Icon Presets
			- UI Font: `WP Class Clown`
			- Progress Bar Type: `Flat`
		- Quick Actions
			- 001 - Class VII
				- Icon: `ECAD`
			- 002 - Class VIII
				- Icon: `ECAD`
			- 003 - Class IX
				- Icon: `ECAD`
			- 004 - Class X
				- Icon: `ECAD`
			- Anna's
				- Icon: `E643`
			- Ask AI
				- Icon: `EECB`
			- CLASS 07
				- Icon: `F405`
			- CLASS 08
				- Icon: `F405`
			- CLASS 09
				- Icon: `F405`
			- CLASS 10
				- Icon: `F405`
			- Cloud
				- Icon: `EC8F`
			- Dictionary
				- Icon: `FA48`
			- Filebrowser
				- Icon: `E6C7`
			- FileSync
				- Icon: `241C`
			- Github
				- Icon: `ED2B`
			- Localsend
				- Icon: `E6D9`
			- USB
				- Icon: `E730`
			- Z-library
				- Icon: `E6E6`
10. [Sleep/Wake Tracker Plugin for KOReader](https://github.com/right9code/sleepwaketracker.koplugin)
	- Tools -> Sleep/Wake Tracker
11.  [X-ray](https://github.com/Ultimatejimmy/koreader-xray-plugin)
	- /Tools -> X-ray
	- set api keys in `xray_config.lua` or in the menu
```secret-lock
ExjB/A5V4t5D0kSKhm8z6XhfGpMtmxBALSUPWsQSJRT9pBWFZ6bpSDfHfPXjRCveJxXfuL45QIH+fJKaUpH9u+7vAazN6EVG95nEX1+f34lGeLQswqt94IhFU5hOIOwd5pMUo4vNj5fNA/Zyrawn3sny1qdg7FtK+0WyfMk+WE317txH2oiQEW3Rqc54VRlepMHdNUGjRzkOTio4bAqee74ELMPvicfUFMQRNwO1n7P6ot3W01nCmgo+npKwbPy5MIt11xHTVk2GnXq3ta40iIbN/FP4ilvHfTn8T8POLpkwFe9f7uvEFBGbEIC4AC0QHAjqkSwOR9879v1KfZ0MnWX8F1wd9hxSSzl6cFRvmfLX95Fae4EvdpVALJT+c1B3VAwKTm6UFifwgr3s9JsDdFGyT9enRuMqmlUmunmnEur3yeId5DXrvpVcZq6opUBoYAUmq/f2bsNDwedWC0mS64DXHPOeVegSExcipZMWQP2zZdOdKnTEnFmV/XyHQPwWNMWxkLRJDNSFTcJwKETgBKXjYSIMYDLT/C+0WPJhRnFMImUfwQ/LUB0K6YtK/hHxLCi9RQfDeYmUQd6q48c3N7aCo2Xkh26L1vx3LEWiP/xYK/sOC78aSAAmMc08UNXMy6nj/7t//6gveRiyP64Mhj59r8BTeuJrV041QAC9ZUH4EjitKASDc8/DYkr5BlsvsMcX512Ifk6F7xexgqKnjr9j5A7NbLcBHWMQBfKVwyB/05kT5Sxt42nB0dYaNednwmbJrPvd8MvyMgwzSClBpHZKw5hmeDeEXUulEThWv0hOXglW8DaY26McRfQ7KePlDgsbwkWzckYetPD/bV3LyqeoS5gY66XtzCtIITQZzJ0aeDWkds1NGFbfcU/xxnSIeEldj3NZdXdEtPz4wAQcLdY6EFNRaj+UlR9VIxjtsogRwDi8LKP8U4/G3KBOPL+aYzomwXgUiAqVzCWD7j6uOVjCTUjl8VMQCwngOFKRU9W40WeXuabcdUmf7hEr0wO+iBsyxri7XEa5Jtp5a2e+A42w1jVZ84R/s28kmLWZLaRjf5t1B/3+RM4GF1ZmSDUE6JUVegobipkMCEC7JNyr6mTJZGRwU3xdYYshg5TdtRxpdbjxxFyhZa+2deDRid9zXgk3Ml5IOYgORQfS0q7q7KEFXA7V9fdDSmXKVoMu0+FjmolbW9mR6sQUSX3mBUHM7iYh+h8qoWKhQZs7+2XDMW4QsMnua/ya11JdBppiLMev3lxJKAfAvCbsS+NgJjR+oMY9u36dwKgwKKa861DPFFN0G4yfGFk19GvZKiW0Lx1CtUXhf/oEaJLxpQ4wrUII05BwM6DsR1ccj8xVvnv2L1whwXI2CHdjSe6wmlFKudzFANUUoxuWST64mG4/EYKhRYSYaapOcagxj9Y+XXuTwztot5lCmkgYVSBa+Q7yYRRbQ5r/7aWRTIHgIBcqvuXsRm+6hOP2zWsYVTicFsm4CiMa2vuB2eYRD+EmNjdyLXtudn7qsplB5nHtqsOwVpLrpA/B/a23zJ8AQzylPcgB4US3YxPyC8zM/F9WPkDSwDdI53le/H9zo5w9x89Bk2cgPbk+0MyZ6OemikW77+vpS+ACHwjROB0zpuKm38o0h9lSTz7/agubbxq7v3oDAWD7jFdt1cFcCF9PJ5oFwC1f/7leF+OaixC9lDlLn/aZN+Wk7dMeLIL4a2KBVpk43N27x64TdM5Fd/6JlSBYq3+6FOdYypLxcVjCCXCQdUKu/fsc64b5r5Ef0Yh5gn/j3+2VTyei9A3286ahVIKuqIE2YO1/4Cy74drSNvhE+Di1Hds1NrGYEKYquesJiUKlkgMHBTOdYtMgXhG1yQDkU0LWURT3qEXKs0Ng2XUTZRlrQuoFk06plpubMzi/0SleHqKsjJUJ6fpMpt42UiI32PIVjxvCSXQ8nT69K8JUDXPyYbJ4N+vDQBpgZ0GgIzMU4Pu67Qh1YGMmzm46cCY8MJYC/SNOtfHKzyOnoBNQgM18hv7DZTtaqwURapWXNOQH6LRSC5Qst8DyCcUaq8dLS9IZ2+2VeLI9ggM0Nin9Xced2DZjZBJEb73jYeCloGDgmXxfbSzbWSvh/afsgt+7KB9pLbRT6U17L1hJYWualaaqOTQam3o1153ZBNJ8yYBjm+yQbnY3X/uDhHDk785GqirdocwxUkJ8s0XOl02D0MpCydu8BCj7Wfi974KPHsI6dp2q06Um0L54DTwv3RERx2sVOfwPtW4itNTbZK2IInhrobvego//suqtSFAvoDZCsLy+dlCaKT+UC/wMWvGJR+eUUkob1GInEeYkodOC6aUW
```
13. #[Rakuyomi](https://github.com/tachibana-shin/rakuyomi):`Kindle`
	- Search -> Rakuyomi
	- Create a `settings.json` file in rakuyomi folder `.adds/koreader/rakuyomi` as instructed [here](https://tachibana-shin.github.io/rakuyomi/user-guide/installation/installing-to-your-device.html)
14. [Send2telegram for KOReader](https://github.com/omer-faruq/send2telegram.koplugin)
	- Tools -> Send note to Telegram
	- copy  `send2telegram_configuration.sample.lua` to `send2telegram_configuration.lua`
	- Setup token and id
```secret-lock
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
```
15. [ScreenLock Pin](https://github.com/oleasteo/koreader-screenlockpin)
	- Settings -> Screen -> Lockscreen
		- Enable
16. [SimpleUI Extra](https://github.com/omer-faruq/simpleui_ext.koplugin)


### 4. Setup Patches
Make patches directory `.adds/koreader/patches`
Copy the patch lua lifes to `.adds/koreader/patches` directory.
You can manually disable patch by renaming file extension from `.lua`  to `.lua.disabled`
To avoid any errors always disable all patches first by adding suffix `.disabled` to each patch file and then enable from patched menu at `Tools -> More tools -> Patch management` 
- [2-fast-reading.lua](https://github.com/right9code/KOReader.patches/blob/main/2-fast-reading.lua)
- [2-reading-insights-stats.lua](https://github.com/right9code/KOReader.patches/blob/main/2-reading-insights-stats.lua)
- [2-reading-insights-popup.lua](https://github.com/right9code/KOReader.patches/blob/main/2-reading-insights-popup.lua)
- [2-reading-stats-popup.lua](https://github.com/right9code/KOReader.patches/blob/main/2-reading-stats-popup.lua)
- [2-cvs-receipt-frankenpatch.lua](https://github.com/right9code/KOReader.patches/blob/main/2-cvs-receipt-frankenpatch.lua)
- [2-mini-receipt-frankenpatch.lua](https://github.com/right9code/KOReader.patches/blob/main/2-mini-receipt-frankenpatch.lua)

### 5. Setup Menu
#### 4.1 **Top Menu** : `Set Defaults` 
- Setup Default Home Folder as `02 - LIBRARY/01 - READING ROOM` by long pressing folder.
- Similarly you can set an option default by long pressing on it.
1. **File Browser**
	- Display Mode -> `Mosaic with cover images`
		- Check `Use this mode everywhere`
	- Settings
		- Untick `Show hidden files`
		- Untick `Show unsupported files`
	- Start with: `Home Screen`
2. **Settings**
	- Frontlight
	- Night mode
	- Network
		- FilebrowserPlus
			- FIlebrowserPlus Data Path: `/mnt/onboard/`
			- Login without password
		- Localsend
			- Save directory: `/mnt/onboard/00 - Unsorted`
			- Settings
				- Device: `R9CBW`
	- Screen
		- Sleep screen
			- Wallpaper -> Show book cover on sleep screen
			- Sleep screen message
				- Container and position
					- TIck `Banner`
					- Vertical position: `top`
		- Lock screen
			- Tick `Enable`
			- Tick `Lock on wakeup`
	- Taps and gestures
		- #### **Gesture manager**
			- Tap corner
				- Bottom right: Device -> **Toggle touch input**
			- Long-press on corner
				- Bottom right: Device -> **Exit KOReader**
			- One-finger swipe
				- Top edge left: General -> **File browser**
				- Bottom edge left: General -> **Open previous document**
			- Two-finger tap corner
				- Top-Bottom: General -> **Reader statistics: reading insights**
		- Activate menu
			- Uncheck `with a tap`
	- Device
		- Time and date
			- Check `12-hour clock`
3. **/Settings** `open an epub file first`
	- Taps and gestures
		- #### **Gesture manager**
			- Tap corner
				- Bottom right: Device -> **Toggle touch input**
			- Long-press on corner
				- Top left: Screen and lights -> **Toggle night mode**
			- Double Tap
				- Bottom right -> Device -> **Toogle touch input**
			- One-finger swipe
				- Top edge left: General -> **File browser**
				- Bottom edge left: General -> **Open previous document**
			- Two-finger swipe
				- Top-Bottom edge -> Reader -> **Reading statistics: overview**
	- Status bar
		- Tick `Show progress bar`
			- Position: `Above Items`
			- Thickness, height & colors
				- Check `Thin` 
				- Height: `3`
			- Check `Show Initial Position Marker`
		- Status bar Items
			- Current Page * Current Time * Time left to finish chapter
		- Configure items
			- Check `Show all selected items at once`
			- Item font
				- Item font size: `14`
				- Check `Items in bold`
			- item symbols: `compact`
			- Item seperator: `bullet`
		- Check `Long press on status bar to skim`

4. **/Document Settings** `open an epub file first`
	- Style tweaks
		- Tables, links, images
			- Tables
				- Check `Center small tables`
				- Check `Alternate background color of table rows`
	- Font : `Ko Bitter`
5. **Tools**
	- Cloud Storage
		- Add -> WebDAV -> RIGHT9CODE
			- Server Display Name: `RIGHT9CODE`
			- WebDAV address: `https://app.koofr.net/dav/Koofr`
- Username:
```secret-lock
soNyj8ABA0vVnATwy+8nuzPez0yStW1buE/Q6/6lGo7ZH5Jax9PvqNrhUfui9U25becd5rdgOOV2C658lmPBPg==
```
- Password: 
```secret-lock
ZeXjPFaG1Yv3yovmAlj6SmhYEHfOnObE+9sAjn4gaWRnEHGHRGk9+ogKku3eE4b4sk4TlJWbf1xPWlvF
```

#### 4.2 **Bottom Menu** : `Set Defaults: On opening an EPUB` 
1. **C1**
2. **C2**
	- L/R Margins -> `L/R:70`
	- Sync T/B Margins -> `off`
	- Top Margin -> `50`
	- Bottom Margin -> `20`
3. **C3**
	- Zoom (dpi) -> `300`
	- Line Spacing -> 100% / 125%
4. **C4**
	- Default FONT SIZE -> `30`
5. **C5**
	- Font Weight -> `+1/2`

### 6. Dictionary setup
KOReader uses `StarDict` format for dictionaries.
A valid dictionary folder must contain: `.ifo` file (information), `.idx` file (index), `dict.dz` or `.dict` file (the data)

Copy the dictionary directory to `.adds/koreader/data/dict/`

1. [Reader.dict.EN](https://www.reader-dict.com/download/en)
2. [Shorter Oxford English Dictionary (SOED) StarDicts for KOReader](https://archive.org/details/soedrich-star-dict-2022-11-11)

### 7. OCR setup
Copy the OCR file to `.adds/koreader/data/tessdata`

1. [English TessData](https://github.com/tesseract-ocr/tessdata_best/blob/main/eng.traineddata)
2. [Japanese TessData](https://github.com/tesseract-ocr/tessdata_best/blob/main/jpn.traineddata)
3. [Korean TessData](https://github.com/tesseract-ocr/tessdata_best/blob/main/kor.traineddata)
