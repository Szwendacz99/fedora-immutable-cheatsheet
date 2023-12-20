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

1. Download last 5 commits (parameter `--commit-metadata-only` is optional but makes things faster)
   
```bash
ostree pull --commit-metadata-only --depth 5 fedora fedora/38/x86_64/kinoite
```

2. (Optionally) pin current deployment if there is chance of need to return to it. 0 is index of current deployment. to list deploymends run `rpm-ostree status` or `ostree admin status`

```bash
ostree admin pin 0
```

3. List downloaded commits to choose the best one, basing on the timestamp or anything elese

```bash
ostree log fedora:fedora/38/x86_64/kinoite
```

4. Deploy selected commit using its hash (or version, like `38.20231008.0`)

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
rpm-ostree rebase fedora:fedora/39/x86_64/kinoite
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

## encrypted BTRFS on ssd-like disk maintenance/health

It is **very** important for performance to have trimming working. To test it
run the below to trim all mounted filesystems (can take long time, like over
10,20 minutes) (if root disk is not trimmed, it have discard not enabled at
some level):

```bash
fstrim -av
```

If trimming is not working, check the below:

- `/etc/crypttab` if it have discard set for root
- `discard=async`, `discard=sync`, or `discard` flags set in `/etc/fstab`
- `lsblk -o +DISC-GRAN,DISC-MAX` - the additional columns should have non-zero values
- `cryptsetup luksDump /dev/nvme0n1p3` - if flag `allow-discards` is set
(optional but best way to make cryptsetup allow discards by default)

Setting performance improving flags on luks volume:

```bash
cryptsetup refresh \
    --allow-discards \
    --perf-no_read_workqueue \
    --perf-no_write_workqueue \
        --persistent \
            /dev/mapper/<disk>
```

Arch wiki about workqueues: https://wiki.archlinux.org/title/Dm-crypt/Specialties#Disable_workqueue_for_increased_solid_state_drive_(SSD)_performance

Recommended flags for btrfs volume mounting (compression flag optional, I guess):

```
noatime,space_cache=v2,discard=async,compress-force=zstd:1
```

If for some reason `space_cache` is not set properly on volume mount, do this
, (fs need to be unmounted):

- clear any v1 and v2 space_cache
- mount with space_cache=v2 flag
- check if flag is set correctly (maybe also create some file there?)

```bash
btrfs check --clear-space-cache v1 /dev/mapper/<device>
btrfs check --clear-space-cache v2 /dev/mapper/<device>

mount -o space_cache=v2 /dev/mapper/<device> /mnt
mount | grep space_cache
echo "hello" > /mnt/some/path/file.txt
umount /mnt
```
