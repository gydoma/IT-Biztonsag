A Windows NTFS jogosultságok helyes beállítása igen komplex feladatnak tűnik. Ha az alap beállításokon módosítunk annak következményei szerteágazóak lehetnek. Itt jön képbe az Effective Access és az Auditing.

A Windows rendszer az állománytulajdonságokat felvonultató párbeszédablakba beépítve tartalmaz egy _Effective Access_ nevű eszközt, amellyel az állományjogosultságok aktuális beállításai ellenőrizhetőek. Kiválasztva egy felhasználót vagy csoportot megnézhetjük annak aktuális jogosultságait.

Az Auditing egy rendszergazdáknak szóló eszköz amellyel kérhető az állomány hozzáférések naplózása. Külön engedélyezni kell az operációs rendszeren belül:

`gpedit.msc > computer configuration > windows settings > security settings > advanced audit policy configuration > system audit policies > object access > audit file system`

Szabályozható állományonként az állománytulajdonságok párbeszédablakában.

`Properties > Security >Advanced > Auditing` 

És az így nyújtott adatok böngészhetők az Event Viewer alkalmazásban:

`eventvwr.msc > Windows Logs > Security`