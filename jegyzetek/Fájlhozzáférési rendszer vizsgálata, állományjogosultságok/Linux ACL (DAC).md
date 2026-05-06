Maradjunk továbbra is az `ls` parancs alap listakimenténél. Egyetlen karaktert kell csak tovább menni, hogy egy új világ nyíljon meg számunkra ami a felhasználói jogok testreszabhatóságát illeti. Ha az első 10 karaktert egy pont zárja abból két következtetést vonhatunk le: az elején rögzített 3 felhasználói körön kívül nincs más felhasználónak vagy csoportnak további jogosultsága, de lehetne (a rendszer támogatja). A _read_, _write_ és _execute_ jogok további felhasználókhoz és csoportokhoz is hozzárendelhetők ACL (access control list) bejegyzések segítségével. ACL bejegyzésekből 3-at már ismerünk, de nézzük a teljes képet:

- ACL_USER_OBJ: az _owner user_ `r`, `w` ,`x` joga
- ACL_USER: tetszőleges felhasználó `r`, `w` ,`x` joga
- ACL_GROUP_OBJ: az _owner group_ `r`, `w` ,`x` joga
- ACL_GROUP: tetszőleges csoport `r`, `w` ,`x` joga
- ACL_MASK: az előbbi 3 bejegyzést korlátozó (maximáló) beállítás
- ACL_OTHER: az other r, w,x joga

Az ACL bejegyzésekhez kapcsolódó parancsok: `getfacl`, `setfacl`.

Az ACL_MASK egy alap esetben automatikusan számított érték ami az ACL_USER, ACL_GROUP_OBJ és ACL_GROUP bejegyzésekben foglalt jogokat összegzi, de akár külön is állítható (annak érdekében, hogy az ezen bejegyzésekben megadott jogokat korlátozza). Az ACL_MASK a két szélsőséges esetre (ACL_USER_OBJ, ACL_OTHER) nem vonatkozik. Ha egy állomány rendelkezik ACL_USER, ACL_GROUP, ACL_MASK bejegyzéssel, akkor valójában az `ls` parancs alapértelmezett lista kimenetében az ACL_MASK látszik az ACL_GROUP_OBJ helyén.

Az, hogy adott módon hozzáférhetünk-e egy állományhoz az alábbiak algoritmus szerint dől el:

1. ha a hozzáférést kezdeményező folyamat effective user id értéke megegyezik az állomány owner user értékével, akkor a hozzáférés engedélyezéséről a döntést az ACL_USER_OBJ alapján kell meghozni.
2. ha a hozzáférést kezdeményező folyamat effective user id értéke megegyezik valamely az állományhoz tartozó ACL_USER bejegyzésben megnevezett felhasználóval, akkor a hozzáférés engedélyezéséről a döntést ezen ACL_USER értéke alapján kell meghozni figyelembe véve az ACL_MASK (a jogosultságok körét esetleg szűkítő) értékét is.
3. ha a hozzáférést kezdeményező folyamat effective group id vagy supplementary group ids értékei közül legalább egy megegyezik az állomány owner group értékével vagy valamely ACL_GROUP bejegyzésben megnevezett csoporttal, akkor - figyelembe véve az ACL_MASK (a jogosultságok körét esetleg szűkítő) értékét is - a jogosultság pontosan akkor engedélyezett, ha a fentiek között van legalább egy ACL_GROUP vagy ACL_GROUP_OBJ bejegyzés amely ezt engedélyezi.
4. a fentiek hiányában a hozzáférés engedélyezéséről a döntést az ACL_OTHER értéke alapján kell meghozni.

Fontos, ahogy azt korábban is láttuk, hogy a fentiek alapján a jogosultságok nem adódnak össze. Ha az ACL_OTHER bejegyzés többet enged meg mint az ACL_USER_OBJ akkor ez az állomány tulajdonosát az korlátozni fogja az utóbbiban beállítottakra. Ne felejtsük el azt sem, hogy a root felhasználóra az ACL nem vonatkozik. Annak jogosultságait nem korlátozza.

A Linux ACL DAC (Discretionary Access Control) modellt alkalmaz, melynek lényege, hogy az objektum (esetünkben az állományok) tulajdonosa dönt a jogosultságokról. Kivételt a rendszergazda (_root_) képez, akinek jogosultságait ez nem korlátozza.

Mi történik az ACL bejegyzésekkel új állomány létrehozásakor? Az ACL bejegyzéseket a tartalmazó katalógusból származtathatjuk, ha volt alapértelmezett (_default_) ACL beállítás a katalógusban.

### Fájlrendszer szintű korlátok

Nem minden fájlrendszer támogatja az ACL bejegyzéseket. A helyzetet bonyolítja, hogy a Linux operációs rendszer által kezelt fájlrendszer-hierarchia (vagy katalógusstruktúra) több különböző eszközön (például lemezen) lévő fájlrendszerből tevődik össze. Ezen fájlrendszereket a fájlrendszer-hierarchia különböző katalógusaiba csatoljuk (mount). Azt, hogy egy fájlrendszerrel mit is tehetünk meg, a csatoláskor korlátozni lehet. Értelemszerű, hogy egy csak olvasható meghajtón (mondjuk egy CD-lemezen) nem hajtható végre írási művelet. Ugyanakkor az elvégezhető műveletekre való korlátozás a csatoláskor is beállítható. Számunkra most a legfontosabbak:

- `ro`: a fájlrendszer csak olvasható
- `noexec`: bináris fájlok nem futtathatók (az _execute_ jog megléte ez esetben hatástalan)
- `nosuid`: _setuid_ és _setgid_ bitek hatástalanok

Gyakran előfordul, hogy a felhasználók saját (home) katalógusai külön lemezre és külön fájlrendszerre kerülnek, amely csatolása a _noexec_ opció használatával történik. Ezzel megakadályozható, hogy a felhasználók által letöltött programok indíthatóak legyenek (hiába tesz rá valaki _execute_ jogosultságot), a felhasználóink így csak az előre telepített alkalmazásokat használhatják.