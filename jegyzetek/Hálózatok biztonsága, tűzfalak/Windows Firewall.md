A Microsoft tűzfalmegoldása annyiban hasonlít a `firewalld` zónáihoz, hogy külön konfigurálható profilokra oszthatjuk a szabályokat. A hálózati interfészekhez egy-egy profilt rendelünk, de profilból csak három van: _Domain_, _Private_, _Public_. Három gyors beállítási eszközünk van a profilok konfigurálására:

- egy-egy profilban a tűzfal egyszerűen ki/bekapcsolható,
- az összes bejövő kapcsolat tiltható (pánik gomb) és
- alkalmazásonkénti engedély adható.

A speciális beállítási lehetőségek már szabályok megfogalmazását teszik lehetővé, külön a kimenő és külön a bejövő csomagokra. A szabályokra nem illeszkedő csomagok esetében beállítható az alapértelmezett viselkedés (engedélyezés vagy blokkolás). A szabályok alkalmazásának sorrendje:

1. Az engedélyezési szabályok elsőbbséget élveznek az alapértelmezett blokkolási beállítással szemben.
2. A blokkolási szabályok elsőbbséget élveznek az egymással ütköző engedélyezési szabályokkal szemben.
3. A specifikusabb szabályok (a 2-es eset kivételével) elsőbbséget élveznek a kevésbé specifikus szabályokkal szemben.

A szabályok eredménye nem csak az engedélyezés vagy blokkolás lehet. Például egy engedélyező szabály előírhatja a csomag naplózását.