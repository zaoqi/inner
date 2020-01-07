# Installation

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
