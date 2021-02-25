---
layout: post
title:  "Using External Storage as Primary Storage for NextCloud"
date: February 24th, 2021
categories: technical
excerpt_separator: <!--more-->
---

While setting up my self-hosted NextCloud instance on a Raspberry Pi, I wasn't able to find instructions on how to use external storage as my primary storage for it. This post will try to explain the steps that I had to go through in order be able to do just that. The main things it required were: a linux-compatible file system with proper permissions, and being able to mount the external drive automatically.

<!--more-->

The first thing you need to do is ensure that it uses a linux compatible file system. That is a requirement because the folder needs to be given permissions to read and write as www-data user. Using the [ext4](https://wiki.archlinux.org/index.php/ext4) filesystem fits that requirement.

### Repartition and format your external drive

First, plug in your hard drive. To find the name of your device, you can use the `lsblk` command to get that information. In this example, I will be using `/dev/sda` as the name of the device. If the external drive will only be used as your primary NextCloud storage, you can partition your disk to have only one `ext4` partition that will use all the available space. In the case that you will sometimes use it with other operating systems, I would recommend creating a second partition with a filesystem that is more compatible other operating systems, such as `exFAT` or `ntfs`. 

> Warning: Make sure you don't have any existing data on the drive before repartitioning your disk, all the data will be wiped after completing these steps.

Using [parted](https://www.gnu.org/software/parted/manual/parted.html), set the partitioning standard to GPT and created the partition that will use 100% of the available storage space:
```
sudo parted /dev/sda mklabel gpt
sudo parted -a opt /dev/sda mkpart primary ext4 0% 100%
```
> `-a opt` sets the alignment-type as optimal 

After creating those partitions, format the partition with a filesystem.

```
sudo mkfs.ext4 -L new-volume-label /dev/sda1
```

> `-L new-volume-label` sets the volume label

### Mounting your external storage
Once you have the partitions ready to go, you have to ensure that the drive is permanently mounted on your Pi. Create a directory under `/mnt/` and mount the partition that you'll be using as main storage for your NextCloud instance. 

```
sudo mkdir /mnt/myexternalstorage
sudo mount /dev/sda1 /mnt/myexternaldrive
```

In order to mount it permanently, find the UUID of the device's partition you'll be using.

```
lsblk --fs
```
> `--fs` sets it to output info about filesystems 

With that UUID, add this line to the fstab file (found under `/etc/fstab`) to automate the process to mount partitions.
```
UUID=YOUR_UUID_HERE /mnt/myexternaldrive ext4 defaults,auto,users,rw,nofail 0 0
```
> replace `YOUR_UUID_HERE` with the one you found

 
### Setting it as your main Nextcloud storage
Create the directory that will host your primary NextCloud data folder. You can simply create a directory called `nextcloud-data`. Give that directory full ownership to the `www-data` user and ensure that `nextcloud-data` directory has proper permissions.

```
sudo chown -R www-data:www-data /mnt/myexternaldrive/nextcloud-data/
sudo chmod 750 /mnt/myexternaldrive/nextcloud-data/
```
> `-R` is used to operate on files and directories recursively

> `750` give the owner `read, write, and execute` permissions, the group `read and execute` permissions and others no (0) permissions.


During the initial NextCloud setup, you'll be asked to provide the directory you'll want use as your data directory. Enter your external storage's folder `/mnt/myexternaldrive/nextcloud-data`. You are now using your external storage as the main storage for your NextCloud instance. :) 

---

Resources used to help write this post:

1. [PiMyLifeUp - How to Setup a Raspberry Pi Nextcloud Server](https://pimylifeup.com/raspberry-pi-nextcloud-server/)
2. [DigitalOcean - How To Partition and Format Storage Devices in Linux](https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux)
3. [LinuxCommand - Permissions](https://linuxcommand.org/lc3_lts0090.php)
 
  
