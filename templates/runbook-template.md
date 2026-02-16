# <Runbook Basligi>

## Amac

Bu prosedurun hedefini 1-2 cumle ile yazin.

## Kapsam

- Hangi sistemler etkilenir?
- Hangi ortamlarda kullanilir? (dev/test/prod)

## Onkosullar

- Gerekli paketler ve yetkiler
- Gerekli cihaz/volume/arayuz bilgileri
- Bakim penceresi veya kesinti notlari

## Risk ve Geri Donus (Rollback)

- Riskler:
  - Veri kaybi
  - Servis kesintisi
  - Yanlis hedefte komut calisma riski
- Yedekleme:
```bash
# Ornek
cp /etc/fstab /etc/fstab.bak.$(date +%F-%H%M%S)
```
- Rollback:
```bash
# Ornek: yedekten geri yukleme
cp /etc/fstab.bak.<timestamp> /etc/fstab
```

## Adimlar

1. On kontrol:
```bash
# Ornek
lsblk
```
2. Degisiklik:
```bash
# Ornek
echo "degisiklik komutu"
```
3. Gerekirse kalici hale getirme:
```bash
# Ornek
echo "persist adimi"
```

## Dogrulama

```bash
# Ornek
df -hT
```

## Sorun Giderme

- Sik hata:
  - Belirti
  - Olasi neden
  - Cozum komutu

## Referanslar

- Dahili dokuman veya resmi kaynak linkleri
