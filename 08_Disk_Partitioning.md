# Disk Partitioning

This section covers disk partitioning, logical volume management (LVM), filesystem creation, resizing, and related tools on Oracle Linux.

---


## List Block Devices

```bash
lsblk
```

---

## parted Utility

`parted` is a modern disk partitioning tool, replacing the classic `fdisk`. It supports partition resizing, GPT, and scripting.

### Example usage:

```bash
parted /dev/sdb
```

Inside `parted`, common commands:

```bash
print                        # Show disk state
mklabel gpt                  # Create GPT partition table
mkpart primary 0% 100%       # Create primary partition (full size)
set 1 lvm on                 # Enable LVM flag on partition 1
quit                         # Exit parted
```

After changes:

```bash
partprobe                    # Notify OS of partition changes
```

---

## Logical Volume Management (LVM)

LVM abstracts filesystems from physical disks, allowing flexible resizing and grouping.

### Configure LVM

```bash
vi /etc/lvm/lvm.conf
# Ensure:
use_devicesfile=0
```

---

### Physical Volumes (PV)

```bash
pvcreate /dev/sdb1   # Create PV
pvremove /dev/sdb1   # Remove PV
pvscan               # Scan PVs
pvdisplay            # Display PV info
```

---

### Volume Groups (VG)

```bash
vgcreate ol /dev/sdb1   # Create VG
vgextend ol /dev/sdc1   # Add PV to VG
vgreduce ol /dev/sdc1   # Remove PV from VG
vgscan
vgdisplay
```

---

### Logical Volumes (LV)

```bash
lvcreate -L 10G -n local ol       # Fixed size
lvcreate -l 100%VG -n data ol     # Use all VG space
lvcreate -l 100%FREE -n data ol   # Use remaining VG space
lvscan
lvdisplay
```

Device path:

```
/dev/ol/local  or  /dev/mapper/ol-local
```

---

## File System Operations

### Format LVs:

```bash
mkfs.xfs /dev/mapper/ol-local
mkfs.xfs -f /dev/mapper/ol-local   # Force
```

### Create SWAP:

```bash
mkswap /dev/mapper/ol-swap
```

---

## Resize Logical Volumes

### Shrink or grow:

```bash
lvresize -L 15G /dev/mapper/ol-local
lvresize -L+5G /dev/mapper/ol-local
lvreduce -L-5G /dev/mapper/ol-local
lvextend -L+10G /dev/mapper/ol-local
lvextend -l +100%FREE /dev/mapper/ol-local
```

After resizing:

```bash
xfs_growfs /dev/mapper/ol-local
```

If not reflected:

```bash
echo "- - -" > /sys/class/scsi_host/<host_id>/scan
partprobe -s
```

### Add mount to `/etc/fstab`:

```fstab
/dev/mapper/ol-local   /local   xfs   defaults   0   0
```

---

## XFS Repair

```bash
xfs_repair <pv_id>
```

---

## Extend LVM from VMware

```bash
echo '1' > /sys/class/scsi_device/<scsi_assress>/device/rescan
partprobe -s
parted /dev/<partition> (resizepart, <partition_number>, 100%)
pvresize /dev/<partition>
lvextend -l +100%FREE /dev/mapper/ol-local
xfs_growfs /dev/mapper/ol-local
```

---

## Virtual Memory Filesystem (RAM Disk)

```bash
mkdir /mnt/ramdisk
mount -t tmpfs -o size=1G tmpfs /mnt/ramdisk
```

### Auto-mount at startup:

```fstab
tmpfs   /mnt/ramdisk   tmpfs   rw,relatime,size=1G   0   0
```

---

## USB Format (FAT32)

Example: `/dev/sdj`

```bash
dd if=/dev/zero of=/dev/sdj bs=4096 status=progress
parted /dev/sdj --script -- mklabel msdos
parted /dev/sdj --script -- mkpart primary fat32 1MiB 100%
mkfs.vfat -F32 /dev/sdj1
```

---

## Reference

- [Oracle (2025). "Linux Disk Partitioning (fdisk, parted)," *Oracle-Base*](https://oracle-base.com/articles/linux/linux-disk-partitioning)
- [Oracle (2025). "Linux Logical Volume Management," *Oracle-Base*](https://oracle-base.com/articles/linux/linux-logical-volume-management)
