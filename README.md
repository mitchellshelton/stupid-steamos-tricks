# Stupid SteamOS Tricks

I am very happy with SteamOS and the related hardware that Valve has released. Both the Steam Deck and Steam Machine are excellent gaming machines. When the Steam Machine was announced I decided to abandon macOS and Windows and make the permanent switch to Linux and SteamOS. I used the Steam Deck as my daily primary computer starting in November of 2025. In July I was finally able to purchase a Steam Machine and SteamOS continues to be my daily driver.

As much as I enjoy these machines and SteamOS I do not think that Linux is ready for the everyday computer user. I love Linux and have used it both professionally and personally for many years. Unfortunately, it remains nearly impossible to get anything useful out of the operating system without heavily using the command line. Despite the best efforts by the community, it is still a difficult operating system to make daily use of.

SteamOS does a lot to make this easier. KDE Plasma is a great graphical shell and provides a wealth of tools to make things easier. Even with that, the command line is still a necessary part of using SteamOS. Below is a series of commands and tips to help make the journey a little easier. SteamOS has some quirks of its own. Built on Arch Linux with some bizarre decisions from Valve, it brings an often even more difficult learning curve than something like Ubuntu. Good luck on your journey and please feel free to contribute to this document to help it grow. While this is mostly here for my personal benefit, I wanted to share it with others as the journey to gather and learn this information was often filled with frustration. Even with that frustration, at almost a year in as a full time Linux user, I can't see myself going back to macOS or Windows.

---

- Problem
  - You don't know the sudo password for the deck user
- Solution
  - A password must be manually set for the deck user
  ```bash
  passwd
  ```

---

- Problem
  - The file system is not writable
- Solution
  ```bash
  sudo steamos-readonly disable
  ```

---

- Problem
  - Pacman doesn't work
- Solution
  ```bash
  sudo pacman-key --init
  sudo pacman-key --populate archlinux
  sudo pacman-key --populate holo
  sudo pacman -Scc --noconfirm
  ```

---

- Problem
  - I am trying to install stuff with pacman but I'm out of disk space.
- Explanation
  - I don't have a work around for this one. The best I can recommend is that you try to use AppImages and Flatpak for most things. Install things to /home/deck whenever possible. Pacman packages utilize a very small partition on the main drive. This is only somewhat permanent storage as it is wiped with SteamOS updates. This is a known pain point and hopefully Valve will provide a solution in the future.

---

- Problem
  - Vital Synth won't run in flatpak Bitwig because it is looking for libcurl-gnutls.so.4
- Solution
  - Install libcurl-gnutls.so.4 with pacman (this might be optional...)
    ```bash
    sudo pacman -S libcurl-gnutls
    ```
  - Run Bitwig outside of flatpak to bypass sandboxing
    ```bash
    /var/lib/flatpak/app/com.bitwig.BitwigStudio/current/active/files/bitwig-studio
    ```
    - You can edit the menu item and put this under "Program" from the KDE Menu Editor (also remove the Command-line arguments section)

---

- Problem
  - When SteamOS updates it deletes anything outside of "/home/deck". This means anything installed with pacman must be reinstalled.
- Solution
  - Create a shell script that you can run after an update to restore your settings.

---

## Example Script
- Much of this is optional but should help give you ideas for things that you may need.

```bash
# Disable SteamOS read only mode
sudo steamos-readonly disable

# Initialize and setup pacman key to enable package installation
sudo pacman-key --init
sudo pacman-key --populate archlinux
sudo pacman-key --populate holo
sudo pacman -Scc --noconfirm
# Clear the pacman cache
sudo rm /usr/lib/holo/pacmandb/db.lck

# Downgrade wireplumber to fix audio in Bitwig
sudo pacman --noconfirm -U https://archive.archlinux.org/packages/l/libwireplumber/libwireplumber-0.5.10-1-x86_64.pkg.tar.zst https://archive.archlinux.org/packages/w/wireplumber/wireplumber-0.5.10-1-x86_64.pkg.tar.zst && systemctl --user restart wireplumber -y

# Sublime Text (instructions from sublime site)
curl -O https://download.sublimetext.com/sublimehq-pub.gpg && sudo pacman-key --add sublimehq-pub.gpg && sudo pacman-key --lsign-key 8A8F901A && rm sublimehq-pub.gpg
if ! grep -q "^\[sublime-text\]" /etc/pacman.conf; then
    echo -e "\n[sublime-text]\nServer = https://download.sublimetext.com/arch/stable/x86_64" | sudo tee -a /etc/pacman.conf
fi
sudo pacman -S --noconfirm sublime-text 

# Reinstall printer drivers 
# I keep a local copy of my printer drivers in my /home/deck/.config/custom-config/ folder, the same place I keep this script.
echo 'Printer location: http://192.168.1.35/ipp/print'
sudo cp -r /home/deck/.config/custom-config/brother/filter/* /lib/cups/filter/
sudo cp -r /home/deck/.config/custom-config/brother/share/* /usr/share/

# Check for crontab and install if missing
command -v crontab &> /dev/null || sudo pacman -S --noconfirm cronie

# Make sure the prevent speaker standby cron exists 
# I use KRK speakers and find this useful although my success rate with it has been limited.
/home/deck/.config/custom-config/prevent-speaker-standby/installCronTab.sh
sudo systemctl start cronie.service
sudo systemctl enable cronie.service

# prep basic C and C++ development environment
sudo pacman -S --noconfirm base-devel cmake git qt6 glibc linux-api-headers qt6-tools gdb clang

# Add server alias
# This adds a custom line to your hosts file for local development or servers
LINE="192.1.1.1   customserver.local"
grep -qF "$LINE" /etc/hosts || echo "$LINE" | sudo tee -a /etc/hosts
```

---
