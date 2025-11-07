**step-by-step guide** to *create LVM on Linux*, with the exact commands and short explanations so you can use it in interviews or on real servers.

**Create LVM on Linux**

⚠ **Safety first:** do this on a test VM or with empty disks/partitions. LVM commands can destroy data. Always back up important data before proceeding.

1) # **Prepare physical disk or partition**

You can use a whole disk (e.g. /dev/sdb) or a partition (e.g. /dev/sdb1). Example (create a partition with fdisk):

```bash

sudo fdisk /dev/sdb

```

\# inside fdisk: n (new), p (primary), choose defaults, then w (write)

Or use the whole disk directly (no partitioning).

2) # **Create a Physical Volume (PV)**

Mark the device as an LVM PV:

```
sudo pvcreate /dev/sdb1
# or:
 sudo pvcreate /dev/sdb
```
Check PVs:
```
sudo pvs
sudo pvdisplay /dev/sdb1
```

3) # **Create a Volume Group (VG)**

Group one or more PVs into a Volume Group:
```
sudo vgcreate vg_data /dev/sdb1
...
..
.

# vg_data is the VG name
```
Check VGs:
```
sudo vgs
sudo vgdisplay vg_data
```
4) # **Create a Logical Volume (LV)**

Decide size (e.g. 10G) and name:
```
sudo lvcreate -L 10G -n lv_backup vg_data
```
\# or to use all free space:
```
sudo lvcreate \-l 100%FREE \-n lv_backup vg_data
```
Check LVs:
```
sudo lvs
sudo lvdisplay /dev/vg_data/lv_backup
```
5) # **Create a filesystem on the LV**

Choose filesystem (ext4 or xfs are common): 

```
sudo mkfs.ext4 /dev/vg_data/lv_backup
```
\# or:
```
sudo mkfs.xfs /dev/vg\_data/lv\_backup
```
6) # **Mount the LV**

Create a mountpoint and mount:
```
sudo mkdir -p /mnt/backup
```
```
sudo mount /dev/vg_data/lv_backup /mnt/backup
```
# **Confirm mount:**
```
df -h | grep /mnt/backup
```

7) # **Make mount persistent (fstab)**

Get the device mapper path or #***UUID:***
```
sudo blkid /dev/vg_data/lv_backup
```
# ***Example output: ***
```
/dev/mapper/vg_data-lv_backup: UUID="..." TYPE="ext4"
```
 # ***Add to /etc/fstab (use UUID or /dev/mapper path). Example line using device mapper:

/dev/mapper/vg_data-lv_backup /mnt/backup ext4 defaults 0 2

Then test:

sudo umount /mnt/backup

sudo mount \-a  \# should mount without errors

8) # **Common management tasks**

## **Extend LV (and filesystem)**

If you add PVs to VG or have free space, extend LV:
```
sudo lvextend -L +5G /dev/vg_data/lv_backup # add 5GB

# or use all free: sudo lvextend -l +100%FREE /dev/vg_data/lv_backup
```
Then grow filesystem:

* For ext4 (can be grown online while mounted):
```
sudo resize2fs /dev/vg_data/lv_backup
```
* For xfs (must be mounted):
```
sudo xfs_growfs /mnt/backup
```

## **Reduce LV (dangerous — do backup first)**

1. Unmount filesystem.

2. Run fsck and resize filesystem down.

3. Use lvreduce with caution:
```
sudo umount /mnt/backup
```
```
sudo e2fsck \-f /dev/vg\_data/lv\_backup
```
```
sudo resize2fs /dev/vg_data/lv_backup 8G 
```

# shrink FS to 8G
```
sudo lvreduce -L 8G /dev/vg_data/lv_backup
```
```
sudo mount /dev/vg_data/lv_backup /mnt/backup
```


## **Remove LV / VG / PV**

Remove LV:
```
sudo umount /mnt/backup

sudo lvremove /dev/vg_data/lv_backup
```
Remove VG:
```
sudo vgremove vg_data
```
Remove PV:
```
sudo pvremove /dev/sdb1
```
9) # **Useful commands (quick reference)**

# show physical volumes summary   ## detailed PV info
```
pvs
#or
pvdisplay    
```
		
# show volume groups summary   /# detailed VG info 
```
vgs
#or
vgdisplay      
```
# show logical volumes summary   #/# detailed LV info  
```
lvs
or
lvdisplay    
```
  
vgdisplay
vgextend / vgreduce /# add/remove PVs from VG lvextend / lvreduce /# grow/shrink logical volumes

10) # **Extras (short interview points)**

    * LVM layers: **PV → VG → LV**.

    * LVM allows resizing volumes, snapshots, and combining multiple disks into a single logical pool.

    * Snapshots: lvcreate \-s (used for backups) — note: snapshot performance/behavior varies by LVM version.

    * Always back up before shrinking or removing volumes.

