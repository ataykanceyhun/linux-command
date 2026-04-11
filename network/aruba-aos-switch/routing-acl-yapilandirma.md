# ArubaOS-Switch Routing ve ACL Yapilandirma

| Alan | Deger |
| --- | --- |
| Risk | High |
| Son Dogrulama | 2026-04-11 |
| Tahmini Sure | 30-60 dk |
| Kesinti Etkisi | Kismi |

## Amac

ArubaOS-Switch uzerinde IP routing aktif ederek inter-VLAN yonlendirmesi
yapmak, statik rotalar tanimlamak ve ACL (Access Control List) ile trafik
filtrelemesi uygulamak.

## Kapsam

- HPE Aruba 3xxx, 5xxx serisi Layer-3 switchler (ArubaOS-Switch / ProCurve OS)
- VLAN'lar arasi yonlendirme (routing), statik rota ve ACL senaryolari
- Not: Layer-2 only modeldeki switchlerde (bazi 2xxx serisi) ip routing
  desteklenmeyebilir; `show system` ile kontrol edin

## Onkosullar

- Manager seviyesinde yetkili kullanici
- Ilgili VLAN'lar switch uzerinde tanimlanmis olmali
- Adres plani (IP, mask, gateway) hazirlanmis olmali
- ACL degisikliklerinde etkilenecek trafik belirlenmeli ve bakim penceresi alinmali
- Dikkat: ACL'nin yanlis VLAN'a ya da yanlis yonde uygulanmasi trafigi tamamen
  durdurabilir

| Degisken | Aciklama | Ornek |
| --- | --- | --- |
| `<vlan_id>` | VLAN numarasi | `10` |
| `<svi_ip>` | VLAN SVI IP adresi | `192.168.10.1` |
| `<svi_mask>` | SVI subnet mask | `255.255.255.0` |
| `<dest_network>` | Statik rota hedef agi | `10.0.0.0` |
| `<dest_mask>` | Hedef ag maskesi | `255.0.0.0` |
| `<next_hop>` | Sonraki atlama IP'si | `192.168.1.254` |
| `<acl_name>` | ACL adi | `sunucu-giris-filtre` |
| `<src_ip>` | ACL kaynak IP | `192.168.20.0` |
| `<src_wildcard>` | Kaynak wildcard mask | `0.0.0.255` |
| `<dst_ip>` | ACL hedef IP | `192.168.10.0` |
| `<dst_wildcard>` | Hedef wildcard mask | `0.0.0.255` |
| `<direction>` | ACL uygulama yonu | `in` veya `out` |

## Risk ve Geri Donus (Rollback)

- Riskler:
  - IP routing aktif edildiginde switch layer-2 davranisi degisir
  - Yanlis ACL yonu veya kural sirasi tum trafigi engelleyebilir
  - SVI IP'si agin mevcut gateway'iyle catisirsa yonlendirme dongusu olusabilir
- Yedekleme:
```bash
show running-config
```

```bash
copy running-config tftp <tftp_server> routing-backup-$(date +%F).cfg
```

- Rollback: ACL kaldirma
```bash
configure
no vlan <vlan_id> ip access-group <acl_name> <direction>
no ip access-list extended <acl_name>
exit
write memory
```

- Rollback: IP routing devre disi
```bash
configure
no ip routing
exit
write memory
```

## Adimlar

### Senaryo A: IP Routing Aktif Etme ve Inter-VLAN Routing

1. On kontrol: Routing durumu ve mevcut VLAN'lari goruntule.
```bash
show ip
show vlans
show ip route
```

2. IP routing'i etkinlestir.
```bash
configure
ip routing
```

3. Birinci VLAN icin SVI IP'si tanimla.
```bash
vlan <vlan_id>
  ip address <svi_ip> <svi_mask>
  exit
```

4. Diger VLAN'lar icin ayni adimlari tekrarla (her VLAN icin bir SVI).

5. Kaydet.
```bash
exit
write memory
```

---

### Senaryo B: Statik Rota Tanimlama

1. On kontrol: Mevcut rota tablosunu goruntule.
```bash
show ip route
```

2. Statik rota ekle.
```bash
configure
ip route <dest_network> <dest_mask> <next_hop>
```

3. Default route ekle (internet cikisi veya upstream gateway).
```bash
ip route 0.0.0.0 0.0.0.0 <next_hop>
```

4. Kaydet.
```bash
exit
write memory
```

---

### Senaryo C: ACL Olusturma ve VLAN'a Uygulama

ACL kurallari sirali olarak islenir; eslesme bulunduğunda islem durur.
Son satirda her zaman gizli bir `deny any any` bulunur; acikca eklenmesi
onerilen bir pratiktir.

1. On kontrol: Mevcut ACL'leri goruntule.
```bash
show access-list
show ip access-lists
```

2. ACL olustur ve kurallari tanimla.
```bash
configure
ip access-list extended <acl_name>
  permit tcp <src_ip> <src_wildcard> <dst_ip> <dst_wildcard> eq 80
  permit tcp <src_ip> <src_wildcard> <dst_ip> <dst_wildcard> eq 443
  permit icmp <src_ip> <src_wildcard> <dst_ip> <dst_wildcard>
  deny ip any any log
  exit
```

3. ACL'yi VLAN SVI arayuzune uygula.
```bash
vlan <vlan_id>
  ip access-group <acl_name> <direction>
  exit
```

4. Kaydet.
```bash
exit
write memory
```

---

### Senaryo D: ACL Kurali Guncelleme (Yeniden Yazma)

ArubaOS-Switch'te bireysel ACL satiri silme yoktur; tum ACL'yi kaldir ve
yeniden olustur.

1. ACL'yi VLAN'dan kaldir.
```bash
configure
no vlan <vlan_id> ip access-group <acl_name> <direction>
```

2. ACL'yi sil.
```bash
no ip access-list extended <acl_name>
```

3. Guncellenmis ACL'yi yeniden olustur (Senaryo C adimlarini takip et).

4. Kaydet.
```bash
exit
write memory
```

## Dogrulama

```bash
show ip route
```

```bash
show ip
show access-list <acl_name>
show statistics vlan <vlan_id>
```

Beklenen sonuclar:

- `show ip route` ciktisinda dogrudan bagli VLAN aglari ve statik rotalar
  gorulur
- `show ip` ciktisinda routing statusu `IP Routing Enabled: Yes` gorunur
- `show access-list <acl_name>` ciktisinda kurallar ve hit sayaci gorulur
  (trafik gectikten sonra artmaya baslar)
- Kaynak VLAN'dan hedef VLAN'a ping basarili olur:
```bash
ping <hedef_host_ip> source vlan <kaynak_vlan_id>
```

## Sorun Giderme

- Sorun: `show ip route` ciktisinda dogrudan VLAN agi gorulmuyor.
  - Sebep: SVI IP atanmamis veya VLAN'da aktif port yok.
  - Cozum:
```bash
show vlans <vlan_id>
show ip
```

- Sorun: VLAN'lar arasi ping calismiyor, routing aktif.
  - Sebep: ACL trafigi engelliyor veya SVI IP yanlis subnet'te.
  - Cozum:
```bash
show access-list <acl_name>
show ip route
ping <svi_ip> source vlan <kaynak_vlan_id>
```

- Sorun: ACL uygulandiktan sonra tum trafik kesildi.
  - Sebep: Kural siralamasinda `deny any any` cok erken eklendi veya
    `permit` kurallari yanlis yazildi.
  - Acil rollback:
```bash
configure
no vlan <vlan_id> ip access-group <acl_name> <direction>
exit
write memory
```

- Sorun: Statik rota tanimli, ancak trafik gitmiyor.
  - Sebep: Next-hop erisilebilir degil veya daha spesifik bir rota override
    ediyor.
  - Cozum:
```bash
show ip route
ping <next_hop>
traceroute <dest_network>
```

- Sorun: `ip routing` komutu kabul edilmiyor.
  - Sebep: Switch modeli Layer-3 routing desteklemiyor (L2-only model).
  - Cozum:
```bash
show system
show version
```

## Referanslar

- `show ip route` — rota tablosu
- `show access-list` — ACL kural ve hit sayaci
- `ping ... source vlan ...` — kaynak VLAN belirterek ping testi
- HPE Aruba ArubaOS-Switch Multicast ve Routing Kilavuzu (MK-H1011XE)
- RFC 1812 — Requirements for IP Version 4 Routers
