# Doporučovací systémy
Jsou systémy pomáhající uživatelům najít co chtějí obzvláště v místech informačního overloadu
### Vstupy doporučovacích systémů
- implicitní - uživatel si něco koupil, někam šel (nevyžaduje akci od uživatele)
- explicitní - uživatel dal hodnocení - lidé většinou moc nedávají
- kontextové - zrovna teď se dívá na vysavače (asi hledá vysavač), je po 22:00 (asi se nedívá se svými dětmi)
- komunitní - ostatní uživatelé kupují toto, ostatní podobní uživatelé měli rádi také toto

### Vlastnosti dat v doporučování
- Nejdůležitější jsou nejnovější data
- Je potřeba umět rychle upravovat model aby počítal i s novými daty

### Měření podobnosti vektorů
- Informace o datech ať už ratingy filmů od uživatele, relevantnost tagů pro uživatele nebo další často dostáváme ve formě vektorů
- Když měříme podobnost dvou vektorů je potřeba se zamyslet jak reprezentujeme chybějící data vzhledem k datům která máme 0 za chybějící a pak počet hvězdiček je divné neboť 1 hvězdička je pro film asi horší než že ho uživatel neviděl
- Podobnost můžeme měřit pouze na průniku kde máme hodnoty, ten ale musíme omezit nějakou minimální velikostí, jinak budeme dostávat hromadu 100% podobných uživatelů, kteří mají s naším jen jeden společný předmět a náhodou ten stejný
- **Hyperparameter** - Tato velikost minimálního průniku je jedním z hyperparametrů, ten můžeme vytunit vzhledem k úspěšnosti modelu, pokud si nejsme jisti jak ho nastavit

### Metody měření podobnosti vektorů
- Když už víme co chceme porovnávat máme několik možností jak to porovnávat:
- **Jaccardova podobnost** - dobrá pro implicitní feedback průnik děleno sjednocení
- **Eukleidovská, Manhatnovská vzdálenost** - vzdálenost vektorů
- **Cosine similarity** - úhel mezi vektory (vynásobení vektorů dělené jejich vynásobenými délkami)
- **Pearsonova korelace** - měří lineární závislost mezi dvěma vektory, má hodnoty od 0 do 1, 1 jsou stejné (nepř. oba rostou), 0 jsou nezávislé, -1 jsou opačné (např. jeden roste a druhý klesá)
- vypočte se jako suma odchylek hodnocení itemů prvního uživatele od průměrného hodnocení tohoto uživatele * to samé od druhého dělená produktem odmocnin druhých mocnin toho samého  

### Short head a Long tail
Je potřeba aby doporučovací systém správně pároval itemy z long-tailu, neboli více niche itemy se správnými uživateli - serendipity  
Pokuď by všem doporučoval jen to co mají rádi všichni odešli by z platformy více niche tvůrci

### Druhy doporučovacích systémů
- collaborative - na základě toho co se líbí ostatním uživatelům
- content-based - ukaž mi více věcí podobných těm co jsem již viděl
- knowledge-based - často použití knowledge grafů, ukaž mi něco co bych chtěl vidět na základě mých předchozích interakcí, dodají nám informaci že nižší cen aje lepší, uspořádání atributů vzhledem k preferenci, hyerarchický stav
- hybrid - dnes používané - složení všech informací co máme o uživatelích dohromady

# Neperzonalizované doporučovací systémy
- Nepoužívají data o uživateli, jen kontext (momentální item), všichni dostanou to samé když na něm jsou
- Například věci často nakupované spolu, editors selections na zprávách
- Support - Ze všech nákupů mlíko a chleba byli v 10%
- Confidence - Pokud si uživatel koupil chleba koupil také mlíko platí u 50% uživatelů co si koupili chleba
- Možný postup kdy vytvoříme pravidla A -> něco tak, že vezmeme všechny transakce a vezmeme ta A a něco která mají support alespoň 5% a pak vezmeme A -> něco kde je confidence alespoň 50%
- To že osekáváme support je trochu zvláštní neboť to že moc lidí nekupuje bazény není důvodem k tomu jim k bazénu rovnou nedoporučit filtraci
- Stereotypy - muziku co doporučíme určíme podle věku uživatele

# Collaborative filtering - CF
- Před používáním hybridních algoritmů to byl nejlepší z postupů
- Předpokládá, že pokuď si historicky byli uživatelé podobní (líbili se jim stejné věci budou si podobní i dále)
- Má výhodu univerzálnosti - funguje podobně na různých doménách - eshopy, filmy, knihy ...
- **Vstup** - matice uživatelských hodnocení itemů
- **Výstup** - predikce jak moc se bude uživateli líbit daný item, k nejlepších itemů

### Použití v praxi
- Je potřeba aby algoritmus zvládnul vydat výsledky v řádu sekund
- Je potřeba si něco předpočítat to jde lépe u itemů než u uživatelů
- Můžeme si pamatovat u každého uživatele 10 jemu nejpodobnějších
- **Problém** - ta nejdůležitější data jsou většinou ta nejnovější
- **Řešení** - Vypočtemem podobnosti itemů
- Ty jsou většinou s časem stejně podobný - Matrix 1 bude podobný Matrixu 2, ale když se mi líbil Matrix před rokem nemusí se mi nutně líbit stále
- Itemů také většinou bývá méně než uživatelů a jeijich podobnosti se nemění s časem, řešíme problém
- **Dobré je nedělat item rating prediction** (skoro vždy je nat o málo dat) a pouze uživateli ukázat k nejlepších predikcí
- Odpověď je potřeba podat v řádu sekund, je potřeba si něco předpočítat třeba 10 nejpodobnějších uživatelů
- To lze dělat pro itemy ale špatně pro uživatele, neboť tam nemáme nejnovější data která jsou nejdůležitější
- Velká a děravá dat ase dají ukládat ve **Sparse matici** která ukládá jen data co máme a pro zbylé předpokládá nějakou hodnotu, ušetří se místo
- Ty itemy kde nemáme data nebo je neohodnotili uživatelé podobní našemu uživateli můžemem vypustit a doporučovat jiné

### User-based CF
- Doporučujeme itemy který se líbili uživatelům podobným našemu uživateli
- Mějme 100 000 uživatelů a 10 000 itemů
- **Problém** - porovnávání trvá velmi dlouho
- **Problém** - matice je velmi řídká, lidé kteří viděli nějaký objekt nemají s naším uživatelem nic společného, je těžké najít někoho kdo je podobný, často nikoho tkaového nenajdeme

### User-based KNN
- Najdeme uživatele s alespoň nějakým počtem stejných hodnocení jako náš uživatel a spočtemem jejich podobnost k našemu uživateli
- Pokud chceme top-k itemů na doporučení, dopočteme jak by asi náš uživatel hodnotil itemy z hodnocení jemu podobných uživatelů a jeho podobnosti s nimi, nebo je můžeme pouze zprůměrovat pro k nejbližších sousedů (pak může být problém s outlieramam kterým jsou podobný jen 2 je dobré to škálovat podobností)
- Pokud chceme predikovat na kolik se bude uživateli líbit film chceme predikovat rating, vezmeme jen uživatele co ho hodnotili a vynásobíme jejich korelaci s jejich ratingem, zde se používá Pearsonova korelace která dává hodnoty od 1 do -1
- Můžeme také zvážit průměrný rating který uživatel nebo uživatelé dávají neboť každý uživatel má trochu jinou škálu hodnocení

### Z explicitního na implicitní feedback
Často není španté uvažovat implicitní feedback např. u hodnocení filmů jen poku d dal uživatel alespoň 4 hvězdičky tak se mu film líbil, přicházíme ale o nějaká dat)

### Item-based CF
- Doporučujeme podle podobnosti itemů, koukneme se na item kde potřebujeme rating a podíváme se jak náš uživatel ohodnotil itemy které jsou mu nejpodobnější, to zjistíme z uživatelů kteří oba itemy ohodnotili
- **Problém** item může být úplný outlier, který není ničemu příliš podobný
- používá se kosínová podobnost a pak vážený průměr hodnocneí pomocí váhy z podobnosti

### Item-based KNN
- První dobrý algoritmus který i dnes je stále dobrý, když si předpočítám podobnosti
- **Důležité** - je potřeba si uvědomit, že podobnosti itemů zde není podobnosti vlastností ale že jsou často nakupovány jedním uživatelem a proto by je i náš uživatel mohl chtít koupit oba (má podobně peněz, podobné nároky jako ostatní uživatelé kteří nakoupili ty itemy které on také)
- Použití na Amazonu 2003 - počítali s unární implicitní zpětnou vazbou jen zdali uživatel koupil item nebo ne
- offline předpočet - mám matici item x item a pokud uživatel koupil oba itemy přiču do daného políčka 1, pak tohle číslo vydělím počtem náku pu jednoho a druhého n- jaccard a získám jejich podobnost
- Většina itemů si není podobná, stačí si pamatovat třeba těch 1000 nejpodobněších
- online - Projedeme itemy který uživatel ohodnotil a spočteme ten který se jim všem nejvíc podobá na to nám stačí i 1000 nejpodobnějších pro každý item, prostě jen sčítám podobnosti s ostatními itemy, dá se to ještě vážit ítm jak ohodnotil uživatel oproti svému průměrnému hodnocení item ke kterému podobnosti počítáme
- lze použít i pro explicitní hodnocení, místo jaccarda použijeme cosinovou similaritu
- od non personalized doporučení se to liší že nebereme jen ty itemy podobný tomu na kterým je ale podobný všem na který se historicky koukal

### Memory-based a model-base
- User-based KNN je memory-based, problém je, že nestihneme vyřešit dotaz uživatele dost rychle, když si uživatele předpočteme přicházíme o nejnovější informace o uživateli
- Model-based, kdy něco předpočítáváme mají problém, že pracujeme se starými daty
- To ale například u podobnosti itemů nevadí
- Další možností je si uživatele nepamatovat jako id ale jen jako posloupnost itemů které si koupil nebo něco jiného (session based doporučovací systémy)
- Je důležité jak rychle lze model natrénovat, jestli při přidání dat umí nějaký UPDATE, nebo se musí celý přetrénovat

### Spreading activation algoritmus
- Skáčeme po grafu user item user item, knn item nebo user based jsou doporučování po cestě o délce tři když si to rozkreslíme
- Pomocí delších cest můžeme doskákat dále když máme méně dat
- Můžeme skákat tak dlouho až se síť vyváží nebo až dojedeme do stoku když hrany nejdříve orientujeme
- Část aktivace, udaná nějkaým hyperparametrem zůstane v nodu a zbytek se pošle dál do jeho sousedů
- Hodně podobné tomu jak google rankoval stránky
- Začne být zajímavé když zapojíme také content based vlastnosti podle kterých se tkaé můžu posouvat a třeba udávat kolik toho mám poslat

## Faktorizace matic
- Matici uživatelských hodnocení itemů můžeme reprezentovat pomocí dvou matic jejichž vynásobením dostaneme hodnoty co nejbližší hodnotám v dané matici
- Předpokládáme, že takto můžeme dostat dobré hodnoty i na místech pro která v matici data nemáme, můžeme predikovat ratingy které nám chybí
- Máme nějakou funcki minimalizace erroru, kterou zlepšujeme pomocí stochastic radient descent kdy poskakujeme po funkci po skocích velkých podle její derivace a snažíme se najít minimum
- To jak dobře to zvládneme může hodně záviset na na počtu parametrů které pro výpočet použijeme, je potřeba jich nemít málo, ale zase ne moc abychom předešli overfittingu kdy nedonutíme algoritmus informaci o datech v původní matici komprimovat a hodnoty mimo ty známe nebudou pak relevantní
- Problémem může být že minimalizace vzdálenosti predikovaných může být dobrá pro predikci ratingu, ale už nemusí být dobrá pro pořadí itemů, neboli pro zjištění headu, který chcemem uživateli nabídnout
- Nevíme co jednotlivé faktory znamenají al eměli by vystihnout jak jim itemy nebo uživatelé odpovídají

### Jak faktorizaci udělat
- **Regularizace** - chceme aby parametrů nebylo moc a aby jejich hodnoty nebyli příliš vysoké, toto oboje dělá regularizační část u celkové chybové sumy, která po vynásobení s hyperparametrem lambda bere součet druhých mocnin faktorů pro uživatele a pro itemy a přidává ji ke sumě druhých mocnin rozdílu výsledku a správné hodnoty v matici
- použijeme **Stochastic radient descent** - rozložíme sumu na jednotlivé body z dat, který závisí jen na jednom řádku a na jednom sloupci
- **Algoritmus**: Hodnoty matic U a V nahodíme náhodně
- Projdu všechna trénovací data klidně několikrát do té doby než se loss funkce už moc nezlepšuje - asi jsem našel ideální hodnotu (nebo jsem udělal moc iterací), a v každé iteraci:
- Pro každé políčko matice (každé 1 hodnocení):
- Vezmu pro každý faktor, který používáme:
- Ostaní faktory vezmu jako konstanty, tento jako proměnnou a spočtu pro něj derivaci, tím pádem není derivace moc složitá
- Následně od daného faktoru pro usera odečtu learning rate krát error krát hodnotu faktoru pro item (čim je větší tim odčítáme víc je zvláštní ale vychází to z derivace) a také od faktoru odečtemem lambda * původní faktor (snažíme se minimalizovat hodnoty - regularizační část)
- Pro item to uděláme stejně akorát násobíme faktorem pro uživatele

### V čase dotazu
- Zjistim jak by uživatel ohodnotil další itemy a vyberu z nich ten s nejvyšším hodnocením
- **Problém** - Jak přidávat nová data, celý model nezvládneme přetrénovat
- **Částečné řešení** - Když máme nová data od uživatele můžeme updatovat jen daný sloupec a řádek user a item matic aby i tato nová hodnota vycházela dobře, to zvládneme v čase dotazu
- Budemem potřebovat nějaký sheduler aby jen jendo vlákno matici modifikovalo
- Problém je nový uživatel (nebo nový item) který se objevil až po přetrénování modelu, můžemem přidat nový sloupec a náhodně ho nainicializovat ale asi nám nestihne zkonvergovat tam kam byjsme potřebovali
- To se dá řešit pomocí hybridních algoritmů kde zde doporučujeme nějakýmjiným algoritmem
- Faktorizace matic se tolika nehodí pro nestatické prostředí, má to ale moderní řešení
- **Session based algoritmy** - uživatele reprezentujeme jen jako posloupnost interakcí a pro tu posloupnost interakcí kterou náš nynější uživatel provedl již nejspíše máme hodnoty v user a item matici spočítané
- **Problém** - optimalizujemem vlastně RMSE která dělá odmocninu ze skutečného a predikovaného ratingu uživatele
- jenže to že je nízká nemusí objekty správně seřadit, čim víc RMSE zlepšíme se můžemem odchýlit od správného pořadí
- **Problém** - Některý věci se učíme složitě, rating itemu je složen z globálního rating průměru, biasu uživatele, biasu itemu a skalárního součinu faktorů (preferencí)
- Od opravdové hodnoty odčítáme i globální průměr a oba biasy a také je máme v regularizační části (ty biasy) aby byly co nejmenší
- této vylepšené verzi se říká FUNKSVD++ a celkem se používá

### BPR Matrix facorization
- Ranking oriented matrix facorizace
- maximalizujeme logaritmus funkce rozdílu správné a špatného itemu pro daného uživatele, odčítáme regularizační termy
- máme trainset trojic (user, good, bad) chceme aby rating dobrého itemu byl větší než špatného
- Problém je v tom jak vybírat dobrý a špatný itemy pro uživatele
- Pro všechny trojice máme hodně negativních pro málo pozitivních když brali žádnou interakci s itemem jako negativní a nějakou jako positivní, tak se přetrénovávali dobrý itemy se spoustou špatných vybírali tedy vždy pro trénování stejný počet negativních jako pozitivních
- nebo se dá negativních parametrů vzít k
- když máme nebinární zpětnou vazbu můžeme dost různě zvolit co je pozitivní a co je negativní
- hodnocneí 4 vs 3 nebo hodnocení 4 vs 0, těžko říci podle čeho trénovat aby byl model dobrý
- nepotřebujeme vědět jaké je setřídění tailu stačí nám setřídění headu na tailu nás to moc nezajímá

### Nevýhody a poznatky k faktorizaci matic
- Trénování probíhá jednou za nějakou dobu problém s novými uživateli
- Je tam hodně hyperparametrů které hodně ovlivňují výsledek, což je celkem nepříjemné neboť musíme dělat náhodné kombinace parametrů, není moc dobrý grid search neb se nemusim trefit kam chci
- některý hyperparametr může mít větší vliv
- Lokální optimum versus globální, existují již lepší postupy než stochastic radient descent 

# Content-based algoritmy
- Když máme málo dat tak jsou metody kolaborativního filtrování málo efektivní
- Máme zde problém cold startu, ten řeší content-based algoritmy
- Snažíme se doporučovat na základě tagů, embedingu obrázku např. kabelky, obsahu
- Jednoduchá možnost - podobnost podle Jaccardovi similarity určitých tagů, je podobná ItemKNN kde akorát jinak získáváme podobnost itemů, ne z hodnocneí ale obsahu
- dobré pro začínající a malé platformy

### Zpracování textu na tagy
- **TF-IDF Term-Frequency - Inverse Document Frequency**
- ne všechna slova mají stejný význam jak definují item
- slova která jsou v každém dokumentu, dokumenty neodliší
- TF - frekvence klíčového slova v documentu
- IDF - logaritmus z počtu dokumentů které můžemem doporučit děleno počet dokumentů kde se slovo vyskytuje
- TF-IDF = TF * IDF - čím je častější a čím je vzácnější, tím více dokument definuje
- Následně použijeme cosinovou podobnost
- u novinových článků je důležité ještě jak je článek nový

### Metody zpracování textu
- Můžem vyhodit spojky, dát slovesa do infinitiv, nechat jen 100 nejčastějších slov ...
- Pořád ale nezaznamenáváme význam slov např. nejsou tam žádní ...
- Dneska se pužívá spíš word2vec nebo text2vec který nám dá embeding textu do nějakého vektoru

### Jiné vstupy než text
- časté jsou obrázky, zvuk, video - taky uděláme nějaký embeding kdy podobné itemy jsou si vektorově podobné
- a také atributy - klíč hodnota - nominální atributy Výrobce:"Acer", numerické atributy RAM:32GB
- nominální atributy dáme do vektorů snadno uděláme jenu složku pro každého výrobce a ten který to je dostane 1
- Když do vektoru hodíme numerickou hodnotu tak riskujeme že jedna je proporčně větší a pak se doporučuje pouze podle ní
- Můžeme hdonoty normalizovat, nebo rozdělit do kategorií
- používá se **Fuzzy membership** - kde se dá nějaká hodnota pro 8GB RAM i do 2-4 a do 16-32 abychom započetli že to není úplně daleko
- Je možné přidat retention vrstvu která s eučí důležitost jednotlivých atributů pro uživatele
- Nemůžememe zde používat TF-IDF neboť například muž a žena je častý ale důležitý parametr na seznamce

### Rochiova metoda
- Pracuje i s negativním feedbackem
- Pro itemy co se líbily vytvořím těžiště a pro ty co se nelíbily taky
- u itemů zjistíme jak blízko jsou k těmto centroidům a podle toho doporučujeme
- problém je když jsou preference uživatele více rozházené má rád romcomy a horrory
- Můžeme použít Bayesan relevance feedback dál nabízíme ty itemy které jsou blízko k tomu který byl vybrán a tím se přesouváme po mapě až do ideálního bodu
- trošku problém když je dimenzí hodně pak jsou všechny vektroy daleko od sebe

### Rule based systémy
- Lze použít pro preproccesing kde vyhodíme nějaká data jako například Patrik v EasyStudy

### Problémy Content based doporučovacích systémů
- Doporučujeme to co je podobné tomu co se mu líbilo, problém s overspecialization, například pro Harryho Poterra mu nabídneme 6 dalších Harryho Potterů
- Potřebujeme přidat vzájemnou různorodost výsledků
- jednoduchou metodou je maximal marginal relevace - vezmeme doporučení původního systému první objekt doporučíme
- následně relevanci dalších objektů spočteme jako alpha*relevance - (1-alpha)*max(podobnost k již vybraným itemům)
- je pomalé pro dlouhé seznamy, můžeme takto přeskládat jen head třeba 100 itemů co jsem dostali
- alpha a způsob měření podobnosti jsou hyperparametry

# Knowledge based modely
- Máme nějaké vztahy mezi produktovými vlastnostmi, hyerarchie, co je lepší a co horší
- Uživatelé často udávají nějaké podmínky např. auto má být černé a mít tohle a tamto
- Podobné konverzačnímu procesu kdy mu něco dáme on nám dá nějaký feedback a tak mu to upravíme, jako kdyby se bavil například se SIRI
- různé constraints mohou mít pro uživatele růlznou důležitost, to nám může takké sdělit
- preference na objektech můžeme podle jejich hodnocení modelovat lineárníma funkcema a naučit se regresí důležitost jednotlivých atributů, potřebujeme na to ale hodně dat

### Constraint based
- Máme nějaká explicitně definovaná pravidla např. cena mezi x a y, nebo i něco high level jako přijatelná kvalita za rozumnou cenu a my jsme to rozparsovali

### Case based
- Uživateli jsme něco dali a on nám řekl je to dobrý ale tenhle atribut je špatně (chtěl bych lepší baterku) a my to zkusíme upravit
- Snažíme se doporučit podobné ale lepší uživatel může dát nějaký threshold vůči ukázanému itemu

### Poznámky a nevýhody
- Snadno se s constraintamam dostaneme někam kde nemáme co doporučit
- Dobré je si odnést, že lepší než doporučovat blízké objekty je doporučovat lepší objekty
- Dneska se rozvíjejí konverzační doporučovací systémy - SIRI pusť mi nějaký jazzík, zkus nějaký rychlejší než tenhle

# Evaluace doporučovacích systémů
- Je potřeba umět poznat co je nejlepší doporučovací systém pro danou doménu
- Je potřeba definovat nějaké metriky jak měřit zdali doporučovací systém pracuje dobře
- napříkald na Netflixu to je kolik lidí co si zaplo členství zdarma se neodhlásí a začne platit
- tyto high level metriky se často těžko měří, napříkald musíme ten měsíc počkat než se dozvíme výsledky

### Online evaluace
- Vyhodnocujeme přímo jak se doporučovací systém projeví u uživatelů
- Potřebujeme i něco jiného, zde se dozvíme výsledky většinou za delší dobu a také se nám může stát, že nasadíme něco co vůbec nefunguje a to rozhodně nechceme ani pro část uživatelů
- jsou peněžně a časově náročné a těžko opakovatelné
- proto máme offline evaluaci a user studies
- bliží se našim klíčovým metrikám, dají se měřit i změny GUI

### Offline evaluace
- Data co mám si rozdělím na trénovací data a testovací data
- Můžeme zde měřit různé metriky
- **Problém** tyto metriky jsou hodně biased tím jaký systém jsme používali když jsme sbírali data
- vlastně tedy netestujeme kvalitu pro uživatele, ale podobnost předchozímu doporučovacímu systému, dá se to debiasovat
- Potřebujeme aby jsme mohli testovat hyperparametry a testovat algoritmy
- Provede nám úvodní filtrování algoritmů, abychom do onlinu nasadili jen ty co fungují a mají šanci být nejlepší

### User studies
- rekrutujeme nějaké uživatele a ti pracujou s naším webem nebo s nějakým mockupem
- sledujeme co dělali a můžeme se jich ptát jak to funguje a jestli byli spokojeni, co bylo špatně ...
- ne vždy se budou chovat stejně jako naši uživatelé, lidé kteří jsou ochotní dělat user studies nejsou stejní jako lidé co používají náš systém
- výsledky se nedají snadno zopakovat a je to drahé ale jsme schopni se přiblížit k našim klíčovým metrikám, můžemem měřit i změny GUI

### Jak dělat online evaluaci
- Lidi rozdělim do skupin a každé dám pro ně doporučovací metodu
- Je potřeba skupin nedělat moc, aby byly výsledky relevantní - čim méně metrika nastává, například nákup tím více dat potřebujeme abychom si mohli být jisti, že je tento algoritmus lepší (obzvláště když je často lepší o pár procent)
- Metriky měření úspěchu tedy často volíme jako nějaké s co nejpřímější navázaností na ty cílové, například kliknutí na item u kterých věříme že jejich zlepšení povede k zlepšní cílových metrik, ale přitom např. nastávají častěji a dají se tedy lépe změřit
- od nejlepšího:
- 1) Cross-sale increase uživatel něco nakoupil a pak nakoupil ještě to co jsme mu k tomu doporučili, uživatel se častěji vrací nad platformu, premium subscriptions
- 2) Conversions rate uživatel koupil item co jsem mu nabídl, comment na itemu, like
- 3) Click-through rate, přijde a hned neodejde
- 4) Uživatel si někdy item co jsme mu nabídli zobrazil, možná ne hned ale odhadli jsme ho správně

- **Důležité** jsou také technické metriky - hlavně response time, train, re-train, model update time
- Memory, GPU, CPU consumption
- Recall on objects .není nějaká část objektů ignorovaná například ty z long-tailu

### Offline evaluace
- Data set si rozdělíme na trénovací, validační a testovací část
- cross validation kdy posouváme validační část po té trénovací má ten problém, že můžeme trénovat na novějších datech než kde validujeme a víme tedy dopředu např. že nějaký objekt bude virální, toto nechceme
- rozdělení provedeme vzhledem k času nejstarší data pro trénování, novější pro validaci a nejnovější pro testování, problém je že pak skoro žádní uživatelé nejsou ve všech třech těchto částech a častou jsou jen v 1
- **Řešení** - data si rozdělíme per user podle času tak je dostaneme všude, je to kompromis
- Pokud to nemusím dělit per user dá se dělat sliding test set kdy postupně zvětšuji trénovací část a s ní posouvám tu testovací abych otestoval na skoro celých datech, zachovávám stále časovou souslednost
- Na validační sadě vyberu nejlepší sadu hyperparametrů
- S těmi to pak natrénuji na trénovací i validační sadě a predikuji na testovací sadě
- Nechceme trénovat hyperparametry na testovací sadě neboť tak by jsme je pro ní vlastně leaknuli a výhodu by měly metody s větším množstvím hyperparametrů
- jsou opakovatelné (na stejných datech), rychlé, opakovatelné (když se něco nepovede)

### Metriky offline evaluace
- Precision top-k - kolikrát se item z testovací sady objevil v doporučení v top-k
- nDCG, MAP- na jaký pozici se objevili relevantní itemy
- když dostáváme přímo hodnocení MeanAbsoluteError, ... už se moc nepoužívají
- **diversity** - **intra-list diversity** - Průměrná podobnost všech dvojic v našem doporučení, můžeme to brát přes content-based a nebo přes podobnost kolaborativní, spočteme jako pro všechny dvojice suma(1 - podobnost dvojice) * počet dvojic ((každá je tam 2x))
- **novelty** - nově vznikly nebo o ně uživatel zatím nejevil zájem a asi je tedy nezná
- měřit můžeme jako s čím uživatel zatím neinteragoval, co uživatel nezná (spočteme novelty z celkové popularity itemu, čim menší tím lepší) např. -log2(lidi co item ohodnotili / pocet lidi), jestli vznikla nedávno
- **coverage** - jakou část našeho celkového katalogu ukazujeme, je potřeba abychom něco neignorovali
- **popularity bias/ popularity lift** - hodně doporučovacích systému příliš protěžuje populární itemy můžeme měřit popularity lift jako (prumer popularity nabizenych veci - prumer popularity z dat) / prumer popularity z dat
- **precision at top-k** - z prvních k nabídnutých itemů kolik bylo dobrých
- má varianty kde se počítá s konkrétní pozicí neboť pozice má zásadní význam, závisí to hodně na interfacu
- **recall at top-k** - kolik z celkového počtu dobrých předmětů jsme ukázali, když jej jich hodně tak hodnoty budou malé nemůžeme jich ukázat více než na kolik máme místo

### Metriky na výpočet precision vzhledem k pozici
- Discounted cumulative gain DCG = relevance prvního + suma přes (relevance dalších / log2 (jejich pozice)), umí pracovat i s relevancí v podobě hodnocení
- normalizovaná mezi 0 a 1 vydělim DCG toh co jsme doporučil IDCG (neboli DCG pro ideální seřazení itemů pro uživatele)
- rankscore 2 na (rank(i)-1 deleny alfa ranking half life) - má drastičtější efekt
- Liftindex - rozdělíme si to do oblastí a v každé to násobíme nějakým lineárním koeficientem (zde jsou stejně velké) ale obecně by nemusely být můžeme nastavit podle GUI
- **MAP** - procházíme postupně relevatní objekty v našem doporučení a vždy přičteme kolikatej objekt to je / pozice a pak to cele vydelime poctem objektu - má trochu drastičtější pokles než NDCG
- **Důležité** - je potřeba vzít nebo vymysle t takové metriky které odpovídají tomu jak lidé náš systém používají a podle čeho měříme úspěch, například s emůžem kouknout v jakým poměru lidé klikají na pozice
- Některé metriky jsou nutně negativně korelované např. (diversity a novelty) a precision jdou proti sobě

### Multiarmed bandits
- Chceme si být dost jistí, že jsme vyzkoušeli všechny algoritmy ale přitom chceme minimalizovat regret který ztrácíme používaním horšího algoritmu
- Použití v A/B testování
- metody výběru epsilon greedy vyzkousim to co mi dává nejvetsi zisk a s sanci 1-epsilon vyberu jiny nahodny
- metoda upper confidence bound kdy vezmu pro každé rameno horní ohraniční intervalu spolehlivosti
- metoda Thompsonova samplování - držíme si úspěchy a neúspěchy a používáme náhodnou hodnotu z beta funkce, jejíž rozdělení nějak odpovídá úspěšnosti - podle výzkumu ideální metoda
- multi armed bandits mohou pracovat i s kontextem například podle parametrů pro koho doporučují nějak upraví vliv jednotlivých algoritmů na výsledek

### Debiasing offline evaluation
- Lidé často koukaj na itemy nahoře vlevo a pak po okrajích
- Výsledek na kterej předtim skoro nikd neklik se kliká hodně když na googlu otočili pořadí v kterým je ukazujou
- Uživatelé a jejich chování jsou ovlivněny tím co je jim prezentováno
- Pokud jsem uživateli item nedoporučil a je těžko dohledatelný tak ho nejspíš nemá ani v testovacích datech i když by s emu například líbil, další algoritmus mu ho tedy zase nejspíš nedoporučí, neboť se natrénuje na tom, že ho vidět nechtěl
- To že uživatel neviděl populární item je více důležité než že neviděl průměrný item neboť ten populární dost možná ignoroval záměrně
- Když byl nejdřív systém co doporučoval populární itemy a na těchto datech trénujem nějaký co se snaží doporučovat long-tail tak bude mít špatné metriky neboť long tail v test datech bute chybět
- vlastně tedy nehledáme systém který by byl dobrý pro uživatele, ale který je podobný předchozímu
- Dá se to debiasovat když vím co jsem uživateli doporučil ne jen na co kliknul a pak další algoritmus testuji jen na tom co jsem uživateli v minulosti ukázal třeba áženym tim jaká je podle GUI šance že na to klikne
- Když tato data nemáme dá se pracovat s celkovou popularitou neboť se předpokládá že populární item uživatel někdy viděl s větší šancí a vypočtem tedy každému itemu **propensity score** - proporční k počtu dat které máme o itemu s nějakým nastavitelným koeficientem gamma, ten můžeme nastavit na základě předchozího algoritmu
- Potom prostě násobíme každou metriku tímto skóre a méně poulární item tím získávají výhodu
- **Důležité** - Nad biasy je potřeba přemýšlet neb mohou testovací data hodně ovlivňovat

## Hybridní doporučovací systémy
- Jeden algoritmus může pokrýt slabinu druhého např. cold start
- Máme víc možností jak použít víc doporučovacích systémů
1) odděleně na různých místech v aplikaci
2) paralelní - agregujeme vstupy (weighted average, Thompson Sampling, Fuzzy D'Hont) z více algoritmů
3) Monolitic hybrid - v dnešní době téměř všechny v researchi, často nějaké neuronové sítě které používají všechna data co mají (na to jsou dobré), collaborative i content based, knowledge based, proto jsou hybridní ale je to jeden model, je to black box když nevidíme co se děje uvnitř
4) Můžeme je používat postupně v pipelině jeden to předělá pro další

- Jendoduchý monolitic hybrid - mám v matici hodnocení málo hodnocení, tak doplním content-based podobné itemy od uživatele podobnými hodnoceními a těmto místům například dám nějakou menší váhu takže kolaborativní podobnost ovlivňují o něco míň

### Paralelní
- Většinou použijeme nějaký content-based a nějaký kolaborativní
- Vážené braní algoritmů podle jejich úspěšnosti - je dobré aby nebyla statická neboť úspěšnost se může měnit například s učením
- Váhy pro algoritmy mohou být hyperparametry, to se ale blbě škáluje když máme algoritmů více
- Výsledek se často bere vážený průměr relevance pro item z obou metod
- Je potřeba dostat hodnoty z algoritmů na podobnou úroveň aby měli stejnou sémantickou relevanci
- problém je pokud uživatelé mají nevyrovnaná hodnocení
- Průměr má problém, že když budu mít dva algoritmy které doporučují podobně a jeden který hodně jinak v konečném seřazení vždy vyhrají itemy doporučované těmi dvěma pokud mají stejné váhy
- Dává tedy smysl spíš udělat switching hybrid kdy buď jeden algoritmus udělá všechna doporučení, to je ale zase špatné když je nějaký špatný tak že celé doporučení bude špatné, a další doporučení udělá jiný, nebo switchujeme po itemech
- Na switchování po itemech zase použijeme multiarmed bandits

#### Thompsonovo samplování
- Používame beta distribuci kde střední hodnota odpovídá pravděpodobnosti na výhru a s více pokusu se zužuje interval kde dostáváme hodnoty
- **Problém** Potenciálně špatné je že bere doporučovače nezávisle, takže když se nějaké recomendry shodli na itemu ale třeba ho žádný nemá na prvních místech tak se nevybere i když by jeho vybrání dávalo smysl

#### Fuzzy D'Hont
- Používá se pro volby vždy vezmu počet hlasů strany a připřidělení křesla vydělím počet hlasů počtem zatím přidělených křesel + 1 
- Můžeme to použít na algoritmy, že vezmeme jejich momentální váhy a přidělujeme od nejlepší dodporučovací pozice tímto způsobem
- původní algoritmus nemá schopnost sdílet kandidáty, tohle ale můžeme přidat
- přidáme fuzzy memebership každý recommender podporuje nějakého kandidáta hodnotou od 0 do 1
- když kandidáta vyberu vydělím hlasy recommenderů hodnotou jak moc už jsme je vybírali která začíná na 0 a vždy do ní iterativně přidám relevanci vybraného kandidáta, když ho vyberu
- váhu pro recommendery můžeme měnit podle toho s jakou jistotou kandidát, kterého vybral uživatel navrhli

### Pipeline
- Dělí se na kaskádové a meta
- Káskadové mají výhodu, že někdy zjistim že nestíhám doběhnout tak vezmu momentální stav pipeliny, používá se hodně např. na Seznamu, další recommender dostává nějaký head z přechozího
- meta inicializace u faktorizace matic na ne náhodné hodnoty, celkem to opadlo když se začali používat rozumnější optimalizační metody než stochastic radient discent, např., ADAM, momentum based které rychleji konvergují
- trochu se to dá chápat jako použití u deep learningu kdy vytvoření embedingů z itemů je vlastně takový první krok pipeliny

### Monolitické hybridy
- Pro spreading activation algoritmus kdy do grafu dáme nejen vazby hodnocení ale i content based vazby, žánr a třeba vazby mezi uživatelama podle věku pohlaví ...
- Budeme mít mnohem víc kolaborativních hran než kontent based, dáme teda hranám váhy abychom to vyvážili můžeme pro různé hrany zkusit různé váhy jako hyperparametry, ale pro menší změny to vychází podobně
- vracíme nakonci ty nody co mají největší aktivaci

- u faktorizace matic můžeme itemy definovat jako souhrny jejich atributů
- mám matici pro jednotlivé atributy a item definuju jako nějaký souhrn z této matice
- loss funkci ve faktorizaci matic můžemem upravit podle conten-based podobností itemů

## Použití umělé inteligence a deep learning
- Kolem roku 2017 dosáhl na mnoha tascích deep learning úrovně člověka, například klasifikace obrázků
- Deep learning natrénuje co by měly být ty správné popisné vlastnosti napříkald pro obrázek
- Do deep learning modelů se dobře zahrnují další informace nebo bussiness pravidla
- Dobře se škálují s množstvím dat, mají horší starty ale s více daty se stále umí zlepšovat, u některých jiných algoritmů mohou narazit na svůj vrchol kdy jim více dat již nepomůže
- Přístupy s použitím umělé inteligence nemusí být nutně lepší než ty bez ní, je dobrá když je dat velmi mnoho
- Novinky ve světě AI se většinou promítnou do doporučovacích systémů za 1 až 2 roky
- Některé problémy ze světa doporučovacích systémů jdou převádět na ty z AI například doplnění kam slovo nejlépe zapadá do věty z které bylo odebráno, pokud pouze změníme slova na itemy, můžeme hledat mezi jaké itemy nejlépe zapadne ten který máme
- Dalším místem použití AI v doporučovacích systémech je klasifikace obrázků, kdy se z obrázku snažíme získat nějaká data abychom ho uměli lépe zařadit a následně lépe doporučovat
- Neurals jsou dobré když dáváme dohromady více různých typů dat
- Playlisty a nebo session výběr co nabídnout dalšího - session based recommendation odpovídá posloupnosti slov co dalšího by sem pasovalo

### Jak fungují neuronové sítě
- V neuronu sesumujeme aktivaci která přišla z předchozího neuronu a podle sigmoidí funkce ho s nějakou pravděpodobností aktivujeme je mezi 0 a 1
- Jednoduchá verze přijde nám nějaký vstup pak máme na sebe navázané layery kdy to vždy vede o jedna do dalšího layeru a nakonec poslední layer může být nějaká klasifikace podletoho jaký neuron se aktivoval
- To co se učíme jsou váhy který odpovídají jednotlivým hranám
- Můžemem si váhy označit vektorama, výstup porovnat s očekávanym výstupem a optimalizovat pomocí třeba stochastic radient descent loss funkci očekávanýho a výstupu co přišel 
- Musíme počítat s tím že váhy dál jsou ovlivněny těmi předchozími to zahrneme do výpočtu derivace
- Problém sigmoidí funkce byl, že na krajích je derivace malá a tak se pomalu učí a pro hlubší vrstvy je to násobené těmi předchozími takže se nikdy nedoučíme
- Změníme funkci na ReLu která je lineární a v záporu pomalu klesá
- Proti overtrainingu se bráníme takzvaným dropoutem, kdy vyhodíme náhodně některé nody, braníme se tím také redundanci přenosu informace
- Změny neděláme po 1 váze ale po 50 - 100 abychom moc nevystřelovali ale zase nejeli moc pomalu jako pro celé
- Místo stochastic radient descent použijeme moderní optimalizační algoritmy, jak se blížíme zmenšujeme postupně kroky učení
- standartem je ADAM

### Použití neuronových sítí v doporučovacích systémech
- Velmi dobré jsou jako information extractory z obrázků, textu, audia, videa
- Dobře pracují s různorodými daty což se nám hodí
- Mohou dobře reprezentovat sekvenční chování což je to co se nám hodí například na eshopu kde uživatel udělal několik akcí
- Často ale jednoduší modely nemusí být horší napříkald deep collaborative filtering který potom debunkli že neuronky jsou tam k ničemu
- V seesion based recomendations se používají retenční sítě spíš než neuronky ale ty fungují podobně

### Item to vec a word to vec
- Vytvoříme nějaký embeding daného itemu jako ve faktorizaci matic, následně můžeme testovat podobnost itemů
- Pro dynamický data není dobré mít vektor reprezentující IDčka - one node encoded, jako vstup pro komplecnější metodu použijeme nějaký embeding
- ve faktorizaci matic vlastně vezmeme one node encoding vynásobíme ho váhami uživatele a pak váhami itemů a máme rating prediction, tohle se můžeme učit
- **Word to vec** pro slovo děláme embeding pomocí tří slov na každou stranu okolo
- jako vstup pro neuronku máme k-node encoding kde máme 1 na slovech která tvoří kontext
- my to ténujeme s cílem dostat slovo uprostřed a kkdyž to vyténujeme vezmeme mezilayer embedingu původního vektoru
- **Použijeme** negative sampling negativních hodnot je strašně moc a bylo by to pomalé mi vybereme z těch negativních náhodně jen nějaké ale po delší době projdeme všechny casy, něco podobného jsme použili u BPR metody u faktorizace matic kdy jsme na jeden dobrý item brali jeden špatný item
- U embedingů může fungovat nějaká aritmetika king - man + woman = queen
- Paragraph to context, místo slov vezmeme paragrafy
- U doporučovacích systémů to uděláme stejně nahradíme slova za id a nemáme context kolem ale jen předtím, můžeme načíst další item co bude následovat, dostanu jeho embeding a z itemů co mám doporučím ty nejbližší
- Paragraph můžu nahradit za uživatele dostanu reprezentaci kontextu
- Můžeme se snažit predikovat i metadata itemu co chceme predikovat

### Item embeding extraction
- Ke každému itemu v BPR můžemem přidat fixní faktory které odpovídají jeho embedingu například z obrázku a optimalizujeme zbylé parametry
- embedingy můžemem získat z konvoluční neuronové sítě
- jako descriptory objektu bereme většinou přeposlední vrstvu, v poslední se jen dozvíme jeslti to je auto nebo květináč, která by měla dobře na high-level úrovni representovat daný objekt
- vezmeme menší matice které hledají hrany to jak je hledají také zlepšujeme s celou neuronovou sítí
- Z těchto hran poté můžeme hledat větší objekty
- Můžeme používat předtrénované sítě co od někoho dostaneme na jiných tascích zahodim poslední vrstvy a použiji na můj vstup
- Audio můžemem reprezentovat vizuálně a na to to zas použít
- Rekurenční neuronový sítě na session based mají výhodu že nejsou závislé na velikosti vstupních dat, použití např. chceme informaci z věty která může být různě dlouhá
- použití při poslouchání hudby, rozšíříme playlist, doporučujeme na základě sekvence
- next item addition je častá věc co potřebujem a přesně RNN na to použijem - recurrent neuron networks
- když je session moc krátká můžeme před ní přidat něco z minulé sessiony nebo další


## Context dependent recommendations
- V červenci asi nechceme doporučovat Vánoční písně ani zimní kabát
- s rodičemu a bez rodičů dost možná sledujeme něco jiného
- muzika se odvijí podle nálady
- čas dne, víkend, nevíkend jsou zajímavé ale jinak se kontextové informace získávají špatně

- v ecommerence u žehliček nedoporučuju pračky, contextem je co se hledá v této session
- u matrix faktorizace může být třetí dimenze čas
- Problém je že když bereme pouze část dat, tak si zhoršujeme výkonnost systému
- Dají se snadno přidat do neuronových sítí, přidáme pro ně další embeding kontextu

### Explanation volby
- U black box algoritmů se dá dělat tak, že se kouknu jestli by s edoporučovaný item stejně doporučoval, kdybych mu změnil nějaké vlastnosti, když už ne tak jsou dost možná důležité a můžu je uživateli vypíchnout
- Je potřeba prodat uživateli, že je to dobrý doporučení


```python

```
