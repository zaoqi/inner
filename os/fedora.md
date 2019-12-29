# updating

```
sudo dnf upgrade && dnf repoquery --installonly --latest-limit=-1 -q | xargs -r sudo dnf remove && sudo dnf clean all
```
