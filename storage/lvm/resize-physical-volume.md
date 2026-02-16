# LVM Physical Volume Buyutme

## Amac

Hypervisor veya fiziksel diskte kapasite buyutulduktan sonra, LVM Physical
Volume alanini isletim sistemi tarafinda genisletmek.

## Kapsam

- LVM kullanan Linux sunucular
- Isletim sistemi tarafinda kapasite degisikliginin yansitilmasi

## Onkosullar

- `root` veya `sudo` yetkisi
- `lvm2` paketleri kurulu olmali
- Disk kapasitesi altyapida gercekten buyutulmus olmali
- Distro notu:
  - `RHEL/CentOS notu:` Paket genellikle varsayilan gelir.
  - `Ubuntu/Debian notu:` Gerekirse `sudo apt install lvm2`.

## Risk ve Geri Donus (Rollback)

- Riskler:
  - Yanlis disk uzerinde komut calisma riski
  - Beklenmeyen metadata sorunu
- Yedekleme:
```bash
sudo vgcfgbackup <vg_name>
```
- Rollback:
```bash
# Yalnizca yeterli bos alan varsa eski boyuta donulebilir
sudo pvresize --setphysicalvolumesize <old_size> /dev/<pv_device>
```

## Adimlar

1. On kontrol yap.
```bash
lsblk
sudo pvs
sudo vgs
sudo lvs
```
2. PV boyutunu yeniden tara ve buyut.
```bash
sudo pvresize /dev/<pv_device>
```
3. Gerekirse yeni bos alani ilgili logical volume'a ekle.
```bash
sudo lvextend -l +100%FREE /dev/<vg_name>/<lv_name>
```
4. Dosya sistemini buyut.
```bash
# XFS icin
sudo xfs_growfs <mount_point>

# ext4 icin
sudo resize2fs /dev/<vg_name>/<lv_name>
```

## Dogrulama

```bash
sudo pvs
sudo vgs
sudo lvs
df -hT
```

## Sorun Giderme

- Sorun: Yeni disk boyutu gorunmuyor.
  - Cozum:
```bash
sudo partprobe
lsblk
```
- Sorun: `pvresize` hata veriyor.
  - Cozum: Hedef cihaz adini tekrar dogrula ve LVM metadata yedegini kontrol et.

## Referanslar

- `man pvresize`
- `man vgcfgbackup`
- [VDO Olusturma](../vdo/create-vdo.md)
