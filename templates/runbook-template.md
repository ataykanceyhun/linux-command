# <Runbook Basligi>

| Alan | Deger |
| --- | --- |
| Risk | <Low|Medium|High> |
| Son Dogrulama | <YYYY-MM-DD> |
| Tahmini Sure | <or: 15 dk> |
| Kesinti Etkisi | <Yok|Kismi|Tam> |

## Amac

Bu prosedurun hedefini 1-2 cumle ile yazin.

## Kapsam

- Hangi sistemler etkilenir?
- Hangi ortamlarda kullanilir? (dev/test/prod)

## Onkosullar

- Gerekli paketler ve yetkiler
- Gerekli cihaz/volume/arayuz bilgileri
- Bakim penceresi veya kesinti notlari

Ornek degisken tablosu:

| Degisken | Aciklama | Ornek |
| --- | --- | --- |
| `<disk_device>` | Fiziksel disk cihazi | `sdb` |
| `<vg_name>` | Volume group adi | `vg_data` |
| `<mount_point>` | Mount noktasi | `/mnt/data` |

## Risk ve Geri Donus (Rollback)

- Riskler:
  - Veri kaybi
  - Servis kesintisi
  - Yanlis hedefte komut calisma riski
- Yedekleme:
```bash
cp /etc/fstab /etc/fstab.bak.$(date +%F-%H%M%S)
```
- Rollback:
```bash
cp /etc/fstab.bak.<timestamp> /etc/fstab
```

## Adimlar

1. On kontrol:
```bash
lsblk
```
2. Degisiklik:
```bash
echo "degisiklik komutu"
```
3. Gerekirse kalici hale getirme:
```bash
echo "persist adimi"
```

## Dogrulama

```bash
df -hT
mount
```

## Sorun Giderme

- Sik hata:
  - Belirti
  - Olasi neden
  - Cozum komutu

## Referanslar

- Dahili dokuman veya resmi kaynak linkleri
