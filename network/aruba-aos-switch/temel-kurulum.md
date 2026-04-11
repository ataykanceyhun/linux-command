# ArubaOS-Switch Temel Kurulum

| Alan | Deger |
| --- | --- |
| Risk | Low |
| Son Dogrulama | 2026-04-11 |
| Tahmini Sure | 20-30 dk |
| Kesinti Etkisi | Kismi |

## Amac

Yeni bir ArubaOS-Switch cihazina hostname, yonetim IP, NTP, SNMP, banner ve
yerel kullanici tanimlarini yapmak; temel guvenlik ve izlenebilirlik
gereksinimlerini karsilamak.

## Kapsam

- HPE Aruba 2xxx, 3xxx, 5xxx serisi switchler (ArubaOS-Switch / ProCurve OS)
- Ilk konfigurasyonu yapilacak fabrika cikisi veya sifirlama sonrasi cihazlar

## Onkosullar

- Console kablosu ile fiziksel erisim veya mevcut yonetim IP'si uzerinden SSH
- Manager seviyesinde yetkili kullanici
- NTP ve SNMP sunucu bilgileri hazir olmali
- Bakim penceresi: Yapilan degisiklikler anlik etkili olmaz; `write memory` ile
  kalici hale getirilene kadar yeniden baslatmada kaybolur

| Degisken | Aciklama | Ornek |
| --- | --- | --- |
| `<hostname>` | Switch hostname | `sw-core-01` |
| `<mgmt_vlan>` | Yonetim VLAN numarasi | `1` |
| `<mgmt_ip>` | Switch yonetim IP adresi | `192.168.1.10` |
| `<mgmt_mask>` | Subnet mask | `255.255.255.0` |
| `<gateway_ip>` | Default gateway | `192.168.1.1` |
| `<ntp_server>` | NTP sunucu IP | `192.168.1.5` |
| `<community_string>` | SNMP community | `monitoring` |
| `<snmp_contact>` | SNMP iletisim bilgisi | `noc@sirket.com` |
| `<snmp_location>` | Fiziksel konum | `DC1-Rack-A3` |
| `<manager_user>` | Yonetici kullanici adi | `admin` |
| `<operator_user>` | Operasyon kullanici adi | `netops` |

## Risk ve Geri Donus (Rollback)

- Riskler:
  - Yanlis yonetim IP'si girildigi durumda uzak erisim kesilir
  - `write memory` oncesinde cihaz kapanirsa degisiklikler kaybolur
- Yedekleme:
```bash
show running-config
```

```bash
copy running-config tftp <tftp_server> <hostname>-backup-$(date +%F).cfg
```

- Rollback (console erisimi gerektirir):
```bash
erase startup-config
reload
```

## Adimlar

1. On kontrol: Mevcut konfigurasyonu goruntule.
```bash
show running-config
show version
show system
```

2. Konfigurasyon moduna gec.
```bash
configure
```

3. Hostname ayarla.
```bash
hostname <hostname>
```

4. Yonetim VLAN'ina IP ata.
```bash
vlan <mgmt_vlan>
  ip address <mgmt_ip> <mgmt_mask>
  exit
```

5. Default gateway tanimla.
```bash
ip default-gateway <gateway_ip>
```

6. NTP yapilandir.
```bash
timesync ntp
ntp unicast
ntp server <ntp_server>
```

7. SNMP yapilandir.
```bash
snmp-server community "<community_string>" operator
snmp-server contact "<snmp_contact>"
snmp-server location "<snmp_location>"
```

8. Giris banneri tanimla.
```bash
banner motd "Yetkisiz erisim yasaktir. Tum islemler kayit altindadir."
```

9. Console ve SSH oturum zaman asimi ayarla.
```bash
console inactivity-timer 10
```

10. Manager kullanicisi olustur.
```bash
password manager user-name <manager_user>
```

11. Operator kullanicisi olustur.
```bash
password operator user-name <operator_user>
```

12. Konfigurasyon modundan cik ve kaydet.
```bash
exit
write memory
```

## Dogrulama

```bash
show running-config
```

```bash
show system
show ntp status
show snmp-server
show ip
```

Beklenen sonuclar:

- `show system` ciktisinda hostname guncellenmis gorunur
- `show ntp status` ciktisinda NTP sunucusuyla senkronizasyon gorulur
- `show ip` ciktisinda yonetim IP ve gateway dogru atanmis gorunur
- `show snmp-server` ciktisinda community ve konum bilgileri gorulur

## Sorun Giderme

- Sorun: NTP senkronizasyonu saglanamadi.
  - Sebep: Firewall veya ACL NTP (UDP 123) trafiklerini engelliyordur.
  - Cozum:
```bash
show ntp status
show ntp associations
ping <ntp_server>
```

- Sorun: TFTP backup komutu calismiyor.
  - Sebep: TFTP sunucusu erisilebilir degil veya switch'in yonetim IP'si henuz
    dogru atanmamis.
  - Cozum:
```bash
ping <tftp_server>
show ip
```

- Sorun: `write memory` sonrasi konfigurasyonun bir kismi kayboldu.
  - Sebep: Flash bellek dolulugu.
  - Cozum:
```bash
show flash
```

- Sorun: SSH ile baglanti saglanamadi.
  - Sebep: SSH servisi aktif degil.
  - Cozum:
```bash
show crypto host-public-key
crypto key generate ssh
```

## Referanslar

- `show running-config` — tam konfigurasyon goruntuleme
- `show system` — donanim ve yazilim surumu bilgisi
- HPE Aruba ArubaOS-Switch Yonetim ve Konfigurasyon Kilavuzu (MK-H1002XE)
