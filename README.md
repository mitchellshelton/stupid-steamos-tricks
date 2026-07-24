# Stupid SteamOS Tricks

- Problem
  - The file system is not writable
- Solution
  - sudo steamos-readonly disable

---

- Problem
  - Pacman doesn't work
- Solution
  - sudo pacman-key --init
  - sudo pacman-key --populate archlinux
  - sudo pacman-key --populate holo
  - sudo pacman -Scc --noconfirm

---

- Problem
  - Vital Synth won't run in flatpak Bitwig because it is looking for libcurl-gnutls.so.4
- Solution
  - Install libcurl-gnutls.so.4 with pacman (this might be optional...)
    - sudo pacman -S libcurl-gnutls
  - Run Bitwig outside of the flatpak to bypass sandboxing
    - /var/lib/flatpak/app/com.bitwig.BitwigStudio/current/active/files/bitwig-studio
    - You can edit you menu item and put this under "Program" from the KDE Menu Editor (also remove the Command-line arguments section)

---

- Problem
  - When Steam updates it deletes anything outside of "/home/deck". This means anything installed with pacman must be reinstalled.
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
#sudo pacman -S --noconfirm base-devel cmake git qt6 glibc linux-api-headers qt6-tools gdb clang

# Add server alias
# This adds a custom line to your hosts file for local development or servers
LINE="192.1.1.1   customserver.local"
grep -qF "$LINE" /etc/hosts || echo "$LINE" | sudo tee -a /etc/hosts
```

---
