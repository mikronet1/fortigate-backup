# Bash script ile Fortigate firewall cihazında otomatik Konfigürasyon backup işlemi 

Fortigate firewall cihazında otomatik konfigurasyon yedeklemesi yapıp ilgili yedek dosyasını ftp sunucusuna upload eden bir bash script hazırladım .

  - ubuntu 
  - ftp sunucu
  - crontab
### Bash Script
```sh
#!/bin/bash

fortiip="192.168.17.50"									
fortikullanici="mikronet"								
fortisifre="1234567891100"									
fortihostname="Mikronet"

ftpserver="192.168.17.1"									
ftpkullanici="mikronet"									
ftpsifre="1000987654321"									
									
echo "Backup alınıyor, lütfen bekleyin...."
CMD=$@
NOW=$(date +"_%m-%d-%Y")  

script=$(expect -c "
		spawn ssh -o StrictHostKeyChecking=no $kullanici@$fortigateip -p 22 $CMD
		match_max 100000
		expect \"*?password:*\"
		send -- \"$ftpsifre\r\"
		expect \"*?#*\"
		send -- \"show full-configuration\"
		send -- \"\r\"
		sleep  30
		expect eof
		puts $expect_out()
")
echo "$script" > $fortiip$NOW.txt

ftp -n 192.168.17.1 <<END_SCRIPT
quote USER "$ftpkullanici"
quote PASS "$ftpsifre"
binary
put $fortiip$NOW.txt
bye
ENDFTP
```
Alınan her yedek kaydedilirken, o günün tarihine göre isimlendirilecek.
Alınan yedeği windows sunucusuna göndermek için sunucuda  ftp server kurup , konfig dosyasını da  backup işleminden sonra ftp sunucuna upload ettim.
### Fortigate

Not: firewall a  ssh yapıp "show full-configuration" çıktısı istediğimiz için öncesinde cihazımızda 
```sh
config system console
     set output standard
end
```
şeklinde forti cihazımızda  konfig değişikliliği yapmamız gerekiyor , bu sayede show çıktısı istediğimizde konfigürasyon çıktımızı bir seferde elde edeceğiz  aksi durumda ise  yarım konfigürasyon çıktısı elde edilir.

### Ubuntu
Yukarıdaki Bash scripti ubuntu serverda backup.sh şeklinde kaydediyoruz.Daha sonra script dosyamızı çalıştırabilir duruma getirmek için ;

```sh
chmod +x  backup.sh
```
şeklinde dosyamıza gerekli çalıştırma iznini veriyoruz. 
Bu noktadan sonra scriptimizi ister ./backup.sh ile manuel çalıştırıp yedeğimizi alabiliriz istersek bunu otomatik hale getirip manuel yedek alma işinden kurtuluruz.
### Crontab

Ben burada Crontab görev zamanlayıcısını kullanıp script dosyamızın belirlediğimiz saatlerde otomatik çalışmasını sağladım.
Crontab, belirlediğimiz bir zaman dilimiinde istediğimiz komut,script veya uygulamayı çalıştırmamızı sağlar. 

 /etc/crontab dosyasını editlereyek kullanabiliriz.

Ayrıca ;

```sh
crontab -e 
``` 

komutunu girdikten sonra crontab konfig dosyamız açılır.Eğer dosyamız yok ise dosya otomatik oluşturulur.
Crontab dosyamızı açtıktan sonra crontab dosyamızın sonuna 
```sh
00 20 * * *  /home/ituser/backup.sh 
``` 
komut satırını ekleriz.Burada her gün Saat 20.00'da ilgili /home/ituser dizininde bulunan backup.sh dosyamız run edilsin şeklinde bir girdi hazırlamış olduk.Bu satırı siz kendinie göre editleyebilirsiniz.

