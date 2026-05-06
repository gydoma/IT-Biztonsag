# IT-Biztonság – Cheatsheet

## 1) Helyreállítás és root hozzáférés

### Rescue mód (kernel paraméter)
```bash
rd.break
```

### Rendszerpartíció írhatóvá tétele
```bash
mount -o remount,rw /sysroot
```

### Belépés a chroot környezetbe
```bash
chroot /sysroot
```

### Jelszócsere
```bash
passwd
```

### SELinux újracímkézés indítása
```bash
touch ./autorelabel
```

### Felhasználóváltás
```bash
su
```

## 2) Felhasználók és jelszavak

### Felhasználó létrehozás
```bash
useradd user1
```

### Felhasználó azonosítók
```bash
id user1
```

### Csoporttagság módosítás
```bash
usermod user1 -G users
```

### Jelszó zárolása
```bash
passwd -l user1
```

### Jelszó feloldása
```bash
passwd -u user1
```

### Zárolási státusz
```bash
passwd -S user1
```

### Jelszó-életciklus listázás
```bash
chage -l user1
```

### Utolsó jelszócsere dátum beállítás
```bash
chage -d 2020-01-01 user1
```

### Max. jelszó-élettartam beállítás
```bash
chage -M 10 user1
```

### Teljes jelszó-életciklus beállítás
```bash
chage -d 2020-01-01 -m 0 -M 30 -I 30 -E 2020-04-01 user1
```

### Shadow mezők ellenőrzése
```bash
grep user1 /etc/shadow | cut -d : -f 1,3-8
```

## 3) Sudo és jogosultság delegálás

### Program helye
```bash
which shutdown
```

### Link követése státuszban
```bash
stat -L /usr/sbin/shutdown
```

### Sudoers szerkesztés
```bash
visudo
```

### Sudoers sor hozzáadása
```bash
echo "admin localhost=(root) /usr/sbin/shutdown" >>/etc/sudoers
```

### Sudo futtatás más néven
```bash
sudo -u root shutdown
```

### Jelszó nélküli sudo engedély
```bash
echo "admin localhost=(root) NOPASSWD:/usr/sbin/reboot" >>/etc/sudoers
```

### Hostnév módosítás
```bash
hostnamectl set-hostname it-security-vm
```

## 4) SSH és kulcskezelés

### Kulcspár generálás
```bash
ssh-keygen
```

### Kulcspár generálás névvel
```bash
ssh-keygen -f new-key
```

### RSA kulcs 4096 bit
```bash
ssh-keygen -t rsa -b 4096
```

### Bejelentkezés SSH-val
```bash
ssh ssh-user-2@localhost
```

### Távoli parancs futtatása
```bash
ssh ssh-user-2@localhost pwd
```

### authorized_keys létrehozás
```bash
cat .ssh/id_rsa.pub >.ssh/authorized_keys
```

### authorized_keys bővítés
```bash
cat new-key.pub >>.ssh/authorized_keys
```

### .ssh jogosultságok
```bash
chmod 700 /home/ssh-user-2/.ssh
```

### authorized_keys jogosultságok
```bash
chmod 600 /home/ssh-user-2/.ssh/authorized_keys
```

### Kulcs másolása szerverre
```bash
ssh-copy-id ssh-user-3@localhost
```

### Privát kulcs jelszavas védése
```bash
ssh-keygen -p -f .ssh/id_rsa
```

## 5) Jogosultságok, ACL, SELinux, linkek

### Alap jogosultságok listázása
```bash
ls -l
```

### Jogosultság módosítás
```bash
chmod 640 file.txt
```

### Tulajdonos/csoport módosítás
```bash
chown user:group file.txt
```

### Alap jogosultság maszk
```bash
umask
```

### ACL lekérdezés
```bash
getfacl file.txt
```

### ACL beállítás
```bash
setfacl -m u:user:rwx file.txt
```

### SELinux kontextus lekérdezés
```bash
ls -Z /etc/shadow*
```

### SELinux kontextus ideiglenes módosítás
```bash
chcon -t httpd_sys_content_t /var/www/html/index.html
```

### SELinux kontextus visszaállítás
```bash
restorecon -v /var/www/html/index.html
```

### Hard link
```bash
ln source.txt link.txt
```

### Soft link
```bash
ln -s /path/source /path/link
```

## 6) Tűzfal és szolgáltatások

### HTTP szerver telepítés
```bash
dnf install -y httpd
```

### Hallgatózó portok
```bash
ss -ltnp
```

### Firewalld státusz
```bash
firewall-cmd --state
```

### Szolgáltatás engedélyezése zónában
```bash
firewall-cmd --zone=internal --add-service=http
```

### Futó szabályok mentése
```bash
firewall-cmd --runtime-to-permanent
```

### Szolgáltatás definíciók
```bash
cat /usr/lib/firewalld/services/http.xml
```

### Netfilter szabályok listázása
```bash
nft list ruleset
```

### Cockpit indítása
```bash
systemctl start cockpit
```

## 7) Hálózati forgalom rögzítés

### Interfész állapot
```bash
ip link show dev enp0s3
```

### Promiscuous mód bekapcsolás
```bash
ip link set enp0s3 promisc on
```

### Dumpcap 30 másodpercig
```bash
dumpcap -i enp0s3 -w /tmp/1.pcap -a duration:30
```

### Szűrés host szerint
```bash
dumpcap -i enp0s3 -f "host 10.0.0.1"
```

### ICMP szűrés
```bash
dumpcap -i enp0s3 -f "host 10.0.0.1 && (icmp[0] == 0 || icmp[0] == 8 )"
```

### TCP port szűrés
```bash
dumpcap -i enp0s3 -f "host 10.0.0.1 && (tcp[0:2] == 8080 )"
```

## 8) Titkosított tárolók (LUKS, loop)

### Eszközlista fájlrendszerrel
```bash
lsblk -f
```

### Nyers fájl létrehozás
```bash
dd if=/dev/zero of=/device0 bs=4096 count=1024
```

### Fájl gyors allokálás
```bash
fallocate -l 4MiB /device1
```

### Loop eszköz hozzárendelés
```bash
losetup -f /device0
```

### Loop eszközök listázása
```bash
losetup -a
```

### LUKS formázás
```bash
cryptsetup luksFormat /dev/sdb1
```

### LUKS megnyitás
```bash
cryptsetup luksOpen /dev/sdb1 enc-home
```

### LUKS lezárás
```bash
cryptsetup luksClose enc-home
```

### Ext2 fájlrendszer létrehozás
```bash
mkfs.ext2 /dev/mapper/enc-home
```

### XFS fájlrendszer létrehozás
```bash
mkfs.xfs /dev/mapper/secret
```

### Csatolás
```bash
mount /dev/mapper/enc-home /home
```

### Csatolás leválasztás
```bash
umount /home
```

### Crypttab ellenőrzés
```bash
cat /etc/crypttab
```

### Fstab ellenőrzés
```bash
cat /etc/fstab | grep -v '#'
```

### Titkosított tartalom ellenőrzés
```bash
grep -ac "Secret" /device1
```

## 9) Kulcsfájl és automount

### Kulcsfájl generálás
```bash
dd if=/dev/random of=/keys/home.key bs=16 count=1
```

### Kulcsfájl jogosultság
```bash
chmod 000 /keys/home.key
```

### LUKS kulcsfájllal
```bash
cryptsetup luksFormat --key-file /keys/home.key /dev/sdb1 secret
```

### Kulccsal nyitás
```bash
cryptsetup luksOpen /dev/sdb1 secret
```

### Automount szerkesztés
```bash
nano /etc/fstab
```

### Crypttab szerkesztés
```bash
nano /etc/crypttab
```

## 10) OpenSSL – szimmetrikus titkosítás

### AES-128 ECB titkosítás
```bash
openssl aes-128-ecb -in file -out file.aes -K 012345678901234567890123456789ab
```

### AES-128 ECB visszafejtés
```bash
openssl aes-128-ecb -d -in file.aes -K 012345678901234567890123456789ab
```

### AES-128 CBC jelszóval
```bash
openssl aes-128-cbc -k PASSWORD -in file -out file.aes -p
```

### AES-128 CBC visszafejtés
```bash
openssl aes-128-cbc -d -k PASSWORD -in file.aes
```

### DES ECB titkosítás
```bash
openssl des-ecb -in date -K AC22B54854967CA9 -out date.enc
```

### DES ECB visszafejtés
```bash
openssl des-ecb -d -in date.enc -K AC22B54854967CA9
```

### DES CBC inicializáló vektorral
```bash
openssl des-cbc -in zeros -K AC22B54854967CA9 -iv 0 -nosalt | xxd
```

### TDES titkosítás
```bash
openssl des-ede-cbc -in date -k PASSWORD -out date.tdes -p
```

### TDES visszafejtés
```bash
openssl des-ede-cbc -d -in date.tdes -k PASSWORD
```

### Hex dump
```bash
xxd file
```

## 11) OpenSSL – RSA, aláírás, PKCS

### RSA kulcspár generálás
```bash
openssl genpkey -algorithm RSA -out key.pem -pkeyopt rsa_keygen_bits:2048
```

### Publikus kulcs export
```bash
openssl rsa -in key.pem -pubout -out key.pem.pub
```

### RSA kulcsméret ellenőrzés
```bash
openssl rsa -in private.pem -noout -text | head -n 1
```

### Aláírás készítése
```bash
openssl dgst -sign key.pem -sha256 -out document.sign document
```

### Aláírás ellenőrzés
```bash
openssl dgst -verify key.pem.pub -sha256 -signature document.sign document
```

### RSA titkosítás (public)
```bash
openssl rsautl -encrypt -pubin -inkey public.pem -in document -out document-enc1
```

### RSA visszafejtés (private)
```bash
openssl rsautl -decrypt -inkey private.pem -in document-enc1
```

### RSA titkosítás (private)
```bash
openssl rsautl -sign -inkey private.pem -in document -out document-enc2
```

### RSA visszafejtés (public)
```bash
openssl rsautl -inkey public.pem -pubin -in document-enc2
```

### PKCS#8 konvertálás
```bash
openssl pkcs8 -topk8 -in ssh-generated-key -nocrypt -out pkcs8-key
```

### PKCS#8 titkosítás
```bash
openssl pkcs8 -topk8 -in ssh-generated-key -out pkcs8-key.encrypted
```

### PKEYUTIL titkosítás
```bash
openssl pkeyutl -encrypt -inkey pubkey -pubin -in file -out file.enc
```

### PKEYUTIL visszafejtés
```bash
openssl pkeyutl -decrypt -inkey privkey -in file.enc -out file
```

## 12) OpenSSL – tanúsítványok

### CA privát kulcs
```bash
openssl genrsa -out ca.key 2048
```

### CA CSR készítés
```bash
openssl req -new -key ca.key -out ca.csr
```

### CA tanúsítvány kiállítás
```bash
openssl x509 -req -in ca.csr -out ca.crt -days 1000 -signkey ca.key
```

### CA tanúsítvány ellenőrzés
```bash
openssl x509 -text -noout -in ca.crt
```

### Web CSR készítés
```bash
openssl req -new -key web.key -out web.csr
```

### Web tanúsítvány kiállítás
```bash
openssl x509 -req -in web.csr -out web.crt -days 100 -CA ca.crt -CAkey ca.key -CAserial ca.srl -CAcreateserial
```

### Trust store lista
```bash
trust list | grep HARICA
```

### CA tanúsítvány telepítés
```bash
sudo trust anchor ca.crt
```

## 13) Kulcstranszformáció (SSH/OpenSSL)

### OpenSSH publikus kulcs export
```bash
ssh-keygen -f ssh-generated-key -e
```

### PEM export
```bash
ssh-keygen -f ssh-generated-key -e -m pem
```

### PKCS#8 export
```bash
ssh-keygen -f ssh-generated-key.pub -e -m pkcs8
```

### PKCS#1 export fájlba
```bash
ssh-keygen -f ssh-generated-key.pub -e -m pem >rsa-public-key
```

### Import OpenSSH formátumba
```bash
ssh-keygen -i -f rsa-public-key -m pem
```

### OpenSSL pubkey export
```bash
openssl rsa -in ssh-generated-key -pubout
```

### PKCS#1 -> PKCS#8 konverzió
```bash
openssl rsa -RSAPublicKey_in -in rsa-public-key -pubout >public-key
```

## 14) Windows biztonsági eszközök

### Audit házirend
```bash
gpedit.msc
```

### Event Viewer
```bash
eventvwr.msc
```
