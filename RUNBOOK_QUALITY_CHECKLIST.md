# Runbook Quality Checklist

Bu checklist, yeni veya guncellenen runbooklarin minimum kalite standardini
karsiladigini dogrulamak icin kullanilir.

## Icerik Kalitesi

- [ ] Amac bolumu net ve tek cumlelik hedef veriyor
- [ ] Kapsam bolumu etki alanini acikca belirtiyor
- [ ] Onkosullar, yetki/paket/ortam gereksinimlerini iceriyor
- [ ] Risk ve rollback adimlari net, uygulanabilir ve test edilebilir
- [ ] Adimlar kronolojik ve kopyalanabilir komut bloklariyla yazildi
- [ ] Dogrulama komutlari degisikligin basarisini olcuyor
- [ ] Sorun giderme bolumunde en az 2 failure-mode var

## Teknik Kalite

- [ ] Dosya adi ASCII ve `kebab-case`
- [ ] Tum komut bloklari `bash` fenced block
- [ ] Placeholder isimleri standart formatta (`<vg_name>` vb.)
- [ ] Yerel linkler kirik degil

## Operasyonel Kalite

- [ ] Uretim riski olan adimlar icin yedek alma komutu yazildi
- [ ] On kontrol adiminda mevcut durum snapshot'i alinabiliyor
- [ ] Geri donus adimlari gercekten uygulanabilir
- [ ] Distro farklari varsa not dusuldu
