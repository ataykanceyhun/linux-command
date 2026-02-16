# LVM Uzerinde XFS Volume Olusturma

## Amac

Yeni diskten LVM volume olusturup XFS formatlamak, mount etmek ve kalici
`fstab` kaydi ile kullanima almak.

## Kapsam

- Yeni depolama alani hazirlama
- LVM + XFS tabanli mount yapilandirmasi

## Onkosullar

- `root` veya `sudo` yetkisi
- `lvm2` ve `xfsprogs` paketleri kurulu olmali
- Kullanilacak disk: `/dev/<disk_device>`
- Distro notu:
  - `RHEL/CentOS notu:` `sudo dnf install lvm2 xfsprogs`
  - `Ubuntu/Debian notu:` `sudo apt install lvm2 xfsprogs`

## Risk ve Geri Donus (Rollback)

- Riskler:
  - Yanlis diskin formatlanmasi
  - Yanlis `fstab` girisi
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

1. On kontrol yap.
```bash
lsblk
sudo pvs
sudo vgs
sudo lvs
```
2. Physical volume olustur.
```bash
sudo pvcreate /dev/<disk_device>
```
3. Volume group olustur.
```bash
sudo vgcreate <vg_name> /dev/<disk_device>
```
4. Logical volume olustur.
```bash
sudo lvcreate -n <lv_name> -l 100%FREE <vg_name>
```
5. XFS formatla.
```bash
sudo mkfs.xfs -b size=4096 -m reflink=1,crc=1 /dev/<vg_name>/<lv_name>
```
6. Mount noktasi olustur ve mount et.
```bash
sudo mkdir -p <mount_point>
sudo mount /dev/<vg_name>/<lv_name> <mount_point>
```
7. UUID ile kalici mount kaydi ekle.
```bash
UUID=$(sudo blkid -s UUID -o value /dev/<vg_name>/<lv_name>)
echo "UUID=$UUID <mount_point> xfs defaults 0 1" | sudo tee -a /etc/fstab
```

## Dogrulama

```bash
mount | grep <mount_point>
df -hT | grep <mount_point>
sudo blkid /dev/<vg_name>/<lv_name>
```

## Sorun Giderme

- Sorun: `mount -a` sonrasi hata.
  - Cozum: `fstab` satirini kontrol et ve daemon cache yenile.
```bash
sudo systemctl daemon-reload
sudo mount -a
```
- Sorun: XFS metadata hatasi.
  - Cozum:
```bash
sudo xfs_repair /dev/<vg_name>/<lv_name>
```

## Referanslar

- `man vgcreate`
- `man lvcreate`
- `man mkfs.xfs`
