# Installation
[Installation guide - ArchWiki](https://web.archive.org/web/20200107092239/https://wiki.archlinux.org/index.php/Installation_guide)
```
... # Connect to the internet
timedatectl set-ntp true
... # configure partitions
pacstrap /mnt base base-devel nano linux linux-firmware
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt
ln -sf /usr/share/zoneinfo/< Region >/< City > /etc/localtime
hwclock --systohc
locale-gen
nano /etc/locale.gen # Uncomment `en_US.UTF-8 UTF-8` in /etc/locale.gen
locale-gen
echo localhost > /etc/hostname
# 127.0.0.1	localhost
# ::1		localhost
nano /etc/hosts
passwd
useradd -m user
passwd user
echo 'user ALL=(ALL) ALL' > /etc/sudoers.d/user
chmod 440 /etc/sudoers.d/user
```
[Microcode - ArchWiki](https://web.archive.org/web/20200107093945/https://wiki.archlinux.org/index.php/Microcode)
```
pacman -S intel-ucode
```
[GRUB - ArchWiki](https://web.archive.org/web/20200107094103/https://wiki.archlinux.org/index.php/GRUB)
```
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```
[Btrfs subvolumes with swap](https://web.archive.org/web/20200107092204/https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#Btrfs_subvolumes_with_swap)

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
