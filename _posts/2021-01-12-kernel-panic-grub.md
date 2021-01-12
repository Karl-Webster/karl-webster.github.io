---
title: vfs unable to mount aws
author: Karl Webster
date: 2021-01-07
categories: [DevOps, Centos]
tags: [DevOps, Centos]
---

# The Error
So when I want to clone a instance on AWS I will do the following:

* Take a snapshot of the Instance
* Create a volume from the snapshot
* Launch a new instance and attach the volume as the root volume `/dev/sda1`
* Delete the volume that came with the instance

Sometimes the instances fail to boot, and if you get a instance screenshot using the web console you will see an error:
```
Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)
```

In my opinion this is a cryptic error and could do with being more verbose

## What causes it

In my limited experience this is caused when you have high uptime instances that have had a kernel update and not been rebooted.

I saw this on a webserver with an uptime of about 100 days, it received regular updates so I am sure the package manager had updated the kernel. Then I took a snapshot of the webserver but since we did not reboot the newer kernel had not finished installing fully, but AWS thinks it has since the packages and metadata is there in grub and the `initramfs`

Again this is my interpretation of the issue and the fix reflects this.

# Fix

This will attempt to fix the kernel and if that fails revert back a version

This assumes you have a volume that needs fixing. You can get this by creating an AMI and then launching or just creating directly from the snapshots. For the purposes of this guide it does not matter.


* First create an instance with the same OS as the one we are trying to fix, for me this is Centos7, instance type does not matter (Make sure its the same architecture)

* Turn the instance off
* Attach the "broken" volume to the rescue instance (I like attaching it to `/dev/sdc`)

## Part 1
* Run `lsblk` to get the disk / partition name

* Create a mount for the drive
```bash
sudo mkdir -p /mnt/rescue/
```

* Mount the drive:
```bash
mount /dev/nvme1p1 /mnt/rescue
```

* Chroot it
```bash
for i in dev proc sys run; do mount -o bind /$i /mnt/rescue/$i; done

chroot /mnt/rescue
```

* Run the following
```
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

sudo dracut -f -vvvvv
```

Now exit the chroot
```bash
exit
for i in dev proc sys run; do sudo umount /mnt/rescue/$i; done
```

* Now re-attach the "broken" volume to the instance and check if it works, if so great! if not we need to revert the kernel

* So re-attach the volume to the rescue instance again

## Part 2
* Run `lsblk` to get the disk / partition name

* Create a mount for the drive
```bash
sudo mkdir -p /mnt/rescue/
```

* Mount the drive:
```bash
mount /dev/nvme1p1 /mnt/rescue
```

* Chroot it
```bash
for i in dev proc sys run; do mount -o bind /$i /mnt/rescue/$i; done

chroot /mnt/rescue
```

* Run the following
```
sed -i 's/GRUB_DEFAULT=0/GRUB_DEFAULT=saved/g' /etc/default/grub

sudo grub2-mkconfig -o /boot/grub2/grub.cfg

sudo grub2-set-default 1
```

Now exit the chroot
```bash
exit
for i in dev proc sys run; do sudo umount /mnt/rescue/$i; done
```


# Other Notes

If you are using a different OS please look at the docs

Part 1: https://aws.amazon.com/premiumsupport/knowledge-center/ec2-linux-kernel-panic-unable-mount/


Part2: https://aws.amazon.com/premiumsupport/knowledge-center/revert-stable-kernel-ec2-reboot/
