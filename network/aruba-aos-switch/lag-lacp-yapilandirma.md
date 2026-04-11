# ArubaOS-Switch LAG ve LACP Yapilandirma

| Alan | Deger |
| --- | --- |
| Risk | High |
| Son Dogrulama | 2026-04-11 |
| Tahmini Sure | 20-40 dk |
| Kesinti Etkisi | Kismi |

## Amac

ArubaOS-Switch uzerinde Link Aggregation Group (LAG) olusturmak; LACP
(dinamik) veya statik trunk modunu kullanarak portlari birlesik bir mantiksal
arayuze baglamak ve VLAN atamalarini yapilandirmak.

## Kapsam

- HPE Aruba 2xxx, 3xxx, 5xxx serisi switchler (ArubaOS-Switch / ProCurve OS)
- Switch-to-switch ve switch-to-server LAG senaryolari

## Onkosullar

- Manager seviyesinde yetkili kullanici
- LAG'a dahil edilecek portlarin ayni hiz ve duplex modunda olmasi gerekir
- Karsi taraf cihazda da LAG / LACP yapılandırmasi tamamlanmis olmali
- Portlar uzerinde aktif trafik varsa bakim penceresi alinmalidir
- LACP modunda karsi taraf LACP'i desteklemeli; desteklemiyorsa statik trunk
  kullanilmali

| Degisken | Aciklama | Ornek |
| --- | --- | --- |
| `<port_list>` | LAG'a eklenecek portlar | `A1-A2` veya `1,2` |
| `<trk_id>` | Trunk grup numarasi | `1` (trk1 olarak gorunur) |
| `<native_vlan>` | Trunk native (untagged) VLAN | `1` |
| `<tagged_vlan_list>` | Trunk uzerinde gecen VLAN listesi | `10,20,30` |
| `<lag_desc>` | LAG aciklamasi | `uplink-core-sw01` |

## Risk ve Geri Donus (Rollback)

- Riskler:
  - Yanlis port listesiyle trunk olusturmak uplink'i kesebilir
  - LACP yerine statik trunk veya tam tersi kullanildiginda karsi tarafla
    uyumsuzluk olusur ve baglantilar kaybolur
  - Trunk altina dahil edilen portlarin bireysel VLAN atamalari sifirlanir
- Yedekleme:
```bash
show running-config
```

```bash
copy running-config tftp <tftp_server> lag-backup-$(date +%F).cfg
```

- Rollback: LAG'i kaldir ve portlari bagimsiz hale getir.
```bash
configure
no trunk trk<trk_id>
exit
write memory
```

## Adimlar

### Senaryo A: LACP (Dinamik) LAG Olusturma

LACP, karsi tarafla dinamik musaitlik anlasarak portlari otomatik
birlestirmek icin IEEE 802.3ad protokolunu kullanir.

1. On kontrol: Mevcut trunk ve port durumlarini goruntule.
```bash
show trunks
show lacp
show interfaces brief
```

2. Portlarin bireysel olarak up oldugunu dogrula.
```bash
show interfaces <port_list>
```

3. LACP trunk olustur.
```bash
configure
trunk <port_list> trk<trk_id> lacp
```

4. Trunk arayuzune aciklama ekle.
```bash
interface trk<trk_id>
  name "<lag_desc>"
  exit
```

5. Trunk arayuzune VLAN atamasini yap.
```bash
vlan <tagged_vlan_list>
  tagged trk<trk_id>
  exit
vlan <native_vlan>
  untagged trk<trk_id>
  exit
```

6. Kaydet.
```bash
exit
write memory
```

---

### Senaryo B: Statik Trunk (LAG) Olusturma

Karsi taraf LACP desteklemiyorsa (eski switch, NIC teaming - balance-xor vb.)
statik trunk kullanilir.

1. On kontrol.
```bash
show trunks
show interfaces brief
```

2. Statik trunk olustur.
```bash
configure
trunk <port_list> trk<trk_id> trunk
```

3. Trunk arayuzune aciklama ve VLAN atamasini yap.
```bash
interface trk<trk_id>
  name "<lag_desc>"
  exit
vlan <tagged_vlan_list>
  tagged trk<trk_id>
  exit
vlan <native_vlan>
  untagged trk<trk_id>
  exit
```

4. Kaydet.
```bash
exit
write memory
```

---

### Senaryo C: Mevcut LAG'a Port Ekleme

1. On kontrol.
```bash
show trunks
show lacp
```

2. Yeni portu mevcut trunk grubuna ekle.
```bash
configure
trunk <yeni_port> trk<trk_id> lacp
exit
write memory
```

---

### Senaryo D: LAG'i Kaldirma

1. On kontrol: LAG ustundeki VLAN atamalarini not al.
```bash
show trunks
show vlans
```

2. LAG'i kaldir.
```bash
configure
no trunk trk<trk_id>
exit
write memory
```

Kaldininca portlar bagimsiz hale gelir; gerekirse tek tek VLAN yeniden atanir.

## Dogrulama

```bash
show trunks
```

```bash
show lacp
show interfaces trk<trk_id>
show vlans trk<trk_id>
```

Beklenen sonuclar:

- `show trunks` ciktisinda `trk<trk_id>` ve icerisindeki portlar gorulur
- `show lacp` ciktisinda LACP senaryosunda tum portlarin `Active` veya
  `Passive` durumunda oldugu ve `Partner` bilgilerinin dolu oldugu gorulur
- `show interfaces trk<trk_id>` ciktisinda arayuzun `Up` oldugu gorulur
- `show vlans trk<trk_id>` ciktisinda tagged ve untagged VLAN atamalari dogru
  gorulur

## Sorun Giderme

- Sorun: `show lacp` ciktisinda portlar `Blocked` veya `Down` gorunuyor.
  - Sebep: Karsi taraf LACP aktif degil veya port hiz/duplex uyumsuzlugu var.
  - Cozum:
```bash
show lacp peer
show interfaces <port_list>
```

- Sorun: Trunk uzerinden VLAN trafiği gecmiyor.
  - Sebep: VLAN switch uzerinde tanimli degil veya trk arayuzune tagged
    olarak atanmamis.
  - Cozum:
```bash
show vlans
show vlans trk<trk_id>
```

- Sorun: LAG olusturuldu ancak karsi tarafla baglantiyi kesildi.
  - Sebep: Karsi taraf statik trunk beklerken bu tarafta LACP secildi
    (veya tersi).
  - Cozum: Karsi taraf konfigurasyonu dogrula; gerekirse trunk modunu
    eslestir. Rollback icin:
```bash
configure
no trunk trk<trk_id>
exit
write memory
```

- Sorun: `trunk` komutu hatasi veriyor, port eklenemiyor.
  - Sebep: Port hali hazirda baska bir trunk grubuna dahil.
  - Cozum:
```bash
show trunks
```

## Referanslar

- `show trunks` — trunk grup ozeti
- `show lacp` — LACP durumu ve partner bilgisi
- `show interfaces trk<trk_id>` — LAG arayuz detayi
- IEEE 802.3ad — Link Aggregation Control Protocol
- HPE Aruba ArubaOS-Switch Advanced Traffic Management Kilavuzu (MK-H1010XE)
