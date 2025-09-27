---
title: Quick Terminal CPU temp
date: 2023-08-26 13:00
categories: [homelab, CPU, Temperature, Thermometer]
tags: [cpu, temperature]
published: false
---

# The why?

I've written many things, and this one is more just a quick way of documenting something i'll use for checking CPU temps while stressing the CPU.

This doesn't need any real great write up, it's a cpu temperature check.

Simple one liner for the Terminal...

```bash
while true; do clear; printf '%s %.2f %s %(%H:%M:%S)T \n' "Temperature is =" "$(( $(< /sys/class/thermal/thermal_zone*/temp) ))e-3" "deg C," ; sleep 2 ; done
```

