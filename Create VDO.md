**VDO** 

**10:1 Oranında Tekilleştirme için Ayarlar**

> Fiziksel alan belirleniyor

    sudo pvcreate /dev/sdb

> Volume Oluşturuluyor

    sudo vgcreate vg_data /dev/sdb

> VDO alanı oluşturuluyor

    sudo lvcreate --type vdo --name vdo1 --size 1T --virtualsize 30T vg_data

> Oluşturulan alan formatlanıyor

    sudo mkfs.xfs -K /dev/vg_data/vdo1

> Mount edilecek alan için bir klasör oluşturuluyor.

    sudo mkdir /mnt/vdo1

> Oluşturulan VDO alanı mount edilecek klasöre bağlanıyor

    sudo mount -o discard /dev/vg_data/vdo1 /mnt/vdo1

> Mount edilen yolları sistem düzeyinde kayıt ediyoruz

    sudo nano /etc/fstab

    add -> /dev/vg_data/vdo1        /mnt/vdo1       xfs     defaults,discard      0 0
