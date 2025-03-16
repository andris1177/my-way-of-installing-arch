# My way of installing Arch Linux

## Installing arch can be hard for the first time but if you understand the steps it is not that difficult, the installation can be separated into three major part the first is makeing the partitions formating and mounting them, the second part is installing the base system and creating the fstab, and the third step is setting up the system. You could use the archinstall script that is on the archiso but I recommend to install arch with this methode bacuse you can learn a lot about the system so if it break you can have a better chance fixing it. A couple of months ago the grub have completly died on my system but I was able to boot into the arch iso mount my drives chroot into my system and reinstall grub so I was able to avoid reinstalling my entire system. 

## This tutorial will show you how to install arch linux on a uefi system without encryption.

### If your terminal gets cluttered, you can use ctrl+l to clear it.

## Connecting to the internet

### If you want to use wifi, you need to connect to the internet first. You can do this with iwctl.

``` bash
iwctl
device list # Find your WiFi device name
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

### Now, you can start partitioning the drive. I think the best way is to use cfdisk. You have to create three partitions: a boot partition around 100 megabits, a swap partition, which should be the size of your ram and a home partition, which is going to contain the os. First, you have to run lsblk. This will print out all of your connected drives. Choose the drive that you are going to install on. For example, I have two NVMe ssd in my system, so it looks like this.

``` bash
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
nvme1n1     259:0    0 465.8G  0 disk 
├─nvme1n1p1 259:1    0   100M  0 part 
├─nvme1n1p2 259:2    0    16M  0 part 
├─nvme1n1p3 259:3    0 465.1G  0 part 
└─nvme1n1p4 259:4    0   530M  0 part 
nvme0n1     259:5    0 931.5G  0 disk 
```

### In this case, on my first nvme ssd, I have Windows installed. I want to install Arch on the secodn nvme drive, so I'm going to use cfdisk to partition it. If you have a sata drive, you're going to use sda, or any other drive that starts with sd and is followed by an alphabet character. In cfdiks, you have to use the arrow key to navigate. You can use up and down to select the partition, and left and right to select what you want to do with it. If you have a partition on this drive first, you need to delete it and then create the new partitions. After that write and quit and, you should see something like this after running lsblk.

``` bash
nvme0n1     259:5    0 931.5G  0 disk 
├─nvme0n1p1 259:6    0   100M  0 part
├─nvme0n1p2 259:7    0    32G  0 part
└─nvme0n1p3 259:8    0 899.4G  0 part 
```

### After this, you can format the drives, in my case, nvme0n1p3 is the home, nvme0n1p1 is the boot, and nvme0n1p2 is the swap.

``` bash
mkfs.ext4 /dev/nvme0n1p3
mkfs.fat -F 32 /dev/nvme0n1p1
mkswap /dev/nvme0n1p2
```

### Now that yoou have the partitions you should mount them.

``` bash
mount /dev/nvme0n1p3 /mnt
mkdir -p /mnt/boot/efi
mount /dev/nvme0n1p1 /mnt/boot/efi
swapon /dev/nvme0n1p2
```

### After mounting the drives, you should see something like this.

``` bash
nvme0n1     259:5    0 931.5G  0 disk 
├─nvme0n1p1 259:6    0   100M  0 part /mnt/boot/efi
├─nvme0n1p2 259:7    0    32G  0 part [SWAP]
└─nvme0n1p3 259:8    0 899.4G  0 part /mnt
```

## Installing the os

### This will install the bare minimum system, you can install appas after finishing the os installation.

``` bash
pacstrap /mnt base linux linux-firmware sof-firmware base-devel grub efibootmgr neovim networkmanager
```

### Generating fstab can be done by hand, but I recommend against it, especially in the first install.

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

### To set the os language the loacle-gen need to be edited. If you chose neovim as the text editor without using it before the most important is to quit vim press esc and write :w or :wq to save the changes to get into insert mode press i. 

``` bash
nvim /etc/locale.gen
```

### Now that you selected the language that you want to use generate it. 

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

### If you use a non us layout keyboard you should edit vconsole.conf to set the correct layout.

``` bash
nvim /etc/vconsole.conf
```

### example config
```bash
KEYMAP=uk
```

### You can set the hostname here.

``` bash
nvim /etc/hostname
```

## Setting up the users

### First thing first you should set a root password.

``` bash
passwd
```

### To add a user you can use useradd

``` bash
useradd -m -G wheel -s /bin/bash name
```

### And dont forget to set a password for the new user.

``` bash
passwd name
```

### To enable sudo access, you have to edit the sudoers file with visudo.

``` bash
EDITOR=nvim visudo
```

### You only have to remove one # befor this line

Example config
``` bash
...
%wheel ALL=(ALL:ALL) ALL
...
```

### If you want to test it you can log inot your account and run a command that require a sudo password to see if you have the right to run commands with sudo.

``` bash
su name
sudo pacman -Syu
exit
```

## Enabeling system services

### At this point, you don't have a lot of things to enable, only the network manager is needed to be enabled.

``` bash
systemctl enable NetworkManager
```

## Configuring grub

``` bash
grub-install /dev/your disk
grub-mkconfig -o /boot/grub/grub.cfg
```

## Exiting the installed system and rebooting.

### When umounting the drives, not all drives will be unmounted this is normal you can reboot. 

``` bash
exit
umount -a
reboot
```

## Setting up the system

### If you have done this correctly, you are going to see the grub, and after that, a tty console where you can log in with your username and password.

###  Connecting to the internet. If you use wired internet, you can skip this step.

``` bash
nmcli device status
nmcli dev wifi list
sudo nmcli dev wifi connect network-ssid password "network-password"
```

### After connecting to the internet, you should check if you have internet. You can do this by pinging a website, for example.

``` bash
ping archlinux.org
```

### expected output

``` bash
64 bytes from archlinux.org (95.217.163.246): icmp_seq=2 ttl=44 time=39.4 ms
```

### Now you have to decide which dekstop environment (de) or window manager (wm) do you want to use, you can see the available options here <br> de: https://wiki.archlinux.org/title/desktop_environment, <br> wm: https://wiki.archlinux.org/title/window_manager, <br> I'm going to use i3-wm but feel free to chose any other de or wm but you may have to configure your system differently so if you chose something else see the archwiki or finde anathor tutorial for that specific de or wm. You dont have to be worry to much about wich one to chose you can install more than one, i use i3 as my main wm but i have installed dwm and hyprland to test them out. If you installed more than one you can selecvt it in your login screen. 

``` bash 
sudo pacman -S xorg xorg-xinit xorg-server lightdm-gtk-greeter i3-wm kitty i3status dmenu
sudo systemctl enable lightdm
```

### A couple of applications in the standard repo and the aur require 32bit libraries, so you should enable it by removing the # before the multilib in the config file. 

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

### In Arch Linux, a lot of programs are in the standard repository, but a lot of applications aren't, so you may have to use the Arch user repository (AUR). When you are using it, you have to be more careful about what you install because it can contain broken, ore unmaintained packages.

```
sudo pacman -S git
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

### After installing yay you can aces the aur, yay works almost the same way as pacman one exepcion is that you dont have to run yay as root.

### installing Nvidia GPU drivers

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


### Installing pipewire

``` bash
sudo pacman -S pipewire pipewire-pulse
```

### If you want to use a GUI wiffi manager you can install this but networkmanager includes a tui tool so you can use nmtui as weell.

``` bash
sudo pacman -S nm-connection-editor
```

### If you want a gui for pipewire.

``` bash
sudo pacman -S pavucontrol
```

### For pipewire you can also install wireplumber wich is a realy cool way to manage audio/viode io between apps so you shouldt try it out. https://www.youtube.com/watch?v=Zv1P6-kUn0c&pp=ygUMI3dpcmVwbHVtYmVy

``` bash
sudo pacman -S wireplumber
```

### If you want to set your displays in a gui you can use arandr, but keep in minde that this tool will only generate a script that uses xrandr to set the correct resolution and orientation so you have to make sure to run the output script in start in i3 you can do this by adding a line to the config file.

``` bash
sudo pacman -S arandr
```

### example autorun in i3
``` bash
exec--no-startup-id path_to_script
```

### Some programs that i usually instal.

``` bash
sudo pacman -S steam fastfetch firefox thunar libreoffice-fresh gvfs breeze-gtk breeze lxappearance nitrogen mpv nsxiv okular btop cmake python3 imagemagick xclip tumbler playerctl ripgre unzip 
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

## Setting up a firewall

```
sudo pacman -S ufw
sudo systemctl enable ufw --now
sudo ufw enable
```

### You can install a GUI to manage UFW, but I don't recommend it. You have to set the firewal rules once so doing it in the terminal shouldn't be a scourge. 

``` bash
sudo pacman -S gufw
sudo gufw
```

## Laptop specific settings

### If you are using a laptop, you may want to set the display scaling. You can do this by creating a config file, it wouldn't work on the tty, but it will be visible after a restart in i3.

``` bash
echo "Xft.dpi: 140" > ~/.Xresources
xrdb -merge ~/.Xresources
```

### Setting up the trackpad

``` bash
sudo touch /etc/X11/xorg.conf.d/90-touchpad.conf
sudo nvim /etc/X11/xorg.conf.d/90-touchpad.conf
```

### the content of the file

``` bash
# /etc/X11/xorg.conf.d/90-touchpad.conf
Section "InputClass"
        Identifier "touchpad"
        MatchIsTouchpad "on"
        Driver "libinput"
        Option "Tapping" "on"
        Option "NaturalScrolling" "true"
        Option "TappingButtonMap" "lrm" # 1/2/3 finger, for 3-finger middle lrm
EndSection
```

### improving batterry life probably one of the most important thing in a laptop is to set up the system so it can work from a battery for a reasonable amount of time. I recommend auto-cpufrq, which is probably the best tool to use

``` bash
yay -S auto-cpufreq
sudo auto-cpufreq --install
sudo systemctl enable --now auto-cpufreq
```

### Now you can reboot and you will see the lightdm display manager, and now you can log in. If you chose i3 you can check out my config for refference. https://github.com/andris1177/my-dotfiles
