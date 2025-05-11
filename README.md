# My way of installing Arch Linux

## Installing arch can be hard for the first time but if you understand the steps it isn't that difficult, the installation can be broken down into three major part. The first is making the partitions formatting and mounting them, the second part is installing the base system and creating the fstab, and the third step is setting up the system. You could use the built in archinstall script but I recommend to install arch with this method because you can learn a lot about your system so if it breaks you have a better chance fixing it.

## This tutorial will show you how to install arch linux on a uefi system without encryption.

### If your terminal gets cluttered, you can use ctrl+l to clear it.

## Connecting to the internet

### If you want to use wifi, you need to connect to a wifi network first. You can do this with iwctl.

``` bash
iwctl
device list 
device <device_name> set-property Powered on
station <device_name> scan
station <device_name> get-networks
station <device_name> connect SSID
exit
```

### After connecting to the internet, you should check if you have internet. You can do this by pinging a website, for example.

``` bash
ping archlinux.org
```

### expected output

``` bash
64 bytes from archlinux.org (95.217.163.246): icmp_seq=2 ttl=44 time=39.4 ms
```

## Partitioning the drives

### Now, you can start partitioning the drive. You can do this in a lot of ways. My go to method right now is to have a separate root and home partition so if I need to reinstall arch I can leave my personal data on the home partition and only format the root partition. Also you can use swap if you don’t have enough ram, I created an 8gb swap partition the last time I have reinstalled arch but probably I wouldn't create that as my system never used it. 

``` bash
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
nvme0n1     259:5    0 931.5G  0 disk 
```

### I'm going to use cfdisk to partition it. If you have a sata drive, you're going to use sda, or any other drive that starts with sd and is followed by an alphabet character. In cfdisk, you have to use the arrow keys to navigate. You can use up and down to select the partition, and left and right to select what you want to do with it. If you have a partition on this drive first, you need to delete it and then create the new partitions. After that write and quit and, you should see something like this after running lsblk. Also if you have other drives in your system you can format them and mount them later, this way when you generate the fstab it will be in it so the will be mounted at boot just like the system partitions. For example I have two ssd I have mounted beside my os ssd. 

``` bash
nvme1n1     259:0    0 465.8G  0 disk 
├─nvme1n1p1 259:1    0     1G  0 part 
├─nvme1n1p2 259:2    0   100G  0 part
├─nvme1n1p3 259:3    0     8G  0 part 
└─nvme1n1p4 259:4    0 356.8G  0 part 
```

``` bash
nvme0n1     259:5    0 931.5G  0 disk 
└─nvme0n1p1 259:6    0 931.5G  0 part
sda           8:0    0 447.1G  0 disk 
└─sda1        8:1    0 447.1G  0 part 
```

### Now you can format the drives, in my case, nvme1n1p1 is the boot, nvme1n1p2 is the toot, nvme1n1p3 is the swap and nvme1n1p4 is the home.

``` bash
mkfs.fat -F 32 /dev/nvme0n1p1
mkfs.ext4 /dev/nvme0n1p2
mkswap /dev/nvme0n1p3
mkfs.ext4 /dev/nvme0n1p4
```

### Now that you have the partitions you should mount them.

``` bash
mkdir -p /mnt/boot/efi
mount /dev/nvme0n1p1 /mnt/boot/efi
mount /dev/nvme0n1p2 /mnt
swapon /dev/nvme0n1p3
mount /dev/nvme0n1p4 /mnt/home
```

### After mounting the drives, you should see something like this. 

``` bash
nvme1n1     259:0    0 465.8G  0 disk 
├─nvme1n1p1 259:1    0     1G  0 part mnt/boot/efi
├─nvme1n1p2 259:2    0   100G  0 part /mnt
├─nvme1n1p3 259:3    0     8G  0 part [SWAP]
└─nvme1n1p4 259:4    0 356.8G  0 part /mnt/home
```

### If you have added other drives make sure you see them and they are mounted to the correct folder

``` bash
nvme0n1     259:5    0 931.5G  0 disk 
└─nvme0n1p1 259:6    0 931.5G  0 part /games
sda           8:0    0 447.1G  0 disk 
└─sda1        8:1    0 447.1G  0 part /data
```

## Installing the os

### This will install the bare minimum system, you can install apps after finishing the os installation.

``` bash
pacstrap /mnt base linux linux-firmware sof-firmware base-devel grub efibootmgr neovim networkmanager git
```

### Generating fstab can be done by hand, but there is no reason to do so.

``` bash
genfstab -U /mnt > /mnt/etc/fstab
```

### Now that the system is installed and fstab is done you can log into the system and start configuring the basic settings

``` bash
arch-chroot /mnt
```

## Localization

### first you have to set the region.

``` bash
ln -sf /usr/share/zoneinfo/Continent/city /etc/localtime
hwclock --systohc
```

### To set the os language you have to edit the locale.gen.

``` bash
nvim /etc/locale.gen
```

### Now that you selected the language you want to use locale-gen to generate it. 

``` bash
locale-gen
```

### A couple of application doesn't use the output of locale-gen but instead read /etc/locale.conf, so you want to edit that as well.

``` bash
nvim /etc/locale.conf
```

### example config
```bash
LANG=en_GB.utf-8
```

### If you are using a non us layout keyboard you can set it in vconsole.conf.

``` bash
nvim /etc/vconsole.conf
```

### example config
```bash
KEYMAP=uk
```

### To set the hostname edit.

``` bash
nvim /etc/hostname
```

## Setting up the users

### First thing first you should set a root password.

``` bash
passwd
```

### To add a user you can use

``` bash
useradd -m -G wheel -s /bin/bash name
```

### And don’t forget to set a password for the new user.

``` bash
passwd name
```

### To enable sudo access, you have to edit the sudoers file with visudo.

``` bash
EDITOR=nvim visudo
```

### You only have to remove one # before this line

Example config
``` bash
...
%wheel ALL=(ALL:ALL) ALL
...
```

### If you want to test it you can log inot your account and run a command that require a sudo to see if you have the right to run commands with sudo.

``` bash
su name
sudo pacman -Syu
exit
```

## Enabling system services

### At this point, you don't have a lot of things to enable, you only have to enable NetworkManager.

``` bash
systemctl enable NetworkManager
```

## Configuring grub

``` bash
grub-install /dev/your disk
grub-mkconfig -o /boot/grub/grub.cfg
```

## Exit from the chroot environment and reboot.

### When umounting the drives, not all drives will be unmounted this is normal, you can reboot. 

``` bash
exit
umount -a
reboot
```

## Setting up the system

### If you have done this correctly, you are going to see the grub, and after that, a tty console where you can log in with your username and password.

###  Connecting to the internet. If you are using wired internet, you can skip this step.

``` bash
nmcli device status
nmcli dev wifi list
sudo nmcli dev wifi connect network-ssid [SSID] password [PASSWORD]
```

### After connecting to the internet, check it by pinging a something.

``` bash
ping archlinux.org
```

### expected output

``` bash
64 bytes from archlinux.org (95.217.163.246): icmp_seq=2 ttl=44 time=39.4 ms
```

## installing Nvidia GPU drivers

``` bash
sudo pacman -S nvidia
sudo pacman -S lib32-nvidia-utils
```

### Edit the mkinitcpio to prevent loading the nouveau driver at boot.

``` bash
sudo nvim /etc/mkinitcpio.conf
```
### Remove kms from HOOKS

``` bash
HOOKS=(base udev autodetect microcode modconf keyboard keymap consolefont block filesystems fsck)
```

### After editing the mkinitcpio, you have to rebuild it.

``` bash
 mkinitcpio -P
```

### In Arch Linux, a lot of programs are in the standard repository, but a lot of applications aren't, so you may have to use the Arch user repository (AUR). When you are using it, you have to be more careful about what you install because it can contain broken, ore unmaintained packages.

``` bash
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

### Now you have to decide which desktop environment (de) or window manager (wm) do you want to use, you can see the available options here <br> de: https://wiki.archlinux.org/title/desktop_environment, <br> wm: https://wiki.archlinux.org/title/window_manager, <br> I'm going to use hyprland because that's the objectively correct option. Joke aside feel free to choose any of the available option I used to use kde and i3 but hyprland is the best one I have tried so far. If you choose hyprland here are my relevant dotfiles. https://github.com/andris1177/dotfiles.git

``` bash 
sudo pacman -S hyprland hyprpaper hyprlock xdg-desktop-portal-hyprland
```

### If you want to have the latest version available you can install the aur git version. Don’t forget that if you do so you have to rebuild these packages at every update, otherwise you will have a broken wm. 

``` bash
yay -S hyprland hyprpaper hyprlock xdg-desktop-portal-hyprland-git
```

### To have a log in manager don't forget to install one I usually go with sddm

``` bash
sudo pacman -S sddm
sudo systemctl enabel sddm 
```

### Recommended by the hyprland wiki

``` bash
sudo pacman -S qt5-wayland qt6-wayland
```

### If you have an nvidia gpu please check out the hyprland wiki, https://wiki.hyprland.org/Nvidia/

### A couple of applications in the standard repo and the aur require 32bit libraries, so you should enable it by removing the # before the multilib in the config file. One example is stem. 

``` bash
sudo nvim /etc/pacman.conf
sudo pacman -Syu
```

### example config
``` bash
...
[multilib]
Include = /etc/pacman.d/mirrorlist
...
```

### Installing pipewire

``` bash
sudo pacman -S pipewire pipewire-pulse
```

### If you want a gui for pipewire.

``` bash
sudo pacman -S pavucontrol
```

### If you want to use a GUI wiffi manager you can install this but networkmanager includes a tui tool so you can use nmtui as well.

``` bash
sudo pacman -S nm-connection-editor
```

### For pipewire you can also install wireplumber wich is a really cool way to manage audio/video io between apps so you should try it out. https://www.youtube.com/watch?v=Zv1P6-kUn0c&pp=ygUMI3dpcmVwbHVtYmVy

``` bash
sudo pacman -S wireplumber
```

### 

``` bash
sudo pacman -S steam fastfetch firefox thunar libreoffice-fresh gvfs breeze-gtk breeze lxappearance nitrogen mpv nsxiv okular btop cmake python3 imagemagick xclip tumbler playerctl ripgre unzip cmake discord
```
steam
fastfetch
firefox 
thunar # file browser
libreoffice-fresh
gvfs # for thunar to work properly
breeze-gtk # theme
breeze # theme
lxappearance # theme selector
nitrogen # wallpaper tool
mpv # video player
nsxiv # image viewer
okular # pdf viewer
btop # system resources viewer
cmake
python3
imagemagick # for screenshot
xclip # to save the screenshot to the clipboard
tumbler # give thumbnail to images 
playerctl
unzip
cmake
discord


## Setting up a firewall

```
sudo pacman -S ufw
sudo systemctl enable ufw --now
sudo ufw enable
```

### You can install a GUI to manage UFW, but I don't recommend it. You have to set the firewall rules once so doing it in the terminal shouldn't be a scourge. 

``` bash
sudo pacman -S gufw
sudo gufw
```

## Laptop specific settings

### improving battery life probably one of the most important thing in a laptop is to set up the system so it can work from a battery for a reasonable amount of time. I recommend auto-cpufrq, which is probably the best tool to use

``` bash
yay -S auto-cpufreq
sudo auto-cpufreq --install
sudo systemctl enable --now auto-cpufreq
```

### Now you can reboot and you will see the sddm display manager, and now you can log in.

## My final recommendation is RTFM, hyprland and arch has a really good wiki so you can find everything relevant there. 
https://wiki.hyprland.org/ <br>
https://wiki.archlinux.org/title/Main_page