<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400"></a></p>

Tartalom:
- [Szerveroldali webprogramozás - Laravel](#szerveroldali-webprogramozás---laravel)
  - [Röviden a Laravelről](#röviden-a-laravelről)
  - [Egy Laraveles projekt felépítése](#egy-laraveles-projekt-felépítése)
  - [Laravel dokumentációk](#laravel-dokumentációk)

# Szerveroldali webprogramozás - Laravel

## Röviden a Laravelről
A Laravel egy PHP MVC keretrendszer, azaz a kód strukurálására a Modell-Nézet-Vezérlő mintát ajánlja. Ebben a **modell** felelős az adatok tárolásáért és felületfüggetlen feldolgozásáért, a **nézet** az adatok megjelenítéséért, azaz a mi esetünkben a HTML előállításáért, és a **vezérlő** az, ami fogadja a HTTP kérést, beolvassa az adatokat, meghívja a modellréteg feldolgozó függvényeit, majd ezek eredményét a kiírja a nézet meghívásával. A legtöbb MVC-s keretrendszernek központi eleme még a  **routing**, amely során egy URL, egy végpontot a megfelelő vezérlőlogikához rendelünk.

Egy egyszerű HTML oldal megjelenítéséhez nem kell adat, viszont végpont alatt jelennek az oldalak, azaz kell bele routing, ami egy vezérlőmetódushoz irányítja a végrehajtást, ami egyszerűen megjeleníti a nézetet, ami a HTML-ünket tartalmazza. Azaz ezt az egyszerű kis dolog elvégézéséhez rögtön három komponens kell:
- routing
- controller
- view

## Egy Laraveles projekt felépítése
- Első ránézésre a Laravel projektszerkezete egyáltalán nem egyértelmű olyan embereknek, akik először látják azt. Erre igazán csak gyakorlati úton lehet ráérezni.
-  Nagyon nagy vonalakban a projekt szerkezete a következő:
   - **app**:
     - Gyakorlatilag az alkalmazás kódjának a magját tartalmazza
     - A félév során megkerülhetetlen lesz, fontos taglalni az almappákat is:
       - **app/Models**:
         - létre fogunk hozni modelleket, ezek ebben a mappában találhatók 
       - **app/Http**
         - Tartalmazza a controllereket, middleware-ket, és a formokhoz tartozó request-eket
         - Mondhatni itt dolgozzuk fel szinte az összes kérést, ami az alkalmazáshoz érkezik a kliensektől
         - **app/Http/Controllers**
           - Különböző vezérlőlogikák, amiket hozzá tudunk rendelni az egyes végpontokhoz
         - **app/Http/Middleware**
           - A middleware fogalma gyakran elő fog jönni a félévben (a Node.js-nél is)
           - Amikor bejön egy kérés, middleware-k sora hajtódik végre, tehát ez úgymond egy köztes logika.
           - Például: 
             - megvan a middleware-k sorrendje, ha bejön egy kérés, azt a web middleware veszi át, majd utána hajtódik végre a végpont mögött rejlő vezérlési logika. Azonban ha beállítunk egy autentikációt, akkor a web middleware után egy auth middleware hajtódik végre. Itt pedig ha nem sikerül a hitelesítés, akkor pl. átirányítja a felhasználót a login oldalra, nem pedig a végponthoz tartozó vezérlés következik.
             - [web mw] -> [auth mw] -> [kiszolgálás] 
        - **app/Http/Requests**
          -  Amikor form-okat (űrlapokat) küld el a felhasználó, akkor létre lehet hozni ilyen request objektumokat, amik egy-egy ilyen form logikáját le tudják kezelni. Ezek találhatók itt.
   - **bootstrap**
     - Ez a félév során számunkra érdektelen, ha bele is kell valaha nyúlni, akkor azt csak indokolt esetben és körültekintően érdemes megtenni.
     - Itt található az app.php fájl, ami kvázi "felállítja" (bootstrap-eli, bár erre nincs nagyon egyszavas értelmes magyar fordítás) az alkalmazást, illetve a gyorsítótárazás is itt van kezelve, ami arra szolgál, hogy segítsen optimalizálni a teljesítményt
   - **config**
     - Elég egyértelmű a neve alapján, és pontosan arra is való, amit az ember sejtene mögötte: itt van az alkalmazás konfigurációja
     - Elég érthetően kategóriákra van bontva a fájlok szerint, minimális szinten bele fogunk nézni a félév során
   - **database**
     - Ennek is egyértelmű a neve, az adatbázissal kapcsolatos dolgokat tartalmazza
     - Három almappát tartalmaz, mindegyik aktívan kelleni fog majd a félév során:
       - **migrations**:
         - itt írjuk le az adatbázis szerkezetét, fontos, hogy a fájlok neve előtt van egy timestamp (időbélyeg), a Laravel eszerint rendezi sorba őket
         - felfogható az adatbázis szerkezethez tartozó "verziókezelésként" is    
       - **factories**:
         - modell"gyárak", egy megadott logika szerint generálhatunk modelleket
       - **seeders**:
         - A seeder arra való, hogy feltöltse az adatbázist adatokkal
         - A migration-ök, factory-k és a seederek egymásra épülnek, hiszen az adatbázis szerkezetét, a táblákat, kapcsolatokat a migration-ök adják meg, az egyes adatokat a factory-k, a factory-k által generált modelleket pedig a seederben tudjuk egy meghatározott logika szerint feltölteni
    - **public**
      -  Itt található az index.php, ami az összes bejövő kérés belépési pontja, valamint meghatározza az alkalmazás autoload-ját
      - Ezen felül itt találhatók az "asset-ek": képek, JS és CSS fájlok
         - ha npm-et használunk és pl npm segítségével töltjük le a Bootstrap-et, a Laravel Mix build kimenetei is ide kerülnek tipikusan JS és CSS fájlok formájában
      - Ha egy hostingra töltjük fel az appot, ezt a mappát kell az úgynevezett *www* vagy *public_html* mappába tenni, és az alkalmazás többi részét egy szinttel feljebb
    - **resources**
      - Ez is egy fontos mappa lesz a félév során, már rögtön a legelején
      - Ez tartalmazza a css és js fájlok "natív" verzióit (a public mappába ezeknek a build-je kerül általában), a nyelvi fájlokat (lang), valamint a **view-okat** (az MVC logikából a V), ezekben fogjuk létrehozni a Blade template-ket
      - A későbbiekben nézünk példát a lang magyarosítására is
   - **routes**
     - Meghatározza az összes útvonalat, ami az alkalmazáshoz tartozik
     - Nekünk alapvetően ebből csak a **web.php** az, ami fontos lesz
     - De tulajdonképpen mik is vannak itt:
       - **web.php**
         - A `RouteServiceProvider`-re épül, ami biztosít session-t (munkamenetet), CSRF védelmet (OWASP top 10-ben is benne van, és EA anyag is lesz), valamint cookie titkosítást is
         - Ha az alkalmazás nem használ állapotmentes (stateless) RESTful API-t (márpedig a mienk nem fog a félévben), akkor kb. ezzel a fájllal le is fedtük az összes route-ot.
       - **api.php**: 
         - Szintén a `RouteServiceProvider`-re épül, viszont ezek stateless (állapotmentes) útvonalak, ezért tokennel kell hitelesíteni őket. Ezt a fájlt kell(ene) használni, ha REST API-t készítünk és ahhoz veszünk fel route-okat, viszont azt mi a félév második felében nem Laravellel, hanem Express JS-el fogunk csinálni, Node.js környezetben.
        - **console.php**
          - Gyakorlatilag artisan parancsokat lehet felvenni, pl. `php artisan valami`, nem fogunk ilyet csinálni a félévben. 
        -  **channels.php**
           - Ez akkor lényeges, ha használunk valamilyen event broadcasting-ot, gyakorlatilag ezzel lehet live chatet, játékot stb. csinálni, de nagyon nem ennek a félévnek az anyaga.
    - **storage**
      - Ez a könyvtár elég sok minden tartalmaz, de csak érintőlegesen fogunk vele foglalkozni a gyakorlatokon
      - Például itt vannak a naplófájlok, illetve minden olyan fájl, amit az alkalmazás hoz létre / generál ki
      - Alapvetően három almappára van bontva: *app, framework, logs* (alkalmazás és framework által generált dolgok, valamint a generált naplók)
      - A **storage/app/public** mappát fogjuk használni, ide berakhatók a felhasználókhoz köthető fájlok (pl. egy kép, amit feltölt valamihez). Ehhez a mappához lehet készíteni egy *symlink*-et (symbolic link), hogy hozzáférhető legyen a public mappából is, erre lesz majd a `php artisan storage:link` parancs, de ezt majd később... lehet olyat is csinálni, hogy egy végpont különböző fájlműveletekkel lekérjen innen célzottan egy fájlt és azt visszaadja, és akkor nem kell symlink. Legfeljebb ha nem létezik a lekérni kívánt fájl, akkor valami default oldalra irányít, vagy 404 választ ad.
    - **tests**
      - Itt találhatóak az automatikusan futtatható unit testek, a félév során erre nem lesz idő
    - **vendor**
      - Itt találhatóak a Composeres csomagok, amiket a *Composer* a távoli szerverről töltött le a *composer.json* fájl alapján.
    - **node_modules**
      - Itt találhatóak az npm-es csomagok, amiket a *Node Package Manager* (röviden *npm*) a távoli szerverről töltött le a *package.json* fájl alapján.
- Az alkalmazás gyökérmappájában található fájlokról röviden, hogy mi micsoda:
  - **.env**:
    - environment, azaz környezeti fájl, leírja, hogy az alkalmazáshoz egy adott környezetben milyen konfiguráció tartozik (pl. más a fejlesztési környezet és az a környezet ahol az alkalmazás majd ténylegesen fut, más az adatbázis kapcsolódás, stb.)
    - Biztonsági okokból ez nincs verziókezelve a git által (a .gitignore része)
  - **.env.example**
    - Egy jó kiindulópont, ha *.env* fájlt akarunk csinálni, csak lemásoljuk és átnevezzük *.env*-nek, az összes basic dolog benne van, csak minimálisan kell átírni, majd egy APP_KEY-t generálni a `php artisan key:generate` paranccsal.
  - **composer.json**, **package.json**, illetve a lock fájlok:
    - Ezek leírják, hogy a Composer és az npm milyen csomagokat telepítsen, a lock fájlokat csak ehhez generálják (a lock file csak annyit csinál, hogy az adott csomag milyen verziójú alcsomagokat követel meg, és mivel generált fájl, ezért nem szabad átírni)
  - **.gitattributes**, **.gitignore**
    - Git-hez tartozó beállításokat, kizárásokat tartalmazó fájlok
  - **.editorconfig**
    - Ez egy konfigurációs fájl, ami az editoroknak (VSCode, PHPStorm, stb) ad meg különböző beállításokat, pl. egyes fájlok, fájltípusok karakterkódolását, tab-ot/space-t használjanak, stb.
  - **webpack.mix.js**
    - Ezt az npm működteti, és a Laravel Mix konfigurációját adja meg, vagyis azt, hogy milyen logika szerint generálja ki a frontend oldali asset-eket
  - **artisan**
    - Az Artisan konzolt működteti
  - **server.php**
    - Hasonló a **public/index.php**-hoz, azonban az Apache "mod_rewrite" funkcionalitását szimulálja, így ezáltal "valódi" webszerver nélkül is tesztelhető az alkalmazás, a [PHP beépített dev webszerverét](https://www.php.net/manual/en/features.commandline.webserver.php) használva
  - **phpunit.xml**
    - Fentebb már volt szó a tesztelésről, ezt a *PHPUnit* nevű csomag látja el, ez az XML fájl pedig annak a [konfigurációját](https://phpunit.readthedocs.io/en/9.5/configuration.html) írja le
  - **.styleci.yml**
    - A *phpunit.xml*-hez hasonlóan ez is egy konfigurációs fájl, a *StyleCI* csomaghoz

## Laravel dokumentációk
- A Laravel jól dokumentált, és a dokumentációja jól forgatható.
- A Laravelnek alapvetően kétféle dokumentációja van:
  - egy beszédesebb, amiben a főbb dolgok és az irányvonal le van írva, és ehhez számtalan példát is mutat:
    - [https://laravel.com/docs](https://laravel.com/docs)
      - ez a link mindig a legújabb kiadásra fog átirányítani 
  - és van egy "szárazabb" verzió is, ami leírja az összes funkció felépítését: 
    - [https://laravel.com/api/8.x/index.html](https://laravel.com/api/8.x/index.html)
      - értelemszerűen a listából mindig ki kell választani az aktuálisan legfrissebb verziót
