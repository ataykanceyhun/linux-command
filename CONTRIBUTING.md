# Contributing Guide

Bu repo, runbook kalitesini korumak icin standart bir akis kullanir.

## Yeni Dokuman Ekleme

1. `templates/runbook-template.md` dosyasini kopyala.
2. Dosyayi uygun konu klasorune koy:
   - `storage/`
   - `network/`
3. Dosya adini `kebab-case` ve ASCII olacak sekilde ver.
4. `STYLE_GUIDE.md` kurallarina gore icerigi tamamla.

## Zorunlu Kontroller

- Her runbook 9 zorunlu basligi icermelidir.
- Riskli komutlarda on kontrol, yedek, dogrulama ve rollback adimi zorunludur.
- Kod bloklari `bash` fenced block formatinda yazilmalidir.

## Commit ve PR

- Kisa ve anlamli commit mesaji kullan.
- PR aciklamasinda su bolumleri doldur:
  - Amaç
  - Kapsam
  - Risk
  - Dogrulama adimlari

## PR Checklist

- [ ] Dosya adi `kebab-case` ve ASCII
- [ ] Zorunlu basliklar tam
- [ ] Komutlar test/validasyon adimi iceriyor
- [ ] Rollback adimi var
- [ ] CI kontrolleri gecti
