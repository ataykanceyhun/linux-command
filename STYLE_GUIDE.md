# Style Guide

Bu dokuman, repo genelinde tek tip runbook kalitesi saglamak icin kullanilir.

## Dil ve Yazim

- Ana dil Turkce.
- Teknik komutlar ve Linux terimleri orijinal haliyle kalabilir.
- Gerektiginde distro farklari kisa not olarak belirtilir:
  - `RHEL/CentOS notu: ...`
  - `Ubuntu/Debian notu: ...`

## Dosya ve Baslik Standardi

- Dosya adlari ASCII ve `kebab-case` olmalidir.
- Her runbook su basliklari ayni sira ile icermelidir:
  1. `# Baslik`
  2. `## Amac`
  3. `## Kapsam`
  4. `## Onkosullar`
  5. `## Risk ve Geri Donus (Rollback)`
  6. `## Adimlar`
  7. `## Dogrulama`
  8. `## Sorun Giderme`
  9. `## Referanslar`

## Komut Formati

- Tum komutlar fenced code block icinde ve `bash` etiketiyle yazilir.
- Her kritik adimdan sonra en az bir dogrulama komutu bulunur.
- Placeholder formati standarttir:
  - `<disk_device>`
  - `<vg_name>`
  - `<mount_point>`

## Guvenlik ve Guardrail

Riskli komut iceriginde asagidaki akis zorunludur:

1. On kontrol (`lsblk`, `pvs`, `vgs`, `lvs`, mevcut kural/config goruntusu)
2. Yedekleme (`cp`, `iptables-save`, `vgcfgbackup` vb.)
3. Degisiklik uygulama
4. Dogrulama
5. Rollback komutu
