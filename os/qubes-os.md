# hardware without IOMMU

```
qvm-prefs --set sys-net virt_mode pv
qvm-prefs --set sys-usb virt_mode pv
```

## References

[Running Qubes OS 4.0.1 under KVM](https://www.reddit.com/r/Qubes/comments/b5cgc4/running_qubes_os_401_under_kvm/)

# Fedora

## ibus

```
echo '
# ibus
export GTK_IM_MODULE=ibus
export XMODIFIERS=@im=ibus
export QT_IM_MODULE=ibus
export QT4_IM_MODULE=ibus
ibus-daemon -d
' >> ~/.profile &&
ibus-setup
```

## Docker

```
echo '#Docker
systemctl start docker
chown user /var/run/docker.sock' | sudo tee -a /rw/config/rc.local &&
sudo mkdir -p /rw/config/qubes-bind-dirs.d &&
echo "binds+=( '/var/lib/docker' )
binds+=( '/etc/docker' )" | sudo tee /rw/config/qubes-bind-dirs.d/50_user_docker.conf &&
sudo systemctl start docker &&
sudo /usr/lib/qubes/init/bind-dirs.sh
```

# Debian

## Docker

```
echo '#Docker
chown user /var/run/docker.sock' | sudo tee -a /rw/config/rc.local &&
sudo mkdir -p /rw/config/qubes-bind-dirs.d &&
echo "binds+=( '/var/lib/docker' )
binds+=( '/etc/docker' )" | sudo tee /rw/config/qubes-bind-dirs.d/50_user_docker.conf &&
sudo /usr/lib/qubes/init/bind-dirs.sh
```

# Nix

```
sudo mkdir -p /rw/config/qubes-bind-dirs.d &&
echo "binds+=( '/nix' )" | sudo tee /rw/config/qubes-bind-dirs.d/50_user_nix.conf &&
sudo mkdir /nix &&
sudo /usr/lib/qubes/init/bind-dirs.sh &&
sudo chown -R user /nix &&
curl https://nixos.org/nix/install | sh
```
