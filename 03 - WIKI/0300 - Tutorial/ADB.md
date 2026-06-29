# 🤖 ADB Essentials: Android Debug Bridge Commands

The **Android Debug Bridge (adb)** is a versatile command-line tool for managing an Android device from a desktop terminal. It enables device management, file transfer, shell access, and wireless debugging. This note collects practical, copy-ready commands.

## 🛠️ Setup & Connection

### Setup Requirements

  * Install **Android SDK Platform Tools** and ensure the `adb` executable is in your system's **PATH**.
  * Enable **Developer options** and **USB/Wireless debugging** on the Android device.
  * Authorize the computer when prompted upon initial connection.

### Verify Connection

Use these commands to check the connection status and retrieve device information.

| Command                                      | Description                                                 |
| :------------------------------------------- | :---------------------------------------------------------- |
| `adb devices`                                | Lists all connected devices/emulators.                      |
| `adb get-state`                              | Reports the device status (`device`, `offline`, `unknown`). |
| `adb shell getprop ro.product.model`         | Gets the device model name (e.g., `Pixel 7`).               |
| `adb shell getprop ro.build.version.release` | Gets the Android OS version (e.g., `14`).                   |

-----

## 📁 File System Navigation

### Common Storage Paths

These paths are typically used for file transfers to and from shared storage.

| Path                  | Description                                                  |
| :-------------------- | :----------------------------------------------------------- |
| `/sdcard`             | The common symbolic link to the primary shared storage root. |
| `/storage/emulated/0` | The actual path for the primary shared storage root.         |
| `/sdcard/Download`    | Standard folder for downloaded files.                        |
| `/sdcard/DCIM/Camera` | Standard folder for photos and videos taken with the camera. |

### List and Search Files

Commands to list files, including recursive search and filtering.

| Command                                                     | Description                                                 |
| :---------------------------------------------------------- | :---------------------------------------------------------- |
| `adb shell ls -al /sdcard/`                                 | Lists shared storage root contents with details.            |
| `adb shell ls -al /sdcard/Download`                         | Lists a specific folder's contents.                         |
| `adb shell ls -Ral /sdcard/`                                | **Recursive list** of the entire shared storage tree.       |
| `adb shell 'find /sdcard -iname "*report*.pdf"'`            | Finds all files matching a case-insensitive name pattern.   |
| `adb shell ls -Ral /sdcard/DCIM/Camera \| grep -i '\.mp4$'` | Lists only files ending in `.mp4` within the Camera folder. |

-----

## 🔄 File Transfer Commands

The fundamental commands for moving files between your desktop and the Android device.

### Pull: Device → Desktop (Download)

| Command                                       | Description                                                             | Example                                           |
| :-------------------------------------------- | :---------------------------------------------------------------------- | :------------------------------------------------ |
| `adb pull <remote> <local_folder>/`           | Downloads a single file to a specific local folder.                     | `adb pull /sdcard/Download/file.pdf ~/Downloads/` |
| `adb pull <remote>`                           | Downloads a single file to the current local directory.                 | `adb pull /sdcard/Download/file.pdf`              |
| `adb pull -p <remote_folder> <local_folder>/` | Downloads an entire folder **recursively** with **progress** displayed. | `adb pull -p /sdcard/DCIM/Camera ~/Pictures/`     |

### Push: Desktop → Device (Upload)

| Command                                    | Description                                           | Example                                   |
| :----------------------------------------- | :---------------------------------------------------- | :---------------------------------------- |
| `adb push <local_file> <remote_folder>/`   | Uploads a single file to the specified device folder. | `adb push ./myfile.pdf /sdcard/Download/` |
| `adb push <local_folder> <remote_folder>/` | Uploads an entire folder **recursively**.             | `adb push ./MyFolder /sdcard/Documents/`  |

### Targeted Transfers (Filtering)

Use a combination of `find`, `xargs`, and `adb pull` to filter and copy specific file types.

| Command                                                                                             | Description                                               |
| :-------------------------------------------------------------------------------------------------- | :-------------------------------------------------------- |
| `adb shell 'find /sdcard/DCIM/Camera -type f -iname "*.mp4"' \| xargs -I{} adb pull "{}" ~/Videos/` | Pulls **only MP4 videos** from a specified folder tree.   |
| `adb shell 'find /sdcard/Download -type f -iname "*.pdf"' \| xargs -I{} adb pull "{}" ~/Downloads/` | Pulls **all PDF files** from a folder and its subfolders. |

-----

## 🗜️ Robust Archiving

Use `tar` to archive entire directories efficiently while **preserving the folder structure**.

### Create Tar Stream & Save Locally (Recommended)

This method streams the data directly, requiring less device storage.

| Step               | Command                                                 | Description                                                              |
| :----------------- | :------------------------------------------------------ | :----------------------------------------------------------------------- |
| 1. Create & Stream | `adb exec-out tar c sdcard/FolderName > FolderName.tar` | Creates a tar archive stream on the device and pipes it to a local file. |
| 2. Extract Locally | `tar xf FolderName.tar -C ~/`                           | Extracts the archive locally.                                            |

### Create GZipped Tar On-Device (If `tar` supports `-z`)

| Step              | Command                                                            | Description                                                           |
| :---------------- | :----------------------------------------------------------------- | :-------------------------------------------------------------------- |
| 1. Create Archive | `adb shell 'cd /sdcard && tar czf /sdcard/archive.tgz FolderName'` | Creates a compressed tarball (`.tgz`) on the device's shared storage. |
| 2. Pull Archive   | `adb pull /sdcard/archive.tgz .`                                   | Pulls the single, compressed archive file to the desktop.             |

-----

## 💻 Shell Essentials

### Interactive Shell Access

  * **Open interactive shell:**

    ```bash
    adb shell
    ```

  * **Useful commands inside the shell:**

    ```bash
    pwd             # Print working directory
    ls -al          # List contents with details
    df -h           # Check filesystem disk usage
    getprop         # List all system properties
    top -n 1        # Show a snapshot of running processes
    ```

### Storage Usage

| Command                   | Description                                                      |
| :------------------------ | :--------------------------------------------------------------- |
| `adb shell df -h /sdcard` | Checks storage usage specifically for the shared storage volume. |

-----

## 📱 App Management & Data

### APK Management

| Command                                                                                           | Description                                                             |
| :------------------------------------------------------------------------------------------------ | :---------------------------------------------------------------------- |
| `adb shell pm list packages`                                                                      | Lists all installed packages/apps.                                      |
| `adb shell pm list packages \| grep -i example`                                                   | Filters the list for a specific name pattern.                           |
| `adb shell pm path com.example.app`                                                               | Shows the file path(s) of a specific app's APK(s).                      |
| `adb shell pm path com.example.app \| sed 's/package://g' \| xargs -I{} adb pull "{}" ~/Desktop/` | A chained command to pull a specific app's **APK file** to the desktop. |

### App Data (Requires Root or Debuggable App)

**Note:** Access to `/data/data` (private app storage) is **restricted on stock, non-rooted devices.** The `run-as` command only works for **debuggable apps**.

| Command                                                                                               | Description                                                                            |
| :---------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------- |
| `adb shell 'run-as com.example.app ls -al /data/data/com.example.app/files'`                          | Tries to list private files for a debuggable app using `run-as`.                       |
| `adb shell 'run-as com.example.app cat /data/data/com.example.app/files/config.json' > ./config.json` | Pulls a private file by reading its contents with `cat` and piping the output locally. |

-----

## 📶 Wireless Debugging

Wireless debugging removes the need for a physical USB connection. **The device and desktop must be on the same network.**

### Android 11+ (Pairing Workflow)

1.  Enable **Wireless debugging** in Developer options.
2.  Get the **Pairing Code** and **IP address/port** from the device screen.
    ```bash
    adb pair PHONE_IP:PAIR_PORT
    # (Enter the pairing code when prompted)
    ```
3.  Connect using the device's IP and *standard* port.
    ```bash
    adb connect PHONE_IP:PORT
    ```
4.  Verify connection.
    ```bash
    adb devices
    ```

### Legacy TCP/IP Workflow (Older Android)

1.  **Start over USB:** Initiate the TCP/IP connection mode on the device.
    ```bash
    adb tcpip 5555
    ```
2.  **Disconnect USB** and connect remotely.
    ```bash
    adb connect PHONE_IP:5555
    ```

-----

## 📸 Media & Performance Tools

### Screenshots and Screen Recording

| Command                                                   | Description                                              |
| :-------------------------------------------------------- | :------------------------------------------------------- |
| `adb shell screencap -p /sdcard/screen.png`               | Takes a screenshot and saves it to shared storage.       |
| `adb pull /sdcard/screen.png`                             | Pulls the screenshot to the current local directory.     |
| `adb shell screenrecord --time-limit 30 /sdcard/demo.mp4` | Records the screen for a maximum of 30 seconds.          |
| `adb pull /sdcard/demo.mp4`                               | Pulls the recorded video to the current local directory. |

### Performance, Reliability, and Quoting

  * **Show progress:** Use the `-p` flag for `pull` operations, especially for large transfers.
    ```bash
    adb pull -p /sdcard/BigFolder ~/BigFolder/
    ```
  * **Quote paths with spaces:** Always enclose paths containing spaces in double quotes.
    ```bash
    adb pull "/sdcard/My Folder With Spaces/file.txt" .
    ```
  * **Retries:** Rerunning an identical `adb pull` command typically skips files that were already successfully copied and are identical, making it a simple retry strategy.

-----

## 🗑️ Cleanup (Use with Caution\!)

| Command                                       | Description                                                                     |
| :-------------------------------------------- | :------------------------------------------------------------------------------ |
| `adb shell rm /sdcard/Download/old.zip`       | Removes a single file on shared storage.                                        |
| `adb shell rm -r /sdcard/Documents/OldFolder` | **Recursively removes** a folder and all its contents (like `rm -rf` on Linux). |

-----

## ⚡ Quick Cheatsheet

A list of the most essential, day-to-day commands.

```bash
adb devices
adb shell ls -al /sdcard/
adb pull /sdcard/Download/file.pdf ~/Downloads/
adb push ./local.txt /sdcard/Documents/
adb shell 'find /sdcard -iname "*.pdf"'
adb exec-out tar c sdcard/Folder > Folder.tar
adb connect PHONE_IP:PORT
adb shell pm list packages
adb shell screencap -p /sdcard/s.png && adb pull /sdcard/s.png .
```

