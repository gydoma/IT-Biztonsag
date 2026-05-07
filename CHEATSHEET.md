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


## 15) Kapcsolók és paraméterek rövid magyarázata (a fenti parancsokhoz)

### 15.1 Helyreállítás és root hozzáférés
- `rd.break`: kernelparaméter, initramfs-ben megáll és root helyreállítást tesz lehetővé.
- `mount -o remount,rw /sysroot`: `-o` opciólista; `remount` újracsatol; `rw` írható mód.
- `chroot /sysroot`: a `/sysroot` lesz az új gyökér, a parancsok azon a rendszeren futnak.
- `passwd`: jelszócsere az aktuális felhasználónak.
- `touch ./autorelabel`: üres fájl létrehozása; SELinux újracímkézés indítása.
- `su`: felhasználóváltás; paraméter nélkül root-ra vált.

### 15.2 Felhasználók és jelszavak
- `useradd user1`: `user1` az új felhasználónév.
- `id user1`: a `user1` UID/GID azonosítói.
- `usermod user1 -G users`: `-G` kiegészítő csoport(ok), `users` a célcsoport.
- `passwd -l user1`: `-l` zárolás (lock) a jelszón.
- `passwd -u user1`: `-u` feloldás (unlock).
- `passwd -S user1`: `-S` státusz lekérdezés.
- `chage -l user1`: `-l` életciklus lista.
- `chage -d 2020-01-01 user1`: `-d` utolsó jelszócsere dátum (YYYY-MM-DD).
- `chage -M 10 user1`: `-M` max. napok a cseréig.
- `chage -d ... -m 0 -M 30 -I 30 -E 2020-04-01 user1`: `-m` min. napok; `-I` inaktív napok; `-E` fiók lejárati dátum.
- `grep user1 /etc/shadow | cut -d : -f 1,3-8`: `-d` elválasztó; `-f` mezők listája.

### 15.3 Sudo és jogosultság delegálás
- `which shutdown`: teljes elérési út megkeresése.
- `stat -L /usr/sbin/shutdown`: `-L` szimbolikus link követése.
- `visudo`: sudoers szerkesztés ellenőrzéssel.
- `echo "..." >>/etc/sudoers`: `>>` hozzáfűzés, sudoers szabály hozzáadása.
- `sudo -u root shutdown`: `-u` célfelhasználó.
- `NOPASSWD:`: jelszó nélküli futtatás.
- `hostnamectl set-hostname it-security-vm`: új hostnév megadása.

### 15.4 SSH és kulcskezelés
- `ssh-keygen`: kulcspár generálás alapértelmezett beállításokkal.
- `ssh-keygen -f new-key`: `-f` fájlnév.
- `ssh-keygen -t rsa -b 4096`: `-t` kulcstípus, `-b` bitméret.
- `ssh user@host`: `user@host` cél azonosítása.
- `ssh user@host pwd`: távoli parancs futtatása.
- `cat .ssh/id_rsa.pub >.ssh/authorized_keys`: publikus kulcs fájl.
- `chmod 700 ~/.ssh`: `700` csak tulajdonosnak teljes jog.
- `chmod 600 ~/.ssh/authorized_keys`: `600` csak tulajdonos olvas/ír.
- `ssh-copy-id user@host`: publikus kulcs feltöltése.
- `ssh-keygen -p -f .ssh/id_rsa`: `-p` jelszócsere a privát kulcson.

### 15.5 Jogosultságok, ACL, SELinux, linkek
- `ls -l`: részletes jogosultságlista.
- `chmod 640 file.txt`: `6/4/0` felhasználó/csoport/más jogosultságok.
- `chown user:group file.txt`: tulajdonos és csoport beállítás.
- `umask`: alap jogosultsági maszk.
- `getfacl file.txt`: ACL lekérdezés.
- `setfacl -m u:user:rwx file.txt`: `-m` módosítás; `u:` felhasználó.
- `ls -Z`: SELinux kontextus.
- `chcon -t httpd_sys_content_t ...`: `-t` típus.
- `restorecon -v ...`: `-v` részletes mód.
- `ln source.txt link.txt`: hard link.
- `ln -s /path/source /path/link`: `-s` szimbolikus link.

### 15.6 Tűzfal és szolgáltatások
- `dnf install -y httpd`: `-y` automatikus igen.
- `ss -ltnp`: `-l` hallgatózó; `-t` TCP; `-n` numerikus; `-p` folyamat.
- `firewall-cmd --state`: firewalld állapot.
- `firewall-cmd --zone=internal --add-service=http`: `--zone` zóna; `--add-service` szolgáltatás.
- `firewall-cmd --runtime-to-permanent`: futó szabályok mentése.
- `nft list ruleset`: netfilter szabályok listázása.
- `systemctl start cockpit`: szolgáltatás indítás.

### 15.7 Hálózati forgalom rögzítés
- `ip link show dev enp0s3`: `dev` cél interfész.
- `ip link set enp0s3 promisc on`: promiszkuitás beállítása.
- `dumpcap -i enp0s3 -w /tmp/1.pcap -a duration:30`: `-i` interfész, `-w` fájl, `-a` automatikus leállás.
- `dumpcap -i enp0s3 -f "host 10.0.0.1"`: `-f` BPF szűrő.

### 15.8 Titkosított tárolók (LUKS, loop)
- `lsblk -f`: fájlrendszer információk.
- `dd if=/dev/zero of=/device0 bs=4096 count=1024`: `if/of` bemenet/kimenet, `bs` blokkméret.
- `fallocate -l 4MiB /device1`: `-l` méret.
- `losetup -f /device0`: szabad loop eszköz keresése.
- `losetup -a`: loop lista.
- `cryptsetup luksFormat /dev/sdb1`: LUKS formázás.
- `cryptsetup luksOpen /dev/sdb1 enc-home`: LUKS megnyitás (mapper név).
- `mkfs.ext2 /dev/mapper/enc-home`: fájlrendszer létrehozás.
- `mount /dev/mapper/enc-home /home`: csatolás.
- `umount /home`: leválasztás.
- `cat /etc/crypttab`: automatikus LUKS bejegyzések.
- `cat /etc/fstab | grep -v '#'`: fstab bejegyzések (komment nélkül).
- `grep -ac "Secret" /device1`: `-a` bináris; `-c` darabszám.

### 15.9 Kulcsfájl és automount
- `dd if=/dev/random of=/keys/home.key bs=16 count=1`: 16 bájtos kulcs.
- `chmod 000 /keys/home.key`: semmilyen jogosultság.
- `cryptsetup luksFormat --key-file /keys/home.key /dev/sdb1 secret`: `--key-file` kulcsfájl.
- `cryptsetup luksOpen /dev/sdb1 secret`: megnyitás kulccsal.
- `nano /etc/fstab` és `nano /etc/crypttab`: automatikus felcsatolás beállítása.

### 15.10 OpenSSL – szimmetrikus titkosítás
- `openssl aes-128-ecb -in file -out file.aes -K ...`: `-K` hex kulcs.
- `openssl aes-128-cbc -k PASSWORD -in file -out file.aes -p`: `-k` jelszó; `-p` kulcs/IV kiírás.
- `openssl aes-128-cbc -d ...`: `-d` visszafejtés.
- `openssl des-cbc -iv 0 -nosalt`: `-iv` inicializáló vektor; `-nosalt` sózás nélkül.
- `xxd file`: hex dump.

### 15.11 OpenSSL – RSA, aláírás, PKCS
- `openssl genpkey -algorithm RSA -out key.pem -pkeyopt rsa_keygen_bits:2048`: RSA kulcsgenerálás.
- `openssl rsa -in key.pem -pubout -out key.pem.pub`: publikus kulcs.
- `openssl dgst -sign key.pem -sha256 -out document.sign document`: `-sign` aláírás.
- `openssl dgst -verify key.pem.pub -sha256 -signature document.sign document`: aláírás ellenőrzés.
- `openssl rsautl -encrypt -pubin -inkey public.pem -in document -out document-enc1`: `-pubin` publikus kulcs.
- `openssl rsautl -decrypt -inkey private.pem -in document-enc1`: privát kulcsos visszafejtés.
- `openssl pkcs8 -topk8 ... -nocrypt`: PKCS#8 konverzió titkosítás nélkül.
- `openssl pkeyutl -encrypt -inkey pubkey -pubin -in file -out file.enc`: PKEYUTIL titkosítás.

### 15.12 OpenSSL – tanúsítványok
- `openssl genrsa -out ca.key 2048`: CA privát kulcs.
- `openssl req -new -key ca.key -out ca.csr`: CSR készítés.
- `openssl x509 -req -in ca.csr -out ca.crt -days 1000 -signkey ca.key`: tanúsítvány.
- `openssl x509 -text -noout -in ca.crt`: tanúsítvány kiírás.
- `openssl x509 -req -in web.csr -out web.crt -days 100 -CA ca.crt -CAkey ca.key -CAserial ca.srl -CAcreateserial`: CA-aláírás.
- `trust list | grep HARICA`: trust store keresés.
- `sudo trust anchor ca.crt`: CA import.

### 15.13 Kulcstranszformáció
- `ssh-keygen -f ssh-generated-key -e`: publikus kulcs export.
- `ssh-keygen -f ssh-generated-key -e -m pem`: `-m` formátum.
- `ssh-keygen -f ssh-generated-key.pub -e -m pkcs8`: PKCS#8 export.
- `ssh-keygen -i -f rsa-public-key -m pem`: importálás.
- `openssl rsa -in ssh-generated-key -pubout`: OpenSSL pubkey.

### 15.14 Windows eszközök
- `gpedit.msc`: helyi házirend szerkesztő.
- `eventvwr.msc`: eseménynapló.

## 16) Kiegészítő feladatok (új követelmények)

### 16.1 chmod numerikus értékek (0–7) jelentése
- `0 = ---` (nincs jog)
- `1 = --x` (csak végrehajtás)
- `2 = -w-` (csak írás)
- `3 = -wx` (írás + végrehajtás)
- `4 = r--` (csak olvasás)
- `5 = r-x` (olvasás + végrehajtás)
- `6 = rw-` (olvasás + írás)
- `7 = rwx` (teljes jog)
- Három számjegy: `user/group/other`. Négy számjegy: az első `4=setuid`, `2=setgid`, `1=sticky`.

### 16.2 /dev/sdc partíció és fájlrendszer létrehozás, csatolás (/dev/sdc1)
```bash
lsblk -f                                 # lemezek és fájlrendszerek
sudo wipefs -a /dev/sdc                  # régi aláírások törlése
sudo parted -s /dev/sdc mklabel gpt      # GPT partíciós tábla
sudo parted -s /dev/sdc mkpart primary ext4 1MiB 100%  # partíció létrehozása
sudo partprobe /dev/sdc                  # kernel újraolvasás
lsblk /dev/sdc                           # /dev/sdc1 ellenőrzése
sudo mkfs.ext4 -L data /dev/sdc1         # ext4 fájlrendszer címkével
sudo mkfs.xfs -L dataxfs /dev/sdc1       # alternatíva: XFS
sudo mkdir -p /mnt/sdc1                  # csatolási pont
sudo mount /dev/sdc1 /mnt/sdc1           # csatolás
mount | grep /mnt/sdc1                   # ellenőrzés
sudo blkid /dev/sdc1                     # UUID lekérdezés
sudo sh -c 'echo "UUID=... /mnt/sdc1 ext4 defaults,noatime 0 2" >> /etc/fstab'  # automount
```

### 16.3 Fájl titkosítása publikus kulccsal (kulcs a vágólapról)
```bash
cat > public.pem                         # kulcs beillesztése, majd Ctrl-D
chmod 600 public.pem                     # kulcsfájl védelme
openssl pkey -pubin -in public.pem -text -noout  # kulcs ellenőrzése
openssl pkeyutl -encrypt -pubin -inkey public.pem -in secret.txt -out secret.txt.enc  # RSA titkosítás
# nagy fájlhoz hibrid módszer
openssl rand -out sym.key 32             # szimmetrikus kulcs generálás
openssl enc -aes-256-gcm -salt -pbkdf2 -iter 200000 -in secret.txt -out secret.txt.enc -pass file:./sym.key
openssl pkeyutl -encrypt -pubin -inkey public.pem -in sym.key -out sym.key.enc
```

### 16.4 Jelszó lejárati dátum beállítása (dátummal)
```bash
chage -E 2026-12-31 user1                 # fiók lejárat dátum
chage -W 7 -I 14 user1                    # figyelmeztetés + inaktív napok
passwd --expire user1                     # következő belépéskor kötelező csere
chage -l user1                            # ellenőrzés
```

## 17) Bővített opciók és minták (3k+ sor)
Az alábbi blokkok célja, hogy sokféle opcióra és paraméterre példát adjanak.

### 17.1 chmod 000–777 referencia (minden kombináció)
```bash
chmod 000 file.txt  # u=---, g=---, o=---
chmod 001 file.txt  # u=---, g=---, o=--x
chmod 002 file.txt  # u=---, g=---, o=-w-
chmod 003 file.txt  # u=---, g=---, o=-wx
chmod 004 file.txt  # u=---, g=---, o=r--
chmod 005 file.txt  # u=---, g=---, o=r-x
chmod 006 file.txt  # u=---, g=---, o=rw-
chmod 007 file.txt  # u=---, g=---, o=rwx
chmod 010 file.txt  # u=---, g=--x, o=---
chmod 011 file.txt  # u=---, g=--x, o=--x
chmod 012 file.txt  # u=---, g=--x, o=-w-
chmod 013 file.txt  # u=---, g=--x, o=-wx
chmod 014 file.txt  # u=---, g=--x, o=r--
chmod 015 file.txt  # u=---, g=--x, o=r-x
chmod 016 file.txt  # u=---, g=--x, o=rw-
chmod 017 file.txt  # u=---, g=--x, o=rwx
chmod 020 file.txt  # u=---, g=-w-, o=---
chmod 021 file.txt  # u=---, g=-w-, o=--x
chmod 022 file.txt  # u=---, g=-w-, o=-w-
chmod 023 file.txt  # u=---, g=-w-, o=-wx
chmod 024 file.txt  # u=---, g=-w-, o=r--
chmod 025 file.txt  # u=---, g=-w-, o=r-x
chmod 026 file.txt  # u=---, g=-w-, o=rw-
chmod 027 file.txt  # u=---, g=-w-, o=rwx
chmod 030 file.txt  # u=---, g=-wx, o=---
chmod 031 file.txt  # u=---, g=-wx, o=--x
chmod 032 file.txt  # u=---, g=-wx, o=-w-
chmod 033 file.txt  # u=---, g=-wx, o=-wx
chmod 034 file.txt  # u=---, g=-wx, o=r--
chmod 035 file.txt  # u=---, g=-wx, o=r-x
chmod 036 file.txt  # u=---, g=-wx, o=rw-
chmod 037 file.txt  # u=---, g=-wx, o=rwx
chmod 040 file.txt  # u=---, g=r--, o=---
chmod 041 file.txt  # u=---, g=r--, o=--x
chmod 042 file.txt  # u=---, g=r--, o=-w-
chmod 043 file.txt  # u=---, g=r--, o=-wx
chmod 044 file.txt  # u=---, g=r--, o=r--
chmod 045 file.txt  # u=---, g=r--, o=r-x
chmod 046 file.txt  # u=---, g=r--, o=rw-
chmod 047 file.txt  # u=---, g=r--, o=rwx
chmod 050 file.txt  # u=---, g=r-x, o=---
chmod 051 file.txt  # u=---, g=r-x, o=--x
chmod 052 file.txt  # u=---, g=r-x, o=-w-
chmod 053 file.txt  # u=---, g=r-x, o=-wx
chmod 054 file.txt  # u=---, g=r-x, o=r--
chmod 055 file.txt  # u=---, g=r-x, o=r-x
chmod 056 file.txt  # u=---, g=r-x, o=rw-
chmod 057 file.txt  # u=---, g=r-x, o=rwx
chmod 060 file.txt  # u=---, g=rw-, o=---
chmod 061 file.txt  # u=---, g=rw-, o=--x
chmod 062 file.txt  # u=---, g=rw-, o=-w-
chmod 063 file.txt  # u=---, g=rw-, o=-wx
chmod 064 file.txt  # u=---, g=rw-, o=r--
chmod 065 file.txt  # u=---, g=rw-, o=r-x
chmod 066 file.txt  # u=---, g=rw-, o=rw-
chmod 067 file.txt  # u=---, g=rw-, o=rwx
chmod 070 file.txt  # u=---, g=rwx, o=---
chmod 071 file.txt  # u=---, g=rwx, o=--x
chmod 072 file.txt  # u=---, g=rwx, o=-w-
chmod 073 file.txt  # u=---, g=rwx, o=-wx
chmod 074 file.txt  # u=---, g=rwx, o=r--
chmod 075 file.txt  # u=---, g=rwx, o=r-x
chmod 076 file.txt  # u=---, g=rwx, o=rw-
chmod 077 file.txt  # u=---, g=rwx, o=rwx
chmod 100 file.txt  # u=--x, g=---, o=---
chmod 101 file.txt  # u=--x, g=---, o=--x
chmod 102 file.txt  # u=--x, g=---, o=-w-
chmod 103 file.txt  # u=--x, g=---, o=-wx
chmod 104 file.txt  # u=--x, g=---, o=r--
chmod 105 file.txt  # u=--x, g=---, o=r-x
chmod 106 file.txt  # u=--x, g=---, o=rw-
chmod 107 file.txt  # u=--x, g=---, o=rwx
chmod 110 file.txt  # u=--x, g=--x, o=---
chmod 111 file.txt  # u=--x, g=--x, o=--x
chmod 112 file.txt  # u=--x, g=--x, o=-w-
chmod 113 file.txt  # u=--x, g=--x, o=-wx
chmod 114 file.txt  # u=--x, g=--x, o=r--
chmod 115 file.txt  # u=--x, g=--x, o=r-x
chmod 116 file.txt  # u=--x, g=--x, o=rw-
chmod 117 file.txt  # u=--x, g=--x, o=rwx
chmod 120 file.txt  # u=--x, g=-w-, o=---
chmod 121 file.txt  # u=--x, g=-w-, o=--x
chmod 122 file.txt  # u=--x, g=-w-, o=-w-
chmod 123 file.txt  # u=--x, g=-w-, o=-wx
chmod 124 file.txt  # u=--x, g=-w-, o=r--
chmod 125 file.txt  # u=--x, g=-w-, o=r-x
chmod 126 file.txt  # u=--x, g=-w-, o=rw-
chmod 127 file.txt  # u=--x, g=-w-, o=rwx
chmod 130 file.txt  # u=--x, g=-wx, o=---
chmod 131 file.txt  # u=--x, g=-wx, o=--x
chmod 132 file.txt  # u=--x, g=-wx, o=-w-
chmod 133 file.txt  # u=--x, g=-wx, o=-wx
chmod 134 file.txt  # u=--x, g=-wx, o=r--
chmod 135 file.txt  # u=--x, g=-wx, o=r-x
chmod 136 file.txt  # u=--x, g=-wx, o=rw-
chmod 137 file.txt  # u=--x, g=-wx, o=rwx
chmod 140 file.txt  # u=--x, g=r--, o=---
chmod 141 file.txt  # u=--x, g=r--, o=--x
chmod 142 file.txt  # u=--x, g=r--, o=-w-
chmod 143 file.txt  # u=--x, g=r--, o=-wx
chmod 144 file.txt  # u=--x, g=r--, o=r--
chmod 145 file.txt  # u=--x, g=r--, o=r-x
chmod 146 file.txt  # u=--x, g=r--, o=rw-
chmod 147 file.txt  # u=--x, g=r--, o=rwx
chmod 150 file.txt  # u=--x, g=r-x, o=---
chmod 151 file.txt  # u=--x, g=r-x, o=--x
chmod 152 file.txt  # u=--x, g=r-x, o=-w-
chmod 153 file.txt  # u=--x, g=r-x, o=-wx
chmod 154 file.txt  # u=--x, g=r-x, o=r--
chmod 155 file.txt  # u=--x, g=r-x, o=r-x
chmod 156 file.txt  # u=--x, g=r-x, o=rw-
chmod 157 file.txt  # u=--x, g=r-x, o=rwx
chmod 160 file.txt  # u=--x, g=rw-, o=---
chmod 161 file.txt  # u=--x, g=rw-, o=--x
chmod 162 file.txt  # u=--x, g=rw-, o=-w-
chmod 163 file.txt  # u=--x, g=rw-, o=-wx
chmod 164 file.txt  # u=--x, g=rw-, o=r--
chmod 165 file.txt  # u=--x, g=rw-, o=r-x
chmod 166 file.txt  # u=--x, g=rw-, o=rw-
chmod 167 file.txt  # u=--x, g=rw-, o=rwx
chmod 170 file.txt  # u=--x, g=rwx, o=---
chmod 171 file.txt  # u=--x, g=rwx, o=--x
chmod 172 file.txt  # u=--x, g=rwx, o=-w-
chmod 173 file.txt  # u=--x, g=rwx, o=-wx
chmod 174 file.txt  # u=--x, g=rwx, o=r--
chmod 175 file.txt  # u=--x, g=rwx, o=r-x
chmod 176 file.txt  # u=--x, g=rwx, o=rw-
chmod 177 file.txt  # u=--x, g=rwx, o=rwx
chmod 200 file.txt  # u=-w-, g=---, o=---
chmod 201 file.txt  # u=-w-, g=---, o=--x
chmod 202 file.txt  # u=-w-, g=---, o=-w-
chmod 203 file.txt  # u=-w-, g=---, o=-wx
chmod 204 file.txt  # u=-w-, g=---, o=r--
chmod 205 file.txt  # u=-w-, g=---, o=r-x
chmod 206 file.txt  # u=-w-, g=---, o=rw-
chmod 207 file.txt  # u=-w-, g=---, o=rwx
chmod 210 file.txt  # u=-w-, g=--x, o=---
chmod 211 file.txt  # u=-w-, g=--x, o=--x
chmod 212 file.txt  # u=-w-, g=--x, o=-w-
chmod 213 file.txt  # u=-w-, g=--x, o=-wx
chmod 214 file.txt  # u=-w-, g=--x, o=r--
chmod 215 file.txt  # u=-w-, g=--x, o=r-x
chmod 216 file.txt  # u=-w-, g=--x, o=rw-
chmod 217 file.txt  # u=-w-, g=--x, o=rwx
chmod 220 file.txt  # u=-w-, g=-w-, o=---
chmod 221 file.txt  # u=-w-, g=-w-, o=--x
chmod 222 file.txt  # u=-w-, g=-w-, o=-w-
chmod 223 file.txt  # u=-w-, g=-w-, o=-wx
chmod 224 file.txt  # u=-w-, g=-w-, o=r--
chmod 225 file.txt  # u=-w-, g=-w-, o=r-x
chmod 226 file.txt  # u=-w-, g=-w-, o=rw-
chmod 227 file.txt  # u=-w-, g=-w-, o=rwx
chmod 230 file.txt  # u=-w-, g=-wx, o=---
chmod 231 file.txt  # u=-w-, g=-wx, o=--x
chmod 232 file.txt  # u=-w-, g=-wx, o=-w-
chmod 233 file.txt  # u=-w-, g=-wx, o=-wx
chmod 234 file.txt  # u=-w-, g=-wx, o=r--
chmod 235 file.txt  # u=-w-, g=-wx, o=r-x
chmod 236 file.txt  # u=-w-, g=-wx, o=rw-
chmod 237 file.txt  # u=-w-, g=-wx, o=rwx
chmod 240 file.txt  # u=-w-, g=r--, o=---
chmod 241 file.txt  # u=-w-, g=r--, o=--x
chmod 242 file.txt  # u=-w-, g=r--, o=-w-
chmod 243 file.txt  # u=-w-, g=r--, o=-wx
chmod 244 file.txt  # u=-w-, g=r--, o=r--
chmod 245 file.txt  # u=-w-, g=r--, o=r-x
chmod 246 file.txt  # u=-w-, g=r--, o=rw-
chmod 247 file.txt  # u=-w-, g=r--, o=rwx
chmod 250 file.txt  # u=-w-, g=r-x, o=---
chmod 251 file.txt  # u=-w-, g=r-x, o=--x
chmod 252 file.txt  # u=-w-, g=r-x, o=-w-
chmod 253 file.txt  # u=-w-, g=r-x, o=-wx
chmod 254 file.txt  # u=-w-, g=r-x, o=r--
chmod 255 file.txt  # u=-w-, g=r-x, o=r-x
chmod 256 file.txt  # u=-w-, g=r-x, o=rw-
chmod 257 file.txt  # u=-w-, g=r-x, o=rwx
chmod 260 file.txt  # u=-w-, g=rw-, o=---
chmod 261 file.txt  # u=-w-, g=rw-, o=--x
chmod 262 file.txt  # u=-w-, g=rw-, o=-w-
chmod 263 file.txt  # u=-w-, g=rw-, o=-wx
chmod 264 file.txt  # u=-w-, g=rw-, o=r--
chmod 265 file.txt  # u=-w-, g=rw-, o=r-x
chmod 266 file.txt  # u=-w-, g=rw-, o=rw-
chmod 267 file.txt  # u=-w-, g=rw-, o=rwx
chmod 270 file.txt  # u=-w-, g=rwx, o=---
chmod 271 file.txt  # u=-w-, g=rwx, o=--x
chmod 272 file.txt  # u=-w-, g=rwx, o=-w-
chmod 273 file.txt  # u=-w-, g=rwx, o=-wx
chmod 274 file.txt  # u=-w-, g=rwx, o=r--
chmod 275 file.txt  # u=-w-, g=rwx, o=r-x
chmod 276 file.txt  # u=-w-, g=rwx, o=rw-
chmod 277 file.txt  # u=-w-, g=rwx, o=rwx
chmod 300 file.txt  # u=-wx, g=---, o=---
chmod 301 file.txt  # u=-wx, g=---, o=--x
chmod 302 file.txt  # u=-wx, g=---, o=-w-
chmod 303 file.txt  # u=-wx, g=---, o=-wx
chmod 304 file.txt  # u=-wx, g=---, o=r--
chmod 305 file.txt  # u=-wx, g=---, o=r-x
chmod 306 file.txt  # u=-wx, g=---, o=rw-
chmod 307 file.txt  # u=-wx, g=---, o=rwx
chmod 310 file.txt  # u=-wx, g=--x, o=---
chmod 311 file.txt  # u=-wx, g=--x, o=--x
chmod 312 file.txt  # u=-wx, g=--x, o=-w-
chmod 313 file.txt  # u=-wx, g=--x, o=-wx
chmod 314 file.txt  # u=-wx, g=--x, o=r--
chmod 315 file.txt  # u=-wx, g=--x, o=r-x
chmod 316 file.txt  # u=-wx, g=--x, o=rw-
chmod 317 file.txt  # u=-wx, g=--x, o=rwx
chmod 320 file.txt  # u=-wx, g=-w-, o=---
chmod 321 file.txt  # u=-wx, g=-w-, o=--x
chmod 322 file.txt  # u=-wx, g=-w-, o=-w-
chmod 323 file.txt  # u=-wx, g=-w-, o=-wx
chmod 324 file.txt  # u=-wx, g=-w-, o=r--
chmod 325 file.txt  # u=-wx, g=-w-, o=r-x
chmod 326 file.txt  # u=-wx, g=-w-, o=rw-
chmod 327 file.txt  # u=-wx, g=-w-, o=rwx
chmod 330 file.txt  # u=-wx, g=-wx, o=---
chmod 331 file.txt  # u=-wx, g=-wx, o=--x
chmod 332 file.txt  # u=-wx, g=-wx, o=-w-
chmod 333 file.txt  # u=-wx, g=-wx, o=-wx
chmod 334 file.txt  # u=-wx, g=-wx, o=r--
chmod 335 file.txt  # u=-wx, g=-wx, o=r-x
chmod 336 file.txt  # u=-wx, g=-wx, o=rw-
chmod 337 file.txt  # u=-wx, g=-wx, o=rwx
chmod 340 file.txt  # u=-wx, g=r--, o=---
chmod 341 file.txt  # u=-wx, g=r--, o=--x
chmod 342 file.txt  # u=-wx, g=r--, o=-w-
chmod 343 file.txt  # u=-wx, g=r--, o=-wx
chmod 344 file.txt  # u=-wx, g=r--, o=r--
chmod 345 file.txt  # u=-wx, g=r--, o=r-x
chmod 346 file.txt  # u=-wx, g=r--, o=rw-
chmod 347 file.txt  # u=-wx, g=r--, o=rwx
chmod 350 file.txt  # u=-wx, g=r-x, o=---
chmod 351 file.txt  # u=-wx, g=r-x, o=--x
chmod 352 file.txt  # u=-wx, g=r-x, o=-w-
chmod 353 file.txt  # u=-wx, g=r-x, o=-wx
chmod 354 file.txt  # u=-wx, g=r-x, o=r--
chmod 355 file.txt  # u=-wx, g=r-x, o=r-x
chmod 356 file.txt  # u=-wx, g=r-x, o=rw-
chmod 357 file.txt  # u=-wx, g=r-x, o=rwx
chmod 360 file.txt  # u=-wx, g=rw-, o=---
chmod 361 file.txt  # u=-wx, g=rw-, o=--x
chmod 362 file.txt  # u=-wx, g=rw-, o=-w-
chmod 363 file.txt  # u=-wx, g=rw-, o=-wx
chmod 364 file.txt  # u=-wx, g=rw-, o=r--
chmod 365 file.txt  # u=-wx, g=rw-, o=r-x
chmod 366 file.txt  # u=-wx, g=rw-, o=rw-
chmod 367 file.txt  # u=-wx, g=rw-, o=rwx
chmod 370 file.txt  # u=-wx, g=rwx, o=---
chmod 371 file.txt  # u=-wx, g=rwx, o=--x
chmod 372 file.txt  # u=-wx, g=rwx, o=-w-
chmod 373 file.txt  # u=-wx, g=rwx, o=-wx
chmod 374 file.txt  # u=-wx, g=rwx, o=r--
chmod 375 file.txt  # u=-wx, g=rwx, o=r-x
chmod 376 file.txt  # u=-wx, g=rwx, o=rw-
chmod 377 file.txt  # u=-wx, g=rwx, o=rwx
chmod 400 file.txt  # u=r--, g=---, o=---
chmod 401 file.txt  # u=r--, g=---, o=--x
chmod 402 file.txt  # u=r--, g=---, o=-w-
chmod 403 file.txt  # u=r--, g=---, o=-wx
chmod 404 file.txt  # u=r--, g=---, o=r--
chmod 405 file.txt  # u=r--, g=---, o=r-x
chmod 406 file.txt  # u=r--, g=---, o=rw-
chmod 407 file.txt  # u=r--, g=---, o=rwx
chmod 410 file.txt  # u=r--, g=--x, o=---
chmod 411 file.txt  # u=r--, g=--x, o=--x
chmod 412 file.txt  # u=r--, g=--x, o=-w-
chmod 413 file.txt  # u=r--, g=--x, o=-wx
chmod 414 file.txt  # u=r--, g=--x, o=r--
chmod 415 file.txt  # u=r--, g=--x, o=r-x
chmod 416 file.txt  # u=r--, g=--x, o=rw-
chmod 417 file.txt  # u=r--, g=--x, o=rwx
chmod 420 file.txt  # u=r--, g=-w-, o=---
chmod 421 file.txt  # u=r--, g=-w-, o=--x
chmod 422 file.txt  # u=r--, g=-w-, o=-w-
chmod 423 file.txt  # u=r--, g=-w-, o=-wx
chmod 424 file.txt  # u=r--, g=-w-, o=r--
chmod 425 file.txt  # u=r--, g=-w-, o=r-x
chmod 426 file.txt  # u=r--, g=-w-, o=rw-
chmod 427 file.txt  # u=r--, g=-w-, o=rwx
chmod 430 file.txt  # u=r--, g=-wx, o=---
chmod 431 file.txt  # u=r--, g=-wx, o=--x
chmod 432 file.txt  # u=r--, g=-wx, o=-w-
chmod 433 file.txt  # u=r--, g=-wx, o=-wx
chmod 434 file.txt  # u=r--, g=-wx, o=r--
chmod 435 file.txt  # u=r--, g=-wx, o=r-x
chmod 436 file.txt  # u=r--, g=-wx, o=rw-
chmod 437 file.txt  # u=r--, g=-wx, o=rwx
chmod 440 file.txt  # u=r--, g=r--, o=---
chmod 441 file.txt  # u=r--, g=r--, o=--x
chmod 442 file.txt  # u=r--, g=r--, o=-w-
chmod 443 file.txt  # u=r--, g=r--, o=-wx
chmod 444 file.txt  # u=r--, g=r--, o=r--
chmod 445 file.txt  # u=r--, g=r--, o=r-x
chmod 446 file.txt  # u=r--, g=r--, o=rw-
chmod 447 file.txt  # u=r--, g=r--, o=rwx
chmod 450 file.txt  # u=r--, g=r-x, o=---
chmod 451 file.txt  # u=r--, g=r-x, o=--x
chmod 452 file.txt  # u=r--, g=r-x, o=-w-
chmod 453 file.txt  # u=r--, g=r-x, o=-wx
chmod 454 file.txt  # u=r--, g=r-x, o=r--
chmod 455 file.txt  # u=r--, g=r-x, o=r-x
chmod 456 file.txt  # u=r--, g=r-x, o=rw-
chmod 457 file.txt  # u=r--, g=r-x, o=rwx
chmod 460 file.txt  # u=r--, g=rw-, o=---
chmod 461 file.txt  # u=r--, g=rw-, o=--x
chmod 462 file.txt  # u=r--, g=rw-, o=-w-
chmod 463 file.txt  # u=r--, g=rw-, o=-wx
chmod 464 file.txt  # u=r--, g=rw-, o=r--
chmod 465 file.txt  # u=r--, g=rw-, o=r-x
chmod 466 file.txt  # u=r--, g=rw-, o=rw-
chmod 467 file.txt  # u=r--, g=rw-, o=rwx
chmod 470 file.txt  # u=r--, g=rwx, o=---
chmod 471 file.txt  # u=r--, g=rwx, o=--x
chmod 472 file.txt  # u=r--, g=rwx, o=-w-
chmod 473 file.txt  # u=r--, g=rwx, o=-wx
chmod 474 file.txt  # u=r--, g=rwx, o=r--
chmod 475 file.txt  # u=r--, g=rwx, o=r-x
chmod 476 file.txt  # u=r--, g=rwx, o=rw-
chmod 477 file.txt  # u=r--, g=rwx, o=rwx
chmod 500 file.txt  # u=r-x, g=---, o=---
chmod 501 file.txt  # u=r-x, g=---, o=--x
chmod 502 file.txt  # u=r-x, g=---, o=-w-
chmod 503 file.txt  # u=r-x, g=---, o=-wx
chmod 504 file.txt  # u=r-x, g=---, o=r--
chmod 505 file.txt  # u=r-x, g=---, o=r-x
chmod 506 file.txt  # u=r-x, g=---, o=rw-
chmod 507 file.txt  # u=r-x, g=---, o=rwx
chmod 510 file.txt  # u=r-x, g=--x, o=---
chmod 511 file.txt  # u=r-x, g=--x, o=--x
chmod 512 file.txt  # u=r-x, g=--x, o=-w-
chmod 513 file.txt  # u=r-x, g=--x, o=-wx
chmod 514 file.txt  # u=r-x, g=--x, o=r--
chmod 515 file.txt  # u=r-x, g=--x, o=r-x
chmod 516 file.txt  # u=r-x, g=--x, o=rw-
chmod 517 file.txt  # u=r-x, g=--x, o=rwx
chmod 520 file.txt  # u=r-x, g=-w-, o=---
chmod 521 file.txt  # u=r-x, g=-w-, o=--x
chmod 522 file.txt  # u=r-x, g=-w-, o=-w-
chmod 523 file.txt  # u=r-x, g=-w-, o=-wx
chmod 524 file.txt  # u=r-x, g=-w-, o=r--
chmod 525 file.txt  # u=r-x, g=-w-, o=r-x
chmod 526 file.txt  # u=r-x, g=-w-, o=rw-
chmod 527 file.txt  # u=r-x, g=-w-, o=rwx
chmod 530 file.txt  # u=r-x, g=-wx, o=---
chmod 531 file.txt  # u=r-x, g=-wx, o=--x
chmod 532 file.txt  # u=r-x, g=-wx, o=-w-
chmod 533 file.txt  # u=r-x, g=-wx, o=-wx
chmod 534 file.txt  # u=r-x, g=-wx, o=r--
chmod 535 file.txt  # u=r-x, g=-wx, o=r-x
chmod 536 file.txt  # u=r-x, g=-wx, o=rw-
chmod 537 file.txt  # u=r-x, g=-wx, o=rwx
chmod 540 file.txt  # u=r-x, g=r--, o=---
chmod 541 file.txt  # u=r-x, g=r--, o=--x
chmod 542 file.txt  # u=r-x, g=r--, o=-w-
chmod 543 file.txt  # u=r-x, g=r--, o=-wx
chmod 544 file.txt  # u=r-x, g=r--, o=r--
chmod 545 file.txt  # u=r-x, g=r--, o=r-x
chmod 546 file.txt  # u=r-x, g=r--, o=rw-
chmod 547 file.txt  # u=r-x, g=r--, o=rwx
chmod 550 file.txt  # u=r-x, g=r-x, o=---
chmod 551 file.txt  # u=r-x, g=r-x, o=--x
chmod 552 file.txt  # u=r-x, g=r-x, o=-w-
chmod 553 file.txt  # u=r-x, g=r-x, o=-wx
chmod 554 file.txt  # u=r-x, g=r-x, o=r--
chmod 555 file.txt  # u=r-x, g=r-x, o=r-x
chmod 556 file.txt  # u=r-x, g=r-x, o=rw-
chmod 557 file.txt  # u=r-x, g=r-x, o=rwx
chmod 560 file.txt  # u=r-x, g=rw-, o=---
chmod 561 file.txt  # u=r-x, g=rw-, o=--x
chmod 562 file.txt  # u=r-x, g=rw-, o=-w-
chmod 563 file.txt  # u=r-x, g=rw-, o=-wx
chmod 564 file.txt  # u=r-x, g=rw-, o=r--
chmod 565 file.txt  # u=r-x, g=rw-, o=r-x
chmod 566 file.txt  # u=r-x, g=rw-, o=rw-
chmod 567 file.txt  # u=r-x, g=rw-, o=rwx
chmod 570 file.txt  # u=r-x, g=rwx, o=---
chmod 571 file.txt  # u=r-x, g=rwx, o=--x
chmod 572 file.txt  # u=r-x, g=rwx, o=-w-
chmod 573 file.txt  # u=r-x, g=rwx, o=-wx
chmod 574 file.txt  # u=r-x, g=rwx, o=r--
chmod 575 file.txt  # u=r-x, g=rwx, o=r-x
chmod 576 file.txt  # u=r-x, g=rwx, o=rw-
chmod 577 file.txt  # u=r-x, g=rwx, o=rwx
chmod 600 file.txt  # u=rw-, g=---, o=---
chmod 601 file.txt  # u=rw-, g=---, o=--x
chmod 602 file.txt  # u=rw-, g=---, o=-w-
chmod 603 file.txt  # u=rw-, g=---, o=-wx
chmod 604 file.txt  # u=rw-, g=---, o=r--
chmod 605 file.txt  # u=rw-, g=---, o=r-x
chmod 606 file.txt  # u=rw-, g=---, o=rw-
chmod 607 file.txt  # u=rw-, g=---, o=rwx
chmod 610 file.txt  # u=rw-, g=--x, o=---
chmod 611 file.txt  # u=rw-, g=--x, o=--x
chmod 612 file.txt  # u=rw-, g=--x, o=-w-
chmod 613 file.txt  # u=rw-, g=--x, o=-wx
chmod 614 file.txt  # u=rw-, g=--x, o=r--
chmod 615 file.txt  # u=rw-, g=--x, o=r-x
chmod 616 file.txt  # u=rw-, g=--x, o=rw-
chmod 617 file.txt  # u=rw-, g=--x, o=rwx
chmod 620 file.txt  # u=rw-, g=-w-, o=---
chmod 621 file.txt  # u=rw-, g=-w-, o=--x
chmod 622 file.txt  # u=rw-, g=-w-, o=-w-
chmod 623 file.txt  # u=rw-, g=-w-, o=-wx
chmod 624 file.txt  # u=rw-, g=-w-, o=r--
chmod 625 file.txt  # u=rw-, g=-w-, o=r-x
chmod 626 file.txt  # u=rw-, g=-w-, o=rw-
chmod 627 file.txt  # u=rw-, g=-w-, o=rwx
chmod 630 file.txt  # u=rw-, g=-wx, o=---
chmod 631 file.txt  # u=rw-, g=-wx, o=--x
chmod 632 file.txt  # u=rw-, g=-wx, o=-w-
chmod 633 file.txt  # u=rw-, g=-wx, o=-wx
chmod 634 file.txt  # u=rw-, g=-wx, o=r--
chmod 635 file.txt  # u=rw-, g=-wx, o=r-x
chmod 636 file.txt  # u=rw-, g=-wx, o=rw-
chmod 637 file.txt  # u=rw-, g=-wx, o=rwx
chmod 640 file.txt  # u=rw-, g=r--, o=---
chmod 641 file.txt  # u=rw-, g=r--, o=--x
chmod 642 file.txt  # u=rw-, g=r--, o=-w-
chmod 643 file.txt  # u=rw-, g=r--, o=-wx
chmod 644 file.txt  # u=rw-, g=r--, o=r--
chmod 645 file.txt  # u=rw-, g=r--, o=r-x
chmod 646 file.txt  # u=rw-, g=r--, o=rw-
chmod 647 file.txt  # u=rw-, g=r--, o=rwx
chmod 650 file.txt  # u=rw-, g=r-x, o=---
chmod 651 file.txt  # u=rw-, g=r-x, o=--x
chmod 652 file.txt  # u=rw-, g=r-x, o=-w-
chmod 653 file.txt  # u=rw-, g=r-x, o=-wx
chmod 654 file.txt  # u=rw-, g=r-x, o=r--
chmod 655 file.txt  # u=rw-, g=r-x, o=r-x
chmod 656 file.txt  # u=rw-, g=r-x, o=rw-
chmod 657 file.txt  # u=rw-, g=r-x, o=rwx
chmod 660 file.txt  # u=rw-, g=rw-, o=---
chmod 661 file.txt  # u=rw-, g=rw-, o=--x
chmod 662 file.txt  # u=rw-, g=rw-, o=-w-
chmod 663 file.txt  # u=rw-, g=rw-, o=-wx
chmod 664 file.txt  # u=rw-, g=rw-, o=r--
chmod 665 file.txt  # u=rw-, g=rw-, o=r-x
chmod 666 file.txt  # u=rw-, g=rw-, o=rw-
chmod 667 file.txt  # u=rw-, g=rw-, o=rwx
chmod 670 file.txt  # u=rw-, g=rwx, o=---
chmod 671 file.txt  # u=rw-, g=rwx, o=--x
chmod 672 file.txt  # u=rw-, g=rwx, o=-w-
chmod 673 file.txt  # u=rw-, g=rwx, o=-wx
chmod 674 file.txt  # u=rw-, g=rwx, o=r--
chmod 675 file.txt  # u=rw-, g=rwx, o=r-x
chmod 676 file.txt  # u=rw-, g=rwx, o=rw-
chmod 677 file.txt  # u=rw-, g=rwx, o=rwx
chmod 700 file.txt  # u=rwx, g=---, o=---
chmod 701 file.txt  # u=rwx, g=---, o=--x
chmod 702 file.txt  # u=rwx, g=---, o=-w-
chmod 703 file.txt  # u=rwx, g=---, o=-wx
chmod 704 file.txt  # u=rwx, g=---, o=r--
chmod 705 file.txt  # u=rwx, g=---, o=r-x
chmod 706 file.txt  # u=rwx, g=---, o=rw-
chmod 707 file.txt  # u=rwx, g=---, o=rwx
chmod 710 file.txt  # u=rwx, g=--x, o=---
chmod 711 file.txt  # u=rwx, g=--x, o=--x
chmod 712 file.txt  # u=rwx, g=--x, o=-w-
chmod 713 file.txt  # u=rwx, g=--x, o=-wx
chmod 714 file.txt  # u=rwx, g=--x, o=r--
chmod 715 file.txt  # u=rwx, g=--x, o=r-x
chmod 716 file.txt  # u=rwx, g=--x, o=rw-
chmod 717 file.txt  # u=rwx, g=--x, o=rwx
chmod 720 file.txt  # u=rwx, g=-w-, o=---
chmod 721 file.txt  # u=rwx, g=-w-, o=--x
chmod 722 file.txt  # u=rwx, g=-w-, o=-w-
chmod 723 file.txt  # u=rwx, g=-w-, o=-wx
chmod 724 file.txt  # u=rwx, g=-w-, o=r--
chmod 725 file.txt  # u=rwx, g=-w-, o=r-x
chmod 726 file.txt  # u=rwx, g=-w-, o=rw-
chmod 727 file.txt  # u=rwx, g=-w-, o=rwx
chmod 730 file.txt  # u=rwx, g=-wx, o=---
chmod 731 file.txt  # u=rwx, g=-wx, o=--x
chmod 732 file.txt  # u=rwx, g=-wx, o=-w-
chmod 733 file.txt  # u=rwx, g=-wx, o=-wx
chmod 734 file.txt  # u=rwx, g=-wx, o=r--
chmod 735 file.txt  # u=rwx, g=-wx, o=r-x
chmod 736 file.txt  # u=rwx, g=-wx, o=rw-
chmod 737 file.txt  # u=rwx, g=-wx, o=rwx
chmod 740 file.txt  # u=rwx, g=r--, o=---
chmod 741 file.txt  # u=rwx, g=r--, o=--x
chmod 742 file.txt  # u=rwx, g=r--, o=-w-
chmod 743 file.txt  # u=rwx, g=r--, o=-wx
chmod 744 file.txt  # u=rwx, g=r--, o=r--
chmod 745 file.txt  # u=rwx, g=r--, o=r-x
chmod 746 file.txt  # u=rwx, g=r--, o=rw-
chmod 747 file.txt  # u=rwx, g=r--, o=rwx
chmod 750 file.txt  # u=rwx, g=r-x, o=---
chmod 751 file.txt  # u=rwx, g=r-x, o=--x
chmod 752 file.txt  # u=rwx, g=r-x, o=-w-
chmod 753 file.txt  # u=rwx, g=r-x, o=-wx
chmod 754 file.txt  # u=rwx, g=r-x, o=r--
chmod 755 file.txt  # u=rwx, g=r-x, o=r-x
chmod 756 file.txt  # u=rwx, g=r-x, o=rw-
chmod 757 file.txt  # u=rwx, g=r-x, o=rwx
chmod 760 file.txt  # u=rwx, g=rw-, o=---
chmod 761 file.txt  # u=rwx, g=rw-, o=--x
chmod 762 file.txt  # u=rwx, g=rw-, o=-w-
chmod 763 file.txt  # u=rwx, g=rw-, o=-wx
chmod 764 file.txt  # u=rwx, g=rw-, o=r--
chmod 765 file.txt  # u=rwx, g=rw-, o=r-x
chmod 766 file.txt  # u=rwx, g=rw-, o=rw-
chmod 767 file.txt  # u=rwx, g=rw-, o=rwx
chmod 770 file.txt  # u=rwx, g=rwx, o=---
chmod 771 file.txt  # u=rwx, g=rwx, o=--x
chmod 772 file.txt  # u=rwx, g=rwx, o=-w-
chmod 773 file.txt  # u=rwx, g=rwx, o=-wx
chmod 774 file.txt  # u=rwx, g=rwx, o=r--
chmod 775 file.txt  # u=rwx, g=rwx, o=r-x
chmod 776 file.txt  # u=rwx, g=rwx, o=rw-
chmod 777 file.txt  # u=rwx, g=rwx, o=rwx
```

### 17.2 firewall-cmd port lista (TCP 1–1024)
```bash
firewall-cmd --permanent --zone=public --add-port=1/tcp  # --permanent: mentés, TCP port 1
firewall-cmd --permanent --zone=public --add-port=2/tcp  # --permanent: mentés, TCP port 2
firewall-cmd --permanent --zone=public --add-port=3/tcp  # --permanent: mentés, TCP port 3
firewall-cmd --permanent --zone=public --add-port=4/tcp  # --permanent: mentés, TCP port 4
firewall-cmd --permanent --zone=public --add-port=5/tcp  # --permanent: mentés, TCP port 5
firewall-cmd --permanent --zone=public --add-port=6/tcp  # --permanent: mentés, TCP port 6
firewall-cmd --permanent --zone=public --add-port=7/tcp  # --permanent: mentés, TCP port 7
firewall-cmd --permanent --zone=public --add-port=8/tcp  # --permanent: mentés, TCP port 8
firewall-cmd --permanent --zone=public --add-port=9/tcp  # --permanent: mentés, TCP port 9
firewall-cmd --permanent --zone=public --add-port=10/tcp  # --permanent: mentés, TCP port 10
firewall-cmd --permanent --zone=public --add-port=11/tcp  # --permanent: mentés, TCP port 11
firewall-cmd --permanent --zone=public --add-port=12/tcp  # --permanent: mentés, TCP port 12
firewall-cmd --permanent --zone=public --add-port=13/tcp  # --permanent: mentés, TCP port 13
firewall-cmd --permanent --zone=public --add-port=14/tcp  # --permanent: mentés, TCP port 14
firewall-cmd --permanent --zone=public --add-port=15/tcp  # --permanent: mentés, TCP port 15
firewall-cmd --permanent --zone=public --add-port=16/tcp  # --permanent: mentés, TCP port 16
firewall-cmd --permanent --zone=public --add-port=17/tcp  # --permanent: mentés, TCP port 17
firewall-cmd --permanent --zone=public --add-port=18/tcp  # --permanent: mentés, TCP port 18
firewall-cmd --permanent --zone=public --add-port=19/tcp  # --permanent: mentés, TCP port 19
firewall-cmd --permanent --zone=public --add-port=20/tcp  # --permanent: mentés, TCP port 20
firewall-cmd --permanent --zone=public --add-port=21/tcp  # --permanent: mentés, TCP port 21
firewall-cmd --permanent --zone=public --add-port=22/tcp  # --permanent: mentés, TCP port 22
firewall-cmd --permanent --zone=public --add-port=23/tcp  # --permanent: mentés, TCP port 23
firewall-cmd --permanent --zone=public --add-port=24/tcp  # --permanent: mentés, TCP port 24
firewall-cmd --permanent --zone=public --add-port=25/tcp  # --permanent: mentés, TCP port 25
firewall-cmd --permanent --zone=public --add-port=26/tcp  # --permanent: mentés, TCP port 26
firewall-cmd --permanent --zone=public --add-port=27/tcp  # --permanent: mentés, TCP port 27
firewall-cmd --permanent --zone=public --add-port=28/tcp  # --permanent: mentés, TCP port 28
firewall-cmd --permanent --zone=public --add-port=29/tcp  # --permanent: mentés, TCP port 29
firewall-cmd --permanent --zone=public --add-port=30/tcp  # --permanent: mentés, TCP port 30
firewall-cmd --permanent --zone=public --add-port=31/tcp  # --permanent: mentés, TCP port 31
firewall-cmd --permanent --zone=public --add-port=32/tcp  # --permanent: mentés, TCP port 32
firewall-cmd --permanent --zone=public --add-port=33/tcp  # --permanent: mentés, TCP port 33
firewall-cmd --permanent --zone=public --add-port=34/tcp  # --permanent: mentés, TCP port 34
firewall-cmd --permanent --zone=public --add-port=35/tcp  # --permanent: mentés, TCP port 35
firewall-cmd --permanent --zone=public --add-port=36/tcp  # --permanent: mentés, TCP port 36
firewall-cmd --permanent --zone=public --add-port=37/tcp  # --permanent: mentés, TCP port 37
firewall-cmd --permanent --zone=public --add-port=38/tcp  # --permanent: mentés, TCP port 38
firewall-cmd --permanent --zone=public --add-port=39/tcp  # --permanent: mentés, TCP port 39
firewall-cmd --permanent --zone=public --add-port=40/tcp  # --permanent: mentés, TCP port 40
firewall-cmd --permanent --zone=public --add-port=41/tcp  # --permanent: mentés, TCP port 41
firewall-cmd --permanent --zone=public --add-port=42/tcp  # --permanent: mentés, TCP port 42
firewall-cmd --permanent --zone=public --add-port=43/tcp  # --permanent: mentés, TCP port 43
firewall-cmd --permanent --zone=public --add-port=44/tcp  # --permanent: mentés, TCP port 44
firewall-cmd --permanent --zone=public --add-port=45/tcp  # --permanent: mentés, TCP port 45
firewall-cmd --permanent --zone=public --add-port=46/tcp  # --permanent: mentés, TCP port 46
firewall-cmd --permanent --zone=public --add-port=47/tcp  # --permanent: mentés, TCP port 47
firewall-cmd --permanent --zone=public --add-port=48/tcp  # --permanent: mentés, TCP port 48
firewall-cmd --permanent --zone=public --add-port=49/tcp  # --permanent: mentés, TCP port 49
firewall-cmd --permanent --zone=public --add-port=50/tcp  # --permanent: mentés, TCP port 50
firewall-cmd --permanent --zone=public --add-port=51/tcp  # --permanent: mentés, TCP port 51
firewall-cmd --permanent --zone=public --add-port=52/tcp  # --permanent: mentés, TCP port 52
firewall-cmd --permanent --zone=public --add-port=53/tcp  # --permanent: mentés, TCP port 53
firewall-cmd --permanent --zone=public --add-port=54/tcp  # --permanent: mentés, TCP port 54
firewall-cmd --permanent --zone=public --add-port=55/tcp  # --permanent: mentés, TCP port 55
firewall-cmd --permanent --zone=public --add-port=56/tcp  # --permanent: mentés, TCP port 56
firewall-cmd --permanent --zone=public --add-port=57/tcp  # --permanent: mentés, TCP port 57
firewall-cmd --permanent --zone=public --add-port=58/tcp  # --permanent: mentés, TCP port 58
firewall-cmd --permanent --zone=public --add-port=59/tcp  # --permanent: mentés, TCP port 59
firewall-cmd --permanent --zone=public --add-port=60/tcp  # --permanent: mentés, TCP port 60
firewall-cmd --permanent --zone=public --add-port=61/tcp  # --permanent: mentés, TCP port 61
firewall-cmd --permanent --zone=public --add-port=62/tcp  # --permanent: mentés, TCP port 62
firewall-cmd --permanent --zone=public --add-port=63/tcp  # --permanent: mentés, TCP port 63
firewall-cmd --permanent --zone=public --add-port=64/tcp  # --permanent: mentés, TCP port 64
firewall-cmd --permanent --zone=public --add-port=65/tcp  # --permanent: mentés, TCP port 65
firewall-cmd --permanent --zone=public --add-port=66/tcp  # --permanent: mentés, TCP port 66
firewall-cmd --permanent --zone=public --add-port=67/tcp  # --permanent: mentés, TCP port 67
firewall-cmd --permanent --zone=public --add-port=68/tcp  # --permanent: mentés, TCP port 68
firewall-cmd --permanent --zone=public --add-port=69/tcp  # --permanent: mentés, TCP port 69
firewall-cmd --permanent --zone=public --add-port=70/tcp  # --permanent: mentés, TCP port 70
firewall-cmd --permanent --zone=public --add-port=71/tcp  # --permanent: mentés, TCP port 71
firewall-cmd --permanent --zone=public --add-port=72/tcp  # --permanent: mentés, TCP port 72
firewall-cmd --permanent --zone=public --add-port=73/tcp  # --permanent: mentés, TCP port 73
firewall-cmd --permanent --zone=public --add-port=74/tcp  # --permanent: mentés, TCP port 74
firewall-cmd --permanent --zone=public --add-port=75/tcp  # --permanent: mentés, TCP port 75
firewall-cmd --permanent --zone=public --add-port=76/tcp  # --permanent: mentés, TCP port 76
firewall-cmd --permanent --zone=public --add-port=77/tcp  # --permanent: mentés, TCP port 77
firewall-cmd --permanent --zone=public --add-port=78/tcp  # --permanent: mentés, TCP port 78
firewall-cmd --permanent --zone=public --add-port=79/tcp  # --permanent: mentés, TCP port 79
firewall-cmd --permanent --zone=public --add-port=80/tcp  # --permanent: mentés, TCP port 80
firewall-cmd --permanent --zone=public --add-port=81/tcp  # --permanent: mentés, TCP port 81
firewall-cmd --permanent --zone=public --add-port=82/tcp  # --permanent: mentés, TCP port 82
firewall-cmd --permanent --zone=public --add-port=83/tcp  # --permanent: mentés, TCP port 83
firewall-cmd --permanent --zone=public --add-port=84/tcp  # --permanent: mentés, TCP port 84
firewall-cmd --permanent --zone=public --add-port=85/tcp  # --permanent: mentés, TCP port 85
firewall-cmd --permanent --zone=public --add-port=86/tcp  # --permanent: mentés, TCP port 86
firewall-cmd --permanent --zone=public --add-port=87/tcp  # --permanent: mentés, TCP port 87
firewall-cmd --permanent --zone=public --add-port=88/tcp  # --permanent: mentés, TCP port 88
firewall-cmd --permanent --zone=public --add-port=89/tcp  # --permanent: mentés, TCP port 89
firewall-cmd --permanent --zone=public --add-port=90/tcp  # --permanent: mentés, TCP port 90
firewall-cmd --permanent --zone=public --add-port=91/tcp  # --permanent: mentés, TCP port 91
firewall-cmd --permanent --zone=public --add-port=92/tcp  # --permanent: mentés, TCP port 92
firewall-cmd --permanent --zone=public --add-port=93/tcp  # --permanent: mentés, TCP port 93
firewall-cmd --permanent --zone=public --add-port=94/tcp  # --permanent: mentés, TCP port 94
firewall-cmd --permanent --zone=public --add-port=95/tcp  # --permanent: mentés, TCP port 95
firewall-cmd --permanent --zone=public --add-port=96/tcp  # --permanent: mentés, TCP port 96
firewall-cmd --permanent --zone=public --add-port=97/tcp  # --permanent: mentés, TCP port 97
firewall-cmd --permanent --zone=public --add-port=98/tcp  # --permanent: mentés, TCP port 98
firewall-cmd --permanent --zone=public --add-port=99/tcp  # --permanent: mentés, TCP port 99
firewall-cmd --permanent --zone=public --add-port=100/tcp  # --permanent: mentés, TCP port 100
firewall-cmd --permanent --zone=public --add-port=101/tcp  # --permanent: mentés, TCP port 101
firewall-cmd --permanent --zone=public --add-port=102/tcp  # --permanent: mentés, TCP port 102
firewall-cmd --permanent --zone=public --add-port=103/tcp  # --permanent: mentés, TCP port 103
firewall-cmd --permanent --zone=public --add-port=104/tcp  # --permanent: mentés, TCP port 104
firewall-cmd --permanent --zone=public --add-port=105/tcp  # --permanent: mentés, TCP port 105
firewall-cmd --permanent --zone=public --add-port=106/tcp  # --permanent: mentés, TCP port 106
firewall-cmd --permanent --zone=public --add-port=107/tcp  # --permanent: mentés, TCP port 107
firewall-cmd --permanent --zone=public --add-port=108/tcp  # --permanent: mentés, TCP port 108
firewall-cmd --permanent --zone=public --add-port=109/tcp  # --permanent: mentés, TCP port 109
firewall-cmd --permanent --zone=public --add-port=110/tcp  # --permanent: mentés, TCP port 110
firewall-cmd --permanent --zone=public --add-port=111/tcp  # --permanent: mentés, TCP port 111
firewall-cmd --permanent --zone=public --add-port=112/tcp  # --permanent: mentés, TCP port 112
firewall-cmd --permanent --zone=public --add-port=113/tcp  # --permanent: mentés, TCP port 113
firewall-cmd --permanent --zone=public --add-port=114/tcp  # --permanent: mentés, TCP port 114
firewall-cmd --permanent --zone=public --add-port=115/tcp  # --permanent: mentés, TCP port 115
firewall-cmd --permanent --zone=public --add-port=116/tcp  # --permanent: mentés, TCP port 116
firewall-cmd --permanent --zone=public --add-port=117/tcp  # --permanent: mentés, TCP port 117
firewall-cmd --permanent --zone=public --add-port=118/tcp  # --permanent: mentés, TCP port 118
firewall-cmd --permanent --zone=public --add-port=119/tcp  # --permanent: mentés, TCP port 119
firewall-cmd --permanent --zone=public --add-port=120/tcp  # --permanent: mentés, TCP port 120
firewall-cmd --permanent --zone=public --add-port=121/tcp  # --permanent: mentés, TCP port 121
firewall-cmd --permanent --zone=public --add-port=122/tcp  # --permanent: mentés, TCP port 122
firewall-cmd --permanent --zone=public --add-port=123/tcp  # --permanent: mentés, TCP port 123
firewall-cmd --permanent --zone=public --add-port=124/tcp  # --permanent: mentés, TCP port 124
firewall-cmd --permanent --zone=public --add-port=125/tcp  # --permanent: mentés, TCP port 125
firewall-cmd --permanent --zone=public --add-port=126/tcp  # --permanent: mentés, TCP port 126
firewall-cmd --permanent --zone=public --add-port=127/tcp  # --permanent: mentés, TCP port 127
firewall-cmd --permanent --zone=public --add-port=128/tcp  # --permanent: mentés, TCP port 128
firewall-cmd --permanent --zone=public --add-port=129/tcp  # --permanent: mentés, TCP port 129
firewall-cmd --permanent --zone=public --add-port=130/tcp  # --permanent: mentés, TCP port 130
firewall-cmd --permanent --zone=public --add-port=131/tcp  # --permanent: mentés, TCP port 131
firewall-cmd --permanent --zone=public --add-port=132/tcp  # --permanent: mentés, TCP port 132
firewall-cmd --permanent --zone=public --add-port=133/tcp  # --permanent: mentés, TCP port 133
firewall-cmd --permanent --zone=public --add-port=134/tcp  # --permanent: mentés, TCP port 134
firewall-cmd --permanent --zone=public --add-port=135/tcp  # --permanent: mentés, TCP port 135
firewall-cmd --permanent --zone=public --add-port=136/tcp  # --permanent: mentés, TCP port 136
firewall-cmd --permanent --zone=public --add-port=137/tcp  # --permanent: mentés, TCP port 137
firewall-cmd --permanent --zone=public --add-port=138/tcp  # --permanent: mentés, TCP port 138
firewall-cmd --permanent --zone=public --add-port=139/tcp  # --permanent: mentés, TCP port 139
firewall-cmd --permanent --zone=public --add-port=140/tcp  # --permanent: mentés, TCP port 140
firewall-cmd --permanent --zone=public --add-port=141/tcp  # --permanent: mentés, TCP port 141
firewall-cmd --permanent --zone=public --add-port=142/tcp  # --permanent: mentés, TCP port 142
firewall-cmd --permanent --zone=public --add-port=143/tcp  # --permanent: mentés, TCP port 143
firewall-cmd --permanent --zone=public --add-port=144/tcp  # --permanent: mentés, TCP port 144
firewall-cmd --permanent --zone=public --add-port=145/tcp  # --permanent: mentés, TCP port 145
firewall-cmd --permanent --zone=public --add-port=146/tcp  # --permanent: mentés, TCP port 146
firewall-cmd --permanent --zone=public --add-port=147/tcp  # --permanent: mentés, TCP port 147
firewall-cmd --permanent --zone=public --add-port=148/tcp  # --permanent: mentés, TCP port 148
firewall-cmd --permanent --zone=public --add-port=149/tcp  # --permanent: mentés, TCP port 149
firewall-cmd --permanent --zone=public --add-port=150/tcp  # --permanent: mentés, TCP port 150
firewall-cmd --permanent --zone=public --add-port=151/tcp  # --permanent: mentés, TCP port 151
firewall-cmd --permanent --zone=public --add-port=152/tcp  # --permanent: mentés, TCP port 152
firewall-cmd --permanent --zone=public --add-port=153/tcp  # --permanent: mentés, TCP port 153
firewall-cmd --permanent --zone=public --add-port=154/tcp  # --permanent: mentés, TCP port 154
firewall-cmd --permanent --zone=public --add-port=155/tcp  # --permanent: mentés, TCP port 155
firewall-cmd --permanent --zone=public --add-port=156/tcp  # --permanent: mentés, TCP port 156
firewall-cmd --permanent --zone=public --add-port=157/tcp  # --permanent: mentés, TCP port 157
firewall-cmd --permanent --zone=public --add-port=158/tcp  # --permanent: mentés, TCP port 158
firewall-cmd --permanent --zone=public --add-port=159/tcp  # --permanent: mentés, TCP port 159
firewall-cmd --permanent --zone=public --add-port=160/tcp  # --permanent: mentés, TCP port 160
firewall-cmd --permanent --zone=public --add-port=161/tcp  # --permanent: mentés, TCP port 161
firewall-cmd --permanent --zone=public --add-port=162/tcp  # --permanent: mentés, TCP port 162
firewall-cmd --permanent --zone=public --add-port=163/tcp  # --permanent: mentés, TCP port 163
firewall-cmd --permanent --zone=public --add-port=164/tcp  # --permanent: mentés, TCP port 164
firewall-cmd --permanent --zone=public --add-port=165/tcp  # --permanent: mentés, TCP port 165
firewall-cmd --permanent --zone=public --add-port=166/tcp  # --permanent: mentés, TCP port 166
firewall-cmd --permanent --zone=public --add-port=167/tcp  # --permanent: mentés, TCP port 167
firewall-cmd --permanent --zone=public --add-port=168/tcp  # --permanent: mentés, TCP port 168
firewall-cmd --permanent --zone=public --add-port=169/tcp  # --permanent: mentés, TCP port 169
firewall-cmd --permanent --zone=public --add-port=170/tcp  # --permanent: mentés, TCP port 170
firewall-cmd --permanent --zone=public --add-port=171/tcp  # --permanent: mentés, TCP port 171
firewall-cmd --permanent --zone=public --add-port=172/tcp  # --permanent: mentés, TCP port 172
firewall-cmd --permanent --zone=public --add-port=173/tcp  # --permanent: mentés, TCP port 173
firewall-cmd --permanent --zone=public --add-port=174/tcp  # --permanent: mentés, TCP port 174
firewall-cmd --permanent --zone=public --add-port=175/tcp  # --permanent: mentés, TCP port 175
firewall-cmd --permanent --zone=public --add-port=176/tcp  # --permanent: mentés, TCP port 176
firewall-cmd --permanent --zone=public --add-port=177/tcp  # --permanent: mentés, TCP port 177
firewall-cmd --permanent --zone=public --add-port=178/tcp  # --permanent: mentés, TCP port 178
firewall-cmd --permanent --zone=public --add-port=179/tcp  # --permanent: mentés, TCP port 179
firewall-cmd --permanent --zone=public --add-port=180/tcp  # --permanent: mentés, TCP port 180
firewall-cmd --permanent --zone=public --add-port=181/tcp  # --permanent: mentés, TCP port 181
firewall-cmd --permanent --zone=public --add-port=182/tcp  # --permanent: mentés, TCP port 182
firewall-cmd --permanent --zone=public --add-port=183/tcp  # --permanent: mentés, TCP port 183
firewall-cmd --permanent --zone=public --add-port=184/tcp  # --permanent: mentés, TCP port 184
firewall-cmd --permanent --zone=public --add-port=185/tcp  # --permanent: mentés, TCP port 185
firewall-cmd --permanent --zone=public --add-port=186/tcp  # --permanent: mentés, TCP port 186
firewall-cmd --permanent --zone=public --add-port=187/tcp  # --permanent: mentés, TCP port 187
firewall-cmd --permanent --zone=public --add-port=188/tcp  # --permanent: mentés, TCP port 188
firewall-cmd --permanent --zone=public --add-port=189/tcp  # --permanent: mentés, TCP port 189
firewall-cmd --permanent --zone=public --add-port=190/tcp  # --permanent: mentés, TCP port 190
firewall-cmd --permanent --zone=public --add-port=191/tcp  # --permanent: mentés, TCP port 191
firewall-cmd --permanent --zone=public --add-port=192/tcp  # --permanent: mentés, TCP port 192
firewall-cmd --permanent --zone=public --add-port=193/tcp  # --permanent: mentés, TCP port 193
firewall-cmd --permanent --zone=public --add-port=194/tcp  # --permanent: mentés, TCP port 194
firewall-cmd --permanent --zone=public --add-port=195/tcp  # --permanent: mentés, TCP port 195
firewall-cmd --permanent --zone=public --add-port=196/tcp  # --permanent: mentés, TCP port 196
firewall-cmd --permanent --zone=public --add-port=197/tcp  # --permanent: mentés, TCP port 197
firewall-cmd --permanent --zone=public --add-port=198/tcp  # --permanent: mentés, TCP port 198
firewall-cmd --permanent --zone=public --add-port=199/tcp  # --permanent: mentés, TCP port 199
firewall-cmd --permanent --zone=public --add-port=200/tcp  # --permanent: mentés, TCP port 200
firewall-cmd --permanent --zone=public --add-port=201/tcp  # --permanent: mentés, TCP port 201
firewall-cmd --permanent --zone=public --add-port=202/tcp  # --permanent: mentés, TCP port 202
firewall-cmd --permanent --zone=public --add-port=203/tcp  # --permanent: mentés, TCP port 203
firewall-cmd --permanent --zone=public --add-port=204/tcp  # --permanent: mentés, TCP port 204
firewall-cmd --permanent --zone=public --add-port=205/tcp  # --permanent: mentés, TCP port 205
firewall-cmd --permanent --zone=public --add-port=206/tcp  # --permanent: mentés, TCP port 206
firewall-cmd --permanent --zone=public --add-port=207/tcp  # --permanent: mentés, TCP port 207
firewall-cmd --permanent --zone=public --add-port=208/tcp  # --permanent: mentés, TCP port 208
firewall-cmd --permanent --zone=public --add-port=209/tcp  # --permanent: mentés, TCP port 209
firewall-cmd --permanent --zone=public --add-port=210/tcp  # --permanent: mentés, TCP port 210
firewall-cmd --permanent --zone=public --add-port=211/tcp  # --permanent: mentés, TCP port 211
firewall-cmd --permanent --zone=public --add-port=212/tcp  # --permanent: mentés, TCP port 212
firewall-cmd --permanent --zone=public --add-port=213/tcp  # --permanent: mentés, TCP port 213
firewall-cmd --permanent --zone=public --add-port=214/tcp  # --permanent: mentés, TCP port 214
firewall-cmd --permanent --zone=public --add-port=215/tcp  # --permanent: mentés, TCP port 215
firewall-cmd --permanent --zone=public --add-port=216/tcp  # --permanent: mentés, TCP port 216
firewall-cmd --permanent --zone=public --add-port=217/tcp  # --permanent: mentés, TCP port 217
firewall-cmd --permanent --zone=public --add-port=218/tcp  # --permanent: mentés, TCP port 218
firewall-cmd --permanent --zone=public --add-port=219/tcp  # --permanent: mentés, TCP port 219
firewall-cmd --permanent --zone=public --add-port=220/tcp  # --permanent: mentés, TCP port 220
firewall-cmd --permanent --zone=public --add-port=221/tcp  # --permanent: mentés, TCP port 221
firewall-cmd --permanent --zone=public --add-port=222/tcp  # --permanent: mentés, TCP port 222
firewall-cmd --permanent --zone=public --add-port=223/tcp  # --permanent: mentés, TCP port 223
firewall-cmd --permanent --zone=public --add-port=224/tcp  # --permanent: mentés, TCP port 224
firewall-cmd --permanent --zone=public --add-port=225/tcp  # --permanent: mentés, TCP port 225
firewall-cmd --permanent --zone=public --add-port=226/tcp  # --permanent: mentés, TCP port 226
firewall-cmd --permanent --zone=public --add-port=227/tcp  # --permanent: mentés, TCP port 227
firewall-cmd --permanent --zone=public --add-port=228/tcp  # --permanent: mentés, TCP port 228
firewall-cmd --permanent --zone=public --add-port=229/tcp  # --permanent: mentés, TCP port 229
firewall-cmd --permanent --zone=public --add-port=230/tcp  # --permanent: mentés, TCP port 230
firewall-cmd --permanent --zone=public --add-port=231/tcp  # --permanent: mentés, TCP port 231
firewall-cmd --permanent --zone=public --add-port=232/tcp  # --permanent: mentés, TCP port 232
firewall-cmd --permanent --zone=public --add-port=233/tcp  # --permanent: mentés, TCP port 233
firewall-cmd --permanent --zone=public --add-port=234/tcp  # --permanent: mentés, TCP port 234
firewall-cmd --permanent --zone=public --add-port=235/tcp  # --permanent: mentés, TCP port 235
firewall-cmd --permanent --zone=public --add-port=236/tcp  # --permanent: mentés, TCP port 236
firewall-cmd --permanent --zone=public --add-port=237/tcp  # --permanent: mentés, TCP port 237
firewall-cmd --permanent --zone=public --add-port=238/tcp  # --permanent: mentés, TCP port 238
firewall-cmd --permanent --zone=public --add-port=239/tcp  # --permanent: mentés, TCP port 239
firewall-cmd --permanent --zone=public --add-port=240/tcp  # --permanent: mentés, TCP port 240
firewall-cmd --permanent --zone=public --add-port=241/tcp  # --permanent: mentés, TCP port 241
firewall-cmd --permanent --zone=public --add-port=242/tcp  # --permanent: mentés, TCP port 242
firewall-cmd --permanent --zone=public --add-port=243/tcp  # --permanent: mentés, TCP port 243
firewall-cmd --permanent --zone=public --add-port=244/tcp  # --permanent: mentés, TCP port 244
firewall-cmd --permanent --zone=public --add-port=245/tcp  # --permanent: mentés, TCP port 245
firewall-cmd --permanent --zone=public --add-port=246/tcp  # --permanent: mentés, TCP port 246
firewall-cmd --permanent --zone=public --add-port=247/tcp  # --permanent: mentés, TCP port 247
firewall-cmd --permanent --zone=public --add-port=248/tcp  # --permanent: mentés, TCP port 248
firewall-cmd --permanent --zone=public --add-port=249/tcp  # --permanent: mentés, TCP port 249
firewall-cmd --permanent --zone=public --add-port=250/tcp  # --permanent: mentés, TCP port 250
firewall-cmd --permanent --zone=public --add-port=251/tcp  # --permanent: mentés, TCP port 251
firewall-cmd --permanent --zone=public --add-port=252/tcp  # --permanent: mentés, TCP port 252
firewall-cmd --permanent --zone=public --add-port=253/tcp  # --permanent: mentés, TCP port 253
firewall-cmd --permanent --zone=public --add-port=254/tcp  # --permanent: mentés, TCP port 254
firewall-cmd --permanent --zone=public --add-port=255/tcp  # --permanent: mentés, TCP port 255
firewall-cmd --permanent --zone=public --add-port=256/tcp  # --permanent: mentés, TCP port 256
firewall-cmd --permanent --zone=public --add-port=257/tcp  # --permanent: mentés, TCP port 257
firewall-cmd --permanent --zone=public --add-port=258/tcp  # --permanent: mentés, TCP port 258
firewall-cmd --permanent --zone=public --add-port=259/tcp  # --permanent: mentés, TCP port 259
firewall-cmd --permanent --zone=public --add-port=260/tcp  # --permanent: mentés, TCP port 260
firewall-cmd --permanent --zone=public --add-port=261/tcp  # --permanent: mentés, TCP port 261
firewall-cmd --permanent --zone=public --add-port=262/tcp  # --permanent: mentés, TCP port 262
firewall-cmd --permanent --zone=public --add-port=263/tcp  # --permanent: mentés, TCP port 263
firewall-cmd --permanent --zone=public --add-port=264/tcp  # --permanent: mentés, TCP port 264
firewall-cmd --permanent --zone=public --add-port=265/tcp  # --permanent: mentés, TCP port 265
firewall-cmd --permanent --zone=public --add-port=266/tcp  # --permanent: mentés, TCP port 266
firewall-cmd --permanent --zone=public --add-port=267/tcp  # --permanent: mentés, TCP port 267
firewall-cmd --permanent --zone=public --add-port=268/tcp  # --permanent: mentés, TCP port 268
firewall-cmd --permanent --zone=public --add-port=269/tcp  # --permanent: mentés, TCP port 269
firewall-cmd --permanent --zone=public --add-port=270/tcp  # --permanent: mentés, TCP port 270
firewall-cmd --permanent --zone=public --add-port=271/tcp  # --permanent: mentés, TCP port 271
firewall-cmd --permanent --zone=public --add-port=272/tcp  # --permanent: mentés, TCP port 272
firewall-cmd --permanent --zone=public --add-port=273/tcp  # --permanent: mentés, TCP port 273
firewall-cmd --permanent --zone=public --add-port=274/tcp  # --permanent: mentés, TCP port 274
firewall-cmd --permanent --zone=public --add-port=275/tcp  # --permanent: mentés, TCP port 275
firewall-cmd --permanent --zone=public --add-port=276/tcp  # --permanent: mentés, TCP port 276
firewall-cmd --permanent --zone=public --add-port=277/tcp  # --permanent: mentés, TCP port 277
firewall-cmd --permanent --zone=public --add-port=278/tcp  # --permanent: mentés, TCP port 278
firewall-cmd --permanent --zone=public --add-port=279/tcp  # --permanent: mentés, TCP port 279
firewall-cmd --permanent --zone=public --add-port=280/tcp  # --permanent: mentés, TCP port 280
firewall-cmd --permanent --zone=public --add-port=281/tcp  # --permanent: mentés, TCP port 281
firewall-cmd --permanent --zone=public --add-port=282/tcp  # --permanent: mentés, TCP port 282
firewall-cmd --permanent --zone=public --add-port=283/tcp  # --permanent: mentés, TCP port 283
firewall-cmd --permanent --zone=public --add-port=284/tcp  # --permanent: mentés, TCP port 284
firewall-cmd --permanent --zone=public --add-port=285/tcp  # --permanent: mentés, TCP port 285
firewall-cmd --permanent --zone=public --add-port=286/tcp  # --permanent: mentés, TCP port 286
firewall-cmd --permanent --zone=public --add-port=287/tcp  # --permanent: mentés, TCP port 287
firewall-cmd --permanent --zone=public --add-port=288/tcp  # --permanent: mentés, TCP port 288
firewall-cmd --permanent --zone=public --add-port=289/tcp  # --permanent: mentés, TCP port 289
firewall-cmd --permanent --zone=public --add-port=290/tcp  # --permanent: mentés, TCP port 290
firewall-cmd --permanent --zone=public --add-port=291/tcp  # --permanent: mentés, TCP port 291
firewall-cmd --permanent --zone=public --add-port=292/tcp  # --permanent: mentés, TCP port 292
firewall-cmd --permanent --zone=public --add-port=293/tcp  # --permanent: mentés, TCP port 293
firewall-cmd --permanent --zone=public --add-port=294/tcp  # --permanent: mentés, TCP port 294
firewall-cmd --permanent --zone=public --add-port=295/tcp  # --permanent: mentés, TCP port 295
firewall-cmd --permanent --zone=public --add-port=296/tcp  # --permanent: mentés, TCP port 296
firewall-cmd --permanent --zone=public --add-port=297/tcp  # --permanent: mentés, TCP port 297
firewall-cmd --permanent --zone=public --add-port=298/tcp  # --permanent: mentés, TCP port 298
firewall-cmd --permanent --zone=public --add-port=299/tcp  # --permanent: mentés, TCP port 299
firewall-cmd --permanent --zone=public --add-port=300/tcp  # --permanent: mentés, TCP port 300
firewall-cmd --permanent --zone=public --add-port=301/tcp  # --permanent: mentés, TCP port 301
firewall-cmd --permanent --zone=public --add-port=302/tcp  # --permanent: mentés, TCP port 302
firewall-cmd --permanent --zone=public --add-port=303/tcp  # --permanent: mentés, TCP port 303
firewall-cmd --permanent --zone=public --add-port=304/tcp  # --permanent: mentés, TCP port 304
firewall-cmd --permanent --zone=public --add-port=305/tcp  # --permanent: mentés, TCP port 305
firewall-cmd --permanent --zone=public --add-port=306/tcp  # --permanent: mentés, TCP port 306
firewall-cmd --permanent --zone=public --add-port=307/tcp  # --permanent: mentés, TCP port 307
firewall-cmd --permanent --zone=public --add-port=308/tcp  # --permanent: mentés, TCP port 308
firewall-cmd --permanent --zone=public --add-port=309/tcp  # --permanent: mentés, TCP port 309
firewall-cmd --permanent --zone=public --add-port=310/tcp  # --permanent: mentés, TCP port 310
firewall-cmd --permanent --zone=public --add-port=311/tcp  # --permanent: mentés, TCP port 311
firewall-cmd --permanent --zone=public --add-port=312/tcp  # --permanent: mentés, TCP port 312
firewall-cmd --permanent --zone=public --add-port=313/tcp  # --permanent: mentés, TCP port 313
firewall-cmd --permanent --zone=public --add-port=314/tcp  # --permanent: mentés, TCP port 314
firewall-cmd --permanent --zone=public --add-port=315/tcp  # --permanent: mentés, TCP port 315
firewall-cmd --permanent --zone=public --add-port=316/tcp  # --permanent: mentés, TCP port 316
firewall-cmd --permanent --zone=public --add-port=317/tcp  # --permanent: mentés, TCP port 317
firewall-cmd --permanent --zone=public --add-port=318/tcp  # --permanent: mentés, TCP port 318
firewall-cmd --permanent --zone=public --add-port=319/tcp  # --permanent: mentés, TCP port 319
firewall-cmd --permanent --zone=public --add-port=320/tcp  # --permanent: mentés, TCP port 320
firewall-cmd --permanent --zone=public --add-port=321/tcp  # --permanent: mentés, TCP port 321
firewall-cmd --permanent --zone=public --add-port=322/tcp  # --permanent: mentés, TCP port 322
firewall-cmd --permanent --zone=public --add-port=323/tcp  # --permanent: mentés, TCP port 323
firewall-cmd --permanent --zone=public --add-port=324/tcp  # --permanent: mentés, TCP port 324
firewall-cmd --permanent --zone=public --add-port=325/tcp  # --permanent: mentés, TCP port 325
firewall-cmd --permanent --zone=public --add-port=326/tcp  # --permanent: mentés, TCP port 326
firewall-cmd --permanent --zone=public --add-port=327/tcp  # --permanent: mentés, TCP port 327
firewall-cmd --permanent --zone=public --add-port=328/tcp  # --permanent: mentés, TCP port 328
firewall-cmd --permanent --zone=public --add-port=329/tcp  # --permanent: mentés, TCP port 329
firewall-cmd --permanent --zone=public --add-port=330/tcp  # --permanent: mentés, TCP port 330
firewall-cmd --permanent --zone=public --add-port=331/tcp  # --permanent: mentés, TCP port 331
firewall-cmd --permanent --zone=public --add-port=332/tcp  # --permanent: mentés, TCP port 332
firewall-cmd --permanent --zone=public --add-port=333/tcp  # --permanent: mentés, TCP port 333
firewall-cmd --permanent --zone=public --add-port=334/tcp  # --permanent: mentés, TCP port 334
firewall-cmd --permanent --zone=public --add-port=335/tcp  # --permanent: mentés, TCP port 335
firewall-cmd --permanent --zone=public --add-port=336/tcp  # --permanent: mentés, TCP port 336
firewall-cmd --permanent --zone=public --add-port=337/tcp  # --permanent: mentés, TCP port 337
firewall-cmd --permanent --zone=public --add-port=338/tcp  # --permanent: mentés, TCP port 338
firewall-cmd --permanent --zone=public --add-port=339/tcp  # --permanent: mentés, TCP port 339
firewall-cmd --permanent --zone=public --add-port=340/tcp  # --permanent: mentés, TCP port 340
firewall-cmd --permanent --zone=public --add-port=341/tcp  # --permanent: mentés, TCP port 341
firewall-cmd --permanent --zone=public --add-port=342/tcp  # --permanent: mentés, TCP port 342
firewall-cmd --permanent --zone=public --add-port=343/tcp  # --permanent: mentés, TCP port 343
firewall-cmd --permanent --zone=public --add-port=344/tcp  # --permanent: mentés, TCP port 344
firewall-cmd --permanent --zone=public --add-port=345/tcp  # --permanent: mentés, TCP port 345
firewall-cmd --permanent --zone=public --add-port=346/tcp  # --permanent: mentés, TCP port 346
firewall-cmd --permanent --zone=public --add-port=347/tcp  # --permanent: mentés, TCP port 347
firewall-cmd --permanent --zone=public --add-port=348/tcp  # --permanent: mentés, TCP port 348
firewall-cmd --permanent --zone=public --add-port=349/tcp  # --permanent: mentés, TCP port 349
firewall-cmd --permanent --zone=public --add-port=350/tcp  # --permanent: mentés, TCP port 350
firewall-cmd --permanent --zone=public --add-port=351/tcp  # --permanent: mentés, TCP port 351
firewall-cmd --permanent --zone=public --add-port=352/tcp  # --permanent: mentés, TCP port 352
firewall-cmd --permanent --zone=public --add-port=353/tcp  # --permanent: mentés, TCP port 353
firewall-cmd --permanent --zone=public --add-port=354/tcp  # --permanent: mentés, TCP port 354
firewall-cmd --permanent --zone=public --add-port=355/tcp  # --permanent: mentés, TCP port 355
firewall-cmd --permanent --zone=public --add-port=356/tcp  # --permanent: mentés, TCP port 356
firewall-cmd --permanent --zone=public --add-port=357/tcp  # --permanent: mentés, TCP port 357
firewall-cmd --permanent --zone=public --add-port=358/tcp  # --permanent: mentés, TCP port 358
firewall-cmd --permanent --zone=public --add-port=359/tcp  # --permanent: mentés, TCP port 359
firewall-cmd --permanent --zone=public --add-port=360/tcp  # --permanent: mentés, TCP port 360
firewall-cmd --permanent --zone=public --add-port=361/tcp  # --permanent: mentés, TCP port 361
firewall-cmd --permanent --zone=public --add-port=362/tcp  # --permanent: mentés, TCP port 362
firewall-cmd --permanent --zone=public --add-port=363/tcp  # --permanent: mentés, TCP port 363
firewall-cmd --permanent --zone=public --add-port=364/tcp  # --permanent: mentés, TCP port 364
firewall-cmd --permanent --zone=public --add-port=365/tcp  # --permanent: mentés, TCP port 365
firewall-cmd --permanent --zone=public --add-port=366/tcp  # --permanent: mentés, TCP port 366
firewall-cmd --permanent --zone=public --add-port=367/tcp  # --permanent: mentés, TCP port 367
firewall-cmd --permanent --zone=public --add-port=368/tcp  # --permanent: mentés, TCP port 368
firewall-cmd --permanent --zone=public --add-port=369/tcp  # --permanent: mentés, TCP port 369
firewall-cmd --permanent --zone=public --add-port=370/tcp  # --permanent: mentés, TCP port 370
firewall-cmd --permanent --zone=public --add-port=371/tcp  # --permanent: mentés, TCP port 371
firewall-cmd --permanent --zone=public --add-port=372/tcp  # --permanent: mentés, TCP port 372
firewall-cmd --permanent --zone=public --add-port=373/tcp  # --permanent: mentés, TCP port 373
firewall-cmd --permanent --zone=public --add-port=374/tcp  # --permanent: mentés, TCP port 374
firewall-cmd --permanent --zone=public --add-port=375/tcp  # --permanent: mentés, TCP port 375
firewall-cmd --permanent --zone=public --add-port=376/tcp  # --permanent: mentés, TCP port 376
firewall-cmd --permanent --zone=public --add-port=377/tcp  # --permanent: mentés, TCP port 377
firewall-cmd --permanent --zone=public --add-port=378/tcp  # --permanent: mentés, TCP port 378
firewall-cmd --permanent --zone=public --add-port=379/tcp  # --permanent: mentés, TCP port 379
firewall-cmd --permanent --zone=public --add-port=380/tcp  # --permanent: mentés, TCP port 380
firewall-cmd --permanent --zone=public --add-port=381/tcp  # --permanent: mentés, TCP port 381
firewall-cmd --permanent --zone=public --add-port=382/tcp  # --permanent: mentés, TCP port 382
firewall-cmd --permanent --zone=public --add-port=383/tcp  # --permanent: mentés, TCP port 383
firewall-cmd --permanent --zone=public --add-port=384/tcp  # --permanent: mentés, TCP port 384
firewall-cmd --permanent --zone=public --add-port=385/tcp  # --permanent: mentés, TCP port 385
firewall-cmd --permanent --zone=public --add-port=386/tcp  # --permanent: mentés, TCP port 386
firewall-cmd --permanent --zone=public --add-port=387/tcp  # --permanent: mentés, TCP port 387
firewall-cmd --permanent --zone=public --add-port=388/tcp  # --permanent: mentés, TCP port 388
firewall-cmd --permanent --zone=public --add-port=389/tcp  # --permanent: mentés, TCP port 389
firewall-cmd --permanent --zone=public --add-port=390/tcp  # --permanent: mentés, TCP port 390
firewall-cmd --permanent --zone=public --add-port=391/tcp  # --permanent: mentés, TCP port 391
firewall-cmd --permanent --zone=public --add-port=392/tcp  # --permanent: mentés, TCP port 392
firewall-cmd --permanent --zone=public --add-port=393/tcp  # --permanent: mentés, TCP port 393
firewall-cmd --permanent --zone=public --add-port=394/tcp  # --permanent: mentés, TCP port 394
firewall-cmd --permanent --zone=public --add-port=395/tcp  # --permanent: mentés, TCP port 395
firewall-cmd --permanent --zone=public --add-port=396/tcp  # --permanent: mentés, TCP port 396
firewall-cmd --permanent --zone=public --add-port=397/tcp  # --permanent: mentés, TCP port 397
firewall-cmd --permanent --zone=public --add-port=398/tcp  # --permanent: mentés, TCP port 398
firewall-cmd --permanent --zone=public --add-port=399/tcp  # --permanent: mentés, TCP port 399
firewall-cmd --permanent --zone=public --add-port=400/tcp  # --permanent: mentés, TCP port 400
firewall-cmd --permanent --zone=public --add-port=401/tcp  # --permanent: mentés, TCP port 401
firewall-cmd --permanent --zone=public --add-port=402/tcp  # --permanent: mentés, TCP port 402
firewall-cmd --permanent --zone=public --add-port=403/tcp  # --permanent: mentés, TCP port 403
firewall-cmd --permanent --zone=public --add-port=404/tcp  # --permanent: mentés, TCP port 404
firewall-cmd --permanent --zone=public --add-port=405/tcp  # --permanent: mentés, TCP port 405
firewall-cmd --permanent --zone=public --add-port=406/tcp  # --permanent: mentés, TCP port 406
firewall-cmd --permanent --zone=public --add-port=407/tcp  # --permanent: mentés, TCP port 407
firewall-cmd --permanent --zone=public --add-port=408/tcp  # --permanent: mentés, TCP port 408
firewall-cmd --permanent --zone=public --add-port=409/tcp  # --permanent: mentés, TCP port 409
firewall-cmd --permanent --zone=public --add-port=410/tcp  # --permanent: mentés, TCP port 410
firewall-cmd --permanent --zone=public --add-port=411/tcp  # --permanent: mentés, TCP port 411
firewall-cmd --permanent --zone=public --add-port=412/tcp  # --permanent: mentés, TCP port 412
firewall-cmd --permanent --zone=public --add-port=413/tcp  # --permanent: mentés, TCP port 413
firewall-cmd --permanent --zone=public --add-port=414/tcp  # --permanent: mentés, TCP port 414
firewall-cmd --permanent --zone=public --add-port=415/tcp  # --permanent: mentés, TCP port 415
firewall-cmd --permanent --zone=public --add-port=416/tcp  # --permanent: mentés, TCP port 416
firewall-cmd --permanent --zone=public --add-port=417/tcp  # --permanent: mentés, TCP port 417
firewall-cmd --permanent --zone=public --add-port=418/tcp  # --permanent: mentés, TCP port 418
firewall-cmd --permanent --zone=public --add-port=419/tcp  # --permanent: mentés, TCP port 419
firewall-cmd --permanent --zone=public --add-port=420/tcp  # --permanent: mentés, TCP port 420
firewall-cmd --permanent --zone=public --add-port=421/tcp  # --permanent: mentés, TCP port 421
firewall-cmd --permanent --zone=public --add-port=422/tcp  # --permanent: mentés, TCP port 422
firewall-cmd --permanent --zone=public --add-port=423/tcp  # --permanent: mentés, TCP port 423
firewall-cmd --permanent --zone=public --add-port=424/tcp  # --permanent: mentés, TCP port 424
firewall-cmd --permanent --zone=public --add-port=425/tcp  # --permanent: mentés, TCP port 425
firewall-cmd --permanent --zone=public --add-port=426/tcp  # --permanent: mentés, TCP port 426
firewall-cmd --permanent --zone=public --add-port=427/tcp  # --permanent: mentés, TCP port 427
firewall-cmd --permanent --zone=public --add-port=428/tcp  # --permanent: mentés, TCP port 428
firewall-cmd --permanent --zone=public --add-port=429/tcp  # --permanent: mentés, TCP port 429
firewall-cmd --permanent --zone=public --add-port=430/tcp  # --permanent: mentés, TCP port 430
firewall-cmd --permanent --zone=public --add-port=431/tcp  # --permanent: mentés, TCP port 431
firewall-cmd --permanent --zone=public --add-port=432/tcp  # --permanent: mentés, TCP port 432
firewall-cmd --permanent --zone=public --add-port=433/tcp  # --permanent: mentés, TCP port 433
firewall-cmd --permanent --zone=public --add-port=434/tcp  # --permanent: mentés, TCP port 434
firewall-cmd --permanent --zone=public --add-port=435/tcp  # --permanent: mentés, TCP port 435
firewall-cmd --permanent --zone=public --add-port=436/tcp  # --permanent: mentés, TCP port 436
firewall-cmd --permanent --zone=public --add-port=437/tcp  # --permanent: mentés, TCP port 437
firewall-cmd --permanent --zone=public --add-port=438/tcp  # --permanent: mentés, TCP port 438
firewall-cmd --permanent --zone=public --add-port=439/tcp  # --permanent: mentés, TCP port 439
firewall-cmd --permanent --zone=public --add-port=440/tcp  # --permanent: mentés, TCP port 440
firewall-cmd --permanent --zone=public --add-port=441/tcp  # --permanent: mentés, TCP port 441
firewall-cmd --permanent --zone=public --add-port=442/tcp  # --permanent: mentés, TCP port 442
firewall-cmd --permanent --zone=public --add-port=443/tcp  # --permanent: mentés, TCP port 443
firewall-cmd --permanent --zone=public --add-port=444/tcp  # --permanent: mentés, TCP port 444
firewall-cmd --permanent --zone=public --add-port=445/tcp  # --permanent: mentés, TCP port 445
firewall-cmd --permanent --zone=public --add-port=446/tcp  # --permanent: mentés, TCP port 446
firewall-cmd --permanent --zone=public --add-port=447/tcp  # --permanent: mentés, TCP port 447
firewall-cmd --permanent --zone=public --add-port=448/tcp  # --permanent: mentés, TCP port 448
firewall-cmd --permanent --zone=public --add-port=449/tcp  # --permanent: mentés, TCP port 449
firewall-cmd --permanent --zone=public --add-port=450/tcp  # --permanent: mentés, TCP port 450
firewall-cmd --permanent --zone=public --add-port=451/tcp  # --permanent: mentés, TCP port 451
firewall-cmd --permanent --zone=public --add-port=452/tcp  # --permanent: mentés, TCP port 452
firewall-cmd --permanent --zone=public --add-port=453/tcp  # --permanent: mentés, TCP port 453
firewall-cmd --permanent --zone=public --add-port=454/tcp  # --permanent: mentés, TCP port 454
firewall-cmd --permanent --zone=public --add-port=455/tcp  # --permanent: mentés, TCP port 455
firewall-cmd --permanent --zone=public --add-port=456/tcp  # --permanent: mentés, TCP port 456
firewall-cmd --permanent --zone=public --add-port=457/tcp  # --permanent: mentés, TCP port 457
firewall-cmd --permanent --zone=public --add-port=458/tcp  # --permanent: mentés, TCP port 458
firewall-cmd --permanent --zone=public --add-port=459/tcp  # --permanent: mentés, TCP port 459
firewall-cmd --permanent --zone=public --add-port=460/tcp  # --permanent: mentés, TCP port 460
firewall-cmd --permanent --zone=public --add-port=461/tcp  # --permanent: mentés, TCP port 461
firewall-cmd --permanent --zone=public --add-port=462/tcp  # --permanent: mentés, TCP port 462
firewall-cmd --permanent --zone=public --add-port=463/tcp  # --permanent: mentés, TCP port 463
firewall-cmd --permanent --zone=public --add-port=464/tcp  # --permanent: mentés, TCP port 464
firewall-cmd --permanent --zone=public --add-port=465/tcp  # --permanent: mentés, TCP port 465
firewall-cmd --permanent --zone=public --add-port=466/tcp  # --permanent: mentés, TCP port 466
firewall-cmd --permanent --zone=public --add-port=467/tcp  # --permanent: mentés, TCP port 467
firewall-cmd --permanent --zone=public --add-port=468/tcp  # --permanent: mentés, TCP port 468
firewall-cmd --permanent --zone=public --add-port=469/tcp  # --permanent: mentés, TCP port 469
firewall-cmd --permanent --zone=public --add-port=470/tcp  # --permanent: mentés, TCP port 470
firewall-cmd --permanent --zone=public --add-port=471/tcp  # --permanent: mentés, TCP port 471
firewall-cmd --permanent --zone=public --add-port=472/tcp  # --permanent: mentés, TCP port 472
firewall-cmd --permanent --zone=public --add-port=473/tcp  # --permanent: mentés, TCP port 473
firewall-cmd --permanent --zone=public --add-port=474/tcp  # --permanent: mentés, TCP port 474
firewall-cmd --permanent --zone=public --add-port=475/tcp  # --permanent: mentés, TCP port 475
firewall-cmd --permanent --zone=public --add-port=476/tcp  # --permanent: mentés, TCP port 476
firewall-cmd --permanent --zone=public --add-port=477/tcp  # --permanent: mentés, TCP port 477
firewall-cmd --permanent --zone=public --add-port=478/tcp  # --permanent: mentés, TCP port 478
firewall-cmd --permanent --zone=public --add-port=479/tcp  # --permanent: mentés, TCP port 479
firewall-cmd --permanent --zone=public --add-port=480/tcp  # --permanent: mentés, TCP port 480
firewall-cmd --permanent --zone=public --add-port=481/tcp  # --permanent: mentés, TCP port 481
firewall-cmd --permanent --zone=public --add-port=482/tcp  # --permanent: mentés, TCP port 482
firewall-cmd --permanent --zone=public --add-port=483/tcp  # --permanent: mentés, TCP port 483
firewall-cmd --permanent --zone=public --add-port=484/tcp  # --permanent: mentés, TCP port 484
firewall-cmd --permanent --zone=public --add-port=485/tcp  # --permanent: mentés, TCP port 485
firewall-cmd --permanent --zone=public --add-port=486/tcp  # --permanent: mentés, TCP port 486
firewall-cmd --permanent --zone=public --add-port=487/tcp  # --permanent: mentés, TCP port 487
firewall-cmd --permanent --zone=public --add-port=488/tcp  # --permanent: mentés, TCP port 488
firewall-cmd --permanent --zone=public --add-port=489/tcp  # --permanent: mentés, TCP port 489
firewall-cmd --permanent --zone=public --add-port=490/tcp  # --permanent: mentés, TCP port 490
firewall-cmd --permanent --zone=public --add-port=491/tcp  # --permanent: mentés, TCP port 491
firewall-cmd --permanent --zone=public --add-port=492/tcp  # --permanent: mentés, TCP port 492
firewall-cmd --permanent --zone=public --add-port=493/tcp  # --permanent: mentés, TCP port 493
firewall-cmd --permanent --zone=public --add-port=494/tcp  # --permanent: mentés, TCP port 494
firewall-cmd --permanent --zone=public --add-port=495/tcp  # --permanent: mentés, TCP port 495
firewall-cmd --permanent --zone=public --add-port=496/tcp  # --permanent: mentés, TCP port 496
firewall-cmd --permanent --zone=public --add-port=497/tcp  # --permanent: mentés, TCP port 497
firewall-cmd --permanent --zone=public --add-port=498/tcp  # --permanent: mentés, TCP port 498
firewall-cmd --permanent --zone=public --add-port=499/tcp  # --permanent: mentés, TCP port 499
firewall-cmd --permanent --zone=public --add-port=500/tcp  # --permanent: mentés, TCP port 500
firewall-cmd --permanent --zone=public --add-port=501/tcp  # --permanent: mentés, TCP port 501
firewall-cmd --permanent --zone=public --add-port=502/tcp  # --permanent: mentés, TCP port 502
firewall-cmd --permanent --zone=public --add-port=503/tcp  # --permanent: mentés, TCP port 503
firewall-cmd --permanent --zone=public --add-port=504/tcp  # --permanent: mentés, TCP port 504
firewall-cmd --permanent --zone=public --add-port=505/tcp  # --permanent: mentés, TCP port 505
firewall-cmd --permanent --zone=public --add-port=506/tcp  # --permanent: mentés, TCP port 506
firewall-cmd --permanent --zone=public --add-port=507/tcp  # --permanent: mentés, TCP port 507
firewall-cmd --permanent --zone=public --add-port=508/tcp  # --permanent: mentés, TCP port 508
firewall-cmd --permanent --zone=public --add-port=509/tcp  # --permanent: mentés, TCP port 509
firewall-cmd --permanent --zone=public --add-port=510/tcp  # --permanent: mentés, TCP port 510
firewall-cmd --permanent --zone=public --add-port=511/tcp  # --permanent: mentés, TCP port 511
firewall-cmd --permanent --zone=public --add-port=512/tcp  # --permanent: mentés, TCP port 512
firewall-cmd --permanent --zone=public --add-port=513/tcp  # --permanent: mentés, TCP port 513
firewall-cmd --permanent --zone=public --add-port=514/tcp  # --permanent: mentés, TCP port 514
firewall-cmd --permanent --zone=public --add-port=515/tcp  # --permanent: mentés, TCP port 515
firewall-cmd --permanent --zone=public --add-port=516/tcp  # --permanent: mentés, TCP port 516
firewall-cmd --permanent --zone=public --add-port=517/tcp  # --permanent: mentés, TCP port 517
firewall-cmd --permanent --zone=public --add-port=518/tcp  # --permanent: mentés, TCP port 518
firewall-cmd --permanent --zone=public --add-port=519/tcp  # --permanent: mentés, TCP port 519
firewall-cmd --permanent --zone=public --add-port=520/tcp  # --permanent: mentés, TCP port 520
firewall-cmd --permanent --zone=public --add-port=521/tcp  # --permanent: mentés, TCP port 521
firewall-cmd --permanent --zone=public --add-port=522/tcp  # --permanent: mentés, TCP port 522
firewall-cmd --permanent --zone=public --add-port=523/tcp  # --permanent: mentés, TCP port 523
firewall-cmd --permanent --zone=public --add-port=524/tcp  # --permanent: mentés, TCP port 524
firewall-cmd --permanent --zone=public --add-port=525/tcp  # --permanent: mentés, TCP port 525
firewall-cmd --permanent --zone=public --add-port=526/tcp  # --permanent: mentés, TCP port 526
firewall-cmd --permanent --zone=public --add-port=527/tcp  # --permanent: mentés, TCP port 527
firewall-cmd --permanent --zone=public --add-port=528/tcp  # --permanent: mentés, TCP port 528
firewall-cmd --permanent --zone=public --add-port=529/tcp  # --permanent: mentés, TCP port 529
firewall-cmd --permanent --zone=public --add-port=530/tcp  # --permanent: mentés, TCP port 530
firewall-cmd --permanent --zone=public --add-port=531/tcp  # --permanent: mentés, TCP port 531
firewall-cmd --permanent --zone=public --add-port=532/tcp  # --permanent: mentés, TCP port 532
firewall-cmd --permanent --zone=public --add-port=533/tcp  # --permanent: mentés, TCP port 533
firewall-cmd --permanent --zone=public --add-port=534/tcp  # --permanent: mentés, TCP port 534
firewall-cmd --permanent --zone=public --add-port=535/tcp  # --permanent: mentés, TCP port 535
firewall-cmd --permanent --zone=public --add-port=536/tcp  # --permanent: mentés, TCP port 536
firewall-cmd --permanent --zone=public --add-port=537/tcp  # --permanent: mentés, TCP port 537
firewall-cmd --permanent --zone=public --add-port=538/tcp  # --permanent: mentés, TCP port 538
firewall-cmd --permanent --zone=public --add-port=539/tcp  # --permanent: mentés, TCP port 539
firewall-cmd --permanent --zone=public --add-port=540/tcp  # --permanent: mentés, TCP port 540
firewall-cmd --permanent --zone=public --add-port=541/tcp  # --permanent: mentés, TCP port 541
firewall-cmd --permanent --zone=public --add-port=542/tcp  # --permanent: mentés, TCP port 542
firewall-cmd --permanent --zone=public --add-port=543/tcp  # --permanent: mentés, TCP port 543
firewall-cmd --permanent --zone=public --add-port=544/tcp  # --permanent: mentés, TCP port 544
firewall-cmd --permanent --zone=public --add-port=545/tcp  # --permanent: mentés, TCP port 545
firewall-cmd --permanent --zone=public --add-port=546/tcp  # --permanent: mentés, TCP port 546
firewall-cmd --permanent --zone=public --add-port=547/tcp  # --permanent: mentés, TCP port 547
firewall-cmd --permanent --zone=public --add-port=548/tcp  # --permanent: mentés, TCP port 548
firewall-cmd --permanent --zone=public --add-port=549/tcp  # --permanent: mentés, TCP port 549
firewall-cmd --permanent --zone=public --add-port=550/tcp  # --permanent: mentés, TCP port 550
firewall-cmd --permanent --zone=public --add-port=551/tcp  # --permanent: mentés, TCP port 551
firewall-cmd --permanent --zone=public --add-port=552/tcp  # --permanent: mentés, TCP port 552
firewall-cmd --permanent --zone=public --add-port=553/tcp  # --permanent: mentés, TCP port 553
firewall-cmd --permanent --zone=public --add-port=554/tcp  # --permanent: mentés, TCP port 554
firewall-cmd --permanent --zone=public --add-port=555/tcp  # --permanent: mentés, TCP port 555
firewall-cmd --permanent --zone=public --add-port=556/tcp  # --permanent: mentés, TCP port 556
firewall-cmd --permanent --zone=public --add-port=557/tcp  # --permanent: mentés, TCP port 557
firewall-cmd --permanent --zone=public --add-port=558/tcp  # --permanent: mentés, TCP port 558
firewall-cmd --permanent --zone=public --add-port=559/tcp  # --permanent: mentés, TCP port 559
firewall-cmd --permanent --zone=public --add-port=560/tcp  # --permanent: mentés, TCP port 560
firewall-cmd --permanent --zone=public --add-port=561/tcp  # --permanent: mentés, TCP port 561
firewall-cmd --permanent --zone=public --add-port=562/tcp  # --permanent: mentés, TCP port 562
firewall-cmd --permanent --zone=public --add-port=563/tcp  # --permanent: mentés, TCP port 563
firewall-cmd --permanent --zone=public --add-port=564/tcp  # --permanent: mentés, TCP port 564
firewall-cmd --permanent --zone=public --add-port=565/tcp  # --permanent: mentés, TCP port 565
firewall-cmd --permanent --zone=public --add-port=566/tcp  # --permanent: mentés, TCP port 566
firewall-cmd --permanent --zone=public --add-port=567/tcp  # --permanent: mentés, TCP port 567
firewall-cmd --permanent --zone=public --add-port=568/tcp  # --permanent: mentés, TCP port 568
firewall-cmd --permanent --zone=public --add-port=569/tcp  # --permanent: mentés, TCP port 569
firewall-cmd --permanent --zone=public --add-port=570/tcp  # --permanent: mentés, TCP port 570
firewall-cmd --permanent --zone=public --add-port=571/tcp  # --permanent: mentés, TCP port 571
firewall-cmd --permanent --zone=public --add-port=572/tcp  # --permanent: mentés, TCP port 572
firewall-cmd --permanent --zone=public --add-port=573/tcp  # --permanent: mentés, TCP port 573
firewall-cmd --permanent --zone=public --add-port=574/tcp  # --permanent: mentés, TCP port 574
firewall-cmd --permanent --zone=public --add-port=575/tcp  # --permanent: mentés, TCP port 575
firewall-cmd --permanent --zone=public --add-port=576/tcp  # --permanent: mentés, TCP port 576
firewall-cmd --permanent --zone=public --add-port=577/tcp  # --permanent: mentés, TCP port 577
firewall-cmd --permanent --zone=public --add-port=578/tcp  # --permanent: mentés, TCP port 578
firewall-cmd --permanent --zone=public --add-port=579/tcp  # --permanent: mentés, TCP port 579
firewall-cmd --permanent --zone=public --add-port=580/tcp  # --permanent: mentés, TCP port 580
firewall-cmd --permanent --zone=public --add-port=581/tcp  # --permanent: mentés, TCP port 581
firewall-cmd --permanent --zone=public --add-port=582/tcp  # --permanent: mentés, TCP port 582
firewall-cmd --permanent --zone=public --add-port=583/tcp  # --permanent: mentés, TCP port 583
firewall-cmd --permanent --zone=public --add-port=584/tcp  # --permanent: mentés, TCP port 584
firewall-cmd --permanent --zone=public --add-port=585/tcp  # --permanent: mentés, TCP port 585
firewall-cmd --permanent --zone=public --add-port=586/tcp  # --permanent: mentés, TCP port 586
firewall-cmd --permanent --zone=public --add-port=587/tcp  # --permanent: mentés, TCP port 587
firewall-cmd --permanent --zone=public --add-port=588/tcp  # --permanent: mentés, TCP port 588
firewall-cmd --permanent --zone=public --add-port=589/tcp  # --permanent: mentés, TCP port 589
firewall-cmd --permanent --zone=public --add-port=590/tcp  # --permanent: mentés, TCP port 590
firewall-cmd --permanent --zone=public --add-port=591/tcp  # --permanent: mentés, TCP port 591
firewall-cmd --permanent --zone=public --add-port=592/tcp  # --permanent: mentés, TCP port 592
firewall-cmd --permanent --zone=public --add-port=593/tcp  # --permanent: mentés, TCP port 593
firewall-cmd --permanent --zone=public --add-port=594/tcp  # --permanent: mentés, TCP port 594
firewall-cmd --permanent --zone=public --add-port=595/tcp  # --permanent: mentés, TCP port 595
firewall-cmd --permanent --zone=public --add-port=596/tcp  # --permanent: mentés, TCP port 596
firewall-cmd --permanent --zone=public --add-port=597/tcp  # --permanent: mentés, TCP port 597
firewall-cmd --permanent --zone=public --add-port=598/tcp  # --permanent: mentés, TCP port 598
firewall-cmd --permanent --zone=public --add-port=599/tcp  # --permanent: mentés, TCP port 599
firewall-cmd --permanent --zone=public --add-port=600/tcp  # --permanent: mentés, TCP port 600
firewall-cmd --permanent --zone=public --add-port=601/tcp  # --permanent: mentés, TCP port 601
firewall-cmd --permanent --zone=public --add-port=602/tcp  # --permanent: mentés, TCP port 602
firewall-cmd --permanent --zone=public --add-port=603/tcp  # --permanent: mentés, TCP port 603
firewall-cmd --permanent --zone=public --add-port=604/tcp  # --permanent: mentés, TCP port 604
firewall-cmd --permanent --zone=public --add-port=605/tcp  # --permanent: mentés, TCP port 605
firewall-cmd --permanent --zone=public --add-port=606/tcp  # --permanent: mentés, TCP port 606
firewall-cmd --permanent --zone=public --add-port=607/tcp  # --permanent: mentés, TCP port 607
firewall-cmd --permanent --zone=public --add-port=608/tcp  # --permanent: mentés, TCP port 608
firewall-cmd --permanent --zone=public --add-port=609/tcp  # --permanent: mentés, TCP port 609
firewall-cmd --permanent --zone=public --add-port=610/tcp  # --permanent: mentés, TCP port 610
firewall-cmd --permanent --zone=public --add-port=611/tcp  # --permanent: mentés, TCP port 611
firewall-cmd --permanent --zone=public --add-port=612/tcp  # --permanent: mentés, TCP port 612
firewall-cmd --permanent --zone=public --add-port=613/tcp  # --permanent: mentés, TCP port 613
firewall-cmd --permanent --zone=public --add-port=614/tcp  # --permanent: mentés, TCP port 614
firewall-cmd --permanent --zone=public --add-port=615/tcp  # --permanent: mentés, TCP port 615
firewall-cmd --permanent --zone=public --add-port=616/tcp  # --permanent: mentés, TCP port 616
firewall-cmd --permanent --zone=public --add-port=617/tcp  # --permanent: mentés, TCP port 617
firewall-cmd --permanent --zone=public --add-port=618/tcp  # --permanent: mentés, TCP port 618
firewall-cmd --permanent --zone=public --add-port=619/tcp  # --permanent: mentés, TCP port 619
firewall-cmd --permanent --zone=public --add-port=620/tcp  # --permanent: mentés, TCP port 620
firewall-cmd --permanent --zone=public --add-port=621/tcp  # --permanent: mentés, TCP port 621
firewall-cmd --permanent --zone=public --add-port=622/tcp  # --permanent: mentés, TCP port 622
firewall-cmd --permanent --zone=public --add-port=623/tcp  # --permanent: mentés, TCP port 623
firewall-cmd --permanent --zone=public --add-port=624/tcp  # --permanent: mentés, TCP port 624
firewall-cmd --permanent --zone=public --add-port=625/tcp  # --permanent: mentés, TCP port 625
firewall-cmd --permanent --zone=public --add-port=626/tcp  # --permanent: mentés, TCP port 626
firewall-cmd --permanent --zone=public --add-port=627/tcp  # --permanent: mentés, TCP port 627
firewall-cmd --permanent --zone=public --add-port=628/tcp  # --permanent: mentés, TCP port 628
firewall-cmd --permanent --zone=public --add-port=629/tcp  # --permanent: mentés, TCP port 629
firewall-cmd --permanent --zone=public --add-port=630/tcp  # --permanent: mentés, TCP port 630
firewall-cmd --permanent --zone=public --add-port=631/tcp  # --permanent: mentés, TCP port 631
firewall-cmd --permanent --zone=public --add-port=632/tcp  # --permanent: mentés, TCP port 632
firewall-cmd --permanent --zone=public --add-port=633/tcp  # --permanent: mentés, TCP port 633
firewall-cmd --permanent --zone=public --add-port=634/tcp  # --permanent: mentés, TCP port 634
firewall-cmd --permanent --zone=public --add-port=635/tcp  # --permanent: mentés, TCP port 635
firewall-cmd --permanent --zone=public --add-port=636/tcp  # --permanent: mentés, TCP port 636
firewall-cmd --permanent --zone=public --add-port=637/tcp  # --permanent: mentés, TCP port 637
firewall-cmd --permanent --zone=public --add-port=638/tcp  # --permanent: mentés, TCP port 638
firewall-cmd --permanent --zone=public --add-port=639/tcp  # --permanent: mentés, TCP port 639
firewall-cmd --permanent --zone=public --add-port=640/tcp  # --permanent: mentés, TCP port 640
firewall-cmd --permanent --zone=public --add-port=641/tcp  # --permanent: mentés, TCP port 641
firewall-cmd --permanent --zone=public --add-port=642/tcp  # --permanent: mentés, TCP port 642
firewall-cmd --permanent --zone=public --add-port=643/tcp  # --permanent: mentés, TCP port 643
firewall-cmd --permanent --zone=public --add-port=644/tcp  # --permanent: mentés, TCP port 644
firewall-cmd --permanent --zone=public --add-port=645/tcp  # --permanent: mentés, TCP port 645
firewall-cmd --permanent --zone=public --add-port=646/tcp  # --permanent: mentés, TCP port 646
firewall-cmd --permanent --zone=public --add-port=647/tcp  # --permanent: mentés, TCP port 647
firewall-cmd --permanent --zone=public --add-port=648/tcp  # --permanent: mentés, TCP port 648
firewall-cmd --permanent --zone=public --add-port=649/tcp  # --permanent: mentés, TCP port 649
firewall-cmd --permanent --zone=public --add-port=650/tcp  # --permanent: mentés, TCP port 650
firewall-cmd --permanent --zone=public --add-port=651/tcp  # --permanent: mentés, TCP port 651
firewall-cmd --permanent --zone=public --add-port=652/tcp  # --permanent: mentés, TCP port 652
firewall-cmd --permanent --zone=public --add-port=653/tcp  # --permanent: mentés, TCP port 653
firewall-cmd --permanent --zone=public --add-port=654/tcp  # --permanent: mentés, TCP port 654
firewall-cmd --permanent --zone=public --add-port=655/tcp  # --permanent: mentés, TCP port 655
firewall-cmd --permanent --zone=public --add-port=656/tcp  # --permanent: mentés, TCP port 656
firewall-cmd --permanent --zone=public --add-port=657/tcp  # --permanent: mentés, TCP port 657
firewall-cmd --permanent --zone=public --add-port=658/tcp  # --permanent: mentés, TCP port 658
firewall-cmd --permanent --zone=public --add-port=659/tcp  # --permanent: mentés, TCP port 659
firewall-cmd --permanent --zone=public --add-port=660/tcp  # --permanent: mentés, TCP port 660
firewall-cmd --permanent --zone=public --add-port=661/tcp  # --permanent: mentés, TCP port 661
firewall-cmd --permanent --zone=public --add-port=662/tcp  # --permanent: mentés, TCP port 662
firewall-cmd --permanent --zone=public --add-port=663/tcp  # --permanent: mentés, TCP port 663
firewall-cmd --permanent --zone=public --add-port=664/tcp  # --permanent: mentés, TCP port 664
firewall-cmd --permanent --zone=public --add-port=665/tcp  # --permanent: mentés, TCP port 665
firewall-cmd --permanent --zone=public --add-port=666/tcp  # --permanent: mentés, TCP port 666
firewall-cmd --permanent --zone=public --add-port=667/tcp  # --permanent: mentés, TCP port 667
firewall-cmd --permanent --zone=public --add-port=668/tcp  # --permanent: mentés, TCP port 668
firewall-cmd --permanent --zone=public --add-port=669/tcp  # --permanent: mentés, TCP port 669
firewall-cmd --permanent --zone=public --add-port=670/tcp  # --permanent: mentés, TCP port 670
firewall-cmd --permanent --zone=public --add-port=671/tcp  # --permanent: mentés, TCP port 671
firewall-cmd --permanent --zone=public --add-port=672/tcp  # --permanent: mentés, TCP port 672
firewall-cmd --permanent --zone=public --add-port=673/tcp  # --permanent: mentés, TCP port 673
firewall-cmd --permanent --zone=public --add-port=674/tcp  # --permanent: mentés, TCP port 674
firewall-cmd --permanent --zone=public --add-port=675/tcp  # --permanent: mentés, TCP port 675
firewall-cmd --permanent --zone=public --add-port=676/tcp  # --permanent: mentés, TCP port 676
firewall-cmd --permanent --zone=public --add-port=677/tcp  # --permanent: mentés, TCP port 677
firewall-cmd --permanent --zone=public --add-port=678/tcp  # --permanent: mentés, TCP port 678
firewall-cmd --permanent --zone=public --add-port=679/tcp  # --permanent: mentés, TCP port 679
firewall-cmd --permanent --zone=public --add-port=680/tcp  # --permanent: mentés, TCP port 680
firewall-cmd --permanent --zone=public --add-port=681/tcp  # --permanent: mentés, TCP port 681
firewall-cmd --permanent --zone=public --add-port=682/tcp  # --permanent: mentés, TCP port 682
firewall-cmd --permanent --zone=public --add-port=683/tcp  # --permanent: mentés, TCP port 683
firewall-cmd --permanent --zone=public --add-port=684/tcp  # --permanent: mentés, TCP port 684
firewall-cmd --permanent --zone=public --add-port=685/tcp  # --permanent: mentés, TCP port 685
firewall-cmd --permanent --zone=public --add-port=686/tcp  # --permanent: mentés, TCP port 686
firewall-cmd --permanent --zone=public --add-port=687/tcp  # --permanent: mentés, TCP port 687
firewall-cmd --permanent --zone=public --add-port=688/tcp  # --permanent: mentés, TCP port 688
firewall-cmd --permanent --zone=public --add-port=689/tcp  # --permanent: mentés, TCP port 689
firewall-cmd --permanent --zone=public --add-port=690/tcp  # --permanent: mentés, TCP port 690
firewall-cmd --permanent --zone=public --add-port=691/tcp  # --permanent: mentés, TCP port 691
firewall-cmd --permanent --zone=public --add-port=692/tcp  # --permanent: mentés, TCP port 692
firewall-cmd --permanent --zone=public --add-port=693/tcp  # --permanent: mentés, TCP port 693
firewall-cmd --permanent --zone=public --add-port=694/tcp  # --permanent: mentés, TCP port 694
firewall-cmd --permanent --zone=public --add-port=695/tcp  # --permanent: mentés, TCP port 695
firewall-cmd --permanent --zone=public --add-port=696/tcp  # --permanent: mentés, TCP port 696
firewall-cmd --permanent --zone=public --add-port=697/tcp  # --permanent: mentés, TCP port 697
firewall-cmd --permanent --zone=public --add-port=698/tcp  # --permanent: mentés, TCP port 698
firewall-cmd --permanent --zone=public --add-port=699/tcp  # --permanent: mentés, TCP port 699
firewall-cmd --permanent --zone=public --add-port=700/tcp  # --permanent: mentés, TCP port 700
firewall-cmd --permanent --zone=public --add-port=701/tcp  # --permanent: mentés, TCP port 701
firewall-cmd --permanent --zone=public --add-port=702/tcp  # --permanent: mentés, TCP port 702
firewall-cmd --permanent --zone=public --add-port=703/tcp  # --permanent: mentés, TCP port 703
firewall-cmd --permanent --zone=public --add-port=704/tcp  # --permanent: mentés, TCP port 704
firewall-cmd --permanent --zone=public --add-port=705/tcp  # --permanent: mentés, TCP port 705
firewall-cmd --permanent --zone=public --add-port=706/tcp  # --permanent: mentés, TCP port 706
firewall-cmd --permanent --zone=public --add-port=707/tcp  # --permanent: mentés, TCP port 707
firewall-cmd --permanent --zone=public --add-port=708/tcp  # --permanent: mentés, TCP port 708
firewall-cmd --permanent --zone=public --add-port=709/tcp  # --permanent: mentés, TCP port 709
firewall-cmd --permanent --zone=public --add-port=710/tcp  # --permanent: mentés, TCP port 710
firewall-cmd --permanent --zone=public --add-port=711/tcp  # --permanent: mentés, TCP port 711
firewall-cmd --permanent --zone=public --add-port=712/tcp  # --permanent: mentés, TCP port 712
firewall-cmd --permanent --zone=public --add-port=713/tcp  # --permanent: mentés, TCP port 713
firewall-cmd --permanent --zone=public --add-port=714/tcp  # --permanent: mentés, TCP port 714
firewall-cmd --permanent --zone=public --add-port=715/tcp  # --permanent: mentés, TCP port 715
firewall-cmd --permanent --zone=public --add-port=716/tcp  # --permanent: mentés, TCP port 716
firewall-cmd --permanent --zone=public --add-port=717/tcp  # --permanent: mentés, TCP port 717
firewall-cmd --permanent --zone=public --add-port=718/tcp  # --permanent: mentés, TCP port 718
firewall-cmd --permanent --zone=public --add-port=719/tcp  # --permanent: mentés, TCP port 719
firewall-cmd --permanent --zone=public --add-port=720/tcp  # --permanent: mentés, TCP port 720
firewall-cmd --permanent --zone=public --add-port=721/tcp  # --permanent: mentés, TCP port 721
firewall-cmd --permanent --zone=public --add-port=722/tcp  # --permanent: mentés, TCP port 722
firewall-cmd --permanent --zone=public --add-port=723/tcp  # --permanent: mentés, TCP port 723
firewall-cmd --permanent --zone=public --add-port=724/tcp  # --permanent: mentés, TCP port 724
firewall-cmd --permanent --zone=public --add-port=725/tcp  # --permanent: mentés, TCP port 725
firewall-cmd --permanent --zone=public --add-port=726/tcp  # --permanent: mentés, TCP port 726
firewall-cmd --permanent --zone=public --add-port=727/tcp  # --permanent: mentés, TCP port 727
firewall-cmd --permanent --zone=public --add-port=728/tcp  # --permanent: mentés, TCP port 728
firewall-cmd --permanent --zone=public --add-port=729/tcp  # --permanent: mentés, TCP port 729
firewall-cmd --permanent --zone=public --add-port=730/tcp  # --permanent: mentés, TCP port 730
firewall-cmd --permanent --zone=public --add-port=731/tcp  # --permanent: mentés, TCP port 731
firewall-cmd --permanent --zone=public --add-port=732/tcp  # --permanent: mentés, TCP port 732
firewall-cmd --permanent --zone=public --add-port=733/tcp  # --permanent: mentés, TCP port 733
firewall-cmd --permanent --zone=public --add-port=734/tcp  # --permanent: mentés, TCP port 734
firewall-cmd --permanent --zone=public --add-port=735/tcp  # --permanent: mentés, TCP port 735
firewall-cmd --permanent --zone=public --add-port=736/tcp  # --permanent: mentés, TCP port 736
firewall-cmd --permanent --zone=public --add-port=737/tcp  # --permanent: mentés, TCP port 737
firewall-cmd --permanent --zone=public --add-port=738/tcp  # --permanent: mentés, TCP port 738
firewall-cmd --permanent --zone=public --add-port=739/tcp  # --permanent: mentés, TCP port 739
firewall-cmd --permanent --zone=public --add-port=740/tcp  # --permanent: mentés, TCP port 740
firewall-cmd --permanent --zone=public --add-port=741/tcp  # --permanent: mentés, TCP port 741
firewall-cmd --permanent --zone=public --add-port=742/tcp  # --permanent: mentés, TCP port 742
firewall-cmd --permanent --zone=public --add-port=743/tcp  # --permanent: mentés, TCP port 743
firewall-cmd --permanent --zone=public --add-port=744/tcp  # --permanent: mentés, TCP port 744
firewall-cmd --permanent --zone=public --add-port=745/tcp  # --permanent: mentés, TCP port 745
firewall-cmd --permanent --zone=public --add-port=746/tcp  # --permanent: mentés, TCP port 746
firewall-cmd --permanent --zone=public --add-port=747/tcp  # --permanent: mentés, TCP port 747
firewall-cmd --permanent --zone=public --add-port=748/tcp  # --permanent: mentés, TCP port 748
firewall-cmd --permanent --zone=public --add-port=749/tcp  # --permanent: mentés, TCP port 749
firewall-cmd --permanent --zone=public --add-port=750/tcp  # --permanent: mentés, TCP port 750
firewall-cmd --permanent --zone=public --add-port=751/tcp  # --permanent: mentés, TCP port 751
firewall-cmd --permanent --zone=public --add-port=752/tcp  # --permanent: mentés, TCP port 752
firewall-cmd --permanent --zone=public --add-port=753/tcp  # --permanent: mentés, TCP port 753
firewall-cmd --permanent --zone=public --add-port=754/tcp  # --permanent: mentés, TCP port 754
firewall-cmd --permanent --zone=public --add-port=755/tcp  # --permanent: mentés, TCP port 755
firewall-cmd --permanent --zone=public --add-port=756/tcp  # --permanent: mentés, TCP port 756
firewall-cmd --permanent --zone=public --add-port=757/tcp  # --permanent: mentés, TCP port 757
firewall-cmd --permanent --zone=public --add-port=758/tcp  # --permanent: mentés, TCP port 758
firewall-cmd --permanent --zone=public --add-port=759/tcp  # --permanent: mentés, TCP port 759
firewall-cmd --permanent --zone=public --add-port=760/tcp  # --permanent: mentés, TCP port 760
firewall-cmd --permanent --zone=public --add-port=761/tcp  # --permanent: mentés, TCP port 761
firewall-cmd --permanent --zone=public --add-port=762/tcp  # --permanent: mentés, TCP port 762
firewall-cmd --permanent --zone=public --add-port=763/tcp  # --permanent: mentés, TCP port 763
firewall-cmd --permanent --zone=public --add-port=764/tcp  # --permanent: mentés, TCP port 764
firewall-cmd --permanent --zone=public --add-port=765/tcp  # --permanent: mentés, TCP port 765
firewall-cmd --permanent --zone=public --add-port=766/tcp  # --permanent: mentés, TCP port 766
firewall-cmd --permanent --zone=public --add-port=767/tcp  # --permanent: mentés, TCP port 767
firewall-cmd --permanent --zone=public --add-port=768/tcp  # --permanent: mentés, TCP port 768
firewall-cmd --permanent --zone=public --add-port=769/tcp  # --permanent: mentés, TCP port 769
firewall-cmd --permanent --zone=public --add-port=770/tcp  # --permanent: mentés, TCP port 770
firewall-cmd --permanent --zone=public --add-port=771/tcp  # --permanent: mentés, TCP port 771
firewall-cmd --permanent --zone=public --add-port=772/tcp  # --permanent: mentés, TCP port 772
firewall-cmd --permanent --zone=public --add-port=773/tcp  # --permanent: mentés, TCP port 773
firewall-cmd --permanent --zone=public --add-port=774/tcp  # --permanent: mentés, TCP port 774
firewall-cmd --permanent --zone=public --add-port=775/tcp  # --permanent: mentés, TCP port 775
firewall-cmd --permanent --zone=public --add-port=776/tcp  # --permanent: mentés, TCP port 776
firewall-cmd --permanent --zone=public --add-port=777/tcp  # --permanent: mentés, TCP port 777
firewall-cmd --permanent --zone=public --add-port=778/tcp  # --permanent: mentés, TCP port 778
firewall-cmd --permanent --zone=public --add-port=779/tcp  # --permanent: mentés, TCP port 779
firewall-cmd --permanent --zone=public --add-port=780/tcp  # --permanent: mentés, TCP port 780
firewall-cmd --permanent --zone=public --add-port=781/tcp  # --permanent: mentés, TCP port 781
firewall-cmd --permanent --zone=public --add-port=782/tcp  # --permanent: mentés, TCP port 782
firewall-cmd --permanent --zone=public --add-port=783/tcp  # --permanent: mentés, TCP port 783
firewall-cmd --permanent --zone=public --add-port=784/tcp  # --permanent: mentés, TCP port 784
firewall-cmd --permanent --zone=public --add-port=785/tcp  # --permanent: mentés, TCP port 785
firewall-cmd --permanent --zone=public --add-port=786/tcp  # --permanent: mentés, TCP port 786
firewall-cmd --permanent --zone=public --add-port=787/tcp  # --permanent: mentés, TCP port 787
firewall-cmd --permanent --zone=public --add-port=788/tcp  # --permanent: mentés, TCP port 788
firewall-cmd --permanent --zone=public --add-port=789/tcp  # --permanent: mentés, TCP port 789
firewall-cmd --permanent --zone=public --add-port=790/tcp  # --permanent: mentés, TCP port 790
firewall-cmd --permanent --zone=public --add-port=791/tcp  # --permanent: mentés, TCP port 791
firewall-cmd --permanent --zone=public --add-port=792/tcp  # --permanent: mentés, TCP port 792
firewall-cmd --permanent --zone=public --add-port=793/tcp  # --permanent: mentés, TCP port 793
firewall-cmd --permanent --zone=public --add-port=794/tcp  # --permanent: mentés, TCP port 794
firewall-cmd --permanent --zone=public --add-port=795/tcp  # --permanent: mentés, TCP port 795
firewall-cmd --permanent --zone=public --add-port=796/tcp  # --permanent: mentés, TCP port 796
firewall-cmd --permanent --zone=public --add-port=797/tcp  # --permanent: mentés, TCP port 797
firewall-cmd --permanent --zone=public --add-port=798/tcp  # --permanent: mentés, TCP port 798
firewall-cmd --permanent --zone=public --add-port=799/tcp  # --permanent: mentés, TCP port 799
firewall-cmd --permanent --zone=public --add-port=800/tcp  # --permanent: mentés, TCP port 800
firewall-cmd --permanent --zone=public --add-port=801/tcp  # --permanent: mentés, TCP port 801
firewall-cmd --permanent --zone=public --add-port=802/tcp  # --permanent: mentés, TCP port 802
firewall-cmd --permanent --zone=public --add-port=803/tcp  # --permanent: mentés, TCP port 803
firewall-cmd --permanent --zone=public --add-port=804/tcp  # --permanent: mentés, TCP port 804
firewall-cmd --permanent --zone=public --add-port=805/tcp  # --permanent: mentés, TCP port 805
firewall-cmd --permanent --zone=public --add-port=806/tcp  # --permanent: mentés, TCP port 806
firewall-cmd --permanent --zone=public --add-port=807/tcp  # --permanent: mentés, TCP port 807
firewall-cmd --permanent --zone=public --add-port=808/tcp  # --permanent: mentés, TCP port 808
firewall-cmd --permanent --zone=public --add-port=809/tcp  # --permanent: mentés, TCP port 809
firewall-cmd --permanent --zone=public --add-port=810/tcp  # --permanent: mentés, TCP port 810
firewall-cmd --permanent --zone=public --add-port=811/tcp  # --permanent: mentés, TCP port 811
firewall-cmd --permanent --zone=public --add-port=812/tcp  # --permanent: mentés, TCP port 812
firewall-cmd --permanent --zone=public --add-port=813/tcp  # --permanent: mentés, TCP port 813
firewall-cmd --permanent --zone=public --add-port=814/tcp  # --permanent: mentés, TCP port 814
firewall-cmd --permanent --zone=public --add-port=815/tcp  # --permanent: mentés, TCP port 815
firewall-cmd --permanent --zone=public --add-port=816/tcp  # --permanent: mentés, TCP port 816
firewall-cmd --permanent --zone=public --add-port=817/tcp  # --permanent: mentés, TCP port 817
firewall-cmd --permanent --zone=public --add-port=818/tcp  # --permanent: mentés, TCP port 818
firewall-cmd --permanent --zone=public --add-port=819/tcp  # --permanent: mentés, TCP port 819
firewall-cmd --permanent --zone=public --add-port=820/tcp  # --permanent: mentés, TCP port 820
firewall-cmd --permanent --zone=public --add-port=821/tcp  # --permanent: mentés, TCP port 821
firewall-cmd --permanent --zone=public --add-port=822/tcp  # --permanent: mentés, TCP port 822
firewall-cmd --permanent --zone=public --add-port=823/tcp  # --permanent: mentés, TCP port 823
firewall-cmd --permanent --zone=public --add-port=824/tcp  # --permanent: mentés, TCP port 824
firewall-cmd --permanent --zone=public --add-port=825/tcp  # --permanent: mentés, TCP port 825
firewall-cmd --permanent --zone=public --add-port=826/tcp  # --permanent: mentés, TCP port 826
firewall-cmd --permanent --zone=public --add-port=827/tcp  # --permanent: mentés, TCP port 827
firewall-cmd --permanent --zone=public --add-port=828/tcp  # --permanent: mentés, TCP port 828
firewall-cmd --permanent --zone=public --add-port=829/tcp  # --permanent: mentés, TCP port 829
firewall-cmd --permanent --zone=public --add-port=830/tcp  # --permanent: mentés, TCP port 830
firewall-cmd --permanent --zone=public --add-port=831/tcp  # --permanent: mentés, TCP port 831
firewall-cmd --permanent --zone=public --add-port=832/tcp  # --permanent: mentés, TCP port 832
firewall-cmd --permanent --zone=public --add-port=833/tcp  # --permanent: mentés, TCP port 833
firewall-cmd --permanent --zone=public --add-port=834/tcp  # --permanent: mentés, TCP port 834
firewall-cmd --permanent --zone=public --add-port=835/tcp  # --permanent: mentés, TCP port 835
firewall-cmd --permanent --zone=public --add-port=836/tcp  # --permanent: mentés, TCP port 836
firewall-cmd --permanent --zone=public --add-port=837/tcp  # --permanent: mentés, TCP port 837
firewall-cmd --permanent --zone=public --add-port=838/tcp  # --permanent: mentés, TCP port 838
firewall-cmd --permanent --zone=public --add-port=839/tcp  # --permanent: mentés, TCP port 839
firewall-cmd --permanent --zone=public --add-port=840/tcp  # --permanent: mentés, TCP port 840
firewall-cmd --permanent --zone=public --add-port=841/tcp  # --permanent: mentés, TCP port 841
firewall-cmd --permanent --zone=public --add-port=842/tcp  # --permanent: mentés, TCP port 842
firewall-cmd --permanent --zone=public --add-port=843/tcp  # --permanent: mentés, TCP port 843
firewall-cmd --permanent --zone=public --add-port=844/tcp  # --permanent: mentés, TCP port 844
firewall-cmd --permanent --zone=public --add-port=845/tcp  # --permanent: mentés, TCP port 845
firewall-cmd --permanent --zone=public --add-port=846/tcp  # --permanent: mentés, TCP port 846
firewall-cmd --permanent --zone=public --add-port=847/tcp  # --permanent: mentés, TCP port 847
firewall-cmd --permanent --zone=public --add-port=848/tcp  # --permanent: mentés, TCP port 848
firewall-cmd --permanent --zone=public --add-port=849/tcp  # --permanent: mentés, TCP port 849
firewall-cmd --permanent --zone=public --add-port=850/tcp  # --permanent: mentés, TCP port 850
firewall-cmd --permanent --zone=public --add-port=851/tcp  # --permanent: mentés, TCP port 851
firewall-cmd --permanent --zone=public --add-port=852/tcp  # --permanent: mentés, TCP port 852
firewall-cmd --permanent --zone=public --add-port=853/tcp  # --permanent: mentés, TCP port 853
firewall-cmd --permanent --zone=public --add-port=854/tcp  # --permanent: mentés, TCP port 854
firewall-cmd --permanent --zone=public --add-port=855/tcp  # --permanent: mentés, TCP port 855
firewall-cmd --permanent --zone=public --add-port=856/tcp  # --permanent: mentés, TCP port 856
firewall-cmd --permanent --zone=public --add-port=857/tcp  # --permanent: mentés, TCP port 857
firewall-cmd --permanent --zone=public --add-port=858/tcp  # --permanent: mentés, TCP port 858
firewall-cmd --permanent --zone=public --add-port=859/tcp  # --permanent: mentés, TCP port 859
firewall-cmd --permanent --zone=public --add-port=860/tcp  # --permanent: mentés, TCP port 860
firewall-cmd --permanent --zone=public --add-port=861/tcp  # --permanent: mentés, TCP port 861
firewall-cmd --permanent --zone=public --add-port=862/tcp  # --permanent: mentés, TCP port 862
firewall-cmd --permanent --zone=public --add-port=863/tcp  # --permanent: mentés, TCP port 863
firewall-cmd --permanent --zone=public --add-port=864/tcp  # --permanent: mentés, TCP port 864
firewall-cmd --permanent --zone=public --add-port=865/tcp  # --permanent: mentés, TCP port 865
firewall-cmd --permanent --zone=public --add-port=866/tcp  # --permanent: mentés, TCP port 866
firewall-cmd --permanent --zone=public --add-port=867/tcp  # --permanent: mentés, TCP port 867
firewall-cmd --permanent --zone=public --add-port=868/tcp  # --permanent: mentés, TCP port 868
firewall-cmd --permanent --zone=public --add-port=869/tcp  # --permanent: mentés, TCP port 869
firewall-cmd --permanent --zone=public --add-port=870/tcp  # --permanent: mentés, TCP port 870
firewall-cmd --permanent --zone=public --add-port=871/tcp  # --permanent: mentés, TCP port 871
firewall-cmd --permanent --zone=public --add-port=872/tcp  # --permanent: mentés, TCP port 872
firewall-cmd --permanent --zone=public --add-port=873/tcp  # --permanent: mentés, TCP port 873
firewall-cmd --permanent --zone=public --add-port=874/tcp  # --permanent: mentés, TCP port 874
firewall-cmd --permanent --zone=public --add-port=875/tcp  # --permanent: mentés, TCP port 875
firewall-cmd --permanent --zone=public --add-port=876/tcp  # --permanent: mentés, TCP port 876
firewall-cmd --permanent --zone=public --add-port=877/tcp  # --permanent: mentés, TCP port 877
firewall-cmd --permanent --zone=public --add-port=878/tcp  # --permanent: mentés, TCP port 878
firewall-cmd --permanent --zone=public --add-port=879/tcp  # --permanent: mentés, TCP port 879
firewall-cmd --permanent --zone=public --add-port=880/tcp  # --permanent: mentés, TCP port 880
firewall-cmd --permanent --zone=public --add-port=881/tcp  # --permanent: mentés, TCP port 881
firewall-cmd --permanent --zone=public --add-port=882/tcp  # --permanent: mentés, TCP port 882
firewall-cmd --permanent --zone=public --add-port=883/tcp  # --permanent: mentés, TCP port 883
firewall-cmd --permanent --zone=public --add-port=884/tcp  # --permanent: mentés, TCP port 884
firewall-cmd --permanent --zone=public --add-port=885/tcp  # --permanent: mentés, TCP port 885
firewall-cmd --permanent --zone=public --add-port=886/tcp  # --permanent: mentés, TCP port 886
firewall-cmd --permanent --zone=public --add-port=887/tcp  # --permanent: mentés, TCP port 887
firewall-cmd --permanent --zone=public --add-port=888/tcp  # --permanent: mentés, TCP port 888
firewall-cmd --permanent --zone=public --add-port=889/tcp  # --permanent: mentés, TCP port 889
firewall-cmd --permanent --zone=public --add-port=890/tcp  # --permanent: mentés, TCP port 890
firewall-cmd --permanent --zone=public --add-port=891/tcp  # --permanent: mentés, TCP port 891
firewall-cmd --permanent --zone=public --add-port=892/tcp  # --permanent: mentés, TCP port 892
firewall-cmd --permanent --zone=public --add-port=893/tcp  # --permanent: mentés, TCP port 893
firewall-cmd --permanent --zone=public --add-port=894/tcp  # --permanent: mentés, TCP port 894
firewall-cmd --permanent --zone=public --add-port=895/tcp  # --permanent: mentés, TCP port 895
firewall-cmd --permanent --zone=public --add-port=896/tcp  # --permanent: mentés, TCP port 896
firewall-cmd --permanent --zone=public --add-port=897/tcp  # --permanent: mentés, TCP port 897
firewall-cmd --permanent --zone=public --add-port=898/tcp  # --permanent: mentés, TCP port 898
firewall-cmd --permanent --zone=public --add-port=899/tcp  # --permanent: mentés, TCP port 899
firewall-cmd --permanent --zone=public --add-port=900/tcp  # --permanent: mentés, TCP port 900
firewall-cmd --permanent --zone=public --add-port=901/tcp  # --permanent: mentés, TCP port 901
firewall-cmd --permanent --zone=public --add-port=902/tcp  # --permanent: mentés, TCP port 902
firewall-cmd --permanent --zone=public --add-port=903/tcp  # --permanent: mentés, TCP port 903
firewall-cmd --permanent --zone=public --add-port=904/tcp  # --permanent: mentés, TCP port 904
firewall-cmd --permanent --zone=public --add-port=905/tcp  # --permanent: mentés, TCP port 905
firewall-cmd --permanent --zone=public --add-port=906/tcp  # --permanent: mentés, TCP port 906
firewall-cmd --permanent --zone=public --add-port=907/tcp  # --permanent: mentés, TCP port 907
firewall-cmd --permanent --zone=public --add-port=908/tcp  # --permanent: mentés, TCP port 908
firewall-cmd --permanent --zone=public --add-port=909/tcp  # --permanent: mentés, TCP port 909
firewall-cmd --permanent --zone=public --add-port=910/tcp  # --permanent: mentés, TCP port 910
firewall-cmd --permanent --zone=public --add-port=911/tcp  # --permanent: mentés, TCP port 911
firewall-cmd --permanent --zone=public --add-port=912/tcp  # --permanent: mentés, TCP port 912
firewall-cmd --permanent --zone=public --add-port=913/tcp  # --permanent: mentés, TCP port 913
firewall-cmd --permanent --zone=public --add-port=914/tcp  # --permanent: mentés, TCP port 914
firewall-cmd --permanent --zone=public --add-port=915/tcp  # --permanent: mentés, TCP port 915
firewall-cmd --permanent --zone=public --add-port=916/tcp  # --permanent: mentés, TCP port 916
firewall-cmd --permanent --zone=public --add-port=917/tcp  # --permanent: mentés, TCP port 917
firewall-cmd --permanent --zone=public --add-port=918/tcp  # --permanent: mentés, TCP port 918
firewall-cmd --permanent --zone=public --add-port=919/tcp  # --permanent: mentés, TCP port 919
firewall-cmd --permanent --zone=public --add-port=920/tcp  # --permanent: mentés, TCP port 920
firewall-cmd --permanent --zone=public --add-port=921/tcp  # --permanent: mentés, TCP port 921
firewall-cmd --permanent --zone=public --add-port=922/tcp  # --permanent: mentés, TCP port 922
firewall-cmd --permanent --zone=public --add-port=923/tcp  # --permanent: mentés, TCP port 923
firewall-cmd --permanent --zone=public --add-port=924/tcp  # --permanent: mentés, TCP port 924
firewall-cmd --permanent --zone=public --add-port=925/tcp  # --permanent: mentés, TCP port 925
firewall-cmd --permanent --zone=public --add-port=926/tcp  # --permanent: mentés, TCP port 926
firewall-cmd --permanent --zone=public --add-port=927/tcp  # --permanent: mentés, TCP port 927
firewall-cmd --permanent --zone=public --add-port=928/tcp  # --permanent: mentés, TCP port 928
firewall-cmd --permanent --zone=public --add-port=929/tcp  # --permanent: mentés, TCP port 929
firewall-cmd --permanent --zone=public --add-port=930/tcp  # --permanent: mentés, TCP port 930
firewall-cmd --permanent --zone=public --add-port=931/tcp  # --permanent: mentés, TCP port 931
firewall-cmd --permanent --zone=public --add-port=932/tcp  # --permanent: mentés, TCP port 932
firewall-cmd --permanent --zone=public --add-port=933/tcp  # --permanent: mentés, TCP port 933
firewall-cmd --permanent --zone=public --add-port=934/tcp  # --permanent: mentés, TCP port 934
firewall-cmd --permanent --zone=public --add-port=935/tcp  # --permanent: mentés, TCP port 935
firewall-cmd --permanent --zone=public --add-port=936/tcp  # --permanent: mentés, TCP port 936
firewall-cmd --permanent --zone=public --add-port=937/tcp  # --permanent: mentés, TCP port 937
firewall-cmd --permanent --zone=public --add-port=938/tcp  # --permanent: mentés, TCP port 938
firewall-cmd --permanent --zone=public --add-port=939/tcp  # --permanent: mentés, TCP port 939
firewall-cmd --permanent --zone=public --add-port=940/tcp  # --permanent: mentés, TCP port 940
firewall-cmd --permanent --zone=public --add-port=941/tcp  # --permanent: mentés, TCP port 941
firewall-cmd --permanent --zone=public --add-port=942/tcp  # --permanent: mentés, TCP port 942
firewall-cmd --permanent --zone=public --add-port=943/tcp  # --permanent: mentés, TCP port 943
firewall-cmd --permanent --zone=public --add-port=944/tcp  # --permanent: mentés, TCP port 944
firewall-cmd --permanent --zone=public --add-port=945/tcp  # --permanent: mentés, TCP port 945
firewall-cmd --permanent --zone=public --add-port=946/tcp  # --permanent: mentés, TCP port 946
firewall-cmd --permanent --zone=public --add-port=947/tcp  # --permanent: mentés, TCP port 947
firewall-cmd --permanent --zone=public --add-port=948/tcp  # --permanent: mentés, TCP port 948
firewall-cmd --permanent --zone=public --add-port=949/tcp  # --permanent: mentés, TCP port 949
firewall-cmd --permanent --zone=public --add-port=950/tcp  # --permanent: mentés, TCP port 950
firewall-cmd --permanent --zone=public --add-port=951/tcp  # --permanent: mentés, TCP port 951
firewall-cmd --permanent --zone=public --add-port=952/tcp  # --permanent: mentés, TCP port 952
firewall-cmd --permanent --zone=public --add-port=953/tcp  # --permanent: mentés, TCP port 953
firewall-cmd --permanent --zone=public --add-port=954/tcp  # --permanent: mentés, TCP port 954
firewall-cmd --permanent --zone=public --add-port=955/tcp  # --permanent: mentés, TCP port 955
firewall-cmd --permanent --zone=public --add-port=956/tcp  # --permanent: mentés, TCP port 956
firewall-cmd --permanent --zone=public --add-port=957/tcp  # --permanent: mentés, TCP port 957
firewall-cmd --permanent --zone=public --add-port=958/tcp  # --permanent: mentés, TCP port 958
firewall-cmd --permanent --zone=public --add-port=959/tcp  # --permanent: mentés, TCP port 959
firewall-cmd --permanent --zone=public --add-port=960/tcp  # --permanent: mentés, TCP port 960
firewall-cmd --permanent --zone=public --add-port=961/tcp  # --permanent: mentés, TCP port 961
firewall-cmd --permanent --zone=public --add-port=962/tcp  # --permanent: mentés, TCP port 962
firewall-cmd --permanent --zone=public --add-port=963/tcp  # --permanent: mentés, TCP port 963
firewall-cmd --permanent --zone=public --add-port=964/tcp  # --permanent: mentés, TCP port 964
firewall-cmd --permanent --zone=public --add-port=965/tcp  # --permanent: mentés, TCP port 965
firewall-cmd --permanent --zone=public --add-port=966/tcp  # --permanent: mentés, TCP port 966
firewall-cmd --permanent --zone=public --add-port=967/tcp  # --permanent: mentés, TCP port 967
firewall-cmd --permanent --zone=public --add-port=968/tcp  # --permanent: mentés, TCP port 968
firewall-cmd --permanent --zone=public --add-port=969/tcp  # --permanent: mentés, TCP port 969
firewall-cmd --permanent --zone=public --add-port=970/tcp  # --permanent: mentés, TCP port 970
firewall-cmd --permanent --zone=public --add-port=971/tcp  # --permanent: mentés, TCP port 971
firewall-cmd --permanent --zone=public --add-port=972/tcp  # --permanent: mentés, TCP port 972
firewall-cmd --permanent --zone=public --add-port=973/tcp  # --permanent: mentés, TCP port 973
firewall-cmd --permanent --zone=public --add-port=974/tcp  # --permanent: mentés, TCP port 974
firewall-cmd --permanent --zone=public --add-port=975/tcp  # --permanent: mentés, TCP port 975
firewall-cmd --permanent --zone=public --add-port=976/tcp  # --permanent: mentés, TCP port 976
firewall-cmd --permanent --zone=public --add-port=977/tcp  # --permanent: mentés, TCP port 977
firewall-cmd --permanent --zone=public --add-port=978/tcp  # --permanent: mentés, TCP port 978
firewall-cmd --permanent --zone=public --add-port=979/tcp  # --permanent: mentés, TCP port 979
firewall-cmd --permanent --zone=public --add-port=980/tcp  # --permanent: mentés, TCP port 980
firewall-cmd --permanent --zone=public --add-port=981/tcp  # --permanent: mentés, TCP port 981
firewall-cmd --permanent --zone=public --add-port=982/tcp  # --permanent: mentés, TCP port 982
firewall-cmd --permanent --zone=public --add-port=983/tcp  # --permanent: mentés, TCP port 983
firewall-cmd --permanent --zone=public --add-port=984/tcp  # --permanent: mentés, TCP port 984
firewall-cmd --permanent --zone=public --add-port=985/tcp  # --permanent: mentés, TCP port 985
firewall-cmd --permanent --zone=public --add-port=986/tcp  # --permanent: mentés, TCP port 986
firewall-cmd --permanent --zone=public --add-port=987/tcp  # --permanent: mentés, TCP port 987
firewall-cmd --permanent --zone=public --add-port=988/tcp  # --permanent: mentés, TCP port 988
firewall-cmd --permanent --zone=public --add-port=989/tcp  # --permanent: mentés, TCP port 989
firewall-cmd --permanent --zone=public --add-port=990/tcp  # --permanent: mentés, TCP port 990
firewall-cmd --permanent --zone=public --add-port=991/tcp  # --permanent: mentés, TCP port 991
firewall-cmd --permanent --zone=public --add-port=992/tcp  # --permanent: mentés, TCP port 992
firewall-cmd --permanent --zone=public --add-port=993/tcp  # --permanent: mentés, TCP port 993
firewall-cmd --permanent --zone=public --add-port=994/tcp  # --permanent: mentés, TCP port 994
firewall-cmd --permanent --zone=public --add-port=995/tcp  # --permanent: mentés, TCP port 995
firewall-cmd --permanent --zone=public --add-port=996/tcp  # --permanent: mentés, TCP port 996
firewall-cmd --permanent --zone=public --add-port=997/tcp  # --permanent: mentés, TCP port 997
firewall-cmd --permanent --zone=public --add-port=998/tcp  # --permanent: mentés, TCP port 998
firewall-cmd --permanent --zone=public --add-port=999/tcp  # --permanent: mentés, TCP port 999
firewall-cmd --permanent --zone=public --add-port=1000/tcp  # --permanent: mentés, TCP port 1000
firewall-cmd --permanent --zone=public --add-port=1001/tcp  # --permanent: mentés, TCP port 1001
firewall-cmd --permanent --zone=public --add-port=1002/tcp  # --permanent: mentés, TCP port 1002
firewall-cmd --permanent --zone=public --add-port=1003/tcp  # --permanent: mentés, TCP port 1003
firewall-cmd --permanent --zone=public --add-port=1004/tcp  # --permanent: mentés, TCP port 1004
firewall-cmd --permanent --zone=public --add-port=1005/tcp  # --permanent: mentés, TCP port 1005
firewall-cmd --permanent --zone=public --add-port=1006/tcp  # --permanent: mentés, TCP port 1006
firewall-cmd --permanent --zone=public --add-port=1007/tcp  # --permanent: mentés, TCP port 1007
firewall-cmd --permanent --zone=public --add-port=1008/tcp  # --permanent: mentés, TCP port 1008
firewall-cmd --permanent --zone=public --add-port=1009/tcp  # --permanent: mentés, TCP port 1009
firewall-cmd --permanent --zone=public --add-port=1010/tcp  # --permanent: mentés, TCP port 1010
firewall-cmd --permanent --zone=public --add-port=1011/tcp  # --permanent: mentés, TCP port 1011
firewall-cmd --permanent --zone=public --add-port=1012/tcp  # --permanent: mentés, TCP port 1012
firewall-cmd --permanent --zone=public --add-port=1013/tcp  # --permanent: mentés, TCP port 1013
firewall-cmd --permanent --zone=public --add-port=1014/tcp  # --permanent: mentés, TCP port 1014
firewall-cmd --permanent --zone=public --add-port=1015/tcp  # --permanent: mentés, TCP port 1015
firewall-cmd --permanent --zone=public --add-port=1016/tcp  # --permanent: mentés, TCP port 1016
firewall-cmd --permanent --zone=public --add-port=1017/tcp  # --permanent: mentés, TCP port 1017
firewall-cmd --permanent --zone=public --add-port=1018/tcp  # --permanent: mentés, TCP port 1018
firewall-cmd --permanent --zone=public --add-port=1019/tcp  # --permanent: mentés, TCP port 1019
firewall-cmd --permanent --zone=public --add-port=1020/tcp  # --permanent: mentés, TCP port 1020
firewall-cmd --permanent --zone=public --add-port=1021/tcp  # --permanent: mentés, TCP port 1021
firewall-cmd --permanent --zone=public --add-port=1022/tcp  # --permanent: mentés, TCP port 1022
firewall-cmd --permanent --zone=public --add-port=1023/tcp  # --permanent: mentés, TCP port 1023
firewall-cmd --permanent --zone=public --add-port=1024/tcp  # --permanent: mentés, TCP port 1024
```

### 17.3 nftables TCP port szabályok (1–1024)
```bash
nft add rule inet filter input tcp dport 1 accept  # tcp dport 1
nft add rule inet filter input tcp dport 2 accept  # tcp dport 2
nft add rule inet filter input tcp dport 3 accept  # tcp dport 3
nft add rule inet filter input tcp dport 4 accept  # tcp dport 4
nft add rule inet filter input tcp dport 5 accept  # tcp dport 5
nft add rule inet filter input tcp dport 6 accept  # tcp dport 6
nft add rule inet filter input tcp dport 7 accept  # tcp dport 7
nft add rule inet filter input tcp dport 8 accept  # tcp dport 8
nft add rule inet filter input tcp dport 9 accept  # tcp dport 9
nft add rule inet filter input tcp dport 10 accept  # tcp dport 10
nft add rule inet filter input tcp dport 11 accept  # tcp dport 11
nft add rule inet filter input tcp dport 12 accept  # tcp dport 12
nft add rule inet filter input tcp dport 13 accept  # tcp dport 13
nft add rule inet filter input tcp dport 14 accept  # tcp dport 14
nft add rule inet filter input tcp dport 15 accept  # tcp dport 15
nft add rule inet filter input tcp dport 16 accept  # tcp dport 16
nft add rule inet filter input tcp dport 17 accept  # tcp dport 17
nft add rule inet filter input tcp dport 18 accept  # tcp dport 18
nft add rule inet filter input tcp dport 19 accept  # tcp dport 19
nft add rule inet filter input tcp dport 20 accept  # tcp dport 20
nft add rule inet filter input tcp dport 21 accept  # tcp dport 21
nft add rule inet filter input tcp dport 22 accept  # tcp dport 22
nft add rule inet filter input tcp dport 23 accept  # tcp dport 23
nft add rule inet filter input tcp dport 24 accept  # tcp dport 24
nft add rule inet filter input tcp dport 25 accept  # tcp dport 25
nft add rule inet filter input tcp dport 26 accept  # tcp dport 26
nft add rule inet filter input tcp dport 27 accept  # tcp dport 27
nft add rule inet filter input tcp dport 28 accept  # tcp dport 28
nft add rule inet filter input tcp dport 29 accept  # tcp dport 29
nft add rule inet filter input tcp dport 30 accept  # tcp dport 30
nft add rule inet filter input tcp dport 31 accept  # tcp dport 31
nft add rule inet filter input tcp dport 32 accept  # tcp dport 32
nft add rule inet filter input tcp dport 33 accept  # tcp dport 33
nft add rule inet filter input tcp dport 34 accept  # tcp dport 34
nft add rule inet filter input tcp dport 35 accept  # tcp dport 35
nft add rule inet filter input tcp dport 36 accept  # tcp dport 36
nft add rule inet filter input tcp dport 37 accept  # tcp dport 37
nft add rule inet filter input tcp dport 38 accept  # tcp dport 38
nft add rule inet filter input tcp dport 39 accept  # tcp dport 39
nft add rule inet filter input tcp dport 40 accept  # tcp dport 40
nft add rule inet filter input tcp dport 41 accept  # tcp dport 41
nft add rule inet filter input tcp dport 42 accept  # tcp dport 42
nft add rule inet filter input tcp dport 43 accept  # tcp dport 43
nft add rule inet filter input tcp dport 44 accept  # tcp dport 44
nft add rule inet filter input tcp dport 45 accept  # tcp dport 45
nft add rule inet filter input tcp dport 46 accept  # tcp dport 46
nft add rule inet filter input tcp dport 47 accept  # tcp dport 47
nft add rule inet filter input tcp dport 48 accept  # tcp dport 48
nft add rule inet filter input tcp dport 49 accept  # tcp dport 49
nft add rule inet filter input tcp dport 50 accept  # tcp dport 50
nft add rule inet filter input tcp dport 51 accept  # tcp dport 51
nft add rule inet filter input tcp dport 52 accept  # tcp dport 52
nft add rule inet filter input tcp dport 53 accept  # tcp dport 53
nft add rule inet filter input tcp dport 54 accept  # tcp dport 54
nft add rule inet filter input tcp dport 55 accept  # tcp dport 55
nft add rule inet filter input tcp dport 56 accept  # tcp dport 56
nft add rule inet filter input tcp dport 57 accept  # tcp dport 57
nft add rule inet filter input tcp dport 58 accept  # tcp dport 58
nft add rule inet filter input tcp dport 59 accept  # tcp dport 59
nft add rule inet filter input tcp dport 60 accept  # tcp dport 60
nft add rule inet filter input tcp dport 61 accept  # tcp dport 61
nft add rule inet filter input tcp dport 62 accept  # tcp dport 62
nft add rule inet filter input tcp dport 63 accept  # tcp dport 63
nft add rule inet filter input tcp dport 64 accept  # tcp dport 64
nft add rule inet filter input tcp dport 65 accept  # tcp dport 65
nft add rule inet filter input tcp dport 66 accept  # tcp dport 66
nft add rule inet filter input tcp dport 67 accept  # tcp dport 67
nft add rule inet filter input tcp dport 68 accept  # tcp dport 68
nft add rule inet filter input tcp dport 69 accept  # tcp dport 69
nft add rule inet filter input tcp dport 70 accept  # tcp dport 70
nft add rule inet filter input tcp dport 71 accept  # tcp dport 71
nft add rule inet filter input tcp dport 72 accept  # tcp dport 72
nft add rule inet filter input tcp dport 73 accept  # tcp dport 73
nft add rule inet filter input tcp dport 74 accept  # tcp dport 74
nft add rule inet filter input tcp dport 75 accept  # tcp dport 75
nft add rule inet filter input tcp dport 76 accept  # tcp dport 76
nft add rule inet filter input tcp dport 77 accept  # tcp dport 77
nft add rule inet filter input tcp dport 78 accept  # tcp dport 78
nft add rule inet filter input tcp dport 79 accept  # tcp dport 79
nft add rule inet filter input tcp dport 80 accept  # tcp dport 80
nft add rule inet filter input tcp dport 81 accept  # tcp dport 81
nft add rule inet filter input tcp dport 82 accept  # tcp dport 82
nft add rule inet filter input tcp dport 83 accept  # tcp dport 83
nft add rule inet filter input tcp dport 84 accept  # tcp dport 84
nft add rule inet filter input tcp dport 85 accept  # tcp dport 85
nft add rule inet filter input tcp dport 86 accept  # tcp dport 86
nft add rule inet filter input tcp dport 87 accept  # tcp dport 87
nft add rule inet filter input tcp dport 88 accept  # tcp dport 88
nft add rule inet filter input tcp dport 89 accept  # tcp dport 89
nft add rule inet filter input tcp dport 90 accept  # tcp dport 90
nft add rule inet filter input tcp dport 91 accept  # tcp dport 91
nft add rule inet filter input tcp dport 92 accept  # tcp dport 92
nft add rule inet filter input tcp dport 93 accept  # tcp dport 93
nft add rule inet filter input tcp dport 94 accept  # tcp dport 94
nft add rule inet filter input tcp dport 95 accept  # tcp dport 95
nft add rule inet filter input tcp dport 96 accept  # tcp dport 96
nft add rule inet filter input tcp dport 97 accept  # tcp dport 97
nft add rule inet filter input tcp dport 98 accept  # tcp dport 98
nft add rule inet filter input tcp dport 99 accept  # tcp dport 99
nft add rule inet filter input tcp dport 100 accept  # tcp dport 100
nft add rule inet filter input tcp dport 101 accept  # tcp dport 101
nft add rule inet filter input tcp dport 102 accept  # tcp dport 102
nft add rule inet filter input tcp dport 103 accept  # tcp dport 103
nft add rule inet filter input tcp dport 104 accept  # tcp dport 104
nft add rule inet filter input tcp dport 105 accept  # tcp dport 105
nft add rule inet filter input tcp dport 106 accept  # tcp dport 106
nft add rule inet filter input tcp dport 107 accept  # tcp dport 107
nft add rule inet filter input tcp dport 108 accept  # tcp dport 108
nft add rule inet filter input tcp dport 109 accept  # tcp dport 109
nft add rule inet filter input tcp dport 110 accept  # tcp dport 110
nft add rule inet filter input tcp dport 111 accept  # tcp dport 111
nft add rule inet filter input tcp dport 112 accept  # tcp dport 112
nft add rule inet filter input tcp dport 113 accept  # tcp dport 113
nft add rule inet filter input tcp dport 114 accept  # tcp dport 114
nft add rule inet filter input tcp dport 115 accept  # tcp dport 115
nft add rule inet filter input tcp dport 116 accept  # tcp dport 116
nft add rule inet filter input tcp dport 117 accept  # tcp dport 117
nft add rule inet filter input tcp dport 118 accept  # tcp dport 118
nft add rule inet filter input tcp dport 119 accept  # tcp dport 119
nft add rule inet filter input tcp dport 120 accept  # tcp dport 120
nft add rule inet filter input tcp dport 121 accept  # tcp dport 121
nft add rule inet filter input tcp dport 122 accept  # tcp dport 122
nft add rule inet filter input tcp dport 123 accept  # tcp dport 123
nft add rule inet filter input tcp dport 124 accept  # tcp dport 124
nft add rule inet filter input tcp dport 125 accept  # tcp dport 125
nft add rule inet filter input tcp dport 126 accept  # tcp dport 126
nft add rule inet filter input tcp dport 127 accept  # tcp dport 127
nft add rule inet filter input tcp dport 128 accept  # tcp dport 128
nft add rule inet filter input tcp dport 129 accept  # tcp dport 129
nft add rule inet filter input tcp dport 130 accept  # tcp dport 130
nft add rule inet filter input tcp dport 131 accept  # tcp dport 131
nft add rule inet filter input tcp dport 132 accept  # tcp dport 132
nft add rule inet filter input tcp dport 133 accept  # tcp dport 133
nft add rule inet filter input tcp dport 134 accept  # tcp dport 134
nft add rule inet filter input tcp dport 135 accept  # tcp dport 135
nft add rule inet filter input tcp dport 136 accept  # tcp dport 136
nft add rule inet filter input tcp dport 137 accept  # tcp dport 137
nft add rule inet filter input tcp dport 138 accept  # tcp dport 138
nft add rule inet filter input tcp dport 139 accept  # tcp dport 139
nft add rule inet filter input tcp dport 140 accept  # tcp dport 140
nft add rule inet filter input tcp dport 141 accept  # tcp dport 141
nft add rule inet filter input tcp dport 142 accept  # tcp dport 142
nft add rule inet filter input tcp dport 143 accept  # tcp dport 143
nft add rule inet filter input tcp dport 144 accept  # tcp dport 144
nft add rule inet filter input tcp dport 145 accept  # tcp dport 145
nft add rule inet filter input tcp dport 146 accept  # tcp dport 146
nft add rule inet filter input tcp dport 147 accept  # tcp dport 147
nft add rule inet filter input tcp dport 148 accept  # tcp dport 148
nft add rule inet filter input tcp dport 149 accept  # tcp dport 149
nft add rule inet filter input tcp dport 150 accept  # tcp dport 150
nft add rule inet filter input tcp dport 151 accept  # tcp dport 151
nft add rule inet filter input tcp dport 152 accept  # tcp dport 152
nft add rule inet filter input tcp dport 153 accept  # tcp dport 153
nft add rule inet filter input tcp dport 154 accept  # tcp dport 154
nft add rule inet filter input tcp dport 155 accept  # tcp dport 155
nft add rule inet filter input tcp dport 156 accept  # tcp dport 156
nft add rule inet filter input tcp dport 157 accept  # tcp dport 157
nft add rule inet filter input tcp dport 158 accept  # tcp dport 158
nft add rule inet filter input tcp dport 159 accept  # tcp dport 159
nft add rule inet filter input tcp dport 160 accept  # tcp dport 160
nft add rule inet filter input tcp dport 161 accept  # tcp dport 161
nft add rule inet filter input tcp dport 162 accept  # tcp dport 162
nft add rule inet filter input tcp dport 163 accept  # tcp dport 163
nft add rule inet filter input tcp dport 164 accept  # tcp dport 164
nft add rule inet filter input tcp dport 165 accept  # tcp dport 165
nft add rule inet filter input tcp dport 166 accept  # tcp dport 166
nft add rule inet filter input tcp dport 167 accept  # tcp dport 167
nft add rule inet filter input tcp dport 168 accept  # tcp dport 168
nft add rule inet filter input tcp dport 169 accept  # tcp dport 169
nft add rule inet filter input tcp dport 170 accept  # tcp dport 170
nft add rule inet filter input tcp dport 171 accept  # tcp dport 171
nft add rule inet filter input tcp dport 172 accept  # tcp dport 172
nft add rule inet filter input tcp dport 173 accept  # tcp dport 173
nft add rule inet filter input tcp dport 174 accept  # tcp dport 174
nft add rule inet filter input tcp dport 175 accept  # tcp dport 175
nft add rule inet filter input tcp dport 176 accept  # tcp dport 176
nft add rule inet filter input tcp dport 177 accept  # tcp dport 177
nft add rule inet filter input tcp dport 178 accept  # tcp dport 178
nft add rule inet filter input tcp dport 179 accept  # tcp dport 179
nft add rule inet filter input tcp dport 180 accept  # tcp dport 180
nft add rule inet filter input tcp dport 181 accept  # tcp dport 181
nft add rule inet filter input tcp dport 182 accept  # tcp dport 182
nft add rule inet filter input tcp dport 183 accept  # tcp dport 183
nft add rule inet filter input tcp dport 184 accept  # tcp dport 184
nft add rule inet filter input tcp dport 185 accept  # tcp dport 185
nft add rule inet filter input tcp dport 186 accept  # tcp dport 186
nft add rule inet filter input tcp dport 187 accept  # tcp dport 187
nft add rule inet filter input tcp dport 188 accept  # tcp dport 188
nft add rule inet filter input tcp dport 189 accept  # tcp dport 189
nft add rule inet filter input tcp dport 190 accept  # tcp dport 190
nft add rule inet filter input tcp dport 191 accept  # tcp dport 191
nft add rule inet filter input tcp dport 192 accept  # tcp dport 192
nft add rule inet filter input tcp dport 193 accept  # tcp dport 193
nft add rule inet filter input tcp dport 194 accept  # tcp dport 194
nft add rule inet filter input tcp dport 195 accept  # tcp dport 195
nft add rule inet filter input tcp dport 196 accept  # tcp dport 196
nft add rule inet filter input tcp dport 197 accept  # tcp dport 197
nft add rule inet filter input tcp dport 198 accept  # tcp dport 198
nft add rule inet filter input tcp dport 199 accept  # tcp dport 199
nft add rule inet filter input tcp dport 200 accept  # tcp dport 200
nft add rule inet filter input tcp dport 201 accept  # tcp dport 201
nft add rule inet filter input tcp dport 202 accept  # tcp dport 202
nft add rule inet filter input tcp dport 203 accept  # tcp dport 203
nft add rule inet filter input tcp dport 204 accept  # tcp dport 204
nft add rule inet filter input tcp dport 205 accept  # tcp dport 205
nft add rule inet filter input tcp dport 206 accept  # tcp dport 206
nft add rule inet filter input tcp dport 207 accept  # tcp dport 207
nft add rule inet filter input tcp dport 208 accept  # tcp dport 208
nft add rule inet filter input tcp dport 209 accept  # tcp dport 209
nft add rule inet filter input tcp dport 210 accept  # tcp dport 210
nft add rule inet filter input tcp dport 211 accept  # tcp dport 211
nft add rule inet filter input tcp dport 212 accept  # tcp dport 212
nft add rule inet filter input tcp dport 213 accept  # tcp dport 213
nft add rule inet filter input tcp dport 214 accept  # tcp dport 214
nft add rule inet filter input tcp dport 215 accept  # tcp dport 215
nft add rule inet filter input tcp dport 216 accept  # tcp dport 216
nft add rule inet filter input tcp dport 217 accept  # tcp dport 217
nft add rule inet filter input tcp dport 218 accept  # tcp dport 218
nft add rule inet filter input tcp dport 219 accept  # tcp dport 219
nft add rule inet filter input tcp dport 220 accept  # tcp dport 220
nft add rule inet filter input tcp dport 221 accept  # tcp dport 221
nft add rule inet filter input tcp dport 222 accept  # tcp dport 222
nft add rule inet filter input tcp dport 223 accept  # tcp dport 223
nft add rule inet filter input tcp dport 224 accept  # tcp dport 224
nft add rule inet filter input tcp dport 225 accept  # tcp dport 225
nft add rule inet filter input tcp dport 226 accept  # tcp dport 226
nft add rule inet filter input tcp dport 227 accept  # tcp dport 227
nft add rule inet filter input tcp dport 228 accept  # tcp dport 228
nft add rule inet filter input tcp dport 229 accept  # tcp dport 229
nft add rule inet filter input tcp dport 230 accept  # tcp dport 230
nft add rule inet filter input tcp dport 231 accept  # tcp dport 231
nft add rule inet filter input tcp dport 232 accept  # tcp dport 232
nft add rule inet filter input tcp dport 233 accept  # tcp dport 233
nft add rule inet filter input tcp dport 234 accept  # tcp dport 234
nft add rule inet filter input tcp dport 235 accept  # tcp dport 235
nft add rule inet filter input tcp dport 236 accept  # tcp dport 236
nft add rule inet filter input tcp dport 237 accept  # tcp dport 237
nft add rule inet filter input tcp dport 238 accept  # tcp dport 238
nft add rule inet filter input tcp dport 239 accept  # tcp dport 239
nft add rule inet filter input tcp dport 240 accept  # tcp dport 240
nft add rule inet filter input tcp dport 241 accept  # tcp dport 241
nft add rule inet filter input tcp dport 242 accept  # tcp dport 242
nft add rule inet filter input tcp dport 243 accept  # tcp dport 243
nft add rule inet filter input tcp dport 244 accept  # tcp dport 244
nft add rule inet filter input tcp dport 245 accept  # tcp dport 245
nft add rule inet filter input tcp dport 246 accept  # tcp dport 246
nft add rule inet filter input tcp dport 247 accept  # tcp dport 247
nft add rule inet filter input tcp dport 248 accept  # tcp dport 248
nft add rule inet filter input tcp dport 249 accept  # tcp dport 249
nft add rule inet filter input tcp dport 250 accept  # tcp dport 250
nft add rule inet filter input tcp dport 251 accept  # tcp dport 251
nft add rule inet filter input tcp dport 252 accept  # tcp dport 252
nft add rule inet filter input tcp dport 253 accept  # tcp dport 253
nft add rule inet filter input tcp dport 254 accept  # tcp dport 254
nft add rule inet filter input tcp dport 255 accept  # tcp dport 255
nft add rule inet filter input tcp dport 256 accept  # tcp dport 256
nft add rule inet filter input tcp dport 257 accept  # tcp dport 257
nft add rule inet filter input tcp dport 258 accept  # tcp dport 258
nft add rule inet filter input tcp dport 259 accept  # tcp dport 259
nft add rule inet filter input tcp dport 260 accept  # tcp dport 260
nft add rule inet filter input tcp dport 261 accept  # tcp dport 261
nft add rule inet filter input tcp dport 262 accept  # tcp dport 262
nft add rule inet filter input tcp dport 263 accept  # tcp dport 263
nft add rule inet filter input tcp dport 264 accept  # tcp dport 264
nft add rule inet filter input tcp dport 265 accept  # tcp dport 265
nft add rule inet filter input tcp dport 266 accept  # tcp dport 266
nft add rule inet filter input tcp dport 267 accept  # tcp dport 267
nft add rule inet filter input tcp dport 268 accept  # tcp dport 268
nft add rule inet filter input tcp dport 269 accept  # tcp dport 269
nft add rule inet filter input tcp dport 270 accept  # tcp dport 270
nft add rule inet filter input tcp dport 271 accept  # tcp dport 271
nft add rule inet filter input tcp dport 272 accept  # tcp dport 272
nft add rule inet filter input tcp dport 273 accept  # tcp dport 273
nft add rule inet filter input tcp dport 274 accept  # tcp dport 274
nft add rule inet filter input tcp dport 275 accept  # tcp dport 275
nft add rule inet filter input tcp dport 276 accept  # tcp dport 276
nft add rule inet filter input tcp dport 277 accept  # tcp dport 277
nft add rule inet filter input tcp dport 278 accept  # tcp dport 278
nft add rule inet filter input tcp dport 279 accept  # tcp dport 279
nft add rule inet filter input tcp dport 280 accept  # tcp dport 280
nft add rule inet filter input tcp dport 281 accept  # tcp dport 281
nft add rule inet filter input tcp dport 282 accept  # tcp dport 282
nft add rule inet filter input tcp dport 283 accept  # tcp dport 283
nft add rule inet filter input tcp dport 284 accept  # tcp dport 284
nft add rule inet filter input tcp dport 285 accept  # tcp dport 285
nft add rule inet filter input tcp dport 286 accept  # tcp dport 286
nft add rule inet filter input tcp dport 287 accept  # tcp dport 287
nft add rule inet filter input tcp dport 288 accept  # tcp dport 288
nft add rule inet filter input tcp dport 289 accept  # tcp dport 289
nft add rule inet filter input tcp dport 290 accept  # tcp dport 290
nft add rule inet filter input tcp dport 291 accept  # tcp dport 291
nft add rule inet filter input tcp dport 292 accept  # tcp dport 292
nft add rule inet filter input tcp dport 293 accept  # tcp dport 293
nft add rule inet filter input tcp dport 294 accept  # tcp dport 294
nft add rule inet filter input tcp dport 295 accept  # tcp dport 295
nft add rule inet filter input tcp dport 296 accept  # tcp dport 296
nft add rule inet filter input tcp dport 297 accept  # tcp dport 297
nft add rule inet filter input tcp dport 298 accept  # tcp dport 298
nft add rule inet filter input tcp dport 299 accept  # tcp dport 299
nft add rule inet filter input tcp dport 300 accept  # tcp dport 300
nft add rule inet filter input tcp dport 301 accept  # tcp dport 301
nft add rule inet filter input tcp dport 302 accept  # tcp dport 302
nft add rule inet filter input tcp dport 303 accept  # tcp dport 303
nft add rule inet filter input tcp dport 304 accept  # tcp dport 304
nft add rule inet filter input tcp dport 305 accept  # tcp dport 305
nft add rule inet filter input tcp dport 306 accept  # tcp dport 306
nft add rule inet filter input tcp dport 307 accept  # tcp dport 307
nft add rule inet filter input tcp dport 308 accept  # tcp dport 308
nft add rule inet filter input tcp dport 309 accept  # tcp dport 309
nft add rule inet filter input tcp dport 310 accept  # tcp dport 310
nft add rule inet filter input tcp dport 311 accept  # tcp dport 311
nft add rule inet filter input tcp dport 312 accept  # tcp dport 312
nft add rule inet filter input tcp dport 313 accept  # tcp dport 313
nft add rule inet filter input tcp dport 314 accept  # tcp dport 314
nft add rule inet filter input tcp dport 315 accept  # tcp dport 315
nft add rule inet filter input tcp dport 316 accept  # tcp dport 316
nft add rule inet filter input tcp dport 317 accept  # tcp dport 317
nft add rule inet filter input tcp dport 318 accept  # tcp dport 318
nft add rule inet filter input tcp dport 319 accept  # tcp dport 319
nft add rule inet filter input tcp dport 320 accept  # tcp dport 320
nft add rule inet filter input tcp dport 321 accept  # tcp dport 321
nft add rule inet filter input tcp dport 322 accept  # tcp dport 322
nft add rule inet filter input tcp dport 323 accept  # tcp dport 323
nft add rule inet filter input tcp dport 324 accept  # tcp dport 324
nft add rule inet filter input tcp dport 325 accept  # tcp dport 325
nft add rule inet filter input tcp dport 326 accept  # tcp dport 326
nft add rule inet filter input tcp dport 327 accept  # tcp dport 327
nft add rule inet filter input tcp dport 328 accept  # tcp dport 328
nft add rule inet filter input tcp dport 329 accept  # tcp dport 329
nft add rule inet filter input tcp dport 330 accept  # tcp dport 330
nft add rule inet filter input tcp dport 331 accept  # tcp dport 331
nft add rule inet filter input tcp dport 332 accept  # tcp dport 332
nft add rule inet filter input tcp dport 333 accept  # tcp dport 333
nft add rule inet filter input tcp dport 334 accept  # tcp dport 334
nft add rule inet filter input tcp dport 335 accept  # tcp dport 335
nft add rule inet filter input tcp dport 336 accept  # tcp dport 336
nft add rule inet filter input tcp dport 337 accept  # tcp dport 337
nft add rule inet filter input tcp dport 338 accept  # tcp dport 338
nft add rule inet filter input tcp dport 339 accept  # tcp dport 339
nft add rule inet filter input tcp dport 340 accept  # tcp dport 340
nft add rule inet filter input tcp dport 341 accept  # tcp dport 341
nft add rule inet filter input tcp dport 342 accept  # tcp dport 342
nft add rule inet filter input tcp dport 343 accept  # tcp dport 343
nft add rule inet filter input tcp dport 344 accept  # tcp dport 344
nft add rule inet filter input tcp dport 345 accept  # tcp dport 345
nft add rule inet filter input tcp dport 346 accept  # tcp dport 346
nft add rule inet filter input tcp dport 347 accept  # tcp dport 347
nft add rule inet filter input tcp dport 348 accept  # tcp dport 348
nft add rule inet filter input tcp dport 349 accept  # tcp dport 349
nft add rule inet filter input tcp dport 350 accept  # tcp dport 350
nft add rule inet filter input tcp dport 351 accept  # tcp dport 351
nft add rule inet filter input tcp dport 352 accept  # tcp dport 352
nft add rule inet filter input tcp dport 353 accept  # tcp dport 353
nft add rule inet filter input tcp dport 354 accept  # tcp dport 354
nft add rule inet filter input tcp dport 355 accept  # tcp dport 355
nft add rule inet filter input tcp dport 356 accept  # tcp dport 356
nft add rule inet filter input tcp dport 357 accept  # tcp dport 357
nft add rule inet filter input tcp dport 358 accept  # tcp dport 358
nft add rule inet filter input tcp dport 359 accept  # tcp dport 359
nft add rule inet filter input tcp dport 360 accept  # tcp dport 360
nft add rule inet filter input tcp dport 361 accept  # tcp dport 361
nft add rule inet filter input tcp dport 362 accept  # tcp dport 362
nft add rule inet filter input tcp dport 363 accept  # tcp dport 363
nft add rule inet filter input tcp dport 364 accept  # tcp dport 364
nft add rule inet filter input tcp dport 365 accept  # tcp dport 365
nft add rule inet filter input tcp dport 366 accept  # tcp dport 366
nft add rule inet filter input tcp dport 367 accept  # tcp dport 367
nft add rule inet filter input tcp dport 368 accept  # tcp dport 368
nft add rule inet filter input tcp dport 369 accept  # tcp dport 369
nft add rule inet filter input tcp dport 370 accept  # tcp dport 370
nft add rule inet filter input tcp dport 371 accept  # tcp dport 371
nft add rule inet filter input tcp dport 372 accept  # tcp dport 372
nft add rule inet filter input tcp dport 373 accept  # tcp dport 373
nft add rule inet filter input tcp dport 374 accept  # tcp dport 374
nft add rule inet filter input tcp dport 375 accept  # tcp dport 375
nft add rule inet filter input tcp dport 376 accept  # tcp dport 376
nft add rule inet filter input tcp dport 377 accept  # tcp dport 377
nft add rule inet filter input tcp dport 378 accept  # tcp dport 378
nft add rule inet filter input tcp dport 379 accept  # tcp dport 379
nft add rule inet filter input tcp dport 380 accept  # tcp dport 380
nft add rule inet filter input tcp dport 381 accept  # tcp dport 381
nft add rule inet filter input tcp dport 382 accept  # tcp dport 382
nft add rule inet filter input tcp dport 383 accept  # tcp dport 383
nft add rule inet filter input tcp dport 384 accept  # tcp dport 384
nft add rule inet filter input tcp dport 385 accept  # tcp dport 385
nft add rule inet filter input tcp dport 386 accept  # tcp dport 386
nft add rule inet filter input tcp dport 387 accept  # tcp dport 387
nft add rule inet filter input tcp dport 388 accept  # tcp dport 388
nft add rule inet filter input tcp dport 389 accept  # tcp dport 389
nft add rule inet filter input tcp dport 390 accept  # tcp dport 390
nft add rule inet filter input tcp dport 391 accept  # tcp dport 391
nft add rule inet filter input tcp dport 392 accept  # tcp dport 392
nft add rule inet filter input tcp dport 393 accept  # tcp dport 393
nft add rule inet filter input tcp dport 394 accept  # tcp dport 394
nft add rule inet filter input tcp dport 395 accept  # tcp dport 395
nft add rule inet filter input tcp dport 396 accept  # tcp dport 396
nft add rule inet filter input tcp dport 397 accept  # tcp dport 397
nft add rule inet filter input tcp dport 398 accept  # tcp dport 398
nft add rule inet filter input tcp dport 399 accept  # tcp dport 399
nft add rule inet filter input tcp dport 400 accept  # tcp dport 400
nft add rule inet filter input tcp dport 401 accept  # tcp dport 401
nft add rule inet filter input tcp dport 402 accept  # tcp dport 402
nft add rule inet filter input tcp dport 403 accept  # tcp dport 403
nft add rule inet filter input tcp dport 404 accept  # tcp dport 404
nft add rule inet filter input tcp dport 405 accept  # tcp dport 405
nft add rule inet filter input tcp dport 406 accept  # tcp dport 406
nft add rule inet filter input tcp dport 407 accept  # tcp dport 407
nft add rule inet filter input tcp dport 408 accept  # tcp dport 408
nft add rule inet filter input tcp dport 409 accept  # tcp dport 409
nft add rule inet filter input tcp dport 410 accept  # tcp dport 410
nft add rule inet filter input tcp dport 411 accept  # tcp dport 411
nft add rule inet filter input tcp dport 412 accept  # tcp dport 412
nft add rule inet filter input tcp dport 413 accept  # tcp dport 413
nft add rule inet filter input tcp dport 414 accept  # tcp dport 414
nft add rule inet filter input tcp dport 415 accept  # tcp dport 415
nft add rule inet filter input tcp dport 416 accept  # tcp dport 416
nft add rule inet filter input tcp dport 417 accept  # tcp dport 417
nft add rule inet filter input tcp dport 418 accept  # tcp dport 418
nft add rule inet filter input tcp dport 419 accept  # tcp dport 419
nft add rule inet filter input tcp dport 420 accept  # tcp dport 420
nft add rule inet filter input tcp dport 421 accept  # tcp dport 421
nft add rule inet filter input tcp dport 422 accept  # tcp dport 422
nft add rule inet filter input tcp dport 423 accept  # tcp dport 423
nft add rule inet filter input tcp dport 424 accept  # tcp dport 424
nft add rule inet filter input tcp dport 425 accept  # tcp dport 425
nft add rule inet filter input tcp dport 426 accept  # tcp dport 426
nft add rule inet filter input tcp dport 427 accept  # tcp dport 427
nft add rule inet filter input tcp dport 428 accept  # tcp dport 428
nft add rule inet filter input tcp dport 429 accept  # tcp dport 429
nft add rule inet filter input tcp dport 430 accept  # tcp dport 430
nft add rule inet filter input tcp dport 431 accept  # tcp dport 431
nft add rule inet filter input tcp dport 432 accept  # tcp dport 432
nft add rule inet filter input tcp dport 433 accept  # tcp dport 433
nft add rule inet filter input tcp dport 434 accept  # tcp dport 434
nft add rule inet filter input tcp dport 435 accept  # tcp dport 435
nft add rule inet filter input tcp dport 436 accept  # tcp dport 436
nft add rule inet filter input tcp dport 437 accept  # tcp dport 437
nft add rule inet filter input tcp dport 438 accept  # tcp dport 438
nft add rule inet filter input tcp dport 439 accept  # tcp dport 439
nft add rule inet filter input tcp dport 440 accept  # tcp dport 440
nft add rule inet filter input tcp dport 441 accept  # tcp dport 441
nft add rule inet filter input tcp dport 442 accept  # tcp dport 442
nft add rule inet filter input tcp dport 443 accept  # tcp dport 443
nft add rule inet filter input tcp dport 444 accept  # tcp dport 444
nft add rule inet filter input tcp dport 445 accept  # tcp dport 445
nft add rule inet filter input tcp dport 446 accept  # tcp dport 446
nft add rule inet filter input tcp dport 447 accept  # tcp dport 447
nft add rule inet filter input tcp dport 448 accept  # tcp dport 448
nft add rule inet filter input tcp dport 449 accept  # tcp dport 449
nft add rule inet filter input tcp dport 450 accept  # tcp dport 450
nft add rule inet filter input tcp dport 451 accept  # tcp dport 451
nft add rule inet filter input tcp dport 452 accept  # tcp dport 452
nft add rule inet filter input tcp dport 453 accept  # tcp dport 453
nft add rule inet filter input tcp dport 454 accept  # tcp dport 454
nft add rule inet filter input tcp dport 455 accept  # tcp dport 455
nft add rule inet filter input tcp dport 456 accept  # tcp dport 456
nft add rule inet filter input tcp dport 457 accept  # tcp dport 457
nft add rule inet filter input tcp dport 458 accept  # tcp dport 458
nft add rule inet filter input tcp dport 459 accept  # tcp dport 459
nft add rule inet filter input tcp dport 460 accept  # tcp dport 460
nft add rule inet filter input tcp dport 461 accept  # tcp dport 461
nft add rule inet filter input tcp dport 462 accept  # tcp dport 462
nft add rule inet filter input tcp dport 463 accept  # tcp dport 463
nft add rule inet filter input tcp dport 464 accept  # tcp dport 464
nft add rule inet filter input tcp dport 465 accept  # tcp dport 465
nft add rule inet filter input tcp dport 466 accept  # tcp dport 466
nft add rule inet filter input tcp dport 467 accept  # tcp dport 467
nft add rule inet filter input tcp dport 468 accept  # tcp dport 468
nft add rule inet filter input tcp dport 469 accept  # tcp dport 469
nft add rule inet filter input tcp dport 470 accept  # tcp dport 470
nft add rule inet filter input tcp dport 471 accept  # tcp dport 471
nft add rule inet filter input tcp dport 472 accept  # tcp dport 472
nft add rule inet filter input tcp dport 473 accept  # tcp dport 473
nft add rule inet filter input tcp dport 474 accept  # tcp dport 474
nft add rule inet filter input tcp dport 475 accept  # tcp dport 475
nft add rule inet filter input tcp dport 476 accept  # tcp dport 476
nft add rule inet filter input tcp dport 477 accept  # tcp dport 477
nft add rule inet filter input tcp dport 478 accept  # tcp dport 478
nft add rule inet filter input tcp dport 479 accept  # tcp dport 479
nft add rule inet filter input tcp dport 480 accept  # tcp dport 480
nft add rule inet filter input tcp dport 481 accept  # tcp dport 481
nft add rule inet filter input tcp dport 482 accept  # tcp dport 482
nft add rule inet filter input tcp dport 483 accept  # tcp dport 483
nft add rule inet filter input tcp dport 484 accept  # tcp dport 484
nft add rule inet filter input tcp dport 485 accept  # tcp dport 485
nft add rule inet filter input tcp dport 486 accept  # tcp dport 486
nft add rule inet filter input tcp dport 487 accept  # tcp dport 487
nft add rule inet filter input tcp dport 488 accept  # tcp dport 488
nft add rule inet filter input tcp dport 489 accept  # tcp dport 489
nft add rule inet filter input tcp dport 490 accept  # tcp dport 490
nft add rule inet filter input tcp dport 491 accept  # tcp dport 491
nft add rule inet filter input tcp dport 492 accept  # tcp dport 492
nft add rule inet filter input tcp dport 493 accept  # tcp dport 493
nft add rule inet filter input tcp dport 494 accept  # tcp dport 494
nft add rule inet filter input tcp dport 495 accept  # tcp dport 495
nft add rule inet filter input tcp dport 496 accept  # tcp dport 496
nft add rule inet filter input tcp dport 497 accept  # tcp dport 497
nft add rule inet filter input tcp dport 498 accept  # tcp dport 498
nft add rule inet filter input tcp dport 499 accept  # tcp dport 499
nft add rule inet filter input tcp dport 500 accept  # tcp dport 500
nft add rule inet filter input tcp dport 501 accept  # tcp dport 501
nft add rule inet filter input tcp dport 502 accept  # tcp dport 502
nft add rule inet filter input tcp dport 503 accept  # tcp dport 503
nft add rule inet filter input tcp dport 504 accept  # tcp dport 504
nft add rule inet filter input tcp dport 505 accept  # tcp dport 505
nft add rule inet filter input tcp dport 506 accept  # tcp dport 506
nft add rule inet filter input tcp dport 507 accept  # tcp dport 507
nft add rule inet filter input tcp dport 508 accept  # tcp dport 508
nft add rule inet filter input tcp dport 509 accept  # tcp dport 509
nft add rule inet filter input tcp dport 510 accept  # tcp dport 510
nft add rule inet filter input tcp dport 511 accept  # tcp dport 511
nft add rule inet filter input tcp dport 512 accept  # tcp dport 512
nft add rule inet filter input tcp dport 513 accept  # tcp dport 513
nft add rule inet filter input tcp dport 514 accept  # tcp dport 514
nft add rule inet filter input tcp dport 515 accept  # tcp dport 515
nft add rule inet filter input tcp dport 516 accept  # tcp dport 516
nft add rule inet filter input tcp dport 517 accept  # tcp dport 517
nft add rule inet filter input tcp dport 518 accept  # tcp dport 518
nft add rule inet filter input tcp dport 519 accept  # tcp dport 519
nft add rule inet filter input tcp dport 520 accept  # tcp dport 520
nft add rule inet filter input tcp dport 521 accept  # tcp dport 521
nft add rule inet filter input tcp dport 522 accept  # tcp dport 522
nft add rule inet filter input tcp dport 523 accept  # tcp dport 523
nft add rule inet filter input tcp dport 524 accept  # tcp dport 524
nft add rule inet filter input tcp dport 525 accept  # tcp dport 525
nft add rule inet filter input tcp dport 526 accept  # tcp dport 526
nft add rule inet filter input tcp dport 527 accept  # tcp dport 527
nft add rule inet filter input tcp dport 528 accept  # tcp dport 528
nft add rule inet filter input tcp dport 529 accept  # tcp dport 529
nft add rule inet filter input tcp dport 530 accept  # tcp dport 530
nft add rule inet filter input tcp dport 531 accept  # tcp dport 531
nft add rule inet filter input tcp dport 532 accept  # tcp dport 532
nft add rule inet filter input tcp dport 533 accept  # tcp dport 533
nft add rule inet filter input tcp dport 534 accept  # tcp dport 534
nft add rule inet filter input tcp dport 535 accept  # tcp dport 535
nft add rule inet filter input tcp dport 536 accept  # tcp dport 536
nft add rule inet filter input tcp dport 537 accept  # tcp dport 537
nft add rule inet filter input tcp dport 538 accept  # tcp dport 538
nft add rule inet filter input tcp dport 539 accept  # tcp dport 539
nft add rule inet filter input tcp dport 540 accept  # tcp dport 540
nft add rule inet filter input tcp dport 541 accept  # tcp dport 541
nft add rule inet filter input tcp dport 542 accept  # tcp dport 542
nft add rule inet filter input tcp dport 543 accept  # tcp dport 543
nft add rule inet filter input tcp dport 544 accept  # tcp dport 544
nft add rule inet filter input tcp dport 545 accept  # tcp dport 545
nft add rule inet filter input tcp dport 546 accept  # tcp dport 546
nft add rule inet filter input tcp dport 547 accept  # tcp dport 547
nft add rule inet filter input tcp dport 548 accept  # tcp dport 548
nft add rule inet filter input tcp dport 549 accept  # tcp dport 549
nft add rule inet filter input tcp dport 550 accept  # tcp dport 550
nft add rule inet filter input tcp dport 551 accept  # tcp dport 551
nft add rule inet filter input tcp dport 552 accept  # tcp dport 552
nft add rule inet filter input tcp dport 553 accept  # tcp dport 553
nft add rule inet filter input tcp dport 554 accept  # tcp dport 554
nft add rule inet filter input tcp dport 555 accept  # tcp dport 555
nft add rule inet filter input tcp dport 556 accept  # tcp dport 556
nft add rule inet filter input tcp dport 557 accept  # tcp dport 557
nft add rule inet filter input tcp dport 558 accept  # tcp dport 558
nft add rule inet filter input tcp dport 559 accept  # tcp dport 559
nft add rule inet filter input tcp dport 560 accept  # tcp dport 560
nft add rule inet filter input tcp dport 561 accept  # tcp dport 561
nft add rule inet filter input tcp dport 562 accept  # tcp dport 562
nft add rule inet filter input tcp dport 563 accept  # tcp dport 563
nft add rule inet filter input tcp dport 564 accept  # tcp dport 564
nft add rule inet filter input tcp dport 565 accept  # tcp dport 565
nft add rule inet filter input tcp dport 566 accept  # tcp dport 566
nft add rule inet filter input tcp dport 567 accept  # tcp dport 567
nft add rule inet filter input tcp dport 568 accept  # tcp dport 568
nft add rule inet filter input tcp dport 569 accept  # tcp dport 569
nft add rule inet filter input tcp dport 570 accept  # tcp dport 570
nft add rule inet filter input tcp dport 571 accept  # tcp dport 571
nft add rule inet filter input tcp dport 572 accept  # tcp dport 572
nft add rule inet filter input tcp dport 573 accept  # tcp dport 573
nft add rule inet filter input tcp dport 574 accept  # tcp dport 574
nft add rule inet filter input tcp dport 575 accept  # tcp dport 575
nft add rule inet filter input tcp dport 576 accept  # tcp dport 576
nft add rule inet filter input tcp dport 577 accept  # tcp dport 577
nft add rule inet filter input tcp dport 578 accept  # tcp dport 578
nft add rule inet filter input tcp dport 579 accept  # tcp dport 579
nft add rule inet filter input tcp dport 580 accept  # tcp dport 580
nft add rule inet filter input tcp dport 581 accept  # tcp dport 581
nft add rule inet filter input tcp dport 582 accept  # tcp dport 582
nft add rule inet filter input tcp dport 583 accept  # tcp dport 583
nft add rule inet filter input tcp dport 584 accept  # tcp dport 584
nft add rule inet filter input tcp dport 585 accept  # tcp dport 585
nft add rule inet filter input tcp dport 586 accept  # tcp dport 586
nft add rule inet filter input tcp dport 587 accept  # tcp dport 587
nft add rule inet filter input tcp dport 588 accept  # tcp dport 588
nft add rule inet filter input tcp dport 589 accept  # tcp dport 589
nft add rule inet filter input tcp dport 590 accept  # tcp dport 590
nft add rule inet filter input tcp dport 591 accept  # tcp dport 591
nft add rule inet filter input tcp dport 592 accept  # tcp dport 592
nft add rule inet filter input tcp dport 593 accept  # tcp dport 593
nft add rule inet filter input tcp dport 594 accept  # tcp dport 594
nft add rule inet filter input tcp dport 595 accept  # tcp dport 595
nft add rule inet filter input tcp dport 596 accept  # tcp dport 596
nft add rule inet filter input tcp dport 597 accept  # tcp dport 597
nft add rule inet filter input tcp dport 598 accept  # tcp dport 598
nft add rule inet filter input tcp dport 599 accept  # tcp dport 599
nft add rule inet filter input tcp dport 600 accept  # tcp dport 600
nft add rule inet filter input tcp dport 601 accept  # tcp dport 601
nft add rule inet filter input tcp dport 602 accept  # tcp dport 602
nft add rule inet filter input tcp dport 603 accept  # tcp dport 603
nft add rule inet filter input tcp dport 604 accept  # tcp dport 604
nft add rule inet filter input tcp dport 605 accept  # tcp dport 605
nft add rule inet filter input tcp dport 606 accept  # tcp dport 606
nft add rule inet filter input tcp dport 607 accept  # tcp dport 607
nft add rule inet filter input tcp dport 608 accept  # tcp dport 608
nft add rule inet filter input tcp dport 609 accept  # tcp dport 609
nft add rule inet filter input tcp dport 610 accept  # tcp dport 610
nft add rule inet filter input tcp dport 611 accept  # tcp dport 611
nft add rule inet filter input tcp dport 612 accept  # tcp dport 612
nft add rule inet filter input tcp dport 613 accept  # tcp dport 613
nft add rule inet filter input tcp dport 614 accept  # tcp dport 614
nft add rule inet filter input tcp dport 615 accept  # tcp dport 615
nft add rule inet filter input tcp dport 616 accept  # tcp dport 616
nft add rule inet filter input tcp dport 617 accept  # tcp dport 617
nft add rule inet filter input tcp dport 618 accept  # tcp dport 618
nft add rule inet filter input tcp dport 619 accept  # tcp dport 619
nft add rule inet filter input tcp dport 620 accept  # tcp dport 620
nft add rule inet filter input tcp dport 621 accept  # tcp dport 621
nft add rule inet filter input tcp dport 622 accept  # tcp dport 622
nft add rule inet filter input tcp dport 623 accept  # tcp dport 623
nft add rule inet filter input tcp dport 624 accept  # tcp dport 624
nft add rule inet filter input tcp dport 625 accept  # tcp dport 625
nft add rule inet filter input tcp dport 626 accept  # tcp dport 626
nft add rule inet filter input tcp dport 627 accept  # tcp dport 627
nft add rule inet filter input tcp dport 628 accept  # tcp dport 628
nft add rule inet filter input tcp dport 629 accept  # tcp dport 629
nft add rule inet filter input tcp dport 630 accept  # tcp dport 630
nft add rule inet filter input tcp dport 631 accept  # tcp dport 631
nft add rule inet filter input tcp dport 632 accept  # tcp dport 632
nft add rule inet filter input tcp dport 633 accept  # tcp dport 633
nft add rule inet filter input tcp dport 634 accept  # tcp dport 634
nft add rule inet filter input tcp dport 635 accept  # tcp dport 635
nft add rule inet filter input tcp dport 636 accept  # tcp dport 636
nft add rule inet filter input tcp dport 637 accept  # tcp dport 637
nft add rule inet filter input tcp dport 638 accept  # tcp dport 638
nft add rule inet filter input tcp dport 639 accept  # tcp dport 639
nft add rule inet filter input tcp dport 640 accept  # tcp dport 640
nft add rule inet filter input tcp dport 641 accept  # tcp dport 641
nft add rule inet filter input tcp dport 642 accept  # tcp dport 642
nft add rule inet filter input tcp dport 643 accept  # tcp dport 643
nft add rule inet filter input tcp dport 644 accept  # tcp dport 644
nft add rule inet filter input tcp dport 645 accept  # tcp dport 645
nft add rule inet filter input tcp dport 646 accept  # tcp dport 646
nft add rule inet filter input tcp dport 647 accept  # tcp dport 647
nft add rule inet filter input tcp dport 648 accept  # tcp dport 648
nft add rule inet filter input tcp dport 649 accept  # tcp dport 649
nft add rule inet filter input tcp dport 650 accept  # tcp dport 650
nft add rule inet filter input tcp dport 651 accept  # tcp dport 651
nft add rule inet filter input tcp dport 652 accept  # tcp dport 652
nft add rule inet filter input tcp dport 653 accept  # tcp dport 653
nft add rule inet filter input tcp dport 654 accept  # tcp dport 654
nft add rule inet filter input tcp dport 655 accept  # tcp dport 655
nft add rule inet filter input tcp dport 656 accept  # tcp dport 656
nft add rule inet filter input tcp dport 657 accept  # tcp dport 657
nft add rule inet filter input tcp dport 658 accept  # tcp dport 658
nft add rule inet filter input tcp dport 659 accept  # tcp dport 659
nft add rule inet filter input tcp dport 660 accept  # tcp dport 660
nft add rule inet filter input tcp dport 661 accept  # tcp dport 661
nft add rule inet filter input tcp dport 662 accept  # tcp dport 662
nft add rule inet filter input tcp dport 663 accept  # tcp dport 663
nft add rule inet filter input tcp dport 664 accept  # tcp dport 664
nft add rule inet filter input tcp dport 665 accept  # tcp dport 665
nft add rule inet filter input tcp dport 666 accept  # tcp dport 666
nft add rule inet filter input tcp dport 667 accept  # tcp dport 667
nft add rule inet filter input tcp dport 668 accept  # tcp dport 668
nft add rule inet filter input tcp dport 669 accept  # tcp dport 669
nft add rule inet filter input tcp dport 670 accept  # tcp dport 670
nft add rule inet filter input tcp dport 671 accept  # tcp dport 671
nft add rule inet filter input tcp dport 672 accept  # tcp dport 672
nft add rule inet filter input tcp dport 673 accept  # tcp dport 673
nft add rule inet filter input tcp dport 674 accept  # tcp dport 674
nft add rule inet filter input tcp dport 675 accept  # tcp dport 675
nft add rule inet filter input tcp dport 676 accept  # tcp dport 676
nft add rule inet filter input tcp dport 677 accept  # tcp dport 677
nft add rule inet filter input tcp dport 678 accept  # tcp dport 678
nft add rule inet filter input tcp dport 679 accept  # tcp dport 679
nft add rule inet filter input tcp dport 680 accept  # tcp dport 680
nft add rule inet filter input tcp dport 681 accept  # tcp dport 681
nft add rule inet filter input tcp dport 682 accept  # tcp dport 682
nft add rule inet filter input tcp dport 683 accept  # tcp dport 683
nft add rule inet filter input tcp dport 684 accept  # tcp dport 684
nft add rule inet filter input tcp dport 685 accept  # tcp dport 685
nft add rule inet filter input tcp dport 686 accept  # tcp dport 686
nft add rule inet filter input tcp dport 687 accept  # tcp dport 687
nft add rule inet filter input tcp dport 688 accept  # tcp dport 688
nft add rule inet filter input tcp dport 689 accept  # tcp dport 689
nft add rule inet filter input tcp dport 690 accept  # tcp dport 690
nft add rule inet filter input tcp dport 691 accept  # tcp dport 691
nft add rule inet filter input tcp dport 692 accept  # tcp dport 692
nft add rule inet filter input tcp dport 693 accept  # tcp dport 693
nft add rule inet filter input tcp dport 694 accept  # tcp dport 694
nft add rule inet filter input tcp dport 695 accept  # tcp dport 695
nft add rule inet filter input tcp dport 696 accept  # tcp dport 696
nft add rule inet filter input tcp dport 697 accept  # tcp dport 697
nft add rule inet filter input tcp dport 698 accept  # tcp dport 698
nft add rule inet filter input tcp dport 699 accept  # tcp dport 699
nft add rule inet filter input tcp dport 700 accept  # tcp dport 700
nft add rule inet filter input tcp dport 701 accept  # tcp dport 701
nft add rule inet filter input tcp dport 702 accept  # tcp dport 702
nft add rule inet filter input tcp dport 703 accept  # tcp dport 703
nft add rule inet filter input tcp dport 704 accept  # tcp dport 704
nft add rule inet filter input tcp dport 705 accept  # tcp dport 705
nft add rule inet filter input tcp dport 706 accept  # tcp dport 706
nft add rule inet filter input tcp dport 707 accept  # tcp dport 707
nft add rule inet filter input tcp dport 708 accept  # tcp dport 708
nft add rule inet filter input tcp dport 709 accept  # tcp dport 709
nft add rule inet filter input tcp dport 710 accept  # tcp dport 710
nft add rule inet filter input tcp dport 711 accept  # tcp dport 711
nft add rule inet filter input tcp dport 712 accept  # tcp dport 712
nft add rule inet filter input tcp dport 713 accept  # tcp dport 713
nft add rule inet filter input tcp dport 714 accept  # tcp dport 714
nft add rule inet filter input tcp dport 715 accept  # tcp dport 715
nft add rule inet filter input tcp dport 716 accept  # tcp dport 716
nft add rule inet filter input tcp dport 717 accept  # tcp dport 717
nft add rule inet filter input tcp dport 718 accept  # tcp dport 718
nft add rule inet filter input tcp dport 719 accept  # tcp dport 719
nft add rule inet filter input tcp dport 720 accept  # tcp dport 720
nft add rule inet filter input tcp dport 721 accept  # tcp dport 721
nft add rule inet filter input tcp dport 722 accept  # tcp dport 722
nft add rule inet filter input tcp dport 723 accept  # tcp dport 723
nft add rule inet filter input tcp dport 724 accept  # tcp dport 724
nft add rule inet filter input tcp dport 725 accept  # tcp dport 725
nft add rule inet filter input tcp dport 726 accept  # tcp dport 726
nft add rule inet filter input tcp dport 727 accept  # tcp dport 727
nft add rule inet filter input tcp dport 728 accept  # tcp dport 728
nft add rule inet filter input tcp dport 729 accept  # tcp dport 729
nft add rule inet filter input tcp dport 730 accept  # tcp dport 730
nft add rule inet filter input tcp dport 731 accept  # tcp dport 731
nft add rule inet filter input tcp dport 732 accept  # tcp dport 732
nft add rule inet filter input tcp dport 733 accept  # tcp dport 733
nft add rule inet filter input tcp dport 734 accept  # tcp dport 734
nft add rule inet filter input tcp dport 735 accept  # tcp dport 735
nft add rule inet filter input tcp dport 736 accept  # tcp dport 736
nft add rule inet filter input tcp dport 737 accept  # tcp dport 737
nft add rule inet filter input tcp dport 738 accept  # tcp dport 738
nft add rule inet filter input tcp dport 739 accept  # tcp dport 739
nft add rule inet filter input tcp dport 740 accept  # tcp dport 740
nft add rule inet filter input tcp dport 741 accept  # tcp dport 741
nft add rule inet filter input tcp dport 742 accept  # tcp dport 742
nft add rule inet filter input tcp dport 743 accept  # tcp dport 743
nft add rule inet filter input tcp dport 744 accept  # tcp dport 744
nft add rule inet filter input tcp dport 745 accept  # tcp dport 745
nft add rule inet filter input tcp dport 746 accept  # tcp dport 746
nft add rule inet filter input tcp dport 747 accept  # tcp dport 747
nft add rule inet filter input tcp dport 748 accept  # tcp dport 748
nft add rule inet filter input tcp dport 749 accept  # tcp dport 749
nft add rule inet filter input tcp dport 750 accept  # tcp dport 750
nft add rule inet filter input tcp dport 751 accept  # tcp dport 751
nft add rule inet filter input tcp dport 752 accept  # tcp dport 752
nft add rule inet filter input tcp dport 753 accept  # tcp dport 753
nft add rule inet filter input tcp dport 754 accept  # tcp dport 754
nft add rule inet filter input tcp dport 755 accept  # tcp dport 755
nft add rule inet filter input tcp dport 756 accept  # tcp dport 756
nft add rule inet filter input tcp dport 757 accept  # tcp dport 757
nft add rule inet filter input tcp dport 758 accept  # tcp dport 758
nft add rule inet filter input tcp dport 759 accept  # tcp dport 759
nft add rule inet filter input tcp dport 760 accept  # tcp dport 760
nft add rule inet filter input tcp dport 761 accept  # tcp dport 761
nft add rule inet filter input tcp dport 762 accept  # tcp dport 762
nft add rule inet filter input tcp dport 763 accept  # tcp dport 763
nft add rule inet filter input tcp dport 764 accept  # tcp dport 764
nft add rule inet filter input tcp dport 765 accept  # tcp dport 765
nft add rule inet filter input tcp dport 766 accept  # tcp dport 766
nft add rule inet filter input tcp dport 767 accept  # tcp dport 767
nft add rule inet filter input tcp dport 768 accept  # tcp dport 768
nft add rule inet filter input tcp dport 769 accept  # tcp dport 769
nft add rule inet filter input tcp dport 770 accept  # tcp dport 770
nft add rule inet filter input tcp dport 771 accept  # tcp dport 771
nft add rule inet filter input tcp dport 772 accept  # tcp dport 772
nft add rule inet filter input tcp dport 773 accept  # tcp dport 773
nft add rule inet filter input tcp dport 774 accept  # tcp dport 774
nft add rule inet filter input tcp dport 775 accept  # tcp dport 775
nft add rule inet filter input tcp dport 776 accept  # tcp dport 776
nft add rule inet filter input tcp dport 777 accept  # tcp dport 777
nft add rule inet filter input tcp dport 778 accept  # tcp dport 778
nft add rule inet filter input tcp dport 779 accept  # tcp dport 779
nft add rule inet filter input tcp dport 780 accept  # tcp dport 780
nft add rule inet filter input tcp dport 781 accept  # tcp dport 781
nft add rule inet filter input tcp dport 782 accept  # tcp dport 782
nft add rule inet filter input tcp dport 783 accept  # tcp dport 783
nft add rule inet filter input tcp dport 784 accept  # tcp dport 784
nft add rule inet filter input tcp dport 785 accept  # tcp dport 785
nft add rule inet filter input tcp dport 786 accept  # tcp dport 786
nft add rule inet filter input tcp dport 787 accept  # tcp dport 787
nft add rule inet filter input tcp dport 788 accept  # tcp dport 788
nft add rule inet filter input tcp dport 789 accept  # tcp dport 789
nft add rule inet filter input tcp dport 790 accept  # tcp dport 790
nft add rule inet filter input tcp dport 791 accept  # tcp dport 791
nft add rule inet filter input tcp dport 792 accept  # tcp dport 792
nft add rule inet filter input tcp dport 793 accept  # tcp dport 793
nft add rule inet filter input tcp dport 794 accept  # tcp dport 794
nft add rule inet filter input tcp dport 795 accept  # tcp dport 795
nft add rule inet filter input tcp dport 796 accept  # tcp dport 796
nft add rule inet filter input tcp dport 797 accept  # tcp dport 797
nft add rule inet filter input tcp dport 798 accept  # tcp dport 798
nft add rule inet filter input tcp dport 799 accept  # tcp dport 799
nft add rule inet filter input tcp dport 800 accept  # tcp dport 800
nft add rule inet filter input tcp dport 801 accept  # tcp dport 801
nft add rule inet filter input tcp dport 802 accept  # tcp dport 802
nft add rule inet filter input tcp dport 803 accept  # tcp dport 803
nft add rule inet filter input tcp dport 804 accept  # tcp dport 804
nft add rule inet filter input tcp dport 805 accept  # tcp dport 805
nft add rule inet filter input tcp dport 806 accept  # tcp dport 806
nft add rule inet filter input tcp dport 807 accept  # tcp dport 807
nft add rule inet filter input tcp dport 808 accept  # tcp dport 808
nft add rule inet filter input tcp dport 809 accept  # tcp dport 809
nft add rule inet filter input tcp dport 810 accept  # tcp dport 810
nft add rule inet filter input tcp dport 811 accept  # tcp dport 811
nft add rule inet filter input tcp dport 812 accept  # tcp dport 812
nft add rule inet filter input tcp dport 813 accept  # tcp dport 813
nft add rule inet filter input tcp dport 814 accept  # tcp dport 814
nft add rule inet filter input tcp dport 815 accept  # tcp dport 815
nft add rule inet filter input tcp dport 816 accept  # tcp dport 816
nft add rule inet filter input tcp dport 817 accept  # tcp dport 817
nft add rule inet filter input tcp dport 818 accept  # tcp dport 818
nft add rule inet filter input tcp dport 819 accept  # tcp dport 819
nft add rule inet filter input tcp dport 820 accept  # tcp dport 820
nft add rule inet filter input tcp dport 821 accept  # tcp dport 821
nft add rule inet filter input tcp dport 822 accept  # tcp dport 822
nft add rule inet filter input tcp dport 823 accept  # tcp dport 823
nft add rule inet filter input tcp dport 824 accept  # tcp dport 824
nft add rule inet filter input tcp dport 825 accept  # tcp dport 825
nft add rule inet filter input tcp dport 826 accept  # tcp dport 826
nft add rule inet filter input tcp dport 827 accept  # tcp dport 827
nft add rule inet filter input tcp dport 828 accept  # tcp dport 828
nft add rule inet filter input tcp dport 829 accept  # tcp dport 829
nft add rule inet filter input tcp dport 830 accept  # tcp dport 830
nft add rule inet filter input tcp dport 831 accept  # tcp dport 831
nft add rule inet filter input tcp dport 832 accept  # tcp dport 832
nft add rule inet filter input tcp dport 833 accept  # tcp dport 833
nft add rule inet filter input tcp dport 834 accept  # tcp dport 834
nft add rule inet filter input tcp dport 835 accept  # tcp dport 835
nft add rule inet filter input tcp dport 836 accept  # tcp dport 836
nft add rule inet filter input tcp dport 837 accept  # tcp dport 837
nft add rule inet filter input tcp dport 838 accept  # tcp dport 838
nft add rule inet filter input tcp dport 839 accept  # tcp dport 839
nft add rule inet filter input tcp dport 840 accept  # tcp dport 840
nft add rule inet filter input tcp dport 841 accept  # tcp dport 841
nft add rule inet filter input tcp dport 842 accept  # tcp dport 842
nft add rule inet filter input tcp dport 843 accept  # tcp dport 843
nft add rule inet filter input tcp dport 844 accept  # tcp dport 844
nft add rule inet filter input tcp dport 845 accept  # tcp dport 845
nft add rule inet filter input tcp dport 846 accept  # tcp dport 846
nft add rule inet filter input tcp dport 847 accept  # tcp dport 847
nft add rule inet filter input tcp dport 848 accept  # tcp dport 848
nft add rule inet filter input tcp dport 849 accept  # tcp dport 849
nft add rule inet filter input tcp dport 850 accept  # tcp dport 850
nft add rule inet filter input tcp dport 851 accept  # tcp dport 851
nft add rule inet filter input tcp dport 852 accept  # tcp dport 852
nft add rule inet filter input tcp dport 853 accept  # tcp dport 853
nft add rule inet filter input tcp dport 854 accept  # tcp dport 854
nft add rule inet filter input tcp dport 855 accept  # tcp dport 855
nft add rule inet filter input tcp dport 856 accept  # tcp dport 856
nft add rule inet filter input tcp dport 857 accept  # tcp dport 857
nft add rule inet filter input tcp dport 858 accept  # tcp dport 858
nft add rule inet filter input tcp dport 859 accept  # tcp dport 859
nft add rule inet filter input tcp dport 860 accept  # tcp dport 860
nft add rule inet filter input tcp dport 861 accept  # tcp dport 861
nft add rule inet filter input tcp dport 862 accept  # tcp dport 862
nft add rule inet filter input tcp dport 863 accept  # tcp dport 863
nft add rule inet filter input tcp dport 864 accept  # tcp dport 864
nft add rule inet filter input tcp dport 865 accept  # tcp dport 865
nft add rule inet filter input tcp dport 866 accept  # tcp dport 866
nft add rule inet filter input tcp dport 867 accept  # tcp dport 867
nft add rule inet filter input tcp dport 868 accept  # tcp dport 868
nft add rule inet filter input tcp dport 869 accept  # tcp dport 869
nft add rule inet filter input tcp dport 870 accept  # tcp dport 870
nft add rule inet filter input tcp dport 871 accept  # tcp dport 871
nft add rule inet filter input tcp dport 872 accept  # tcp dport 872
nft add rule inet filter input tcp dport 873 accept  # tcp dport 873
nft add rule inet filter input tcp dport 874 accept  # tcp dport 874
nft add rule inet filter input tcp dport 875 accept  # tcp dport 875
nft add rule inet filter input tcp dport 876 accept  # tcp dport 876
nft add rule inet filter input tcp dport 877 accept  # tcp dport 877
nft add rule inet filter input tcp dport 878 accept  # tcp dport 878
nft add rule inet filter input tcp dport 879 accept  # tcp dport 879
nft add rule inet filter input tcp dport 880 accept  # tcp dport 880
nft add rule inet filter input tcp dport 881 accept  # tcp dport 881
nft add rule inet filter input tcp dport 882 accept  # tcp dport 882
nft add rule inet filter input tcp dport 883 accept  # tcp dport 883
nft add rule inet filter input tcp dport 884 accept  # tcp dport 884
nft add rule inet filter input tcp dport 885 accept  # tcp dport 885
nft add rule inet filter input tcp dport 886 accept  # tcp dport 886
nft add rule inet filter input tcp dport 887 accept  # tcp dport 887
nft add rule inet filter input tcp dport 888 accept  # tcp dport 888
nft add rule inet filter input tcp dport 889 accept  # tcp dport 889
nft add rule inet filter input tcp dport 890 accept  # tcp dport 890
nft add rule inet filter input tcp dport 891 accept  # tcp dport 891
nft add rule inet filter input tcp dport 892 accept  # tcp dport 892
nft add rule inet filter input tcp dport 893 accept  # tcp dport 893
nft add rule inet filter input tcp dport 894 accept  # tcp dport 894
nft add rule inet filter input tcp dport 895 accept  # tcp dport 895
nft add rule inet filter input tcp dport 896 accept  # tcp dport 896
nft add rule inet filter input tcp dport 897 accept  # tcp dport 897
nft add rule inet filter input tcp dport 898 accept  # tcp dport 898
nft add rule inet filter input tcp dport 899 accept  # tcp dport 899
nft add rule inet filter input tcp dport 900 accept  # tcp dport 900
nft add rule inet filter input tcp dport 901 accept  # tcp dport 901
nft add rule inet filter input tcp dport 902 accept  # tcp dport 902
nft add rule inet filter input tcp dport 903 accept  # tcp dport 903
nft add rule inet filter input tcp dport 904 accept  # tcp dport 904
nft add rule inet filter input tcp dport 905 accept  # tcp dport 905
nft add rule inet filter input tcp dport 906 accept  # tcp dport 906
nft add rule inet filter input tcp dport 907 accept  # tcp dport 907
nft add rule inet filter input tcp dport 908 accept  # tcp dport 908
nft add rule inet filter input tcp dport 909 accept  # tcp dport 909
nft add rule inet filter input tcp dport 910 accept  # tcp dport 910
nft add rule inet filter input tcp dport 911 accept  # tcp dport 911
nft add rule inet filter input tcp dport 912 accept  # tcp dport 912
nft add rule inet filter input tcp dport 913 accept  # tcp dport 913
nft add rule inet filter input tcp dport 914 accept  # tcp dport 914
nft add rule inet filter input tcp dport 915 accept  # tcp dport 915
nft add rule inet filter input tcp dport 916 accept  # tcp dport 916
nft add rule inet filter input tcp dport 917 accept  # tcp dport 917
nft add rule inet filter input tcp dport 918 accept  # tcp dport 918
nft add rule inet filter input tcp dport 919 accept  # tcp dport 919
nft add rule inet filter input tcp dport 920 accept  # tcp dport 920
nft add rule inet filter input tcp dport 921 accept  # tcp dport 921
nft add rule inet filter input tcp dport 922 accept  # tcp dport 922
nft add rule inet filter input tcp dport 923 accept  # tcp dport 923
nft add rule inet filter input tcp dport 924 accept  # tcp dport 924
nft add rule inet filter input tcp dport 925 accept  # tcp dport 925
nft add rule inet filter input tcp dport 926 accept  # tcp dport 926
nft add rule inet filter input tcp dport 927 accept  # tcp dport 927
nft add rule inet filter input tcp dport 928 accept  # tcp dport 928
nft add rule inet filter input tcp dport 929 accept  # tcp dport 929
nft add rule inet filter input tcp dport 930 accept  # tcp dport 930
nft add rule inet filter input tcp dport 931 accept  # tcp dport 931
nft add rule inet filter input tcp dport 932 accept  # tcp dport 932
nft add rule inet filter input tcp dport 933 accept  # tcp dport 933
nft add rule inet filter input tcp dport 934 accept  # tcp dport 934
nft add rule inet filter input tcp dport 935 accept  # tcp dport 935
nft add rule inet filter input tcp dport 936 accept  # tcp dport 936
nft add rule inet filter input tcp dport 937 accept  # tcp dport 937
nft add rule inet filter input tcp dport 938 accept  # tcp dport 938
nft add rule inet filter input tcp dport 939 accept  # tcp dport 939
nft add rule inet filter input tcp dport 940 accept  # tcp dport 940
nft add rule inet filter input tcp dport 941 accept  # tcp dport 941
nft add rule inet filter input tcp dport 942 accept  # tcp dport 942
nft add rule inet filter input tcp dport 943 accept  # tcp dport 943
nft add rule inet filter input tcp dport 944 accept  # tcp dport 944
nft add rule inet filter input tcp dport 945 accept  # tcp dport 945
nft add rule inet filter input tcp dport 946 accept  # tcp dport 946
nft add rule inet filter input tcp dport 947 accept  # tcp dport 947
nft add rule inet filter input tcp dport 948 accept  # tcp dport 948
nft add rule inet filter input tcp dport 949 accept  # tcp dport 949
nft add rule inet filter input tcp dport 950 accept  # tcp dport 950
nft add rule inet filter input tcp dport 951 accept  # tcp dport 951
nft add rule inet filter input tcp dport 952 accept  # tcp dport 952
nft add rule inet filter input tcp dport 953 accept  # tcp dport 953
nft add rule inet filter input tcp dport 954 accept  # tcp dport 954
nft add rule inet filter input tcp dport 955 accept  # tcp dport 955
nft add rule inet filter input tcp dport 956 accept  # tcp dport 956
nft add rule inet filter input tcp dport 957 accept  # tcp dport 957
nft add rule inet filter input tcp dport 958 accept  # tcp dport 958
nft add rule inet filter input tcp dport 959 accept  # tcp dport 959
nft add rule inet filter input tcp dport 960 accept  # tcp dport 960
nft add rule inet filter input tcp dport 961 accept  # tcp dport 961
nft add rule inet filter input tcp dport 962 accept  # tcp dport 962
nft add rule inet filter input tcp dport 963 accept  # tcp dport 963
nft add rule inet filter input tcp dport 964 accept  # tcp dport 964
nft add rule inet filter input tcp dport 965 accept  # tcp dport 965
nft add rule inet filter input tcp dport 966 accept  # tcp dport 966
nft add rule inet filter input tcp dport 967 accept  # tcp dport 967
nft add rule inet filter input tcp dport 968 accept  # tcp dport 968
nft add rule inet filter input tcp dport 969 accept  # tcp dport 969
nft add rule inet filter input tcp dport 970 accept  # tcp dport 970
nft add rule inet filter input tcp dport 971 accept  # tcp dport 971
nft add rule inet filter input tcp dport 972 accept  # tcp dport 972
nft add rule inet filter input tcp dport 973 accept  # tcp dport 973
nft add rule inet filter input tcp dport 974 accept  # tcp dport 974
nft add rule inet filter input tcp dport 975 accept  # tcp dport 975
nft add rule inet filter input tcp dport 976 accept  # tcp dport 976
nft add rule inet filter input tcp dport 977 accept  # tcp dport 977
nft add rule inet filter input tcp dport 978 accept  # tcp dport 978
nft add rule inet filter input tcp dport 979 accept  # tcp dport 979
nft add rule inet filter input tcp dport 980 accept  # tcp dport 980
nft add rule inet filter input tcp dport 981 accept  # tcp dport 981
nft add rule inet filter input tcp dport 982 accept  # tcp dport 982
nft add rule inet filter input tcp dport 983 accept  # tcp dport 983
nft add rule inet filter input tcp dport 984 accept  # tcp dport 984
nft add rule inet filter input tcp dport 985 accept  # tcp dport 985
nft add rule inet filter input tcp dport 986 accept  # tcp dport 986
nft add rule inet filter input tcp dport 987 accept  # tcp dport 987
nft add rule inet filter input tcp dport 988 accept  # tcp dport 988
nft add rule inet filter input tcp dport 989 accept  # tcp dport 989
nft add rule inet filter input tcp dport 990 accept  # tcp dport 990
nft add rule inet filter input tcp dport 991 accept  # tcp dport 991
nft add rule inet filter input tcp dport 992 accept  # tcp dport 992
nft add rule inet filter input tcp dport 993 accept  # tcp dport 993
nft add rule inet filter input tcp dport 994 accept  # tcp dport 994
nft add rule inet filter input tcp dport 995 accept  # tcp dport 995
nft add rule inet filter input tcp dport 996 accept  # tcp dport 996
nft add rule inet filter input tcp dport 997 accept  # tcp dport 997
nft add rule inet filter input tcp dport 998 accept  # tcp dport 998
nft add rule inet filter input tcp dport 999 accept  # tcp dport 999
nft add rule inet filter input tcp dport 1000 accept  # tcp dport 1000
nft add rule inet filter input tcp dport 1001 accept  # tcp dport 1001
nft add rule inet filter input tcp dport 1002 accept  # tcp dport 1002
nft add rule inet filter input tcp dport 1003 accept  # tcp dport 1003
nft add rule inet filter input tcp dport 1004 accept  # tcp dport 1004
nft add rule inet filter input tcp dport 1005 accept  # tcp dport 1005
nft add rule inet filter input tcp dport 1006 accept  # tcp dport 1006
nft add rule inet filter input tcp dport 1007 accept  # tcp dport 1007
nft add rule inet filter input tcp dport 1008 accept  # tcp dport 1008
nft add rule inet filter input tcp dport 1009 accept  # tcp dport 1009
nft add rule inet filter input tcp dport 1010 accept  # tcp dport 1010
nft add rule inet filter input tcp dport 1011 accept  # tcp dport 1011
nft add rule inet filter input tcp dport 1012 accept  # tcp dport 1012
nft add rule inet filter input tcp dport 1013 accept  # tcp dport 1013
nft add rule inet filter input tcp dport 1014 accept  # tcp dport 1014
nft add rule inet filter input tcp dport 1015 accept  # tcp dport 1015
nft add rule inet filter input tcp dport 1016 accept  # tcp dport 1016
nft add rule inet filter input tcp dport 1017 accept  # tcp dport 1017
nft add rule inet filter input tcp dport 1018 accept  # tcp dport 1018
nft add rule inet filter input tcp dport 1019 accept  # tcp dport 1019
nft add rule inet filter input tcp dport 1020 accept  # tcp dport 1020
nft add rule inet filter input tcp dport 1021 accept  # tcp dport 1021
nft add rule inet filter input tcp dport 1022 accept  # tcp dport 1022
nft add rule inet filter input tcp dport 1023 accept  # tcp dport 1023
nft add rule inet filter input tcp dport 1024 accept  # tcp dport 1024
```

### 17.4 chage lejárati dátum minták (2026)
```bash
chage -E 2026-01-01 user1  # fiók lejárat 2026-01-01
chage -E 2026-01-02 user1  # fiók lejárat 2026-01-02
chage -E 2026-01-03 user1  # fiók lejárat 2026-01-03
chage -E 2026-01-04 user1  # fiók lejárat 2026-01-04
chage -E 2026-01-05 user1  # fiók lejárat 2026-01-05
chage -E 2026-01-06 user1  # fiók lejárat 2026-01-06
chage -E 2026-01-07 user1  # fiók lejárat 2026-01-07
chage -E 2026-01-08 user1  # fiók lejárat 2026-01-08
chage -E 2026-01-09 user1  # fiók lejárat 2026-01-09
chage -E 2026-01-10 user1  # fiók lejárat 2026-01-10
chage -E 2026-01-11 user1  # fiók lejárat 2026-01-11
chage -E 2026-01-12 user1  # fiók lejárat 2026-01-12
chage -E 2026-01-13 user1  # fiók lejárat 2026-01-13
chage -E 2026-01-14 user1  # fiók lejárat 2026-01-14
chage -E 2026-01-15 user1  # fiók lejárat 2026-01-15
chage -E 2026-01-16 user1  # fiók lejárat 2026-01-16
chage -E 2026-01-17 user1  # fiók lejárat 2026-01-17
chage -E 2026-01-18 user1  # fiók lejárat 2026-01-18
chage -E 2026-01-19 user1  # fiók lejárat 2026-01-19
chage -E 2026-01-20 user1  # fiók lejárat 2026-01-20
chage -E 2026-01-21 user1  # fiók lejárat 2026-01-21
chage -E 2026-01-22 user1  # fiók lejárat 2026-01-22
chage -E 2026-01-23 user1  # fiók lejárat 2026-01-23
chage -E 2026-01-24 user1  # fiók lejárat 2026-01-24
chage -E 2026-01-25 user1  # fiók lejárat 2026-01-25
chage -E 2026-01-26 user1  # fiók lejárat 2026-01-26
chage -E 2026-01-27 user1  # fiók lejárat 2026-01-27
chage -E 2026-01-28 user1  # fiók lejárat 2026-01-28
chage -E 2026-01-29 user1  # fiók lejárat 2026-01-29
chage -E 2026-01-30 user1  # fiók lejárat 2026-01-30
chage -E 2026-01-31 user1  # fiók lejárat 2026-01-31
chage -E 2026-02-01 user1  # fiók lejárat 2026-02-01
chage -E 2026-02-02 user1  # fiók lejárat 2026-02-02
chage -E 2026-02-03 user1  # fiók lejárat 2026-02-03
chage -E 2026-02-04 user1  # fiók lejárat 2026-02-04
chage -E 2026-02-05 user1  # fiók lejárat 2026-02-05
chage -E 2026-02-06 user1  # fiók lejárat 2026-02-06
chage -E 2026-02-07 user1  # fiók lejárat 2026-02-07
chage -E 2026-02-08 user1  # fiók lejárat 2026-02-08
chage -E 2026-02-09 user1  # fiók lejárat 2026-02-09
chage -E 2026-02-10 user1  # fiók lejárat 2026-02-10
chage -E 2026-02-11 user1  # fiók lejárat 2026-02-11
chage -E 2026-02-12 user1  # fiók lejárat 2026-02-12
chage -E 2026-02-13 user1  # fiók lejárat 2026-02-13
chage -E 2026-02-14 user1  # fiók lejárat 2026-02-14
chage -E 2026-02-15 user1  # fiók lejárat 2026-02-15
chage -E 2026-02-16 user1  # fiók lejárat 2026-02-16
chage -E 2026-02-17 user1  # fiók lejárat 2026-02-17
chage -E 2026-02-18 user1  # fiók lejárat 2026-02-18
chage -E 2026-02-19 user1  # fiók lejárat 2026-02-19
chage -E 2026-02-20 user1  # fiók lejárat 2026-02-20
chage -E 2026-02-21 user1  # fiók lejárat 2026-02-21
chage -E 2026-02-22 user1  # fiók lejárat 2026-02-22
chage -E 2026-02-23 user1  # fiók lejárat 2026-02-23
chage -E 2026-02-24 user1  # fiók lejárat 2026-02-24
chage -E 2026-02-25 user1  # fiók lejárat 2026-02-25
chage -E 2026-02-26 user1  # fiók lejárat 2026-02-26
chage -E 2026-02-27 user1  # fiók lejárat 2026-02-27
chage -E 2026-02-28 user1  # fiók lejárat 2026-02-28
chage -E 2026-03-01 user1  # fiók lejárat 2026-03-01
chage -E 2026-03-02 user1  # fiók lejárat 2026-03-02
chage -E 2026-03-03 user1  # fiók lejárat 2026-03-03
chage -E 2026-03-04 user1  # fiók lejárat 2026-03-04
chage -E 2026-03-05 user1  # fiók lejárat 2026-03-05
chage -E 2026-03-06 user1  # fiók lejárat 2026-03-06
chage -E 2026-03-07 user1  # fiók lejárat 2026-03-07
chage -E 2026-03-08 user1  # fiók lejárat 2026-03-08
chage -E 2026-03-09 user1  # fiók lejárat 2026-03-09
chage -E 2026-03-10 user1  # fiók lejárat 2026-03-10
chage -E 2026-03-11 user1  # fiók lejárat 2026-03-11
chage -E 2026-03-12 user1  # fiók lejárat 2026-03-12
chage -E 2026-03-13 user1  # fiók lejárat 2026-03-13
chage -E 2026-03-14 user1  # fiók lejárat 2026-03-14
chage -E 2026-03-15 user1  # fiók lejárat 2026-03-15
chage -E 2026-03-16 user1  # fiók lejárat 2026-03-16
chage -E 2026-03-17 user1  # fiók lejárat 2026-03-17
chage -E 2026-03-18 user1  # fiók lejárat 2026-03-18
chage -E 2026-03-19 user1  # fiók lejárat 2026-03-19
chage -E 2026-03-20 user1  # fiók lejárat 2026-03-20
chage -E 2026-03-21 user1  # fiók lejárat 2026-03-21
chage -E 2026-03-22 user1  # fiók lejárat 2026-03-22
chage -E 2026-03-23 user1  # fiók lejárat 2026-03-23
chage -E 2026-03-24 user1  # fiók lejárat 2026-03-24
chage -E 2026-03-25 user1  # fiók lejárat 2026-03-25
chage -E 2026-03-26 user1  # fiók lejárat 2026-03-26
chage -E 2026-03-27 user1  # fiók lejárat 2026-03-27
chage -E 2026-03-28 user1  # fiók lejárat 2026-03-28
chage -E 2026-03-29 user1  # fiók lejárat 2026-03-29
chage -E 2026-03-30 user1  # fiók lejárat 2026-03-30
chage -E 2026-03-31 user1  # fiók lejárat 2026-03-31
chage -E 2026-04-01 user1  # fiók lejárat 2026-04-01
chage -E 2026-04-02 user1  # fiók lejárat 2026-04-02
chage -E 2026-04-03 user1  # fiók lejárat 2026-04-03
chage -E 2026-04-04 user1  # fiók lejárat 2026-04-04
chage -E 2026-04-05 user1  # fiók lejárat 2026-04-05
chage -E 2026-04-06 user1  # fiók lejárat 2026-04-06
chage -E 2026-04-07 user1  # fiók lejárat 2026-04-07
chage -E 2026-04-08 user1  # fiók lejárat 2026-04-08
chage -E 2026-04-09 user1  # fiók lejárat 2026-04-09
chage -E 2026-04-10 user1  # fiók lejárat 2026-04-10
chage -E 2026-04-11 user1  # fiók lejárat 2026-04-11
chage -E 2026-04-12 user1  # fiók lejárat 2026-04-12
chage -E 2026-04-13 user1  # fiók lejárat 2026-04-13
chage -E 2026-04-14 user1  # fiók lejárat 2026-04-14
chage -E 2026-04-15 user1  # fiók lejárat 2026-04-15
chage -E 2026-04-16 user1  # fiók lejárat 2026-04-16
chage -E 2026-04-17 user1  # fiók lejárat 2026-04-17
chage -E 2026-04-18 user1  # fiók lejárat 2026-04-18
chage -E 2026-04-19 user1  # fiók lejárat 2026-04-19
chage -E 2026-04-20 user1  # fiók lejárat 2026-04-20
chage -E 2026-04-21 user1  # fiók lejárat 2026-04-21
chage -E 2026-04-22 user1  # fiók lejárat 2026-04-22
chage -E 2026-04-23 user1  # fiók lejárat 2026-04-23
chage -E 2026-04-24 user1  # fiók lejárat 2026-04-24
chage -E 2026-04-25 user1  # fiók lejárat 2026-04-25
chage -E 2026-04-26 user1  # fiók lejárat 2026-04-26
chage -E 2026-04-27 user1  # fiók lejárat 2026-04-27
chage -E 2026-04-28 user1  # fiók lejárat 2026-04-28
chage -E 2026-04-29 user1  # fiók lejárat 2026-04-29
chage -E 2026-04-30 user1  # fiók lejárat 2026-04-30
chage -E 2026-05-01 user1  # fiók lejárat 2026-05-01
chage -E 2026-05-02 user1  # fiók lejárat 2026-05-02
chage -E 2026-05-03 user1  # fiók lejárat 2026-05-03
chage -E 2026-05-04 user1  # fiók lejárat 2026-05-04
chage -E 2026-05-05 user1  # fiók lejárat 2026-05-05
chage -E 2026-05-06 user1  # fiók lejárat 2026-05-06
chage -E 2026-05-07 user1  # fiók lejárat 2026-05-07
chage -E 2026-05-08 user1  # fiók lejárat 2026-05-08
chage -E 2026-05-09 user1  # fiók lejárat 2026-05-09
chage -E 2026-05-10 user1  # fiók lejárat 2026-05-10
chage -E 2026-05-11 user1  # fiók lejárat 2026-05-11
chage -E 2026-05-12 user1  # fiók lejárat 2026-05-12
chage -E 2026-05-13 user1  # fiók lejárat 2026-05-13
chage -E 2026-05-14 user1  # fiók lejárat 2026-05-14
chage -E 2026-05-15 user1  # fiók lejárat 2026-05-15
chage -E 2026-05-16 user1  # fiók lejárat 2026-05-16
chage -E 2026-05-17 user1  # fiók lejárat 2026-05-17
chage -E 2026-05-18 user1  # fiók lejárat 2026-05-18
chage -E 2026-05-19 user1  # fiók lejárat 2026-05-19
chage -E 2026-05-20 user1  # fiók lejárat 2026-05-20
chage -E 2026-05-21 user1  # fiók lejárat 2026-05-21
chage -E 2026-05-22 user1  # fiók lejárat 2026-05-22
chage -E 2026-05-23 user1  # fiók lejárat 2026-05-23
chage -E 2026-05-24 user1  # fiók lejárat 2026-05-24
chage -E 2026-05-25 user1  # fiók lejárat 2026-05-25
chage -E 2026-05-26 user1  # fiók lejárat 2026-05-26
chage -E 2026-05-27 user1  # fiók lejárat 2026-05-27
chage -E 2026-05-28 user1  # fiók lejárat 2026-05-28
chage -E 2026-05-29 user1  # fiók lejárat 2026-05-29
chage -E 2026-05-30 user1  # fiók lejárat 2026-05-30
chage -E 2026-05-31 user1  # fiók lejárat 2026-05-31
chage -E 2026-06-01 user1  # fiók lejárat 2026-06-01
chage -E 2026-06-02 user1  # fiók lejárat 2026-06-02
chage -E 2026-06-03 user1  # fiók lejárat 2026-06-03
chage -E 2026-06-04 user1  # fiók lejárat 2026-06-04
chage -E 2026-06-05 user1  # fiók lejárat 2026-06-05
chage -E 2026-06-06 user1  # fiók lejárat 2026-06-06
chage -E 2026-06-07 user1  # fiók lejárat 2026-06-07
chage -E 2026-06-08 user1  # fiók lejárat 2026-06-08
chage -E 2026-06-09 user1  # fiók lejárat 2026-06-09
chage -E 2026-06-10 user1  # fiók lejárat 2026-06-10
chage -E 2026-06-11 user1  # fiók lejárat 2026-06-11
chage -E 2026-06-12 user1  # fiók lejárat 2026-06-12
chage -E 2026-06-13 user1  # fiók lejárat 2026-06-13
chage -E 2026-06-14 user1  # fiók lejárat 2026-06-14
chage -E 2026-06-15 user1  # fiók lejárat 2026-06-15
chage -E 2026-06-16 user1  # fiók lejárat 2026-06-16
chage -E 2026-06-17 user1  # fiók lejárat 2026-06-17
chage -E 2026-06-18 user1  # fiók lejárat 2026-06-18
chage -E 2026-06-19 user1  # fiók lejárat 2026-06-19
chage -E 2026-06-20 user1  # fiók lejárat 2026-06-20
chage -E 2026-06-21 user1  # fiók lejárat 2026-06-21
chage -E 2026-06-22 user1  # fiók lejárat 2026-06-22
chage -E 2026-06-23 user1  # fiók lejárat 2026-06-23
chage -E 2026-06-24 user1  # fiók lejárat 2026-06-24
chage -E 2026-06-25 user1  # fiók lejárat 2026-06-25
chage -E 2026-06-26 user1  # fiók lejárat 2026-06-26
chage -E 2026-06-27 user1  # fiók lejárat 2026-06-27
chage -E 2026-06-28 user1  # fiók lejárat 2026-06-28
chage -E 2026-06-29 user1  # fiók lejárat 2026-06-29
chage -E 2026-06-30 user1  # fiók lejárat 2026-06-30
chage -E 2026-07-01 user1  # fiók lejárat 2026-07-01
chage -E 2026-07-02 user1  # fiók lejárat 2026-07-02
chage -E 2026-07-03 user1  # fiók lejárat 2026-07-03
chage -E 2026-07-04 user1  # fiók lejárat 2026-07-04
chage -E 2026-07-05 user1  # fiók lejárat 2026-07-05
chage -E 2026-07-06 user1  # fiók lejárat 2026-07-06
chage -E 2026-07-07 user1  # fiók lejárat 2026-07-07
chage -E 2026-07-08 user1  # fiók lejárat 2026-07-08
chage -E 2026-07-09 user1  # fiók lejárat 2026-07-09
chage -E 2026-07-10 user1  # fiók lejárat 2026-07-10
chage -E 2026-07-11 user1  # fiók lejárat 2026-07-11
chage -E 2026-07-12 user1  # fiók lejárat 2026-07-12
chage -E 2026-07-13 user1  # fiók lejárat 2026-07-13
chage -E 2026-07-14 user1  # fiók lejárat 2026-07-14
chage -E 2026-07-15 user1  # fiók lejárat 2026-07-15
chage -E 2026-07-16 user1  # fiók lejárat 2026-07-16
chage -E 2026-07-17 user1  # fiók lejárat 2026-07-17
chage -E 2026-07-18 user1  # fiók lejárat 2026-07-18
chage -E 2026-07-19 user1  # fiók lejárat 2026-07-19
chage -E 2026-07-20 user1  # fiók lejárat 2026-07-20
chage -E 2026-07-21 user1  # fiók lejárat 2026-07-21
chage -E 2026-07-22 user1  # fiók lejárat 2026-07-22
chage -E 2026-07-23 user1  # fiók lejárat 2026-07-23
chage -E 2026-07-24 user1  # fiók lejárat 2026-07-24
chage -E 2026-07-25 user1  # fiók lejárat 2026-07-25
chage -E 2026-07-26 user1  # fiók lejárat 2026-07-26
chage -E 2026-07-27 user1  # fiók lejárat 2026-07-27
chage -E 2026-07-28 user1  # fiók lejárat 2026-07-28
chage -E 2026-07-29 user1  # fiók lejárat 2026-07-29
chage -E 2026-07-30 user1  # fiók lejárat 2026-07-30
chage -E 2026-07-31 user1  # fiók lejárat 2026-07-31
chage -E 2026-08-01 user1  # fiók lejárat 2026-08-01
chage -E 2026-08-02 user1  # fiók lejárat 2026-08-02
chage -E 2026-08-03 user1  # fiók lejárat 2026-08-03
chage -E 2026-08-04 user1  # fiók lejárat 2026-08-04
chage -E 2026-08-05 user1  # fiók lejárat 2026-08-05
chage -E 2026-08-06 user1  # fiók lejárat 2026-08-06
chage -E 2026-08-07 user1  # fiók lejárat 2026-08-07
chage -E 2026-08-08 user1  # fiók lejárat 2026-08-08
chage -E 2026-08-09 user1  # fiók lejárat 2026-08-09
chage -E 2026-08-10 user1  # fiók lejárat 2026-08-10
chage -E 2026-08-11 user1  # fiók lejárat 2026-08-11
chage -E 2026-08-12 user1  # fiók lejárat 2026-08-12
chage -E 2026-08-13 user1  # fiók lejárat 2026-08-13
chage -E 2026-08-14 user1  # fiók lejárat 2026-08-14
chage -E 2026-08-15 user1  # fiók lejárat 2026-08-15
chage -E 2026-08-16 user1  # fiók lejárat 2026-08-16
chage -E 2026-08-17 user1  # fiók lejárat 2026-08-17
chage -E 2026-08-18 user1  # fiók lejárat 2026-08-18
chage -E 2026-08-19 user1  # fiók lejárat 2026-08-19
chage -E 2026-08-20 user1  # fiók lejárat 2026-08-20
chage -E 2026-08-21 user1  # fiók lejárat 2026-08-21
chage -E 2026-08-22 user1  # fiók lejárat 2026-08-22
chage -E 2026-08-23 user1  # fiók lejárat 2026-08-23
chage -E 2026-08-24 user1  # fiók lejárat 2026-08-24
chage -E 2026-08-25 user1  # fiók lejárat 2026-08-25
chage -E 2026-08-26 user1  # fiók lejárat 2026-08-26
chage -E 2026-08-27 user1  # fiók lejárat 2026-08-27
chage -E 2026-08-28 user1  # fiók lejárat 2026-08-28
chage -E 2026-08-29 user1  # fiók lejárat 2026-08-29
chage -E 2026-08-30 user1  # fiók lejárat 2026-08-30
chage -E 2026-08-31 user1  # fiók lejárat 2026-08-31
chage -E 2026-09-01 user1  # fiók lejárat 2026-09-01
chage -E 2026-09-02 user1  # fiók lejárat 2026-09-02
chage -E 2026-09-03 user1  # fiók lejárat 2026-09-03
chage -E 2026-09-04 user1  # fiók lejárat 2026-09-04
chage -E 2026-09-05 user1  # fiók lejárat 2026-09-05
chage -E 2026-09-06 user1  # fiók lejárat 2026-09-06
chage -E 2026-09-07 user1  # fiók lejárat 2026-09-07
chage -E 2026-09-08 user1  # fiók lejárat 2026-09-08
chage -E 2026-09-09 user1  # fiók lejárat 2026-09-09
chage -E 2026-09-10 user1  # fiók lejárat 2026-09-10
chage -E 2026-09-11 user1  # fiók lejárat 2026-09-11
chage -E 2026-09-12 user1  # fiók lejárat 2026-09-12
chage -E 2026-09-13 user1  # fiók lejárat 2026-09-13
chage -E 2026-09-14 user1  # fiók lejárat 2026-09-14
chage -E 2026-09-15 user1  # fiók lejárat 2026-09-15
chage -E 2026-09-16 user1  # fiók lejárat 2026-09-16
chage -E 2026-09-17 user1  # fiók lejárat 2026-09-17
chage -E 2026-09-18 user1  # fiók lejárat 2026-09-18
chage -E 2026-09-19 user1  # fiók lejárat 2026-09-19
chage -E 2026-09-20 user1  # fiók lejárat 2026-09-20
chage -E 2026-09-21 user1  # fiók lejárat 2026-09-21
chage -E 2026-09-22 user1  # fiók lejárat 2026-09-22
chage -E 2026-09-23 user1  # fiók lejárat 2026-09-23
chage -E 2026-09-24 user1  # fiók lejárat 2026-09-24
chage -E 2026-09-25 user1  # fiók lejárat 2026-09-25
chage -E 2026-09-26 user1  # fiók lejárat 2026-09-26
chage -E 2026-09-27 user1  # fiók lejárat 2026-09-27
chage -E 2026-09-28 user1  # fiók lejárat 2026-09-28
chage -E 2026-09-29 user1  # fiók lejárat 2026-09-29
chage -E 2026-09-30 user1  # fiók lejárat 2026-09-30
chage -E 2026-10-01 user1  # fiók lejárat 2026-10-01
chage -E 2026-10-02 user1  # fiók lejárat 2026-10-02
chage -E 2026-10-03 user1  # fiók lejárat 2026-10-03
chage -E 2026-10-04 user1  # fiók lejárat 2026-10-04
chage -E 2026-10-05 user1  # fiók lejárat 2026-10-05
chage -E 2026-10-06 user1  # fiók lejárat 2026-10-06
chage -E 2026-10-07 user1  # fiók lejárat 2026-10-07
chage -E 2026-10-08 user1  # fiók lejárat 2026-10-08
chage -E 2026-10-09 user1  # fiók lejárat 2026-10-09
chage -E 2026-10-10 user1  # fiók lejárat 2026-10-10
chage -E 2026-10-11 user1  # fiók lejárat 2026-10-11
chage -E 2026-10-12 user1  # fiók lejárat 2026-10-12
chage -E 2026-10-13 user1  # fiók lejárat 2026-10-13
chage -E 2026-10-14 user1  # fiók lejárat 2026-10-14
chage -E 2026-10-15 user1  # fiók lejárat 2026-10-15
chage -E 2026-10-16 user1  # fiók lejárat 2026-10-16
chage -E 2026-10-17 user1  # fiók lejárat 2026-10-17
chage -E 2026-10-18 user1  # fiók lejárat 2026-10-18
chage -E 2026-10-19 user1  # fiók lejárat 2026-10-19
chage -E 2026-10-20 user1  # fiók lejárat 2026-10-20
chage -E 2026-10-21 user1  # fiók lejárat 2026-10-21
chage -E 2026-10-22 user1  # fiók lejárat 2026-10-22
chage -E 2026-10-23 user1  # fiók lejárat 2026-10-23
chage -E 2026-10-24 user1  # fiók lejárat 2026-10-24
chage -E 2026-10-25 user1  # fiók lejárat 2026-10-25
chage -E 2026-10-26 user1  # fiók lejárat 2026-10-26
chage -E 2026-10-27 user1  # fiók lejárat 2026-10-27
chage -E 2026-10-28 user1  # fiók lejárat 2026-10-28
chage -E 2026-10-29 user1  # fiók lejárat 2026-10-29
chage -E 2026-10-30 user1  # fiók lejárat 2026-10-30
chage -E 2026-10-31 user1  # fiók lejárat 2026-10-31
chage -E 2026-11-01 user1  # fiók lejárat 2026-11-01
chage -E 2026-11-02 user1  # fiók lejárat 2026-11-02
chage -E 2026-11-03 user1  # fiók lejárat 2026-11-03
chage -E 2026-11-04 user1  # fiók lejárat 2026-11-04
chage -E 2026-11-05 user1  # fiók lejárat 2026-11-05
chage -E 2026-11-06 user1  # fiók lejárat 2026-11-06
chage -E 2026-11-07 user1  # fiók lejárat 2026-11-07
chage -E 2026-11-08 user1  # fiók lejárat 2026-11-08
chage -E 2026-11-09 user1  # fiók lejárat 2026-11-09
chage -E 2026-11-10 user1  # fiók lejárat 2026-11-10
chage -E 2026-11-11 user1  # fiók lejárat 2026-11-11
chage -E 2026-11-12 user1  # fiók lejárat 2026-11-12
chage -E 2026-11-13 user1  # fiók lejárat 2026-11-13
chage -E 2026-11-14 user1  # fiók lejárat 2026-11-14
chage -E 2026-11-15 user1  # fiók lejárat 2026-11-15
chage -E 2026-11-16 user1  # fiók lejárat 2026-11-16
chage -E 2026-11-17 user1  # fiók lejárat 2026-11-17
chage -E 2026-11-18 user1  # fiók lejárat 2026-11-18
chage -E 2026-11-19 user1  # fiók lejárat 2026-11-19
chage -E 2026-11-20 user1  # fiók lejárat 2026-11-20
chage -E 2026-11-21 user1  # fiók lejárat 2026-11-21
chage -E 2026-11-22 user1  # fiók lejárat 2026-11-22
chage -E 2026-11-23 user1  # fiók lejárat 2026-11-23
chage -E 2026-11-24 user1  # fiók lejárat 2026-11-24
chage -E 2026-11-25 user1  # fiók lejárat 2026-11-25
chage -E 2026-11-26 user1  # fiók lejárat 2026-11-26
chage -E 2026-11-27 user1  # fiók lejárat 2026-11-27
chage -E 2026-11-28 user1  # fiók lejárat 2026-11-28
chage -E 2026-11-29 user1  # fiók lejárat 2026-11-29
chage -E 2026-11-30 user1  # fiók lejárat 2026-11-30
chage -E 2026-12-01 user1  # fiók lejárat 2026-12-01
chage -E 2026-12-02 user1  # fiók lejárat 2026-12-02
chage -E 2026-12-03 user1  # fiók lejárat 2026-12-03
chage -E 2026-12-04 user1  # fiók lejárat 2026-12-04
chage -E 2026-12-05 user1  # fiók lejárat 2026-12-05
chage -E 2026-12-06 user1  # fiók lejárat 2026-12-06
chage -E 2026-12-07 user1  # fiók lejárat 2026-12-07
chage -E 2026-12-08 user1  # fiók lejárat 2026-12-08
chage -E 2026-12-09 user1  # fiók lejárat 2026-12-09
chage -E 2026-12-10 user1  # fiók lejárat 2026-12-10
chage -E 2026-12-11 user1  # fiók lejárat 2026-12-11
chage -E 2026-12-12 user1  # fiók lejárat 2026-12-12
chage -E 2026-12-13 user1  # fiók lejárat 2026-12-13
chage -E 2026-12-14 user1  # fiók lejárat 2026-12-14
chage -E 2026-12-15 user1  # fiók lejárat 2026-12-15
chage -E 2026-12-16 user1  # fiók lejárat 2026-12-16
chage -E 2026-12-17 user1  # fiók lejárat 2026-12-17
chage -E 2026-12-18 user1  # fiók lejárat 2026-12-18
chage -E 2026-12-19 user1  # fiók lejárat 2026-12-19
chage -E 2026-12-20 user1  # fiók lejárat 2026-12-20
chage -E 2026-12-21 user1  # fiók lejárat 2026-12-21
chage -E 2026-12-22 user1  # fiók lejárat 2026-12-22
chage -E 2026-12-23 user1  # fiók lejárat 2026-12-23
chage -E 2026-12-24 user1  # fiók lejárat 2026-12-24
chage -E 2026-12-25 user1  # fiók lejárat 2026-12-25
chage -E 2026-12-26 user1  # fiók lejárat 2026-12-26
chage -E 2026-12-27 user1  # fiók lejárat 2026-12-27
chage -E 2026-12-28 user1  # fiók lejárat 2026-12-28
chage -E 2026-12-29 user1  # fiók lejárat 2026-12-29
chage -E 2026-12-30 user1  # fiók lejárat 2026-12-30
chage -E 2026-12-31 user1  # fiók lejárat 2026-12-31
```

### 17.5 systemctl műveletek gyakori szolgáltatásokra
```bash
systemctl start sshd  # start sshd
systemctl stop sshd  # stop sshd
systemctl restart sshd  # restart sshd
systemctl reload sshd  # reload sshd
systemctl status sshd  # status sshd
systemctl enable sshd  # enable sshd
systemctl disable sshd  # disable sshd
systemctl is-enabled sshd  # is-enabled sshd
systemctl start httpd  # start httpd
systemctl stop httpd  # stop httpd
systemctl restart httpd  # restart httpd
systemctl reload httpd  # reload httpd
systemctl status httpd  # status httpd
systemctl enable httpd  # enable httpd
systemctl disable httpd  # disable httpd
systemctl is-enabled httpd  # is-enabled httpd
systemctl start firewalld  # start firewalld
systemctl stop firewalld  # stop firewalld
systemctl restart firewalld  # restart firewalld
systemctl reload firewalld  # reload firewalld
systemctl status firewalld  # status firewalld
systemctl enable firewalld  # enable firewalld
systemctl disable firewalld  # disable firewalld
systemctl is-enabled firewalld  # is-enabled firewalld
systemctl start nftables  # start nftables
systemctl stop nftables  # stop nftables
systemctl restart nftables  # restart nftables
systemctl reload nftables  # reload nftables
systemctl status nftables  # status nftables
systemctl enable nftables  # enable nftables
systemctl disable nftables  # disable nftables
systemctl is-enabled nftables  # is-enabled nftables
systemctl start auditd  # start auditd
systemctl stop auditd  # stop auditd
systemctl restart auditd  # restart auditd
systemctl reload auditd  # reload auditd
systemctl status auditd  # status auditd
systemctl enable auditd  # enable auditd
systemctl disable auditd  # disable auditd
systemctl is-enabled auditd  # is-enabled auditd
systemctl start chronyd  # start chronyd
systemctl stop chronyd  # stop chronyd
systemctl restart chronyd  # restart chronyd
systemctl reload chronyd  # reload chronyd
systemctl status chronyd  # status chronyd
systemctl enable chronyd  # enable chronyd
systemctl disable chronyd  # disable chronyd
systemctl is-enabled chronyd  # is-enabled chronyd
systemctl start cockpit  # start cockpit
systemctl stop cockpit  # stop cockpit
systemctl restart cockpit  # restart cockpit
systemctl reload cockpit  # reload cockpit
systemctl status cockpit  # status cockpit
systemctl enable cockpit  # enable cockpit
systemctl disable cockpit  # disable cockpit
systemctl is-enabled cockpit  # is-enabled cockpit
systemctl start rsyslog  # start rsyslog
systemctl stop rsyslog  # stop rsyslog
systemctl restart rsyslog  # restart rsyslog
systemctl reload rsyslog  # reload rsyslog
systemctl status rsyslog  # status rsyslog
systemctl enable rsyslog  # enable rsyslog
systemctl disable rsyslog  # disable rsyslog
systemctl is-enabled rsyslog  # is-enabled rsyslog
systemctl start crond  # start crond
systemctl stop crond  # stop crond
systemctl restart crond  # restart crond
systemctl reload crond  # reload crond
systemctl status crond  # status crond
systemctl enable crond  # enable crond
systemctl disable crond  # disable crond
systemctl is-enabled crond  # is-enabled crond
systemctl start NetworkManager  # start NetworkManager
systemctl stop NetworkManager  # stop NetworkManager
systemctl restart NetworkManager  # restart NetworkManager
systemctl reload NetworkManager  # reload NetworkManager
systemctl status NetworkManager  # status NetworkManager
systemctl enable NetworkManager  # enable NetworkManager
systemctl disable NetworkManager  # disable NetworkManager
systemctl is-enabled NetworkManager  # is-enabled NetworkManager
systemctl start postgresql  # start postgresql
systemctl stop postgresql  # stop postgresql
systemctl restart postgresql  # restart postgresql
systemctl reload postgresql  # reload postgresql
systemctl status postgresql  # status postgresql
systemctl enable postgresql  # enable postgresql
systemctl disable postgresql  # disable postgresql
systemctl is-enabled postgresql  # is-enabled postgresql
systemctl start mariadb  # start mariadb
systemctl stop mariadb  # stop mariadb
systemctl restart mariadb  # restart mariadb
systemctl reload mariadb  # reload mariadb
systemctl status mariadb  # status mariadb
systemctl enable mariadb  # enable mariadb
systemctl disable mariadb  # disable mariadb
systemctl is-enabled mariadb  # is-enabled mariadb
systemctl start redis  # start redis
systemctl stop redis  # stop redis
systemctl restart redis  # restart redis
systemctl reload redis  # reload redis
systemctl status redis  # status redis
systemctl enable redis  # enable redis
systemctl disable redis  # disable redis
systemctl is-enabled redis  # is-enabled redis
systemctl start docker  # start docker
systemctl stop docker  # stop docker
systemctl restart docker  # restart docker
systemctl reload docker  # reload docker
systemctl status docker  # status docker
systemctl enable docker  # enable docker
systemctl disable docker  # disable docker
systemctl is-enabled docker  # is-enabled docker
systemctl start podman  # start podman
systemctl stop podman  # stop podman
systemctl restart podman  # restart podman
systemctl reload podman  # reload podman
systemctl status podman  # status podman
systemctl enable podman  # enable podman
systemctl disable podman  # disable podman
systemctl is-enabled podman  # is-enabled podman
systemctl start nginx  # start nginx
systemctl stop nginx  # stop nginx
systemctl restart nginx  # restart nginx
systemctl reload nginx  # reload nginx
systemctl status nginx  # status nginx
systemctl enable nginx  # enable nginx
systemctl disable nginx  # disable nginx
systemctl is-enabled nginx  # is-enabled nginx
systemctl start vsftpd  # start vsftpd
systemctl stop vsftpd  # stop vsftpd
systemctl restart vsftpd  # restart vsftpd
systemctl reload vsftpd  # reload vsftpd
systemctl status vsftpd  # status vsftpd
systemctl enable vsftpd  # enable vsftpd
systemctl disable vsftpd  # disable vsftpd
systemctl is-enabled vsftpd  # is-enabled vsftpd
systemctl start fail2ban  # start fail2ban
systemctl stop fail2ban  # stop fail2ban
systemctl restart fail2ban  # restart fail2ban
systemctl reload fail2ban  # reload fail2ban
systemctl status fail2ban  # status fail2ban
systemctl enable fail2ban  # enable fail2ban
systemctl disable fail2ban  # disable fail2ban
systemctl is-enabled fail2ban  # is-enabled fail2ban
systemctl start cups  # start cups
systemctl stop cups  # stop cups
systemctl restart cups  # restart cups
systemctl reload cups  # reload cups
systemctl status cups  # status cups
systemctl enable cups  # enable cups
systemctl disable cups  # disable cups
systemctl is-enabled cups  # is-enabled cups
systemctl start sssd  # start sssd
systemctl stop sssd  # stop sssd
systemctl restart sssd  # restart sssd
systemctl reload sssd  # reload sssd
systemctl status sssd  # status sssd
systemctl enable sssd  # enable sssd
systemctl disable sssd  # disable sssd
systemctl is-enabled sssd  # is-enabled sssd
```
