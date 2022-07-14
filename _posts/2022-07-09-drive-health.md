---
title: SAS 4 x Drive Health Database
date: 2022-07-09 18:00:00 -500
categories: [mariadb,smartctl,smartmontools,database]
tags: [mariadb,smartctl,smartmontools,database]
---

# Purpose

If anyone else buys older drives on eBay, you will soon realise the importance of keeping tabs on your hard drives, quickly. So, i created this to keep track of aging hard drives, but obviously a good idea to keep track with new hard drives also.

## Create Database

I will assume you know how to enter mariadb, if not, i might come back here and update.
Inside mariadb

Create database called health
```sql
CREATE DATABASE health;
```

Grant permissions to a new user called hdd using a password of mypass (obviously you can use whatever password you desire, and obviously mypass is not a safe password)
```sql
GRANT SELECT, INSERT,DELETE,UPDATE ON health.* TO hdd IDENTIFIED by 'mypass';
```

Navigate into the health database
```sql
use health
```

Create a table with all the fields required (for my SAS drive script, this is what i'm looking at)
```sql
CREATE TABLE stats (id INT NOT NULL AUTO_INCREMENT, Date DATE NOT NULL, HDDdefect INT NOT NULL, NonMedium INT NOT NULL, HealthStatus VARCHAR(20) NOT NULL, ReadErr INT NOT NULL, WriteErr INT NOT NULL, HDD VARCHAR(15) NOT NULL, primary key (id));
```

## The script

```bash
#!/bin/bash

# Author: Jonny Peace, jonnypeace@outlook.com

d1="/dev/sda"
d2="/dev/sdb"
d3="/dev/sdc"
d4="/dev/sdd"

drive_list="$d1 $d2 $d3 $d4"

function helper {
	echo "Usage:

	This script is for 4 x SAS drives, but might update this in future for SATA and
	more dynamic number of drives.

	./health-hdd.sh -h (this helper)
	./health-hdd.sh -u (run the updater, and update the database)
	./health-hdd.sh -d /dev/sda 4 (will show stats for last 4 drive entries for /dev/sda)
	./health-hdd.sh -a (will show the entire table organised by drive)
	./health-hdd.sh -l 4 (will show the last 4 entries)"
}

# Shows all health, drives separated
function all_health {
	for i in $drive_list
	do
		mariadb health -u hdd -e "SELECT * FROM stats WHERE hdd = '$i'"
		echo
	done
}

# Shows the last entries, defined by a number on the command line
function last_lot {
	for i in $drive_list
	do
		mariadb health -u hdd -e "SELECT * FROM (
		SELECT * FROM stats WHERE hdd = '$i' ORDER BY id DESC LIMIT $number 
		)Var1
		ORDER BY id ASC;"
	done
}

# Shows only one specific drive
function drive_hdd {
	if [[ $args == 3 ]]
	then
		mariadb health -u hdd -e "SELECT * FROM (
		SELECT * FROM stats WHERE hdd = '$drive_num' ORDER BY id DESC LIMIT $number
		)Var1
		ORDER BY id ASC;"
	elif [[ $args == 2 ]]
	then
		mariadb health -u hdd -e "SELECT * FROM stats WHERE hdd = '$drive_num'"
	else
		helper
	fi
}

# Updates the database with new data from smartctl
function updater {

now=$(printf '%(%Y-%m-%d)T\n')
maria=$(which mariadb)

for i in $drive_list
do
	mapfile -t -d'\n' array < <(smartctl -a "$i")
	# grown defects
	hd_grow=$(awk '/grown defect list/{print $6}' <<< "${array[@]}")

	# health status
	hd_health=$(awk '/SMART Health/{print $4}' <<< "${array[@]}")

	# non-medium errors (i.e. not the HDD)
	hd_non=$(awk '/Non-medium error/{print $4}' <<< "${array[@]}")

	# read error
	hd_read=$(awk '/^read:/{print $8}' <<< "${array[@]}")

	# write errors
	hd_write=$(awk '/^write:/{print $8}' <<< "${array[@]}")

	# serial
	serial=$(awk '/Serial number:/{print $3}' <<< "${array[@]}")

	statement="INSERT INTO stats (Date,HDDdefect,NonMedium,HealthStatus,HDD,ReadErr,WriteErr,Serial) VALUES
	('$now', '$hd_grow', '$hd_non', '$hd_health', '$i', '$hd_read', '$hd_write', '$serial')"

	$maria health -u hdd << EOF
	$statement
EOF
	if [[ $? == 0 ]]
	then
		echo -e "\nData added successfully for $i\n\n"
		$maria health -u hdd -e "SELECT * FROM stats WHERE hdd = '$i'"
	else
		echo "Something went wrong"
	fi
done
}

while getopts ahl:d:2u opt
do
	case "$opt" in
		a) all_health ;;
		h) helper ;;
		l) number=$2
		   last_lot ;;
		d) drive_num=$2
		   args=$#
		   number=$3
		   drive_hdd ;;
		u) updater ;;
		*) helper
	esac
done

```
