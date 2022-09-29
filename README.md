# proxmox
Proxmox tools

## How to mount a qcow2 disk image
This is a quick guide to mounting a qcow2 disk images on your host server. This is useful to reset passwords, edit files, or recover something without the virtual machine running.

## Step 1 - Enable NBD on the Host
```bash
modprobe nbd max_part=8
```
## Step 2 - Extract VM files RAW
```bash
cd /var/lib/vz/dump
zstd --decompress --stdout vzdump-qemu-100-2022_09_26-11_16_22.vma.zst | vma extract - extract
```
## Step 3 - Convert RAW to QCOWS
```bash
cd /var/lib/vz/dump/extract
qemu-img convert -f raw -O qcow2 disk-drive-scsi0.raw vm-100-disk-1.qcow2
```
## Step 4 - Connect the QCOW2 as network block device
```bash
qemu-nbd --connect=/dev/nbd0 /var/lib/vz/dump/extract/vm-100-disk-1.qcow2
```
## Step 5 - Find The Virtual Machine Partitions
```bash
fdisk /dev/nbd0 -l
```
## Step 6 - Mount the partition from the VM
```bash
mkdir -p /mnt/disk/ && mount /dev/nbd0p1 /mnt/disk/
df -h .; du -sh -- * | sort -hr
```
## Step 7 - After you done, unmount and disconnect
```bash
umount /mnt/disk/
qemu-nbd --disconnect /dev/nbd0
rmmod nbd
```
