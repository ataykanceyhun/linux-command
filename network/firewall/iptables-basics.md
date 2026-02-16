# iptables Temelleri

| Alan | Deger |
| --- | --- |
| Risk | High |
| Son Dogrulama | 2026-02-16 |
| Tahmini Sure | 10-20 dk |
| Kesinti Etkisi | Tam |

## Amac

Temel `iptables` kurallari ile erisim kontrolu saglamak, degisiklikleri
kalicilastirmak ve guvenli rollback akisina sahip olmak.

## Kapsam

- INPUT zinciri temel kurallar
- SSH whitelist kurali
- Kural listeleme, kaydetme ve silme

## Onkosullar

- `root` veya `sudo` yetkisi
- Sunucuda `iptables` kurulu olmali
- Out-of-band erisim (konsol/ILO) hazir olmali
- Distro notu:
  - `RHEL/CentOS notu:` `sudo dnf install iptables-services`
  - `Ubuntu/Debian notu:` `sudo apt install iptables iptables-persistent`

| Degisken | Aciklama | Ornek |
| --- | --- | --- |
| `<interface_name>` | Ag arayuzu | `eno1` |
| `<source_cidr>` | Kaynak ag | `10.100.22.0/26` |
| `<destination_ip>` | Hedef sunucu IP | `10.100.10.39` |
| `<line_number>` | Silinecek kural satiri | `4` |

## Risk ve Geri Donus (Rollback)

- Riskler:
  - SSH baglantisinin kesilmesi
  - Kalici kural dosyasinin hatali yazilmasi
- Yedekleme:
```bash
sudo iptables-save > /root/iptables.backup.$(date +%F-%H%M%S)
```
- Rollback:
```bash
sudo iptables-restore < /root/iptables.backup.<timestamp>
```

## Adimlar

1. On kontrol snapshot'i al.
```bash
sudo iptables -S
sudo iptables -L -v --line-numbers
```
2. Iliskili mevcut baglantilari izinle.
```bash
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```
3. SSH whitelist kurali ekle.
```bash
sudo iptables -A INPUT -i <interface_name> -p tcp --dport 22 \
  -s <source_cidr> -d <destination_ip> -j ACCEPT
```
4. Kurallari kalici hale getir.
```bash
# Ubuntu/Debian
sudo iptables-save | sudo tee /etc/iptables/rules.v4

# RHEL/CentOS
sudo service iptables save
```
5. Gerekirse satir numarasi ile kural sil.
```bash
sudo iptables -D INPUT <line_number>
```

## Dogrulama

```bash
sudo iptables -L -v --line-numbers
sudo iptables -S
sudo ss -tulpen | grep :22
```

Beklenen sonuc:

- INPUT zincirinde yeni SSH kurali listelenir
- Mevcut SSH oturumu kesilmeden devam eder

## Sorun Giderme

- Sorun: SSH erisimi kapandi.
  - Cozum: Konsoldan rollback dosyasini yukle.
```bash
sudo iptables-restore < /root/iptables.backup.<timestamp>
```
- Sorun: Kural yanlis sirada.
  - Cozum:
```bash
sudo iptables -D INPUT <line_number>
sudo iptables -I INPUT 1 -i <interface_name> -p tcp --dport 22 \
  -s <source_cidr> -d <destination_ip> -j ACCEPT
```
- Sorun: Reboot sonrasi kural kayboluyor.
  - Cozum: Kalicilik adimini distroya uygun sekilde yeniden uygula.

## Referanslar

- `man iptables`
- `man iptables-save`
- `man iptables-restore`
