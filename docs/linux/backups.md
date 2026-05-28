# External Hard Drive for Backups

Based on notes posted by [Thomas Stringer](https://trstringer.com/linux-external-disk/)

## Create an Encrypted Partition on an External Disk

First identify the external drive, using `sudo fsarchiver probe simple` or `lsblk`.
Use `fdisk` to create a new partition if needed. 


1. Encrypt the partition with LUKS

	`sudo cryptsetup luksFormat /dev/sda1`

2. Unlock the partition

	`sudo cryptsetup luksOpen /dev/sda1 backup-disk`

3. Format the partition

	`sudo mkfs.ext4 /dev/mapper/backup-disk`

4. Close the Encrypted Disk

	`sudo cryptsetup luksClose backup-disk`

You can now use external drive to perform backups to the encrypted partition. 
Try `rsync -av`. 
