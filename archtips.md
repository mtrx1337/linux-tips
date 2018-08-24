# metr1xx’ random Arch Linux tips that may or may not help/work you.

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

# Printing with an Epson Inkjet printer
- Install <code>cups</code> and <code>cups-pdf</code>
- CUPS installation better explained: http://metr1xx.de/archprinting.html
- Enable and start org.cups.cupsd.service
- <code>systemctl enable org.cups.cupsd</code>
- Install <code>epson-inkjet-printer-escpr</code> (AUR)

# AMDGPU on Southern Island cards
- works with HD7950, HD7970 and 7990(I don't really know but it should.)
- The whole thing better explained: http://metr1xx.de/archamdgpu.html
- Install those packages preferably from the official repositories:

  <code> sudo pacman -S mesa lib32-mesa  xf86-video-amdgpu vulkan-radeon libva-vdpau-driver opencl-amd </code>

  if you can't find them in the official repositories install them from the AUR.
- Add <code>amdgpu</code> as the first module in the <code>#MODULES</code> array in the file <code>/etc/mkinitcpio.conf</code>
- Create a file called <code>noradeon.conf</code> in <code>/etc/modprobe.d/</code> with the content <code>blacklist radeon</code> or just run:
  <code>sudo echo "blacklist radeon" > /etc/modprobe.d/noradeon.conf </code>
- Rebuild the initramfs with <code>sudo mkinitcpio -p linux</code>
- Set the necessary kernel parameters in GRUB:
  - Open <code>/etc/default/grub</code>
  - At the line GRUB_CMDLINE_LINUX_DEFAULT, append the parameters
    <code>amdgpu.si_support=1 radeon.si_support=0</code>
    inside of the quotation marks to pass the necessary kernel parameters
- rebuild the grub config with 
  <code>grub-mkconfig -o /boot/grub/grub.cfg</code>
- reboot

# LightDM installation
- Install <code>lightdm</code> and <code>lightdm-gtk-greeter</code> packages
- Enable lightdm.service to start it at boot
  <code>sudo systemctl enable lightdm</code>
- Install <code>lightdm-gtk-greeter-settings</code> to configure lightdm

# GRUB installation from an archiso stick
make sure the root partition is mounted in /mnt of your usb stick
  (where sda2 is your ROOT partition)
  <code>mount /dev/sda2 /mnt</code>
- make sure the UEFI partition is mounted in /mnt/boot of your usb stick
  (where sda1 is your BOOT partition)
  <code>mount /dev/sda1 /mnt/boot</code>
- change your root directory from the stick to your arch installation
  <code>arch-chroot /mnt</code>
- install grub and efibootmgr
  <code>pacman -S grub efibootmgr</code>
- install grub as your bootloader with 
  <code>grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB-Bootloader</code>
- generate / update grub config with 
  <code>grub-mkconfig -o /boot/grub/grub.cfg</code>
- (Optional) install <code>grub-customizer</code> (AUR) and <code>arch-silence</code> (AUR) grub theme for a nice bootloader screen

# Polybar libjson error after updating

Have you updated polybar and now it's not working anymore? Running <code>polybar example &</code> returns a libjson error? Try this.

- Find out which version the old one is

  <code> ls -l | grep "libjsoncpp.so" </code>

- delete the file and create a symlink to the new one until the dev remembers to update the files again.

  <code> sudo ln -sf /lib/libjsoncpp.so.NEWVERSIONNUMBER /lib/libjsoncpp.so.OLDVERSIONNUMBERNVIM/VIM </code>

  This is a one line command. (It may be displayed in multiple lines because of the length.)

# VIM airline font installation
- Install plug.vim or any other plugin loader supported by airline https://github.com/junegunn/vim-plug#installation
- Install airline for vim https://github.com/vim-airline/vim-airline#installation
- Install the powerline fonts with
  <code>git clone https://github.com/powerline/fonts && cd fonts && chmod u+x install.sh && sh install.sh </code>
  or if you only want the "hack" font, install <code>ttf-hack</code> [community]
- Select a font from here https://github.com/powerline/fonts in your terminal emulator

# Save diskspace when using Markdown
- Install <code>pandoc-bin</code> (AUR) instead of <code>pandoc</code> [community]
- save 700MB diskspace (taken up by haskell dependencies of the [community] repo)
- https://pandoc.org/MANUAL.html#pandocs-markdown

# File Recovery on Linux
- use the tool <code>ddrescue</code>

- only use the broken disk in read-only mode. DON’T write to it or your lost data can be overwritten.

  https://wiki.archlinux.org/index.php/file_recovery

# TeamSpeak mutes other audio sources
- open the pulse audio config file <code>sudo nvim /etc/pulse/default.pa</code>
- comment out the line 

  <code>load-module module-role-cork</code>

- save, close the file and reboot

# US international layout without dead keys
- open your xinitrc <code>nvim /etc/X11/xinit/xinitrc</code>

- append this at the end of the file:

```
Section "InputClass"  Identifier "Keyboard Defaults"
    MatchIsKeyboard "yes"
    Option "XkbLayout" "us"
    Option "XkbVariant" altgr-intl"
EndSection
```
