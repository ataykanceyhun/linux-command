# LVM Uzerinde XFS Volume Olusturma

| Alan | Deger |
| --- | --- |
| Risk | High |
| Son Dogrulama | 2026-02-16 |
| Tahmini Sure | 15-25 dk |
| Kesinti Etkisi | Kismi |

## Amac

Yeni diskten LVM volume olusturup XFS formatlamak, mount etmek ve kalici fstab
kaydi ile kullanima almak.

## Kapsam

- Yeni depolama alani olusturma
- LVM + XFS mount yapilandirmasi

## Onkosullar

- `root` veya `sudo` yetkisi
- `lvm2` ve `xfsprogs` paketleri kurulu olmali
- Kullanilacak disk: `/dev/<disk_device>`
- Distro notu:
  - `RHEL/CentOS notu:` `sudo dnf install lvm2 xfsprogs`
  - `Ubuntu/Debian notu:` `sudo apt install lvm2 xfsprogs`

| Degisken | Aciklama | Ornek |
| --- | --- | --- |
| `<disk_device>` | Fiziksel disk cihazi | `nvme0n3` |
| `<vg_name>` | Volume group adi | `backups_vg` |
| `<lv_name>` | Logical volume adi | `backups_lv` |
| `<mount_point>` | Mount noktasi | `/mnt/backups_data` |

## Risk ve Geri Donus (Rollback)

- Riskler:
  - Yanlis diskin formatlanmasi
  - Yanlis fstab girdisi
- Yedekleme:
```bash
sudo cp /etc/fstab /etc/fstab.bak.$(date +%F-%H%M%S)
```
- Rollback:
```bash
sudo umount <mount_point>
sudo sed -i '\|<mount_point>|d' /etc/fstab
sudo lvremove -y /dev/<vg_name>/<lv_name>
sudo vgremove -y <vg_name>
sudo pvremove -y /dev/<disk_device>
```

## Adimlar

1. On kontrol snapshot'i al.
```bash
lsblk
sudo pvs
sudo vgs
sudo lvs
```
2. PV, VG ve LV olustur.
```bash
sudo pvcreate /dev/<disk_device>
sudo vgcreate <vg_name> /dev/<disk_device>
sudo lvcreate -n <lv_name> -l 100%FREE <vg_name>
```
3. XFS formatla.
```bash
sudo mkfs.xfs -b size=4096 -m reflink=1,crc=1 /dev/<vg_name>/<lv_name>
```
4. Mount ve kalici fstab girisini ekle.
```bash
sudo mkdir -p <mount_point>
sudo mount /dev/<vg_name>/<lv_name> <mount_point>
UUID=$(sudo blkid -s UUID -o value /dev/<vg_name>/<lv_name>)
echo "UUID=$UUID <mount_point> xfs defaults 0 1" | sudo tee -a /etc/fstab
```

## Dogrulama

```bash
mount | grep <mount_point>
df -hT | grep <mount_point>
sudo blkid /dev/<vg_name>/<lv_name>
```

Beklenen sonuc:

- `<mount_point>` aktif mount listesinde gorunur
- `df -hT` boyut ve dosya sistemi tipini (`xfs`) dogru gosterir

## Sorun Giderme

- Sorun: `mount -a` sonrasi hata.
  - Cozum:
```bash
sudo systemctl daemon-reload
sudo mount -a
```
- Sorun: `wrong fs type` hatasi.
  - Cozum:
```bash
sudo blkid /dev/<vg_name>/<lv_name>
sudo xfs_repair /dev/<vg_name>/<lv_name>
```
- Sorun: UUID bos donuyor.
  - Cozum: Format adiminin basarili tamamlandigini kontrol et.

## Referanslar

- `man vgcreate`
- `man lvcreate`
- `man mkfs.xfs`
