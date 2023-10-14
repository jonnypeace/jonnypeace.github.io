---
title: Software Raid Boot Drive Ubuntu
date: 2023-10-14 17:00
categories: [homelab, raid, mdadm, ubuntu, boot]
tags: [homelab, raid, mdadm, ubuntu, boot]
---

## Ubuntu Software Raid on Boot Device

OS: Ubuntu 22.04 server

At Installation...

1. Select "Custom storage layout"
1. If the disks have existing partitions, click on each disk under AVAILABLE DEVICES and then select REFORMAT.
1. Now select the 1st disk to add as "boot" disk (same menu that had REFORMAT in).
1. Do the same with the 2nd disk.
1. You should now see two 1.000M bios_grub partitions created under USED DEVICES.
1. The trick to setup a softRAID array is to create partitions for /boot, swap and / on each disk, but WITHOUT formatting them (and as such, there won't be a mount point for now).
1. Lets create the boot partition. "Add GPT Partition" on the 1st disk, give it a 1G size and choose to leave it unformatted. Do the same for the 2nd disk.
1. Lets Create the swap partition. "Add GPT Partition" on the 1st disk, give it the same or half the size of your RAM and choose to leave it unformatted. Do the same for the 2nd disk.
1. Lets create the root partition. "Add GPT Partition" on the 1st disk, do not set a size (so it uses all available) and choose to leave it unformatted as with all the other partitions you created so far. Do the same for the 2nd disk. 
1. Now click on "Create software RAID (md)" under AVAILABLE DEVICES. We'll create the first softRAID partition (md0) by selecting the two 1G partitions. Click create.
1. Repeat the process for md1 and select the two swap partitions created earlier. Click create.
1. Repeat the process for md2 and select the two root partitions. Click create.
1. We now have 3 pairs of AVAILABLE DEVICES which will now format as the actual softRAID partitions. So select md0 and then "Add GPT Partition", format as EXT4 and mount on /boot.
1. Select md1 and then "Add GPT Partition", format as SWAP.
1. Select md2 and then "Add GPT Partition", format as EXT4 and mount on /.
1. All these mdX softRAID partitions will now appear under USED DEVICES and you are ready to proceed with Ubuntu's installation.
1. At the very bottom, you should now see "Done" enabled so hit it and proceed.
