A fájlhozzáférési rendszer vizsgálatát a Linux fájlrendszerekben alapvető jogosultságok tanulmányozásával kezdjük. Első lépésben egy-egy állomány vonatkozásában 3 jogosultságot _read_, _write_, _execute_ és 3 felhasználói csoportot különböztessünk meg: _user_, _group_, _other_. Attól függően, hogy milyen állományról is beszélünk, a fenti jogosultságok jelentése különböző. Reguláris (közönséges) állományok esetében ezek jelentése a következő:

- _read_: az állomány tartalma olvasható
- _write_: az állomány tartalma megváltoztatható
- _execute_: az állomány (önálló alkalmazásként vagy interpreter segítségével) futtatható

Katalógusok esetében ugyanezen jogosultságok jelentése változik:

- _read_: a katalógus tartalma lekérdezhető
- _write_: a katalógus tartalma megváltoztatható
- _execute_: a katalógus felhasználható (elérési útban és munkakönyvtárként)

A három felhasználói kör:

- _(owner) user_: az állományhoz bejegyzett tulajdonos felhasználó (1db)
- _(owner) group_: az állományhoz bejegyzett tulajdonos felhasználói csoport (1db)
- _other_: bárki más (a fentieken kívül)

A háromféle jogosultság a három felhasználói kör számára 9 biten ábrázolható. Ezt a 9 bitet hármasával csoportosítva háromjegyű oktális reprezentációt kapunk. A háromból a legmagasabb helyiértékű oktális számjegy az _owner user_, az azt követő az _owner group_, végül a legalacsonyabb helyiértékű oktális jegy az _other_ kategóriába eső felhasználók jogosultságait írja le. Egy-egy oktális jegy 3 bitjéből a legmagasabb helyiértéken 1-es akkor szerepel, ha _read_ jogosultsággal bír a felhasználói kör, következő helyiértéken akkor, ha _write_ jogosultsággal, míg a legalacsonyabb helyiérték akkor, ha _execute_ jogosultsággal. A felhasználói köröket és a jogosultságokat kezdőbetűkkel rövidítik, melyek így helyiérték szerint csökkenő sorrendben `u,g,o` (az oktális számjegyeket tekintve) és `r,w,x` (egy-egy oktális számjegy bináris reprezentációjában). Több alkalmazás is használja a bináris ábrázolást olyan formában, hogy a 0 számjegyet '`-`' karakterrel, az 1 számjegyet a bináris helyiértéknek megfelelő betűvel helyettesíti. Ilyet láthatunk például a `stat` vagy az `ls` parancsok kimenetében. Mivel a 3 jogosultság jelentése függ az állomány jellegétől, az előbb tárgyalt 9 karakter egy erre utaló betűvel balról kiegészül. Ez lehet például ‘`-`’ egy közönséges állomány ‘`d`’ egy katalógus (directory) esetében.

Egy állomány jogosultságát (például a `chmod` parancs segítségével) az állomány tulajdonosa módosíthatja. Az állomány tulajdonosát a rendszergazda (_root user_) állíthatja be (például a `chown` paranccsal). A rendszergazda anélkül is megváltoztathat állományjogosultságot, hogy azt a tulajdonába kellene vennie. A rendszergazda itt egy fontos kivétel, akire a beállított jogosultság szabta megszorítások nem vonatkoznak.

Amikor egy felhasználó műveletet végez egy állománnyal, akkor a célnak megfelelő valamely alkalmazást hívja segítségül (futtatja). Az alkalmazás a legegyszerűbb esetben az azt futtató felhasználó jogosítványaival rendelkezik, annak nevében dolgozik, pontosabban örökli az alábbiakat:

- effective user id
- effective group id
- supplementary group ids

Ezt a viselkedést ugyanakkor meg lehet változtatni. Hogy mik is az aktuális értékei a fentieknek az például az `id` paranccsal lekérdezhető. Biztonsági szempontból most számunkra a szélsőséges esetek érdekesek. Mielőtt ebbe belemennénk, pontosítsuk mit is jelentenek az állományjogosultságok. Jó közelítés, ha katalógusokat olyan állományoknak képzeljük, melyek állománynevek és a hozzájuk tartozó állományazonosítók (_inode_) listájából állnak.

- _Read_ jogosultsággal rendelkezve egy katalóguson kiolvasva annak tartalmát megismerhető a benne szereplő állományok neve.
- _Execute_ jogosultsággal rendelkezve egy katalógust használhatjuk elérési útban vagy aktuális munkakönyvtárként, így elérjük (hivatkozhatjuk) a benne szereplő állományokat.

Ritkán látunk olyan katalógust, ahol ez a két jogosultság ne járna együtt. Ugyanakkor jegyezzük meg, hogy nem kell, hogy _read_ jogunk legyen egy katalógushoz, ha fel akarjuk dolgozni a benne található állományt, feltéve, hogy magához az állományhoz is megvan a megfelelő jogosultságunk. A _read_ jog csak arra kell, hogy megnézhessük, milyen állománynevek vannak a katalógusban, de ha a feldolgozandó állomány nevéről már valahogy tudomást szereztünk, akkor ez nem olyan fontos. _Execute_ jog nélkül ugyanakkor a katalógus gyakorlatilag használhatatlan. A benne lévő állományokat nem érjük el.

_Write_ jogosultság birtokában a katalógus – mint állománylista – tartalmát módosíthatjuk. Ez nem csak az új bejegyzések létrehozását teszi lehetővé, hanem a meglévők módosítását és törlését. Ennek eredményeképp a katalógusban szereplő állományok átnevezhetőek sőt le is törölhetők még akkor is ha a tekintett állományhoz semmilyen jogosultsággal sem rendelkezünk. Ezért a bárki (_other_) számára írási joggal rendelkező katalógusok elég ritkák, hiszen (egyéb védelem nélkül) azok tartalmát jószerivel bárki törölhetné. (Ezt azért hamarosan még pontosítjuk.)

Futtatható (bináris) programokat tartalmazó állományok felhasználásához sem kell a _read_ jog, a program az _execute_ jogosultság birtokában elindítható. Figyeljünk ugyanakkor a script fájlokra, melyeket szintén futtathatónak tekintünk, de ezeket valójában egy interpreter dolgozza fel. Az interpreternek ugyanakkor épp a _read_ jogra van szüksége erre. Így a script fájlok esetében a két jog megint csak tipikusan együtt jár. Ha csak _read_ jogunk van egy script fájlhoz, attól azt még a megfelelő interpreter használatával lefuttathatjuk, ha magát az interpretert futtathatjuk, persze magunk felparaméterezve azt. Ugyanakkor kényelmesebb az operációs rendszerre bízni az interpreter indítását azzal, hogy a script fájlra _execute_ jogot is teszünk.

Tipikusan az állományjogosultsághoz kapcsolódó korábban említett három felhasználói kör közül az első (az _owner user_) rendelkezik a legtöbb, míg az utolsó (_other_) a legkevesebb jogosultsággal. Ettől eltérő eset ritka, de előállítható. Például állítsunk elő olyan szituációt, ahol a tulajdonos csoport (_owner group_) tagjai több joggal bírnak mint maga az állomány tulajdonos (_owner user_). Ez akkor érdekes igazán, amikor a felhasználó aki hozzáférést kezdeményez egyben tulajdonos és a tulajdonos csoport tagja is. Ekkor a jogosultságok nem adódnak össze. A tulajdonos csak azt teheti meg amit a tulajdonos (_owner user_) beállításai engedélyeztek számára. A pontos döntési algoritmust hamarosan látni fogjuk.