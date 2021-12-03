# Install Guide

## Remote login

`pacman -S openssh`

passwd

ip addr

## Super basic stuff

```bash
timedatectl set-ntp true
timedatectl status
```

## Disk setup

```bash
modprobe dm-crypt
fdisk -l
gdisk /dev/nvme0n1
```

```
* o
```

`efi`

```
* n
* 1
* 2048
* +512M
* ef00

`root`

* n
* 2
* Return
* w
```
## Encrypt disks and create it

```bash
cryptsetup benchmark
cryptsetup -c aes-xts-plain64 -y -s 512 luksFormat /dev/nvme0n1p2
cryptsetup luksOpen /dev/nvme0n1p2 lvm
pvcreate /dev/mapper/lvm
vgcreate main /dev/mapper/lvm
lvcreate -L 32G -n swap main
lvcreate -l 100%FREE -n root main
mkswap /dev/mapper/main-swap
mkfs.ext4 /dev/mapper/main-root
mkfs.fat -F 32 /dev/nvme0n1p1
mount /dev/mapper/main-root /mnt
mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
swapon /dev/mapper/main-swap
```

## Instal basic packages

```bash
reflector --verbose --latest 5 --sort rate --save /etc/pacman.d/mirrorlist
pacstrap /mnt base base-devel linux linux-firmware lvm2 vim zsh git networkmanager
```

## chroot

```bash
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt
```

## Environment stuff

ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime

hwclock --systohc

en_US.UTF-8 UTF-8 in `/etc/locale.gen`

locale-gen

`vim /etc/locale.conf`

LANG=en_US.UTF-8

`vim /etc/hostname`
foobar

`vim /etc/hosts`

```bash
127.0.0.1       localhost
::1             localhost
127.0.0.1       mdlnx.localdomain     mdlnx
```

`vim /etc/mkinitcpio.conf`

```bash
HOOKS=(base udev autodetect modconf block keyboard keymap encrypt lvm2 resume filesystems fsck)
```

`mkinitcpio -P`

```bash
useradd -m lukas
passwd lukas
pacman -S sudo
echo "lukas      ALL = (ALL:ALL) ALL" > /etc/sudoers.d/lukas
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## bootloader

```bash
mount -t efivarfs efivarfs /sys/firmware/efi/efivars
bootctl --path=/boot install
pacman -S intel-ucode
```

`vim /boot/loader/entries/arch.conf`

```bash
title  Arch Linux
linux  /vmlinuz-linux
initrd  /initramfs-linux.img
options cryptdevice=UUID=<YOUR-PARTITION-UUID>:lvm:allow-discards resume=/dev/mapper/main-swap root=/dev/mapper/main-root rw quiet
```

`systemctl enable NetworkManager`

`vim /boot/loader/loader.conf`

```bash
timeout 0
default arch
editor 0
```

`exit`

```bash
umount /mnt/boot
umount /mnt
reboot
```

## yay

```bash
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

#### Make sure you have the Color option in your /etc/pacman.conf

## INTEL Graphics

pacman -S xf86-video-intel
/etc/mkinitcpio.conf
MODULES=(i915)
mkinitcpio -P

## Sway

`.zlogin`

```bash
if [[ -z $DISPLAY && "$(tty)" == "/dev/tty1" ]]; then
    export LIBSEAT_BACKEND=logind
    export WLR_NO_HARDWARE_CURSORS=1
    #export WLR_DRM_NO_MODIFIERS=1
    exec sway --my-next-gpu-wont-be-nvidia
fi
```

```bash
wlroots-eglstreams-git wlroots-git grimshot-git light-git mako-git networkmanager-dmenu-git rofi-git seatd-git sway-git swayidle-git swaybg-git swaylock-effects-git waypipe-git wf-recorder-git swaynagmode networkmanager-wireguard-git waybar-git bemenu-wayland bemenu aur/alacritty xcb-util-cursor
```

Copy sway config

`yay -S nerd-fonts-complete extra/gnome-keyring aur/visual-studio-code-bin`

Set timedatectl
