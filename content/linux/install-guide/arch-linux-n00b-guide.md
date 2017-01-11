+++
date = "2017-01-10T00:14:35+01:00"
title = "arch linux n00b guide"
draft = false
categories = ["Linux"]
series = ["Install guides"]

+++

![Arch Linux](/img/arch-linux-desktop.png)

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
**TIP:** Leave a 128MiB empty ‘gap’ partition in between your ‘other os’ and your new partition.

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

Enabling Multilib and AUR repos
```
nano /etc/pacman.conf
```

Scroll down and uncomment these lines:

`[multilib]
Include = /etc/pacman.d/mirrorlist`


And add these lines at the bottom of the file:  
`[archlinuxfr]`  
`SigLevel = Never`  
`Server = https://repo.archlinux.fr/$arch`

Update the repos's
```
pacman -Syu
```

### User Management
#### Set root password

```
passwd
```

#### Create user:

```
useradd -m -g users -G wheel,storage,power -s /bin/bash <user>
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
ip link
systemctl enable netctl-auto@wlp2s0b1
```

Installing the bootloader:
```
mount -t efivarfs efivarfs /sys/firmware/efi/efivars
```

```
bootctl install
```

Write long UUID to file for later use
```
blkid | grep sda2 | cut -f2 -d\" &gt; /boot/loader/entries/arch.conf
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
 sudo mkinitcpio -p linux
 ```

### Finalize:
```
exit
umount -R /mnt
reboot
```

On laptops with touchpad:

```
sudo pacman -S xf86-input-synaptics
```

Install graphics driver and display server
```
sudo pacman -S mesa
sudo pacman -S xorg-server xorg-server-utils xorg-xinit xorg-twm xorg-xclock
```

### Installing and enabling a Firewall
```
sudo pacman -S ufw
sudo ufw enable
```

### Installing and enabling OpenSSH-Server
```
sudo pacman -S openssh
sudo systemctl start sshd.socket
```

#### If you want to have SSH running as as service run this
```
sudo systemctl enable sshd.socket
```

#### If needed make sure the SSH connection get's through the Firewall
```
sudo ufw allow 22
```

### Install awesome GUI stuff
```
sudo pacman -S gnome
sudo pacman -S gdm
sudo systemctl enable gdm.service
sudo pacman -S gnome-tweak-tool gnome-keyring
sudo pacman -S yaourt
```

Install some funky themes
```
sudo pacman -S arc-gtk-theme
```

Install some awesome packages
```
sudo pacman -S file-roller firefox nodejs npm vlc keepass vim git xdotool xsel
```

---

## Macbook Specific:
```
sudo pacman -S xf86-video-intel
```

---

## After install stuff [Optional]:

### Terminal preference
See the `zsh n00b guide`


### Auto mounting of `/media/data` and `/media/games`.
First make the folders to mount to:
```
Sudo mkdir -p /media/data
Sudo mkdir -p /media/games
```

#### Find the proper UUIDs:
```
ls -l /dev/disk/by-uuid
```

#### Then add the UUID of desired drive to the fstab like so:
```
UUID=<your-uuid> /media/data ext4 defaults 0 1
UUID=<your-uuid> /media/games ext4 defaults 0 1
```

### Installing steam native runtime:
```
sudo pacman -S steam steam-native-runtime
```

### Link Keepass plugins:
```
sudo cp .mozilla/firefox/5i95pb8t.default/extensions/keefox@chris.tomlinson/deps/KeePassRPC.plgx /usr/share/keepass/Plugins/
```
