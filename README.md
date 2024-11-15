# My way of installing Arch Linux

## This tutorial will show you how to install arch linux on a uefi system without encryption.

### If your terminal gets cluttered, you can use ctrl+l to clear it.

## Connecting to the internet

### If you are using a laptop or a pc with wifi, you need to connect to the internet first. You can do this with iwct.

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

### In this case, on my first nvme ssd, I have Windows installed. I want to install Arch on the secodn nvme drive, so I'm going to use cfdisk to partition it. If you have a sata drive, you're going to use sda, or any other drive that starts with sd and is followed by an alphabet character. In cf diks, you have to use the arrow key to navigate. You can use up and down to select the partition, and left and right to select what you want to do with it. If you have a partition on this drive first, you need to delete it and then create a new partition, a 100M for boot, as many Gs for swapping as much ram as you have, and the rest for home and root. After that write and quit and, you should see something like this.

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

### After this, you can mount your drives.

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

### Generating fstab can be done by hand, but I recommend not, especially in the first install.

``` bash
genfstab -U /mnt > /mnt/etc/fstab
```

### After this, you can enter the newly installed os.

``` bash
arch-chroot /mnt
```

## Localization

### first you have to set the region.

``` bash
ln -sf /usr/share/zoneinfo/Continent/city /etc/localtime
hwclock --systohc
```

### After this, you have to set the os language. You can do this by editing locale.gen and uncommenting the language that you want to use. To exit vim press esc, type wq, and press enter. Uncommenting means removing the # before the language that you want to use.

``` bash
nvim /etc/locale.gen
```

### After that, you can generate it.

``` bash
locale-gen
```

### A couple of applications don't use locale-gen but instead read /etc/locale.conf, so you want to edit that as well. To set the language, you have to type LANG= and the language, like in the locale.gen file. For example, to use British English, you have to use en_GB.utf-8.

``` bash
nvim /etc/locale.conf
```

### After this, you can set the language of the keyboard. It's not important if you use a US keyboard, but if not, you have to set it.

``` bash
nvim /etc/vconsole.conf
# example: KEYMAP=uk
```

### After this, you have to set the hostname.

``` bash
nvim /etc/hostname
```

## Setting up users

### To have a superuser, you have to set a password.

``` bash
passwd
```

### Now you can add users. If you want to access sudo from that account, it is very important to add it to the wheel group.

``` bash
useradd -m -G wheel -s /bin/bash name
```

### Now you should set a password.

``` bash
passwd name
```

### To enable sudo access, you have to edit the sudoers file with visudo.

``` bash
EDITOR=nvim visudo
```

### In this configuration, you want to uncomment: %wheel      ALL=(ALL:ALL) ALL <br> <br> To check if you have done it correctly, you can log in to your new user and check if sudo works. If the system update has been run, sudo has been set up correctly.

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

### When umounting the drives, not all drives can be unmounted, and it's not a problem, you can reboot safely.

``` bash
exit
umount -a
reboot
```

## Setting up the system

### If you have done this correctly, you are going to see the grub, and after that, a tty console where you can log in with your username and password.

###  Connecting to the internet. If you use wired internet, you don't have to do anything.

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

### Now you have decided which dekstop environment (de) or window manager (wm) you want to use, you can see the available options here <br> de: https://wiki.archlinux.org/title/desktop_environment, <br> wm: https://wiki.archlinux.org/title/window_manager, <br> I'm going to use i3-wm.

``` bash 
sudo pacman -S xorg xorg-xinit xorg-server lightdm-gtk-greeter i3-wm kitty i3status dmenu
sudo systemctl enable lightdm
```

### A couple of applications in the standard repo and the aur require 32bit libraries, so you should enable it by enabeling multilib.

``` bash
sudo nvim /etc/pacman.conf
remove # befor multilib
sudo pacman -Syu
```

### In Arch Linux, a lot of programs are in the standard repository, but a lot of applications aren't, so you may have to use the Arch user repository (AUR). When you are using it, you have to be more careful about what you install because it can contain broken, or malicious applications.

```
sudo pacman -S git
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

### After installing yay you can aces the aur, yay works almost the same way as pacman one exepcion is tha you dont have to run yay as root.

### Installing AMD GPU drivers

``` bash
yay -S lib32-mesa
sudo pacman -S xf86-video-amdgpu 
sudo pacman -S vulkan-radeon
sudo pacman -S amdvlk
sudo pacman -S mesa-vdpau
```

### Now you can reboot

``` bash
reboot
```

### installing Nvidia GPU drivers

``` bash
sudo pacman -S nvidia
sudo pacman -S lib32-nvidia-utils
```

### Edit the mkinitcpio to prevent loading the nouveau driver at boot.

``` bash
sudo nvim /etc/mkinitcpio.conf
# remove kms from HOOKS array 
```

### After editing the mkinitcpio, you have to rebuild it.

``` bash
 mkinitcpio -P
```

### Now you can reboot.

``` bash
reboot
```

### Installing pipewire

``` bash
sudo pacman -S pipewire pipewire-pulse
```

### If you want to use a GUI to connect to wifi.

``` bash
network config gui for nmcli: sudo pacman -S nm-connection-editor
```

### If you want a gui for pipewire.

``` bash
sound settings gui: sudo pacman -S pavucontrol
```

### If you want to set your displays in a gui, if you want to change something permanently, you have to save the layout into a script and run it in startap. For example, you can do it in i3 config.

``` bash
display settings gui: sudo pacman -S arandsr
# start script with i3, optional
exec--no-startup-id path_to_script
```

### Some programs that i usually instal.

``` bash
steam: sudo pacman -S steam
neofetch: sudo pacman -S neofetch
sudo pacman -S firefox
sudo pacman -S thunar
sudo pacman -S libreoffice-fresh
sudo pacman -S gvfs # for thunar to work properly
sudo pacman -S breeze-gtk
sudo pacman -S breeze
sudo pacman -S lxappearance 
sudo pacman -S nitrogen
sudo pacman -S mpv
sudo pacman -S nsxiv
sudo pacman -S okular
sudo pacman -S btop
sudo pacman -S cmake
sudo pacman -S python3
sudo pacman -S imagemagick # for screenshot
sudo pacman -S xclip
sudo pacman -S tumbler
sudo pacman -S playerctl
```

## Setting up a firewall

```
sudo pacman -S ufw
sudo systemctl enable ufw --now
sudo ufw enable
```

### You can install a GUI to manage UFW, but I don't recommend it. The GUI is probably better because you are going to set it up once and never touch it, but if you want to use it, you can.

``` bash
sudo pacman -S gufw
sudo gufw
```

## Laptop specific settings

### If you are using a laptop, you probably have to set a display scaling. You can do this by creating a config file, it wouldn't work on the tty, but it will be visible after a restart in i3.

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

### improving batterry life <br> <br> probably the most important thing in a laptop is to set up the system so it can work from a battery a reasonable amount of time. I recommend auto-cpufrq, which is probably the best tool to use

``` bash
yay -S auto-cpufrq
sudo auto-cpufreq --install
sudo systemctl enable --now auto-cpufreq
```

### Now you can reboot and you will see the lightdm display manager, and now you can log in. After logging in, if you decide to use i3, it will go through the initial setup. After that you can configure it by editing the config file in ~/.config/i3/config. Using arch especially with a window manager require some knowledge and most of them you can get by reading the man page or searching for it in the arch website or just searching for old forum posts.
