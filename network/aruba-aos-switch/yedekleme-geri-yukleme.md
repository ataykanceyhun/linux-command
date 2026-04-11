# ArubaOS-Switch Konfigürasyon Yedekleme ve Geri Yükleme

| Alan | Deger |
| --- | --- |
| Risk | High |
| Son Dogrulama | 2026-04-11 |
| Tahmini Sure | 15-30 dk |
| Kesinti Etkisi | Tam (geri yukleme adimlarinda reload gerekir) |

## Amac

ArubaOS-Switch uzerindeki konfigurasyonu TFTP sunucusu veya USB bellek
araciligiyla yedeklemek; gerektiginde yedeği geri yukleyerek cihazikullanilabilir hale getirmek.

## Kapsam

- HPE Aruba 2xxx, 3xxx, 5xxx serisi switchler (ArubaOS-Switch / ProCurve OS)
- Planlı bakim oncesi tam yedekleme
- Konfigürasyon hatasi veya cihaz degisimi sonrasi geri yukleme
- Periyodik arşivleme senaryolari

## Onkosullar

- Manager seviyesinde yetkili kullanici
- TFTP senaryosu icin: Switch'in ulasabilecegi bir TFTP sunucusu aktif olmali
  ve hedef dizinde yazma izni bulunmali
- USB senaryosu icin: FAT32 formatli USB bellek switch USB portuna takili olmali
  (USB destegi olmayan modellerde bu senaryo calismaz)
- Geri yukleme adimlari switch'i yeniden baslatir; bakim penceresi alinmali
- `show flash` ile yeterli flash alani oldugu dogrulanmali

| Degisken | Aciklama | Ornek |
| --- | --- | --- |
| `<tftp_server>` | TFTP sunucu IP adresi | `192.168.1.100` |
| `<backup_file>` | Uzaktaki yedek dosya adi | `sw-core-01-2026-04-11.cfg` |
| `<restore_file>` | Geri yuklenecek dosya adi | `sw-core-01-2026-04-11.cfg` |
| `<usb_file>` | USB uzerindeki dosya adi | `sw-core-01.cfg` |
| `<hostname>` | Switch hostname | `sw-core-01` |

## Risk ve Geri Donus (Rollback)

- Riskler:
  - `copy tftp startup-config` sonrasi `reload` yapilmadan once
    konfigurasyonun dogrulanmamasi hatali sekilde uygulanabilir
  - Yanlis dosyanin geri yuklenmesi cihazi erisim disi birakabilir
  - `erase startup-config` + `reload` geri alinmaz; oncesinde mutlaka yedek alinmali
- Yedekleme (bu runbook'un birincil amaci):
```bash
copy running-config tftp <tftp_server> <backup_file>
```
- Rollback: Geri yukleme sirasinda yanlis dosya uygulandiysa onceki yedegi geri yukle:
```bash
copy tftp startup-config <tftp_server> <onceki_backup_file>
reload
```

## Adimlar

### Senaryo A: TFTP ile Konfigürasyon Yedekleme

1. On kontrol: TFTP sunucusuna ulasildigi dogrulanir.
```bash
ping <tftp_server>
```

2. Flash dolulugunu kontrol et.
```bash
show flash
```

3. Calistiran konfigurasyonu (running-config) TFTP'ye yedekle.
```bash
copy running-config tftp <tftp_server> <backup_file>
```

4. Kaydedilmis konfigurasyonu (startup-config) da ayri yedekle.
```bash
copy startup-config tftp <tftp_server> <hostname>-startup-<tarih>.cfg
```

---

### Senaryo B: USB ile Konfigürasyon Yedekleme

1. USB'nin tanindigini dogrula.
```bash
show usb-storage
```

2. Running-config'i USB'ye yaz.
```bash
copy running-config usb <usb_file>
```

3. Startup-config'i USB'ye ayri dosya olarak yaz.
```bash
copy startup-config usb <hostname>-startup.cfg
```

---

### Senaryo C: TFTP'den Konfigürasyon Geri Yükleme

> **Dikkat:** Bu islem startup-config'i degistirir ve `reload` ile
> uygulanir. Cihaz yeniden baslarken baglantilar kesilir.

1. On kontrol: Mevcut konfigurasyonu son kez yedekle.
```bash
copy running-config tftp <tftp_server> <hostname>-oncesi-<tarih>.cfg
```

2. TFTP'deki dosyayi startup-config olarak yukle.
```bash
copy tftp startup-config <tftp_server> <restore_file>
```

3. Dogrulama: Yuklenen startup-config'i reload oncesinde goruntule.
```bash
show config
```

4. Switch'i yeniden baslatarak konfigurasyonu uygula.
```bash
reload
```

Reload sonrasi konsol veya yeni yonetim IP'si uzerinden baglanti sagla.

---

### Senaryo D: USB'den Konfigürasyon Geri Yükleme

1. USB'nin tanindigini dogrula.
```bash
show usb-storage
```

2. Mevcut konfigurasyonu son kez yedekle.
```bash
copy running-config tftp <tftp_server> <hostname>-oncesi-<tarih>.cfg
```

3. USB'deki dosyayi startup-config olarak yukle.
```bash
copy usb startup-config <usb_file>
```

4. Reload ile uygula.
```bash
reload
```

---

### Senaryo E: Running-Config'e Anlık Uygulama (Merge — Reload Gerekmez)

> Sadece mevcut oturumu etkilemek istediginde kullanilir. Dikkatli ol;
> cakisan satirlar mevcut konfigurasyonun uzerine yazilir.

```bash
copy tftp running-config <tftp_server> <restore_file>
```

Uygulamanin kalici olmasini istiyorsan ardindan kaydet:
```bash
write memory
```

---

### Senaryo F: Fabrika Ayarlarina Sifirlama (Factory Reset) Oncesi Tam Yedek

1. Running ve startup konfigurasyonlari yedekle.
```bash
copy running-config tftp <tftp_server> <hostname>-son-backup.cfg
copy startup-config tftp <tftp_server> <hostname>-startup-son-backup.cfg
```

2. Flash icerigini listele; firmware dosyasini da not al.
```bash
show flash
```

3. Startup-config'i sil ve yeniden baslatarak fabrika ayarlarina don.
```bash
erase startup-config
reload
```

## Dogrulama

```bash
show config
```

```bash
show running-config
ping <tftp_server>
show usb-storage
```

Beklenen sonuclar:

- Yedekleme sonrasi TFTP sunucusunda dosya boyutu sifirdan buyuk gorulur
- `copy tftp startup-config` sonrasi `show config` ciktisi geri yuklenen
  konfigurasyonla eslesir
- Reload sonrasi `show running-config` ciktisi beklenen konfigurasyonu gosterir
- Yonetim IP ve hostname dogru sekilde gelir; switch'e SSH veya konsol ile
  baglanilabilir

## Sorun Giderme

- Sorun: `copy running-config tftp` komutu hata veriyor veya zaman asimina ugruyor.
  - Sebep: TFTP sunucusuna ulasim yok ya da sunucu yazma izni kapali.
  - Cozum:
```bash
ping <tftp_server>
show ip
```

- Sorun: `copy tftp startup-config` sonrasi reload sirasnda switch erisim disi kaldi.
  - Sebep: Geri yuklenen konfigurasyonda yonetim IP'si veya default gateway
    yanlis yazilmis.
  - Cozum: Konsol (console) kablosuyla fiziksel erisim sag; konfigurasyonu
    duzelt ve tekrar kaydet:
```bash
configure
vlan 1
  ip address <dogru_mgmt_ip> <mask>
  exit
ip default-gateway <gateway>
exit
write memory
```

- Sorun: `show usb-storage` ciktisinda USB gorunmuyor.
  - Sebep: USB bellege takilmamis, FAT32 formatlanmamis veya model USB
    desteklemiyor.
  - Cozum: `show system` ile model dogrula; USB'yi cikar/tak.

- Sorun: `erase startup-config` sonrasi reload yapmadan once fikir degistirdim.
  - Cozum: Reload yapilmadiysa startup-config henuz silinmemistir; TFTP'den
    yedegi geri yukle:
```bash
copy tftp startup-config <tftp_server> <backup_file>
```

- Sorun: Geri yukleme sonrasi bazi komutlar eksik veya yanlis uygulanmis.
  - Sebep: Dosya farkli bir yazilim surumuyle kaydedilmis; surum uyumsuzlugu.
  - Cozum:
```bash
show version
show flash
```

## Referanslar

- `copy running-config tftp` — calistiran konfigurasyonu TFTP'ye kopyalar
- `copy startup-config tftp` — kayitli konfigurasyonu TFTP'ye kopyalar
- `copy tftp startup-config` — TFTP'den startup-config'e geri yukler
- `copy usb startup-config` — USB'den startup-config'e geri yukler
- `show flash` — flash bellek icerik ve doluluk bilgisi
- `show usb-storage` — USB bellek durumu
- HPE Aruba ArubaOS-Switch Yonetim ve Konfigurasyon Kilavuzu (MK-H1002XE)
