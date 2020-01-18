# updating

```
sudo dnf clean all && sudo dnf upgrade && sudo dnf remove $(dnf repoquery --installonly --latest-limit=-1 -q) && sudo dnf clean all
```
