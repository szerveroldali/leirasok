# Sequelize asszociációk

Ez a leírás bemutatja, hogy néznek ki Sequelize-ban az adatmodellek közötti kapcsolatok (a Sequelize ezeket nem relációknak, hanem asszociációknak hívja), és milyen metódusokkal tudjuk ezeket létrehozni, elérni, kezelni.

Tartalom:
- [Sequelize asszociációk](#sequelize-asszociációk)
  - [Lehetséges relációk](#lehetséges-relációk)
  - [Relációk megadása](#relációk-megadása)
  - [Elérhető metódusok](#elérhető-metódusok)
  - [Metódusok használata](#metódusok-használata)
  - [Sok-Sok kapcsolat kiépítése](#sok-sok-kapcsolat-kiépítése)
    - [Elméleti háttér](#elméleti-háttér)
    - [Modellek generálása, táblák beállítása](#modellek-generálása-táblák-beállítása)
    - [Adatmodellek beállítása](#adatmodellek-beállítása)
  - [Sok-sok kapcsolat tesztelése](#sok-sok-kapcsolat-tesztelése)
    - [Elérhető metódusok lekérése](#elérhető-metódusok-lekérése)
    - [Részletes példa a metódusok használatára](#részletes-példa-a-metódusok-használatára)

## Lehetséges relációk
- 1-1 (One to One, vagyis Egy-Egy kapcsolat)
  - Egyértelmű kapcsolatot teremt két entitás között.
  - Példa: ha egyik oldalt személyeket tartunk nyilván (Person), másik táblában pedig jogosítványokat (DrivingLicense), akkor egy adott személynek csak egy jogosítványa lehet és egy adott jogosítványhoz is csak egy személy tartozhat.
  - Részletes dokumentáció: https://sequelize.org/master/manual/assocs.html#one-to-one-relationships
  - Ha A és B között 1-1 kapcsolat van, akkor az alábbi metódusokat kapják:
    - A: [hasOne](https://sequelize.org/master/class/lib/associations/has-one.js~HasOne.html) (neki van egy B-je)
    - B: [belongsTo](https://sequelize.org/master/class/lib/associations/belongs-to.js~BelongsTo.html) (ő pedig A-hoz tartozik)
- 1-N (One to Many, vagyis Egy-Sok kapcsolat)
  - Egy entitáshoz akár több másik eintitás is tartozhat.
  - Példa: ha van egy személy (Person), akinek a nyilvántartás szerint lehetnek a nevén autók. Ez lehet egy autó is, lehet kettő, három, de igazából technikailag akárhány, tehát N db autó lehet a nevén. Innen jön az elnevezés, 1 személyhez N db autó kapcsolódhat, tehát 1-N.
  - Részletes dokumentáció: https://sequelize.org/master/manual/assocs.html#one-to-many-relationships
  - Ha A és B között 1-N kapcsolat van, akkor az alábbi metódusokat kapják:
    - A: [hasMany](https://sequelize.org/master/class/lib/associations/has-many.js~HasMany.html) (neki van valamennyi B-je)
    - B: [belongsTo](https://sequelize.org/master/class/lib/associations/belongs-to.js~BelongsTo.html) (ő pedig A-hoz tartozik)
- N-N / N-M (Many to Many, vagyis Sok-Sok kapcsolat)
  - Egy entitáshoz akár több másik entitás is tartozhat, akárcsak az előző 1-N-es példában, azonban míg az egy oldalú volt, itt ez visszafelé is igaz.
  - Hívják N-M kapcsolatnak is, hiszen nem feltétlen kell, hogy ugyannyi kötődés legyen a két oldalon, csak technikailag lehet több, mint egy bármelyik oldalról.
  - Példa: Egy személy (Person), ha autót akar venni, akkor azt több szalonból (Shop) is megteheti, de egy szalon is többféle személynek adhat el autót.
  - Részletes dokumentáció: https://sequelize.org/master/manual/assocs.html#many-to-many-relationships
  - Ha A és B között N-N kapcsolat van, akkor az alábbi metódusokat kapják:
    - A: [belongsToMany](https://sequelize.org/master/class/lib/associations/belongs-to-many.js~BelongsToMany.html) (többszörösen kötődhet B-hez)
    - B: [belongsToMany](https://sequelize.org/master/class/lib/associations/belongs-to-many.js~BelongsToMany.html) (többszörösen kötődhet A-hoz)

## Relációk megadása
A relációkat a Sequelize modellek `associate` nevű statikus metódusán keresztül adjuk meg a fent említett metódusok segítségével.

Ehhez generáljunk ki mondjuk egy Shop és egy Item modelt:
```shell
npx sequelize model:generate --name Shop --attributes name:string,address:string
npx sequelize model:generate --name Item --attributes name:string,price:integer
```

Majd a `models` mappában adjuk meg a relációkat:

```js
class Shop extends Model {
    // A models objektumot megkapja paraméterben, ebből elérhető az összes többi model
    static associate(models) {
        // A this a jelenlegi model, vagyis a "Shop"
        // Ha itt adunk mondjuk egy hasMany-t a Items-re, azzal azt mondjuk, hogy a Shop-hoz tartozik valamennyi Item (1-N)
        this.hasMany(models.Item);
    }
}
```

Ilyenkor az Item modell a következőképpen néz ki:

```js
class Item extends Model {
    static associate(models) {
        // A this a jelenlegi model, vagyis a "Item"
        // Mivel ez a "Item" a "Shop"-hoz tartozik, ide belongsTo()-t adunk, amiben megjelöljük a "Shop" modelt
        this.belongsTo(models.Shop);
    }
}
```

Mivel itt 1-N kapcsolatot mutattunk be, vagyis, hogy a bolthoz tartozik egy árukészlet, nyilvánvalóan valahogy az adatbázisban ábrázolni is kell ezt a függést. Az "Item" nevű modelben szokott lenni egy plusz mező, ami a "Shop"-ra hivatkozik, jellemzően a "Shop"-nak az ID mezőjére. Tehát az "Item" migration-jébe kell, hogy rakjuk még valami ilyesmit:

```js
// ...
ShopId: {
    type: Sequelize.INTEGER,
},
// ...
```

Felmerül a kérdés, hogy fogja a Sequelize megtalálni egy a `ShopId` nevű külső kulcsot? Úgy, hogy megnézi, az Item-nél a belongsTo-nak milyen modelt adtunk paraméterül, annak lekéri a nevét és az elsődleges kulcsát (primary key) és CamelCase szerűen egymás után írja őket: ModelNeveElsodlegesKulcsNeve, vagyis ha a model neve Shop, az elsődleges kulcsé pedig id, akkor ShopId fog előállni. Lásd például [itt](https://github.com/sequelize/sequelize/blob/main/lib/associations/belongs-to.js#L34-L48).

Ettől természetesen el lehet térni, tehát nem muszáj, hogy ez `ShopId` néven szerepeljen, akkor viszont meg kell mondani a hasMany-nek és a belongsTo-nak is az érintett modellekben, hogy mit állítottunk be egyedileg külső kulcsnak, mivel eltértünk az elnevezési konvencióktól. 

Tehát ha például a `ShopId`-t átírjuk a migration-ben `CustomShopId`-ra, akkor ki kell egészíteni a metódusokat a modelfájlokban is, hogy tudja az ORM, milyen külső kulcsot keressen:

Ez a Shop esetében:
```js
this.hasMany(models.Item, {
    foreignKey: "CustomShopId",
});
```

Mivel a dolog oda-vissza működik, ne felejtsük el a másik oldalt sem, az Items-t:
```js
this.belongsTo(models.Shop, {
    foreignKey: "CustomShopId",
});
```

Mint az látszik, itt is fontosak az elnevezési konvenciók, akárcsak Laravelben. A Sequelize ugyanis egy [Inflection](https://github.com/dreamerslab/node.inflection) nevű library-t használ az elnevezések nyelvtani kezeléséhez, hasonlóan, ahogy ezt a Laravel is teszi, csak PHP-s környezetben. Ennek a működését valahogy így kell elképzelni:

```js
const inflection = require('inflection');

// A pluralize többes számba rakja az item-et, vagyis items lesz
// A capitalize pedig nagybetűsíti, tehát az items Items lesz
console.log(inflection.capitalize(inflection.pluralize('item'))); // Items

// Ez egy picit bonyolultabb eset, mivel a category többes száma categories lesz,
// ez a fv. valóban ezt is fogja visszaadni
console.log(inflection.pluralize('category')); // categories

// Ez is azt adja vissza, hogy categories, ha már többes számban van,
// akkor egyszerűen abban is fogja hagyni.
console.log(inflection.pluralize('categories')); // categories

// Ez a többes számot alakítja egyes számmá, vagyis category lesz
console.log(inflection.singularize('categories')); // category

// Továbbá tudja kezelni az úgynevezett snake_case és CamelCase módokat is.
// A snake_case neve onnan ered, hogy a _ karakter olyan, mint egy kígyó (snake).
// A CamelCase pedig onnan, hogy a nagybetű olyan, mint a teve (camel) púpja.
console.log(inflection.camelize('first_second')); // FirstSecond
console.log(inflection.underscore('FirstSecond')); // first_second
```

Tehát a Sequelize fogja a model nevét, bedobja ebbe az Inflection-be, ami kiadja mondjuk a többesszámát, de igény szerint bármi mást is tehet vele, mint láttuk.

## Elérhető metódusok
Ha megteremtettük a relációkat a fenti módon, akkor különböző metódusok válnak elérhetővé, mint például ilyenek:

```js
const db = require("./models");
const { Shop } = db;
const faker = require("faker");

;(async () => {
    // Shop létrehozása
    const shop = await Shop.create({
        name: faker.lorem.word(),
        address: faker.address.secondaryAddress(),
    });

    // Item-ek létrehozása a Shop-hoz
    await shop.createItem({
        name: faker.lorem.word(),
        price: faker.datatype.number({ min: 500, max: 20000 }),
    });
    const secondItem = await shop.createItem({
        name: faker.lorem.word(),
        price: faker.datatype.number({ min: 500, max: 20000 }),
    });

    // Shop-hoz tartozó Item-ek lekérése és kiírása a konzolra
    console.log(await shop.getItems({ raw: true }));
    // Shop-hoz tartozó Item-ek számának lekérése és kiírása a konzolra
    console.log(await shop.countItems());

    // A 2. Item eltávolítása a Shop-ból
    await shop.removeItem(secondItem);
    console.log("Item eltávolítva");

    // Shop-hoz tartozó Item-ek számának lekérése és kiírása a konzolra
    // Ez már csak egy Item-et fog jelezni, hiszen a másodikkal megszüntettük
    // a kapcsolatot (nem az Item-et töröltük, csak a shopId mezőjét állítottuk át)
    console.log(await shop.countItems());
})();
```

Itt el is érkeztünk egy további érdekes pontra, hogy hogyan kerülnek kigenerálásra ezek a metódusok, amiket a reláció valamelyik oldaláról elérünk. 

Látszik, hogy az elnevezésük függ a modelljeinktől és az általunk megadott nevektől, mégis honnan jön a nevük? 

Nos, alapvetően itt is a model nevét veszi alapul a rendszer, hacsak a fenti hasMany, belongsTo metódusoknál nem adunk meg valamilyen "aliast" ("as" property). [Lásd itt.](https://github.com/sequelize/sequelize/blob/main/lib/associations/belongs-to.js#L24-L32) A model neve egyes- és többesszámban így kérhető le (fontos, hogy magát a models mappából jövő model-t kell megadni, és nem annak egy példányát):

```js
console.log(Shop.options.name);
```

Ez a következőt fogja kiírni:
```js
{ plural: 'Shops', singular: 'Shop' }
```

Az elnevezések a következő módon néznek ki:

- belongsTo oldalról:
  - Dokumentáció: https://sequelize.org/master/class/lib/associations/belongs-to.js~BelongsTo.html
  - Forráskód: https://github.com/sequelize/sequelize/blob/main/lib/associations/belongs-to.js#L73
  - Metódusok:			
    - get: `get${egyes szám}`,
    - set: `set${egyes szám}`,
    - create: `create${egyes szám}`
- belongsToMany oldalról:
  - Dokumentáció: http://docs.sequelizejs.com/class/lib/associations/belongs-to-many.js~BelongsToMany.html
  - Forráskód: https://github.com/sequelize/sequelize/blob/main/lib/associations/belongs-to-many.js#L209
  - Metódusok:
    - get: `get${többes szám}`,
    - set: `set${többes szám}`,
    - addMultiple: `add${többes szám}`,
    - add: `add${egyes szám}`,
    - create: `create${egyes szám}`,
    - remove: `remove${egyes szám}`,
    - removeMultiple: `remove${többes szám}`,
    - hasSingle: `has${egyes szám}`,
    - hasAll: `has${többes szám}`,
    - count: `count${többes szám}`
- hasMany oldalról:
  - Dokumentáció: http://docs.sequelizejs.com/class/lib/associations/has-many.js~HasMany.html
  - Forráskód: https://github.com/sequelize/sequelize/blob/main/lib/associations/has-many.js#L98
  - Metódusok:
    - get: `get${többes szám}`,
    - set: `set${többes szám}`,
    - addMultiple: `add${többes szám}`,
    - add: `add${egyes szám}`,
    - create: `create${egyes szám}`,
    - remove: `remove${egyes szám}`,
    - removeMultiple: `remove${többes szám}`,
    - hasSingle: `has${egyes szám}`,
    - hasAll: `has${többes szám}`,
    - count: `count${többes szám}`

Az alábbi segédfüggvénnyel ki is lehet listázni egy adott modellhez tartozó metódusokat:

```js
const db = require("./models");
const { Shop, Item } = db;

const getModelAccessorMethods = (model) => {
    console.log(`${model.name}:`);
    Object.entries(model.associations).forEach(([_, associatedModel]) => {
        Object.entries(associatedModel.accessors).forEach(([action, accessor]) => {
            console.log(`  ${action}: ${model.name}.${accessor}(...)`);
        });
    });
};

;(async () => {
    getModelAccessorMethods(Shop);
    getModelAccessorMethods(Item);
})();
```

Ez valami ilyesmit fog kiírni:
```
Shop:
  get: Shop.getItems(...)
  set: Shop.setItems(...)
  addMultiple: Shop.addItems(...)
  add: Shop.addItem(...)
  create: Shop.createItem(...)
  remove: Shop.removeItem(...)
  removeMultiple: Shop.removeItems(...)
  hasSingle: Shop.hasItem(...)
  hasAll: Shop.hasItems(...)
  count: Shop.countItems(...)
Item:
  get: Item.getShop(...)
  set: Item.setShop(...)
  create: Item.createShop(...)
```

## Metódusok használata
A fenti metódusok gyakorlati működését az alábbi kommentekkel ellátott példa mutatja.

```js
const models = require("./models");
const { Shop } = models;
const faker = require("faker");

;(async () => {
    const shop = await Shop.findByPk(1);
    // A Shop a hasMany oldalon van, tehát...
    // 1.) Item-ek lekérése: get${többes szám}, tehát getItems
    console.log(await shop.getItems());
    // 2.) Item-ek beállítása (ezentúl csak ezek az Item-ek lesznek hozzárendelve): set${többes szám}, tehát setItems([Item-ek tömbje, lehet id-k tömbje, vagy konkrét modelleké])
    console.log(await shop.setItems([1,2]));
    // 3.) Egy Item hozzáadása (a meglévő Item-ek mellé): add${egyes szám}, tehát addItem(Item id / Item modell)
    console.log(await shop.addItem(3));
    // 4.) Több Item hozzáadása (a meglévő Item-ek mellé): add${többes szám}, tehát addItems([Item-ek tömbje, lehet id-k tömbje, vagy konkrét modelleké])
    console.log(await shop.addItems([4,5]));
    // 5.) Új Item létrehozása, majd a modellhez rendelése (a meglévő Item-ek mellé): create${egyes szám}, tehát createItem({ Item mezői })
    console.log(await shop.createItem({
        name: faker.commerce.productName(),
        price: faker.commerce.price(),
    }));
    // 6.) Egy Item eltávolítása (a többi megmarad): remove${egyes szám}, tehát removeItem(Item id / item modell)
    console.log(await shop.removeItem(3));
    // 7.) Több Item eltávolítása (a többi megmarad): remove${többes szám}, tehát removeItems([Item-ek tömbje, lehet id-k tömbje, vagy konkrét modelleké])
    console.log(await shop.removeItems([4,5]));
    // 8.) Egy adott Item hozzá van-e rendelve a Shop-hoz: has${egyes szám}, tehát hasItem(Item id / Item modell)
    console.log(await shop.hasItem(1)); // true
    console.log(await shop.hasItem(4)); // false, hiszen a 4 az előbb el lett távolítva a kapcsolatból
    // 9.) Az összes megadott Item hozzá van-e rendelve a Shop-hoz: has${többes szám}, tehát hasItems([Item-ek tömbje, lehet id-k tömbje, vagy konkrét modelleké])
    console.log(await shop.hasItems([1,2])); // true
    console.log(await shop.hasItems([1,2,4])); // false, hiszen a 4 az előbb el lett távolítva a kapcsolatból, tehát nincs mind hozzárendelve
    // 10.) A Shop-hoz kapcsolt Item-ek számának lekérése: count${többes szám}, tehát countItems()
    console.log(await shop.countItems());
})();
```

## Sok-Sok kapcsolat kiépítése

### Elméleti háttér
Az 1-1, 1-N kapcsolatoknál elég az A-hoz kötötdő B-nek megadni egy mezőt, ami az A elsődleges kulcsára hivatkozik, a fenti példánál a Shop-hoz tartoztak Item-ek, ezért az Item-ben meg kellett adni a Shop ID-ját, ami a Shop elsődleges kulcsa.

A sok-sok kapcsolat esetében azonban ez így nem elegendő adatbázis reprezentáció, mivel nem lehetne vele rendesen ábrázolni egy ilyen összetett kapcsolatot. Ezért a sok-sok kapcsolat esetében nem az egyes modellekhez veszünk fel plusz mezőket, hanem hagyjuk őket, és bevezetünk egy plusz táblát, az úgynevezett "kapcsolótáblát".

A kapcsolótáblába bejegyzések kerülnek, amelyben egy bejegyzés azt reprezentálja, hogy A és B között kapcsolat van. A tábla szerkezete az alábbi módon néz ki.

- Kapcsolótábla
  - id: a bejegyzés ID-ja
  - AId: Az "A" model ID-ja
  - BId: A "B" model ID-ja
  - CreatedAt: Mikor jött létre a bejegyzés a kapcsolótáblában
  - UpdatedAt: Mikor módosult a bejegyzés a kapcsolótáblában

Továbbá érdemes egy olyan megkötést is alkalmazni a táblán, hogy egy páros csak egyszer kerülhet bele, vagyis egy "AId" és "BId" páros csak egyszer szerepelhet a táblában, így a bejegyzések egyértelműek lesznek.

Ez pongyolán ábrázolva valahogy így néz ki: 
- Unique [AId, BId]

Nyilván a pontos implementáció attól a keretrendszertől függ, amiben dolgozunk, ezt mindjárt látjuk picit később.

Vegyük mondjuk azt a példát, hogy van egy blogunk, amiben vannak kategóriáink és bejegyzéseink. Ezek között sok-sok kapcsolat van, hiszen egy kategóriához akármennyi bejegyzés tartozhat és egy bejegyzéshez is akármennyi kategória tartozhat. Így lehetőségünk van megjeleníteni egy kategóriához az összes hozzá tartozó bejegyzést, és a bejegyzés oldalán is az összes hozzá tartozó kategóriát.

### Modellek generálása, táblák beállítása

Sequelize-ban ennek a megvalósítása a következőképpen néz ki.

Először is ki kell generálni a Kategória és Bejegyzés modelleket az alábbi parancsokkal:

```shell
npx sequelize model:generate --name Category --attributes name:string,color:string
npx sequelize model:generate --name Post --attributes title:string,text:string
```

Ennek a két parancsnak a hatására alapvetően két irányból történik változás:
1. Létrejön a `models` mappában a `category.js` és a `post.js`
2. Létrejön a `migrations` mappában a `${timestamp}-create-category.js` és a `${timestamp}-create-post.js`

A sok-sok kapcsolat implementációját először a migration oldalról kezdjük, ehhez ki kell generálni a kapcsolótáblához tartozó migration-t:
```shell
npx sequelize migration:generate --name create-category-post
```
Ennek az elnevezése egyébként nincs kőbe vésve, de célszerű követni a szokásokat, és a `create-{táblanév}` modellt követni.

Miután ezt a parancsot kiadtuk, a `migrations` mappában megjelenik a `${timestamp}-create-category-post.js`, aminek a tartalma valami ilyesmi lesz:

```js
"use strict";

module.exports = {
    up: async (queryInterface, Sequelize) => {
        // Ez akkor fut le, amikor feltöltjük a migrationt (migrate)
    },

    down: async (queryInterface, Sequelize) => {
        // Ez pedig akkor amikor visszavonjuk (undo)
    }
};
```

A feladat most az, hogy a fentebb már ismertetett kapcsolótábla felépítés szerint kialakítsuk a Sequelize-os eszközökkel a migrationt. Ez így fog kinézni:

```js
"use strict";

module.exports = {
    up: async (queryInterface, Sequelize) => {
        // Tábla létrehozása
        await queryInterface.createTable("CategoryPost", {
            // ID mező, ugyanaz, mint bármelyik másik generált modellnél
            id: {
                allowNull: false,
                autoIncrement: true,
                primaryKey: true,
                type: Sequelize.INTEGER,
            },
            // Kategória ID, Bejegyzés ID
            // Kategória ID, Bejegyzés ID
            CategoryId: {
                type: Sequelize.INTEGER,
                // Nem vehet fel NULL értéket, mindenképpen valamilyen INTEGER-nek kell lennie
                allowNull: false,
                // Megadjuk, hogy ez egy külső kulcs, ami a "Categories" táblán belüli "id"-re hivatkozik
                // https://sequelize.org/master/class/lib/dialects/abstract/query-interface.js~QueryInterface.html#instance-method-createTable
                references: {
                    model: "Categories",
                    key: "id",
                },
                // Ha pedig a kategória (Category) törlődik, akkor a kapcsolótáblában lévő bejegyzésnek is törlődnie kell,
                // hiszen okafogyottá válik, hiszen egy nem létező kategóriára hivatkozik
                onDelete: "cascade",
            },
            PostId: {
                type: Sequelize.INTEGER,
                // Nem vehet fel NULL értéket, mindenképpen valamilyen INTEGER-nek kell lennie
                allowNull: false,
                // Megadjuk, hogy ez egy külső kulcs, ami a "Posts" táblán belüli "id"-re hivatkozik
                references: {
                    model: "Posts",
                    key: "id",
                },
                // Ha pedig a bejegyzés (Post) törlődik, akkor a kapcsolótáblában lévő bejegyzésnek is törlődnie kell,
                // hiszen okafogyottá válik, hiszen egy nem létező kategóriára hivatkozik
                onDelete: "cascade",
            },
            // Időbélyegek
            createdAt: {
                allowNull: false,
                type: Sequelize.DATE,
            },
            updatedAt: {
                allowNull: false,
                type: Sequelize.DATE,
            },
        });

        // Megkötés a kapcsolótáblára, amelyben megmondjuk, hogy egy CategoryId - PostId páros csak egyszer szerepelhet a kapcsolótáblában
        await queryInterface.addConstraint("CategoryPost", {
            fields: ["CategoryId", "PostId"],
            type: "unique",
        });
    },

    down: async (queryInterface, Sequelize) => {
        // Ha visszavonásra kerül a migration, egyszerűen töröljük ki a táblát
        await queryInterface.dropTable("CategoryPost");
    },
};
```

Ezt követően fel kell tölteni a három létrehozott táblát az adatbázisba, vagyis meg kell hívni a migrate parancsot:
```shell
npx sequelize-cli db:migrate
```

Ez valami ilyesmi eredményt kell, hogy adjon sikeres migration esetén:
```
D:\szerveroldali\teszt>npx sequelize-cli db:migrate

Sequelize CLI [Node: 14.15.1, CLI: 6.3.0, ORM: 6.9.0]

Loaded configuration file "config\config.json".
Using environment "development".
== 20211112111029-create-category: migrating =======
== 20211112111029-create-category: migrated (0.027s)

== 20211112111041-create-post: migrating =======
== 20211112111041-create-post: migrated (0.017s)

== 20211112111100-create-category-post: migrating =======
== 20211112111100-create-category-post: migrated (0.010s)
```

### Adatmodellek beállítása

Ezen a ponton sikeresen megcsináltuk az adatbázis részt, az `sqlite` fájlunkban szerepel minden tábla. Eljött az ideje, hogy a Sequelize adatmodelljeinek a szintjén is implementáljuk a sok-sok kapcsolatot. Ehhez alapvetően két dolgot kell tenni: a `models` mappán belül a `category.js` és a `post.js`-ben lévő `association` metódusokban megadni a relációkat. Mivel sok-sok kapcsolatunk van, mindkét oldalról (mindkét fájlban) az `belongsToMany`-t fogjuk használni.

Példa, hogy kell kinézzen a két fájl:

`category.js`:

```js
"use strict";
const { Model } = require("sequelize");

module.exports = (sequelize, DataTypes) => {
    class Category extends Model {
        static associate(models) {
            this.belongsToMany(models.Post, {
                through: "CategoryPost",
            });
        }
    }
    Category.init(
        {
            name: DataTypes.STRING,
            color: DataTypes.STRING,
        },
        {
            sequelize,
            modelName: "Category",
        }
    );
    return Category;
};
```

`post.js`:

```js
"use strict";
const { Model } = require("sequelize");

module.exports = (sequelize, DataTypes) => {
    class Post extends Model {
        static associate(models) {
            this.belongsToMany(models.Category, {
                through: "CategoryPost",
            });
        }
    }
    Post.init(
        {
            title: DataTypes.STRING,
            text: DataTypes.STRING,
        },
        {
            sequelize,
            modelName: "Post",
        }
    );
    return Post;
};
```

Fontos, hogy a `belongsToMany`-nek meg kell adni második paraméterben egy objektumot, ami a beállításokat tartalmazza, és itt mindenképp értéket kell adni a `through`-nak, ez mondja meg, hogy melyik az a kapcsolótábla, amelyiken keresztül össze vannak kötve. Ezen kívül a Category a Post-hoz kötődik (tehát ott az 1. paraméter a Post), míg a Post a Category-hoz kötődik, tehát a Post-on belüli belongsToMany 1. paramétere a Category lesz.

Ha eddig mindent jól csináltunk, akkor sikeresen implementáltuk a sok-sok kapcsolatot.

## Sok-sok kapcsolat tesztelése

### Elérhető metódusok lekérése

A leírásban korábban említve volt egy függvény, amivel le lehet kérni a relációk kezelésére vonatkozó metódusokat. Ez a script így néz ki átírva Category-ra és a Post-ra:

```js
const db = require("./models");
const { Category, Post } = db;

const getModelAccessorMethods = (model) => {
    console.log(`${model.name}:`);
    Object.entries(model.associations).forEach(([_, associatedModel]) => {
        Object.entries(associatedModel.accessors).forEach(([action, accessor]) => {
            console.log(`  ${action}: ${model.name}.${accessor}(...)`);
        });
    });
};

;(async () => {
    getModelAccessorMethods(Category);
    getModelAccessorMethods(Post);
})();
```

Ha lefuttatjuk, kiírja, hogy milyen metódusok érhetők el `Category` illetve `Post` alatt:

```
D:\szerveroldali\teszt>node accessors.js
Category:
  get: Category.getPosts(...)
  set: Category.setPosts(...)
  addMultiple: Category.addPosts(...)
  add: Category.addPost(...)
  create: Category.createPost(...)
  remove: Category.removePost(...)
  removeMultiple: Category.removePosts(...)
  hasSingle: Category.hasPost(...)
  hasAll: Category.hasPosts(...)
  count: Category.countPosts(...)
Post:
  get: Post.getCategories(...)
  set: Post.setCategories(...)
  addMultiple: Post.addCategories(...)
  add: Post.addCategory(...)
  create: Post.createCategory(...)
  remove: Post.removeCategory(...)
  removeMultiple: Post.removeCategories(...)
  hasSingle: Post.hasCategory(...)
  hasAll: Post.hasCategories(...)
  count: Post.countCategories(...)
```

Látszik itt is, hogy van egyes- és többes szám, és az is, hogy az Inflection pl. a Category-ból Categories-t csinált, tehát a többesszámosítás nem csak annyiból áll, hogy utána biggyeszt egy s betűt, ennél sokkal hatékonyabb.

### Részletes példa a metódusok használatára

Innentől kezdve már csak az a dolgunk, hogy használjuk ezeket a metódusokat. Ehhez itt egy részletes "playground" példa script:

```js
const db = require("./models");
const { Category, Post } = db;

;(async () => {
    // Korábbi kategóriák, bejegyzések törlése
    // Ez ilyen SQL kéréseket fog kiadni:
    //      Executing (default): DELETE FROM `Categories`; DELETE FROM `sqlite_sequence` WHERE `name` = 'Categories';
    //      Executing (default): DELETE FROM `Posts`; DELETE FROM `sqlite_sequence` WHERE `name` = 'Posts';
    // A kapcsolótábla adatai a CASCADE miatt fognak törlődni
    // A "restartIdentity" elméletileg azt jelentené, hogy az ID ismét 1-ről induljon (az sqlite_sequence-ből törli a táblát),
    // de ez nem működik alapból SQLite-al: https://github.com/sequelize/sequelize/issues/11152
    await Category.destroy({ truncate: true, restartIdentity: true });
    await Post.destroy({ truncate: true, restartIdentity: true });

    // Három kategória létrehozása: c1,c2,c3
    const c1 = await Category.create({
        name: "category1",
        color: "red",
    });
    const c2 = await Category.create({
        name: "category2",
        color: "white",
    });
    const c3 = await Category.create({
        name: "category3",
        color: "green",
    });

    // Két bejegyzés létrehozása: p1,p2
    const p1 = await Post.create({
        title: "post1",
        text: "post1 content",
    });
    const p2 = await Post.create({
        title: "post2",
        text: "post2 content",
    });

    // Összes bejegyzés lekérése
    //console.log(await Post.findAll());

    // Egy adott bejegyzés lekérése a primary key (elsődleges kulcs) alapján, ami alapból az id
    //console.log(await Post.findByPk(1));

    // 1. kategória hozzáadása az 1. bejegyzéshez
    await p1.addCategory(c1); // vagy await c1.addPost(p1);
    // 1. bejegyzéshez tartozó kategóriák neveinek listázása
    console.log((await p1.getCategories({ attributes: ["name"], raw: true })).map((entry) => entry.name));
    // 1. bejegyzéshez tartozó kategóriák megszámolása, ez 1 lesz
    console.log(await p1.countCategories());
    // 1. kategória és 1. bejegyzés közötti kapcsolat megszüntetése
    // Ez nem törli se a bejegyzést se a kategóriát, csak a kapcsolatot szünteti meg köztük,
    // vagyis a kapcsolótáblából törli ki a bejegyzést, ami rájuk vonatkozik
    console.log(await p1.removeCategory(c1)); // vagy await c1.removePost(p1)
    // Ezután ha listázni akarjuk a kategóriák neveit üres tömböt kapunk
    console.log((await p1.getCategories({ attributes: ["name"], raw: true })).map((entry) => entry.name));
    // Adjuk hozzá a p1 bejegyzéshez a c2,c3 kategóriákat
    await p1.addCategories([c2, c3]);
    console.log((await p1.getCategories({ attributes: ["name"], raw: true })).map((entry) => entry.name));
    // Az add mindig csak hozzáad, de a set beállít, tehát ha azt mondjuk, hogy
    // a p1-hez tartozzon a c1 és a c2, nem kell a c3-at eltávolítani, csak azt mondjuk,
    // hogy set c1,c2:
    await p1.setCategories([c1, c2]);
    console.log((await p1.getCategories({ attributes: ["name"], raw: true })).map((entry) => entry.name));
    // Tudunk akár egy 4. kategóriát is csinálni közvetlenül a bejegyzésből:
    await p1.createCategory({
        name: "category4",
        color: "navy",
    });
    // Ilyenkor a Category létrejön a Categories táblában, illetve hozzá is kapcsolódik a bejegyzéshez.
    console.log((await p1.getCategories({ attributes: ["name"], raw: true })).map((entry) => entry.name));
    // Ez úgy is mehet, hogy a kategória alatt hozunk létre bejegyzést:
    const p3 = await c1.createPost({
        title: "post3",
        text: "post3 content",
    });
    // Ilyenkor a Posts táblában létrejön egy új bejegyzés, és hozzá lesz csatolva a c1 kategóriához
    // Ha le akarjuk kérni a c1 kategóriához tartozó bejegyzések címeit, azt így tehetjük meg:
    console.log((await c1.getPosts({ attributes: ["title"], raw: true })).map((entry) => entry.title));
    // Ugye például most a post1 is hozzá van rendelve a c1-hez, de ha ezt ellenőrizni akarjuk,
    // akkor arra ott a "has":
    console.log(await c1.hasPost(p1));
    // Ez true értéket adott, de ha mondjuk a 2. postot nézzük, az már false lesz:
    console.log(await c1.hasPost(p2));
    // A has is megy több posttal, pl most az 1 és a 3 van hozzárendelve a c1-hez, tehát ez truet ad:
    console.log(await c1.hasPosts([p1, p3]));
    // De ha bevesszük a p2-t is, akkor hiába van a p1 és a p3 hozzárendelve, mivel p2 nincs, ez false lesz:
    console.log(await c1.hasPosts([p1, p2, p3]));
    // Ha el akarjuk az összes bejegyzést távolítani a c1-ből, azt így tehetjük meg:
    //console.log(await c1.removePosts(await c1.getPosts()));
    // Nyilván ez egy tömböt vár ilyenkor, a getPosts pedig az aktuális postokat adja meg
    // Konzolra egy kettest fog kiírni, hogy 2 postot vett le

    // Azonban ha kézzel adjuk meg, mondjuk ezt:
    console.log(await c1.removePosts([p1, p2, p3]));
    // Akkor ez is kettőt fog logolni, de a p2-t skipeli, hiszen az nem volt hozzárendelve
    console.log((await c1.getPosts({ attributes: ["title"], raw: true })).map((entry) => entry.title));

    // Kérjük le úgy a Postokat, hogy egyúttal megjelenítsük a hozzájuk tartozó kategóriákat!
    // A findAll, de a többi lekérő metódus tud fogadni egy options objektumot, amiben be tudjuk állítani
    // mondjuk azt, hogy tartalmazzon-e még valamit, ahogy alább tesszük is.

    console.log(
        JSON.stringify(
            await Post.findAll({
                include: [
                    {
                        model: Category,
                        // "as" tulajdonképpen nem kell, ha a models-ben nem adtunk meg alias-t
                    },
                ],
            }),
            // JSON.stringify testreszabása
            null,
            4
        )
    );

    // Ugyanez, picit jobban testreszabva:
    console.log(
        JSON.stringify(
            await Post.findAll({
                // A bejegyzésből csak az id, title és text mezők jelenjenek meg
                attributes: ["id", "title", "text"],
                // És a bejegyzés tartalmazza még...
                include: [
                    {
                        // ... a kategória modelt ...
                        model: Category,
                        // ... mint "Categories" alias ...
                        as: "Categories",
                        // ... és ezeket a mezőit kérje le:
                        attributes: ["id", "name"],

                        // Ez pedig azért kell, hogy a kapcsolótáblát ne szemetelje bele,
                        // a legjobb ha kikommentezed az alábbi sort és megnézed, mi változik
                        through: { attributes: [] },
                    },
                ],
            }),
            // JSON.stringify testreszabása
            null,
            4
        )
    );

    //await p2.addCategory(c1); // Alapból csak az 1-es bejegyzéshez tartozik, de itt megnézhetjük, mi történik, ha a 2-eshez is rendelünk kategóriát

    // Csak azon bejegyzések lekérése, amelyekhez tartozik legalább egy kategória
    console.log(
        JSON.stringify(
            await Post.findAll({
                // A bejegyzés minden mezőjét lekérjük, kivéve a timestamp-eket
                attributes: {
                    exclude: ["createdAt", "updatedAt"],
                },
                include: [
                    {
                        model: Category,
                        // Megköveteljük, hogy a szülőhöz tartozzon a gyerek (Post-hoz a Category, hiszen a Post-ra hívtuk a findAll-t)
                        required: true,
                        // Továbbá megmondjuk, hogy a gyerekből (Category) semmilyen mezőt nem akarunk látni, csak a bejegyzésre vagyunk kíváncsiak
                        attributes: [],
                        //through: { attributes: [] },
                    },
                ],
            }),
            // JSON.stringify testreszabása
            null,
            4
        )
    );
})();
```
