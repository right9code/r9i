---
type:
  - setup
date: 2026-01-28
timestamp: 2026-01-28 10:39:52

tags:
  - xx
  - yy
  - zz
---
# My OS Setup on Laptop

## 1. Pacstraping the OS
### 1.1 SSH into laptop
- Boot into `cachyos.iso`
- Check IP
```bash
ip a
```
- Setup SSH passed
```bash
passwd
```
- SSH into laptop from Termux 
```bash
ssh liveuser@ip
``` 
### 1.2 Setup `tmux`session
- Install `tmux`
```bash
sudo pacman -Sy
sudo pacman -S tmux
```
- Start tmux session on Laptop
```bash
tmux new -s in
```
- Attach session on Termux
```bash
tmux attach -t in
```
### 1.3 Update system clock
```bash
timedatectl set-ntp true
timedatectl status | sed -n '1p;/NTP service/p;/System clock synchronized/p'
```
### 1.4  Partition disk
- List disks
```bash
lsblk
```
- Delete any previous partitions on disk
```bash
sudo cfdisk /dev/nvme0n1
```
- Wipe the complete disk
```bash
sudo dd if=/dev/zero of=/dev/nvme0n1 bs=4M status=progress
```

OR  a faster way is
```bash
sudo blkdiscard -f /dev/nvme0n1
```
- Make two partitions 
	- EFI System : 1G
	- Linux filesystem : Rest
```bash
sudo cfdisk /dev/nvme0n1 # gpt 1GB EFI System rest Linux filesystem
```
- Format both partitions
```bash
sudo mkfs.vfat -F32 -n EFI /dev/nvme0n1p1
sudo mkfs.btrfs -f -L RIGHT9xOS /dev/nvme0n1p2
```
### 1.5 Create `btrfs` partitions

```bash
sudo mount /dev/nvme0n1p2 /mnt
cd /mnt
sudo btrfs subvolume create @
sudo btrfs subvolume create @home
sudo btrfs subvolume create @opt
sudo btrfs subvolume create @srv
sudo btrfs subvolume create @tmp
sudo btrfs subvolume create @log
sudo btrfs subvolume create @spool
sudo btrfs subvolume create @var_tmp
sudo btrfs subvolume create @lv
sudo btrfs subvolume create @lv_images
sudo btrfs subvolume create @wd
sudo btrfs subvolume create @wd_images
sudo btrfs subvolume create @pkg
sudo btrfs subvolume create @Z
sudo btrfs subvolume create @snapshots
cd /
sudo umount /mnt
```
### 1.6 Mount `btrfs` partitions

```bash
sudo mount -o rw,noatime,compress=zstd:3,ssd,discard=async,space_cache=v2,subvol=@ /dev/nvme0n1p2 /mnt

sudo mkdir -p /mnt/{efi,home,opt,srv,tmp,var,var/log,var/spool,var/tmp,var/lib/libvirt,var/lib/waydroid,var/cache/pacman/pkg,.snapshots}

sudo mount -o rw,noatime,compress=zstd:3,ssd,discard=async,space_cache=v2,subvol=@home /dev/nvme0n1p2 /mnt/home
sudo mount -o rw,noatime,compress=zstd:3,ssd,discard=async,space_cache=v2,subvol=@opt /dev/nvme0n1p2 /mnt/opt
sudo mount -o rw,noatime,compress=zstd:3,ssd,discard=async,space_cache=v2,subvol=@srv /dev/nvme0n1p2 /mnt/srv
sudo mount -o rw,noatime,compress=zstd:3,ssd,discard=async,space_cache=v2,subvol=@tmp /dev/nvme0n1p2 /mnt/tmp
sudo mount -o rw,noatime,compress=zstd:3,ssd,discard=async,space_cache=v2,subvol=@log /dev/nvme0n1p2 /mnt/var/log
sudo mount -o rw,noatime,compress=zstd:3,ssd,discard=async,space_cache=v2,subvol=@spool /dev/nvme0n1p2 /mnt/var/spool
sudo mount -o rw,noatime,compress=zstd:3,ssd,discard=async,space_cache=v2,subvol=@var_tmp /dev/nvme0n1p2 /mnt/var/tmp
sudo mount -o rw,noatime,compress=zstd:3,ssd,discard=async,space_cache=v2,subvol=@lv /dev/nvme0n1p2 /mnt/var/lib/libvirt
sudo mount -o rw,noatime,compress=zstd:3,ssd,discard=async,space_cache=v2,subvol=@wd /dev/nvme0n1p2 /mnt/var/lib/waydroid
sudo mount -o rw,noatime,compress=zstd:3,ssd,discard=async,space_cache=v2,subvol=@pkg /dev/nvme0n1p2 /mnt/var/cache/pacman/pkg
sudo mount -o rw,noatime,compress=zstd:3,ssd,discard=async,space_cache=v2,subvol=@snapshots /dev/nvme0n1p2 /mnt/.snapshots

sudo mkdir -p /mnt/{var/lib/libvirt/images,var/lib/waydroid/images}

sudo mount -o rw,noatime,compress=zstd:3,ssd,discard=async,space_cache=v2,subvol=@lv_images /dev/nvme0n1p2 /mnt/var/lib/libvirt/images
sudo mount -o rw,noatime,compress=zstd:3,ssd,discard=async,space_cache=v2,subvol=@wd_images /dev/nvme0n1p2 /mnt/var/lib/waydroid/images

sudo mount /dev/nvme0n1p1 /mnt/efi
```

### 1.7 Pacstrap cachyos
- Update PGP keys
```bash
sudo pacman-key --init && sudo pacman-key --populate archlinux cachyos
```
- Edit and set parallel downloads
```bash
sudo micro /etc/pacman.conf
```
- Pacstrap cachyos `1682.24 MiB`
```bash
sudo pacstrap -K /mnt base base-devel linux-cachyos-lts linux-cachyos-lts-headers linux-cachyos linux-cachyos-headers linux-firmware amd-ucode cachyos-settings cachyos-keyring cachyos-hooks cachyos-mirrorlist btrfs-progs snapper grub-btrfs efibootmgr grub linux-cachyos-nvidia-open linux-cachyos-lts-nvidia-open  nvidia-utils nvidia-settings lib32-nvidia-utils networkmanager dosfstools bluez bluez-utils openssh git tmux bat micro ncdu yazi zip bash-completion perl-image-exiftool nano
```
- Generate Fstab
```bash
genfstab -U /mnt | sudo tee /mnt/etc/fstab
```

## 2. Chroot and System Configuration
### 2.1 Chroot and setup new system
- Chroot new system
```bash
sudo arch-chroot /mnt
```
- Add `nofail` to non-essential folders
```bash
sed -i '/subvol=\/@\(snapshots\|tmp\|log\|spool\|var_tmp\|lv\|lv_images\|wd\|wd_images\|pkg\)/ s/\([[:space:]]*[0-9][[:space:]]*[0-9][[:space:]]*\)$/,nofail \1/' /etc/fstab
```


- Disable `cow` on image folders
```bash
sudo chattr -VR +C /var/lib/libvirt/images
sudo chattr -VR +C /var/lib/waydroid/images
```
- Set hostname, clock, locale and hosts
```bash
echo "RIGHT9xOS" > /etc/hostname
ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime && hwclock --systohc
echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
echo "127.0.1.1 RIGHT9xOS.localdomain RIGHT9xOS" >> /etc/hosts
```
- Setup root `passwd`
```bash
passwd
```
- Setup new user
```bash
useradd -m -G wheel,video,audio,optical,storage,input right9zzz && passwd right9zzz
```
- Set `wheel` as sudoer
```bash
sed -i 's/^# %wheel ALL=(ALL:ALL) ALL/%wheel ALL=(ALL:ALL) ALL/' /etc/sudoers
```
- Set keymaps
```bash
echo "KEYMAP=us" > /etc/vconsole.conf
```
- Setup zram
```bash
echo -e "[zram0]\nzram-size = ram / 2\ncompression-algorithm = zstd\nswap-priority = 100" > /etc/systemd/zram-generator.conf
```
### 2.2 Setup grub
- Install dual boot support
```bash
pacman -S fuse3 ntfs-3g os-prober
```
- Uncomment `GRUB_DISABLE_OS_PROBER=false
```bash
sudo sed -i 's/^#\s*\(GRUB_DISABLE_OS_PROBER=false\)/\1/' /etc/default/grub
```
- Setup grub parameters
```bash
sed -i 's/GRUB_CMDLINE_LINUX_DEFAULT=.*/GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet nvidia-drm.modeset=1"/' /etc/default/grub
sed -i 's/GRUB_PRELOAD_MODULES="part_gpt part_msdos"/GRUB_PRELOAD_MODULES="part_gpt part_msdos btrfs"/' /etc/default/grub
```
- Install grub
```bash
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=right9xOS --recheck
grub-mkconfig -o /boot/grub/grub.cfg
```
### 2.3 Enable relevant services 
```bash
systemctl enable NetworkManager bluetooth sshd fstrim.timer
```
### 2.4 Generate initramfs & Exit
```bash
mkinitcpio -P
exit
```
### 2.5 Unmount and Reboot
```bash
sudo umount -lR /mnt && reboot
```

## 3. Setup cachyos-repo on Base System 

### 3.1 SSH into laptop
- Check IP
```bash
ip a
```
- SSH into laptop from Termux 
```bash
ssh right9zzz@ip
``` 
### 3.2 Setup `tmux` session
- Start tmux session on Laptop
```bash
tmux new -s in
```
- Attach session on Termux
```bash
tmux attach -t in
```
### 3.3 Setup bash aliases
- Setup `~/.bash_aliases` file
```bash
# Create (or overwrite) the universal alias file with the improved aal function
cat << 'EOF' > ~/.bash_aliases
alias c='clear'
alias nano='micro'


# aal: Add Alias Locally
# Usage: aal "g=git status"
aal() { 
    if [ -z "$1" ]; then
        echo "Usage: aal 'name=command'"
        return 1
    fi
    echo "alias $1" >> ~/.bash_aliases
    . ~/.bash_aliases
    echo "✅ Alias '$1' added to ~/.bash_aliases and sourced."
}
EOF
```
- Setup `~/.bashrc` for alias
```bash
# Add the loader to .bashrc
echo '[[ -f ~/.bash_aliases ]] && . ~/.bash_aliases' >> ~/.bashrc

# Refresh Bash configuration
source ~/.bashrc
```
### 3.4 Setup `cachyos-repo`
- Enable multilib support
```bash
sudo sed -i '/^#\[multilib\]$/,/^#Include = \/etc\/pacman.d\/mirrorlist$/ s/^#//' /etc/pacman.conf
```
- Install cachyos repositories `1096.17 MiB `
```bash
curl -O https://mirror.cachyos.org/cachyos-repo.tar.xz
tar xvf cachyos-repo.tar.xz && cd cachyos-repo
sudo ./cachyos-repo.sh
cd ..
rm -rf cachyos*
```
- Reboot
```bash
sudo reboot
```

## 4. Setup Base System 

### 4.1 SSH into laptop
- Check IP
```bash
ip a
```
- SSH into laptop from Termux 
```bash
ssh right9zzz@ip
``` 
### 4.2 Setup `tmux`session
- Start tmux session on Laptop
```bash
tmux new -s in
```
- Attach session on Termux
```bash
tmux attach -t in
```

### 4.0 Setup `tailscale`
- Download the `tailscale` package
```bash
sudo pacman -Syu tailscale
```
- Enable `tailscale` service
```bash
sudo systemctl enable --now tailscaled
```
- Generate the login url
```bash
sudo tailscale up --force-reauth
```

### 4.3 Setup firewall
- Setup tmux session on laptop
```bash
tmux new -s in
ip a
```
- Connect session on termux
```bash
ssh right9zzz@ip
tmux attach -t in
```
- Set up `ufw`
```bash
sudo pacman -S ufw
sudo systemctl enable --now ufw
sudo ufw enable
sudo ufw allow 22/tcp # or sudo ufw allow ssh
sudo ufw limit ssh # rate-limit brute force
sudo ufw allow in on tailscale0
```
- Mount the `~/Z` folder
```
mkdir -p ~/Z
# 1. Add the line
echo "UUID=$(sudo blkid -s UUID -o value /dev/nvme0n1p2) /home/right9zzz/Z btrfs rw,noatime,compress=zstd:3,space_cache=v2,subvol=@Z,nofail 0 0" | sudo tee -a /etc/fstab

# 2. Show what was added (last 3 lines)
tail -n 3 /etc/fstab

# 3. Test the new entry
sudo mount -a
```

### 4.4 Setup additional home sub-volumes 

```bash
# Home Directory
sudo btrfs subvolume create --parents ~/.zen
sudo btrfs subvolume create --parents ~/.mozilla
sudo btrfs subvolume create --parents ~/.vscode-oss
sudo btrfs subvolume create --parents ~/.vscode
sudo btrfs subvolume create --parents ~/.abdm
sudo btrfs subvolume create --parents ~/.ssh
sudo btrfs subvolume create --parents ~/.lmstudio
sudo btrfs subvolume create --parents ~/.AppImages
sudo btrfs subvolume create --parents ~/Downloads
sudo btrfs subvolume create --parents ~/PWD

# ~/.config/
sudo btrfs subvolume create --parents ~/.config/BraveSoftware
sudo btrfs subvolume create --parents ~/.config/chromium
sudo btrfs subvolume create --parents ~/.config/google-chrome
sudo btrfs subvolume create --parents ~/.config/VSCodium
sudo btrfs subvolume create --parents ~/.config/mpv
sudo btrfs subvolume create --parents ~/.config/qBittorrent
sudo btrfs subvolume create --parents ~/.config/vesktop
sudo btrfs subvolume create --parents ~/.config/Antigravity
sudo btrfs subvolume create --parents ~/.config/Proton\ Pass

# ~/.local/share/
sudo btrfs subvolume create --parents ~/.local/share/qBittorrent
sudo btrfs subvolume create --parents ~/.local/share/fonts
sudo btrfs subvolume create --parents ~/.local/share/lutris
sudo btrfs subvolume create --parents ~/.local/share/AyuGramDesktop
sudo btrfs subvolume create --parents ~/.local/share/gnome-boxes/images

# ~/.cache/
sudo btrfs subvolume create --parents ~/.cache/yay
sudo btrfs subvolume create --parents ~/.cache/paru
sudo btrfs subvolume create --parents ~/.cache/pip
```
### 4.5 Setup permissions for the sub-volumes

```bash
sudo chown -Rv $USER:$USER ~
sudo chmod 700 ~/.ssh
```

```bash

# ~/.local/share/
sudo chown -Rv $USER:$USER ~/.local/share/AyuGramDesktop
sudo chown -Rv $USER:$USER ~/.local/share/qBittorrent
sudo chown -Rv $USER:$USER ~/.local/share/lutris
sudo chown -Rv $USER:$USER ~/.local/share/fonts
sudo chown -Rv $USER:$USER ~/.local/share/gnome-boxes/images

sudo chown -Rv $USER:$USER ~/.local/share/gnome-boxes
sudo chown -Rv $USER:$USER ~/.local/share
sudo chown -Rv $USER:$USER ~/.local

# ~/.config/
sudo chown -Rv $USER:$USER ~/.config/google-chrome
sudo chown -Rv $USER:$USER ~/.config/BraveSoftware
sudo chown -Rv $USER:$USER ~/.config/qBittorrent
sudo chown -Rv $USER:$USER ~/.config/chromium
sudo chown -Rv $USER:$USER ~/.config/VSCodium
sudo chown -Rv $USER:$USER ~/.config/vesktop
sudo chown -Rv $USER:$USER ~/.config/mpv
sudo chown -Rv $USER:$USER ~/.config/Antigravity
sudo chown -Rv $USER:$USER ~/.config/Proton Pass


sudo chown -Rv $USER:$USER ~/.config

# ~/.cache/
sudo chown -Rv $USER:$USER ~/.cache/pip
sudo chown -Rv $USER:$USER ~/.cache/yay
sudo chown -Rv $USER:$USER ~/.cache/paru

sudo chown -Rv $USER:$USER ~/.cache

# Home Directory
sudo chown -Rv $USER:$USER ~/.vscode-oss
sudo chown -Rv $USER:$USER ~/.AppImages
sudo chown -Rv $USER:$USER ~/.lmstudio
sudo chown -Rv $USER:$USER ~/.mozilla
sudo chown -Rv $USER:$USER ~/.vscode
sudo chown -Rv $USER:$USER ~/.abdm
sudo chown -Rv $USER:$USER ~/.ssh
sudo chown -Rv $USER:$USER ~/.zen
sudo chown -Rv $USER:$USER ~/Downloads
sudo chown -Rv $USER:$USER ~/PWD
sudo chown -Rv $USER:$USER ~/Z

sudo chmod 700 ~/.ssh
```
### 4.6 Setup aur access with `yay`

```bash
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
cd ..
rm -rf yay
```
### 4.7 Setup snapper

- Install snapper and related packages
```bash
sudo pacman -S --needed snapper inotify-tools plocate smartmontools dialog

yay -S snapper-rollback --needed
``` 
- Configure snapper
```bash

sudo umount /.snapshots
sudo rm -rf /.snapshots
sudo snapper -c root create-config /
sudo snapper list-configs
sudo btrfs subvolume delete /.snapshots
sudo rm -r /.snapshots
sudo mkdir /.snapshots
sudo mount -a

sudo snapper -c home create-config /home
sudo snapper list-configs
sudo snapper -c root set-config ALLOW_USERS="$USER" SYNC_ACL=yes
sudo snapper -c home set-config ALLOW_USERS="$USER" SYNC_ACL=yes

sudo chown :right9zzz /.snapshots
sudo chmod 750 /.snapshots

```
- Add `.snapshots` to PRUNENAMES in `updatedb.conf`
```bash
sudo sed -i '/^PRUNENAMES[[:space:]]*=/ { s/"\([^"]*\)"/"\1 .snapshots"/; s/'\''\([^'\'']*\)'\'\''/'\''\1 .snapshots'\''/; }; $a PRUNENAMES=".snapshots"' /etc/updatedb.conf
```
- Disable snapper services
```bash
sudo systemctl disable --now snapper-timeline.timer snapper-cleanup.timer
```
- Add `grub-btrfs-overlayfs` to HOOKS in `mkinitcpio.conf`
```bash
sudo sed -i '/^HOOKS=/ s/)$/ grub-btrfs-overlayfs)/' /etc/mkinitcpio.conf
```
- Regenrate initramfs
```bash
sudo mkinitcpio -P
```
- Create pacman hook that update grub on kernel modification or grub-update
```bash
sudo mkdir -p /etc/pacman.d/hooks && \
cat << 'EOF' | sudo tee /etc/pacman.d/hooks/zz-grub-update.hook
[Trigger]
Operation = Install
Operation = Upgrade
Operation = Remove
Type = Path
Target = usr/lib/modules/*/vmlinuz
Target = boot/vmlinuz-*

[Action]
Description = Updating GRUB configuration after kernel changes...
When = PostTransaction
Exec = /usr/bin/grub-mkconfig -o /boot/grub/grub.cfg
EOF

cat << 'EOF' | sudo tee /etc/pacman.d/hooks/zz-grub-install.hook
[Trigger]
Operation = Upgrade
Type = Package
Target = grub

[Action]
Description = Re-installing GRUB and updating config...
When = PostTransaction
Exec = /bin/sh -c "/usr/bin/grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=right9xOS --recheck && /usr/bin/grub-mkconfig -o /boot/grub/grub.cfg"
EOF
```
- Enable services
```bash
sudo systemctl enable --now grub-btrfsd.service
sudo systemctl enable --now snapper-timeline.timer snapper-cleanup.timer
```
- Install `snap-pac` for snapshots on installing programs
```bash
sudo pacman -S snap-pac
```
- List or delete any irrelevant snapshots
```bash
sudo snapper -c root list
sudo snapper -c root delete 1
```
- Create grub entries
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
- Take base system snapshots
```bash
sudo snapper -c root create --description "Base System" --cleanup-algorithm empty
sudo snapper -c home create --description "Base System" --cleanup-algorithm empty
```

## 5. Set up user applications
- Remove `snap-pac ` 
```bash
sudo pacman -Rsu snap-pac
```
- Install user applications `2967.62 MiB`
```bash
# 1. Browsers & Communication
yay -S zen-browser-bin chromium ayugram-desktop localsend okular --needed

# 2. Productivity & Development
yay -S obsidian code ghostty tesseract tesseract-data-hin tesseract-data-eng --needed

# 3. Media & Entertainment
yay -S mpv yt-dlp qbittorrent nicotine+ ab-download-manager lutris-git protonplus surge --needed

# 4. NVIDIA & Graphics
yay -S libva-nvidia-driver cuda nvidia-prime gpu-screen-recorder obs-studio --needed

# 5. System Utilities & Monitoring
yay -S jq btop nethogs mission-center jp2a lsd ripgrep unrar zip downgrade bindfs tldr wtype sanskrit-fonts noto-fonts noto-fonts-extra bc noto-fonts-emoji --needed

# 6. Mirror Management & Btrfs
yay -S rate-mirrors cachyos-rate-mirrors btrfs-assistant --needed

# 7. Specialized Tools & Virtualization
yay -S proton-pass-bin scrcpy waydroid apphub-git antigravity --needed
```
- Setup `pipewire-jack` for `jack` if needed
```bash
sudo pacman -S pipewire-jack lib32-pipewire-jack
```
- Install `snap-pac` again
```bash
sudo pacman -S snap-pac
```
- - List or delete any irrelevant snapshots after the first onne
```bash
sudo snapper -c root list
sudo snapper -c root delete --sync 2-
```
- Take base + apps snapshots
```bash
sudo snapper -c root create --description "Base System + Apps" --cleanup-algorithm empty
sudo snapper -c home create --description "Base System + Apps" --cleanup-algorithm empty
```

## 6. Install HyDE
- Setup rustup to stable
```bash
rustup default stable
```
- Copy HyDE repo
```
sudo pacman -S --needed git base-devel
git clone --depth 1 https://github.com/HyDE-Project/HyDE ~/HyDE
cd ~/HyDE/Scripts
```
- Uncomment  required packages in `pkg_extra.lst`
```bash
sed -i \
  -e 's/^[[:space:]]*#[[:space:]]*libinput-gestures[[:space:]]*/libinput-gestures /' \
  -e 's/^[[:space:]]*#[[:space:]]*gestures[[:space:]]*/gestures /' \
  -e 's/^[[:space:]]*#[[:space:]]*power-profiles-daemon[[:space:]]*/power-profiles-daemon /' \
  -e 's/^[[:space:]]*#[[:space:]]*emote[[:space:]]*/emote /' \
  -e 's/^[[:space:]]*#[[:space:]]*gamescope[[:space:]]*/gamescope /' \
  -e 's/^[[:space:]]*#[[:space:]]*electron[[:space:]]*/electron /' \
  -e 's/^[[:space:]]*#[[:space:]]*swaylock-effects-git[[:space:]]*/swaylock-effects-git /' \
  ~/HyDE/Scripts/pkg_extra.lst
```
- Comment  the non required packages in `pkg_extra.lst`
```bash
sed -i \
  -e 's/^[[:space:]]*wf-recorder[[:space:]]*/# wf-recorder /' \
  -e 's/^[[:space:]]*steam[[:space:]]*/# steam /' \
  ~/HyDE/Scripts/pkg_extra.lst
  
sed -i \
  -e 's/^[[:space:]]*firefox[[:space:]]*/# firefox /' \
  ~/HyDE/Scripts/pkg_core.lst  
```
- Start HyDE installation skipping `nvidia` flags
```bash
./install.sh -irsn pkg_extra.lst
# select 2nd option for both grub-pochita and sddm-corners
```

## 7. Setup HyDE configurations
### 7.1 SSH into laptop
- Check IP
```bash
ip a
```
- SSH into laptop from Termux 
```bash
ssh right9zzz@ip
``` 
### 7.2 Setup `tmux`session
- Start tmux session on Laptop
```bash
tmux new -s in
```
- Attach session on Termux
```bash
tmux attach -t in
```
### 7.3 Run install again after reboot to fix errors

```bash
./install.sh -irsn pkg_extra.lst
# select 2nd option for both grub-pochita and sddm-corners
```
### 7.4 Setup zsh for new aliases

```zsh
# Add the loader to .zshrc
echo '[[ -f ~/.bash_aliases ]] && . ~/.bash_aliases' >> ~/.zshrc

# Refresh Zsh configuration
source ~/.zshrc
```
### 7.5 Fix electron apps issue

```bash
mv ~/.config/electron-flags.conf ~/.config/electon-flags.conf.bak
```
### 7.6 Setup screen recording script

```bash
mkdir -p ~/.config/hypr/scripts && cat << 'EOF' > ~/.config/hypr/scripts/recorder.sh && chmod +x ~/.config/hypr/scripts/recorder.sh
#!/bin/bash

# 1. Stop if already recording
if pgrep -x "gpu-screen-recorder" > /dev/null; then
    killall -s SIGINT gpu-screen-recorder
    notify-send "Recording" "Stopped and Saved" -i video-display
    exit 0
fi

# 2. Hardcoded Neutral Grey Glass Style
ROFI_STYLE="
* {
    background-color:            transparent;
    text-color:                  #FFFFFF;
    margin:                      0px;
    padding:                     0px;
}
window {
    width:                       300px;
    border:                      1px;
    border-radius:               12px;
    border-color:                #ffffff22;
    background-color:            #1a1a1ae6;
}
mainbox {
    padding:                     15px;
    children:                    [ \"inputbar\", \"listview\" ];
}
inputbar {
    padding:                     8px;
    margin:                      0px 0px 10px 0px;
    children:                    [ \"prompt\", \"entry\" ];
    border:                      0px 0px 1px 0px;
    border-color:                #ffffff11;
}
prompt {
    text-color:                  #ffffff88;
    margin:                      0px 5px 0px 0px;
}
entry {
    placeholder:                 \"Search...\";
    placeholder-color:           #ffffff44;
}
listview {
    lines:                       7;
    fixed-height:                true;
    scrollbar:                   false;
    spacing:                     4px;
}
element {
    padding:                     8px 12px;
    border-radius:               8px;
}
element normal.normal, element alternate.normal {
    background-color:            transparent;
    text-color:                  #FFFFFF;
}
element selected.normal {
    background-color:            #ffffff22;
    text-color:                  #FFFFFF;
    border:                      1px;
    border-color:                #ffffff44;
}
element-text {
    text-color:                  inherit;
    vertical-align:              0.5;
}
"

# 3. Options (30fps at the bottom)
options="60fps_Audio\n60fps_Silent\n90fps_Audio\n90fps_Silent\n120fps_Silent\n30fps_Audio\n30fps_Silent"

# 4. Launch Rofi
choice=$(echo -e "$options" | rofi -dmenu -i -p "󰑋 " -config /dev/null -theme-str "$ROFI_STYLE")

[ -z "$choice" ] && exit

# 5. Build High Quality Command (H264 Ultra)
fps=$(echo "$choice" | sed 's/fps.*//')
audio_check=$(echo "$choice" | grep -i "Audio")

cmd="gpu-screen-recorder -w portal -f $fps -k h264 -encoder gpu -q ultra -fm cfr -c mp4"

if [ -n "$audio_check" ]; then
    cmd="$cmd -a $(pactl get-default-sink).monitor"
fi

# 6. Execute Instantly (No notification, no sleep)
$cmd -o ~/Videos/rec_$(date +%Y%m%d_%H-%M-%S).mp4
EOF
```
### 7.7 Setup `userperfs.conf`
- Fix SKIKO issue in  ` ~/.config/hypr/userprefs.conf ` 
```bash
micro ~/.config/hypr/userprefs.conf `
```
- Add Following configurations
```ini
# █░█ █▀ █▀▀ █▀█   █▀█ █▀█ █▀▀ █▀▀ █▀
# █▄█ ▄█ ██▄ █▀▄   █▀▀ █▀▄ ██▄ █▀░ ▄█

# Set your personal hyprland configuration here
# See https://wiki.hyprland.org/Configuring for more information

# █▀▀ █▀▀ █▄░█ █▀▀ █▀█ ▄▀█ █░░
# █▄█ ██▄ █░▀█ ██▄ █▀▄ █▀█ █▄▄


# █▀   █▀█   █ █   █▀▄   █▀▀   █▀▀
# ▄█   █▄█   █▄█   █▀▄   █▄▄   ██▄
# --- Source ---
source = ./nvidia.conf



# ▄▀█ █░█ ▀█▀ █▀█ █▀ ▀█▀ ▄▀█ █▀█ ▀█▀
# █▀█ █▄█ ░█░ █▄█ ▄█ ░█░ █▀█ █▀▄ ░█░
# --- Autostart ---

# Autostart Fcitx5 daemon
exec-once = fcitx5 -d
exec-once = tailscale systray
exec-once = surge server

# █▀▄▀█ █▀█ █▄░█ █ ▀█▀ █▀█ █▀█ █▀
# █░▀░█ █▄█ █░▀█ █ ░█░ █▄█ █▀▄ ▄█
monitor=eDP-1,1920x1080@120.21,auto,1

# █▀▀ █▄░█ █░█   █░█ ▄▀█ █▀█
# ██▄ █░▀█ ▀▄▀   ▀▄▀ █▀█ █▀▄
# --- Environment Variables ---

# Fix for ABDownloadManager (Forces software rendering for Kotlin/Skiko)
env = SKIKO_RENDER_API,SOFTWARE

# Environment variables for Input Method
env = QT_IM_MODULE,fcitx
env = XMODIFIERS,@im=fcitx
env = SDL_IM_MODULE,fcitx
env = GLFW_IM_MODULE,ibus
env = ELECTRON_OZONE_PLATFORM_HINT,wayland


# █▄▀ █▀▀ █▄█ █▄▄ █ █▄░█ █▀▄ █ █▄░█ █▀▀ █▀
# █░█ ██▄ ░█░ █▄█ █ █░▀█ █▄▀ █ █░▀█ █▄█ ▄█
# --- Keybindings ---

# --- Unbinds ---
unbind = $mainMod, B,
unbind = $mainMod, Z,
unbind = $mainMod, X, 
unbind = $mainMod, W, 
unbind = $mainMod, P, 
unbind = $mainMod Shift, P, 
unbind = $mainMod Control, P, 
unbind = $mainMod Alt, P, 
unbind = $mainMod Shift, P, 
unbind = , Print,
unbind = $mainMod Alt, Print
unbind = $mainMod Control, Print
unbind = ,XF86Calculator,
unbind = $mainMod, C,
unbind = $mainMod Shift, F,
unbind = $mainMod, F,
unbind = $mainMod, TAB,
unbind = $mainMod, E,


# Browser -------------------------------------------------------------- 
bindd = $mainMod, E, $d file explorer , exec, dolphin
bindd = $mainMod, Z, $d web browser , exec, zen-browser --new-window 
bindd = $mainMod Alt, Z, $d chromium web browser , exec, chromium --new-window 
bindd = $mainMod, X, $d ghostty , exec, ghostty
bindd = $mainMod, SPACE, $d obsidian , exec, obsidian
bindd = $mainMod, C, $d text editor , exec, code
bindd = $mainMod, N, $d telegram , exec, AyuGram
bindd = $mainMod, D, Toggle Scrcpy, exec, sh -c "pgrep -x scrcpy && pkill scrcpy || scrcpy --serial=5hy9s4uwpvha7pyl --no-audio"
bindd = $mainMod Shift, D, Launch Surge TUI, exec, ghostty --class="surge-tui" -e surge connect localhost:1700
bindd = $mainMod Alt, E, Launch Surge TUI, exec, ghostty --class="yazi" -e yazi
bindd = $mainMod Ctrl, SPACE,Launch Handy transcription, exec, pkill -USR2 handy


bindd = $mainMod SHIFT, Q, $d Quit Active Window, exec, hyprctl activewindow | grep pid | tr -d 'pid:' | xargs kill # Quit active window and all open instances
bindd = $mainMod, W, $d Toggle floating, exec, bash -c 'mw=$(hyprctl -j monitors | jq -r ".[] | select(.focused==true) | .width"); mh=$(hyprctl -j monitors | jq -r ".[] | select(.focused==true) | .height"); hyprctl dispatch togglefloating && hyprctl dispatch resizeactive exact $((mw/2)) $((mh/2)) && hyprctl dispatch movewindowpixel "exact $((mw/4)) $((mh*44/1000)),address:$(hyprctl activewindow -j | jq -r .address)"'
bindd = $mainMod, P, $d toggle pin on focused window, exec, hyde-shell windowpin
bindd = $mainMod Shift, P, $d color picker, exec, hyprpicker -an # Pick color (Hex) >> clipboard#

# Opacity Control
bindd = $mainMod Alt, bracketright, $d increase window opacity, exec, ~/.config/hypr/scripts/opacity.sh up
bindd = $mainMod Alt, bracketleft, $d decrease window opacity, exec, ~/.config/hypr/scripts/opacity.sh down
bindd = $mainMod Alt, backslash, $d reset window opacity, exec, ~/.config/hypr/scripts/opacity.sh reset

# Visual Control - Blur/Xray Toggles (Super+Ctrl)
bindd = $mainMod Ctrl, bracketleft, $d toggle blur, exec, ~/.config/hypr/scripts/visuals.sh noblur toggle
bindd = $mainMod Ctrl, bracketright, $d toggle xray, exec, ~/.config/hypr/scripts/visuals.sh xray toggle
bindd = $mainMod Ctrl, backslash, $d reset visuals, exec, ~/.config/hypr/scripts/visuals.sh reset_all

# Visual Control - Rounding (Super+Shift)
bindd = $mainMod Shift, bracketright, $d increase rounding, exec, ~/.config/hypr/scripts/visuals.sh rounding up
bindd = $mainMod Shift, bracketleft, $d decrease rounding, exec, ~/.config/hypr/scripts/visuals.sh rounding down
bindd = $mainMod Shift, backslash, $d reset rounding, exec, ~/.config/hypr/scripts/visuals.sh rounding reset


# Toggle HUD (Super + Ctrl + Alt + G)
bindd = $mainMod CTRL ALT, G, Toggle Waydroid, exec, bash ~/.config/hypr/scripts/toggle_waydroid.sh

# Toggle Fullscreen (Super + Ctrl + Alt + Shift + G)
bindd = $mainMod CTRL ALT SHIFT, G, Toggle Waydroid Fullscreen, exec, bash ~/.config/hypr/scripts/toggle_waydroid.sh fullscreen

 
# Move/Resize focused window
$d=[$wm|Move & Resize]
binddm = $mainMod Shift, Z, $d hold to move window , movewindow 
binddm = $mainMod Shift, X, $d hold to resize window, resizewindow

bindd = $mainMod, F,$d hold to move window, fullscreen, 0                                                           # Set active window to fullscreen
bindd = $mainMod Shift, F,$d hold to move window, fullscreen, 1                                                           # Maximize Window

bindd = $mainMod Shift, TAB, $d window switcher , exec, pkill -x rofi || $rofi-launch w
bind = $mainMod, Tab, focuscurrentorlast, # switch between last two windows

$d=[$ut|Screen Capture]
bindd = , Print, $d snip screen to clipboard, exec, bash -c 'file=/tmp/screenshot_$(date "+%Y-%m-%d_%H-%M-%S").png && grim -g "$(slurp)" "$file" && wl-copy < "$file" && hyprctl notify 2 2000 "rgb(00ff00)" "Copied to clipboard"'
bindd = $mainMod, Print, $d OCR selected area to clipboard, exec, grim -g "$(slurp)" - | tesseract stdin stdout | wl-copy && hyprctl notify 2 2000 "rgb(00ff00)" "Text copied to clipboard"
bindd = $mainMod Alt, Print, $d freeze and snip screen, exec, hyde-shell screenshot sf # partial screenshot capture (frozen screen)
bindd = $mainMod Shift, Print, $d print monitor , exec, hyde-shell screenshot m # monitor screenshot capture

bindd = , XF86Calculator, $d snip screen to clipboard, exec, bash -c 'file=/tmp/screenshot_$(date "+%Y-%m-%d_%H-%M-%S").png && grim -g "$(slurp)" "$file" && wl-copy < "$file" && hyprctl notify 2 2000 "rgb(00ff00)" "Copied to clipboard"'
bindd = $mainMod, XF86Calculator, $d OCR selected area to clipboard, exec, grim -g "$(slurp)" - | tesseract stdin stdout | wl-copy && hyprctl notify 2 2000 "rgb(00ff00)" "Text copied to clipboard"
bindd = $mainMod Alt, XF86Calculator, $d freeze and snip screen, exec, hyde-shell screenshot sf # partial screenshot capture (frozen screen)
bindd = $mainMod Shift, XF86Calculator, $d print monitor , exec, hyde-shell screenshot m # monitor screenshot capture



# ----------------------------------------------------- 
# SCREEN RECORDING WORKFLOW
# ----------------------------------------------------- 

# 1. Quick Record WITH Audio (60fps, HEVC)
# This starts recording immediately. Use Alt+R to stop.
bind = $mainMod, R, exec, notify-send "Recording" "Started with Audio (60fps)" && gpu-screen-recorder -w portal -f 60 -k hevc -encoder gpu -q high -a "$(pactl get-default-sink).monitor" -o ~/Videos/rec_audio_$(date +%H-%M-%S).mp4

# 2. Quick Record SILENT (60fps, HEVC)
bind = $mainMod CTRL, R, exec, notify-send "Recording" "Started Silent (60fps)" && gpu-screen-recorder -w portal -f 60 -k hevc -encoder gpu -q high -o ~/Videos/rec_silent_$(date +%H-%M-%S).mp4

# 3. Advanced Menu (90fps/120fps options)
# This opens the Rofi script we created earlier.
bind = SUPER_SHIFT_CTRL_ALT, R, exec, ~/.config/hypr/scripts/recorder.sh

# 4. STOP any active recording (Universal Stop)
bind = $mainMod ALT, R, exec, bash -c 'killall -s SIGINT gpu-screen-recorder && notify-send "Recording" "Stopped and Saved"'


# // █ █▄░█ █▀█ █░█ ▀█▀
# // █ █░▀█ █▀▀ █▄█ ░█░
# --- Custom App Fixes ---

#  Uncomment to enable // change to a preferred value
# 🔗 See https://wiki.hyprland.org/Configuring/Variables/#input
input {
    # kb_layout = us
    # follow_mouse = 1
    # sensitivity = 0
    # force_no_accel = 0
    # accel_profile = flat 
    # numlock_by_default = true

    # 🔗 See https://wiki.hyprland.org/Configuring/Variables/#touchpad
    touchpad {
        natural_scroll = no
    }

}



# 🔗 See https://wiki .hyprland.org/Configuring/Variables/#gestures
gestures {
#     workspace_swipe = true
#     workspace_swipe_fingers = 3
}

# for window shallow similar to devour
misc {
    # enable_swallow = true
    # swallow_regex = (foot|kitty|allacritty|Alacritty|ghostty|Ghostty|org.wezfurlong.wezterm)
}

# Don't show update on first launch
ecosystem {
  # no_update_news = true
}


# █░█░█ █ █▄░█ █▀▄ █▀█ █░█░█   █▀█ █░█ █░░ █▀▀ █▀
# ▀▄▀▄▀ █ █░▀█ █▄▀ █▄█ ▀▄▀▄▀   █▀▄ █▄█ █▄▄ ██▄ ▄█

# See https://wiki.hyprland.org/Configuring/Window-Rules/
layerrule = ignore_alpha 0.5,match:namespace waybar
windowrule = no_blur on, opacity 0 override, match:tag transparent
bind = $mainMod Alt, slash, tagwindow, transparent
bind = $mainMod Alt, X, tagwindow, transparent
windowrule = opaque on, match:tag opaque
bind = $mainMod Alt, period, tagwindow, opaque
bind = $mainMod Alt, period, exec, hyprctl reload

# windowrule = opacity 0.90 0.90,match:class ^(code-oss)$ # code-oss
windowrule = opacity 0.90 0.80,match:class ^(chromium)$ # AyuGram
windowrule = opacity 0.70 0.80,match:class ^(obsidian)$ # obsidian
windowrule = opacity 0.90 0.80,match:class ^(com.ayugram.desktop)$ # AyuGram
windowrule = opacity 0.90 $& 0.80 $& 1,match:class ^(ghostty)$
windowrule = opacity 0.90 $& 0.70 $& 1,match:class ^(zen)$
windowrule = opacity 0.8 0.5 override, match:class scrcpy, match:title M2103K19PI, # no_focus on
# windowrule = pin true, float true, match:class scrcpy, match:title M2103K19PI
# windowrule = move 1798 418, match:class scrcpy, match:title M2103K19PI
# windowrule = size 120 248, match:class scrcpy, match:title M2103K19PI
# windowrule = no_shadow on,  match:class scrcpy, match:title M2103K19PI
# Dynamic scrcpy window rules - adapts to any monitor resolution
windowrule = match:class scrcpy, match:title M2103K19PI, float 1
windowrule = match:class scrcpy, match:title M2103K19PI, pin 1
windowrule = match:class scrcpy, match:title M2103K19PI, size (monitor_w*0.0625) (monitor_h*0.2296)
windowrule = match:class scrcpy, match:title M2103K19PI, move (monitor_w-120-13) (monitor_h-248-15)
windowrule = match:class scrcpy, match:title M2103K19PI, no_shadow 1


# █▀▄▀█ █ █▀ █▀▀
# █░▀░█ █ ▄█ █▄▄


# 🔗 See https://wiki .hyprland.org/Configuring/Variables/#gestures
gestures {
#     workspace_swipe = true
#     workspace_swipe_fingers = 3
}

# for window shallow similar to devour
misc {
    # enable_swallow = true
    # swallow_regex = (foot|kitty|allacritty|Alacritty|ghostty|Ghostty|org.wezfurlong.wezterm)
}

# Don't show update on first launch
ecosystem {
  # no_update_news = true
}


# Dynamic Waydroid Rules
source = ~/.config/hypr/waydroid.conf

# Dynamic Opacity Configuration (Must be at end)
source = ~/.config/hypr/opacity.conf
source = ~/.config/hypr/visuals.conf

# Auto-Reset Dynamic Visuals on Startup (Clean Slate)
exec-once = echo "" > ~/.config/hypr/opacity.conf && echo "" > ~/.config/hypr/visuals.conf
아직은 외부가
```

### 7.8 Install additional themes on Hyde-shell
```bash
hydectl theme import # get Solarized Dark Theme
```

```bash
hydectl theme import --name "Amethyst-Aura" --url "https://github.com/jackpawlik26/Amethyst-Aura"
```

### 7.10 Set Collapsed system tray in waybar
```zsh
# 1. Create the module in the SOURCE directory
mkdir -p ~/.local/share/waybar/modules/ && \
cat <<EOF > ~/.local/share/waybar/modules/group-hide-tray.jsonc
{
    "group/hide-tray": {
        "orientation": "inherit",
        "drawer": {
            "transition-duration": "0.5",
            "children-class": "hide-tray-drawer",
            "transition-left-to-right": true,
            "click-to-reveal": false
        },
        "modules": [ "custom/dropdown#arrow", "tray" ]
    },
    "custom/dropdown#arrow": {
        "format": " <span size='large'>\uef8f </span>",
        "tooltip": true,
        "tooltip-format": "Reveal tray"
    }
}
EOF
# 2. Update all SOURCE layouts
find ~/.local/share/waybar/layouts/ -name "*.jsonc" -exec sed -i 's/"tray"/"group\/hide-tray"/g' {} +
# 3. Ensure CSS is present in the user config (Global override)
(grep -q "Universal Tray Drawer v2" ~/.config/waybar/user-style.css 2>/dev/null || cat <<EOF >> ~/.config/waybar/user-style.css
/* --- Universal Tray Drawer v2 --- */
/* 1. Default Behavior (Standard Layouts like MacOS) */
.hide-tray-drawer {
    margin-right: 10px;
    padding: 0 5px;
}
#custom-dropdown-arrow {
    padding-left: 10px;
    padding-right: 10px;
}
/* 2. Fix for Nested Groups (Pill Layouts) */
#pill #group-hide-tray,
#pill-right #group-hide-tray,
#pill-right1 #group-hide-tray,
#pill-right2 #group-hide-tray,
#pill-right3 #group-hide-tray,
#pill-left #group-hide-tray,
#pill-left1 #group-hide-tray,
#pill-left2 #group-hide-tray,
#pill-center #group-hide-tray,
#pill-center1 #group-hide-tray,
#pill-center2 #group-hide-tray {
    margin: 0;
    padding: 0;
}
#pill #group-hide-tray > .hide-tray-drawer,
#pill-right #group-hide-tray > .hide-tray-drawer,
#pill-right1 #group-hide-tray > .hide-tray-drawer,
#pill-right2 #group-hide-tray > .hide-tray-drawer,
#pill-right3 #group-hide-tray > .hide-tray-drawer,
#pill-left #group-hide-tray > .hide-tray-drawer,
#pill-left1 #group-hide-tray > .hide-tray-drawer,
#pill-left2 #group-hide-tray > .hide-tray-drawer,
#pill-center #group-hide-tray > .hide-tray-drawer,
#pill-center1 #group-hide-tray > .hide-tray-drawer,
#pill-center2 #group-hide-tray > .hide-tray-drawer {
   margin: 0;
   padding: 0;
}
EOF
)
# 4. Trigger a reload (Layout switch will now respect the new files)
~/.local/lib/hyde/waybar.py --reload
```

### 7.11 Setup shadows and transparency for waybar
```bash
cat >> ~/.config/waybar/user-style.css << 'EOF'

window#waybar {
    background: transparent;
}


/* Add shadows to module groups for contrast */
#leaf,
#leaf-inverse,
#leaf-up,
#leaf-down,
#leaf-right,
#leaf-left,
#pill,
#pill-right,
#pill-left,
#pill-down,
#pill-up,
#pill-in,
#pill-out {
    box-shadow: 1px 1px 3px rgba(0, 0, 0, 0.5);
}
EOF
```
### 7.12 Setup visual toggle scripts
```bash
# 1. Create Directories
mkdir -p ~/.config/hypr/scripts/

# 2. Write opacity.sh
cat << 'EOF' > ~/.config/hypr/scripts/opacity.sh
#!/bin/bash

# Configuration
OPACITY_CONF="$HOME/.config/hypr/opacity.conf"
OPACITY_STEP=0.05

# Get system default active opacity (usually 0.9 or 1.0)
DEFAULT_FLOAT=$(hyprctl getoption decoration:active_opacity -j | jq '.float')
# Format to remove trailing zeros (e.g. 0.900000 -> 0.9)
DEFAULT_VAL=$(printf "%.2f" $DEFAULT_FLOAT | awk '{print +$0}')

# Get active window info
WINDOW_INFO=$(hyprctl activewindow -j)
WINDOW_ADDR=$(echo "$WINDOW_INFO" | jq -r '.address')
TAG_NAME="opacity_${WINDOW_ADDR}"

# Ensure window is tagged
HAS_TAG=$(echo "$WINDOW_INFO" | jq -r ".tags" | grep "$TAG_NAME")
if [ -z "$HAS_TAG" ]; then
    hyprctl dispatch tagwindow "$TAG_NAME"
fi

# Find existing opacity in the config file
SAFE_TAG=$(echo "$TAG_NAME" | sed 's/[.[\*^$]/\\&/g')
CURRENT_LINE=$(grep "match:tag $SAFE_TAG" "$OPACITY_CONF")

if [ -n "$CURRENT_LINE" ]; then
    # Extract current active opacity from dynamic rules
    CURRENT_OPACITY=$(echo "$CURRENT_LINE" | sed -n 's/.*opacity \([0-9.]*\).*/\1/p')
else
    # If no dynamic rule, check if there's a specific static rule in userprefs.conf
    # Get window class
    WINDOW_CLASS=$(echo "$WINDOW_INFO" | jq -r '.class')
    
    # Search for a rule matching this class in userprefs.conf
    USER_PREFS="$HOME/.config/hypr/userprefs.conf"
    STATIC_RULE=$(grep "match:class.*$WINDOW_CLASS" "$USER_PREFS" | tail -n 1)
    
    if [ -n "$STATIC_RULE" ]; then
        # Extract the first number after 'opacity'
        CURRENT_OPACITY=$(echo "$STATIC_RULE" | sed -n 's/.*opacity \([0-9.]*\).*/\1/p')
        # Check if it's a valid number
        if [[ ! "$CURRENT_OPACITY" =~ ^[0-9.]+$ ]]; then
             CURRENT_OPACITY=$DEFAULT_VAL
        fi
    else
        CURRENT_OPACITY=$DEFAULT_VAL
    fi
fi

# Function to remove rule
remove_rule() {
    if [ -n "$CURRENT_LINE" ]; then
        grep -v "match:tag $SAFE_TAG" "$OPACITY_CONF" > "${OPACITY_CONF}.tmp" && mv "${OPACITY_CONF}.tmp" "$OPACITY_CONF"
    fi
    HAS_TAG_NOW=$(hyprctl activewindow -j | jq -r ".tags" | grep "$TAG_NAME")
    if [ -n "$HAS_TAG_NOW" ]; then
        hyprctl dispatch tagwindow "$TAG_NAME"
    fi
}

case "$1" in
    up)
        # Bash bc calculation
        NEW_OPACITY=$(echo "$CURRENT_OPACITY + $OPACITY_STEP" | bc)
        # Cap at 1.0
        NEW_OPACITY=$(echo "if ($NEW_OPACITY > 1.0) 1.0 else $NEW_OPACITY" | bc)
        ;;
    down)
        NEW_OPACITY=$(echo "$CURRENT_OPACITY - $OPACITY_STEP" | bc)
        # Cap at 0.1
        NEW_OPACITY=$(echo "if ($NEW_OPACITY < 0.1) 0.1 else $NEW_OPACITY" | bc)
        ;;
    reset)
        remove_rule
        hyprctl reload
        hyprctl notify 1 3000 "rgb(ff1ea3)" "Opacity: Config Default"
        exit 0
        ;;
    *)
        echo "Usage: $0 {up|down|reset}"
        exit 1
        ;;
esac

# Formatted new opacity (e.g. .95 instead of .9500)
NEW_OPACITY=$(printf "%.2f" $NEW_OPACITY | awk '{print +$0}')

# Apply new rule (Active Opacity Only)
NEW_RULE="windowrule = opacity $NEW_OPACITY override,match:tag $TAG_NAME"

if [ -n "$CURRENT_LINE" ]; then
    sed "s|.*match:tag $SAFE_TAG.*|$NEW_RULE|" "$OPACITY_CONF" > "${OPACITY_CONF}.tmp" && mv "${OPACITY_CONF}.tmp" "$OPACITY_CONF"
else
    echo "$NEW_RULE" >> "$OPACITY_CONF"
fi

hyprctl reload
hyprctl notify 1 1500 "rgb(ff1ea3)" "Opacity: $NEW_OPACITY"
EOF

# 3. Write visuals.sh
cat << 'EOF' > ~/.config/hypr/scripts/visuals.sh
#!/bin/bash

# Configuration
CONF_FILE="$HOME/.config/hypr/visuals.conf"

# Steps
ROUNDING_STEP=5
BORDER_STEP=1

# Get active window info
WINDOW_INFO=$(hyprctl activewindow -j)
WINDOW_ADDR=$(echo "$WINDOW_INFO" | jq -r '.address')

# Helper to manage rules
update_rule() {
    local PROP=$1       # e.g. rounding, no_blur
    local VALUE=$2      # e.g. 10, 0, 1
    local TAG="${PROP}_${WINDOW_ADDR}"
    local SAFE_TAG=$(echo "$TAG" | sed 's/[.[\*^$]/\\&/g')
    
    # 1. Ensure Tag Exists
    HAS_TAG=$(echo "$WINDOW_INFO" | jq -r ".tags" | grep "$TAG")
    if [ -z "$HAS_TAG" ]; then
        hyprctl dispatch tagwindow "$TAG"
    fi
    
    # 2. Update File
    NEW_RULE="windowrule = $PROP $VALUE override,match:tag $TAG"
    if grep -q "match:tag $SAFE_TAG" "$CONF_FILE"; then
        # Update existing
        sed "s|.*match:tag $SAFE_TAG.*|$NEW_RULE|" "$CONF_FILE" > "${CONF_FILE}.tmp" && mv "${CONF_FILE}.tmp" "$CONF_FILE"
    else
        # Append new
        echo "$NEW_RULE" >> "$CONF_FILE"
    fi
    
    hyprctl reload
    hyprctl notify 1 1500 "rgb(ff1ea3)" "$3: $VALUE"
}

remove_rule() {
    local PROP=$1
    local TAG="${PROP}_${WINDOW_ADDR}"
    local SAFE_TAG=$(echo "$TAG" | sed 's/[.[\*^$]/\\&/g')
    
    # Remove from file
    grep -v "match:tag $SAFE_TAG" "$CONF_FILE" > "${CONF_FILE}.tmp" && mv "${CONF_FILE}.tmp" "$CONF_FILE"
    
    # Untag
    HAS_TAG=$(hyprctl activewindow -j | jq -r ".tags" | grep "$TAG")
    if [ -n "$HAS_TAG" ]; then
        hyprctl dispatch tagwindow "$TAG"
    fi
    
    hyprctl reload
    hyprctl notify 1 1500 "rgb(ff1ea3)" "$3: Reset"
}

get_current_int() {
    local PROP=$1
    local DEFAULT=$2
    local TAG="${PROP}_${WINDOW_ADDR}"
    local SAFE_TAG=$(echo "$TAG" | sed 's/[.[\*^$]/\\&/g')
    
    # Check dynamic rule first
    LINE=$(grep "match:tag $SAFE_TAG" "$CONF_FILE")
    if [ -n "$LINE" ]; then
        echo "$LINE" | sed -n "s/.*$PROP \([0-9]*\).*/\1/p"
        return
    fi
    
    # Check system default
    if [ "$PROP" == "rounding" ]; then
        hyprctl getoption decoration:rounding -j | jq '.int'
    else
        echo "$DEFAULT"
    fi
}

MODE=$1
ACTION=$2

case "$MODE" in
    rounding)
        CURRENT=$(get_current_int rounding 10)
        case "$ACTION" in
            up) NEW=$((CURRENT + ROUNDING_STEP)) ;;
            down) NEW=$((CURRENT - ROUNDING_STEP)); [ $NEW -lt 0 ] && NEW=0 ;;
            reset) remove_rule rounding "Rounding"; exit ;;
        esac
        update_rule rounding $NEW "Rounding"
        ;;
        
    noblur)
        case "$ACTION" in
            toggle)
                 TAG="noblur_${WINDOW_ADDR}"
                 SAFE_TAG=$(echo "$TAG" | sed 's/[.[\*^$]/\\&/g')
                 # Check for rule (no_blur rule means Blur is OFF)
                 if grep -q "match:tag $SAFE_TAG" "$CONF_FILE"; then
                     remove_rule no_blur "Blur"
                     hyprctl notify 1 1500 "rgb(ff1ea3)" "Blur: ON (Default)"
                 else
                     update_rule no_blur 1 "Blur" "OFF"
                     hyprctl notify 1 1500 "rgb(ff1ea3)" "Blur: OFF"
                 fi
                ;;
            reset)
                remove_rule no_blur "Blur"
                ;;
        esac
        ;;
        
    xray)
        case "$ACTION" in
            toggle)
                 TAG="xray_${WINDOW_ADDR}"
                 SAFE_TAG=$(echo "$TAG" | sed 's/[.[\*^$]/\\&/g')
                 if grep -q "match:tag $SAFE_TAG" "$CONF_FILE"; then
                     remove_rule xray "Xray"
                     hyprctl notify 1 1500 "rgb(ff1ea3)" "Xray: OFF (Default)"
                 else
                     update_rule xray 1 "Xray" "ON"
                     hyprctl notify 1 1500 "rgb(ff1ea3)" "Xray: ON"
                 fi
                ;;
            reset)
                 remove_rule xray "Xray"
                 ;;
        esac
        ;;
        
    reset_all)
        # Reset everything
        remove_rule no_blur "Blur"
        remove_rule xray "Xray"
        # Legacy cleanup
        grep -v "match:tag noblur_${WINDOW_ADDR}" "$CONF_FILE" > "${CONF_FILE}.tmp" && mv "${CONF_FILE}.tmp" "$CONF_FILE"
        hyprctl dispatch tagwindow "-noblur_${WINDOW_ADDR}"
        
        hyprctl notify 1 1500 "rgb(ff1ea3)" "Visuals: Reset"
        ;;
        
    *)
        echo "Usage: $0 {rounding} {up|down|reset} OR $0 {noblur|xray} {toggle|reset} OR $0 reset_all"
        exit 1
        ;;
esac
EOF

# 4. Make Executable
chmod +x ~/.config/hypr/scripts/opacity.sh ~/.config/hypr/scripts/visuals.sh

# 5. Initialize Config Files (Clear them)
touch ~/.config/hypr/opacity.conf ~/.config/hypr/visuals.conf
echo "" > ~/.config/hypr/opacity.conf
echo "" > ~/.config/hypr/visuals.conf

echo "Done! Scripts restored and executable. ✨"
```
### 7.13 Fix weather
```bash
cat <<EOF >> ~/.local/state/hyde/config
export WEATHER_LOCATION="Mussoorie"
export WEATHER_TEMPERATURE_UNIT="c"
export WEATHER_TIME_FORMAT="12h"
export WEATHER_WINDSPEED_UNIT="km/h"
EOF
```

### 7.14 Setup Hindi Input
- Download `fcxit` packages
```bash
sudo pacman -S fcitx5-im fcitx5-m17n fcitx5-configtool
```
- Check and add following variables in `userperfs.conf`
```ini
# Environment variables for Input Method
env = QT_IM_MODULE,fcitx
env = XMODIFIERS,@im=fcitx
env = SDL_IM_MODULE,fcitx
env = GLFW_IM_MODULE,ibus

# Autostart Fcitx5 daemon
exec-once = fcitx5 -d
```
- Configure with `Fcitx 5 Configuration` app.
### 7.15 Snapshot `Base HyDE`
- Delete any non required snapshots
```zsh
sudo snapper -c root list 
sudo snapper -c home list 
```

```
sudo snapper -c root delete 3-
```
- Take base HyDE snapshots
```zsh
sudo snapper -c root create --description "Base HyDE" --cleanup-algorithm empty
sudo snapper -c home create --description "Base HyDE" --cleanup-algorithm empty
```


## Misc

curl -fsSL https://christitus.com/linux | sh

### Setup spotify adblock
```bash
sudo pacman -S zip unzip curl
bash <(curl -sSL https://raw.githubusercontent.com/SpotX-Official/SpotX-Bash/main/spotx.sh)
```
### Set Up Waydroid
- cd to images folder
```bash
cd PWD
```
- Create the waydroid image directory if it doesn't exist:
```bash
sudo mkdir -p /usr/share/waydroid-extra/images/
```
- Copy your images:
```bash
sudo cp system.img /usr/share/waydroid-extra/images/
sudo cp vendor.img /usr/share/waydroid-extra/images/
```
- Re-initialize Waydroid -f flag to use custom images
```bash
sudo waydroid init -f
```

After that u can move images and edit `waydroid.cfg` to reflect the right path
```bash
sudo mv /usr/share/waydroid-extra/images/system.img /var/lib/waydroid/images/
sudo mv /usr/share/waydroid-extra/images/vendor.img /var/lib/waydroid/images/
sudo micro /var/lib/waydroid/waydroid.cfg
```

- Restart Waydroid Services
```bash
sudo systemctl restart waydroid-container
```
- Start a session and UI:`sudo ufw allow in on waydroid0`
```bash
waydroid session start &
waydroid show-full-ui
```
- Run waydroid script and install `gapps`, `libhoudini`(fir app support) and `smartdock`
```bash
cd ~/PWD
git clone https://github.com/casualsnek/waydroid_script
cd waydroid_script
python3 -m venv venv
venv/bin/pip install -r requirements.txt
sudo venv/bin/python3 main.py
```
- Disable `ufw` 
```bash
sudo ufw allow in on waydroid0
sudo ufw allow out on waydroid0

# Allow routing from Waydroid to the world
sudo ufw route allow in on waydroid0
sudo ufw route allow out on waydroid0
sudo ufw reload
```
- Find `android_id` and register at (https://www.google.com/android/uncertified/) to certification. (you can find it with waydroid-script too)
```bash
sudo waydroid shell 'sqlite3 /data/data/com.google.android.gsf/databases/gservices.db "select * from main where name = 'android_id';"'
```
#### My mpv setup
#### clone repo
```bash
git clone https://github.com/noelsimbolon/mpv-config ~/.config/mpv
```
Comment line  `gpu-api=vulkan...`
```
micro ~/.config/mpv/mpv.conf
```
https://iamscum.wordpress.com/guides/videoplayback-guide/mpv-conf/


Since you are using Btrfs Assistant, if you ever need to do a "Full Restore" again:
Perform the restore in the app.
Don't reboot immediately.
Run `sudo grub-mkconfig -o /boot/grub/grub.cfg` one time manually.


## 6. Snapper Rollback Setup (Arch Linux Native)
Arch Linux snapshots do not easily boot to GUI by default due to their Read-Only nature and the lack of a native `snapper rollback` implementation for flat layouts. Here are the 3 flawless fallback methods for Arch Linux:

### 6.1 Btrfs Layout Initialization Reminder
If you missed Step 4.7 during install, remember that Snapper naturally creates `.snapshots` as a nested subvolume inside `@`, which breaks flat layout rollbacks! To fix it after the fact:
```bash
# 1. Temporarily remove the fstab mount point
sudo umount /.snapshots
sudo rmdir /.snapshots

# 2. Let Snapper generate the config and nested subvolume
sudo snapper -c root create-config /

# 3. Delete the nested subvolume and replace with our top-level one
sudo btrfs subvolume delete /.snapshots
sudo mkdir /.snapshots
sudo mount -a  # Remounts your top-level @snapshots back to /.snapshots from fstab

# 4. Set permissions
sudo snapper -c root set-config ALLOW_USERS="right9zzz" SYNC_ACL=yes
```

### 6.2 Rollback Method 1: Btrfs Assistant (GUI)
If your live system boots into XFCE but has broken packages, the most native way is to use **Btrfs Assistant**.
1. Open the **Btrfs Assistant** Application.
2. Go to the **Snapper** tab.
3. Highlight the clean snapshot, and click **Restore**.
4. Reboot your system. It will perfectly swap `@` and rollback safely using your flat layout.

### 6.3 Rollback Method 2: `snapper-rollback` (AUR - CLI)
If your desktop environment crashes but your TTY works, use the AUR tool designed precisely for flat layouts:
1. **Install:** `aur/snapper-rollback`
2. **Configure:** Edit `/etc/snapper-rollback.conf` and set `dev = /dev/nvme0n1p2`, `subvol_snapshots = @snapshots`.
3. **Rollback:** Run `sudo snapper-rollback <NUMBER>` and reboot.

### 6.4 Rollback Method 3: The Manual Subvolume Swap (Ultimate Recovery)
If the system is totally dead (won't boot) or if you are deliberately booted inside a read-only broken snapshot from GRUB, run this raw Btrfs subvolume swap sequence:

```bash
# 1. Gain root and mount the topmost Btrfs root (subvolid=5)
sudo -s
mkdir -p /mnt
mount /dev/nvme0n1p2 -o subvolid=5 /mnt
cd /mnt

# 2. Swap out the broken system into a backup container
mv @ @.broken

# 3. Clone your clean snapshot to become the new live system
btrfs subvolume snapshot ./@snapshots/<CLEAN_SNAPSHOT_ID>/snapshot ./@

# 4. Reboot!
reboot

# 5. Clean up (Optional, once stable)
sudo mount /dev/nvme0n1p2 -o subvolid=5 /mnt
sudo btrfs subvolume delete /mnt/@.broken
```

```bash
chattr -VR +C ~/.local/share/gnome-boxes/images
```


### 6.5 Automate Wireless Scrcpy
1. Connect phone via usb and run the command to open the network port on the phone:
```bash
adb tcpip 5555
```

2. Create the executable script:
```bash
cat << 'EOF' > ~/.config/hypr/scripts/connect-phone.sh && chmod +x ~/.config/hypr/scripts/connect-phone.sh
#!/bin/bash
# --- CONFIG ---
IP="100.123.244.105"
LOCKFILE="/tmp/scrcpy_launcher.lock"
# --------------

# 1. IMMEDIATE CHECK: Look for the Lock File, not the process
if [ -f "$LOCKFILE" ]; then
    # Toggle logic: If already running, kill it and remove lock
    pkill scrcpy
    rm "$LOCKFILE"
    exit 0
fi

# 2. CREATE LOCK: Do this BEFORE adb connect
touch "$LOCKFILE"

# 3. CLEANUP: Delete lock when scrcpy eventually closes
# This ensures the lock is removed even if you close the window manually
trap 'rm -f "$LOCKFILE"' EXIT

# 4. STARTUP SEQUENCE
adb connect $IP:5555 > /dev/null 2>&1
scrcpy --serial "$IP:5555" --no-audio --video-bit-rate 2M --max-size 1024 --always-on-top
EOF
```

3. Add the keybind in `userperfs.conf` :

```ini
bindd = $mainMod Alt, D, Toggle Scrcpy, exec, ~/.config/hypr/scripts/connect-phone.sh
```


### PMKID


- 

```bash
yay -S aircrack-ng hcxtools hashcat seclists cuda rocm-opencl-runtime --needed
```

- 

```bash
yay -S airgeddon arping-th asleap beefproject-git bettercap bully ccze crunch curl dhcp dnsmasq ethtool ettercap hashcat hcxdumptool hcxtools hostapd hostapd-mana-git hostapd-wpe john iptables lighttpd mdk3 mdk4 nftables openssl pixiewps reaver rfkill sox systemd tcpdump usbutils wget wireshark-cli xorg-xdpyinfo xorg-xset xorg-xhost --needed
```

- 

```bash
xhost +SI:localuser:root
```

```bash
yay -S --needed --noconfirm \
wine winetricks goverlay protonplus \
nvidia-utils lib32-nvidia-utils nvidia-prime \
vulkan-icd-loader lib32-vulkan-icd-loader \
gnutls lib32-gnutls gtk3 lib32-gtk3 \
libpulse lib32-libpulse alsa-lib lib32-alsa-lib alsa-utils alsa-plugins lib32-alsa-plugins \
giflib lib32-giflib libpng lib32-libpng \
libldap lib32-libldap openal lib32-openal \
libxcomposite lib32-libxcomposite libxinerama lib32-libxinerama \
ncurses lib32-ncurses ocl-icd lib32-ocl-icd libva lib32-libva \
gst-plugins-base-libs lib32-gst-plugins-base-libs \
sdl2 lib32-sdl2 v4l-utils lib32-v4l-utils \
sqlite lib32-sqlite
```


## Highlight
