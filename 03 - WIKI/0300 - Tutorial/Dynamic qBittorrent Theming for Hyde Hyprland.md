# Complete Tutorial: Dynamic qBittorrent Theming for Hyde/Hyprland

A comprehensive guide to automatically synchronize qBittorrent's appearance with your Hyde/Hyprland system themes using wallbash-generated colors.

---

## 📋 Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Installation](#installation)
4. [Complete Source Code](#complete-source-code)
5. [Configuration](#configuration)
6. [Testing & Verification](#testing--verification)
7. [Usage](#usage)
8. [Customization](#customization)
9. [Troubleshooting](#troubleshooting)
10. [Uninstallation](#uninstallation)

---

## Overview

This implementation creates a dynamic theming system for qBittorrent that:
- Automatically generates qBittorrent themes from wallbash colors
- Integrates seamlessly with Hyde's theme switching workflow
- Ensures visual harmony between qBittorrent and your desktop environment
- Requires minimal user intervention after initial setup

**How it works:**
1. You switch Hyde themes
2. Wallbash generates color palettes from your wallpaper
3. Theme generator script creates matching qBittorrent [.qbtheme](file:///home/right9zzz/.config/qBittorrent/themes/template.qbtheme) files
4. Restart qBittorrent to see the new theme

---

## Prerequisites

### Required Software

- **Hyde/HyDE** dotfiles installed and configured
- **Hyprland** compositor
- **qBittorrent** (any recent version supporting custom themes)
- **Wallbash** (part of Hyde for color generation)

### Required Knowledge

- Basic terminal/command line usage
- Text editor familiarity (nano, vim, or any editor)

### File System Locations

Ensure these paths exist or will be created:
- `~/.config/hypr/themes/` - Hyprland theme configuration
- `~/HyDE/Scripts/` - Hyde scripts directory
- `~/.local/lib/hyde/` - Hyde library scripts

---

## Installation

### Step 1: Create Directory Structure

```bash
# Create qBittorrent themes directory
mkdir -p ~/.config/qBittorrent/themes

# Create Hyde library directory (if not exists)
mkdir -p ~/.local/lib/hyde
```

### Step 2: Install QSS Template

Create the file: `~/.config/qBittorrent/themes/template.qbtheme`

Use your preferred text editor:

```bash
nano ~/.config/qBittorrent/themes/template.qbtheme
```

**Paste the complete template code** (see [QSS Template Code](#1-qss-template-templateqbtheme) section below).

### Step 3: Install Theme Generator Script

Create the file: `~/.local/lib/hyde/qbtheme_generator.sh`

```bash
nano ~/.local/lib/hyde/qbtheme_generator.sh
```

**Paste the complete generator script** (see [Generator Script Code](#2-theme-generator-script-qbtheme_generatorsh) section below).

Make it executable:

```bash
chmod +x ~/.local/lib/hyde/qbtheme_generator.sh
```

### Step 4: Install Optional Reload Helper

Create the file: `~/.local/lib/hyde/qbtheme_reload.sh`

```bash
nano ~/.local/lib/hyde/qbtheme_reload.sh
```

**Paste the reload script** (see [Reload Helper Code](#3-reload-helper-script-optional-qbtheme_reloadsh) section below).

Make it executable:

```bash
chmod +x ~/.local/lib/hyde/qbtheme_reload.sh
```

### Step 5: Integrate with Hyde Theme Switcher

Edit Hyde's theme patcher script:

```bash
nano ~/HyDE/Scripts/themepatcher.sh
```

Find the section around **line 351** that contains:

```bash
if [ "${3}" != "--skipcaching" ]; then
    "$HOME/.local/lib/hyde/swwwallcache.sh" -t "${Fav_Theme}"
    "$HOME/.local/lib/hyde/theme.switch.sh"
fi
```

**Add the qBittorrent integration** right after `theme.switch.sh`:

```bash
if [ "${3}" != "--skipcaching" ]; then
    "$HOME/.local/lib/hyde/swwwallcache.sh" -t "${Fav_Theme}"
    "$HOME/.local/lib/hyde/theme.switch.sh"
    
    # Update qBittorrent theme to match system theme
    if [ -x "$HOME/.local/lib/hyde/qbtheme_generator.sh" ]; then
        print_prompt -g "[qBittorrent] " "Updating theme..."
        "$HOME/.local/lib/hyde/qbtheme_generator.sh" "${Fav_Theme}" &>/dev/null && \
            print_prompt -g "[qBittorrent] " "Theme updated" || \
            print_prompt -y "[qBittorrent] " "Theme update failed (non-critical)"
    fi
fi
```

**Save and exit** (Ctrl+O, Enter, Ctrl+X in nano).

---

## Complete Source Code

### 1. QSS Template (template.qbtheme)

```css
/* qBittorrent QSS Theme Template - Generated from Hyde Wallbash Colors */
/* This template uses color placeholders that will be replaced by actual wallbash colors */

/* === GLOBAL STYLES === */
QWidget {
    background-color: #{{WALLBASH_PRY1}};
    color: #{{WALLBASH_TXT1}};
    selection-background-color: #{{WALLBASH_1XA4}};
    selection-color: #{{WALLBASH_TXT1}};
}

/* === MAIN WINDOW === */
QMainWindow {
    background-color: #{{WALLBASH_PRY1}};
}

QMainWindow::separator {
    background-color: #{{WALLBASH_1XA2}};
    width: 1px;
    height: 1px;
}

/* === MENU BAR === */
QMenuBar {
    background-color: #{{WALLBASH_PRY1}};
    color: #{{WALLBASH_TXT1}};
    border-bottom: 1px solid #{{WALLBASH_1XA2}};
}

QMenuBar::item {
    background-color: transparent;
    padding: 4px 8px;
}

QMenuBar::item:selected {
    background-color: #{{WALLBASH_1XA3}};
}

QMenuBar::item:pressed {
    background-color: #{{WALLBASH_1XA4}};
}

/* === MENUS === */
QMenu {
    background-color: #{{WALLBASH_1XA1}};
    color: #{{WALLBASH_TXT1}};
    border: 1px solid #{{WALLBASH_1XA3}};
}

QMenu::item {
    padding: 5px 25px 5px 20px;
}

QMenu::item:selected {
    background-color: #{{WALLBASH_1XA4}};
}

QMenu::separator {
    height: 1px;
    background-color: #{{WALLBASH_1XA2}};
    margin: 2px 5px;
}

QMenu::indicator {
    width: 13px;
    height: 13px;
}

/* === TOOLBAR === */
QToolBar {
    background-color: #{{WALLBASH_PRY1}};
    border: none;
    padding: 2px;
    spacing: 3px;
}

QToolButton {
    background-color: transparent;
    color: #{{WALLBASH_TXT1}};
    border: 1px solid transparent;
    border-radius: 3px;
    padding: 3px;
    margin: 1px;
}

QToolButton:hover {
    background-color: #{{WALLBASH_1XA3}};
    border-color: #{{WALLBASH_1XA4}};
}

QToolButton:pressed {
    background-color: #{{WALLBASH_1XA2}};
}

QToolButton:checked {
    background-color: #{{WALLBASH_1XA4}};
    border-color: #{{WALLBASH_1XA5}};
}

/* === STATUS BAR === */
QStatusBar {
    background-color: #{{WALLBASH_PRY1}};
    color: #{{WALLBASH_TXT1}};
    border-top: 1px solid #{{WALLBASH_1XA2}};
}

QStatusBar::item {
    border: none;
}

QStatusBar QLabel {
    padding: 2px 4px;
}

/* === TABLE VIEWS (Transfer List) === */
QTableView {
    background-color: #{{WALLBASH_PRY1}};
    alternate-background-color: #{{WALLBASH_1XA1}};
    selection-background-color: #{{WALLBASH_1XA4}};
    selection-color: #{{WALLBASH_TXT1}};
    gridline-color: #{{WALLBASH_1XA2}};
    border: 1px solid #{{WALLBASH_1XA2}};
    border-radius: 0px;
}

QTableView::item {
    padding: 2px;
}

QTableView::item:hover {
    background-color: #{{WALLBASH_1XA3}};
}

QTableView::item:selected {
    background-color: #{{WALLBASH_1XA4}};
    color: #{{WALLBASH_TXT1}};
}

/* === HEADER VIEW === */
QHeaderView {
    background-color: #{{WALLBASH_1XA2}};
    border: none;
}

QHeaderView::section {
    background-color: #{{WALLBASH_1XA2}};
    color: #{{WALLBASH_TXT1}};
    padding: 4px 8px;
    border: none;
    border-right: 1px solid #{{WALLBASH_1XA1}};
    border-bottom: 1px solid #{{WALLBASH_1XA1}};
    font-weight: bold;
}

QHeaderView::section:hover {
    background-color: #{{WALLBASH_1XA3}};
}

QHeaderView::section:pressed {
    background-color: #{{WALLBASH_1XA4}};
}

QHeaderView::down-arrow {
    image: none;
    border: none;
}

QHeaderView::up-arrow {
    image: none;
    border: none;
}

/* === TREE VIEW (Side Panel) === */
QTreeView {
    background-color: #{{WALLBASH_PRY1}};
    selection-background-color: #{{WALLBASH_1XA4}};
    selection-color: #{{WALLBASH_TXT1}};
    border: 1px solid #{{WALLBASH_1XA2}};
    show-decoration-selected: 1;
}

QTreeView::item {
    padding: 3px;
}

QTreeView::item:hover {
    background-color: #{{WALLBASH_1XA3}};
}

QTreeView::item:selected {
    background-color: #{{WALLBASH_1XA4}};
    color: #{{WALLBASH_TXT1}};
}

QTreeView::branch {
    background-color: #{{WALLBASH_PRY1}};
}

QTreeView::branch:hover {
    background-color: #{{WALLBASH_1XA2}};
}

/* === SCROLLBARS === */
QScrollBar:vertical {
    background-color: #{{WALLBASH_PRY1}};
    width: 14px;
    border: none;
    margin: 0px;
}

QScrollBar::handle:vertical {
    background-color: #{{WALLBASH_1XA3}};
    border-radius: 7px;
    min-height: 20px;
    margin: 2px;
}

QScrollBar::handle:vertical:hover {
    background-color: #{{WALLBASH_1XA5}};
}

QScrollBar::handle:vertical:pressed {
    background-color: #{{WALLBASH_1XA6}};
}

QScrollBar::add-line:vertical,
QScrollBar::sub-line:vertical {
    height: 0px;
}

QScrollBar::add-page:vertical,
QScrollBar::sub-page:vertical {
    background-color: transparent;
}

QScrollBar:horizontal {
    background-color: #{{WALLBASH_PRY1}};
    height: 14px;
    border: none;
    margin: 0px;
}

QScrollBar::handle:horizontal {
    background-color: #{{WALLBASH_1XA3}};
    border-radius: 7px;
    min-width: 20px;
    margin: 2px;
}

QScrollBar::handle:horizontal:hover {
    background-color: #{{WALLBASH_1XA5}};
}

QScrollBar::handle:horizontal:pressed {
    background-color: #{{WALLBASH_1XA6}};
}

QScrollBar::add-line:horizontal,
QScrollBar::sub-line:horizontal {
    width: 0px;
}

QScrollBar::add-page:horizontal,
QScrollBar::sub-page:horizontal {
    background-color: transparent;
}

/* === BUTTONS === */
QPushButton {
    background-color: #{{WALLBASH_1XA3}};
    color: #{{WALLBASH_TXT1}};
    border: 1px solid #{{WALLBASH_1XA4}};
    border-radius: 3px;
    padding: 5px 15px;
    min-height: 18px;
}

QPushButton:hover {
    background-color: #{{WALLBASH_1XA4}};
    border-color: #{{WALLBASH_1XA6}};
}

QPushButton:pressed {
    background-color: #{{WALLBASH_1XA2}};
    border-color: #{{WALLBASH_1XA3}};
}

QPushButton:disabled {
    background-color: #{{WALLBASH_1XA1}};
    color: #{{WALLBASH_1XA2}};
    border-color: #{{WALLBASH_1XA1}};
}

QPushButton:default {
    border: 2px solid #{{WALLBASH_1XA5}};
}

/* === INPUT FIELDS === */
QLineEdit,
QTextEdit,
QPlainTextEdit {
    background-color: #{{WALLBASH_1XA1}};
    color: #{{WALLBASH_TXT1}};
    border: 1px solid #{{WALLBASH_1XA2}};
    border-radius: 3px;
    padding: 3px;
    selection-background-color: #{{WALLBASH_1XA4}};
}

QLineEdit:focus,
QTextEdit:focus,
QPlainTextEdit:focus {
    border-color: #{{WALLBASH_1XA5}};
}

QLineEdit:disabled,
QTextEdit:disabled,
QPlainTextEdit:disabled {
    background-color: #{{WALLBASH_PRY1}};
    color: #{{WALLBASH_1XA2}};
}

/* === SPIN BOX === */
QSpinBox,
QDoubleSpinBox {
    background-color: #{{WALLBASH_1XA1}};
    color: #{{WALLBASH_TXT1}};
    border: 1px solid #{{WALLBASH_1XA2}};
    border-radius: 3px;
    padding: 3px;
}

QSpinBox:focus,
QDoubleSpinBox:focus {
    border-color: #{{WALLBASH_1XA5}};
}

QSpinBox::up-button,
QDoubleSpinBox::up-button {
    background-color: #{{WALLBASH_1XA2}};
    border-left: 1px solid #{{WALLBASH_1XA2}};
    border-radius: 0px;
}

QSpinBox::up-button:hover,
QDoubleSpinBox::up-button:hover {
    background-color: #{{WALLBASH_1XA3}};
}

QSpinBox::down-button,
QDoubleSpinBox::down-button {
    background-color: #{{WALLBASH_1XA2}};
    border-left: 1px solid #{{WALLBASH_1XA2}};
    border-radius: 0px;
}

QSpinBox::down-button:hover,
QDoubleSpinBox::down-button:hover {
    background-color: #{{WALLBASH_1XA3}};
}

/* === COMBO BOX === */
QComboBox {
    background-color: #{{WALLBASH_1XA1}};
    color: #{{WALLBASH_TXT1}};
    border: 1px solid #{{WALLBASH_1XA2}};
    border-radius: 3px;
    padding: 3px 5px;
    min-height: 18px;
}

QComboBox:hover {
    border-color: #{{WALLBASH_1XA4}};
}

QComboBox:focus {
    border-color: #{{WALLBASH_1XA5}};
}

QComboBox::drop-down {
    border: none;
    background-color: #{{WALLBASH_1XA2}};
    width: 20px;
}

QComboBox::drop-down:hover {
    background-color: #{{WALLBASH_1XA3}};
}

QComboBox::down-arrow {
    image: none;
    border: none;
}

QComboBox QAbstractItemView {
    background-color: #{{WALLBASH_1XA1}};
    color: #{{WALLBASH_TXT1}};
    border: 1px solid #{{WALLBASH_1XA3}};
    selection-background-color: #{{WALLBASH_1XA4}};
    selection-color: #{{WALLBASH_TXT1}};
}

/* === CHECKBOXES === */
QCheckBox {
    color: #{{WALLBASH_TXT1}};
    spacing: 5px;
}

QCheckBox::indicator {
    width: 15px;
    height: 15px;
    border: 1px solid #{{WALLBASH_1XA3}};
    border-radius: 3px;
    background-color: #{{WALLBASH_1XA1}};
}

QCheckBox::indicator:hover {
    border-color: #{{WALLBASH_1XA5}};
}

QCheckBox::indicator:checked {
    background-color: #{{WALLBASH_1XA4}};
    border-color: #{{WALLBASH_1XA5}};
}

QCheckBox::indicator:disabled {
    background-color: #{{WALLBASH_PRY1}};
    border-color: #{{WALLBASH_1XA1}};
}

/* === RADIO BUTTONS === */
QRadioButton {
    color: #{{WALLBASH_TXT1}};
    spacing: 5px;
}

QRadioButton::indicator {
    width: 15px;
    height: 15px;
    border: 1px solid #{{WALLBASH_1XA3}};
    border-radius: 8px;
    background-color: #{{WALLBASH_1XA1}};
}

QRadioButton::indicator:hover {
    border-color: #{{WALLBASH_1XA5}};
}

QRadioButton::indicator:checked {
    background-color: #{{WALLBASH_1XA4}};
    border-color: #{{WALLBASH_1XA5}};
}

/* === PROGRESS BAR === */
QProgressBar {
    background-color: #{{WALLBASH_1XA1}};
    border: 1px solid #{{WALLBASH_1XA2}};
    border-radius: 3px;
    text-align: center;
    color: #{{WALLBASH_TXT1}};
    height: 18px;
}

QProgressBar::chunk {
    background-color: #{{WALLBASH_1XA6}};
    border-radius: 2px;
}

/* === SLIDERS === */
QSlider::groove:horizontal {
    background-color: #{{WALLBASH_1XA1}};
    height: 6px;
    border-radius: 3px;
}

QSlider::handle:horizontal {
    background-color: #{{WALLBASH_1XA4}};
    width: 14px;
    margin: -4px 0;
    border-radius: 7px;
}

QSlider::handle:horizontal:hover {
    background-color: #{{WALLBASH_1XA6}};
}

QSlider::groove:vertical {
    background-color: #{{WALLBASH_1XA1}};
    width: 6px;
    border-radius: 3px;
}

QSlider::handle:vertical {
    background-color: #{{WALLBASH_1XA4}};
    height: 14px;
    margin: 0 -4px;
    border-radius: 7px;
}

QSlider::handle:vertical:hover {
    background-color: #{{WALLBASH_1XA6}};
}

/* === TAB WIDGET === */
QTabWidget::pane {
    border: 1px solid #{{WALLBASH_1XA2}};
    background-color: #{{WALLBASH_PRY1}};
    border-radius: 3px;
}

QTabWidget::tab-bar {
    alignment: left;
}

QTabBar::tab {
    background-color: #{{WALLBASH_1XA2}};
    color: #{{WALLBASH_TXT1}};
    border: 1px solid #{{WALLBASH_1XA1}};
    border-bottom: none;
    padding: 6px 12px;
    min-width: 80px;
    margin-right: 2px;
    border-top-left-radius: 3px;
    border-top-right-radius: 3px;
}

QTabBar::tab:hover {
    background-color: #{{WALLBASH_1XA3}};
}

QTabBar::tab:selected {
    background-color: #{{WALLBASH_1XA4}};
    border-bottom: 1px solid #{{WALLBASH_1XA4}};
}

QTabBar::tab:!selected {
    margin-top: 2px;
}

/* === DIALOGS === */
QDialog {
    background-color: #{{WALLBASH_PRY1}};
    color: #{{WALLBASH_TXT1}};
}

QDialogButtonBox {
    button-layout: 0;
}

/* === GROUP BOX === */
QGroupBox {
    background-color: #{{WALLBASH_PRY1}};
    border: 1px solid #{{WALLBASH_1XA2}};
    border-radius: 3px;
    margin-top: 10px;
    padding-top: 10px;
    font-weight: bold;
}

QGroupBox::title {
    color: #{{WALLBASH_TXT1}};
    subcontrol-origin: margin;
    subcontrol-position: top left;
    padding: 0 5px;
    background-color: transparent;
}

/* === TOOL TIP === */
QToolTip {
    background-color: #{{WALLBASH_1XA2}};
    color: #{{WALLBASH_TXT1}};
    border: 1px solid #{{WALLBASH_1XA4}};
    border-radius: 3px;
    padding: 4px;
}

/* === SPLITTER === */
QSplitter::handle {
    background-color: #{{WALLBASH_1XA2}};
}

QSplitter::handle:hover {
    background-color: #{{WALLBASH_1XA4}};
}

QSplitter::handle:horizontal {
    width: 2px;
}

QSplitter::handle:vertical {
    height: 2px;
}

/* === LIST WIDGET === */
QListWidget {
    background-color: #{{WALLBASH_PRY1}};
    selection-background-color: #{{WALLBASH_1XA4}};
    selection-color: #{{WALLBASH_TXT1}};
    border: 1px solid #{{WALLBASH_1XA2}};
}

QListWidget::item {
    padding: 3px;
}

QListWidget::item:hover {
    background-color: #{{WALLBASH_1XA3}};
}

QListWidget::item:selected {
    background-color: #{{WALLBASH_1XA4}};
    color: #{{WALLBASH_TXT1}};
}

/* === LABEL === */
QLabel {
    color: #{{WALLBASH_TXT1}};
    background-color: transparent;
}

/* === SPECIFIC QBITTORRENT WIDGETS === */

/* Search Tab */
SearchJobWidget QLabel {
    margin-right: 5px;
}

/* RSS Feed */
RSSSideBar QTreeWidget {
    background-color: #{{WALLBASH_PRY1}};
}

/* Execution Log */
#executionLog QTextEdit {
    font-family: monospace;
}

/* Properties panel */
PropertiesWidget {
    background-color: #{{WALLBASH_PRY1}};
}
```

---

### 2. Theme Generator Script (qbtheme_generator.sh)

```bash
#!/usr/bin/env bash
#|---/ /+------------------------------------------------+---/ /|#
#|--/ /-| qBittorrent Theme Generator for Hyde/Wallbash |--/ /-|#
#|/ /---+------------------------------------------------+/ /---|#
#
# This script generates qBittorrent .qbtheme files from wallbash colors
# Integrates with Hyde's theme switching system

set -euo pipefail

# Color output functions
print_msg() {
    local color="$1"
    shift
    case "$color" in
        green)  echo -e "\e[32m$*\e[0m" ;;
        yellow) echo -e "\e[33m$*\e[0m" ;;
        red)    echo -e "\e[31m$*\e[0m" ;;
        blue)   echo -e "\e[34m$*\e[0m" ;;
        *)      echo "$*" ;;
    esac
}

# Determine current theme name
get_current_theme() {
    local theme_name=""
    
    # Try to get from theme.conf GTK_THEME variable
    if [ -f "$HOME/.config/hypr/themes/theme.conf" ]; then
        theme_name=$(grep '^\$GTK_THEME' "$HOME/.config/hypr/themes/theme.conf" | cut -d'=' -f2 | tr -d ' ' || echo "")
    fi
    
    # Fallback: use provided argument or default
    if [ -z "$theme_name" ]; then
        theme_name="${1:-HydeTheme}"
    fi
    
    echo "$theme_name"
}

# Extract hex color from variable (removes alpha if present)
extract_color() {
    local color="$1"
    # Remove any leading # and alpha channel, keep only first 6 characters
    echo "$color" | sed 's/^#//' | cut -c1-6
}

# Main function
main() {
    local theme_name="${1:-$(get_current_theme)}"
    local colors_conf="$HOME/.config/hypr/themes/colors.conf"
    local template_file="$HOME/.config/qBittorrent/themes/template.qbtheme"
    local output_dir="$HOME/.config/qBittorrent/themes"
    local output_file="${output_dir}/${theme_name}.qbtheme"
    local qb_conf="$HOME/.config/qBittorrent/qBittorrent.conf"
    
    print_msg blue "=== qBittorrent Theme Generator ==="
    print_msg yellow "Theme: $theme_name"
    
    # Check if colors.conf exists
    if [ ! -f "$colors_conf" ]; then
        print_msg red "Error: colors.conf not found at $colors_conf"
        print_msg yellow "Make sure wallbash has generated colors for your theme"
        exit 1
    fi
    
    # Check if template exists
    if [ ! -f "$template_file" ]; then
        print_msg red "Error: Template not found at $template_file"
        exit 1
    fi
    
    # Parse colors from colors.conf (they use $ prefix in Hyprland format)
    print_msg green "Loading wallbash colors..."
    
    # Extract colors by parsing the file (not sourcing it)
    wallbash_pry1=$(grep '^\$wallbash_pry1 =' "$colors_conf" | awk '{print $3}' | tr -d ' ')
    wallbash_txt1=$(grep '^\$wallbash_txt1 =' "$colors_conf" | awk '{print $3}' | tr -d ' ')
    wallbash_1xa1=$(grep '^\$wallbash_1xa1 =' "$colors_conf" | awk '{print $3}' | tr -d ' ')
    wallbash_1xa2=$(grep '^\$wallbash_1xa2 =' "$colors_conf" | awk '{print $3}' | tr -d ' ')
    wallbash_1xa3=$(grep '^\$wallbash_1xa3 =' "$colors_conf" | awk '{print $3}' | tr -d ' ')
    wallbash_1xa4=$(grep '^\$wallbash_1xa4 =' "$colors_conf" | awk '{print $3}' | tr -d ' ')
    wallbash_1xa5=$(grep '^\$wallbash_1xa5 =' "$colors_conf" | awk '{print $3}' | tr -d ' ')
    wallbash_1xa6=$(grep '^\$wallbash_1xa6 =' "$colors_conf" | awk '{print $3}' | tr -d ' ')
    wallbash_1xa7=$(grep '^\$wallbash_1xa7 =' "$colors_conf" | awk '{print $3}' | tr -d ' ')
    wallbash_1xa8=$(grep '^\$wallbash_1xa8 =' "$colors_conf" | awk '{print $3}' | tr -d ' ')
    wallbash_1xa9=$(grep '^\$wallbash_1xa9 =' "$colors_conf" | awk '{print $3}' | tr -d ' ')
    wallbash_pry2=$(grep '^\$wallbash_pry2 =' "$colors_conf" | awk '{print $3}' | tr -d ' ')
    wallbash_txt2=$(grep '^\$wallbash_txt2 =' "$colors_conf" | awk '{print $3}' | tr -d ' ')

    # Create output directory if needed
    mkdir -p "$output_dir"
    
    # Extract colors (removing any rgba prefix/suffix and just getting hex)
    local WALLBASH_PRY1=$(extract_color "${wallbash_pry1}")
    local WALLBASH_TXT1=$(extract_color "${wallbash_txt1}")
    local WALLBASH_1XA1=$(extract_color "${wallbash_1xa1}")
    local WALLBASH_1XA2=$(extract_color "${wallbash_1xa2}")
    local WALLBASH_1XA3=$(extract_color "${wallbash_1xa3}")
    local WALLBASH_1XA4=$(extract_color "${wallbash_1xa4}")
    local WALLBASH_1XA5=$(extract_color "${wallbash_1xa5}")
    local WALLBASH_1XA6=$(extract_color "${wallbash_1xa6}")
    local WALLBASH_1XA7=$(extract_color "${wallbash_1xa7}")
    local WALLBASH_1XA8=$(extract_color "${wallbash_1xa8}")
    local WALLBASH_1XA9=$(extract_color "${wallbash_1xa9}")
    
    # Also get secondary color group for variety
    local WALLBASH_PRY2=$(extract_color "${wallbash_pry2}")
    local WALLBASH_TXT2=$(extract_color "${wallbash_txt2}")
    
    print_msg green "Generating theme file..."
    print_msg yellow "  Primary: #$WALLBASH_PRY1"
    print_msg yellow "  Text:    #$WALLBASH_TXT1"
    print_msg yellow "  Accent:  #$WALLBASH_1XA4"
    
    # Generate the theme file by substituting placeholders
    sed -e "s/{{WALLBASH_PRY1}}/${WALLBASH_PRY1}/g" \
        -e "s/{{WALLBASH_TXT1}}/${WALLBASH_TXT1}/g" \
        -e "s/{{WALLBASH_1XA1}}/${WALLBASH_1XA1}/g" \
        -e "s/{{WALLBASH_1XA2}}/${WALLBASH_1XA2}/g" \
        -e "s/{{WALLBASH_1XA3}}/${WALLBASH_1XA3}/g" \
        -e "s/{{WALLBASH_1XA4}}/${WALLBASH_1XA4}/g" \
        -e "s/{{WALLBASH_1XA5}}/${WALLBASH_1XA5}/g" \
        -e "s/{{WALLBASH_1XA6}}/${WALLBASH_1XA6}/g" \
        -e "s/{{WALLBASH_1XA7}}/${WALLBASH_1XA7}/g" \
        -e "s/{{WALLBASH_1XA8}}/${WALLBASH_1XA8}/g" \
        -e "s/{{WALLBASH_1XA9}}/${WALLBASH_1XA9}/g" \
        -e "s/{{WALLBASH_PRY2}}/${WALLBASH_PRY2}/g" \
        -e "s/{{WALLBASH_TXT2}}/${WALLBASH_TXT2}/g" \
        "$template_file" > "$output_file"
    
    print_msg green "✓ Theme file created: $output_file"
    
    # Update qBittorrent config if it exists
    if [ -f "$qb_conf" ]; then
        print_msg green "Updating qBittorrent configuration..."
        
        # Backup config
        cp "$qb_conf" "${qb_conf}.backup"
        
        # Check if CustomUITheme key exists
        if grep -q "General\\\\CustomUITheme" "$qb_conf"; then
            # Update existing key
            sed -i "s|^General\\\\\\\\CustomUITheme=.*|General\\\\\\\\CustomUITheme=${output_file}|" "$qb_conf"
        else
            # Add key under [Preferences] section
            if grep -q "^\[Preferences\]" "$qb_conf"; then
                # Add after [Preferences] line
                sed -i "/^\[Preferences\]/a General\\\\\\\\CustomUITheme=${output_file}" "$qb_conf"
            else
                # Create [Preferences] section
                echo -e "\n[Preferences]\nGeneral\\\\CustomUITheme=${output_file}" >> "$qb_conf"
            fi
        fi
        
        print_msg green "✓ Configuration updated"
        print_msg yellow "  Backup saved: ${qb_conf}.backup"
    else
        print_msg yellow "! qBittorrent config not found"
        print_msg yellow "  You can manually set the theme in qBittorrent:"
        print_msg yellow "  Tools → Options → Behavior → Interface → Use custom UI Theme"
        print_msg yellow "  Select: $output_file"
    fi
    
    # Check if qBittorrent is running
    if pgrep -x qbittorrent &>/dev/null || pgrep -x qbittorrent-nox &>/dev/null; then
        print_msg yellow "\n! qBittorrent is running"
        print_msg green "  ✓ Theme file updated: ${output_file}"
        print_msg yellow "  → Restart qBittorrent to see the new theme"
        print_msg blue "  (or change theme manually in: Tools → Options → Behavior → Interface)"
    fi
    
    print_msg green "\n✓ qBittorrent theme generation complete!"
    echo ""
}

# Run main function
main "$@"
```

---

### 3. Reload Helper Script (Optional) (qbtheme_reload.sh)

```bash
#!/usr/bin/env bash
#|---/ /+------------------------------------------+---/ /|#
#|--/ /-| qBittorrent Theme Reload Helper Script |--/ /-|#
#|/ /---+------------------------------------------+/ /---|#
#
# Optional reload automation via WebUI API or process restart

QB_HOST="${QB_HOST:-http://localhost:8080}"
QB_USER="${QB_USER:-admin}"
QB_PASS_FILE="$HOME/.config/qBittorrent/.webui_pass"

print_msg() {
    local color="$1"
    shift
    case "$color" in
        green)  echo -e "\e[32m$*\e[0m" ;;
        yellow) echo -e "\e[33m$*\e[0m" ;;
        red)    echo -e "\e[31m$*\e[0m" ;;
        blue)   echo -e "\e[34m$*\e[0m" ;;
        *)      echo "$*" ;;
    esac
}

reload_webui() {
    local theme_path="$1"
    
    if [ ! -f "$QB_PASS_FILE" ]; then
        print_msg yellow "WebUI password file not found at $QB_PASS_FILE"
        print_msg yellow "To enable automatic reload, create this file with your WebUI password"
        return 1
    fi
    
    local QB_PASS=$(cat "$QB_PASS_FILE")
    
    # Login and get cookie
    local cookie_file=$(mktemp)
    local response=$(curl -s -i --data "username=${QB_USER}&password=${QB_PASS}" \
        -c "$cookie_file" "${QB_HOST}/api/v2/auth/login" 2>/dev/null)
    
    if echo "$response" | grep -q "Ok."; then
        # Set preference
        curl -s --cookie "$cookie_file" \
            --data "json={\"general_ui_theme_path\":\"${theme_path}\"}" \
            "${QB_HOST}/api/v2/app/setPreferences" &>/dev/null
        
        rm -f "$cookie_file"
        print_msg green "✓ Theme reloaded via WebUI API"
        return 0
    else
        rm -f "$cookie_file"
        print_msg red "✗ WebUI authentication failed"
        return 1
    fi
}

reload_process() {
    if pgrep -x qbittorrent &>/dev/null; then
        print_msg yellow "Restarting qBittorrent..."
        pkill -x qbittorrent
        sleep 1
        nohup qbittorrent &>/dev/null &
        print_msg green "✓ qBittorrent restarted"
        return 0
    elif pgrep -x qbittorrent-nox &>/dev/null; then
        print_msg yellow "Restarting qbittorrent-nox..."
        pkill -x qbittorrent-nox
        sleep 1
        nohup qbittorrent-nox &>/dev/null &
        print_msg green "✓ qbittorrent-nox restarted"
        return 0
    else
        print_msg yellow "qBittorrent is not running"
        return 1
    fi
}

main() {
    local theme_path="$1"
    
    if [ -z "$theme_path" ]; then
        print_msg red "Usage: $0 <path-to-qbtheme-file>"
        exit 1
    fi
    
    # Check if qBittorrent is running
    if ! pgrep -x "qbittorrent" &>/dev/null && ! pgrep -x "qbittorrent-nox" &>/dev/null; then
        print_msg yellow "qBittorrent is not running - theme will apply on next start"
        exit 0
    fi
    
    # Try WebUI first, fall back to process restart
    if ! reload_webui "$theme_path"; then
        print_msg yellow "WebUI reload failed, restart qBittorrent manually or use process restart"
        # Uncomment the line below to enable automatic process restart
        # reload_process
    fi
}

main "$@"
```

---

## Configuration

### Initial Theme Application

After installation, you need to enable the theme in qBittorrent **once**:

1. **Close qBittorrent** (if running)

2. **Generate the first theme**:
   ```bash
   ~/.local/lib/hyde/qbtheme_generator.sh
   ```

3. **Start qBittorrent**

4. **Enable custom theme in qBittorrent UI**:
   - Go to: `Tools → Options` (or press `Alt+O`)
   - Navigate to: `Behavior` tab
   - Scroll to: `Interface` section
   - **Check** ✅ `Use custom UI Theme`
   - Click the **folder icon** 📁
   - Select your theme file (e.g., `/home/username/.config/qBittorrent/themes/Gruvbox-Retro.qbtheme`)
   - Click **OK**

5. **Theme applies immediately!**

### Optional: WebUI Reload Setup

For automatic theme reloading without restarting qBittorrent:

1. **Enable WebUI in qBittorrent**:
   - `Tools → Options → Web UI`
   - Check `Enable the Web User Interface`
   - Set username (default: `admin`) and password
   - Note the port (default: `8080`)

2. **Store password securely** (optional):
   ```bash
   echo "your-webui-password" > ~/.config/qBittorrent/.webui_pass
   chmod 600 ~/.config/qBittorrent/.webui_pass
   ```

3. **Use reload script**:
   ```bash
   ~/.local/lib/hyde/qbtheme_reload.sh "/path/to/theme.qbtheme"
   ```

---

## Testing & Verification

### Step 1: Verify File Installation

```bash
# Check template exists
ls -lh ~/.config/qBittorrent/themes/template.qbtheme

# Check generator script exists and is executable
ls -lh ~/.local/lib/hyde/qbtheme_generator.sh
[ -x ~/.local/lib/hyde/qbtheme_generator.sh ] && echo "✓ Executable" || echo "✗ Not executable"

# Check Hyde integration
grep -A 5 "qbtheme_generator" ~/HyDE/Scripts/themepatcher.sh
```

### Step 2: Test Theme Generation

```bash
# Generate theme for current system theme
~/.local/lib/hyde/qbtheme_generator.sh
```

**Expected output:**
```
=== qBittorrent Theme Generator ===
Theme: Gruvbox-Retro
Loading wallbash colors...
Generating theme file...
  Primary: #1F2223
  Text:    #FFFFFF
  Accent:  #57818F
✓ Theme file created: /home/username/.config/qBittorrent/themes/Gruvbox-Retro.qbtheme
✓ Configuration updated
```

### Step 3: Verify Generated Theme

```bash
# Check theme file was created
ls -lh ~/.config/qBittorrent/themes/*.qbtheme

# Verify colors were substituted (should see hex colors, not placeholders)
head -20 ~/.config/qBittorrent/themes/Gruvbox-Retro.qbtheme | grep "background-color"
```

**Should show**: `background-color: #1F2223;` (actual hex color, not `{{WALLBASH_PRY1}}`)

### Step 4: Test Integration with Hyde

```bash
# If you have themepatcher available, test it
# (This depends on your Hyde setup and how you switch themes)
# The qBittorrent generator should run automatically after theme switching
```

---

## Usage

### Daily Usage

**Once configured, theming is automatic!**

1. **Switch Hyde themes** using your preferred method:
   - Hyde theme selector GUI
   - CLI theme switcher
   - Any Hyde theme switching mechanism

2. **qBittorrent theme auto-generates** in the background

3. **Restart qBittorrent** to see the new theme

**That's it!** No manual intervention needed.

### Manual Theme Generation

**Generate for current theme:**
```bash
~/.local/lib/hyde/qbtheme_generator.sh
```

**Generate for specific theme:**
```bash
~/.local/lib/hyde/qbtheme_generator.sh "Catppuccin-Mocha"
```

**Generate for all Hyde themes:**
```bash
for theme in ~/.config/hyde/themes/*; do
    theme_name=$(basename "$theme")
    ~/.local/lib/hyde/qbtheme_generator.sh "$theme_name"
done
```

---

## Customization

### Modifying Colors

Don't like the default color mapping? Edit the template:

```bash
nano ~/.config/qBittorrent/themes/template.qbtheme
```

**Available color variables:**
- `{{WALLBASH_PRY1}}` - Primary background
- `{{WALLBASH_TXT1}}` - Text color
- `{{WALLBASH_1XA1}}` through `{{WALLBASH_1XA9}}` - Accent gradient
- `{{WALLBASH_PRY2}}` - Secondary primary color
- `{{WALLBASH_TXT2}}` - Secondary text color

**Example customization:**
```css
/* Make progress bars use a different accent */
QProgressBar::chunk {
    background-color: #{{WALLBASH_1XA7}};  /* Changed from 1XA6 to 1XA7 */
    border-radius: 2px;
}
```

After editing, regenerate themes:
```bash
~/.local/lib/hyde/qbtheme_generator.sh
```

### Creating Theme Variants

Create multiple templates for different aesthetic styles:

```bash
# Copy template to create variant
cp ~/.config/qBittorrent/themes/template.qbtheme \
   ~/.config/qBittorrent/themes/template-minimal.qbtheme

# Edit the variant
nano ~/.config/qBittorrent/themes/template-minimal.qbtheme

# Modify generator to use variant (advanced)
# Or manually generate with template substitution
```

### Adjusting UI Elements

**Want bigger buttons?**
```css
QPushButton {
    padding: 8px 20px;  /* Changed from 5px 15px */
    min-height: 24px;   /* Changed from 18px */
}
```

**Want different scrollbar width?**
```css
QScrollBar:vertical {
    width: 16px;  /* Changed from 14px */
}
```

**Want no rounded corners?**
```css
QPushButton {
    border-radius: 0px;  /* Changed from 3px */
}
```

---

## Troubleshooting

### Theme Not Applying

**Symptom:** qBittorrent still shows default theme

**Solutions:**

1. **Check theme checkbox in qBittorrent:**
   - `Tools → Options → Behavior → Interface`
   - Ensure `Use custom UI Theme` is **checked** ✅

2. **Verify theme path:**
   ```bash
   grep "CustomUITheme" ~/.config/qBittorrent/qBittorrent.conf
   ```
   Should show the correct path to your [.qbtheme](file:///home/right9zzz/.config/qBittorrent/themes/template.qbtheme) file

3. **Restart qBittorrent:**
   - Theme changes require a restart
   - Close and reopen qBittorrent

4. **Check file permissions:**
   ```bash
   ls -l ~/.config/qBittorrent/themes/*.qbtheme
   ```
   Should be readable (at least `-rw-r--r--`)

### Colors Look Wrong

**Symptom:** Theme applies but colors are incorrect or ugly

**Solutions:**

1. **Regenerate wallbash colors:**
   ```bash
   # Your Hyde wallbash command (depends on Hyde version)
   # Example:
   ~/.local/lib/hyde/wallbash.sh
   ```

2. **Regenerate qBittorrent theme:**
   ```bash
   ~/.local/lib/hyde/qbtheme_generator.sh
   ```

3. **Check colors.conf:**
   ```bash
   cat ~/.config/hypr/themes/colors.conf | head -20
   ```
   Ensure wallbash has generated colors

4. **Verify color substitution:**
   ```bash
   grep "background-color" ~/.config/qBittorrent/themes/*.qbtheme | head -5
   ```
   Should show actual hex colors like `#1F2223`, not placeholders

### Script Fails to Run

**Symptom:** Script errors or doesn't execute

**Solutions:**

1. **Check script permissions:**
   ```bash
   chmod +x ~/.local/lib/hyde/qbtheme_generator.sh
   ```

2. **Run script manually to see errors:**
   ```bash
   ~/.local/lib/hyde/qbtheme_generator.sh 2>&1 | tee /tmp/qbtheme_error.log
   cat /tmp/qbtheme_error.log
   ```

3. **Verify dependencies:**
   ```bash
   # Check required commands exist
   which sed awk grep
   ```

4. **Check colors.conf exists:**
   ```bash
   ls -l ~/.config/hypr/themes/colors.conf
   ```

### Integration Not Working

**Symptom:** Themes don't auto-generate when switching Hyde themes

**Solutions:**

1. **Verify Hyde integration:**
   ```bash
   grep -C 5 "qbtheme_generator" ~/HyDE/Scripts/themepatcher.sh
   ```
   Should show the integration code

2. **Test Hyde theme switching:**
   - Switch a Hyde theme manually
   - Check if qBittorrent generator runs (check terminal output)

3. **Check script location:**
   ```bash
   ls -l ~/.local/lib/hyde/qbtheme_generator.sh
   ```
   Must be in the correct location with execute permissions

4. **Manual integration test:**
   ```bash
   # Run what themepatcher would run
   ~/.local/lib/hyde/qbtheme_generator.sh "TestTheme"
   ```

### qBittorrent Won't Start or Crashes

**Symptom:** qBittorrent crashes or won't start after theme application

**Solutions:**

1. **Disable custom theme temporarily:**
   - Close qBittorrent
   - Edit `~/.config/qBittorrent/qBittorrent.conf`
   - Remove or comment out the `General\\CustomUITheme` line
   - Restart qBittorrent

2. **Check theme file syntax:**
   ```bash
   # Ensure no syntax errors in generated theme
   head -50 ~/.config/qBittorrent/themes/*.qbtheme
   ```
   Look for malformed CSS or missing brackets

3. **Restore backup config:**
   ```bash
   cp ~/.config/qBittorrent/qBittorrent.conf.backup \
      ~/.config/qBittorrent/qBittorrent.conf
   ```

4. **Regenerate theme from clean template:**
   ```bash
   rm ~/.config/qBittorrent/themes/*.qbtheme
   ~/.local/lib/hyde/qbtheme_generator.sh
   ```

---

## Uninstallation

To completely remove qBittorrent theming:

### Step 1: Remove Hyde Integration

Edit themepatcher.sh:
```bash
nano ~/HyDE/Scripts/themepatcher.sh
```

Remove/comment out these lines (around line 351):
```bash
# Update qBittorrent theme to match system theme
# if [ -x "$HOME/.local/lib/hyde/qbtheme_generator.sh" ]; then
#     print_prompt -g "[qBittorrent] " "Updating theme..."
#     "$HOME/.local/lib/hyde/qbtheme_generator.sh" "${Fav_Theme}" &>/dev/null && \
#         print_prompt -g "[qBittorrent] " "Theme updated" || \
#         print_prompt -y "[qBittorrent] " "Theme update failed (non-critical)"
# fi
```

### Step 2: Remove Scripts

```bash
rm ~/.local/lib/hyde/qbtheme_generator.sh
rm ~/.local/lib/hyde/qbtheme_reload.sh
rm ~/.local/lib/hyde/qbtheme_apply_fix.sh  # If you created this
```

### Step 3: Remove Themes

```bash
rm -rf ~/.config/qBittorrent/themes/
```

### Step 4: Reset qBittorrent

1. Open qBittorrent
2. Go to: `Tools → Options → Behavior → Interface`
3. **Uncheck** `Use custom UI Theme`
4. Click **OK**

Or edit config manually:
```bash
# Close qBittorrent first
sed -i '/General\\\\CustomUITheme/d' ~/.config/qBittorrent/qBittorrent.conf
```

---

## Appendix: Quick Reference

### File Locations

```
~/.config/qBittorrent/
├── qBittorrent.conf              # Main config
└── themes/
    ├── template.qbtheme          # Master template
    ├── README.md                 # Documentation
    └── [ThemeName].qbtheme       # Generated themes

~/.local/lib/hyde/
├── qbtheme_generator.sh          # Main generator
└── qbtheme_reload.sh             # Reload helper

~/HyDE/Scripts/
└── themepatcher.sh               # Modified (added hook)
```

### Common Commands

```bash
# Generate theme for current Hyde theme
~/.local/lib/hyde/qbtheme_generator.sh

# Generatefor specific theme
~/.local/lib/hyde/qbtheme_generator.sh "ThemeName"

# Check if theme file exists
ls -lh ~/.config/qBittorrent/themes/

# Verify colors in generated theme
grep "background-color" ~/.config/qBittorrent/themes/*.qbtheme | head

# Check qBittorrent config
grep "CustomUITheme" ~/.config/qBittorrent/qBittorrent.conf

# Backup qBittorrent config
cp ~/.config/qBittorrent/qBittorrent.conf ~/.config/qBittorrent/qBittorrent.conf.backup
```

### Color Variable Reference

| Variable | Purpose | Example Value |
|----------|---------|---------------|
| `{{WALLBASH_PRY1}}` | Primary background | `#1F2223` |
| `{{WALLBASH_TXT1}}` | Primary text | `#FFFFFF` |
| `{{WALLBASH_1XA1}}` | Accent shade 1 | `#294752` |
| `{{WALLBASH_1XA2}}` | Accent shade 2 | `#3A5F6B` |
| `{{WALLBASH_1XA3}}` | Accent shade 3 | `#4B707D` |
| `{{WALLBASH_1XA4}}` | Accent shade 4 | `#57818F` |
| `{{WALLBASH_1XA5}}` | Accent shade 5 | `#6594A3` |
| `{{WALLBASH_1XA6}}` | Accent shade 6 | `#7AB0C2` |
| `{{WALLBASH_1XA7}}` | Accent shade 7 | `#9AD3E6` |
| `{{WALLBASH_1XA8}}` | Accent shade 8 | `#AADEF0` |
| `{{WALLBASH_1XA9}}` | Accent shade 9 | `#CCF2FF` |

---

## Credits & Resources

- **Hyde/HyDE**: [prasanthrangan/hyprdots](https://github.com/prasanthrangan/hyprdots)
- **Hyprland**: [hyprwm/Hyprland](https://github.com/hyprwm/Hyprland)
- **qBittorrent**: [qbittorrent/qBittorrent](https://github.com/qbittorrent/qBittorrent)
- **Qt Stylesheet Reference**: [Qt Documentation](https://doc.qt.io/qt-5/stylesheet-reference.html)

---

**Enjoy your perfectly themed qBittorrent! 🎨✨**
