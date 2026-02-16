# Linux Command Runbooks

Bu repo, Linux operasyonları için profesyonel ve güvenli runbook dokumantasyonu
sunar. Icerik Turkce anlatimla hazirlanir, kritik noktalarda distro farklari
ayri notlar olarak belirtilir.

## Hedef

- Tekrarlanabilir operasyon adimlari
- On kontrol, yedek, dogrulama ve rollback standardi
- Markdown tabanli hafif ama olceklenebilir bilgi mimarisi

## Konu Bazli Dizin

### Storage

- [LVM Physical Volume Buyutme](storage/lvm/resize-physical-volume.md)
- [VDO Olusturma](storage/vdo/create-vdo.md)
- [VDO Kapasite Artirma](storage/vdo/extend-vdo-capacity.md)
- [LVM Uzerinde XFS Volume Olusturma](storage/xfs/create-xfs-on-lvm.md)

### Network

- [iptables Temelleri](network/firewall/iptables-basics.md)

## Hizli Baslangic

1. [STYLE_GUIDE.md](STYLE_GUIDE.md) dosyasini incele.
2. Yeni dokuman icin [templates/runbook-template.md](templates/runbook-template.md)
   dosyasini kopyala.
3. Her prosedurde on kontrol, degisiklik, dogrulama ve rollback adimlarini yaz.
4. Pull request acmadan once CI kontrolunu gec.

## Guvenlik Notu

Bu repodaki komutlar dogrudan uretim sistemlerinde calistirilmadan once test
ortaminda dogrulanmalidir. Riskli islemlerde mutlaka mevcut konfigrasyonu
yedekleyin ve rollback adimini hazirlayin.

## Katki Akisi

Katki kurallari ve PR kontrol listesi icin
[CONTRIBUTING.md](CONTRIBUTING.md) dosyasina bakin.

## Kalite Kontrolu

GitHub Actions workflow dosyasi
`.github/workflows/docs-quality.yml` su kontrolleri calistirir:

- Markdown lint
- Link check
