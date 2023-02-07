---
layout: post
title: "First things in new Proxmox Server"
date: 2023-02-04 09:00:00 +0200
categories: [proxmox]
tags: homelab proxmox 
---


After setting up my Proxmox Hypervisor server, there are a few things I do before I use them for their intended purpose.  This ranges from updates, to storage, to networking and VLANS, to uploading ISOs, to clustering, to NIC teaming with LACP, and more.\
This is the list that i have had that i floow every time i setup a new Proxmox server. Helps me to remember to do things conststently.

So lets begin.


 **1. Enable updates**

We could either get a subscription to Proxmox and get the most stable ones, or turn on the unstable ones. Although i found that installing the unstable ever since Proxmox vesion 6.1 has actually beenn suprisingly stable. I used to have pretty good luvk with them.

The first we have to do is ssh to ours server or use the shell from GUI.

Edit the sources.ist and add the no-productive lists

```shell
nano /etc/apt/sources.list
```
```shell
#not for production use
deb http://download.proxmox.com/debian/pve bullseye pve-no-subscription
```
ctr+x to exit and y + enter to save

Next edit the enterprise list and comment out the lists

```shell
nano /etc/apt/sources.list.d/pve-enterprise.list
```
```shell
#deb https://enterprise.proxmox.com/debian/pve bullseye pve-subscription
```
ctr+x to exit and y + enter to save

Then we could update our list and upgrade the packages
```shell
apt-get update
apt-get upgrade
```

If we want to upgrade the distribution version of server
```shell
apt dist-upgrade
```
 And reboot our Proxmox server.

 **2. Configuring Storage**

We just need to go to our server name and then go into disks and as we see here there are several disks dev/sda - dev/sde. First four dev/sda-dev/sdd are going yo be my staorage and lats one, dev/sde, is my proxmox server installation

![](/template/images/storage-1.JPG) 

So to create a new ZFS pool, first we have to clear the disks that contains partition information or the have been intilized for many reasons. So before create our zfs pool clear out all the information on those disks. SSH back into our server and we will run fdisk /dev/sda, once we are heare hit p to see the partition information and after hit d to delete partition and select all the partitions that shown before. Latest hit w to save and exit
These steps we fooled for all disks.

After we have done all that we can go back to the proxmox ui and choose zfs

![](/template/images/storage-2.JPG) 

- Name our ZFS pool
- Choose RAID type, i choose RAID 10 (or RAID1 + 0) for performance and redundancy
- Compression and shift we keep it to default from proxmox recommendations
- Chose all the disks

![](/template/images/storage-3.JPG)

Then click OK to create the pool.

 **3. Smart Monitoring**

This is something that checked to be sure that is enabled. If we go disks, we will make sure that smart monitoring is turned on
By-default this is turned-on but we might have accidentally turned it off

![](/template/images/smart-1.JPG)

With double-click on every one of that disks we could see the tests passed. Other way to check is from terminal.
 For example disk dev/sda

```shell
smartctl -a /dev/sda
```

**4. IOMMU or PCI passthrough**

this is so we can pass-though devices from the host machine to the guest machine, so that they can take advantages of that hardware.
Now there is a couple things to note before enable iommu. We nedd all the three below things to work.
- A processor that supports iommu
- A motherboard that support iommu
- Make sure that this is turned on in our BIOS

Note that the technology is named different for intel vs amd and a lot of motherboard manufacturers name this option different in the BIOS.
So we will want to check with our motherboard manufacturer how to turn this on.
Once we have donw all that we will ssh into our server and edit file grub

```shell
nano /etc/default/grub
```

```shell
#GRUB_CMDLINE_LINUX_DEFAULT="quiet"
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"

--if you use amd, change the intel string with amd
```
ctr+x to exit and y + enter to save

Then we want to run ``update-grub`` to turn on changes
Then we will want to edit our modules
```shell
nano /etc/modules
```
Add the following lines for vfio
```shell
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```
ctr+x to exit and y + enter to save

And reboot our Proxmox server.

**5. VLAN Aware**

Make our server vlan aware, use vlan tagging. This is a pretty easy setting.
So in order to do this we will go into netowrk, select out bridge interface, then we will check VLAN Aware box and save the settings.
Now our NIC is vlan ware and we can restirct this NIC to different vlans.

![](/template/images/network-1.JPG)
![](/template/images/network-2.JPG)

Also if you wanted to we could just edit via CLI.
```shell
nano /etc/network/interfaces
```
```shell
auto vmbr0
iface vmbr0 inet static
        address 192.168.0.13/24
        gateway 192.168.0.1
        bridge-ports enso1
        bridge-stp off
        bridge-fd 0
        bridge-vlan-aware yes
        bridge-vids 2-4094
```
ctr+x to exit and y + enter to save

If we want to restricted specific only vlans we could change ``bridge-vids`` to any vlans we want to, but I am leaving this open to any vlan and i will restrict them from network switch.

Now in our VM Network Device settings we will be able to choose vlan id of our choice

![](/template/images/network-3.JPG)

**6. Shares**

Add an NFS Share. We could use this share for backups, for ISOs, for a lot of things.
First we will have to have our NFS share already setup. We could use NAS of our choice, Synology, TrueNAS, WD-NAS or a share folder from PC with NFS enable.
To enable NFS share to our NAS we will want to check with our manufacturer how to turn this on.
For Proxmox we have to go at Datacenter level, we will go into storage and click Add -> NFS. Here we will give this an ID (Name the NFS Share), Server (IP address of NFS) and if NFS Share if set up correctly we should see in Export the Folder we have cretaed.Next we have to choose the content tyoe we want (Disk image,ISO images, Container templates etc.)
For NFS version if we want but keep it as default

![](/template/images/share-1.JPG)

Depend the choice of content type we will option to backup our VM or upload ISOs images for using them later.

**7. Schedule Backups**

Since we have a share set up already and even if we do not have a shared backup folder, we can do a local backup.
To schedule backups we will go to the Datacenter level, then we will go into backup option and click Add. So this is creating a backup job, and there we will choose:
* Our Node
* Our Storage to backup, where we store our backups
* Day of week and Start Time
* Whether or not to send you an email
* Whether or not sned an email notification - better option Always
* Compression level - better choice ZSTD(default) which is fastest and minimum effort
* Mode - better choice Snapshot
* Lastly the VM we want to backup

Now we have the backup job. After that we could create a intial backup.
We will go to VM, choose Backup and Backup now, choose Storage location, Mode, Compression and wheterh or not to send email after backup complete.

**8. VirtIO ISO**

If you are going to cretae Windows VM we have to uploading the KVM driver disk for Windows.
Whenever create a new windows virtual guest we always need to add a sepertae disk for drivers.
Download the latest one from here https://pve.proxmox.com/wiki/Windows_VirtIO_Drivers

Uploading them to our proxmox storage. Since version 7 and later we could download directly to proxmox server

![](/template/images/virtio.JPG)

Then we could upload all the ISOs we need as Windows, Linux even other images such as EVE-NG.

**9. NIC Team or LAG**

This is maybe pretty advanced but if we can configure Link-Aggregation is a very good plus point to our proxmox server. In order to do this we will have to have more than one nic.
The reason to do this is for getting failover and geeting more bandwodth. About bandwdith is not able about increase our bandwidth, but increase lines. Two lines with 1 Gbps each for our jobs.

This relies on switch configuration as well as some configuration in proxmox server. I usually use Link-Aggregation Protocol (LACP) which is part of IEEE specs, 802.3ad. 
In your switch you have to check switch manufacturer how to turn this on.
In proxmox server we will want to edit our network interfaces. It is easier from CLI to copy and paste that config to our new bond after we created 

```shell
nano /etc/network/interfaces
```
```shell

iface enso1 inet manual

iface enso2 inet manual

##create a linux bond interface
iface bond0 inet manual
       bond-slaves enso1 enso2
       bond-miimon 100
       bond-mode 802.3ad
       bond-xmit-hash-policy layer2+3

##change bridge interface 
auto vmbr0
iface vmbr0 inet static
        address 192.168.0.13/24
        gateway 192.168.0.1
        bridge-ports bond0
        bridge-stp off
        bridge-fd 0
        bridge-vlan-aware yes
        bridge-vids 2-4094
```
ctr+x to exit and y + enter to save

Then reboot your proxmox server.

## Conslusion
So that is everything we could do when we spin up a new Proxmox Server. This list that i have added to over every new deployment.

Is anything i missed, anything that you do when spin up a new deployment ?
Is there anything you would have done differently or not at all ?
If so not hesitate to comment below to let me know.