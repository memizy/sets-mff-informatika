# 10.přednáška - Document databases
- documenty jsou samo popisné, hierarchické, mohou mít i hashmapy, kolekce, nested documenty v sobě
- předpokládá se že dokumenty v jedné kolekci jsou podobné (o stejné části reality), schéma se může lišit
- vlastně jsou to takové key-value story kde jsou hodnoty examinable - můžeme nad daty dělat různé dotazy, díky tomu že víme strukturu se dají vybudovat různé indexy ...
- query language pomocí jsonu například specifikujem property a pro ní hodnotu
- use casy - event logging  - mohou být rozděleny dle aplikace či typu
- CMS, blogging platformy - nepotřebujem se příliš nad daty složitě dotazovat, prostě děláme CRUD pro uživatele
- web analytics nebo real-time analytics - části dokumentů mohou být snadno updatovány, například přidání nové metriky
- E-commerence aplikace - flexibilní schéma pro produkty a objednávky, data migration není drahá
- nehodí se na složité transakce většinou změna atomická pro jeden dokument
- některé to ale podporují i mezi dokumentově např. RavenDB
- když se mění dotazy které potřebuju pokládat, nemusí se hodit hyerarchie kterou jsem stanovil a data by to chtělo normalizovat (rozložit)

### MongoDB
- indexy nad částmi dokumentů, replikace, eventuální konzistence, automatické škálování a failover managment
- podpora por MapReduce
- dokumenty skládáme do collections (jako tabulka), každý dokument má identifikátor _id, mohu mezi dokumenty dělat reference
- používá JSON - je to uloženo jako BSON což je binární, úspornější reprezentace JSONu
- má maximální velikost 16MB aby se to vešlo do paměti, GridFS tool umí soubory dělit
- omezení na jména fieldů _id je reservováno pro primární klíč, ten je unique a immutable, může mít libovolný typ krom pole
- fieldy nesmějí začínat $ ten je por operátory a ani . který je pro přístup k fieldům

- collekce nevyžadují stejnou strukturu dat - flexibilní schéma, ale dává smysl spolu ukládat stejné věci
- otázka je jak navrhnout strukturu dokumentů, můžeme mít dokumenty vložené (prostě víceúrovňový json) nebo reference
- záleží na dotazech které budeme pokládat, často máme redundantní data
- reference mají více flexibility vzhledem k různým dotazům

- operace - create, update, delete
- modifikujeme vždy všechny dokumenty v kolekci, specifikujeme kritéria které modifikovat či smazat
- db.users.insert({ property: value ...}), můžu nebo nemusim specifikovat _id, když už tam je dostanem error
- db.invertory.update( { properties co které musí mít }, { $set: {qty: 10}} (co chcem nastavit), { upsert:true (pokud neexistuje vytvoř bude mít properties co musí mít a qty se specifikovanýma hodnotama)} )
- db.inventory.save({properties}) - vytvoří se jen pokud _id není specifikované nebo v kolekci neexistuje jinak se modifikuje
- db.inventory.remove ( { type: "food"}, 1) - smažeme jeden dokument co to splňuje
- db.inventory.update ( { type: "book"}, { $inc: {qty:-1}}, { multi: true} ) - změníme to pro všechny takové dokumenty
- db.user.find( { type: { $in: ['food', 'snacks']} age: {$gt: 18} }).sort( {age:1}) - čárka dělá že musí platit všechny ty věci
- můžu dělat i or například db.inventory.find( { 'type.foodType': 'food', $or: [ {qty: {$gt:100}}, { price: {$lt: 9.95}}]}) 

- !!! pomocí . notace nemusíme psát celé vnoření ale zacílíme přímo, ale dělá to něco jiného než pouhé vnoření
- u vnoření tam musí být přesně ty properties co uvedeme se správnými hodnotami ve správném pořadí
- u tečkové hodnoty tam pouze musí být správná property se správnou hodnotou a jinak tam může být cokoliv dalšího
- můžu tam dávat i pole když tam dám find ( { tags: 'fruit'}) tak v tom poli tags (když je ot pole) někde musí být fruit
- můžu dát to samé jen 'tags.0' a timříkám že ot musí být na první pozici, když chci přesně hodnoty z pole co tma musí být tak je vypíšu do pole tags: []

- jako druhý parameter find můžu specifikovat co přesně chci dám tam např , {item: true, qty:1} - dostanu jen item a qty může tam být 1 či true a defaultně vždy _id
- můžu tam dát _id: 0 pak id nedostanu, krom id musim vždy specifikovat buˇfd všechno co chi s 1 nebo všechno co nechci s 0

- za find můžu dát sort a ňáký počet položek a do pomocí -1 nebo 1 v jakym pořadí
- za find taky můžeme přidat .explain() čimž nám vysvětlí plán dotazu - můžu se podívat jestlise mi index použil

#### Indexy
- bez indexů musíme oskenovat každý dokument v kolekci abysme našli co potřebujeme
- můžeme nad položkou nebo množinou položek vybudovat index B-strom podle hodnot v poli
- definovaný na collection levelu, zrychlíme odtazy
- například když výsledek co vracíme chceme setřídit podle položky co máme v indexu tak už to rovnou máme
- nebo když chceme je větší nebo menší pro indexovanou hodnotu

- defultně máme index pro _id, můžeme mít indexy pro jedno, více polí i multikey index pro položky v polích, vytvoří separátní index pro každou položku v poli
- máme taky geoprostový indexy - geometrický 2d a sférický na kouli, máme textové indexy na full-text vyhledávání, máme hashované indexy (můžeme matchovat jen rovnost)
- použijeme db.people.ensureIndex( {"phone-number": 1, category: 1} ), můžu tam přidat jako další parameter {}, {unique: true} - pak nemůžou ostatní aplikace přidávat dokumenty které mají danou hodnotu stejnou (vlastně taková definice klíče)
- ensureIndex( {_id: "hashed"})

#### Replikace
- master/slave replikace
- replica set = group of instances that host the same data set, skupina uzlů které drží repliky pro daný kus dat
- primary - master - dostává všechny zápisy a ty replikujou secondaries - slaves

- zápis se zapíše na primary a zaznamená operace do oplogu, který pošle slavům, kteří je také aplikují
- číst můžeme odkuďkoliv, můžeme si nastavit preferovaný mód čtení - dává smysl když potřebujeme nejnovější dat
- primary, primaryPreferred, secondary, secondaryPreferred, nearest

- když mi primary spadne, tak se spustí election, trvá asi minutu kdy nemůžu do dané repliky zapisovat
- můžemem pro to definovat různé parametry - v rámci repliky se posílá každé 2s Heartbeat a když se uzel 10s neoszve je označen jako nedostupný

- uzlům můžeme nastavit prioritu - vyšší priorita aby se stal masterem
- když má prioritu 0, tak se nemůže stát masterem ani nemůže zahájit volby ale může volit

- když má aktuální primary nejvyšší prioritu a odpovídá tak pohoda
- jakmile je tam uzel s vyšší prioritou a chytne se do 10s a za poslední oplog entry, tak má šanci stát se primary
- connections - node nemůže být primary když se nezvládne připojit k většině uzlů

- aktuální primary odstoupí když má nějak secondary vyšší prioritu nebo když nemůže komunikovat z většinou replica setu
- replica set volí toho kdo má nejvyšší prioritu - na začátku maj všichni 1
- ten kdo první dostane většinu hlasů se stane primary
- můžeme definovat uzly co nesměj hlasovat, dá se jim dát i více hlasů, ale lepší je to nedělat a používat prioritu
- všichni mohou poslat na volby veto např. když volby vyhlásí někdo kdo nemá nejnovější informace
- taky můžeme definovat uzel arbiter, nemá data ani nemůže být primary, jen hlasuje (když mám třeba sudej počet)
- můžeme mít hidden uzly ze kterých se nedá číst
- také můžeme mít delayed - který drží běžící historický snapshot

#### Sharding
- každý shard (kousek dat) je uložen v nějaké replika set
- to kde to je drží config server (má cluster metadata) - doporučeně jsou 3
- pak máme víc Query routers - které zpracovávají požadavky, zjistí od config serveru kde to najít a Router tam šáhne a získá je

- data rozdělujeme podle shard key - indexované pole které má každý dokument v kolekci, může být i compound, je immutable
- jsou rozdělený do chunků - buď range-based nebo hash-based
- když chunk příliš vyroste na velikosti tak je rozdělen - menší chunky víc rovnoměrná distribuce ale častější migrace, větší chunky - méně migrací
- velikost chunku 64MB

##### Range-based Partitioning
- každý shardkey dá nějakou hodnotu od -nekonečna do +nekonečna, linka je rozdělená na nepřekrývající se chunky
- dokumenty s blízkými shard key nesjpíš budou ve stejném chunku
- máme efektivnější range queries, ale data mohou být nerovnoměrně rozdělena

##### Hashed-based Partiotioning
- data jsou distribuované rovnoměrně, ale tim pádem blízké dokumenty nebudou blízko a range query bude muset projít většinu chunků/shardů

##### Journaling
- mongoDb ukládá změny v paměti a v journálu než se změny porvedou na data files
- abychom mohli přivést databázi do konzistentního stavu po hard shutdown, když je normální shutdown, tak se nejdřív všechno uloží
- journal directory - má journal fily
- journal file - je append only, smazán když byly všechny writy provedeny
- když má 1GB dat tak se vyvoří další

##### Transakce
- operace jsou atomický na úrovni jednoho dokumentu, můžeme dělat nested dokumenty kde to funguje
- pro víc dokumentů to atomické není
- můžeme ale přidat db.foo.update({field1:1, $isolated:1{, ..., {multi:true}}) - nikdo neuvidí změny dokud se neprovedou na všech dokumentech
- dvou fázový commit - na jedno místo zapisuji transakce, informace o krocích operace tam uložím
- všichni ví že tam jsou popsaný běžící transakce
- převod peněz - db.transactions.save({source: "A", destination: "B", value: 100, state: "initial"}), mají různé stavy pending, applied, done ...
- do jsonu přidáme že jen pokud přidání peněz není už v prováděných transakcích tak to do nich přidej a proveď to
- stejně tak pak s odebráním
- takže když mi to spadne v půlce tak já vim jestli už byla nebo nebyla operace provedena, neb zapsání do pendingTransactions a provedení změny je atomické
- když chceme udělat rollbak tak můžeme udělat atomicky inverzní transakci, když je v těch pending transakcích tak to vrátim a vyhodim jí z pending transakcí
- když tam chceme mít více aplikací tak musíme ke transaction přidat jaká aplikace transakci vyvolala
- monitoring, kerberos autentizace v enterprise verzi

# 11.přednáška - Graph databases
- Nodes - instance objektů, mají properties
- Edges - mají start a end node, mají směr, typ, mohou mít properties
- Nody organizovány pomocí vztahů, můžeme hledat zajímavé paterny v grafu
- mohou mít sekundární vztahy - jako categorie, časy, předpočítaný cesty

- properties nodů či hran mohou být indexovány, můžeme dělat různá grafová vyhledávání
- use casy - propojená data - sociální sítě, kde je hodně linků
- routing, dispatch a location-based services - kde potřebujem udělat delivery jsou nody a edge jsou jejich vzdálenosti
- doporučovací enginy - hodně vztahů, máme podle čeho doporučovat
- nehodí se na - chceme něco dělat se všema uzlama, změna všude
- je těžké graf distribuovat na více uzlů

- výhoda oproti relačním databázím kdž chceme přidat nový typ hrany, či vrcholu třeba, nemusíme měnit všechny dotazy což by jsme při reprezentaci grafu v relační databázi museli
- v relačních databázích to také strukturujeme podle toho jak graf chceme procházet, když se to mění je to problém neb nemůžeme snadno změnit strukturu dat
- také nemusíme dělat spoustu joinů pro hodně spojení

### Neo4j
- má plné ACID transakce
- může mít až několik miliónů uzlů, vztahů a properties
- umí škálovat pro čtení - zreplikovat celou kopii grafu na další uzly, pro zápis to umí placená verze
- uzly i hrany mohou mít vlastnosti což jsou dvojice klíč-hodnota, hodnota může být i pole, nemůže tam být null, pak tam klíč vůbec neni
- relationships (hrany) - jsou orientované, ale mohou být procházeny i z druhé strany (se stejnou efektivitou a tak směr ignorovat), vždy mají start a end node, node může mít relationship sám se sebou
- má JavaAPI, Gremlin (uzly i hrany mají identifikátor a hrana má label a hrana i node mohou mít vlastnosti, v jazyce má loopy, použití iterátoru) a jazyk CYPHER (podobný SPARQLu)
- podporuje transakce

### Traversal
- ovnivněn více parametry
- expanders - co chceme projít, taky můžeme specifikovat jestli chceme oba směry hran nebo jen 1
- order - BFS/DFS
- uniqueness - lze nody či vztahy, cesty opakovat
- evaluator co se má vrátit a kdy skončit hledání
- starting nodes - kde hledání začneme
- traverser - jak chceme projít výsledek - po uzlech, či po hranách ...

### Cypher příkazy
- START - z jakých nodů vycházíme
- MATCH - po jakém patternu se chceme pohybovat
- WHERE - filtrujeme, RETURN - co se má vrátit
- CREATE - vytváří nody a vztahy
- DELETE - maže, SET - nastavuje hodnoty pro properties
- FOREACH - udělá update akci pro každý element z listu
- WITH - rozdělí dotaz na více odlišených částí (když chci pak jednu použít pro druhou)

- CREATE (a {name : 'Andres'}) RETURN a;
- když chcmeme k uzlu přidat hranu MATCH (a {name:"Andres"}) CREATE (a)-[r:FRIEND {name: "test"}]->(b {name:"Jana"}) return r; vytvoří hranu a Janu
- můžu takhle najednou vytvořit i cestu jako předtim ->()<- z obou stran do nepojmenovanýho nodu co dostane nějaký id
- MATCH (n {name: 'Andres'}) DETACH DELETE n - smaže Andrese i hrany s nim, podobně bych mohl mít Andres-[r:KNOWS]->() DELETE r což smaže hrany z něj
- MATCH p = (begin)-[*]->(END) WHERE begin.name = 'A' AND END.name = 'D' FOREACH (n IN nodes(p) | SET n.marked = TRUE)
- u properties můžeme pomocí IN dělat výčet hodnot, a můžeme to testovat i na regex
- můžu dělat různý agregace - count, sum, min, max ...
- dotaz mi vždy vrátí kolik vytvořil, nodů, kolik hran a kolik properties nastavil

### Transakce
- má ACID -> všechny zápisové operace musí být provedeny v transakci, mohou být i zanořené (rollback zanořené znamená i rollback vnější)
- pro čtení žádné zámky nejsou - dva ready mohou přečíst jinou hodnotu, lze to explicitně nastavit ale dost se to tím zpomalí
- pro psaní - používáme zámky - všechny modifikace pro transakci jsou udržovány v paměti, velké updaty je potřeba rozdělit
- zámky máme defaultně pro všechny zápisy, mazání, když pracujem s hranou, zamknou se i koncové nodes
- mohou nastat deadlocky a jsou detekovány a dostanem vyjímku
- s nodem se smažou i properties, pokud má hrany tak se také smažou, je pak potřeba DETACH DELTE
- na node si můžeme šáhnout dokud nebyl commitnutej ale nelze ho měnit, když ho zkusíme měnit po commitnutí odebrání tak dostaneme vyjímku 

### Indexing
- má vlastní jméno co mu specifikujeme
- indexuje nějaké nody či hrany
- spojujeme key-value páry s nějakým počtem entit, můžemem tam mít ten samý záznam i vícekrát, starou hodnotu je potřeba smazat jinak se vždy přidá nová
- indexy děláme samostatně pro nody a hrany
- například tam můžu mít jeden node s key name a value pokaždý jinou kdy druhá bude třeba přezdívka
- z indexu můžu smazat všechny odkazy na uzel nebo jen odkazy s klíčem name nebo odkazy s konkrétním nodem, key i value
- v indexu můžeme hledat dostaneme zpátky množinu nodů jeden pro každý záznam
- pomocí query můžeme ve value používat regex
- můžeme hledat i v hranách i podle toho čim začína či končí hrana
- díky těmhle indexům pak nemusím prohledávat celej graf když chci třeba všechny herce
- máme jeden automatický index pro vrcholy a hrany, nastavíme na úrovni databáze podle čeho chceme indexovat může tam být víc properties
- distribuce - v enterprise verzi má hodně velké limity, umí distribuovat

### High Availability
- můžeme nastavit několik slave databází který budou přesné repliky mastera
- můžeme tim škálovat pro čtení, změny jsou eventually propagated na slavy
- zápis na slava - okamžitá synchronizace s masterem
- každá databázová instance umí být slave i master, když se má někam připojit jako slave a nepovede se to tak ze sebe udělá mastera
- failure - vybere se nový master, když se starý znova připojí, tak se z nového stane zase slave, odpadlý slave se synchronizuje s clusterem

### Ukládání
- máme několik souborů daných typů a každý záznam má fixní délku a tím pádem se tam můžemem pohybovat pomocí offsetů
- properties jsou uloženy jako spoják key + value + reference na další property, klíč a hodnota jsou odkazy na řetězec
- nody mají dva odkazy jeden na začátek spojáku vlastností a druhý na začátek spojáku hran
- můžu tam mít i místa který nemusí být použitý, první byte obsahuje informace jestli je místo použité
- relationships - mají odkaz na property na počáteční a koncový uzel a odkaz na přdchozí a následující vztah pro počáteční a koncový uzel 
- vlastně se nám to celý rozpadne na mnohem složitější graf kde máme ty spojáky


# 12.přednáška - Multi-model databáze
- polyglot persistence - pro každou část aplikace použijeme ten datový model co se pro daná data nejvíc hodí
- problémem je jak se dotazova cross modelově
- multi-model - jedna databáze, která podporuje více datových modelů
- jsou různé názory, stonebraker tvrí, že nelze mít všechno efektivní v jednom enginu
- OctopusDB - tvrdí že to umí, všechna data se ukládají v logu, log provedených operací s daty
- podle toho jaký typ zpracování potřebujeme tak se log transformuje do view v daném datovém modelu
- query optimization, view maintenance, index selection z toho všeho je jeden problém - storage view selection
- AsterixDB - mají flexibilní datový model - datový model databáze se může měnit
- datový model můžeme definovat pro část dat - closed (můžeme predefinovat co objekty musí mít), a pro část to např. necháme bez modelu - open
- podpora pro Hadoop-based query platforms, key/value a semi-structured data managment

- dřív už byly object-relational databases - použití ORM (může to podporovat přímo databáze) - můžemem ukládat data prostě z objektů
- skoro všechny databázové systémy už jsou multi-model
- záleží na tom jaký typ byl v databázi ten výchozí - do něj se totiž nemapovávaj ostatní

### ArangoDB
- byla dokumentová, přidali podporu pro grafy a key/value
- vrcholy a hrany grafů jsou dokumenty
- tabulky - speciální typy dokumentů s pravidelnou strukturou
- dotazy děláme v jednom jazyce kde se zvládneme dotázat na vše

### OrientDB
- podpora pro grafový, documentový, key/value i objectový model, dotazy v SQL rozšířeném o graph traversal
- vznikla z objektové databáze, v základu grafová tam hned byly prostě pomocí na sebe ukazujících objektů
- SELECT expand( out("Knows").Orders.orderlines.Product_no) FROM Customers WHERE CreditLimit > 3000

### Klasifikace multi-modelových databází
- základní rozdělení podle původního modelu ze kterého vychází, postupně přidávaly nové modely často relační přidaly json
- toto rozšíření o nové modely mohli dělat různými způsoby
- kompletně nová způsob ukládání - např. XML-enabled databáze - mají pro XML speciální úložiště a indexy
- rozšíření původní ukládací strategie - ArangoDB - speciaální edge collection které mají informace o hranách v grafu
- nový inteface pro originální strategii - MarkLogic - přidání Jsonu když byli v základu XML databáze
- (skoro) žádná změna není potřeba - do dokumentové databáze chci ukládat key/value

### Postgre-SQL
- Relational Multi-model DBMS
- mají speciální typ pro json - JSON a JSONB je to prostě jedna položka v tabulce, uložíme tam celý json a máme nad ním různé indexy aby jsme se v něm mohli dotazovat
- podpora i pro ukládání arrays
#### Dotazy
SELECT name,
    orders->>'Order_no' as Order_no,
    orders#>'{Orderlines,1}'->>'Product_Name' as Product_Name // Jendička znamená druhou položku
    FROM customer
    where orders->>'Order_no' <> '0c6d511f';

### Column Multi-model DBMS
- např. v Cassandře uděláme CREATE TYPE myspace.myorder (order_no text, orderlines list<frozen <orderline>>); ten pak můžu prostě použít v hlavní tabulce
- můžu i json vytvářet ze sloupcových dat, možnost převádění

### Key/Value Multi-model
- Oracle NoSQL DB - původně key-value, přidali podporu pro objektově relační model create table Customers (orders array (record order_no ...))
- o dost jiný jazyk než Postgre-SQL - přístup do polí jako v C# přes [] a k properties přes .

### Dokumentové datbáze
- MarkLogic - přidali uzel pro pole hodnot do stromu jsonu a ten uložili do XML - indexují oboje stejně, mají nějaké indexy nad daty defaultně a další můžeme přidat

### Grafové databáze
- OrientDB - v grafové databázi definujeme uzly jako třídy a třídy mohou být mezi sebou vztahu buď jako reference v paměti a nebo embedovaný vztahy zapsaný v recordu

### Multi-model dotazovací jazyky
- simple API - ulož vrať smaž, typicky pro key/value
- SQL Extensions a SQL-Like jazyky - nejčastěji, každý ale podporuje trochu jinou čáast a má trochu jiný vlastní dialekt

### Vyhodnocování dotazů mezi modely
- čim víc se liší nový model od původního, jakou jsme zvolili strategii pro přidání, tim je to těžší
- většinou se snažíme co nejvíc použít to co už máme
- optimalizace B-tree/B+-tree, hashování, invertovaný indexy, native XML index

# 13.přednáška - NewSQL a Array databáze
- chceme relační model s ACID, ale chceme ho škálovatelný a distribuovatelný
- buď se přidává k distribuovaným systémům výhody relačního modelu a ACID - např. VoltDB
- nebo DBMSs rozšířené pro horizontální škálovatelnost - JustOne DB
- v cloudu - NewSQL as a service - scalable relační databáze - Microsoft Azure database

- hodí se například když mám aplikaci která historicky pracuje s relační databází, ale prostě začnu mít hodně dat
- nebo jsem měl velká data a teď jsem začal potřebovat silnou konzistenci

### VoltDB
- původně H-System - vyvíjen na US univerzitách s Intelem a teď je z něj db
- pro uživatele (programátora) je to klasická relační databáze
- automatická distribuce dat v rámci clusteru, můžeme specifikovat podle čeho se distribuuje
- shared-nothing architecture - nody v clusteru nesdílí paměť či disk, komunikují pomocí zpráv

- objevili že tradiční relační databáze tráví pouze asi 10% času dotazováním, většina času se stráví na
- Page Buffer Managment - zápis databázových recordů do fixed, size stránek, loadování, clean a dirty pages
- Concurrency Managment - transakce musí být ACID a databáze má více vláken, takže datové struktury musí být Thread save což je dělá složitější

- tyhle věci zoptimalizovali
- s daty se pracujem v paměti - aby jsme nemuseli stránkovat, dělají se snapshoty nebo se pracuje s command logem, minimalizujeme přístupy na disk
- všechny operace jsou jednovláknový - máme jednoduší datové struktury
- distribuce - každý dotaz je uložený jako stored procedura předcompilovaný, když distribuuju tak distribuuju nejen data ale i procedury
- lokálně spouštěné transakce na jednotlivých partitions systému jsou serializovaný a tim pádem nepotřebujeme zámky
- replikace - peer to peer - partitions, každý node má unikátní část dat i s procedurami pro ně
- celá databáze se také může zálohovat master-slave nebo peer to peer způsobem
- když potřebujeme data jen z jedné partition je to jednoduché, když z víc tak se jeden uzel stane koordinatorem a stará se o sestavení výsledků z více nodů

### Array databáze - SciDB
- databáze multidimezionálních polí
- pozorování z různých přírodních věd - senzory, klima, vesmír ..., data také často v čase další dimenze, vlastnosti při pořízení
- chceme mít dotazy a hlavně analýzy efektivně
- updaty neděláme, prostě vytvoříme data znovu, multidimezionální array je sorted
- máme dva jazyky AFL a AQL (kterej je SQL like a compilovaný do AFL)
- nejdřív vytvoříme pole s typama a případně velikostí
- CREATE ARRAY A <x: double, err: double> [i=0:99,10,0,j=0:99,10,0]
- nejdřív specifikujeme co bude jako hodnota a pak dimenze (můžou bejt pojmenovaný, třeba row (i) a column (j)),
- kde každá má souřadnice, můžou mít 0:* pak velikost data chunků a jak moc se mohou data chunky překrývat
- přes chunky se distribuuje, jeden chunk kolem 10-20MB
- overlaping nemusim využívat, ale může mi zrychlit některé dotazy
- LOAD A FROM 'soubor';
- SELECT i FROM A; [(0), (1) ...] pro jednorozměrné pole vypíše coordinate i, takže prostě co jsme tam nastavili
- SELECT val_a FROM A; [(1), (56), ...] vypíše uložené hodnoty
- můžeme používat různé funkce, WHERE, JOINy, GROUP BY
- SELECT sqrt(val_b) FROM B WHERE j > 3 AND j < 7 - vrátí mi zas celé pole [(), (), (), (), (10.8), (10.5), (10.3), (), (), ()] ostatní hodnoty tam nebudou
- můžeme spojovat dvě arraye a dát je do 3 SELECT * INTO C FROM A,B; dotanu pak dvojice [(1, 101), (2, 102) ...] z prvního a druhého pole, můžu dělat self-join, sčítat hodnoty ...

- každý array má jméno a ordered list (pořadí je důležité) pojmenovaných dimenzí a cell je tuple pojmenovaných typovaných atributů
- scidb umí sparse (matice sousednosti grafu) a dense (obrázky) matice, time series data
- chybějící informace můžeme nagradit pomocí missing code který pro array specifikujeme, můžeme si nastavit jaké budou
- umíme také dělit políčka na více na agregovaných hodnot, dělat různé věci z lineární algebry ..., pracovat s polem po mřížkách a tam třeba počítat průměr

- složité dotazy se dají vyhodnotiti různými způsoby - umí vybrat nejlepší postup neb víme co je komutativní ...
- pracuje s paralelizmem aby to bylo co nejlepší
- nematerializujeme mezivýsledky jako Hadoop MapReduce, jen když je to nutné
- tomu jak data prochází přes jednotlivé části se říká data pump
- můžeme si definovat temporary arrays - drží se v paměti, nemají ACID, když selže SciDB tak se označí jako unavailable ale nesmaže se, nemaj verze vždy přepisujem
- array attributes mohou mít nullable a mohou mít default a jestli je chceme komprimovat

- data co jsou blízko v systému souřadnic jsou na jednom chunku a chunky blízko u sebe na jednom nodu
- různé atributy jsou ukládány zvlášť do různých chunků ale na stejné nody aby šli snadno zkombinovat
- prázdné chunky jsou zakódovány bitovou maskou

### Search Engines - Elastic Search
- data mohou být strukturovaná, semistrukturovaná, nestrukturovaná ...
- nemáme relace, constraints, joiny ani transakce
- použití - relevance-based search, full text search, synonym search, log analysis, geospatial search
- umí distribuovat, narozdíl od NoSQL nepočítají příliš s updaty, hlavně pro analýzy - předpokládáme že data se nám sypou dovnitř

- ElasticSearch - full-text search engine
- komunikace pomocí jsonu přes HTTP, má clienty pro Javu, C# ..
- dobrý s Logstash - pro logy, Kibana - analýzy a visualizace

- přidávání a indexace nových dat je velmi rychlá - stačí kolem 1 sekundy kdy můžeme od přidání vyhledávat
- index používají i jako název pro podobné dokumenty "tabulka"
- indexy se dělí na shardy, shardy mohou mít repliky, rebalancing a routing dělány automaticky
- každý node může být coordinator pro delegování operací na správné shardy

- při dělání indexu specifikujem počet shardů a jejich replik, každý shard je sám plně funkční a nezávislý index - snadná paralelizace, availability
- defaultně 5 halvních shardů pro index a každý s 1 replikou

#### Příkazy - přes HTTP
- věci s podtržítkem jsou klíčová slova příkazu
- GET /_cat/indicies?v - dostanem všechny indexy
- PUT /customer?pretty vytvoříme index customer a hezky vytiskneme výsledný json
- PUT/GET/DELETE /customer/_doc/1?pretty {"name": "JohnDoe"} - obsah jen pro put
- nové id vytvoří nový dokument, žádné náhodné id a stejný přepíše, ale to prostě znamená, že se smaže a vytvoří znova
- můžu dělat změny i pomocí scriptů např POST /customer/_doc/1/_update?pretty {"script": "ctx._source.age += 5"}
- ctx._source je obsah dokumentu, ctx._index jsou metadata
- Můžu za _doc dát /_bulk a pak můžu dělat víc dokumentů naráz a u každého uvedu _id a můžu i uvést operaci

- parametry vyhledávání můžeme předávat už v uri nebo v body pomocí jsonu
- např. GET /bank/_search?q=*&sort=account_number:asc&pretty - bereme vše z bank a seřadíme to
- ve výsledku vidíme took (jak dlouho v milisec to trvalo najít), timed_out, _shards - kolik jsme prohledali shardůkolik se přeskočilo ..., hits - kolik dokumentů vyhovovalo
- hits.hits - defaultně array prvních 10, hits.ssort - sort key pro výsledky, celeé to dostaneme v jsonu
- místo toho to může být json (Query DSL) s GET /bank/search { "query": { "match_all": {} }, from: 10, size: 10, _source: ["account_number", "balance"] - beru jen tyto dvě položky, "sort": {"balance": {"order": "desc" }}}
- místo match_all můžu mít různá filtrování např. "match": {"address": "mill alne"} - ty co mají v adrese podřetězec mill nebo lane
- match_phrase když chci přesně to jedno, když dám dva matche oddělené čárkou tak tam musí být oboje když mám předtim "must", nebo jedno když "should", máme "must_not"

- u výsledků máme skóre, které nám říká jak moc dokument na dotaz sedí například když tam máme nějaká slova, u dotazu dostaneme maximální skóre a skóre jednotlivých výsledků
- můžeme podle toho třídit
- data můžeme agregovat pomocí "group_by_state": {"terms" - {"field": "state.keyword"}, "aggs": {"field": "balance"} }

- ElasticSearch používá Apache Lucene - což je high performance text search knihovna - wildcardy, proximity dotazy (do 20 slov od) ...
- je to vlastně ivertovaný index - dokument rozsekaný a máme index kde slova najdeme
- indexy pro každý dokument, dokument má jeden či více fieldů což jsou name-value páry jako title, content ...

# 14. přednáška - Polystores
- alternativa k Multimodelu, místo toho aby jeden model předělaly do výchazího tak umí všechny v nativním data modelujem
- umí i dotazy mezi modely
- jeden systém neumí optimálně všechno, spojení modelů zajistí polystore
- single-model systémy už jsou mature, optimalizované a vytuněné takže je dobré je využít
- nejsou moc v průmyslu, jsou spíš v akademické sféře, nejsou moc snadno stáhnutelné a nainstalovatelné

### Typy polystorů
- Loosely-coupled - polystore komunikuje s wrapperama systémů, který mají autonomii netrápí ho jak to dělá, wrapper dělá překlad do jazyka databáze z SQL kterým se dotazujeme
- Wrappery mohou dělat různé překlady - Spark SQL ...
- výsledek se posílá a vrací ve společném formátu, global common language
- snažší extensibilita

- Tightly-coupled - přímo pracují s lokálními interfacy, mají menší autonomii, mediator (polystore) komunikuje přímo s databází v jejím jazyce
- mají blíž k Multi-model systémům, oproti MM jsou často hlavně pro čtení a analýzu dat a dotazujeme se nad nimi, transakce napříč systémy nemáme, mají "snažší" migraci mezi systémy - v jedné smažu v druhé vytvořim
- Polybase, HadoopDB - díky nižší autonomii větší efficiency pro Big Data analytics, ke storům se přisupuje v jejich jazyce, umí data migrovat, těžší přidání nového systému

- Hybridní přístupy např. podle systému - někde mám wrapper jinde ne - např. BigDAWG
- stále často nepodporují cokoliv, často relační databáze s HDFS, nejvíc toho má BigDAWG od Stonebrakera

- správa schématu - LAV - lokální schémata jsou funkcí globálního schématu
- GAV - globální schéma je set pohledů na lokální schémata 

- někde se optimalizační plány se dělají s informacemi od jednotlivých modelů, u nějakých říkáme kam data uložit někde se to dělá samo
- heterogenita co umí, autonomie, flexibilita, vše se liší systém od systému
- už dřív byly některé systémy které překládali jazyky různých databází do sql
- polystory mají hodně otevřených problémů, i multimodel databáze

#### BigDAWG
- myšlenka island of information - má datový model, operace a storage engine
- můžeme dělat cross-island queries
- mají PostgreSQL, SciDB, Accumulo - relační, array, sorted key:value
- není jeden společný jazyk - Shim se připojí na ostrov k jednomu nebo více storage engines a mapuje island language na engine language
- Cast - operátory na převod datasetů mezi ostrovy aby se mohly analyzovat na enginu pro ně nejvíc vhodným
- Např. relační ostrov klidně může mít část dat v SciDB např. kvůli snažší analýze nebo duplikování
- optimizer - zjistí na jakém enginu to spustit a jak optimalizovat víc plánů
- monitor mu řekne jaký z nich vybrat na základě dat z předchozích queries
- executor - jak dát různá data dohromady a provede query
- monitor - stará se o migraci když je potřeba - třeba zjistim že by se to pro queries víc hodilo

# 15. přednáška
### Důkaz CAP
- prostě že když nemám stálé spojení tak nemůžu mít silnou konzistenci u distribovaného systému neb pošlu na repliku změnu ale ona tam nedojde
- Consistency a Availability - Single-site databáze, RDMBS
- Consistency a Partition tolerance - Distribuované databáze
- Availability a Partition tolerance - Web caching, DNS

- Consistency - každej read po writu čte novou hodnotu
- Availability - otázka na uzel vrací odpověď o datech, neříkáme za jak dlouho
- Partition Tolerance - síť může ztratit libovolný počet zpráv

- když si nody povídají pomocí zpráv - asynchroní síťovej model
- důkaz sporem zápis z jedné množiny uzlů nepřejde do druhé, pak to není konzistentní když čtu z druhé části, která musí odpovědět, nemůže teda vrátit správnou hodnotu
- i když se zprávy neztrácej tak zprávy mohou být zdržený dýl než čas pro availability a mi nemáme jak určit jestli jsou zdržený či vůbec nebyly poslaný
- můžeme měřit čas od požadavků, ale ani tam nelze mít vše, zas to nejde jako u 1, když do G2 nic nechodí, s readem v G2 začnu až když mi doběhne timer v G1, to druhý už tam neplatí


### Transakce
- máme dvou typů
- Systémová transakce - zámek na krátkou dobu, odečtení a přičtení peněz k účtu
- Business transaction - láhev whisky v košíku - od přidání do koupi, nechci mít zamčený lahve, může se mi třeba změnit cena
- Offline concurrency - chceme obsluhovat všechny co chtěj lahve, nelze mít dlouhé systémové transakce pro celou bussiness transakci
- Přepis necommitovaných dat, čtení necommitovaných dat víc zapíše na jedno místo, dvakrát přečtem jinou hodnotu z místa v jedné transakci

- máme optimistické - řešíme až když nastanou a pesimistické přístupy - vyhybáme se jim
- optimistické - předpokládá že šance na konflikt je málá
- ve chvíli kdy chci bussiness commitovat tak se podívám že ty věci na kterých má bussiness transakce stojí se nezměnili
- když se změnila tak udělám rollback a zruším transakci, pak vezmu zámek pro provedení system transakce to už bude rychlý a zapíšu změny
- bussiness transakce může být dlouhá, dlouhé zpracování dat či jich může být hodně

- pesimistické - data zamknu aby k nim mohla jen 1 bussiness transakce, musí locknout na co šáhne
- lock manager - jednoduchej, jeden pro všechny bussiness transakce, centralizovanej
- může dojít k deadlocku, locky mají timestamp kdy po nějaký době se releasnou
- nebo po chvilce neresponzivnosti se rollbacknou

- coarse-grained lock - když chci zamknout množinu objektů jako celek, potřebujeme sofistikovanější lock manager
- chceme zamykat toho co nejmíň co potřebujeme

- implicit lock - jeden lock navíc či míň toho může hodně zkazit, jednoduchá chyba developera, těžko testovatelné
- hodně systémů má zámky implicitně, lock manager s tim musí počítat správně zamykat a uvolňovat pro bussiness transakce

### Performance Tuning - optimalizace
- MapReduce vytváří bottleneck-free way of scaling out
- chceme snížit latency (dobu odezvy) - paralelizace, optimální algoritmy
- chcme zvýšit throughput - kolik se toho může najednou zpracovávat u paralelizace, hodně uzlů nejsme omezení
- předpokládáme linear scalability - s paralelizací lineárně program zrychlujeme - 2 uzly = 2x rychlejší

#### Amdahl's law
- jak moc můžu zrychlit systém paralelizací
- 1/((1-P) + P/N) - kde N je počet klikrát rychlejší je paralelizovaný řešení problému, než neparalelizovaný, s libovolným počtem uzlů -> jde k nekonečnu
- takže například když 10% nemůže být paralelizováno máme 1/(1-0.9) = 10x rychlejší program (prostě se jakoby vynechá paralelizovatelná část)

#### Little's law
- analýza front ve stabilním systému, L = kW, nezávisí to na ničem jiném
- průměrný počet zákazníků L je roven průměrnému počtu nových zákazníků k (v zákazník/h) * čas který v systému stráví
- to že je systém stabilní znamená že se nemění strávený čas s počtem zákazníků, pak by to nefungovalo
- pumpa 4 zákazníci/h jeden tam stráví 15 minut -> vždy tam bude v průměru jeden zákazník¨

#### Message Cost model
- spočte cenu přenesení zprávy C = a + bN
- C = kolik to stojí, a = cena za overhead poslání, b = cena za jeden byte zprávy, N = počet bytů zprávy
- 100 x (0.3 + 10/125) ms = 38ms
- 10 x (0.3 + 100/125) ms = 11ms
- vždy se vyplatí poslat co největší packet

### Teorie grafových databází
- uzly a hrany můžeme reprezentovat maticí sousednosti - snadné přidávání a odebírání hran
- zjišťování všech sousedů a přidávání mají O(n), je to velký O(n^2), ale grafy jsou většinou řídké
- list sousedů - vše dobré až na zjišťování sousednosti, můžeme ho vysortit pak máme insert i nalezení v O(log(n))
- dá se mít i matici incidence např. pro hypergrafu máme n x m polí říkáme kam hrany vedou
- laplacovské matice - lingebra
- máme sadu jednoduchých kroků, jít přes hranu nějak označenou či do nějak označených vrcholů - sestavením těhle malých kroků tvořím složitější dotazy

- můžeme to zefektivnit, že dáme nody co jsou u sebe blízko sebe - když se načte jedna stránka do cache - budou tam i sousedi a nemusíme načístat
- např. je seřadím podle toho jak je nacházím když dělám BFS
- lepší je strategie - bandwith (max vzdálenost mezi jedničkami na řádce) minimalizace - sesypat incidence na diagonálu, pak máme informaci poblíž sebe
- bandwith matice je maximum z bandwith řádků - je to víc cache friendly, je to NP-úplný problém

- rozdělení velkých grafů - optimalizace pro BFS
- matici sousednosti rozdělíme na čtverečky (2D) - do mřížky, místo na proužky (1D)
- lepší násobení matic neboť když mám např. 4 části tak ve sloupci už půlku mám a v řádku taky, místo se 4 nody mi teda stačí komnikovat se 3

- někdy se mi hodí directed hrany - např. XML, RDF, traffic networks, někdy ne třeba sociální sítě, chemické sloučeniny
- typy grafových databází - non-transactional - malý počet velkých grafů

- transactional - velký počet malých grafů - sloučeniny, lingvistika
- chceme často podgrafy v grafu nebo supergrafy - grafy které obsahují náš graf, podobné grafy
- graf si nějak oindexujeme a ten nám vyhodí grafy které určitě náš dotaz nesplňují, pak až testujem isomorfismus
- non mining-based indexing - indexujeme celé konstrukty grafů
- například si zaindexujeme všechny cesty rádky jsou cesty, sloupce jednotlivé grafy a záznam je kolikrát v grafu cesta je, rychlá ale může zabírat hodně místa
- GString - pro chemii indexujem cycle, star a line v tomto pořadí
- nody zcvrknou do jednoho nodu, jsme shopný strukturu popsat řetězcem, mají na to gramatiku

- v mining-based přístupu indexujeme jen určité featury grafů - to co je v aktuálních datech zajímavý
- např. častou nebo velmi málo častou strukturu udělám si inverted index, v jakých grafech se vyskytují, problím když mi přibyde něco co to mění
- není dobré indexovat časté cesty, to je moc časté, nejlepší je indexovat časté podstromy
