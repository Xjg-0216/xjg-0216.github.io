

# 挂载硬盘

### 1. 检查已挂载的文件系统

使用`df -h`命令查看当前系统中已挂载的文件系统
```bash
df -h
```
这将显示当前所有挂载的分区以及它们的挂载点。

### 2. 检查所有分区
使用命令`lsblk`命令查看所有硬盘及其分区，并确定哪些分区还没有挂载
```bash
lsblk -f
```
如果某个分区没有挂载点，说明它没有挂载， 例如
```text
NAME   FSTYPE LABEL UUID                                 MOUNTPOINT
sda                                                      
├─sda1 ntfs         1A2B3C4D5E6F7G8H                     /mnt/data
├─sda2 swap         2A3B4C5D6E7F8G9H                     
├─sda3 ext4         3A4B5C6D7E8F9G0H                     /mnt/backup
└─sda4 ext4         4A5B6C7D8E9F0G1H                     
sdb                                                      
├─sdb1 ntfs         5A6B7C8D9E0F1G2H                     /mnt/windows
├─sdb2 ntfs         6A7B8C9D0E1F2G3H                     
└─sdb3 ext4         7A8B9C0D1E2F3G4H                     
```
从上面可以看出，`/dev/sda2`、`/dev/sda4`、`/dev/sdb2`和`/dev/sdb3`没有挂载点，因此它们没有被挂载。


### 挂载分区

以sda1分区为例进行挂载

!!! note
    挂载时要注意权限，正常挂载后只有超级用户才有权限，普通用户没有权限访问这些挂载点，因此需要确保普通用户有访问权限
#### 获取用户ID和组ID
```bash
id username
```
假设结果为：
```text
uid=1000(username) gid=1000(username) groups=1000(username),27(sudo),...
```
#### 挂载分区
```bash
sudo mount -o uid=1000,gid=1000 /dev/sda1 /mnt/sda1
```
`/mnt/sda1`是挂载点，需要创建合适位置的挂载点

### 验证挂载
```bash
df -h | grep /mnt/sda1
```


### 取消挂载

```bash
sudo umount /mnt/sda1
```