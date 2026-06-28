---
type:
  - my_setup
date: 2026-06-22
timestamp: 2026-06-22 15:27:45
tags:
  - my_setup
---
```insta-toc
---
title:
  name:
  level:
  center:
exclude:
style:
  listType:
omit:
levels:
  min:
  max:
---

# Table of Contents

- 1. Setup obsidian-ghostly-terminal plugin :
- 2. Setup github repo for the vault :
- 3. Setup git-vault-sync plugin :
- 4. Create Folder Structure:
- 5. Create 00 - Unsorted Folder for each directory:
- 6. Setup templater plugin from community plugins:
- 7. Disable core plugins that are not needed:
- 8. Enable core plugins that are needed:
- 9. Copy Templetes into 0102 - Templates  folder
- 10. Setup .css snippets
- 11. Setup Themes:
- 12. Setup BASIC HOTKEYS:
- 13. Setup Obsidian Settings
- 14. Setup Core plugins
- 13. Setup Community plugins
    - On Trial
```

## 1. Setup `obsidian-ghostly-terminal` plugin :
- Install [obsidian-ghostty-terminal](obsidian://show-plugin?id=ghostty-terminal) plugin
- Create a folder `00 - Unsorted` in vault root 								
```bash
mkdir -p "00 - Unsorted"
```
- Put all notes in `00 - Unsorted` for now
## 2. Setup github repo for the vault :
- [Create a new private repo](https://github.com/new) with your vault name
	- Generate fine-grained personal access token for repo from [here](https://github.com/settings/personal-access-tokens/)
	- Setup relevant permissions for token

| **Permission**      | **Access Level**   | **Why it's needed**                                          |
| ------------------- | ------------------ | ------------------------------------------------------------ |
| **Contents**        | **Read and Write** | To push/pull your actual notes and images                    |
| **Metadata**        | **Read-only**      | **Mandatory**; allows the plugin to "see" your repo.         |
| **Commit statuses** | **Read and Write** | Required for the plugin to verify successful syncs.          |
| **Workflows**       | **Read and Write** | **Only** if you keep your `.github/workflows` in your vault. |
## 3. Setup `git-vault-sync` plugin :
- Install [Git Vault Sync](obsidian://show-plugin?id=git-vault-sync) plugin
- Paste your PAT in the field
```secret-lock
1newAHCud/V5EIIewjPAl1MkEsJYAdQ7E6srvSh7Cz3Nyc/uDbBAnrZp5x3EsyXNWYeJHtP7lq7+EctGZteJPOYrv2vpncqPoK7/k9GOEzyf/t0c82NJPi5umBZZt18zaBi6GvKsCU1gopbdliop9c/fp8VtmaoMZCi6a/JpiXp0W86dzNFiQB0=
```
- Select the repo and Set author name to recognize different devices `r9PCx`
	- Add following to `Excluded files & folders`:
	```.gitignore
	.obsidian/workspace.json
	.obsidian/workspace-mobile.json
	.DS_Store
	.trash/
	.obsidian/
	01 - BACKEND/0106 - Documents/_DummyPDFs/
	01 - BACKEND/0106 - Documents/PDFs/
	00 - Unsorted/
	```
	- Initialize the repo

## 4. Create Folder Structure:
- 00 - Unsorted *(this would be unsynced device folder)*
- 01 - BACKEND
    - 0101 - Attachments
    - 0102 - Templates
    - 0103 - Bases
    - 0104 - Excalidraw
    - 0105 - Canvas
    - 0106 - Documents
	    - `_DummyPDFs` (excluded from sync)
	    - PDFs (excluded from sync)
    - 0107 - Scripts
- 03 - WIKI
    - 0300 - Tutorial
    - 0301 - English
    - 0302 - ComSci
    - 0303 - Psychology
    - 0304 - Mathematics
    - 0305 - Cosmology
    - 0306 - Philosophy
- 05 - ACADEMICS
	- 0501 - PGT
	- 0502 - AP
	- 0503 - TGT
- 07 - ROUGH NOTES
- 09 - PROJECTS
    - 0901 - Health
    - 0902 - Academics
    - 0903 - Writing
    - 0904 - Learn
    - 0905 - Home
    - 0906 - Oneshot
    - 0907 - Miscellaneous
- 11 - PRIVATE
    - 1101 - CC
        - Books
        - GraphicReads
        - Anime
        - AsianDramas
        - Movies
        - Shows
        - Podcasts
        - Games
        - Music
        - Links
    - 1102 - Journal
    - 1103 - Writings
    - 1104 - Quotes
    - 1105 - JournalArchive
    - 1106 - My Setups
    - 1107 - People

```zsh
mkdir -p \
"01 - BACKEND/"{"0101 - Attachments","0102 - Templates","0103 - Bases","0104 - Excalidraw","0105 - Canvas","0106 - Documents/"{"_DummyPDFs","PDFs"},"0107 - Scripts"} \
"03 - WIKI/"{"0300 - Tutorial","0301 - English","0302 - ComSci","0303 - Psychology","0304 - Mathematics","0305 - Cosmology","0306 - Philosophy"} \
"05 - ACADEMICS/"{"0501 - PGT","0502 - AP","0503 - TGT"} \
"07 - ROUGH NOTES" \
"09 - PROJECTS/"{"0901 - Health","0902 - Academics","0903 - Writing","0904 - Learn","0905 - Home","0906 - Oneshot","0907 - Miscellaneous"} \
"11 - PRIVATE/"{"1101 - CC/"{Books,GraphicReads,Anime,AsianDramas,Movies,Shows,Podcasts,Games,Music,Links},"1102 - Journal","1103 - Writings","1104 - Quotes","1105 - JournalArchive","1106 - My Setups","1107 - People"}
```

## 5. Create `00 - Unsorted` Folder for each directory:

```zsh
cat << 'EOF' > create_00unsorted_folders.sh && chmod +x create_00unsorted_folders.sh
#!/usr/bin/env bash
set -euo pipefail

ROOT="${1:-.}"
ROOT="$(cd "$ROOT" && pwd)"

shopt -s globstar nullglob

for dir in "$ROOT"/**/; do
    dir="${dir%/}"
    if [[ "$dir" == "$ROOT" ]]; then
        continue
    fi

    base="$(basename "$dir")"
    if [[ "$base" == "00 - Unsorted"* ]]; then
        continue
    fi
    if [[ "$base" == .* ]]; then
        continue
    fi
    if [[ "$base" == _* ]]; then
        continue
    fi

    if [[ "$base" =~ ^([0-9]+) ]]; then
        id="${BASH_REMATCH[1]}"
    else
        id="$base"
    fi

    target="$dir/00 - Unsorted ($id)"
    if [[ ! -d "$target" ]]; then
        mkdir -p "$target"
        printf 'Created: %s\n' "$target"
    fi
done
EOF
./create_00unsorted_folders.sh
rm -rf create_00unsorted_folders.sh
```

## 6. Setup templater plugin from community plugins:
- Install [templater](obsidian://show-plugin?id=templater-obsidian) plugin
- Set templater directory to `01 - BACKEND/0102 - Templates`
- Copy all templates to templater directory

## 7. Disable `core plugins` that are not needed:
- `templetes`
- `sync`

## 8. Enable `core plugins` that are needed:
- `format converter`
## 9. Copy Templetes into `0102 - Templates`  folder

## 10. Setup `.css snippets`
- create a folder `.obsidian/snippets`
- ```bash
  mkdir -p .obsidian/snippets
  ```
- copy the following snippets
	- 00 - collapse zen.css
	- 01 - zen.css
	- 02 - sidebar-folder-colors.css
	- 03 - fat-sidebar-folders.css
	- 04 - custom-heading-colors.css
	- 05 - fatheading.css
	- 06 - text-formatting.css
	- 07 - hover-properties.css
	- 08 - hide-popups.css
	- 09 - black-bg.css
	- 10 - hide-scroller.css
	- 11 - translucent-view.css
## 11. Setup Themes:
- Primary
- Solarized
- NichNeumor
## 12. Setup BASIC HOTKEYS:
- [x] ==**Toggle left sidebar**==: Alt + Shift + <-
- [x] ==**Toggle right sidebar**==: Alt + Shift + ->
- [x] ==**Toggle ribbon**==: Alt + Shift + UP
- [x] ==**Undo Close Tab**==: Ctrl + Shift + T 
- [x] ==**Delete current file**==: Ctrl + Delete
- [x] ==**Fold all headings and lists**==: Alt + H
- [x] ==**Unfold all headings and lists**==: Alt + Shift + H
- [x] ==**Toggle reading view**==: Alt+ Shift + R
- [x] **==Search & replace in current file==**: Ctrl + Shift + H
- [x] **==Toggle highlight==**: Ctrl + H
- [x] **==Toggle strikethrough==**: Ctrl + Shift + (-)
- [x] **==Split down==**: Ctrl + Shift + DOWN
- [x] **==Split right==**: Ctrl + Shift + ->
- [x] ==**Change Theme**==: Alt + Ctrl + T
- [x] ==Change light/dark mode==: Ctrl+Alt+Shift+M
- [x] **==Move current file to another folder==**: Alt + M
- [x] ==Move current tab to new window==: Ctrl+Alt+Shift+W
- [x] ==Toggle Live Preview/Source mode==: Ctrl + Alt + V
- [x] ==**Insert Footnote**==: Alt+Shift+F
- [x] ==Toggle readable line length==: Ctrl + Alt + Shift + L

## 13. Setup Obsidian Settings
- **`General`**
- **`Editor`**
- **`Files and Links`**
	- Default location for new notes: ` In folder specified below`
	- Folder to create new notes in: `07 - ROUGH NOTES`
	- Default location for new attachments: `In folder specified below`
	- Attachment folder path: `01 - BACKEND/0101 - Attachments`
	- New link format --> Path from current file
	- Check `Automatically update internal links`
	- Uncheck `Use [[Wikilinks]]`
	- Check `Show all file types`
- **`Appearance`**
	- Themes
		- NitchNeumor
	- Fonts
		- Interface font
			- Inter
		- Text font
			- Serious Shanns Bold
			- Trebuchet MS
			- Sama Devanagari
			- Noto Sans Devanagari
			- Mononoki Nerd Font
			- Noto Sans Devanagari
		- Monospace font
			- Mononoki Nerd Font Mono
		- Font Size --> 16
		- Windows frame style --> Native frame
## 14. Setup Core plugins
- **`Backlinks`**
	- Check `Show backlinks at the bottom of the note`
- **`Canvas`**
	- Default location for new canvas files: `In folder specified below`
	- Folder to create new canvas files in: `01 - BACKEND/0105 - Canvas`
- **`Command palette`**
- **`Daily notes`**
		- Date format: `YYYY-MM-DD`
		- New file location : `11 - PRIVATE/1102 - Journal/2026/Daily Notes`
			- Create a note first with templater to create the folder
- **`File recovery`**
- **`Note composer`**
- **`Page preview`**
- **`Quick switcher`**
	- Uncheck `Show Attachments`
## 13. Setup Community plugins
1. [Inline Secret Block](obsidian://show-plugin?id=inline-secret-block) by *Vladimir Artamonov*
2. [Style Setting](obsidian://show-plugin?id=obsidian-style-settings) by *mgmeyers*
3. [Calender](obsidian://show-plugin?id=calendar) by *Liam Cain*
	- Start week on: `Monday`
	- Check `Show week number`
	- Weekly note folder: `11 - PRIVATE/1102 - Journal/2026/Weekly Notes`
		- Create weekly note with templater first
4. [Dataview](obsidian://show-plugin?id=dataview) by *Michael Brenan*
	- Check `Enable JavaScript queries`
	- Check `Enable inline JavaScript queries`
5. [Excalidraw](obsidian://show-plugin?id=obsidian-excalidraw-plugin) by *Zsolt Viczian*
	- Basic
		- Excalidraw folder --> `01 - BACKEND/0104 - Excalidraw`
	- [x] ==Excalidraw: Create new drawing - IN A NEW TAB==: Alt+D
6. [Templeter](obsidian://show-plugin?id=templater-obsidian) by *SilentVoid*
	- Templete hotkeys
		- `01 - BACKEND/0102 - Templates/-new_note_.md`
	- [x] ==Templater: Create -new_note_==: Alt + N
	- [x] ==Templater: Open insert template modal==: Alt +Shift+ T
	- [x] ==Templater: Create new note from template==: Alt + T
7. [Ghostty Terminal](obsidian://show-plugin?id=ghostty-terminal) by *Mayank Lavania*
	- [x] ==Ghostty Terminal: Open terminal in new split== Ctrl + Alt + Shift + T
8. [Floating TOC](obsidian://show-plugin?id=floating-toc) by *pkm-er*
9. [Inline AI](obsidian://show-plugin?id=inlineai) By *FBarrca*
	- Select Gemini model: `models/gemini-2.5-flash`
	- API key:

	```secret-lock
		aO3HMgBMJkZYapzBn0YEyz0ObJCmhsl2i80ynVjtIXpP5enjeAzjxHVefQGEwrJNycH37crF8rbfSBZiuJKA1eBXjDMW2+2XJO8JV8VYIM4wzFFm
	```
	- Check `Message History`
	- [x] ==InlineAI: Show cursor tooltip==: Ctrl+Alt+K
10. [(Marker PDF to MD) OCR-AI](obsidian://show-plugin?id=marker-api) by *L3NOX*
	- MinstralAI API Key: 
	```secret-lock
	dg48Y0JxQKSbMg4qpcMz3ElGXu9WRybtknUM2eWqkzXLuYKVvKdr/zalbX74WdHvW/genk0NuIot4mNbfZbSdFWt5CG2rdcX+txSDw==
	```
11. [Paste image rename and convert](obsidian://show-plugin?id=paste-image-rename-convert) by *yiaos*
	- Duplicate number delimiter `_`
	- Check `Handle all attachments
12.  [Text Extractor](obsidian://show-plugin?id=text-extractor) by *Simon Cambier*
13. [Omnisearch](obsidian://show-plugin?id=omnisearch) by *Simon Cambier*
	- Check `PDFs content indexing`
	- Check `Image OCR indexing`
	- Check `Documents content indexing`
	- [x] ==Omnisearch: In-file search==: Ctrl+Shift+O
	- [x] ==Omnisearch: Vault search==: Ctrl+O (Remove conflict)
14. [Editing Toolbar](obsidian://show-plugin?id=editing-toolbar) by *pkm-er*
	- General **C1**
		- Uncheck: `Top Toolbar`
		- Check: `Following Toolbar`
15. [Various Complements](obsidian://show-plugin?id=various-complements) by *tadashi-alkawa*
16. [PDF++](obsidian://show-plugin?id=pdf-plus) by *Ryota Ushio*
	- **C 27/5**
	- Dummy PDFs for external files
		- Default location for new dummy PDF files: `In the folder specified below`
		- Dummy file folder path: `01 - BACKEND/0106 - Documents/_DummyPDFs`
	- [x] ==PDF++: Create dummy file for external PDF==: Ctrl + Alt + P
17. [BRAT](obsidian://show-plugin?id=obsidian42-brat) by *TfTHacker*
	- Beta plugin list
		1. `yubolun/obsidian-mysnippets-plugins`
	- Beta themes list
18. [Startpage](obsidian://show-plugin?id=start-page) by *kuzzh*
19. [Lite Tabs](obsidian://show-plugin?id=lite-tabs) by *oxdc*
	- Layout style: `List`
20. [WebDAV Sync](obsidian://show-plugin?id=webdav-sync) by *Hesperus*
	- WebDAV server URL: `https://app.koofr.net/dav/Koofr`
	- Account:
	```secret-lock
	mnKbOm37IqY3UMnmv47mrL/adJyU1xv++bkT0Btq+ysn2xdty+3gXH0T2HWDvcQ6B7W1wchrfBxGqsMRiEVqyA==
	```
	- Credential
	```secret-lock
	lXJD1zjwbC8M0Ob3vIqBh8Ma62z2rehr2TdITISXdZ4wUuSXkOCVWWwz087rEjmkCRx7WHNEQ/dWzu7R
	```
21. [Open vault in VSCode](obsidian://show-plugin?id=open-vscode) by *NomarCub*
22. [Heading Level Indent](obsidian://show-plugin?id=heading-level-indent) by *svonjoi*
23. [Insta TOC](obsidian://show-plugin?id=heading-level-indent) by *svonjoi*

### On Trial
1. [Chroma Key Paste](obsidian://show-plugin?id=chroma-key) by *Aziz Zahran*
2. [Petrify](obsidian://show-plugin?id=petrify) by *jo-minjun*
3. [Vaultkeeper](obsidian://show-plugin?id=vaultkeeper)
4. [Unmarkdown](obsidian://show-plugin?id=unmarkdown)
