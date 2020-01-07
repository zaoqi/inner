# snapper
```
sudo pacman -S snap-pac rsync snapper
sudo snapper -c root create-config /
sudo systemctl enable snapper-timeline.timer
sudo systemctl enable snapper-cleanup.timer
cat << 'EOF' | sudo tee /usr/share/libalpm/hooks/50_bootbackup.hook
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
