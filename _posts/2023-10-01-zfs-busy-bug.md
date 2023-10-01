---
title: ZFS Import Busy Dataset
date: 2023-10-01 17:00
categories: [homelab, lvm, zfs]
tags: [homelab, lvm, zfs, dataset, busy, import]
---

## What happened?

OS: Ubuntu 22.04 server

Well, after importing a zpool that was in use on another server (switching servers), I was unable to destroy the pool or dataset, faced with a busy dataset/pool error. I did notice something however, which helped with the trail.

Even though there was no mounts, it was busy, and i believe it was LVM making use of it...

```bash
zd32                      230:32   0    60G  0 disk
├─zd32p1                  230:33   0     1M  0 part
├─zd32p2                  230:34   0     1G  0 part
└─zd32p3                  230:35   0    59G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0  29.5G  0 lvm
```

So on my google search, i found someone who used lvm filter strings...

[Destroy ZFS Pool](https://forum.proxmox.com/threads/cant-destroy-zvol-from-pool-dataset-is-busy-solution-lvm-picked-up-on-vg-pv-inside-need-filter-in-lvm-conf.80877/#post-357562)

If you edit the /etc/lvm/lvm.conf and if in vim, search for global_filter, you'll find the section quick enough to place this custom filter.

```bash
    ### CUSTOM START
    filter = [ "r|^/dev/zd*|" ]
    global_filter = [ "r|^/dev/zd*|" ]
    ### CUSTOM END
```

After this, update initramfs.

```bash
update-initramfs -u
```

And reboot.

I only did this to destroy the ZFS dataset and pool. Afterwards I commented out the custom filter just to make sure it didn't interfere with anything, but something worth remembering.

```bash:w

zfs destroy -r fastZFS
zpool destroy fastZFS
```

