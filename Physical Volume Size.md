## Physical Volume Disk Alanı Genişletme
Örnek olarak sanallaltırma sistemi üzerinde yer alan VM baz alınmıştır.

**Aşamalar**
> Disk Alanı 4TB -> 10TB Yükseltildi
> 
> Sunucu restart edildi
> 
> Disk alanı kontrolü vedisk alanı genişletmek için

    lsblk
    pvresize /dev/sdb ile fiziksel alan genişletildi.
