# Handy: Fresh Start Tutorial (Arch + Hyprland)

This guide provides a clean, 100% reliable setup for Handy on Wayland/Hyprland.

## 1. Clean the Slate (Full Wipe)
If you want to start truly fresh, run this command to kill the app and delete ALL data:
```bash
pkill -9 -f handy ; pkill -9 -f Handy ; rm -rf ~/.local/share/com.pais.handy ~/.local/share/handy ~/.cache/handy ~/.config/autostart/Handy.desktop
```

> [!CAUTION]
> This deletes your settings, history, and downloaded models.

## 2. Install Dependencies
Ensure the system has tools to "type" on Wayland:

```bash
sudo pacman -S wtype gtk-layer-shell
```

## 3. System Configuration
Your user must be in the `input` group to allow Handy to interact with keyboard states:

```bash
sudo usermod -aG input $USER
# NOTE: You must logout and log back in for this to take effect!
```

## 4. Hyprland Native Integration
Built-in global shortcuts often fail on Wayland. Use a native Hyprland bind instead.
Add this to your `~/.config/hypr/userprefs.conf`:

```ini
# Trigger Handy recording via Signal (Super + Ctrl + Space)
bind = SUPER_CTRL, SPACE, exec, pkill -USR2 handy
```

## 5. Optimal Handy Settings
Launch the app and set these in **Settings**:

- **Keyboard Implementation**: `Tauri` (Default)
- **Paste Method**: `Direct`
- **Always-on Microphone**: `OFF` (Crucial for Hyprland stability)
- **Overlay Position**: `Top` (or hidden)
- **Microphone**: Ensure your actual mic (e.g., realme Buds) is selected, not "Monitor".

## 6. Launching
Always run the AppImage with the Wayland backend for best performance:
```bash
GDK_BACKEND=wayland ~/AppImages/Handy_0.7.1_amd64.AppImage &
```

> If your microphone isn't picking up sound, run `pactl set-default-source <YOUR_MIC_NAME>`. You can find the name with `pactl list sources short`.

## 7. Switching Models
To balance speed and accuracy, you can change models in the settings:
1. **Open Settings**: Right-click the Handy tray icon or launch it from your app menu.
2. **Model Selection**: Choose a model from the dropdown. 
   - **Parakeet V3**: Best for speed, runs on any CPU.
   - **Whisper Small**: Good balance of accuracy and speed.
   - **Whisper Medium/Large/Turbo**: Best accuracy. Your **RTX 3050 GPU** is powerful enough to run these smoothly!
3. **Wait for Download**: The first time you pick a model, Handy will download it (this may take a minute).

### 8. Handy File Locations (For Reference)
If you need to backup files or edit code manually, here is where everything is:
- **Settings & Models**: `~/.local/share/com.pais.handy/`
  - [settings_store.json](file:///home/right9zzz/.local/share/com.pais.handy/settings_store.json): Edit this to change settings manually.
  - `models/`: Where your AI models live.
  - `history.db`: Your transcription database.
- **Logs**: `~/.local/share/com.pais.handy/logs/`
- **Autostart**: `~/.config/autostart/Handy.desktop`

### 9. Manual Model Switch (Fix if UI is missing)
If you can't see the model selection in the UI (sometimes it gets cut off):
1. Open `~/.local/share/com.pais.handy/settings_store.json`.
2. Find `"selected_model": "parakeet-tdt-0.6b-v3"`.
3. Change it to one of these:
   - `"whisper-small"`
   - `"whisper-medium"`
   - `"whisper-large-v3-turbo"` (Highly recommended for your GPU!)
4. Restart Handy. It will download the new model on launch.
