# HPE Aruba ArubaOS-Switch Runbooks

Bu klasor, HPE Aruba 2xxx, 3xxx ve 5xxx serisi switchlerde calistiran
ArubaOS-Switch (ProCurve OS) platformu icin operasyonel runbooklari icerir.

## Runbooklar

| Runbook | Risk | Kapsam |
| --- | --- | --- |
| [Temel Kurulum](temel-kurulum.md) | Low | Hostname, yonetim IP, NTP, SNMP, kullanici |
| [VLAN ve Port Yapilandirma](vlan-port-yapilandirma.md) | Medium | VLAN, access/trunk port, port-security |
| [LAG ve LACP Yapilandirma](lag-lacp-yapilandirma.md) | High | Link aggregation, LACP, statik trunk |
| [Routing ve ACL Yapilandirma](routing-acl-yapilandirma.md) | High | IP routing, inter-VLAN, statik rota, ACL |

## Platform Notu

Bu runbooklar ArubaOS-Switch (eski adi: HP ProCurve Switch OS) icin
hazirlanmistir. ArubaOS-CX (6xxx, 8xxx serisi yeni nesil OS) CLI szdizimi
farklidir; bu runbooklar AOS-CX ile kullanilmamalidir.

## Hizli Referans: Sik Kullanilan Komutlar

```bash
show running-config          # calistiran konfigurasyon
show system                  # sistem bilgisi ve model
show version                 # firmware surumu
show vlans                   # VLAN listesi
show interfaces brief        # tum portlar ozet durum
show trunks                  # LAG/trunk gruplari
show lacp                    # LACP durumu
show ip route                # rota tablosu
show access-list             # tum ACL'ler
write memory                 # konfigurasyonu kaydet
copy running-config tftp ... # konfigurasyonu TFTP ile yedekle
```
