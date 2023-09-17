---
title: Backup/Copy LXD container to another host
date: 2023-09-17 17:00
categories: [homelab, lxd, backup, copy, hosts, servers]
tags: [homelab, lxd, backup, hosts]
---

# The why?

Although I run LXD in clusters, my clusters are generally low powered that don't need much compute.

So i do run some services that require compute, on my laptop or desktop in LXD, and here I will show a simple way to copy over the LXD container to another host.

First you need to remote add and get a token, so in this example, lets assume my laptop is running a container and I want that container backed up.

### On my laptop...

I will generate a token.

```bash
lxc config trust add
```

### On my desktop...

I need to add my laptop..

```bash
lxc remote add laptop
```

When you paste in the token, no text will be displayed in the terminal, so don't be alarmed. Just paste it in when prompted and press enter. Now remote switch to the laptop from the desktop.

```bash
lxc remote switch laptop
```

And now Copy the container over from the desktop...

```bash
lxc copy container-name local:container-name-backup --verbose
```

You can then change back your remote to local on the desktop if you with...

```bash
lxc remote switch local
```

Hopefully helps someone... it'll help remind me in future... my memory needs help sometimes...