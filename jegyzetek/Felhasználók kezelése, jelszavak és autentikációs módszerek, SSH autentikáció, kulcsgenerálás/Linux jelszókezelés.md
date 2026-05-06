Linux rendszer esetében megszokhattuk, hogy a konfiguráció gyakran szöveges állományok szerkesztésével valósítható meg. A felhasználók esetén a legalapvetőbb a `passwd`, `group` és a `shadow` fájl megismerése. Sok felhasználóval kapcsolatos művelet (id, useradd, usermod, chage) ezen fájlokat olvassa vagy szerkeszti át. Mindhárom szöveges állomány táblázatként értendő melynek sorai egybeesnek az állomány soraival és oszlopait ‘`:`’ karakter választja el.

 A felhasználó alapvető adatai passwd fájlban mindenki számára elérhetők:

1. a felhasználó neve
2. a felhasználó jelszava
3. a felhasználó azonosítója (uid)
4. a felhasználó elsődleges csoportja (gid)
5. megjegyzés
6. saját (kezdő) katalógus (home)
7. héjprogram (_shell_)

Mivel ez az állomány minden felhasználó számára olvasható, jelszavakat már nem szokás ebben tárolni (még titkosítva sem), hanem erre a shadow állomány használatos. A rendszergazda (superuser) felhasználó neve root és azonosítója 0. (A felhasználónév  elvben megváltoztatható, de ez nem ajánlott.) Minden felhasználó rendelkezik elsődleges csoporttal, ez az ami a `passwd` fájlban van rögzítve, és rendelkezhet tetszőlegesen sok további (_supplementary_) csoporttagsággal, amik már a group állományban vannak tárolva. A megjegyzés mező tipikusan a felhasználó valódi neve, a home katalógus pedig alapesetben a `/home` katalóguson belül elhelyezkedő alkatalógus melynek neve többnyire megegyezik a felhasználó nevével. A superuser home katalógusa általában külön helyen ‘`/root`’ helyezkedik el. A megadott héjprogram (_shell_) az az alkalmazás, amit sikeres bejelentkezést követően indít az operációs rendszer a felhasználó számára. Technikai, a rendszerbe valójában be nem jelentkező, felhasználók esetén héjprogram helyett nem létező vagy érdemben nem használható alkalmazás áll (például `/bin/false` vagy `/bin/nologin`).

A mai rendszerekben a helyben tárolt jelszavakról a shadow fájl gondoskodik. A felhasználó nevét követi a jelszó általában sózott hash értéke, majd a jelszóval kapcsolatos beállítások következnek. Ezek:

- az utolsó jelszóváltás dátuma
    
- a jelszóváltások közt előírt minimális idő napokban
    
- a jelszóváltások közt előírt maximális idő napokban
    
- a jelszó váltásra figyelmeztető időszak hossza napokban
    
- inaktivitási idő napokban
    
- a felhasználói fiók lejáratának dátuma
    

Ha van megadva jelszóváltások közti maximális idő, akkor annak leteltével a jelszó elavulttá válik, bejelentkezésre még használható, de aztán azonnal meg is kell változtatni, különben a bejelentkezési folyamat megszakad. Az értéke napokban értendő, ha 1 akkor a jelszóváltást követő napon még nem járt le a jelszó. Speciális eset a 0 és a -1 érték. Előbbi esetén a jelszó sosem jár le (a lejárati idő végtelen) utóbbi esetén a rendszer nem kezeli a jelszó korát (ami megengedi, hogy az utolsó jelszóváltás dátuma mező üres legyen). 

A figyelmeztetési időszak megmutatja, hogy a jelszó lejárta előtti hányadik naptól kell figyelmeztetni bejelentkezéskor a felhasználót a jelszó lejáratának közeledtére. Speciális eset a 0 és -1 érték melyek esetén nincs figyelmeztetés. Egyes audit rendszerek a 0 értéket hibás beállításnak jelzik (mert ez olvasható úgy, hogy ez a funkció be van állítva, aktív, de hatástalan) és figyelmeztetést küldenek erről.

Az inaktivitási idő esetén már sokkal szembeötlőbb a különbség. Ha nincs beállítva, akkor a jelszó a lejárat után is használható, de azonnal jelszót is kell váltani, ha viszont be van állítva (akár 0 értékre) akkor ha a lejárati idő után ez az intervallum is eltelt, akkor bejelentkezés már nem lehetséges (még jelszóváltás céljából sem). A 0 érték olvasata itt az, hogy a lejárt jelszó azonnal használhatatlanná válik. Az 1 olvasata az, hogy a lejárat napján jelszócsere céljából a jelszavas belépés még használható. Elsőre talán nem logikus, de ez nem csak a jelszavas bejelentkezésre vonatkozik. Nem csupán a jelszó válik inaktívvá, hanem a felhasználói fiók.

A fiókhoz kapcsolt lejárati idő elérés esetén is megszűnik a belépés lehetősége. Ez, ha be van állítva, dátumként kerül rögzítésre (és a beállított napon a fiók már lejártnak minősül).

Jegyezzük meg, hogy meglévő munkamenetekre a fenti beállításoknak nincs hatása. Ha a fiók inaktívvá válik vagy lejár, a már megkezdett munkamenetek nem záródnak le, azokat a felhasználó tovább használhatja.
