## Volume Group içinde Disk alanı Genişletme

| Volume Group | Boyut(TB)|
|--|--|
| Kapasite |  4|
| Kullanılan Alan| 3 |
| Boş Alan| 1 |

**Volume Group üzerindeki boş alandan VDO için Kapasite Yükseltme İşlemleri**

> Pool Kapasitesine 1TB Eklemek için
> *İsimler farklılık gösterebilir*

    lvextend -L+1T vg_data/vpool0 

> VDO Kapasitesine 10:1 oranında arttırmak için
> *İsimler farklılık gösterebilir*

    lvextend -L+10T vg_data/vdo1

> Dosya Sistemi Düzeyinde son işlem için

    xfs_growfs /dev/vg_data/vdo1
