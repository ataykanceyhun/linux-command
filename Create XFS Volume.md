## XFS Volume Oluşturma

> Disk Listeleme
   
    lsblk

    NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
    nvme0n1       259:0    0   30G  0 disk
    ├─nvme0n1p1   259:1    0    1G  0 part /boot
    └─nvme0n1p2   259:2    0   29G  0 part
      ├─rhel-root 253:0    0   26G  0 lvm  /
      └─rhel-swap 253:1    0    3G  0 lvm  [SWAP]
    nvme0n2       259:3    0    1T  0 disk /mnt/veeamrepo01
    nvme0n3       259:4    0  200G  0 disk

> Volume Group Oluşturuluyor

    sudo vgcreate backups_vg /dev/nvme0n3
    Physical volume "/dev/nvme0n3" successfully created.
      Volume group "backups_vg" successfully created

> Logical Volume Oluşturuluyor

    sudo lvcreate -n backups_lv -l 100%FREE backups_vg
      Logical volume "backups_lv" created.

> Mount Edilecek Klasör Oluşturluyor

    sudo mkdir /mnt/backups_data

> Logical Volume XFS olarak Yapılandırılıyor

    sudo mkfs.xfs -b size=4096 -m reflink=1,crc=1 /dev/backups_vg/backups_lv
    meta-data=/dev/backups_vg/backups_lv isize=512    agcount=4, agsize=13106944 blks
             =                       sectsz=512   attr=2, projid32bit=1
             =                       crc=1        finobt=1, sparse=1, rmapbt=0
             =                       reflink=1    bigtime=1 inobtcount=1
    data     =                       bsize=4096   blocks=52427776, imaxpct=25
             =                       sunit=0      swidth=0 blks
    naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
    log      =internal log           bsize=4096   blocks=25599, version=2
             =                       sectsz=512   sunit=0 blks, lazy-count=1
    realtime =none                   extsz=4096   blocks=0, rtextents=0

> Volume'lerin UUID Bilgileri Listeleniyor

    ls -l /dev/disk/by-id/dm-*
    lrwxrwxrwx. 1 root root 10 Nov  2 13:34 /dev/disk/by-id/dm-name-backups_vg-backups_lv -> ../../dm-2
    lrwxrwxrwx. 1 root root 10 Nov  2 13:29 /dev/disk/by-id/dm-name-rhel-root -> ../../dm-0
    lrwxrwxrwx. 1 root root 10 Nov  2 13:29 /dev/disk/by-id/dm-name-rhel-swap -> ../../dm-1
    lrwxrwxrwx. 1 root root 10 Nov  2 13:29 /dev/disk/by-id/dm-uuid-LVM-pjlexphE0gtHkqQjL1X2hpzHIega5Z5DTUmE9fcwCfPXtZydOnoQd9rsUzekrIyd -> ../../dm-0
    lrwxrwxrwx. 1 root root 10 Nov  2 13:29 /dev/disk/by-id/dm-uuid-LVM-pjlexphE0gtHkqQjL1X2hpzHIega5Z5DZ9dkuDUAdwaIUDsWa8GlBcDQ1A3lZ4Xz -> ../../dm-1
    lrwxrwxrwx. 1 root root 10 Nov  2 13:34 /dev/disk/by-id/dm-uuid-LVM-tkAw3qBMFeEoN9if81uSSxbbJSEyqJfPsSn6P3JRDi9muhbCtjLRw8TGJFkO7of4 -> ../../dm-2

> Mount Edilecek Alanın Yapılandırılması

    sudo nano /etc/fstab
    /dev/disk/by-id/dm-uuid-LVM-tkAw3qBMFeEoN9if81uSSxbbJSEyqJfPsSn6P3JRDi9muhbCtjLRw8TGJFkO7of4 /mnt/backups_data xfs defaults 0 1

> Mount İşleminin Yapılması

    sudo mount -a
    mount: (hint) your fstab has been modified, but systemd still uses
           the old version; use 'systemctl daemon-reload' to reload.

> Dosya Sistemindeki Alan Bilgileri Listeleniyor

    df -hT
    Filesystem                        Type      Size  Used Avail Use% Mounted on
    devtmpfs                          devtmpfs  4.0M     0  4.0M   0% /dev
    tmpfs                             tmpfs     1.8G     0  1.8G   0% /dev/shm
    tmpfs                             tmpfs     725M  9.1M  716M   2% /run
    /dev/mapper/rhel-root             xfs        26G  3.6G   23G  14% /
    /dev/nvme0n2                      xfs       1.0T   59G  965G   6% /mnt/veeamrepo01
    /dev/nvme0n1p1                    xfs      1014M  406M  609M  40% /boot
    tmpfs                             tmpfs     363M     0  363M   0% /run/user/1000
    /dev/mapper/backups_vg-backups_lv xfs       200G  1.5G  199G   1% /mnt/backups_data
    

> Yektilendirme işlemleri Yapılandrılıyor

    sudo chown -R vdo-user:vdo-user /mnt/backups_data
    
    sudo chmod 700 /mnt/backups_data/
    
    ls -ltd /mnt/backups_data/
    drwx------. 2 vdo-user vdo-user 6 Nov  2 13:34 /mnt/backups_data/
