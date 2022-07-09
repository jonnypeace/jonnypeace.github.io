---
title: SAS 4 x Drive Health Database
date: 2022-07-09 18:00:00 -500
categories: [mariadb,smartctl,smartmontools,database]
tags: [mariadb,smartctl,smartmontools,database]
---

# Purpose

If anyone else buys older drives on eBay, you'll soon realise the importance of keeping tabs on your hard drives, quickly. So, i created this to keep track of aging hard drives, but obviously a good idea to keep track with new hard drives also.

## Create Database

I will assume you know how to enter mariadb, if not, i might come back here and update.
Inside mariadb

Create database called health
```mysql
CREATE DATABASE health;
```

Grant permissions to a new user called hdd using a password of mypass (obviously you can use whatever password you desire, and obviously mypass is not a safe password)
```mysql
GRANT SELECT, INSERT,DELETE,UPDATE ON health.* TO hdd IDENTIFIED by 'mypass';
```

Navigate into the health database
```mysql
use health
```

Create a table with all the fields required (for my SAS drive script, this is what i'm looking at)
```mysql
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
	./health-hdd.sh -d /dev/sda (will show stats for that drive only)
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
	mariadb health -u hdd -e "SELECT * FROM (
	SELECT * FROM stats ORDER BY id DESC LIMIT $number 
	)Var1
	ORDER BY id ASC;"
}

# Shows only one specific drive
function drive_hdd {
	mariadb health -u hdd -e "SELECT * FROM stats WHERE hdd = '$drive_num'"
}

# Updates the database with new data from smartctl
function updater {

now=$(date "+%Y-%m-%d")

mapfile -t -d'|n' array1 < <(smartctl -a "$d1")
mapfile -t -d'|n' array2 < <(smartctl -a "$d2")
mapfile -t -d'|n' array3 < <(smartctl -a "$d3")
mapfile -t -d'|n' array4 < <(smartctl -a "$d4")

# grown defects
hd_grow_1=$(awk '/grown defect list/{print $6}' <<< "${array1[@]}")
hd_grow_2=$(awk '/grown defect list/{print $6}' <<< "${array2[@]}")
hd_grow_3=$(awk '/grown defect list/{print $6}' <<< "${array3[@]}")
hd_grow_4=$(awk '/grown defect list/{print $6}' <<< "${array4[@]}")

# health status
hd_health_1=$(awk '/SMART Health/{print $4}' <<< "${array1[@]}")
hd_health_2=$(awk '/SMART Health/{print $4}' <<< "${array2[@]}")
hd_health_3=$(awk '/SMART Health/{print $4}' <<< "${array3[@]}")
hd_health_4=$(awk '/SMART Health/{print $4}' <<< "${array4[@]}")

# non-medium errors (i.e. not the HDD)
hd_non_1=$(awk '/Non-medium error/{print $4}' <<< "${array1[@]}")
hd_non_2=$(awk '/Non-medium error/{print $4}' <<< "${array2[@]}")
hd_non_3=$(awk '/Non-medium error/{print $4}' <<< "${array3[@]}")
hd_non_4=$(awk '/Non-medium error/{print $4}' <<< "${array4[@]}")

# read error
hd_read_1=$(awk '/^read:/{print $8}' <<< "${array1[@]}")
hd_read_2=$(awk '/^read:/{print $8}' <<< "${array2[@]}")
hd_read_3=$(awk '/^read:/{print $8}' <<< "${array3[@]}")
hd_read_4=$(awk '/^read:/{print $8}' <<< "${array4[@]}")

# write errors
hd_write_1=$(awk '/^write:/{print $8}' <<< "${array1[@]}")
hd_write_2=$(awk '/^write:/{print $8}' <<< "${array2[@]}")
hd_write_3=$(awk '/^write:/{print $8}' <<< "${array3[@]}")
hd_write_4=$(awk '/^write:/{print $8}' <<< "${array4[@]}")

maria=$(which mariadb)

statement="INSERT INTO stats (Date,HDDdefect,NonMedium,HealthStatus,HDD,ReadErr,WriteErr) VALUES
('$now', '$hd_grow_1', '$hd_non_1', '$hd_health_1', '$d1', '$hd_read_1', '$hd_write_1'),
('$now', '$hd_grow_2', '$hd_non_2', '$hd_health_2', '$d2', '$hd_read_2', '$hd_write_2'),
('$now', '$hd_grow_3', '$hd_non_3', '$hd_health_3', '$d3', '$hd_read_3', '$hd_write_3'),
('$now', '$hd_grow_4', '$hd_non_4', '$hd_health_4', '$d4', '$hd_read_4', '$hd_write_4')"

$maria health -u hdd << EOF
$statement
EOF
if [[ $? == 0 ]]
then
	echo -e "\nData added successfully\n\n"
	$maria health -u hdd -e 'SELECT * from stats'
else
	echo "Something went wrong"
fi
}

while getopts ahl:d:u opt
do
	case "$opt" in
		a) all_health ;;
		h) helper ;;
		l) number=$2
		   last_lot ;;
		d) drive_num=$2
		   drive_hdd ;;
		u) updater ;;
		*) exit
	esac
done
```
