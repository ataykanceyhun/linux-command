# iptables Temelleri

## Amac

Temel `iptables` kurallari ile erisim kontrolu saglamak, degisiklikleri
kalicilastirmak ve guvenli rollback akisina sahip olmak.

## Kapsam

- Giris (INPUT) zinciri temel kurallar
- SSH erisim whitelist ornegi
- Kural listeleme ve silme islemleri

## Onkosullar

- `root` veya `sudo` yetkisi
- Sunucuda `iptables` kurulu olmali
- Distro notu:
  - `RHEL/CentOS notu:` `sudo dnf install iptables-services`
  - `Ubuntu/Debian notu:` `sudo apt install iptables iptables-persistent`

## Risk ve Geri Donus (Rollback)

- Riskler:
  - Yanlis kural nedeniyle SSH baglantisinin kesilmesi
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

1. On kontrol: mevcut kurallari gor.
```bash
sudo iptables -S
sudo iptables -L -v --line-numbers
```
2. Mevcut baglantilari kabul et.
```bash
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```
3. SSH icin izin kurali ekle.
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
```

## Sorun Giderme

- Sorun: SSH erisimi kapandi.
  - Cozum: Konsoldan rollback dosyasini yukle.
```bash
sudo iptables-restore < /root/iptables.backup.<timestamp>
```
- Sorun: Kural beklenen zincirde degil.
  - Cozum:
```bash
sudo iptables -L INPUT -n --line-numbers
```

## Referanslar

- `man iptables`
- `man iptables-save`
- `man iptables-restore`
