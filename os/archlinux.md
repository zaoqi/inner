# Installation
[Installation guide - ArchWiki](https://web.archive.org/web/20200107092239/https://wiki.archlinux.org/index.php/Installation_guide)
```
... # Connect to the internet
timedatectl set-ntp true
... # configure partitions
pacstrap /mnt base base-devel nano linux linux-firmware
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt
ln -sf /usr/share/zoneinfo/$Region/$City /etc/localtime
hwclock --systohc
nano /etc/locale.gen # Uncomment `en_US.UTF-8 UTF-8` in /etc/locale.gen
locale-gen
echo 'LANG=en_US.UTF-8' > /etc/locale.conf
echo localhost > /etc/hostname
# 127.0.0.1	localhost
# ::1		localhost
nano /etc/hosts
passwd --lock root
useradd -m user
passwd user
echo 'user ALL=(ALL) ALL' > /etc/sudoers.d/user
chmod 440 /etc/sudoers.d/user
... # configure bootloader
umount -R /mnt
reboot
```
[Microcode - ArchWiki](https://web.archive.org/web/20200107093945/https://wiki.archlinux.org/index.php/Microcode)
```
pacman -S intel-ucode
```
[Btrfs subvolumes with swap](https://web.archive.org/web/20200107092204/https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#Btrfs_subvolumes_with_swap)
[LVM_on_LUKS](https://web.archive.org/web/20200107092204/https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#LVM_on_LUKS)
[GRUB - ArchWiki](https://web.archive.org/web/20200107094103/https://wiki.archlinux.org/index.php/GRUB)
```
fdisk /dev/sda # sda1: EFI(512 M, type: EFI System) sda2: rest
mkfs.fat -F32 /dev/sda1
cryptsetup luksFormat /dev/sda2
cryptsetup open /dev/sda2 cryptlvm
pvcreate /dev/mapper/cryptlvm
vgcreate CryptVolGrp /dev/mapper/cryptlvm
lvcreate -L 8G CryptVolGrp -n swap
lvcreate -l 100%FREE CryptVolGrp -n root
mkswap /dev/CryptVolGrp/swap
swapon /dev/CryptVolGrp/swap
mkfs.btrfs /dev/CryptVolGrp/root
mount -o compress=zstd /dev/CryptVolGrp/root /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
btrfs subvolume create /mnt/home
... # install Arch
arch-chroot /mnt
pacman -S lvm2 btrfs-progs
nano /etc/mkinitcpio.conf # HOOKS: insert `encrypt` and `lvm2` between `block` and `filesystems`; `keboard` before `encrypt` # BINARIES=("/usr/bin/btrfs")
mkinitcpio -P

pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
# ls -lh /dev/disk/by-uuid/ | grep sda2 | awk '{print $9;}' >> /etc/default/grub # For the convenience of editing
nano /etc/default/grub # add the following kernel parameter: `cryptdevice=UUID=${device-UUID}:cryptlvm` # Ctrl+K: cut this line;Ctrl+U: paste
grub-mkconfig -o /boot/grub/grub.cfg
```

# DE for touchscreen

```
pacman -S gnome gnome-extra network-manager-applet
systemctl enable gdm
systemctl enable NetworkManager
```

## References

[GNOME - ArchWiki](https://web.archive.org/web/20200107092532/https://wiki.archlinux.org/index.php/GNOME)
[GDM - ArchWiki](https://web.archive.org/web/20200107092753/https://wiki.archlinux.org/index.php/GDM)

# snapper
```
pacman -S snap-pac rsync snapper
snapper -c root create-config /
snapper -c home create-config /home
systemctl enable snapper-timeline.timer
systemctl enable snapper-cleanup.timer
cat << 'EOF' | tee /usr/share/libalpm/hooks/50_bootbackup.hook
[Trigger]
Operation = Upgrade
Operation = Install
Operation = Remove
Type = Package
Target = linux*

[Action]
Depends = rsync
Description = Backing up /boot...
When = PreTransaction
Exec = /usr/bin/rsync -a --delete /boot /.bootbackup
EOF
```

## References

[Snapper - ArchWiki](https://web.archive.org/web/20200107091522/https://wiki.archlinux.org/index.php/Snapper)
