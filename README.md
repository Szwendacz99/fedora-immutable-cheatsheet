# fedora-immutable-cheatsheet
A cheatsheet for immutable fedora variants, usually about ostree and rpm-ostree
## system files health checking
Finding all damaged objects. `--delete` flag will delete corrupted files and mark commits with damaged files as "partial", which then allow to redownload it.
```bash
ostree fsck -a
```

current deployment status
```bash
rpm-ostree status
```

## todo
```bash
ostree log fedora/38/x86_64/kinoite
ostree pull --depth 2 fedora fedora/38/x86_64/kinoite
ostree pull fedora fedora/38/x86_64/kinoite
ostree admin pin 0
ostree admin pin -u 1
rpm-ostree rebase fedora/38/x86_64/kinoite
rpm-ostree cleanup -p -b -r -m
rpm-ostree usroverlay
rpm-ostree deploy 520e744c643b85fd14817a3eb948f200e7dec902cad4157e411dfeda2c6d7aab
ostree prune
```
