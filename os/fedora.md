# updating

```
sudo dnf upgrade && sudo dnf remove $(dnf repoquery --installonly --latest-limit=-1 -q) && sudo dnf clean all
```
