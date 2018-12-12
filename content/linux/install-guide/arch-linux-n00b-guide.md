+++
date = "2017-01-10T00:14:35+01:00"
lastMod = "2018-09-25T00:14:06+01:00"
title = "[UPDATED] Arch Linux n00b guide"
draft = false
categories = ["Linux"]
series = ["Install guides"]

+++

![Arch Linux](/img/arch-linux-desktop.jpg)

## Prerequisites
### Check internet connection
```
ping -c 3 www.google.com
```

### If needed try connecting via WiFi
```
wifi-menu
```

### Check EFI vars present
```
efivar -l
```

---

## Main installation

### Set system time using NTP:
```
timedatectl set-ntp true
timedatectl status
```

### Partitioning
**TIP:** If dual-booting leave a 128MiB empty ‘gap’ partition in between your ‘other os’ and your new partition.

#### Zap all data on disk and create a new GPT table.
Show current partition table
```
lsblk
```

NOTE: X represents your drive mountpoint. mine is `sda`.
```
gdisk /dev/sdX
```

`o` # Create a new empty GUID partition table (GPT)  
`y` # Confirm

EFI System Partition (ESP)  
`n` # Add a new partition  
`1` # Partition number  
`[Return]` # First sector  
`+512M` # Last sector = size  
`ef00` # Partition type = EFI System

LUKS container  
`n` # Add a new partition  
`2` # Partition number  
`[Return]` # First sector  
`[Return]` # Last sector = Use remaining space  
`8e00` # Partition type = Linux LVM  

`p` # Check partitions  
`w` # Write changes to disk and exit  
`y` # Confirm


Format EFI System Partition (ESP):
```
mkfs.fat -F32 /dev/sda1
```

Encrypt the other partition with LUKS (512 Bit AES-XTS and SHA512 for passphrase):
```
cryptsetup luksFormat -v -s 512 -h sha512 /dev/sda2
cryptsetup luksOpen /dev/sda2 luks
```

Setup Logical Volume Manager:
```
pvcreate /dev/mapper/luks
vgcreate rootvg /dev/mapper/luks
```

Create Logical Volumes:
```
lvcreate -n swap -L 4G -C y rootvg
lvcreate -n root -L 25G rootvg
lvcreate -n home -l 100%FREE rootvg
```

Check LVM Setup:
```
pvs
vgs
lvs
```

Create filesystems for LVs:
```
mkfs.ext4 /dev/mapper/rootvg-home
mkfs.ext4 /dev/mapper/rootvg-root
mkswap /dev/mapper/rootvg-swap
swapon /dev/mapper/rootvg-swap
```

Mount LVs and ESP for installation:
```
mount /dev/mapper/rootvg-root /mnt
mkdir /mnt/boot
mkdir /mnt/home
mount /dev/mapper/rootvg-home /mnt/home
mount /dev/sda1 /mnt/boot
```

Selecting fastest mirrors
```
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
sed -i 's/^#Server/Server/' /etc/pacman.d/mirrorlist.backup
rankmirrors -n 6 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist
```

### Install base system
```
pacstrap -i /mnt base base-devel
```

### Generate fstab (make sure swapon is set when running these)
```
genfstab -p /mnt >> /mnt/etc/fstab
nano /mnt/etc/fstab (for SSD change swap line from ‘defaults’ to ‘defaults,discard’)
```

### Mount into system
```
arch-chroot /mnt
```

### Set hostname
```
echo arch-laptop > /etc/hostname
```

### Set timezone
```
ln -s /usr/share/zoneinfo/Europe/Amsterdam /etc/localtime
```

### Locales
```
nano /etc/locale.gen
```

Uncomment the following two locales  
`en_US.UTF-8 UTF-8`  
`en_US ISO-8859-1`

```
locale-gen
```

### Set Language
```
echo LANG=en_US.UTF-8 > /etc/locale.conf
export LANG=en_US.UTF-8
```

Enabling the AUR repo
```
nano /etc/pacman.conf
```

Add these lines at the bottom of the file:  
`[archlinuxfr]`  
`SigLevel = Never`  
`Server = https://repo.archlinux.fr/$arch`

### User Management
#### Set root password

```
passwd
```

#### Create user:

```
useradd -m -g users -G wheel,storage,power -s /bin/bash <user>
```
```
passwd <user>
```

#### Setup sudoers:

```
EDITOR=nano visudo
```

Uncomment this line:
`%wheel ALL=(ALL) ALL`

Add to the bottom:
`Defaults rootpw`

## Installing some basic software
Bash completion:
```
pacman -S bash-completion
```

Install packages for WiFi:
```
pacman -S dialog wpa_supplicant wpa_actiond rfkill
```

Find and enable your wireless-interface
```
ip link
```

```
systemctl enable netctl-auto@<YOUR_INTERFACE>
```

Installing the bootloader:
```
bootctl install
```

Write long UUID to file for later use
```
blkid | grep sda2 | cut -f2 -d\" >> /boot/loader/entries/arch.conf
```

Create an entry for Arch
```
nano /boot/loader/entries/arch.conf
```

`title Arch Linux`  
`linux /vmlinuz-linux`  
`initrd /initramfs-linux.img`  
`options cryptdevice=UUID=<YOUR_UUID>:rootvg:allow-discards root=/dev/mapper/rootvg-root rw`


### Install Intel Microcode:
This is useful for when you want to be able to receive firmware updates to your CPU.
You should only follow this step if you run on a Intel CPU.
```
pacman -S intel-ucode
```

Add the following to your previously created boot entry:     
`initrd /intel-ucode.img`  

```
nano /boot/loader/entries/arch.conf
```

Your final boot entry should look like this:  
`title Arch Linux`  
`linux /vmlinuz-linux`  
`initrd /intel-ucode.img`  
`initrd /initramfs-linux.img`  
`options cryptdevice=UUID=<YOUR_UUID>:luks root=/dev/mapper/rootvg-root rw`

### Set required hooks for boot init.
in `/etc/mkinitcpio.conf` make sure the following hooks are present, if not add them:  
`encrypt`  
`lvm2`

An example of a complete string of hooks would be:
`base udev autodetect modconf block filesystems keyboard keymap encrypt lvm2 fsck`

```
mkinitcpio -p linux
```

### Finalize:
```
exit
umount -R /mnt
reboot
```

Congratulations! You are now booted into your new Arch system. As you've noticed things are a little 'texty' out here. Let's add some paint.

---

### Install a graphics driver and display server
```
sudo pacman -S mesa xorg-server xorg-xinit xorg-twm xorg-xclock
```

### Install awesome GUI stuff
```
sudo pacman -S gnome gdm gnome-tweaks chrome-gnome-shell gnome-keyring
```
```
sudo systemctl enable gdm.service
```

### Installing a AUR helper
In Arch we have a great asset that is the Arch User Respository (AUR). Here we can find all kinds of user maintained ~~packages~~ build-scripts that would otherwise be quite hard to install. Using an AUR helper makes installing packages from the AUR as easy as Pacman for instance. Let's install my favorite one, Aurman!
```
gpg --recv-keys 465022E743D71E39
```
```
git clone https://github.com/polygamma/aurman.git
```
```
cd aurman && makepkg -si
```

### Installing and enabling a Firewall
```
sudo pacman -S ufw
```
```
sudo ufw enable
```

### Installing and enabling OpenSSH-Server
```
sudo pacman -S openssh
```
```
sudo systemctl start sshd.socket
```
```
sudo systemctl enable sshd.socket
```

If needed make sure the SSH connection get's through the Firewall
```
sudo ufw allow 22
```

### Enable NetworkManager
```
sudo pacman -S networkmanager networkmanager-openvpn
sudo systemctl enable NetworkManager.service
```

### Install some funky themes
```
sudo pacman -S adapta-gtk-theme
```
```
aurman -S paper-icon-theme-git
```

### Install some awesome packages
```
sudo pacman -S file-roller vlc vim git keepassxc reflector jdk8-openjdk
```

#### Add a Mozilla signature in order to build Firefox
```
gpg --recv-key 0x61B7B526D98F0353
```
```
aurman -S firefox-nightly
```

### Install some support libraries
```
sudo pacman -S xdotool xsel udisks2 dosfstools exfat-utils ntfs-3g
```

Enable minimal media codecs
```
sudo pacman -S gstreamer gst-plugins-good gst-plugins-ugly
```


## After install stuff [Optional]:

### Install some nifty little package installers (Flatpak vs Snap.. FIGHT!)
```
sudo pacman -S flatpak  
```
```
aurman -S snapd
```

### Pacman hooks
Install a Pacman hook to update the mirrorlist to specified criterea upon upgrading the `pacman-mirrorlist` package.
```
sudo mkdir -p /etc/pacman.d/hooks
```

Create a new file in `/etc/pacman.d/hooks/mirrorupgrade.hook` and paste the following code:
```
[Trigger]
Operation = Upgrade
Type = Package
Target = pacman-mirrorlist

[Action]
Description = Updating pacman-mirrorlist with reflector and removing pacnew...
When = PostTransaction
Depends = reflector
Exec = /usr/bin/env sh -c "reflector --country 'Netherlands' --latest 10 --protocol https --age 12 --sort rate --save /etc/pacman.d/mirrorlist; if [[ -f /etc/pacman.d/mirrorlist.pacnew ]]; then rm /etc/pacman.d/mirrorlist.pacnew; fi"
```

Now every time the `pacman-mirrorlist` package gets upgraded (ie. new mirrors get added) we use `reflector` to test the mirror speed and protocol to obtain a new ordered mirrorlist.

NOTE: The above script depends on `reflector` so be sure to have it installed as described above.

### Terminal preference
See the [zsh n00b guide](http://eli5.it/linux/tooling/zsh)


### Auto mounting of `/media/data` and `/media/games`.
First make the folders to mount to:
```
sudo mkdir -p /media/data
sudo mkdir -p /media/games
```

#### Find the proper UUIDs:
```
sudo blkid | grep sdb1  
sudo blkid | grep sdc1
```

#### Then add the UUID of desired drive to the fstab like so:
```
UUID=<your-uuid> /media/data ext4 defaults 0 1
UUID=<your-uuid> /media/games ntfs-3g defaults,discard 0 1
```

### Installing steam native runtime:
```
sudo pacman -S steam steam-native-runtime
```

---

## Device specifics and troubleshooting

### Macbook video driver:
```
sudo pacman -S xf86-video-intel
```

### Use NetworkManager instead of netctl
During installation we enabled the netctl-auto configuration in order to have access to wireless internet post-install. This is because life after install is just a terminal prompt. After installing a GUI though you probably would like to start using a graphical network manager. In this case we're going to assume you installed GNOME.

```
sudo systemctl disable netctl-auto@<YOUR_INTERFACE>.service
```

```
sudo systemctl enable NetworkManager.service
```

Simply reboot and enjoy your new NetworkManager

---

## DOCUMENT HISTORY:
***12-12-2018***

- I no longer recommend to install the synaptics-touch driver since the built-in support now exceeds the performance and configuration options of the third-party integration. It just gets in the way of libinput.

- More logical bundling of package installations and some more consistent use of code blocks. One command per block is my stride.

***12-04-2018***

- I no longer encourage the use of MultiLib.

- Added a step to fetch and install the Aurman signature.

***09-25-2018***

- Added useful Pacman hook for testing the new entries in `pacman-mirrorlist` according to predefined criterea. Credits: [Tead](https://github.com/Tead).

***08-23-2018***

- Swapped out `Yaourt` in favor of `Aurman`. It is no longer recommended to use Yaourt as your package manager since it hasn't received updates or security review in a very long time.

- Added document history for transparency sake.
