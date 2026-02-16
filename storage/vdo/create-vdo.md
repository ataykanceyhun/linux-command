# VDO Olusturma

## Amac

LVM uzerinde VDO logical volume olusturup XFS ile formatlamak ve kalici mount
yapilandirmasini tamamlamak.

## Kapsam

- VDO depolama alani olusturma
- XFS ile kullanima hazir hale getirme

## Onkosullar

- `root` veya `sudo` yetkisi
- `lvm2`, `vdo` ve `xfsprogs` paketleri kurulu olmali
- Kullanilacak disk cihazi bos olmali (`/dev/<disk_device>`)
- Distro notu:
  - `RHEL/CentOS notu:` `sudo dnf install lvm2 vdo xfsprogs`
  - `Ubuntu/Debian notu:` VDO paket adi surume gore degisebilir.

## Risk ve Geri Donus (Rollback)

- Riskler:
  - Yanlis disk secimi nedeniyle veri kaybi
  - `fstab` yanlis satiri nedeniyle boot/mount hatasi
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

1. On kontrol yap.
```bash
lsblk
sudo pvs
sudo vgs
sudo lvs
```
2. Fiziksel volume olustur.
```bash
sudo pvcreate /dev/<disk_device>
```
3. Volume group olustur.
```bash
sudo vgcreate <vg_name> /dev/<disk_device>
```
4. VDO logical volume olustur.
```bash
sudo lvcreate --type vdo --name <vdo_lv_name> \
  --size <vdo_pool_size> --virtualsize <vdo_virtual_size> <vg_name>
```
5. XFS formatla.
```bash
sudo mkfs.xfs -K /dev/<vg_name>/<vdo_lv_name>
```
6. Mount noktasi olustur ve mount et.
```bash
sudo mkdir -p <mount_point>
sudo mount -o discard /dev/<vg_name>/<vdo_lv_name> <mount_point>
```
7. Kalici mount icin `fstab` girisi ekle.
```bash
UUID=$(sudo blkid -s UUID -o value /dev/<vg_name>/<vdo_lv_name>)
echo "UUID=$UUID <mount_point> xfs defaults,discard 0 0" | sudo tee -a /etc/fstab
```

## Dogrulama

```bash
sudo lvs -a -o +data_percent,metadata_percent
mount | grep <mount_point>
df -hT | grep <mount_point>
```

## Sorun Giderme

- Sorun: Mount sonrasi alan gorunmuyor.
  - Cozum:
```bash
sudo lvdisplay /dev/<vg_name>/<vdo_lv_name>
sudo xfs_info /dev/<vg_name>/<vdo_lv_name>
```
- Sorun: `mount -a` hata veriyor.
  - Cozum: `fstab` satirini duzelt, sonra tekrar dene.

## Referanslar

- `man lvcreate`
- `man mkfs.xfs`
- [VDO Kapasite Artirma](extend-vdo-capacity.md)
