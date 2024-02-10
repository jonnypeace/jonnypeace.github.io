---
title: Setting up HP Printer Arch Linux
date: 2023-11-25 11:00
categories: [arch linux, printers, cups, hp]
tags: [hp, printer, arch, linux, cups]
---
 
I wouldn't normally write about something like this, but since the process was different from normal, i decided to write up something about this.

Packages I installed..

```bash
sudo pacman -S python-pyqt5 xsane dbus python-pillow python-reportlab cups hplip
```

Reason for some of these was during an diagnostic, some might not be necessary but they are here for now.

Enable cups...

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now cups
```

Run to Find and add printer...

```bash
hp-setup
```

Run this to make sure it installed...

```bash
hp-info -i
```

And we should have a working HP printer if all goes to plan. This works with the HP envy 5000 series.

