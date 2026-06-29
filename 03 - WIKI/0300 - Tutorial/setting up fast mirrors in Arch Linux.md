# Setting Up Fast Mirrors in Arch Linux

Setting up fast mirrors in Arch Linux can significantly improve download speeds for updates and package installations by connecting to servers closer to your location or with better performance. Below are three methods to achieve this using tools like Reflector and rate-mirrors, as well as a manual approach.
```
rate-mirrors --allow-root --protocol https arch | grep -v '^#' | sudo tee /etc/pacman.d/mirrorlist
```


## Backup Your Mirror List

Before making any changes, it's a good practice to back up your current mirror list to avoid potential issues. Run the following command:

```
sudo cp /etc/pacman.d/mirrorlist/etc/pacman.d/mirrorlist.bak
```

This creates a backup file that you can restore if needed.

## Method 1: Using Reflector

Reflector is a widely used tool in Arch Linux for automatically selecting the fastest mirrors based on various criteria like speed, location, or sync status. Here's how to set it up:

- **Install Reflector**: If it's not already installed, use the following command:
 ```
sudo pacman -S reflector
```

- **Select Mirrors by Location**: Filter mirrors by country to prioritize those closer to you. Replace `<country_name>` with your country (e.g., "India") and adjust the number of mirrors and protocol as needed:
 ```
 sudo reflector -c <country_name> -l 20 -p https --save /etc/pacman.d/mirrorlist
```
This sorts mirrors by download rate and saves the top 20 to your mirror list.

- **Automate with Reflector Service**: To update mirrors automatically on system startup, edit the Reflector configuration file at `/etc/xdg/reflector/reflector.conf` with your preferred settings (e.g., number of mirrors, protocol, sort method), and enable the service:
```
sudo systemctl enable reflector
``` 

Be cautious with frequent updates, as some users suggest running Reflector only weekly or monthly unless your network environment changes often.

## Method 2: Using rate-mirrors

rate-mirrors is another utility that tests mirror speeds and generates a list of the fastest, fully synced mirrors. It's particularly useful for Arch and derivative distributions like EndeavourOS.

- **Install rate-mirrors**: Install the tool using:
```
sudo pacman -S rate-mirrors 
```

Alternatively, if it's not in the official repositories, use an AUR helper like `yay`:

```
yay -S rate-mirrors-bin
```

- **Update Mirror List**: To test and update mirrors for Arch Linux, specify your starting country (if not in the US) and protocol:
 ```
 rate-mirrors --disable-comments-in-file --entry-country=<country_code> --protocol=https arch | sudo tee /etc/pacman.d/mirrorlist
```
Replace `<country_code>` with your country's two-letter code (e.g., "CA" for Canada). This command tests mirrors and writes the fastest ones to your mirror list.

- **Verify the List**: Check the updated mirror list to ensure it reflects the changes:
 ```
cat /etc/pacman.d/mirrorlist
```
