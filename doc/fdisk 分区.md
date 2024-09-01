```
使用fdisk对SD卡进行从新分区。步骤如下：

1. 查看分区情况
## sudo fdisk -l

Disk /dev/sda: 107.4 GB, 107374182400 bytes
255 heads, 63 sectors/track, 13054 cylinders, total 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0005da7c

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048   205522943   102760448   83  Linux
/dev/sda2       205524990   209713151     2094081    5  Extended
/dev/sda5       205524992   209713151     2094080   82  Linux swap / Solaris

Disk /dev/sdb: 31.1 GB, 31104958464 bytes
4 heads, 16 sectors/track, 949248 cylinders, total 60751872 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x3c0c54e0

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048    60751871    30374912   83  Linux

2. 卸载：
    sudo umount /dev/sdb1/

3. 对设备重新分区

Qt@aplex:~$ sudo fdisk /dev/sdb
# 删除分区
Command (m for help): d
No partition is defined yet!
# 新建分区
Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
# 主分区
Select (default p): p
# 分区编号
Partition number (1-4, default 1): 1
# 分区起始扇区位
First sector (2048-60749823, default 2048): 
Using default value 2048
# 分区最后的扇区
Last sector, +sectors or +size{K,M,G} (2048-60749823, default 60749823): 
Using default value 60749823
# 保存
Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 22: Invalid argument.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.

# 查看分区情况 
Qt@aplex:~$ sudo fdisk -l

Disk /dev/sda: 107.4 GB, 107374182400 bytes
255 heads, 63 sectors/track, 13054 cylinders, total 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0005da7c

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048   205522943   102760448   83  Linux
/dev/sda2       205524990   209713151     2094081    5  Extended
/dev/sda5       205524992   209713151     2094080   82  Linux swap / Solaris

Disk /dev/sdb: 31.1 GB, 31104958464 bytes
4 heads, 16 sectors/track, 949248 cylinders, total 60751872 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x3c0c54e0

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048    60751871    30374912   83  Linux

4. 对分区进行格式化

Qt@aplex:~$ sudo mkfs.ext2 /dev/sdb1
mke2fs 1.42 (29-Nov-2011)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
1900544 inodes, 7593728 blocks
379686 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=4294967296
232 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000

Allocating group tables: done                            
Writing inode tables: done                            
Writing superblocks and filesystem accounting information: done   
```