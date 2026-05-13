# IT-Biztonság – Cheatsheet

## 1) Helyreállítás és root hozzáférés

### Rescue mód (kernel paraméter)
```bash
rd.break
```
- `rd.break`: korai megszakítás az initramfs-ben, root shellhez.

### Rendszerpartíció írhatóvá tétele
```bash
mount -o remount,rw /sysroot
```
- `-o`: mount opciók.
- `remount`: újracsatolás.
- `rw`: írható módban csatolás.
- `/sysroot`: a helyreállítási környezet rootja.

### Belépés a chroot környezetbe
```bash
chroot /sysroot
```
- `chroot <path>`: új root könyvtár megadása.

### Jelszócsere
```bash
passwd
```
- Paraméter nélkül: aktuális felhasználó jelszava.
- `passwd user1`: célfelhasználó jelszava.

### SELinux újracímkézés indítása
```bash
touch ./autorelabel
```
- `autorelabel`: újracímkézés indítása reboot után.

### Felhasználóváltás
```bash
su
```
- `su -`: login shellt ad a célfelhasználónak.
- `su user1`: váltás adott felhasználóra.

## 2) Felhasználók és jelszavak

### Felhasználó létrehozás
```bash
useradd user1
```
- `-m`: home könyvtár létrehozása.
- `-s /bin/bash`: beállított shell.
- `-G wheel`: csoporttagság (elsődleges mellé).

### Felhasználó azonosítók
```bash
id user1
```
- `id`: UID, GID és csoportok listája.

### Csoporttagság módosítás
```bash
usermod user1 -G users
```
- `-G`: teljes csoportlista felülírása.
- `-aG`: hozzáadás meglévők megtartásával.

### Jelszó zárolása
```bash
passwd -l user1
```
- `-l`: lock, jelszó hash elé `!` kerül.

### Jelszó feloldása
```bash
passwd -u user1
```
- `-u`: unlock.

### Zárolási státusz
```bash
passwd -S user1
```
- `-S`: státusz (locked/active) és dátumok.

### Jelszó-életciklus listázás
```bash
chage -l user1
```
- `-l`: részletes élettartam adatok.

### Utolsó jelszócsere dátum beállítás
```bash
chage -d 2020-01-01 user1
```
- `-d`: last change dátum.

### Max. jelszó-élettartam beállítás
```bash
chage -M 10 user1
```
- `-M`: maximum napok száma.

### Teljes jelszó-életciklus beállítás
```bash
chage -d 2020-01-01 -m 0 -M 30 -I 30 -E 2020-04-01 user1
```
- `-m`: minimum napok két csere között.
- `-I`: inaktív napok lejárat után.
- `-E`: fiók lejárati dátum.

### Shadow mezők ellenőrzése
```bash
grep user1 /etc/shadow | cut -d : -f 1,3-8
```
- `-d :`: mezőelválasztó.
- `-f 1,3-8`: felhasználó és jelszó-életciklus mezők.

## 3) Sudo és jogosultság delegálás

### Program helye
```bash
which shutdown
```
- `which`: futtatható helye PATH alapján.

### Link követése státuszban
```bash
stat -L /usr/sbin/shutdown
```
- `-L`: szimbolikus link feloldása.

### Sudoers szerkesztés
```bash
visudo
```
- Szintaxisellenőrzéssel ment.

### Sudoers sor hozzáadása
```bash
echo "admin localhost=(root) /usr/sbin/shutdown" >>/etc/sudoers
```
- `>>`: hozzáfűzés (óvatosan, inkább `visudo`).

### Sudo futtatás más néven
```bash
sudo -u root shutdown
```
- `-u`: célfelhasználó.

### Jelszó nélküli sudo engedély
```bash
echo "admin localhost=(root) NOPASSWD:/usr/sbin/reboot" >>/etc/sudoers
```
- `NOPASSWD`: jelszó nélküli futtatás.

### Hostnév módosítás
```bash
hostnamectl set-hostname it-security-vm
```
- `set-hostname`: tartós hostnév.

## 4) SSH és kulcskezelés

### Kulcspár generálás
```bash
ssh-keygen
```
- Interaktív: mentés helye, passphrase.

### Kulcspár generálás névvel
```bash
ssh-keygen -f new-key
```
- `-f`: kimeneti fájlnév.

### RSA kulcs 4096 bit
```bash
ssh-keygen -t rsa -b 4096
```
- `-t`: algoritmus.
- `-b`: kulcshossz.

### Ed25519 kulcs erősített KDF-fel
```bash
ssh-keygen -t ed25519 -a 100 -f ~/.ssh/id_ed25519
```
- `-a`: KDF iterációk száma.

### Bejelentkezés SSH-val
```bash
ssh ssh-user-2@localhost
```
- `-i`: privát kulcs megadása.
- `-p`: port.

### Távoli parancs futtatása
```bash
ssh ssh-user-2@localhost pwd
```
- Parancs a távoli hoston fut.

### authorized_keys létrehozás
```bash
cat .ssh/id_rsa.pub >.ssh/authorized_keys
```
- `>`: felülírja a fájlt.

### authorized_keys bővítés
```bash
cat new-key.pub >>.ssh/authorized_keys
```
- `>>`: hozzáfűzés.

### .ssh jogosultságok
```bash
chmod 700 /home/ssh-user-2/.ssh
```
- `700`: csak tulajdonos fér hozzá.

### authorized_keys jogosultságok
```bash
chmod 600 /home/ssh-user-2/.ssh/authorized_keys
```
- `600`: csak tulajdonos olvas/ír.

### Kulcs másolása szerverre
```bash
ssh-copy-id ssh-user-3@localhost
```
- `-i`: publikus kulcs fájl.

### Privát kulcs jelszavas védése
```bash
ssh-keygen -p -f .ssh/id_rsa
```
- `-p`: passphrase módosítás.

## 5) Jogosultságok, ACL, SELinux, linkek

### Alap jogosultságok listázása
```bash
ls -l
```
- `-l`: hosszú lista tulaj, jogok, idő.

### Jogosultság módosítás
```bash
chmod 640 file.txt
```
- Számjegyek: owner/group/other.
- `chmod u+rx file.txt`: szimbolikus mód.

### Tulajdonos/csoport módosítás
```bash
chown user:group file.txt
```
- `-R`: rekurzív.

### Alap jogosultság maszk
```bash
umask
```
- `umask 027`: új fájlok maszkolása.

### ACL lekérdezés
```bash
getfacl file.txt
```
- `-p`: teljes elérési út.

### ACL beállítás
```bash
setfacl -m u:user:rwx file.txt
```
- `-m`: modify.
- `-R`: rekurzív.

### SELinux kontextus lekérdezés
```bash
ls -Z /etc/shadow*
```
- `-Z`: SELinux címke.

### SELinux kontextus ideiglenes módosítás
```bash
chcon -t httpd_sys_content_t /var/www/html/index.html
```
- `-t`: új type.

### SELinux kontextus visszaállítás
```bash
restorecon -v /var/www/html/index.html
```
- `-v`: részletes.

### Hard link
```bash
ln source.txt link.txt
```
- Ugyanaz az inode, közös tartalom.

### Soft link
```bash
ln -s /path/source /path/link
```
- `-s`: szimbolikus link.

## 6) Tűzfal és szolgáltatások

### HTTP szerver telepítés
```bash
dnf install -y httpd
```
- `-y`: automatikus igen.

### Hallgatózó portok
```bash
ss -ltnp
```
- `-l`: csak listen.
- `-t`: TCP.
- `-n`: numerikus.
- `-p`: process info.

### Firewalld státusz
```bash
firewall-cmd --state
```
- `--state`: running vagy not running.

### Szolgáltatás engedélyezése zónában
```bash
firewall-cmd --zone=internal --add-service=http
```
- `--zone`: célzóna.
- `--add-service`: szolgáltatás hozzáadás.

### Tartós szabály hozzáadása
```bash
firewall-cmd --permanent --zone=internal --add-service=http
```
- `--permanent`: mentett konfiguráció.

### Futó szabályok mentése
```bash
firewall-cmd --runtime-to-permanent
```
- Runtime szabályok átmentése.

### Szolgáltatás definíciók
```bash
cat /usr/lib/firewalld/services/http.xml
```
- XML alapértelmezett definíció.

### Netfilter szabályok listázása
```bash
nft list ruleset
```
- Teljes NFT szabályrendszer.

### Cockpit indítása
```bash
systemctl start cockpit
```
- `enable`: automatikus indítás.

## 7) Hálózati forgalom rögzítés

### Interfész állapot
```bash
ip link show dev enp0s3
```
- `show dev`: adott interfész.

### Promiscuous mód bekapcsolás
```bash
ip link set enp0s3 promisc on
```
- `promisc on`: minden csomag fogadása.

### Dumpcap 30 másodpercig
```bash
dumpcap -i enp0s3 -w /tmp/1.pcap -a duration:30
```
- `-i`: interfész.
- `-w`: fájl.
- `-a duration`: automatikus leállítás.

### Dumpcap fájlméret limit
```bash
dumpcap -i enp0s3 -w /tmp/2.pcap -a filesize:10000
```
- `filesize`: KB limit.

### Szűrés host szerint
```bash
dumpcap -i enp0s3 -f "host 10.0.0.1"
```
- `-f`: BPF szűrő.

### ICMP szűrés
```bash
dumpcap -i enp0s3 -f "host 10.0.0.1 && (icmp[0] == 0 || icmp[0] == 8 )"
```
- `icmp[0] == 8`: echo request.
- `icmp[0] == 0`: echo reply.

### TCP port szűrés
```bash
dumpcap -i enp0s3 -f "host 10.0.0.1 && (tcp[0:2] == 8080 )"
```
- `tcp[0:2]`: port mező.

## 8) Titkosított tárolók (LUKS, loop)

### Eszközlista fájlrendszerrel
```bash
lsblk -f
```
- `-f`: fájlrendszer és UUID.

### Nyers fájl létrehozás
```bash
dd if=/dev/zero of=/device0 bs=4096 count=1024
```
- `if`: bemenet.
- `of`: kimenet.
- `bs`: blokk méret.
- `count`: blokkok száma.

### Fájl gyors allokálás
```bash
fallocate -l 4MiB /device1
```
- `-l`: méret.

### Loop eszköz hozzárendelés
```bash
losetup -f /device0
```
- `-f`: első szabad loop.

### Loop eszközök listázása
```bash
losetup -a
```
- `-a`: összes.

### LUKS formázás
```bash
cryptsetup luksFormat /dev/sdb1
```
- `--type luks2`: LUKS2 formátum.
- `--cipher`: titkosítási algoritmus.

### LUKS megnyitás
```bash
cryptsetup luksOpen /dev/sdb1 enc-home
```
- `enc-home`: mapper név.

### LUKS lezárás
```bash
cryptsetup luksClose enc-home
```
- `luksClose`: mapper lezárás.

### Ext2 fájlrendszer létrehozás
```bash
mkfs.ext2 /dev/mapper/enc-home
```
- `-L`: label.

### XFS fájlrendszer létrehozás
```bash
mkfs.xfs /dev/mapper/secret
```
- `-f`: felülírás megerősítés nélkül.

### Csatolás
```bash
mount /dev/mapper/enc-home /home
```
- `<device> <mountpoint>`.

### Csatolás leválasztás
```bash
umount /home
```
- `-l`: lazy unmount.

### Crypttab ellenőrzés
```bash
cat /etc/crypttab
```
- LUKS automount definíciók.

### Fstab ellenőrzés
```bash
cat /etc/fstab | grep -v '#'
```
- `grep -v`: kommentek kiszűrése.

### Titkosított tartalom ellenőrzés
```bash
grep -ac "Secret" /device1
```
- `-a`: bináris kezelése szövegként.
- `-c`: találatok száma.

## 9) Kulcsfájl és automount

### Kulcsfájl generálás
```bash
dd if=/dev/random of=/keys/home.key bs=16 count=1
```
- `bs=16 count=1`: 16 bájtos kulcs.

### Kulcsfájl jogosultság
```bash
chmod 000 /keys/home.key
```
- `000`: senki ne tudja olvasni.

### LUKS kulcsfájllal
```bash
cryptsetup luksFormat --key-file /keys/home.key /dev/sdb1 secret
```
- `--key-file`: kulcsfájl.

### Kulccsal nyitás
```bash
cryptsetup luksOpen /dev/sdb1 secret
```
- `secret`: mapper név.

### Automount szerkesztés
```bash
nano /etc/fstab
```
- Fájlrendszerek automatikus csatolása.

### Crypttab szerkesztés
```bash
nano /etc/crypttab
```
- Titkosított eszközök automatikus nyitása.

## 10) OpenSSL – szimmetrikus titkosítás

### AES-128 ECB titkosítás
```bash
openssl aes-128-ecb -in file -out file.aes -K 012345678901234567890123456789ab
```
- `-K`: hex kulcs (16 bájt).
- `-in/-out`: bemenet/kimenet.

### AES-128 ECB visszafejtés
```bash
openssl aes-128-ecb -d -in file.aes -K 012345678901234567890123456789ab
```
- `-d`: decrypt.

### AES-128 CBC jelszóval
```bash
openssl aes-128-cbc -k PASSWORD -in file -out file.aes -p
```
- `-k`: jelszó.
- `-p`: kulcs/iv kiírás.
- `-salt -pbkdf2 -iter 200000`: erősített kulcsszármaztatás.

### AES-128 CBC visszafejtés
```bash
openssl aes-128-cbc -d -k PASSWORD -in file.aes
```
- `-d`: decrypt.

### DES ECB titkosítás
```bash
openssl des-ecb -in date -K AC22B54854967CA9 -out date.enc
```
- `des-ecb`: örökölt, tanulási célra.

### DES ECB visszafejtés
```bash
openssl des-ecb -d -in date.enc -K AC22B54854967CA9
```
- `-d`: decrypt.

### DES CBC inicializáló vektorral
```bash
openssl des-cbc -in zeros -K AC22B54854967CA9 -iv 0 -nosalt | xxd
```
- `-iv`: inicializáló vektor.
- `-nosalt`: sózás nélkül.

### TDES titkosítás
```bash
openssl des-ede-cbc -in date -k PASSWORD -out date.tdes -p
```
- `des-ede-cbc`: 3DES.

### TDES visszafejtés
```bash
openssl des-ede-cbc -d -in date.tdes -k PASSWORD
```
- `-d`: decrypt.

### Hex dump
```bash
xxd file
```
- Bináris tartalom hex nézet.

## 11) OpenSSL – RSA, aláírás, PKCS

### RSA kulcspár generálás
```bash
openssl genpkey -algorithm RSA -out key.pem -pkeyopt rsa_keygen_bits:2048
```
- `-pkeyopt rsa_keygen_bits`: kulcshossz.

### Publikus kulcs export
```bash
openssl rsa -in key.pem -pubout -out key.pem.pub
```
- `-pubout`: publikus kulcs.

### RSA kulcsméret ellenőrzés
```bash
openssl rsa -in private.pem -noout -text | head -n 1
```
- `-noout`: ne írja ki a kulcsot.

### Aláírás készítése
```bash
openssl dgst -sign key.pem -sha256 -out document.sign document
```
- `-sha256`: hash algoritmus.

### Aláírás ellenőrzés
```bash
openssl dgst -verify key.pem.pub -sha256 -signature document.sign document
```
- `-verify`: publikus kulccsal ellenőrzés.

### RSA titkosítás (public)
```bash
openssl rsautl -encrypt -pubin -inkey public.pem -in document -out document-enc1
```
- `rsautl`: régi eszköz, tanulási célra.

### RSA visszafejtés (private)
```bash
openssl rsautl -decrypt -inkey private.pem -in document-enc1
```
- `-decrypt`: visszafejtés privát kulccsal.

### RSA titkosítás (private)
```bash
openssl rsautl -sign -inkey private.pem -in document -out document-enc2
```
- `-sign`: aláírásszerű művelet.

### RSA visszafejtés (public)
```bash
openssl rsautl -inkey public.pem -pubin -in document-enc2
```
- `-pubin`: publikus kulcs bemenet.

### PKCS#8 konvertálás
```bash
openssl pkcs8 -topk8 -in ssh-generated-key -nocrypt -out pkcs8-key
```
- `-topk8`: PKCS#8 formátum.
- `-nocrypt`: titkosítás nélkül.

### PKCS#8 titkosítás
```bash
openssl pkcs8 -topk8 -in ssh-generated-key -out pkcs8-key.encrypted
```
- Jelszavas titkosítás.

### PKEYUTIL titkosítás
```bash
openssl pkeyutl -encrypt -inkey pubkey -pubin -in file -out file.enc
```
- `-encrypt`: nyilvános kulccsal titkosítás.

### PKEYUTIL visszafejtés
```bash
openssl pkeyutl -decrypt -inkey privkey -in file.enc -out file
```
- `-decrypt`: privát kulccsal visszafejtés.

## 12) OpenSSL – tanúsítványok

### CA privát kulcs
```bash
openssl genrsa -out ca.key 2048
```
- `2048`: kulcshossz bitben.

### CA CSR készítés
```bash
openssl req -new -key ca.key -out ca.csr
```
- `-new`: új CSR.
- `-subj`: nem-interaktív mezők.

### CA tanúsítvány kiállítás
```bash
openssl x509 -req -in ca.csr -out ca.crt -days 1000 -signkey ca.key
```
- `-days`: érvényesség napokban.
- `-signkey`: önaláírás.

### CA tanúsítvány ellenőrzés
```bash
openssl x509 -text -noout -in ca.crt
```
- `-text`: részletek.

### Web CSR készítés
```bash
openssl req -new -key web.key -out web.csr
```
- `-new`: új CSR.

### Web tanúsítvány kiállítás
```bash
openssl x509 -req -in web.csr -out web.crt -days 100 -CA ca.crt -CAkey ca.key -CAserial ca.srl -CAcreateserial
```
- `-CAserial`: sorozatszám fájl.
- `-CAcreateserial`: létrehozza, ha nincs.

### Trust store lista
```bash
trust list | grep HARICA
```
- `trust list`: rendszeres bizalmi tár.

### CA tanúsítvány telepítés
```bash
sudo trust anchor ca.crt
```
- `trust anchor`: új CA bejegyzés.

## 13) Kulcstranszformáció (SSH/OpenSSL)

### OpenSSH publikus kulcs export
```bash
ssh-keygen -f ssh-generated-key -e
```
- `-e`: export.

### PEM export
```bash
ssh-keygen -f ssh-generated-key -e -m pem
```
- `-m pem`: PEM formátum.

### PKCS#8 export
```bash
ssh-keygen -f ssh-generated-key.pub -e -m pkcs8
```
- `-m pkcs8`: PKCS#8.

### PKCS#1 export fájlba
```bash
ssh-keygen -f ssh-generated-key.pub -e -m pem >rsa-public-key
```
- Kimenet fájlba irányítva.

### Import OpenSSH formátumba
```bash
ssh-keygen -i -f rsa-public-key -m pem
```
- `-i`: import.

### OpenSSL pubkey export
```bash
openssl rsa -in ssh-generated-key -pubout
```
- `-pubout`: publikus kulcs.

### PKCS#1 -> PKCS#8 konverzió
```bash
openssl rsa -RSAPublicKey_in -in rsa-public-key -pubout >public-key
```
- `-RSAPublicKey_in`: PKCS#1 bemenet.

## 14) Windows biztonsági eszközök

### Audit házirend
```bash
gpedit.msc
```
- Lokális csoportházirend-szerkesztő.

### Event Viewer
```bash
eventvwr.msc
```
- Eseménynapló megtekintés.

## 15) Extra – összefoglaló és kiegészítések

### chmod számjegyek (owner/group/other)
| Szám | Jelentés | Részletek |
| --- | --- | --- |
| 0 | --- | nincs jog |
| 1 | --x | végrehajtás / belépés könyvtárba |
| 2 | -w- | írás |
| 3 | -wx | írás + végrehajtás |
| 4 | r-- | olvasás |
| 5 | r-x | olvasás + végrehajtás |
| 6 | rw- | olvasás + írás |
| 7 | rwx | teljes jog |

Példa:
```bash
chmod 750 /opt/app
```
- `7` owner: rwx, `5` group: r-x, `0` other: ---.

### Fájlrendszer létrehozás /dev/sdc eszközön (partíció: /dev/sdc1)
```bash
lsblk -f
sudo parted /dev/sdc --script mklabel gpt mkpart primary ext4 1MiB 100%
sudo partprobe /dev/sdc
sudo mkfs.ext4 -L data /dev/sdc1
sudo mkdir -p /mnt/data
sudo mount /dev/sdc1 /mnt/data
```
- `mklabel gpt`: GPT partíciós tábla.
- `mkpart ... 1MiB 100%`: igazított partíció.
- `partprobe`: kernel partíció tábla frissítése.

Tartós csatolás (UUID-val):
```bash
blkid /dev/sdc1
echo "UUID=<UUID> /mnt/data ext4 defaults 0 2" | sudo tee -a /etc/fstab
```

### Fájl titkosítása publikus kulccsal (PEM a vágólapon)
```bash
cat > /tmp/public.pem
# ide másold a kulcsot, BEGIN PUBLIC KEY ... END PUBLIC KEY
openssl pkeyutl -encrypt -pubin -inkey /tmp/public.pem -in secret.txt -out secret.txt.enc \
	-pkeyopt rsa_padding_mode:oaep -pkeyopt rsa_oaep_md:sha256
```
- `-pubin`: publikus kulcsot olvas.
- `-pkeyopt rsa_padding_mode:oaep`: modern padding.

Macen vágólapról:
```bash
pbpaste > /tmp/public.pem
```

### Jelszó lejárat dátum szerint
```bash
chage -E 2026-12-31 user1
```
- `-E`: fiók lejárati dátum.

Hasznos kiegészítők:
```bash
chage -W 7 user1
passwd -e user1
```
- `-W`: figyelmeztetés napokban.
- `-e`: azonnali jelszócsere kikényszerítése.

## 16) Szövegfeldolgozás

### grep – mintakeresés
```bash
grep "error" logfile.txt
```
- Sorok, amelyekben az "error" szó szerepel.

```bash
grep -i "ERROR" logfile.txt
```
- `-i`: nagy/kisbetű-érzéketlen.

```bash
grep -v "^#" config.conf
```
- `-v`: inverz (nem egyezik).

```bash
grep -c "pattern" file.txt
```
- `-c`: egyezések száma.

```bash
grep -n "error" logfile.txt
```
- `-n`: sorszám kiírása.

```bash
grep -E "^error|^warn" logfile.txt
```
- `-E`: kiterjesztett regex.

```bash
grep -r "password" /etc/
```
- `-r`: rekurzív könyvtárban.

### sed – stream editor
```bash
sed 's/old/new/' file.txt
```
- `s`: helyettesítés (első előfordulás/sor).

```bash
sed 's/old/new/g' file.txt
```
- `/g`: globális (összes előfordulás).

```bash
sed -i 's/old/new/g' file.txt
```
- `-i`: fájl helybeni szerkesztése.

```bash
sed '5d' file.txt
```
- `5d`: 5. sor törlése.

```bash
sed '1,10d' file.txt
```
- `1,10d`: sorok 1-10 törlése.

```bash
sed -n '5p' file.txt
```
- `-n 5p`: csak az 5. sor kiírása.

### awk – oszlop-alapú feldolgozás
```bash
awk '{print $2}' file.txt
```
- `$2`: második oszlop.

```bash
awk -F: '{print $1}' /etc/passwd
```
- `-F:`: mezőelválasztó `:`.

```bash
awk '{sum+=$1} END {print sum}' numbers.txt
```
- Összegzés.

```bash
awk 'NR==5' file.txt
```
- `NR==5`: csak az 5. sor.

```bash
awk 'NF>0' file.txt
```
- `NF>0`: üres sorok szűrése.

### cut – oszlop kiválasztás
```bash
cut -d: -f1 /etc/passwd
```
- `-d:`: mezőelválasztó.
- `-f1`: 1. mező.

```bash
cut -c1-10 file.txt
```
- `-c1-10`: karakterek 1-10.

### tr – karakter csere
```bash
echo "hello" | tr a-z A-Z
```
- Kisbetűt nagybetűvé.

```bash
cat file.txt | tr -d '\r'
```
- `-d`: karakter törlése (CRLF ➜ LF).

## 17) Tömörítés és archíválás

### tar – archiválás (nem tömörít alapból)
```bash
tar -cvf archive.tar file1 file2 dir/
```
- `-c`: új archívum.
- `-v`: verbose.
- `-f`: fájl név.

```bash
tar -xvf archive.tar
```
- `-x`: kicsomagolás.

```bash
tar -tvf archive.tar
```
- `-t`: tartalom listázása.

### gzip – tömörítés
```bash
tar -czvf archive.tar.gz file1 file2
```
- `-z`: gzip tömörítés.
- `.tar.gz`: összetett archívum.

```bash
tar -xzvf archive.tar.gz
```
- Kicsomagolás gzipped archívumból.

```bash
gzip file.txt
```
- `file.txt` ➜ `file.txt.gz`.

```bash
gunzip file.txt.gz
```
- Kitömörítés.

### zip – Windows-típusú tömörítés
```bash
zip -r archive.zip file1 file2 dir/
```
- `-r`: rekurzív.

```bash
unzip archive.zip
```
- Kicsomagolás.

## 18) Fájl keresés

### find – rugalmas keresés
```bash
find /home -name "*.txt"
```
- `-name`: név minta.

```bash
find /home -type f -size +10M
```
- `-type f`: csak fájlok.
- `-size +10M`: nagyobb mint 10 MB.

```bash
find /var -name "*.log" -mtime +30
```
- `-mtime +30`: módosítva több mint 30 napja.

```bash
find /home -perm 777
```
- `-perm 777`: pontosan 777 jogosultság.

```bash
find / -name "*.txt" -exec grep -l "password" {} \;
```
- `-exec`: parancs futtatása minden találatnál.

```bash
find / -name core -type f -delete
```
- `-delete`: fájlok törlése.

## 19) Folyamatok és erőforrások

### ps – futó folyamatok
```bash
ps aux
```
- Összes folyamat részletes info.

```bash
ps ef
```
- Teljes parancspályával.

```bash
ps aux | grep ssh
```
- SSH-val kapcsolódó folyamatok.

### top – valós idejű monitorozás
```bash
top -u user1
```
- `-u`: csak adott felhasználó folyamatai.

```bash
top -p 1234
```
- `-p`: adott PID.

### lsof – nyitott fájlok
```bash
lsof -i :8080
```
- `-i :8080`: mely folyamatok hallgatóznak 8080-on.

```bash
lsof -u user1
```
- `-u`: felhasználó nyitott fájljai.

```bash
lsof /var/log/syslog
```
- Mely folyamatok használják ezt a fájlt.

### systemctl – szolgáltatások
```bash
systemctl start httpd
```
- Szolgáltatás indítása.

```bash
systemctl enable httpd
```
- Automatikus indítás reboot után.

```bash
systemctl status httpd
```
- Állapot ellenőrzése.

```bash
systemctl stop httpd
```
- Leállítás.

### journalctl – rendszernaplók
```bash
journalctl -u httpd
```
- `-u`: adott szolgáltatás naplói.

```bash
journalctl -n 50
```
- `-n 50`: utolsó 50 sor.

```bash
journalctl -f
```
- `-f`: valós idejű követés.

```bash
journalctl --since "2026-05-01" --until "2026-05-08"
```
- Dátumtartomány.

## 20) Hálózat és kapcsolat

### ss – socket statisztika (feljebb mint netstat)
```bash
ss -tlnp
```
- `-t`: TCP.
- `-l`: csak listen.
- `-n`: numerikus.
- `-p`: process.

```bash
ss -an | grep ESTABLISHED
```
- Aktív kapcsolatok.

### netstat – hálózati statisztika (örökölt)
```bash
netstat -tulnp
```
- `-u`: UDP.
- `-t`: TCP.
- `-l`: listen.
- `-n`: numerikus.
- `-p`: process.

### nmap – hálózati feltérképezés
```bash
nmap localhost
```
- Alapvető portra letapogatás.

```bash
nmap -p 80,443 localhost
```
- `-p`: adott portok.

```bash
nmap -sV localhost
```
- `-sV`: verzió felderítés.

```bash
nmap -O localhost
```
- `-O`: OS felderítés (gyök szükséges).

### tcpdump – csomag rögzítés
```bash
sudo tcpdump -i eth0
```
- `-i`: interfész.

```bash
sudo tcpdump -i eth0 -w capture.pcap
```
- `-w`: fájlba írás.

```bash
sudo tcpdump -i eth0 'tcp port 80'
```
- Szűrés portok szerint.

```bash
sudo tcpdump -i eth0 -A
```
- `-A`: ASCII kimenet.

### curl – webhely lekérés
```bash
curl http://example.com
```
- Tartalom letöltése.

```bash
curl -o filename.html http://example.com
```
- `-o`: fájlba mentés.

```bash
curl -H "Authorization: Bearer TOKEN" http://api.example.com
```
- `-H`: egyéni fejléc.

```bash
curl -X POST -d "key=value" http://api.example.com
```
- `-X POST`: POST kérés.

### wget – letöltés
```bash
wget http://example.com/file.tar.gz
```
- Fájl letöltése.

```bash
wget -r http://example.com
```
- `-r`: rekurzív letöltés.

## 21) Fájlok közötti átvitel

### scp – SSH-n keresztüli másolás
```bash
scp file.txt user@host:/tmp/
```
- Fájl másolása locálról távoli hostra.

```bash
scp user@host:/tmp/file.txt ./
```
- Fájl másolása távoli hostróllocálra.

```bash
scp -r dir/ user@host:/tmp/
```
- `-r`: könyvtár másolása.

```bash
scp -P 2222 file.txt user@host:/tmp/
```
- `-P`: egyéni port.

### rsync – szinkronizálás
```bash
rsync -avz localdir/ user@host:remotedir/
```
- `-a`: archív (perms, times).
- `-v`: verbose.
- `-z`: tömörítés.

```bash
rsync -av --delete localdir/ remotedir/
```
- `--delete`: törlés távolendőből.

## 22) Hasítás és kódolás

### md5sum – MD5 hasítás
```bash
md5sum file.txt
```
- MD5 hash.

```bash
md5sum file.txt > file.md5
md5sum -c file.md5
```
- Ellenőrzés.

### sha256sum – SHA256 hasítás
```bash
sha256sum file.txt
```
- SHA256 hash.

### base64 – kódolás/dekódolás
```bash
echo "hello" | base64
```
- Base64 kódolás.

```bash
echo "aGVsbG8=" | base64 -d
```
- `-d`: dekódolás.

```bash
base64 -w 0 < file.bin
```
- `-w 0`: sorvégi törést tiltás.

## 23) GPG – PGP titkosítás

### GPG kulcspár generálás
```bash
gpg --full-generate-key
```
- Interaktív kulcsgenerálás.

### Public kulcs export
```bash
gpg --armor --export user@example.com > public.asc
```
- `-armor`: ASCII formátum.
- `--export`: publikus kulcs.

### Private kulcs export
```bash
gpg --armor --export-secret-keys user@example.com > private.asc
```
- `--export-secret-keys`: privát kulcs (óvatosan!).

### Fájl titkosítása GPG-vel
```bash
gpg --encrypt --recipient user@example.com file.txt
```
- `file.txt.gpg` kimenet.

### Fájl visszafejtése
```bash
gpg --decrypt file.txt.gpg > file.txt
```
- Jelszó kérdez.

### Aláírás (digitális aláírás)
```bash
gpg --sign file.txt
```
- `file.txt.gpg` (bináris).

```bash
gpg --clearsign file.txt
```
- `--clearsign`: olvasható szöveg + aláírás.

## 24) Lemez és tárolás

### df – lemez felhasználás
```bash
df -h
```
- `-h`: emberi olvasható (MB, GB).

```bash
df -i
```
- `-i`: i-node felhasználás.

### du – könyvtár méret
```bash
du -sh /home/
```
- `-s`: összesen.
- `-h`: emberi olvasható.

```bash
du -sh *
```
- Összes alkönyvtár mérete.

### fdisk – partíció szerkesztő
```bash
sudo fdisk -l
```
- `-l`: összes meghajtó listázása.

```bash
sudo fdisk /dev/sdb
```
- Interaktív szerkesztés.

### lvm – logikai kötetkezelés
```bash
lvs
```
- Logikai kötetek listázása.

```bash
pvs
```
- Fizikai kötetek listázása.

## 25) Rendszer információ

### uname – rendszer infó
```bash
uname -a
```
- `-a`: összes infó.

```bash
uname -r
```
- `-r`: kernel verzió.

### lscpu – CPU infó
```bash
lscpu
```
- CPU modellek és magok.

### lsblk – blokkezközök
```bash
lsblk -f
```
- `-f`: fájlrendszer és UUID.

### hostnamectl – hostnév
```bash
hostnamectl
```
- Hostnév és OS infó.

## 26) Ütemezés és automatizálás

### cron – ismétlődő feladatok
```bash
crontab -e
```
- `-e`: szerkesztés.

```bash
crontab -l
```
- `-l`: listázás.

Cron szintaxis: `min hour day month weekday command`

```bash
0 2 * * * /usr/local/bin/backup.sh
```
- Minden nap 02:00-kor.

```bash
*/15 * * * * /usr/local/bin/check.sh
```
- 15 percenként.

```bash
0 0 * * 0 /usr/local/bin/weekly.sh
```
- Vasárnap 00:00-kor.

### at – egyszeri feladat
```bash
echo "command" | at 10:30 PM
```
- Ma 22:30-kor.

```bash
atq
```
- Ütemezett feladatok listázása.

```bash
atrm 1
```
- Feladat eltávolítása.

## 27) Csomag kezelés

### dnf (Fedora/RHEL/CentOS)
```bash
dnf list installed
```
- Telepített csomagok listázása.

```bash
dnf search keyword
```
- Csomag keresése.

```bash
dnf install package
```
- Telepítés.

```bash
dnf update
```
- Összes csomag frissítése.

```bash
dnf remove package
```
- Csomag eltávolítása.

### apt (Debian/Ubuntu)
```bash
apt list --installed
```
- Telepített csomagok.

```bash
apt search keyword
```
- Keresés.

```bash
apt install package
```
- Telepítés.

```bash
apt update && apt upgrade
```
- Frissítés.

## 28) Száraz tanácsokarok és bevált gyakorlatok

### Biztonságos jelszó generálás
```bash
openssl rand -base64 32
```
- Erős jelszó.

```bash
date +%s | sha256sum | base64 | head -c 32
```
- Alternatív módszer.

### Fájl biztonságos törlése
```bash
shred -vfz -n 3 sensitive-file.txt
```
- `-v`: verbose.
- `-f`: force.
- `-z`: nullákkal felülír végén.
- `-n 3`: 3-szor.

### Biztonsági audit jogosultságok
```bash
find / -perm -4000 -type f
```
- SETUID bitek.

```bash
find / -perm -2000 -type f
```
- SETGID bitek.

```bash
find / -perm -1000 -type f
```
- Sticky bit.

### Fájlok törlése adott tulajdonos szerint
Ha egy mappában törölni szeretnénk az összes fájlt, amelynek tulajdonosa az `operator` felhasználó, először listázzuk őket, majd töröljük óvatosan:

Biztonságos lista (ellenőrzés):
```bash
find /path/to/dir -maxdepth 1 -type f -user operator -print
```

Automatikus törlés (VIGYÁZAT: nem visszafordítható):
```bash
find /path/to/dir -maxdepth 1 -type f -user operator -exec rm -f {} \;
```

Magyarázat:
- `-maxdepth 1`: csak a megadott mappa közvetlen fájljai.
- `-type f`: csak fájlokat talál.
- `-user operator`: csak az `operator` tulajdonúakat.
- `-exec rm -i {}`: interaktív törlés (ajánlott).

