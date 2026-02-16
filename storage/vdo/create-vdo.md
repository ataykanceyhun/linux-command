# VDO Olusturma

| Alan | Deger |
| --- | --- |
| Risk | High |
| Son Dogrulama | 2026-02-16 |
| Tahmini Sure | 20-35 dk |
| Kesinti Etkisi | Kismi |

## Amac

LVM uzerinde VDO logical volume olusturup XFS ile formatlamak, mount etmek ve
kalici fstab girdisi ile devreye almak.

## Kapsam

- Yeni VDO depolama alani olusturma
- XFS ile mount ve kalicilik

## Onkosullar

- `root` veya `sudo` yetkisi
- `lvm2`, `vdo`, `xfsprogs` paketleri
- Kullanilacak disk bos olmali (`/dev/<disk_device>`)
- Distro notu:
  - `RHEL/CentOS notu:` `sudo dnf install lvm2 vdo xfsprogs`
  - `Ubuntu/Debian notu:` VDO paket adi surume gore degisebilir.

| Degisken | Aciklama | Ornek |
| --- | --- | --- |
| `<disk_device>` | Fiziksel disk cihazi | `sdb` |
| `<vg_name>` | Volume group adi | `vg_data` |
| `<vdo_lv_name>` | VDO logical volume adi | `vdo1` |
| `<vdo_pool_size>` | Fiziksel VDO pool boyutu | `1T` |
| `<vdo_virtual_size>` | Virtual kapasite | `10T` |
| `<mount_point>` | Mount noktasi | `/mnt/vdo1` |

## Risk ve Geri Donus (Rollback)

- Riskler:
  - Yanlis disk secimi nedeniyle veri kaybi
  - Hatali fstab satiri nedeniyle boot sorunu
- Yedekleme:
```bash
sudo cp /etc/fstab /etc/fstab.bak.$(date +%F-%H%M%S)
```
- Rollback:
```bash
sudo umount <mount_point>
sudo sed -i '\|<mount_point>|d' /etc/fstab
sudo lvremove -y /dev/<vg_name>/<vdo_lv_name>
```

## Adimlar

1. On kontrol snapshot'i al.
```bash
lsblk
sudo pvs
sudo vgs
sudo lvs
```
2. Physical volume ve volume group olustur.
```bash
sudo pvcreate /dev/<disk_device>
sudo vgcreate <vg_name> /dev/<disk_device>
```
3. VDO logical volume olustur.
```bash
sudo lvcreate --type vdo --name <vdo_lv_name> \
  --size <vdo_pool_size> --virtualsize <vdo_virtual_size> <vg_name>
```
4. XFS formatla.
```bash
sudo mkfs.xfs -K /dev/<vg_name>/<vdo_lv_name>
```
5. Mount ve kalicilik islemlerini tamamla.
```bash
sudo mkdir -p <mount_point>
sudo mount -o discard /dev/<vg_name>/<vdo_lv_name> <mount_point>
UUID=$(sudo blkid -s UUID -o value /dev/<vg_name>/<vdo_lv_name>)
echo "UUID=$UUID <mount_point> xfs defaults,discard 0 0" | sudo tee -a /etc/fstab
```

## Dogrulama

```bash
sudo lvs -a -o +data_percent,metadata_percent
mount | grep <mount_point>
df -hT | grep <mount_point>
sudo blkid /dev/<vg_name>/<vdo_lv_name>
```

Beklenen sonuc:

- LV tipi `vdo` olarak listelenir
- `<mount_point>` aktif mount listesinde gorunur

## Sorun Giderme

- Sorun: `mount -a` hata veriyor.
  - Cozum: `fstab` satirini duzeltip tekrar dene.
- Sorun: VDO metadata kullanimi hizla doluyor.
  - Cozum:
```bash
sudo lvs -a -o +data_percent,metadata_percent
```
- Sorun: Yanlis diskte kurulum yapildi.
  - Cozum: Rollback adimlarini uygula, sonra dogru diskle yeniden basla.

## Referanslar

- `man lvcreate`
- `man mkfs.xfs`
- [VDO Kapasite Artirma](extend-vdo-capacity.md)
