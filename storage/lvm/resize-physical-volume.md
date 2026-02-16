# LVM Physical Volume Buyutme

| Alan | Deger |
| --- | --- |
| Risk | Medium |
| Son Dogrulama | 2026-02-16 |
| Tahmini Sure | 10-20 dk |
| Kesinti Etkisi | Kismi |

## Amac

Disk kapasitesi altyapi tarafinda artirildiktan sonra, LVM Physical Volume
alanini isletim sisteminde genisletmek.

## Kapsam

- LVM kullanan Linux sunucular
- `pvresize` sonrasi VG/LV/FS buyutme akisi

## Onkosullar

- `root` veya `sudo` yetkisi
- `lvm2` paketleri kurulu olmali
- Disk kapasitesi hypervisor/fiziksel katmanda artirilmis olmali
- Distro notu:
  - `RHEL/CentOS notu:` `lvm2` genelde kurulu gelir.
  - `Ubuntu/Debian notu:` Gerekirse `sudo apt install lvm2`.

| Degisken | Aciklama | Ornek |
| --- | --- | --- |
| `<pv_device>` | PV cihazi | `sdb` |
| `<vg_name>` | Volume group adi | `vg_data` |
| `<lv_name>` | Logical volume adi | `data_lv` |
| `<mount_point>` | Dosya sistemi mount noktasi | `/mnt/data` |
| `<old_size>` | Geri donus boyutu | `2T` |

## Risk ve Geri Donus (Rollback)

- Riskler:
  - Yanlis diskte komut calistirma
  - Yetersiz bos alan nedeniyle `lvextend` hatasi
- Yedekleme:
```bash
sudo vgcfgbackup <vg_name>
```
- Rollback:
```bash
# Yalnizca yeterli bos alan varsa geri cekilebilir
sudo pvresize --setphysicalvolumesize <old_size> /dev/<pv_device>
```

## Adimlar

1. On kontrol snapshot'i al.
```bash
lsblk
sudo pvs
sudo vgs
sudo lvs
```
2. PV boyutunu yeniden tara.
```bash
sudo pvresize /dev/<pv_device>
```
3. Ihtiyac varsa LV buyut.
```bash
sudo lvextend -l +100%FREE /dev/<vg_name>/<lv_name>
```
4. Dosya sistemini buyut.
```bash
# XFS
sudo xfs_growfs <mount_point>

# ext4
sudo resize2fs /dev/<vg_name>/<lv_name>
```

## Dogrulama

```bash
sudo pvs
sudo vgs
sudo lvs -o +devices
df -hT | grep <mount_point>
```

Beklenen sonuc:

- `pvs` cikti boyutu yeni kapasiteyi gosterir
- `df -hT` cikti boyutu artmis gorunur

## Sorun Giderme

- Sorun: Yeni disk boyutu gorunmuyor.
  - Cozum:
```bash
sudo partprobe
lsblk
```
- Sorun: `Device not found` hatasi.
  - Cozum: Cihaz adini `lsblk` ile dogrula (`sdb`, `nvme0n2` vb.).
- Sorun: `Insufficient free space`.
  - Cozum:
```bash
sudo pvs
sudo vgs
```

## Referanslar

- `man pvresize`
- `man vgcfgbackup`
- [VDO Olusturma](../vdo/create-vdo.md)
