# External Hard Drive for Backups

Based on notes posted by [Thomas Stringer](https://trstringer.com/linux-external-disk/)

## Create an Encrypted Partition on an External Disk

Identify the external drive, using `sudo fsarchiver probe simple` or `lsblk`.
Use `fdisk` to create a new partition if needed. Unmount the drive before 
proceeding as follows:

1. Encrypt the partition with LUKS

	`sudo cryptsetup luksFormat /dev/sda1`

2. Unlock the partition

	`sudo cryptsetup luksOpen /dev/sda1 backup-disk`

3. Format the partition

	`sudo mkfs.ext4 /dev/mapper/backup-disk`

4. Close the Encrypted Disk

	`sudo cryptsetup luksClose backup-disk`

You can now the use external drive to perform backups to the encrypted partition. 
Try `rsync -av`. 

## Erase a Disk

Fastest way is to use `sudo wipefs -a /dev/sdX`. However, data can still be 
recovered from the disk. A more secure approach is to overwrite the entire
disk with zeros: `sudo dd if=/dev/zero of=/dev/sdX bs=4M status=progress`. 
For USB flash drives and SD Cards use a block size of `1M` or `2M` so 
as not to overload the controller. A block size of `16M` would probably work
well with an NVMe drive.
