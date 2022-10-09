# Szerveroldali webprogramozás eszközök

Ez a leírás bemutatja, hogy milyen eszközökre van szükség a Szerveroldali webprogramozás tárgyon, és ezeket honnan lehet elérni. Kérjük vedd figyelembe, hogy a tárgy "hivatalosan támogatott operációs rendszere" a Windows, de a leírás tartalmaz hivatkozásokat Linux-ra és macOS-re is, illetve a tárgyon használt eszközök is támogatják ezeket a rendszereket.

Jelen leírás tartalma:

- [Szerveroldali webprogramozás eszközök](#szerveroldali-webprogramozás-eszközök)
  - [PHP és Composer](#php-és-composer)
  - [Node.js](#nodejs)
    - [Node.js közvetlen telepítése](#nodejs-közvetlen-telepítése)
    - [Node.js telepítése nvm használatával](#nodejs-telepítése-nvm-használatával)
    - [node-gyp függőségek](#node-gyp-függőségek)
  - [VSCode](#vscode)
  - [DB Browser for SQLite](#db-browser-for-sqlite)
  - [(Opcionális) Git](#opcionális-git)

## PHP és Composer

A PHP és a Composer a Laraveles témakörhöz szükséges. A PHP lehetővé teszi a PHP scriptek futtatását közvetlenül a gépeden, a Composer pedig egy csomagkezelő a PHP-hoz.

1. [Ebből a repository-ból](https://github.com/totadavid95/PhpComposerInstaller) le tudod tölteni a PHP, Composer telepítő legfrissebb verzióját. Egyszerűen csak futtasd le a `PhpComposerInstaller.exe` fájlt. Ha kézzel szeretnéd telepíteni a PHP-t és a Composer-t, a telepítő `README.md` fájljában annak a részletes menetét is leírjuk.
2. Telepítés után működnie kell az alábbi parancsoknak:

   ```shell
   php -v
   composer -V
   ```

A telepítő leírásában le van írva a Linux-os telepítés menete:
- https://github.com/totadavid95/PhpComposerInstaller#k%C3%A9zi-telep%C3%ADt%C3%A9s-menete-linux

## Node.js

A Node.js az egész félév során fontos lesz, hiszen ezzel fogjuk kezelni Laravel frontend oldali asset-eit, valamint teljes mértékben erre fog épülni a félév második fele: REST API, GraphQL és a Websocket (Socket.IO). Alapvetően két dolgot tartalmaz: Node.js és npm. A Node.js egy JavaScript runtime, ami a Chrome V8-as motorja köré épül, az *npm* pedig ehhez ad egy csomagkezelőt (*npm*, mint *Node Package Manager*).

Ha a gépedre már telepítve van a Node.js és telepített példány legalább 2 főverzióval le van maradva az aktuálisan letölthető legfrissebbhez képest, akkor érdemes lehet frissíteni, hiszen ha a telepített verzió nagyon le van maradva, akkor elképzelhető, hogy bizonyos dolgok nem fognak működni.

A Node.js kétféle módon is telepíthető:

- közvetlenül VAGY
- nvm-el (Node Version Manager).

### Node.js közvetlen telepítése

- Töltsd le majd telepítsd a legfrissebb LTS verziót a hivatalos Node.js letöltőoldalról: https://nodejs.org/en/download/
- Telepítés után működnie kell az alábbi parancsoknak:
  ```shell
  node -v
  npm -v
  ```

### Node.js telepítése nvm használatával

Az nvm a Node Version Manager rövidítése, amely több Node.js verzió együttes jelenlétét is támogatja, lehetővé teszi ezek egyszerű telepítését és a könnyű váltást az egyes verziók között. A Linux-os / macOS-es nvm NEM ugyanaz a tool, mint a Windows-os, pusztán csak hasonló feature-t valósítanak meg.

- Teendők Windows esetén:
  - Töltsd le a legfrissebb nvm verziót. Ehhez nyisd meg ezt a linket: https://github.com/coreybutler/nvm-windows/releases majd a legutóbbi release-ből töltsd le az **nvm-setup.exe** fájlt. Ezután egyszerűen futtasd a fájlt és telepítsd fel az nvm-et a számítógépedre.
  - _Ha már telepítve van a gépedre valamilyen Node.js verzió, az nvm a telepítését követően felkínálja, hogy átveszi annak a Node.js példánynak a kezelését. Ezt engedélyezd neki._
  - A legfrissebb Node.js LTS verzió telepítéséhez futtasd a következő parancsot: `nvm install lts`
  - Igény szerint akár telepíthetsz további verziókat is, ekkor az `nvm use` paranccsal tudsz váltani a telepített verziók között (a node parancs azt a verziót fogja megívni). Az nvm-ben elérhető parancsok listája: https://github.com/coreybutler/nvm-windows#usage
- Teendők Linux, macOS esetén:
  - Kövesd ezt a leírást: https://github.com/nvm-sh/nvm#installing-and-updating
  - macOS hibaelhárítás: https://github.com/nvm-sh/nvm#macos-troubleshooting
- Telepítés után működnie kell az alábbi parancsoknak:
  ```shell
  node -v
  npm -v
  ```

### node-gyp függőségek

Léteznek olyan csomagok, amelyek különféle binárisokat használnak a működésükhöz. Előfordul, hogy ezeket a binárisokat a csomag telepítésekor kell előállítani pl. C/C++ kódból, és ilyenkor jön képbe a **node-gyp**. Ezen csomagok egyik tipikus esete az **sqlite3**, amit a gyakorlatokon a Sequelize-hoz használunk adatbázis kezelő modulként. Ha a node-gyp működéséhez szükséges függőségek nem állnak rendelkezésre, akkor a binárisok előállítása nem fog sikerülni, ezért a csomagok telepítése is hibához fog vezetni.

A node-gyp build-ek megfelelő működéséhez az alábbi tool-oknak kell rendelkezésre állniuk:

- Windows-on:
  - [Legfrissebb Python3 verzió](https://www.python.org/downloads/windows/)
  - [Microsoft Build Tools](https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=BuildTools)
- Linux-on:
  - Legfrissebb Python3 verzió
  - `make` parancs
  - C/C++ fordító, pl. GCC
- macOS-en:
  - Legfrissebb Python3 verzió
  - XCode Command Line Tools
  - Bővebben: https://github.com/nodejs/node-gyp#on-macos

A Node.js telepítésekor is megadható, hogy automatikusan telepítse ezeket a build-hez szükséges tool-okat:

<img src="https://i.imgur.com/pgNyM4Z.png" width="350px" alt="Node.js build tools">

## VSCode

A VSCode egy elterjedt, jól támogatott és gyors szerkesztő. A gyakorlatok során ezt szoktuk használni.

1. Töltsd le a VSCode legfrissebb verzióját a hivatalos letöltőoldalról: https://code.visualstudio.com/download
2. Telepítsd fel az alábbi kiegészítőket:
   - [Live Share](https://marketplace.visualstudio.com/items?itemName=MS-vsliveshare.vsliveshare) - Gyakorlatokon végzett közös munkához
   - [Laravel Extension Pack](https://marketplace.visualstudio.com/items?itemName=onecentlin.laravel-extension-pack) - Laravel fejlesztéshez
   - [SQLite Viewer](https://marketplace.visualstudio.com/items?itemName=qwtel.sqlite-viewer) - SQLite adatbázisokba betekintéshez (pl. DB Browser helyett)
   - [GraphQL](https://marketplace.visualstudio.com/items?itemName=GraphQL.vscode-graphql) - GraphQL támogatás, syntax highlight
   - [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) - Kódok formázásához

## DB Browser for SQLite

A DB Browser for SQLite segítségével tudod kezelni az `.sqlite` fájlok (adatbázisok) tartalmát, debug célokra kiváló eszköz.

1. Töltsd le a programot a hivatalos letöltőoldalról: https://sqlitebrowser.org/dl/ (labor gépekre nem lehet telepíteni, ott viszont használhatod a portable verziót)
2. Társítsd hozzá az `.sqlite` fájlokhoz, hogy pár kattintással meg tudd őket nyitni.

## (Opcionális) Git

A tantárgyi anyagok GitHub-on vannak vezetve, ezért érdemes lehet letölteni a Git verziókezelőhöz kapcsolódó eszközöket is, hogy egyszerűen le tudd tölteni az új anyagokat is (`pull`). Természetesen a Git ismerete nem része a tantárgy anyagának, így ez teljesen opcionális.

- Git: https://git-scm.com/downloads
- GitHub Desktop: https://desktop.github.com/
  - Kényelmes felhasználói felületet ad, nem nagyon kell parancsokat beírogatnod a terminálba.

Ha nem ismered a Git-et, akkor [itt el tudod olvasni az alapokat](https://www.freecodecamp.org/news/learn-the-basics-of-git-in-under-10-minutes-da548267cc91/).
