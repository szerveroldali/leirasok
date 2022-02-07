# Szerveroldali webprogramozás eszközök

Ez a leírás bemutatja, hogy milyen eszközökre van szükség a Szerveroldali webprogramozás tárgyon, és ezeket honnan lehet elérni.

Tartalom:
- [Szerveroldali webprogramozás eszközök](#szerveroldali-webprogramozás-eszközök)
  - [PHP és Composer](#php-és-composer)
  - [Node.js](#nodejs)
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

## Node.js

A Node.js a REST API, GraphQL és a Websocket (Socket.IO) témakörhöz szükséges. Alapvetően két dolgot telepít: Node.js és npm.

Ha a gépedre már telepítve van a Node.js és telepített példány legalább 2 főverzióval le van maradva az aktuálisan letölthető legfrissebbhez képest, akkor érdemes lehet frissíteni.

1. Töltsd le a legfrissebb LTS verziót a hivatalos Node.js letöltőoldalról: https://nodejs.org/en/download/
2. A telepítésnél ügyelj arra, hogy engedélyezd a **build tool-okat** is:

   ![Build tools](https://i.imgur.com/pgNyM4Z.png)

3. Telepítés után működnie kell az alábbi parancsoknak:
   
   ```shell
   node -v
   npm -v
   ```

## VSCode

A VSCode egy elterjedt, jól támogatott és gyors szerkesztő. A gyakorlatok során ezt szoktuk használni.

1. Töltsd le a VSCode legfrissebb verzióját a hivatalos letöltőoldalról: https://code.visualstudio.com/download
2. Telepítsd fel az alábbi kiegészítőket:
   - [Laravel Extension Pack](https://marketplace.visualstudio.com/items?itemName=onecentlin.laravel-extension-pack) - Laravel fejlesztéshez
   - [Live Share](https://marketplace.visualstudio.com/items?itemName=MS-vsliveshare.vsliveshare) - Gyakorlatokon végzett közös munkához
   - [GraphQL](https://marketplace.visualstudio.com/items?itemName=GraphQL.vscode-graphql) - GraphQL támogatás, syntax highlight
   - [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) - Kódok formázásához

## DB Browser for SQLite

A DB Browser for SQLite segítségével tudod kezelni az `.sqlite` fájlok (adatbázisok) tartalmát, debug célokra kiváló eszköz.

1. Töltsd le a programot a hivatalos letöltőoldalról: https://sqlitebrowser.org/dl/
2. Társítsd hozzá az `.sqlite` fájlokhoz, hogy pár kattintással meg tudd őket nyitni.

## (Opcionális) Git

A tantárgyi anyagok GitHub-on vannak vezetve, ezért érdemes lehet letölteni a Git verziókezelőhöz kapcsolódó eszközöket is, hogy egyszerűen le tudd tölteni az új anyagokat is (`pull`). Természetesen a Git nem része a tantárgy anyagának, így ez teljesen opcionális.

- Git: https://git-scm.com/downloads
- GitHub Desktop: https://desktop.github.com/
   - Kényelmes felhasználói felületet ad, nem nagyon kell parancsokat beírogatnod a terminálba.

Ha nem ismered a Git-et, akkor [itt el tudod olvasni az alapokat](https://www.freecodecamp.org/news/learn-the-basics-of-git-in-under-10-minutes-da548267cc91/).
