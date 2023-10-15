---
title: Nvidia GPU LXD Passthrough
date: 2023-10-15 15:00
categories: [homelab, lxd, ubuntu, gpu, nvidia, jellyfin]
tags: [homelab, lxd, ubuntu, gpu, nvidia, jellyfin]
---

## On the host...

Install Nvidia drivers

```bash
sudo apt install nvidia-driver-535
sudo reboot
```

Check this website, but the deb network install seemed to work best.

[Nvidia Cuda Install Instructions](https://developer.nvidia.com/cuda-downloads)

As of 15/10/2023 These work for ubuntu 22.04 (deb(network) install)

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update
sudo apt-get -y install cuda
sudo reboot
```


## Launch LXD Container

```bash
lxc launch ubuntu:22.04 jelly-belly -c nvidia.runtime=true
```

## Add GPU to container

```bash
lxc config device add jelly-belly gpu gpu
```

## Add Extra Line to Config

```bash
lxc config edit jelly-belly
```

Include the top line alongside the runtime config.
```bash
nvidia.driver.capabilities: all
nvidia.runtime: "true"
```