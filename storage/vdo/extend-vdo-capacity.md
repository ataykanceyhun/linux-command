# VDO Kapasite Artirma

| Alan | Deger |
| --- | --- |
| Risk | High |
| Son Dogrulama | 2026-02-16 |
| Tahmini Sure | 15-30 dk |
| Kesinti Etkisi | Kismi |

## Amac

Mevcut VDO pool ve virtual kapasitesini buyutmek, XFS dosya sistemini yeni
alani kullanacak sekilde genisletmek.

## Kapsam

- `vpool` ve `vdo` logical volume buyutme
- XFS grow islemi

## Onkosullar

- `root` veya `sudo` yetkisi
- `<vg_name>`, `<vpool_lv_name>`, `<vdo_lv_name>`, `<mount_point>` bilgileri
- VG icinde yeterli bos alan
- Distro notu:
  - `RHEL/CentOS notu:` `lvm2` ve `xfsprogs` kurulu olmali.
  - `Ubuntu/Debian notu:` Komut akisi aynidir.

| Degisken | Aciklama | Ornek |
| --- | --- | --- |
| `<vg_name>` | Volume group adi | `vg_data` |
| `<vpool_lv_name>` | VDO pool LV | `vpool0` |
| `<vdo_lv_name>` | VDO LV | `vdo1` |
| `<vpool_increase>` | Pool artisi | `1T` |
| `<vdo_virtual_increase>` | Virtual artisi | `10T` |
| `<mount_point>` | Mount noktasi | `/mnt/vdo1` |

## Risk ve Geri Donus (Rollback)

- Riskler:
  - Yanlis LV uzerinde `lvextend`
  - `xfs_growfs` sonrasi dosya sisteminin kucultulememesi
- Yedekleme:
```bash
sudo vgcfgbackup <vg_name>
sudo xfsdump -J - /dev/<vg_name>/<vdo_lv_name> > /backup/<vdo_lv_name>.$(date +%F).xfsdump
```
- Rollback:
```bash
# xfs_growfs oncesi geri donus
sudo lvreduce -L-<vdo_virtual_increase> /dev/<vg_name>/<vdo_lv_name>
sudo lvreduce -L-<vpool_increase> /dev/<vg_name>/<vpool_lv_name>

# xfs_growfs sonrasi geri donus: yeniden olustur + restore
sudo umount <mount_point>
sudo mkfs.xfs -f /dev/<vg_name>/<vdo_lv_name>
sudo mount /dev/<vg_name>/<vdo_lv_name> <mount_point>
sudo xfsrestore -J - /backup/<vdo_lv_name>.<backup_date>.xfsdump <mount_point>
```

## Adimlar

1. On kontrol snapshot'i al.
```bash
sudo vgs
sudo lvs -a -o +data_percent,metadata_percent
df -hT | grep <mount_point>
```
2. Pool kapasitesini artir.
```bash
sudo lvextend -L+<vpool_increase> /dev/<vg_name>/<vpool_lv_name>
```
3. VDO virtual kapasitesini artir.
```bash
sudo lvextend -L+<vdo_virtual_increase> /dev/<vg_name>/<vdo_lv_name>
```
4. XFS dosya sistemini buyut.
```bash
sudo xfs_growfs <mount_point>
```

## Dogrulama

```bash
sudo lvs -a -o +lv_size,data_percent,metadata_percent
df -hT | grep <mount_point>
sudo xfs_info <mount_point>
```

Beklenen sonuc:

- LV boyutlari artmis gorunur
- `df -hT` yeni kapasiteyi yansitir

## Sorun Giderme

- Sorun: `insufficient free space` hatasi.
  - Cozum:
```bash
sudo vgs
sudo pvs
```
- Sorun: `xfs_growfs` mount point hatasi.
  - Cozum:
```bash
mount | grep <mount_point>
```
- Sorun: `xfsdump` bulunamadi.
  - Cozum: `xfsdump` paketini kur (`sudo dnf install xfsdump` veya `sudo apt install xfsdump`).

## Referanslar

- `man lvextend`
- `man xfs_growfs`
- [LVM Physical Volume Buyutme](../lvm/resize-physical-volume.md)
