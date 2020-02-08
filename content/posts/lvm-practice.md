---
title: "Lvm Practice on AWS EC2 and EBS"
date: 2020-02-07T14:21:21-06:00
draft: true
tags:
- LVM
- EC2
- AWS
- practice
- EBS
- AWS console
- beginner
- walkthrough

---
If you're like me, you may have a "use it or lose it" when it comes to skills. I don't want to lose the skills I've gained in the OCA, so here is a walk through of some LVM practice I did. You could use a VM software, but I used an AWS EC2 instance with an attached EBS volume for this practice in order to also keep up my skills while working in the console. Check out my other post on creating an EC2 instance if you need a refresher on how to do so. 

When I configured my EC2, I used Centos 7 instead of the Amazon Linux 2. It says there can be a charge, but it will be minimal, especially if we delete our instance when we are done. I used Centos 7 since it is closest to Redhat Linux, which is what we learned during our time in the OCA. I added extra storage to this instance as well, an extra volume. This extra volume is where we will modify our partition table. 

Once we configure and launch our instance, we will need to SSH into it. Once there, we do need to install LVM onto our machine. I switched to root, since most of the commands we do need root permissions to be executed. 
```
sudo -i
```
Then we will need to install LVM. I learned that this EC2 instance does not come equipped with it.
```
# yum install lvm2 -y
```
Once it's installed, let's take a look at what we have as far as storage already. 
```
# lsblk
    NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
    xvda    202:0    0   8G  0 disk 
    └─xvda1 202:1    0   8G  0 part /
    xvdb    202:16   0   8G  0 disk 
```
xvda is our root storage, so we will use xvdb when creating our partitions. Here is an example we can use:
>* Create a volume group called Office, with a physical extent size of 32M.
>* Create a logical volume called IT, within the Office volume group that is 50 extents in size and is formatted as ext4 and mounted at /IT.

First we need to determine the size of our partition.
>* Physical extent size of the Volume Group (VG) x logical volume (LV) extent size + 1 extra extent to account for wiggle room when building partitions. 

> **32M x 50 extents (+1) -> 51 extents = 1632M**

We will need this for our first partition. 

Now to create the 1st partition:
```
fdisk /dev/xvdb
```
There will be a welcome message, and then a command line waiting for your next entry. 
```
Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): 
Using default response p
Partition number (1-4, default 1): 
First sector (2048-16777215, default 2048): 
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-16777215, default 16777215): +1632M
Partition 1 of type Linux and of size 1.6fdis GiB is set
```
For the most part, you can click through the options after entering 'n' for new by typing the enter key. Be careful, once you get to the "Last sector" portion, this is where you enter the size. The '+' is important, as well as specifying K, M, or G. 

Before writing to the partition, we want to set the type of partition we're using. Use 't' for 'type.'
```
Command (m for help): t
Selected partition 1
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'
```
To see the changes we've made, let's use 'p' for 'print.' Once we're satisfied with the results, we can use 'w' to alter the partition table. 
```
Command (m for help): p

Disk /dev/xvdb: 8589 MB, 8589934592 bytes, 16777216 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x29eed0eb

    Device Boot      Start         End      Blocks   Id  System
/dev/xvdb1            2048     5490687     2744320   8e  Linux LVM

Command (m for help): w
The partition table has been altered!
```
Now to create our partition, volume group, and logical volume. You can always use --help after the first argument for additional options. 
```
# pvcreate /dev/xvdb1
    Physical volume "/dev/xvdb1" successfully created.
# vgcreate -s 32 Office /dev/xvdb1
    Volume group "Office" successfully created
# lvcreate -l 50 -n IT Office
    Logical volume "IT" created.
```
Another lsblk will show us the new partition table. 
```
# lsblk
    NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    xvda          202:0    0    8G  0 disk 
    └─xvda1       202:1    0    8G  0 part /
    xvdb          202:16   0    8G  0 disk 
    └─xvdb1       202:17   0  1.6G  0 part 
    └─Office-IT 253:0    0  1.6G  0 lvm  
```
To make our logical volume usable and available at boot time, we need to do a few more steps. Let's format our volume to use the ext4 filesystem.
```
# mkfs -t ext4 /dev/Office/IT
```
Now we want it to be available when we reboot our instance. This will require us to modify the /etc/fstab file. Let's make a copy of this file in our home directory first, just in case we break our machine we have a back up.
```
# cp /etc/fstab ~
```
Use the blkid command to find out what the UUID (universally unique identifier) for our logical volume is. It will be different for each person. Copy the identifier. Now let's open the etc/fstab file. I use VIM for this. (VIM will need to be installed on this instance since it does not come pre-installed.) 

```
vim /etc/fstab
```
Once opened, you will see an UUID already there, do not edit this one. Below is an example of me entering the information for our example. 
```
UUID=82587fd7-c0cd-4512-b0a2-db6b5bdd0302 /IT ext4 defaults 0 0
```
The /IT is the mount point and the ext4 is the file type. We need to create the mount point /IT and mount our volume to it.
```
# mkdir /IT
# mount -av
```
Mount -av = mount everything verbosely. One last lsblk will show us the final partition table.
```
# lsblk
    NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    xvda          202:0    0    8G  0 disk 
    └─xvda1       202:1    0    8G  0 part /
    xvdb          202:16   0    8G  0 disk 
    └─xvdb1       202:17   0  1.6G  0 part 
    └─Office-IT 253:0    0  1.6G  0 lvm  /IT
```
A final reboot will show us if everything works. Reboot our instance, and do lsblk again. If everything is still in place, we did it! I know this tutorial is a bit lengthy, but LVM is not always a simple topic. I wanted to be as thorough as I could for being a beginning myself, and hopefully this helped refresh your memory on some simple LVM. 

