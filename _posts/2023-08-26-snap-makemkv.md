---
title: MakeMKV install using snap
date: 2023-08-26 13:00
categories: [homelab, archive, media, HTPC]
tags: [home, archive, blu-ray, dvd, media, htpc]
---

# The why?

This is a useful programme for archiving collections of your blu-rays or DVDs, and for using with applications like jellyfin, plex etc.

## Very Easy Instructions using snaps

The latest/stable version is too old, so you need to install from latest/edge.

```bash
sudo snap install makemkv --channel=latest/edge
```

And then all that is left to do is allow snap access to your dvd/blu-ray drive

```bash
sudo snap connect makemkv:optical-write :optical-drive
```

To complete the installation, makemkv offer a free beta license, or you can support their cause. Link below...

[MakeMKV-Beta-License](https://forum.makemkv.com/forum/viewtopic.php?t=1053)
