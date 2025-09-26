---
title: "Setting up an HP Printer on Arch Linux"
date: 2024-02-10 13:00
categories: [Arch Linux, Printers, CUPS, HP]
tags: [HP, Printer, Arch Linux, CUPS]
---

Installing a printer on Linux can sometimes be a straightforward task, but it becomes interesting when you're working with specific hardware and distributions. Today, I'm sharing my experience with setting up an HP Printer on Arch Linux, which required a slightly different approach than usual. This guide is tailored for the HP Envy 5000 series, but it may be helpful for other models and series as well.

## Prerequisites

Before we dive into the installation process, make sure your system is up to date. It's a good practice to update your package database and upgrade your system packages before installing new software.

```bash
sudo pacman -Syu
```

## Installing Required Packages

The first step involves installing several packages that are necessary for printer setup and operation. While some of these packages might not be essential for everyone, they were useful during my diagnostic checks. Here's what you need to install:

```bash
sudo pacman -S python-pyqt5 xsane dbus python-pillow python-reportlab cups hplip
```

## Enabling and Starting CUPS

CUPS (Common UNIX Printing System) is crucial for printing on Linux. After installing the necessary packages, you'll need to enable and start the CUPS service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now cups
```

## Adding the Printer

With CUPS running, the next step is to add your printer to the system. The hp-setup command will guide you through finding and adding your HP printer:

```bash
hp-setup
```

## Verifying the Installation

To ensure that your printer has been installed correctly and is recognized by your system, run the following command:

```bash
hp-info -i
```

If everything is configured correctly, you should now have a working HP printer on your Arch Linux system.

## Conclusion

Setting up an HP printer on Arch Linux might require a few extra steps, but the process is straightforward once you know what packages are needed and how to enable the necessary services. I hope this guide helps you get your HP printer up and running smoothly on Arch Linux.