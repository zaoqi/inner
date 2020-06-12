# Installation

https://wiki.parabola.nu/Installation_Guide
```
... # connect to the internet and configure partitions
pacman -Sy archlinux-keyring archlinuxarm-keyring parabola-keyring
pacman -U https://www.parabola.nu/packages/core/i686/archlinux32-keyring-transition/download/
pacstrap /mnt base elogind linux-libre-lts
genfstab -p /mnt >> /mnt/etc/fstab
arch-chroot /mnt
pacman -S nano
ln -sf /usr/share/zoneinfo/$Region/$City /etc/localtime
nano /etc/locale.gen # Uncomment `en_US.UTF-8 UTF-8` in /etc/locale.gen
locale-gen
echo 'LANG=en_US.UTF-8' > /etc/locale.conf
#echo 'hostname="localhost"' > /etc/conf.d/hostname # by default
cat << 'EOF' >> /etc/hosts
127.0.0.1 localhost
::1 localhost
EOF
nano /etc/pacman.conf # enable nonsystemd
pacman -S sudo
useradd -m user
passwd user
echo 'user ALL=(ALL) ALL' > /etc/sudoers.d/user
chmod 440 /etc/sudoers.d/user
passwd # root's password is "" by default!
... # configure bootloader
umount -R /mnt
reboot
```

# Partitions

## Utils
https://wiki.parabola.nu/LVM
```
modprobe dm-mod
vgscan
vgchange -ay
pacman -Sy cryptsetup
```

```
fdisk /dev/sda # sda1: EFI(512 M, type: EFI System) sda2: rest
cryptsetup luksFormat --label=parabola_os /dev/sda2
cryptsetup open /dev/sda2 cryptlvm
pvcreate /dev/mapper/cryptlvm
vgcreate CryptVolGrp /dev/mapper/cryptlvm
lvcreate -L 8G CryptVolGrp -n swap
lvcreate -l 100%FREE CryptVolGrp -n root
mkswap /dev/CryptVolGrp/swap
swapon /dev/CryptVolGrp/swap
mkfs.btrfs /dev/CryptVolGrp/root
mount -o compress=zstd /dev/CryptVolGrp/root /mnt
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
umount /mnt
mount -o compress=zstd,subvol=@ /dev/CryptVolGrp/root /mnt
mkdir /mnt/home
mount -o compress=zstd,subvol=@home /dev/CryptVolGrp/root /mnt/home
mkdir /mnt/boot
mkfs.fat -F32 /dev/sda1
mount /dev/sda1 /mnt/boot
... # install parabola
arch-chroot /mnt
pacman -S lvm2 btrfs-progs
nano /etc/mkinitcpio.conf # HOOKS: insert `encrypt` and `lvm2` between `block` and `filesystems`; `keyboard` before `encrypt` # BINARIES=("/usr/bin/btrfs")
mkinitcpio -P
```

# Grub

```
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
nano /etc/default/grub # add the following kernel parameter: `cryptdevice=LABEL=parabola_os:cryptlvm`
grub-mkconfig -o /boot/grub/grub.cfg
```
