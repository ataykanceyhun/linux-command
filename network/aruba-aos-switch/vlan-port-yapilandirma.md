# ArubaOS-Switch VLAN ve Port Yapilandirma

| Alan | Deger |
| --- | --- |
| Risk | Medium |
| Son Dogrulama | 2026-04-11 |
| Tahmini Sure | 15-30 dk |
| Kesinti Etkisi | Kismi |

## Amac

ArubaOS-Switch uzerinde VLAN olusturmak, portlara access (untagged) veya
trunk (tagged) modu atamak ve port guvenligini (port-security) yapilandirmak.

## Kapsam

- HPE Aruba 2xxx, 3xxx, 5xxx serisi switchler (ArubaOS-Switch / ProCurve OS)
- Yeni VLAN ekleme, port modu degistirme ve port guvenlik senaryolari

## Onkosullar

- Manager seviyesinde yetkili kullanici
- VLAN plani ve port haritasi hazirlanmis olmali
- Trunk portlarinda karsi taraf cihazin da ayni VLAN'lari desteklemesi gerekir
- Dikkat: Access port modunu degistirmek bagli cihazin baglantiyi kesmesine
  neden olabilir; bakim penceresi alin

| Degisken | Aciklama | Ornek |
| --- | --- | --- |
| `<vlan_id>` | VLAN numarasi | `10` |
| `<vlan_name>` | VLAN adi | `sunucu-agi` |
| `<access_port>` | Access port numarasi | `5` veya `A5` |
| `<trunk_port>` | Trunk port numarasi | `24` veya `A24` |
| `<native_vlan>` | Trunk native (untagged) VLAN | `1` |
| `<tagged_vlan_list>` | Trunk uzerinde gecen VLAN listesi | `10,20,30` |
| `<mac_address>` | Port security icin izin verilen MAC | `aabbcc-ddeeff` |

## Risk ve Geri Donus (Rollback)

- Riskler:
  - Port modunun yanlis degistirilmesi baglantiyi keser
  - Yanlis VLAN atamasi trafik izolasyonunu bozabilir
  - Trunk'tan native VLAN kaldirilmasi yonetim erisimini keser
- Yedekleme:
```bash
show running-config
```

```bash
copy running-config tftp <tftp_server> vlan-backup-$(date +%F).cfg
```

- Rollback: Yanlis VLAN atamasi yapilan port icin orijinal haline don:
```bash
configure
vlan 1
  untagged <access_port>
  exit
no vlan <vlan_id>
exit
write memory
```

## Adimlar

### Senaryo A: VLAN Olusturma ve Access Port Atama

1. On kontrol: Mevcut VLAN ve port durumunu goruntule.
```bash
show vlans
show interfaces brief
```

2. VLAN olustur.
```bash
configure
vlan <vlan_id>
  name "<vlan_name>"
  exit
```

3. Porta VLAN'i untagged (access) olarak ata.
```bash
interface <access_port>
  untagged vlan <vlan_id>
  exit
```

4. Kaydet.
```bash
exit
write memory
```

---

### Senaryo B: Trunk Port Yapilandirma

1. On kontrol: Trunk yapilacak portu ve karsidaki cihazi kontrol et.
```bash
show vlans
show interfaces <trunk_port>
```

2. Portun mevcut untagged atamasini kaldir (gerekirse).
```bash
configure
no vlan 1 untagged <trunk_port>
```

3. Trunk portuna tagged VLAN listesi ve native VLAN ata.
```bash
vlan <tagged_vlan_list>
  tagged <trunk_port>
  exit
vlan <native_vlan>
  untagged <trunk_port>
  exit
```

4. Kaydet.
```bash
exit
write memory
```

---

### Senaryo C: Port Security Yapilandirma

1. On kontrol: Portun mevcut guvenlik ayarini goruntule.
```bash
show port-security <access_port>
```

2. Portu devre disi birak (degisiklik oncesi).
```bash
configure
interface <access_port>
  disable
  exit
```

3. Port security'yi statik MAC ile etkinlestir.
```bash
port-security <access_port> learn-mode static action send-disable
port-security <access_port> mac-address <mac_address>
```

4. Portu yeniden aktif et.
```bash
interface <access_port>
  enable
  exit
```

5. Kaydet.
```bash
exit
write memory
```

---

### Senaryo D: Port Devre Disi / Aktif Etme

```bash
configure
interface <access_port>
  disable
  exit
exit
```

Yeniden acmak icin:
```bash
configure
interface <access_port>
  enable
  exit
exit
write memory
```

## Dogrulama

```bash
show vlans
```

```bash
show vlans <vlan_id>
show vlan <vlan_id> ports
show port-security <access_port>
show interfaces <trunk_port>
```

Beklenen sonuclar:

- `show vlans` ciktisinda yeni VLAN gorunur
- `show vlans <vlan_id>` ciktisinda port atamalari (untagged/tagged) dogru gorulur
- `show port-security <access_port>` ciktisinda learn-mode ve MAC adresi gorulur
- `show interfaces <trunk_port>` ciktisinda tagged VLAN listesi dogru gorulur

## Sorun Giderme

- Sorun: Baglanmis cihaz VLAN'a erisemiyor.
  - Sebep: Port yanlis VLAN'a untagged atanmis veya hala VLAN 1'de.
  - Cozum:
```bash
show vlan <vlan_id> ports
show interfaces <access_port>
```

- Sorun: Trunk port uzerinden beklenen VLAN'lar gecmiyor.
  - Sebep: VLAN karsi tarafta tanimli degil veya porttan tagged olarak
    aktarilmamis.
  - Cozum:
```bash
show vlans <vlan_id>
show interfaces <trunk_port>
```

- Sorun: Port security devreye girdi, port kapatildi.
  - Sebep: Tanimli MAC disinda bir cihaz portu kullanmaya calisti.
  - Cozum:
```bash
show port-security <access_port>
```

```bash
configure
port-security <access_port> clear-intrusion-flag
interface <access_port>
  enable
  exit
exit
```

- Sorun: `show vlans` ciktisinda VLAN yok.
  - Sebep: VLAN olusturuldu ancak `write memory` yapilmadi ve cihaz yeniden
    baslatildi.
  - Cozum: VLAN'i yeniden olustur ve `write memory` ile kaydet.

## Referanslar

- `show vlans` — tum VLAN listesi
- `show interfaces brief` — tum portlarin ozet durumu
- `show port-security` — port guvenlik durumu
- HPE Aruba ArubaOS-Switch VLAN Yapilandirma Kilavuzu (MK-H1008XE)
