Tudjuk már értelmezni az `ls` parancs alap lista kimenetének 1+9 betűjét? Még nem egészen. A rendszer még egy oktális számjeggyel (még 3 bittel) kiegészül. Ezek a bináris ábrázolásban  helyiérték szerint csökkenően:

- _set uid_
- _set gid_
- _sticky_

Persze ezek jelentése szintén más közönséges állományokon és katalógusokon. Ráadásul a jelentés időben is változott, de mindig az _execute_ joghoz kapcsolódik. A mai rendszereken 3 probléma megoldásra valók:

Láttuk, hogy a katalógusok esetében a _write_ jog problémás lehet, mert a katalógusban lévő állományok törlését is engedélyezi. Ezt küszöböli ki a _sticky bit_ alkalmazása egy katalóguson. Ha használatban van egy katalóguson, akkor a benne lévő állományt csak annak a tulajdonosa, vagy a katalógus tulajdonosa törölheti.

A _set gid_ bitet valamely felhasználói csoporttal megosztott katalógus esetében használják. Ha használatban van a katalóguson, akkor az abba bekerülő (újonnan létrehozott) állományok az _owner group_ attribútumot a katalógustól öröklik meg, ahelyett hogy a létrehozó processz csoportazonosítóját (_effective group id_) kapnák.

Eddig azt láttuk, hogy a processz az őt indító jogosultságait örökli (_effective user id_, _effective group id_). Ezt a működést írja felül a _set uid_ és _set gid_ bit alkalmazása egy futtatható állományon. Előbbi esetén az _effective user id_ értéke az állomány tulajdonosa (_owner user_), utóbbi esetén az _effective group id_ értéke az állományhoz bejegyzett csoport (_owner group_) lesz. Tipikus példa a jelszóváltás lehetővé tevő alkalmazás. A jelszavakat (azok hash értékét) tároló állomány (`/etc/shadow`) esetén egy felhasználónak sincs semmilyen jogosultsága sem. Ezért csak a rendszergazda olvashatja, szerkesztheti. Akkor mégis hogyan változtathat egy felhasználó jelszót? Úgy, hogy az erre szolgáló alkalmazás (`passwd`) tulajdonosa a rendszergazda, és a _set uid_ bit is be van állítva, így futtatáskor a process a rendszergazda jogkörével rendelkezve már hozzáfér a szükséges erőforráshoz. Ehhez persze nagyon meg kell bízni abban, hogy az alkalmazás jól működik, és csak azt enged manipulálni a felhasználónak, ami tényleg hozzá tartozik.

Két eset maradt ki. A fentieket továbbgondolva a _set uid_ bit alkalmazása katalóguson működhetne úgy, hogy az új állományok a katalógus _owner user_ attribútumát örököljék, de ez igen furcsa helyzetet teremtene: az épp létrehozott állományt azonnal el is venné attól, aki készítette. Ezért a _set uid_ bit ugyan egy katalóguson beállítható, de a hatástalan marad.

A _sticky bit_et régi rendszerek futtatható állományokon használták. Innen származik a neve is. A feladata az volt, hogy jelezze, a programot a végrehajtást követően is a memóriában kell tartani, hogy legközelebb gyorsabban indítható legyen (ne kelljen beolvasni ismét a lemezről). A mai rendszerek már nem így működnek, így ez megint csak egy beállítható, de hatástalan bit maradt.

Az új oktális jegy (3 bit) segítségével fontos biztonsági kérdésekre kapunk választ. Használatuk esetén körültekintéssel kell eljárni figyelembe véve minden következményt.

Ábrázolásuk oktálisan a legmagasabb helyiértéken történik, a karakteres reprezentációban pedig az _execute_ jogosultság `x` betűjének helyén jelennek meg rendre `s`, `s` és `t` betűként, hiszen ezeknek az _execute_ joggal együtt van igazán szerepe. (Nagy betűt itt akkor látunk, ha az _execute_ jog nincs megadva, ami elég furcsa eset és hibás beállításra utalhat.)

Mi történik új állomány létrehozásakor? Erre az állományt létrehozó folyamat _umask_ attribútuma ad választ, amely rögzíti, hogy mely jogosultságokat kell megtagadni (törölni) az adott állományon értelmezhető jogok közül. (Közönséges állományon `0666` katalógus esetén `0777` a kiindulási alap, ebből töröljük az _umask_-ban beállított biteket).