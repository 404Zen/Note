# 挂载硬盘
创建挂载目录
`sudo mkdir /mnt/hdd_vol2`
获取硬盘 UUID
```
zen@boom:~$ sudo blkid
....
/dev/sdb2: LABEL="Work" UUID="F626E9A326E964D9" TYPE="ntfs" PARTLABEL="Basic data partition" PARTUUID="7d1ef3fb-a9e9-42b8-b278-2ae8218aa174"
...
```
设定挂载
`sudo nano /etc/fstab`
```
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-RarBmNBR6SluJM3YxMO4V8S6DplgiWdCBIMf0cz2Jj9ZVgAKNpfg0nt4fPy>
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/5d44bd54-4082-4664-bff2-c04dc1417b6d /boot ext4 defaults 0 1
# /boot/efi was on /dev/sda1 during curtin installation
/dev/disk/by-uuid/8D30-A130 /boot/efi vfat defaults 0 1
/dev/disk/by-uuid/F626E9A326E964D9 /mnt/hdd_vol2 ntfs defaults 0 0
/swap.img       none    swap    sw      0       0

```
重启就可以了，或者
`mount /dev/sdb2 /mnt/hdd_vol2` 直接挂载。



# Samba
`sudo apt-get install samba samba-common`

samba服务添加用户
`sudo smbpasswd -a zen`


配置目录
`sudo gedit /etc/samba/smb.conf`

```
[share]
comment=This is samba dir
path=/home/zen/workspace
create mask=0755
directory mask=0755
writeable=yes
valid users=zen
browseable=yes
```

然后重启smaba服务
`sudo service smbd restart`

