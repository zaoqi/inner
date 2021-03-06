# hardware without IOMMU (QubesOS R4.0)

```
qvm-prefs --set sys-net virt_mode pv
qvm-prefs --set sys-usb virt_mode pv
```

## References

[Running Qubes OS 4.0.1 under KVM](http://web.archive.org/web/20191123124538/https://www.reddit.com/r/Qubes/comments/b5cgc4/running_qubes_os_401_under_kvm/)

# Fedora30

## Remove unnecessary kernel in TemplateVM

```
sudo dnf remove kernel kernel-modules kernel-core kernel-debug-devel && sudo dnf install linux-firmware
```

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

## fcitx-googlepinyin

Share State Among Window = No

Use System Tray Icon = false

Status Panel Hide Mode = Hide

### archlinux

```
cat << 'EOF' > ~/.pam_environment
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
EOF
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

# Debian10

## Remove unnecessary kernel in TemplateVM

```
sudo apt purge cryptsetup-initramfs linux-image-4.19.0-6-amd64 linux-image-amd64 linux-headers-4.19.0-6-amd64 linux-headers-amd64 linux-headers-4.19.0-6-common linux-kbuild-4.19
```

## Docker

```
echo '#Docker
chown user /var/run/docker.sock' | sudo tee -a /rw/config/rc.local &&
sudo mkdir -p /rw/config/qubes-bind-dirs.d &&
echo "binds+=( '/var/lib/docker' )
binds+=( '/etc/docker' )" | sudo tee /rw/config/qubes-bind-dirs.d/50_user_docker.conf &&
sudo /usr/lib/qubes/init/bind-dirs.sh
```

## anbox(StandaloneVM)

```
sudo apt install snapd adb android-sdk-platform-tools-common
sudo snap install --devmode --beta anbox
echo deb http://ppa.launchpad.net/morphis/anbox-support/ubuntu disco main | sudo tee /etc/apt/sources.list.d/morphis-ubuntu-anbox-support.list
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 21C6044A875B67B7
sudo apt update && sudo apt install anbox-modules-dkms

## Upgrade system: ## sudo apt update && sudo apt full-upgrade && sudo apt autoremove && sudo apt clean && sudo snap refresh --beta --devmode anbox
```

## 麟卓

```
sudo apt update && sudo apt full-upgrade && sudo apt autoremove && sudo apt clean
wget -O - https://liquidtelecom.dl.sourceforge.net/project/xdroid/2.7/xDroidInstall-x86_64-v2.7000-20190621155253.tar.gz | tar -xzv
./xDroidInstall-x86_64/install.sh
rm -fr xDroidInstall-x86_64
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

# updating

## dom0

```
sudo qubes-dom0-update && sudo dnf remove $(dnf repoquery --installonly --latest-limit=-1 -q)
```


## debian/ubuntu

```
sudo apt update && sudo apt full-upgrade && sudo apt autoremove && sudo apt clean
```

## fedora

```
sudo dnf clean all && sudo dnf upgrade && sudo dnf remove $(dnf repoquery --installonly --latest-limit=-1 -q) && sudo dnf clean all
```
