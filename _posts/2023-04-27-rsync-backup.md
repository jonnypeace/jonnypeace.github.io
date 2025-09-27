---
title: Automate rsync backups
date: 2023-04-27 20:00
categories: [homelab, rsync, backup, automate, health]
tags: [rsync, backup, smartmontools]
published: false
---

## The why..

Fairly straight forward backup script, could be made even easier if you don't want to check drive integrity first.

This script does..

* checks smartmontools for a few attributes in case of failing drive
* keeps two drives in sync, using rsync. Both drives are called and located at /3tb1 and /3tb1
* rsync uses the --backup-dir= flag, so any changes to files are not completely wiped, the old files will be placed in this directory

I think that's about it, nothing too tricky. I have used a version of this script for several years and it's worked well.

I generally use this in cron for daily backups. Unlike my incremental tar archive backups, this one can be run at any time, whereas my tar backup solution is hardcoded for daily backups, but I am thinking of making a change to this.

```bash
#!/bin/bash

# Create an array from smartmontools for reference of /dev/sda
mapfile -t sda_health < <(sudo smartctl -a /dev/sda)

# declare associative arrays
declare -A sda sdb

# assign associative array with health info
sda[pass]=$(awk '/overall-health/{print $6}' < <(printf '%s\n' "${sda_health[@]}"))
sda[pend]=$(awk '/Current_Pending_Sector/{print $10}' < <(printf '%s\n' "${sda_health[@]}"))
sda[reall]=$(awk '/Reallocated_Event_Count/{print $10}' < <(printf '%s\n' "${sda_health[@]}"))
sda[uncor]=$(awk '/Offline_Uncorrectable/{print $10}' < <(printf '%s\n' "${sda_health[@]}"))

# Create an array from smartmontools for reference of /dev/sda
mapfile -t sdb_health < <(sudo smartctl -a /dev/sdb)

# assign associative array with health info
sdb[pass]=$(awk '/overall-health/{print $6}' < <(printf '%s\n' "${sdb_health[@]}"))
sdb[pend]=$(awk '/Current_Pending_Sector/{print $10}' < <(printf '%s\n' "${sdb_health[@]}"))
sdb[reall]=$(awk '/Reallocated_Event_Count/{print $10}' < <(printf '%s\n' "${sdb_health[@]}"))
sdb[uncor]=$(awk '/Offline_Uncorrectable/{print $10}' < <(printf '%s\n' "${sdb_health[@]}"))

# Test mounts for both /dev/sda and sdb
testmount1=$(df -h | awk '/\/3tb1/{print $6}')
testmount2=$(df -h | awk '/\/3tb2/{print $6}')

# if both drives are mounted, -n implies the variables contain a string...
if [[ -n $testmount1 && -n $testmount2 ]] ; then

  # Test health conditions for both drives... if drive is in poor health, dont sync.
  # Allow user intervention to investigate data integrity and replace faulty drive.
  
  if [[ ${sda[pass]} != PASSED || ${sda[pend]} != 0 || ${sda[reall]} != 0 || ${sda[uncor]} != 0 ||
          ${sdb[pass]} != PASSED || ${sdb[pend]} != 0 || ${sdb[reall]} != 0 || ${sdb[uncor]} != 0 ]] ; then

    echo "Check drive health, might be time to replace one of the 3tb1 drives" 
    exit 1
  else
    echo "Syncing drives..."
    rsync -avhrxH --delete --backup-dir=/3tb1/backup /3tb1/ /3tb2/
  fi
else
  echo "Check drives are mounted correctly, or if drives have failed"
  exit 1
fi
```