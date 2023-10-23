# fedora-immutable-cheatsheet

A cheatsheet for immutable fedora variants, usually about ostree and rpm-ostree

## System files health checking

Finding all damaged objects. `--delete` flag will delete corrupted files and mark commits with damaged files as "partial", which then allow to redownload it.
```bash
ostree fsck -a
```

Current deployment status
```bash
rpm-ostree status
```

## Emergency fallback

### falling back to any previous commit of the system branch

1. Download last 5 commits
   
```bash
ostree pull --depth 5 fedora fedora/38/x86_64/kinoite
```

2. (Optionally) pin current deployment if there is chance of need to return to it. 0 is index of current deployment. to list deploymends run `rpm-ostree status` or `ostree admin status`

```bash
ostree admin pin 0
```

3. List downloaded commits to choose the best one, basing on the timestamp or anything elese

```bash
ostree log fedora/38/x86_64/kinoite
```

4. Deploy selected commit using its hash

```bash
rpm-ostree deploy 520e744c643b85fd14817a3eb948f200e7dec902cad4157e411dfeda2c6d7aab
```

5. Reboot into the new deoloyment (to see it before reboot, run `rpm-ostree status`)

6. (Optionally) unpin previously pinned deployment (id will be 1 if it is listed as the one before currently booted one)

```bash
ostree admin pin -u 1
```

### Rebase into different tree (also for switching flavors of the distro)

1. List trees of the ostree repository (fedora example)

```bash
ostree remote summary fedora
```

2. Rebase the system into the selected tree

```bash
rpm-ostree rebase fedora/39/x86_64/kinoite
```

3. reboot into the new deployment

## cleaning

Main tool for cleanup is `rpm-ostree cleanup` command. To see all options run:

```bash
rpm-ostree cleanup --help
```

Full cleanup (including all deployments other than currently deployed)
```bash
rpm-ostree cleanup -p -b -r -m
```

ostree also have cleaning tool

```bash
ostree prune
```

## Managing packages deployed with system

Adding package fzf to the (new) deployment

```bash
rpm-ostree install fzf
```

Removing packgage that was additionally installed

```bash
rpm-ostree remove fzf
```

Removing package that is in the system by default

```bash
rpm-ostree override remove firefox
```

Restore default package that was removed with override

```bash
rpm-ostree override reset firefox
```

## Other

### Allow system modification

Temporarily

```bash
rpm-ostree usroverlay --transient
````

Make the changes to `/usr` persist reboots

```bash
rpm-ostree usroverlay --hotfix
```
