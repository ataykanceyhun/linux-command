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

## Metadata Standardi

Her runbook basliginin hemen altinda su alanlari iceren bir tablo bulunur:

- `Risk` (`Low`, `Medium`, `High`)
- `Son Dogrulama` (`YYYY-MM-DD`)
- `Tahmini Sure`
- `Kesinti Etkisi`

## Komut Formati

- Tum komutlar fenced code block icinde ve `bash` etiketiyle yazilir.
- Prompt kullanma (`$` veya `#` yazma), sadece komut satiri ver.
- Her kritik adimdan sonra en az bir dogrulama komutu bulunur.
- Placeholder formati standarttir:
  - `<disk_device>`
  - `<vg_name>`
  - `<lv_name>`
  - `<mount_point>`
  - `<interface_name>`

## Guvenlik ve Guardrail

Riskli komut iceriginde asagidaki akis zorunludur:

1. On kontrol (`lsblk`, `pvs`, `vgs`, `lvs`, mevcut kural/config goruntusu)
2. Yedekleme (`cp`, `iptables-save`, `vgcfgbackup` vb.)
3. Degisiklik uygulama
4. Dogrulama
5. Rollback komutu

## Dogrulama Kalitesi

- Dogrulama bolumunde en az 2 komut bulunmali.
- En az bir komut degisiklik etkisini acik gostermeli (`df -hT`, `mount`, `iptables -S`).
- Sorun giderme bolumu en az 2 farkli failure-mode icermeli.
