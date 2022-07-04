---
title: DPKG get-selections
date: 2022-07-02 18:00:00 -500
categories: [ubuntu,dpkg,automation,dselect]
tags: [dpkg,get-selections,dselect]
---
# Quick way to get a copy of your debain packages and reinstall

* Export installed packages

```bash
dpkg --get-selections > dpkg.list
```
* Update the cache

```bash
sudo apt update
```

* Install dselect

```bash
sudo apt install dselect
```

* Update dselect

```bash
sudo dselect update
```

* Set selections

```bash
sudo dpkg --set-selections < dpkg.list
```

* Install from dpkg.list

```bash
sudo apt-get dselect-upgrade
```
