Az aszimmetrikus titkosító eljárások közül tekintsük most az RSA alapút. Az openssl a genrsa parancsával képes számunkra RSA titkos (privát) kulcsot generálni. A titkos kulcsból a nyilvános (publikus) kulcs már előállítható.

A kulcs pár egyik tagját felhasználva titkosításra, a visszafejtés a kulcspár másik tagja birtokában megvalósítható. Üzenet küldése esetén az üzenet titkosítása a címzett nyilvános kulcsával történik és azt visszafejteni csak a címzett titkos kulcsával lehet.

Az eljárás a kulcsok felcserélésével is működik, de ennek hasznát már nem üzenetküldésnél, hanem hitelesítésnél (aláírásnál) látjuk.

Az üzenet aláírás nem más mint az üzenet kivonatának (hash értékének) titkosítása a küldő (vagy aláíró) titkos kulcsával. A kivonatot a nyilvános kulcs birtokában vissza lehet fejteni és így összevethető az üzenetből számított kivonattal. Ha az üzenet menet közben nem változik, akkor a hash értékeknek egyeznie kell. Az aláírást (a titkosított hash értéket) csak az aláíró tudja előállítani hiszen ehhez a titkos kulcsára van szükség. Ezel bizonyítható, hogy az aláírás tőle származik. Mivel az ellenőrzéshez a nyilvános kulcs kell, azt így bárki elvégezheti.