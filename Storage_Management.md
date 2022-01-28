# Storage Management

## List, create, delete, and modify physical storage partitions

* `lsblk` lists all available disk devices plus available partitions

* `fdisk` it is used to manage disk partition in MBR modality

  * E.g. `fdisk /dev/sda`

    This will open an interactive menu that will permit to show current status of partitions or create a delete new partitions

* `gdisk` it is used to manage disk partition in GPT modality

  - E.g. `gdisk /dev/sda`

* Destroy all MBR partition on a disk

  * `gdisk /dev/sda` -> `x` (expert) -> `z` (zap)

* Convert MBT to GPT

  * `gdisk /dev/sda` -> `W` -> `Y`

## Manage and configure LVM storage

* Before create a Logical Volume must be created in sequence a physical volume and after a volume group
* A physical volume is a partition that can be part of volume group. Inside a volume group can be created logical volume
* The advance of logical volume is that their dimension can be managed easly
* If more space is need a volume group can be extended as well



Physical Volume

* `pvcreate /dev/sdb1`

  To create a physical volume with partition sbd1

* `pvs` lists available physical volumes

* `pvdisplay /dev/sdb1` shows info of a physical volume

Volume Group

* `vgcreate vgname /dev/sdb1`

  To create a volume group called *vgname* and add the sdb1 physical volume to it

* `vgs` lists available volume groups

* `vgdisplay vgname` shows info of a volume group

* `vgextend vgname /dev/sdc3` extends a volume group adding a new physical volume `/dev/sdc3`

Logical volume

* `lvcreate -n volumename -L 10G vgname`

  To create a logical volume called *volumename*  of size 10GB on volume group *vgname*

* `lvcreate -n volumename -l 100%FREE vgname`

  To create a logical volume called *volumename*  with all available space on volume group *vgname*

* `lvs` list available logical volumes

* `lvdisplay` shows info of all logical volumes

* `lvdisplay vgname/volumename` shows info of a logical volume *volumename* contained in *vgname* volume group

* Before use a logical volume, a file system must be created on it

* `blkid /dev/vgname/volumename ` shows the UUID of a formatted volume group

* `lvextend -L +1G -r vgname/volumename ` extends the logical volume *volumename* of one giga

  * `-r`  is used to resize file system

* `lvreduce -L -1G -r vgname/volumename ` reduce the logical volume *volumename* of one giga

## Create and configure encrypted storage

* To use encrypted storage a kernel module must be loaded
  * `sudo modprobe dm_crypt` Loads kernel module dm_crypt
  * `echo dm_crypt >> /etc/modules-load.d/dm_crypt.conf` to load dm_crypt module automatically when system will be restarted
  * `lsmod` lists all loaded kernel modules
* `yum -y install cryptsetup` install software used to manage encrypted storage



Encrypt

* `cryptsetup luksFormat /dev/vgname/volumename` encrypts a logical volume *volumename* contained in *vgname* volume group

  * A password must be provided
  * When confirmation will be required insert a capital <u>YES</u>

  * **NOTE**: this command can be used with physical volume as well

* `cryptsetup open --type luks /dev/vgname/volumename namenewdevice`

  It open encrypted volume and associate it to a new device called *namenewdevice*

  * Password must be provided

* `mkfs.ext4 /dev/mapper/namenewdevice` 

  It creates a file system in *namenewdevice*

  Now new the new device can be mounted



Close device

* Unmount device
* `cryptsetup close namenewdevice`close *namenewdevice*



Automount

* `echo "passwd" >> /root/key` Insert a string that will be used that will be used as authentication key to open device

* `chmod 400 /root/key` reduces permission on key file
* `cryptsetup luksAddKey /dev/mapper/namenewdevice /root/key` add key to encrypted device called *namenewdevice*
* Edit `/etc/crypttab` and add below row:
  * `namenewdevice /dev/vgname/volumename /root/key`

* Add below row to `/etc/fstab`
  * `/dev/mapper/namenewdevice /mnt/mountpoint ext4 defaults 0 0`

* Reboot system or reload system manager
  * `systemctl daemon-reload`
* The new encrypted volume will be mounted on `/mnt/mountpoint`

## Configure systems to mount file systems at or during boot

* Edit `/etc/fstab` adding a row similar to:

  * /dev/sdb1 /mnt/mountpoint ext4 defaults 0 0

    * Mount device sdb1 to mountpoint. 

    * Device is formatted using ext4 filesystem.

    * Default mount options are used
    * 0 0 -> Dump (bkp) and fsck. 
      * First 0 means no backup required
      * Second 0 means no fsck required in case of not correct umount. To enable fsck insert 2 because number indicate the check order, and 1 is given to operating system disk and two do data disks

* `mount` shows mounted volumes

* `mount -a` reloads /etc/fstab

* `mount -t type  -o options device dir`

  * It mounts a *device* formatted with file system *type* on directory *dir* using a list of options

  * options can be:
    * async -> I/O asincrono
    * auto -> Can be mounted using mount -a
    * default ->Equal to this list of options: async,auto,dev,exec,nouser,rw,suid
    * loop -> To mount an ISO image
    * noexec -> no exec
    * nouser -> A user cannot mount this volume
    * remount -> Mount volume also if it is already mounted
    * ro -> Read only
    * rw -> Read an write
    * relatime -> Modify file access time (atime) if file is changed or one time a day. Alternative, to reduce disk traffic, noatime can be used. This is useful with SSD to avoid not useful write.
