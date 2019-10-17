# metr1xx’ random Arch Linux tips that may or may not help you / work.

# Table of contents

1. [Printing with an Epson Inkjet printer](#printing-with-an-epson-inkjet-printer)
2. [AMDGPU on Southern Island cards (HD7950/70/90(?))](#amdgpu-on-southern-island-cards)
3. [LightDM installation](#lightdm-installation)
4. [GRUB installation from an archiso stick](#grub-installation-from-an-archiso-stick)
5. [Polybar libjson error after updating](#polybar-libjson-error-after-updating)
6. [VIM airline font installation](#vim-airline-font-installation)
7. [Save diskspace when using Markdown](#save-diskspace-when-using-markdown)
8. [File Recovery on Linux](#file-recovery-on-linux)
9. [Teamspeak mutes Spotify and other audio sources](#teamspeak-mutes-other-audio-sources)
10. [US international layout without dead keys](#us-international-layout-without-dead-keys)
11. [Fix PulseAudio crackeling](#pulseaudio-crackeling)
12. [Eduroam on Void with NetworkManager and without a DE](#eduroam-on-void-with-networkmanager-and-without-a-de)

# Printing with an Epson Inkjet printer
- Install `cups` and `cups-pdf`
- CUPS installation better explained: http://metr1xx.de/archprinting.html
- Enable and start org.cups.cupsd.service
- `systemctl enable org.cups.cupsd`
- Install `epson-inkjet-printer-escpr` (AUR)

# AMDGPU on Southern Island cards
- works with HD7950, HD7970 and 7990(I don't really know but it should.)
- The whole thing better explained: http://metr1xx.de/archamdgpu.html
- Install those packages preferably from the official repositories:

  ` sudo pacman -S mesa lib32-mesa  xf86-video-amdgpu vulkan-radeon libva-vdpau-driver opencl-amd `

  if you can't find them in the official repositories install them from the AUR.
- Add `amdgpu` as the first module in the `#MODULES` array in the file `/etc/mkinitcpio.conf`
- Create a file called `noradeon.conf` in `/etc/modprobe.d/` with the content `blacklist radeon` or just run:
  `sudo echo "blacklist radeon" > /etc/modprobe.d/noradeon.conf `
- Rebuild the initramfs with `sudo mkinitcpio -p linux`
- Set the necessary kernel parameters in GRUB:
  - Open `/etc/default/grub`
  - At the line GRUB_CMDLINE_LINUX_DEFAULT, append the parameters
    `amdgpu.si_support=1 radeon.si_support=0`
    inside of the quotation marks to pass the necessary kernel parameters
- rebuild the grub config with 
  `grub-mkconfig -o /boot/grub/grub.cfg`
- reboot

# LightDM installation
- Install `lightdm` and `lightdm-gtk-greeter` packages
- Enable lightdm.service to start it at boot
  `sudo systemctl enable lightdm`
- Install `lightdm-gtk-greeter-settings` to configure lightdm

# GRUB installation from an archiso stick
make sure the root partition is mounted in /mnt of your usb stick
  (where sda2 is your ROOT partition)
  `mount /dev/sda2 /mnt`
- make sure the UEFI partition is mounted in /mnt/boot of your usb stick
  (where sda1 is your BOOT partition)
  `mount /dev/sda1 /mnt/boot`
- change your root directory from the stick to your arch installation
  `arch-chroot /mnt`
- install grub and efibootmgr
  `pacman -S grub efibootmgr`
- install grub as your bootloader with 
  `grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB-Bootloader`
- generate / update grub config with 
  `grub-mkconfig -o /boot/grub/grub.cfg`
- (Optional) install `grub-customizer` (AUR) and `arch-silence` (AUR) grub theme for a nice bootloader screen

# Polybar libjson error after updating

Have you updated polybar and now it's not working anymore? Running `polybar example &` returns a libjson error?

- rebuild `libjson` AUR

`git clone https://aur.archlinux.org/libjson.git libjson && cd libjson && makepkg -csri< && cd .. && rm -rf libjson`

If that doesn't work try the method below. It's dangerous for long time use but is a hacky workaround to get your bar back running.

- Find out which version the old one is

  ` ls -l | grep "libjsoncpp.so" `

- delete the file and create a symlink to the new one until the dev remembers to update the files again.

  ` sudo ln -sf /lib/libjsoncpp.so.NEWVERSIONNUMBER /lib/libjsoncpp.so.OLDVERSIONNUMBERNVIM/VIM `

  This is a one line command. (It may be displayed in multiple lines because of the length.)

# VIM airline font installation
- Install plug.vim or any other plugin loader supported by airline https://github.com/junegunn/vim-plug#installation
- Install airline for vim https://github.com/vim-airline/vim-airline#installation
- Install the powerline fonts with
  `git clone https://aur.archlinux.org/powerline-fonts-git.git powerlineFonts && cd powerlineFonts && makepkg -csri< && cd .. && rm -rf powerlineFonts`
  or if you only want the "hack" font, install `ttf-hack` [community]
- Select a font from here https://github.com/powerline/fonts in your terminal emulator

# Save diskspace when using Markdown
- Install `pandoc-bin` (AUR) instead of `pandoc` [community]
- save 700MB diskspace (taken up by haskell dependencies of the [community] repo)
- https://pandoc.org/MANUAL.html#pandocs-markdown

# File Recovery on Linux
- use the tool `ddrescue`

- only use the broken disk in read-only mode. DON’T write to it or your lost data can be overwritten.

  https://wiki.archlinux.org/index.php/file_recovery

# TeamSpeak mutes other audio sources
- open the pulse audio config file `sudo nvim /etc/pulse/default.pa`
- comment out the line 

  `load-module module-role-cork`

- save, close the file and reboot

# US international layout without dead keys
- open your xinitrc `nvim /etc/X11/xinit/xinitrc`

- append this at the end of the file:

```
Section "InputClass"  Identifier "Keyboard Defaults"
    MatchIsKeyboard "yes"
    Option "XkbLayout" "us"
    Option "XkbVariant" altgr-intl"
EndSection
```

# PulseAudio Crackling
- search for `load-module module-udev-detect` in `/etc/pulse/default.pa`
- append `tsched=0` to the end of it with a space in between
- either restart or run `pulseaudio -k && pulseaudio --start`

## Eduroam with NetworkManager and without a DE

1. Download the correct eduroam_cat installer for your university from [here](https://cat.eduroam.org/)
2. Install the following packages
    ```
    xbps-install NetworkManager dbus python3 python3-dbus
    ```
    dbus and python3 are necessary because the cat installer depends on dbus to talk to NetworkManager.
3. Enable and start the service for dbus and NetworkManager
    ```bash
    ln -s /etc/sv/NetworkManager /var/service
    ln -s /etc/sv/dbus /var/service
    ```
    Just to make sure:
    ```bash
    sv up NetworkManager
    sv up dbus
    ```
4. Execute the eduroam_cat installer with root permissions and follow the instructions.
    ```
    sudo /path/to/eduroam-linux-XXXXX-eduroam.py
    ```

