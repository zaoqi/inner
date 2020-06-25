# Installation
[Installation guide - ArchWiki](https://web.archive.org/web/20200107092239/https://wiki.archlinux.org/index.php/Installation_guide)
```
... # connect to the internet and configure partitions
timedatectl set-ntp true
pacstrap /mnt base linux-lts linux-firmware
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt
pacman -S nano man-db man-pages
ln -sf /usr/share/zoneinfo/$Region/$City /etc/localtime
hwclock --systohc
nano /etc/locale.gen # Uncomment `en_US.UTF-8 UTF-8` in /etc/locale.gen
locale-gen
echo 'LANG=en_US.UTF-8' > /etc/locale.conf
echo localhost > /etc/hostname
cat << 'EOF' >> /etc/hosts
127.0.0.1 localhost
::1 localhost
EOF
passwd
pacman -S sudo
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
```
fdisk /dev/sda # sda1: EFI(512 M, type: EFI System) sda2: rest
cryptsetup luksFormat --label=arch_os /dev/sda2
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
btrfs subvolume create /mnt/@snapshots
btrfs subvolume create /mnt/@home@snapshots
umount /mnt
mount -o compress=zstd,subvol=@ /dev/CryptVolGrp/root /mnt
mkdir /mnt/.snapshots
mount -o compress=zstd,subvol=@snapshots /dev/CryptVolGrp/root /mnt/.snapshots
mkdir /mnt/home
mount -o compress=zstd,subvol=@home /dev/CryptVolGrp/root /mnt/home
mkdir /mnt/home/.snapshots
mount -o compress=zstd,subvol=@home@snapshots /dev/CryptVolGrp/root /mnt/home/.snapshots
mkdir /mnt/boot
mkfs.fat -F32 /dev/sda1
mount /dev/sda1 /mnt/boot
... # install Arch
arch-chroot /mnt
pacman -S lvm2 btrfs-progs
nano /etc/mkinitcpio.conf # HOOKS: insert `encrypt` and `lvm2` between `block` and `filesystems`; `keyboard` before `encrypt` # BINARIES=("/usr/bin/btrfs")
mkinitcpio -P
```
[GRUB - ArchWiki](https://web.archive.org/web/20200107094103/https://wiki.archlinux.org/index.php/GRUB)
```
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
# ls -lh /dev/disk/by-uuid/ | grep sda2 | awk '{print $9;}' >> /etc/default/grub # For the convenience of editing
nano /etc/default/grub # add the following kernel parameter: `cryptdevice=UUID=${device-UUID}:cryptlvm` # Ctrl+K: cut this line;Ctrl+U: paste # or: `cryptdevice=LABEL=arch_os:cryptlvm`
grub-mkconfig -o /boot/grub/grub.cfg
```
or ...
[systemd-boot - ArchWiki](https://web.archive.org/web/20200411100519/https://wiki.archlinux.org/index.php/Systemd-boot)
```
bootctl --path=/boot install
cat << 'EOF' > /usr/share/libalpm/hooks/90_USER_systemd-boot.hook
[Trigger]
Type = Package
Operation = Upgrade
Target = systemd

[Action]
Description = Upgrading systemd-boot ...
When = PostTransaction
Exec = /usr/bin/bootctl update
EOF
cat << 'EOF' > /boot/loader/loader.conf
timeout 4
console-mode max
default arch.conf
editor yes
EOF
cat << 'EOF' > /boot/loader/entries/arch.conf
title Arch Linux
linux /vmlinuz-linux
initrd /intel-ucode.img
initrd /initramfs-linux.img
options cryptdevice=LABEL=arch_os:cryptlvm root=/dev/CryptVolGrp/root rootflags=subvol=@ rw
EOF
```

# GNOME

```
pacman -S gnome
systemctl enable gdm
systemctl enable NetworkManager
systemctl enable bluetooth
```

## References

[GNOME - ArchWiki](https://web.archive.org/web/20200107092532/https://wiki.archlinux.org/index.php/GNOME)
[GDM - ArchWiki](https://web.archive.org/web/20200107092753/https://wiki.archlinux.org/index.php/GDM)

# KDE

```
pacman -S plasma-meta plasma-wayland-session
systemctl enable sddm
systemctl enable NetworkManager
systemctl enable bluetooth
```

## References

[KDE - ArchWiki](https://web.archive.org/web/20200418061958/https://wiki.archlinux.org/index.php/KDE)

# CJK Fonts and IME

```
sudo pacman -S noto-fonts-cjk ibus-rime
mkdir -p ~/.config/ibus/rime
cat << 'EOF' > ~/.config/ibus/rime/default.custom.yaml
patch:
  schema_list:
    - schema: terra_pinyin
EOF
# Optional: install plum
git clone --depth 1 https://github.com/rime/plum.git
cd plum
bash rime-install :preset
```

# snapper
```
pacman -S snap-pac rsync snapper
umount /.snapshots
rmdir /.snapshots
snapper -c root create-config /
rm -r /.snapshots
mkdir /.snapshots
mount /.snapshots
umount /home/.snapshots
rmdir /home/.snapshots
snapper -c home create-config /home
rm -r /home/.snapshots
mkdir /home/.snapshots
mount /home/.snapshots
systemctl enable snapper-timeline.timer
systemctl enable snapper-cleanup.timer
cat << 'EOF' | tee /usr/share/libalpm/hooks/50_USER_bootbackup.hook
[Trigger]
Operation = Upgrade
Operation = Install
Operation = Remove
Type = Package
Target = linux*
Target = ${PACKAGES SUCH AS grub}

[Action]
Depends = rsync
Description = Backing up /boot...
When = PreTransaction
Exec = /usr/bin/rsync -a --delete /boot /.bootbackup
EOF
```

## References

[Snapper - ArchWiki](https://web.archive.org/web/20200107091522/https://wiki.archlinux.org/index.php/Snapper)


# fcitx-googlepinyin

```
sudo pacman -S fcitx-googlepinyin fcitx-configtool fcitx-gtk2 fcitx-gtk3 fcitx-qt5
cat << 'EOF' > ~/.pam_environment
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
EOF
```

## References

[Fcitx - ArchWiki](http://web.archive.org/web/20200119051506/https://wiki.archlinux.org/index.php/fcitx)
