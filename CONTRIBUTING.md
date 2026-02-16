# Contributing Guide

Bu repo, runbook kalitesini korumak icin standart bir akis kullanir.

## Yeni Dokuman Ekleme

1. `templates/runbook-template.md` dosyasini kopyala.
2. Dosyayi uygun konu klasorune koy:
   - `storage/`
   - `network/`
3. Dosya adini `kebab-case` ve ASCII olacak sekilde ver.
4. `STYLE_GUIDE.md` kurallarina gore icerigi tamamla.
5. `RUNBOOK_QUALITY_CHECKLIST.md` listesini dogrula.

## Zorunlu Kontroller

- Her runbook 9 zorunlu basligi icermelidir.
- Riskli komutlarda on kontrol, yedek, dogrulama ve rollback adimi zorunludur.
- Kod bloklari `bash` fenced block formatinda yazilmalidir.
- Runbook metadata tablosu doldurulmalidir.

## Commit ve PR

- Kisa ve anlamli commit mesaji kullan.
- Onerilen commit formati: `docs(<scope>): <kisa-aciklama>`
- PR aciklamasinda su bolumleri doldur:
  - Amac
  - Kapsam
  - Risk
  - Dogrulama adimlari
- `.github/pull_request_template.md` formatini takip et.

## PR Checklist

- [ ] Dosya adi `kebab-case` ve ASCII
- [ ] Zorunlu basliklar tam
- [ ] Komutlar test/validasyon adimi iceriyor
- [ ] Rollback adimi var
- [ ] Runbook metadata tablosu dolu
- [ ] `RUNBOOK_QUALITY_CHECKLIST.md` dogrulandi
- [ ] CI kontrolleri gecti
