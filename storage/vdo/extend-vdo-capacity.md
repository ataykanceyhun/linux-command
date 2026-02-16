# VDO Kapasite Artirma

## Amac

Mevcut VDO pool ve virtual kapasitesini buyutmek, dosya sistemini yeni alani
kullanacak sekilde genisletmek.

## Kapsam

- Mevcut `vg` icindeki VDO hacimlerinin buyutulmasi
- XFS dosya sistemi buyutme islemi

## Onkosullar

- `root` veya `sudo` yetkisi
- `<vg_name>`, `<vpool_lv_name>`, `<vdo_lv_name>`, `<mount_point>` bilgileri
- Volume group icinde yeterli bos alan
- Distro notu:
  - `RHEL/CentOS notu:` `lvm2` ve `xfsprogs` paketleri yuku olmali.
  - `Ubuntu/Debian notu:` Komut akisi aynidir.

## Risk ve Geri Donus (Rollback)

- Riskler:
  - Yanlis LV uzerinde `lvextend` calistirilmasi
  - `xfs_growfs` sonrasi dosya sistemi kucultmenin desteklenmemesi
- Yedekleme:
```bash
sudo vgcfgbackup <vg_name>
sudo xfsdump -J - /dev/<vg_name>/<vdo_lv_name> > /backup/<vdo_lv_name>.$(date +%F).xfsdump
```
- Rollback:
```bash
# xfs_growfs oncesinde donus
sudo lvreduce -L-<vdo_virtual_increase> /dev/<vg_name>/<vdo_lv_name>
sudo lvreduce -L-<vpool_increase> /dev/<vg_name>/<vpool_lv_name>

# xfs_growfs sonrasinda geri donus: yeniden olustur + yedekten restore
sudo umount <mount_point>
sudo mkfs.xfs -f /dev/<vg_name>/<vdo_lv_name>
sudo mount /dev/<vg_name>/<vdo_lv_name> <mount_point>
sudo xfsrestore -J - /backup/<vdo_lv_name>.<backup_date>.xfsdump <mount_point>
```

## Adimlar

1. On kontrol yap.
```bash
sudo vgs
sudo lvs -a -o +data_percent,metadata_percent
df -hT | grep <mount_point>
```
2. VDO pool kapasitesini artir.
```bash
sudo lvextend -L+<vpool_increase> /dev/<vg_name>/<vpool_lv_name>
```
3. VDO virtual kapasitesini artir.
```bash
sudo lvextend -L+<vdo_virtual_increase> /dev/<vg_name>/<vdo_lv_name>
```
4. Dosya sistemini buyut.
```bash
sudo xfs_growfs <mount_point>
```

## Dogrulama

```bash
sudo lvs -a -o +lv_size,data_percent,metadata_percent
df -hT | grep <mount_point>
sudo xfs_info <mount_point>
```

## Sorun Giderme

- Sorun: `insufficient free space` hatasi.
  - Cozum:
```bash
sudo vgs
sudo pvs
```
- Sorun: `xfs_growfs` basarisiz.
  - Cozum: Mount noktasinin aktif oldugunu kontrol et.
```bash
mount | grep <mount_point>
```

## Referanslar

- `man lvextend`
- `man xfs_growfs`
- [LVM Physical Volume Buyutme](../lvm/resize-physical-volume.md)
