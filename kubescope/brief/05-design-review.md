# KubeScope — a design teljes átnézése

> 2026-09-02. Bemenet: `design-workspace/kubescope/design/` — 31 panelképernyő, a fogantyú-kör hat
> lapja, az akciótérkép, a komponens-leltár. Kérdés: **mennyi a redundancia, és hogy áll össze ebből
> egy véglegesíthető app.**

---

## A rövid válasz

**A redundancia valós, de nem ott van, ahol félni lehetett tőle.** A widgetek nincsenek
megduplázva — a tervező 31 képernyőt **27 komponensre és hat panel-testre** vezetett vissza, és ezt
le is írta. Amiből viszont tényleg túl sok van, az a **panelfajta**: több képernyő ugyanazt a három
eszközt rendezi újra más adatra, és külön paneltípusként lett megrajzolva.

A számok, az akciótérképből (`01-Action-Map.dc.html`, 316 művelet):

| | |
|---|---|
| Külön paneltípus, amit valamelyik művelet megnyit | **19** |
| Ebből: `resource list` + `details` | **88 megnyitás a 174-ből — a fele** |
| Négy vagy kevesebb művelet nyitja | **11 típus** |
| Megrajzolt képernyő | **31** (15 kérdés + 9 tárgy + 7 keret-funkció) |
| Panel-test (`PaneBody` switch) | **6** |
| Keret-komponens | **27 + 4 store** |

Tehát: **hat test hordoz tizenkilenc típust**, és a tizenkilencből tizenegyet alig nyit meg valami.
Ez a tizenegy a munka tárgya.

---

## A mérés, amiből a többi következik

A `PaneBody.svelte` egyetlen `switch`-et tartalmaz, és hat ágat:

```
DataTable · LogView · YamlView · TerminalView · MetricView · DetailView
```

Ez a helyes szám, és nem is akarjuk csökkenteni. **A kérdés nem az, hány testet rajzolunk, hanem
hány „típust" tanítunk meg a felhasználónak** — mert minden típus egy sor a menüben, egy glif, egy
billentyű, és egy dolog, amit meg kell jegyezni.

Az akciótérkép eloszlása pedig extrém:

```
resource list  ████████████████████████████████████████████████  48
details        ████████████████████████████████████████          40
form editor    ████████████████████                              20
yaml           ██████████                                        10
log            ████████                                           8
diff           ███████                                            7
events         ██████                                             6
diagnostics    █████                                              5
rbac           ████                                               4
describe · metrics · terminal · certificates · helm   ███         3
what-would-happen                                     ██          2
quota · waste · global search · openshift update      █           1
```

**A hosszú farok a redundancia.** Nem azért, mert rossz képernyők — hanem mert mindegyik egy
`resource list`, amire ráültettek egy számított oszlopot.

---

## Öt összevonás

### 1. A „verdikt-oszlopos lista" — öt képernyő, egy panel

Ez a legnagyobb tétel. Öt képernyő zárónotesze **ugyanazt a mondatot** mondja el:

| képernyő | a tervező saját összefoglalója |
|---|---|
| `19-Certificates` | „**A megújítás-oszlop a lényeg** … a napok száma önmagában a legkevésbé hasznos szám" |
| `21-Kubeconfig` | „**A verdikt-oszlop a panel** … minden context pontosan egy vödörbe esik az ötből" |
| `20-Waste` | „**A pár, nem a százalék** … minden sávnak két jele van" |
| `18-OpenShift` | „a panel ezzel a mondattal kezd, majd **listázza az operátorokat** — legrosszabb elöl" |
| `09-RBAC` | „**a következményt mondja ki előbb**, és a szabályokat alá teszi bizonyítékként" |

Mind az öt: **`DataTable` + egy számított oszlop + egy vezető mondat + rendezés a verdikt szerint.**
A különbség köztük az adatforrás és az oszlopkészlet — vagyis pont az, amit egy **preset** ír le.

**Javaslat:** ne öt paneltípus legyen, hanem **egy `resource list`, mentett nézetekkel**. A preset
tartalma: forrás, oszlopkészlet, a verdikt-szabály, a rendezés, és a vezető mondat sablonja. A
paletta továbbra is „Certificates"-ként kínálja — a *felhasználónak* öt bejárata marad, a *kódnak*
egy panelje.

**Amit nyer:** öt panel-implementáció helyett egy, és a hatodik ilyen igény (kvóta, node-taint,
PDB-lefedettség) nem új panel, hanem egy adatsor.

**Amit veszít:** a vezető mondat panelenként más nyelvtanú. A presetnek ezt sablonként kell vinnie,
és a sablon kevésbé szabad, mint a kézzel írt mondat. Ez a valós ára, és elfogadhatónak tartom.

### 2. `Details` + `Describe` + `Diagnostics` — egy tárgy, három próza

Mindhárom **egy objektumra** nyílik, és mindhárom **mondattal kezd**:

- `10-Details` — „a név és a fázisa egy sorban, 18px-en, **alatta egy mondat prózában**"
- `08-Diagnostics` — „**a mondat, 19px-en** — a termék legnagyobb betűje"
- `09-RBAC` — „**a mondat, megint 19px-en**" ← a tervező maga jelzi az ismétlést
- `23-Describe` — ugyanaz a szöveg, amit a `kubectl describe` ad, három jellel megtoldva

`Describe` **3 megnyitás**, `Diagnostics` **5**. Ezekért külön paneltípust tanítani drága.

**Javaslat:** **egy `Details` panel, három nézettel** — ugyanaz a viszony, ami a `YAML` és a form
között már ki van találva (`13-YAML-and-Editor`: „két nézet ugyanarra a mentetlen objektumra"). A
`Describe` nézet-kapcsoló; a `Diagnostics` **nem külön nézet, hanem egy szekció, ami csak akkor
létezik, ha van baj** — és akkor az első.

**Amit veszít:** a „Miért Pending?" ma egy önálló, megnevezett kérdés, saját billentyűvel (`?`). Ez
maradjon meg: a billentyű a `Details`-t nyitja a diagnosztika-szekcióra ugorva. A bejárat marad, a
panel fogy.

### 3. A „foglalt kontra használt" sáv — három képernyő, egy panel

`07-Resource-Monitor`, `20-Waste`, `24-Node` mind ugyanazt a vizuális eszközt rajzolja:

> `07`: „**a használt sáv a két jelével**… ahol a használt, a kért és a limit egy kép"
> `20`: „**a pár, nem a százalék** — minden sávnak két jele van"
> `24`: „**foglalt kontra használt, kétszer** — a node saját sávja és minden pod sávja ugyanazt a két jelet viseli"

A különbség **csak a hatókör**: élő pod-szint · namespace/flotta-aggregátum · egy node.

**Javaslat:** egy **`Capacity`** panel, aminek a hatókör-választója cluster / namespace / node /
workload. Ez pontosan az a kérdés-panel modell, ami már megvan — a hatókör a fejlécben áll, és a
panel újrakérdezi magát. A `Waste` ennek a panelnek egy rendezése („kért mínusz használt szerint,
csökkenő"), nem külön panel.

**Amit veszít:** a `20-Waste` szándékosan kék, piros nélkül („a pazarlás nem incidens"). Ez a
színszabály a **hatókörhöz, nem a panelhez** kell kötődjön: aggregátum-nézetben nincs piros. Ezt ki
kell mondani, különben elveszik.

### 4. `Cross-Cluster` nem panel, hanem a lista egy módja

A `28-Cross-Cluster` zárónotesze maga mondja ki:

> „**Alapból összevonva — felülírva a scope dokumentumot.** … Flat marad `⇧F`-en, **soronként és
> panelenként**."

Ami soronként és panelenként kapcsolható, az **nem paneltípus, hanem nézetmód**. Ugyanez a `05-Global
Search`: egy `resource list`, aminek a hatóköre „minden cluster" és a szűrője egy lekérdezés.

**Javaslat:** a `resource list` hatóköre legyen `cluster | csoport | flotta`, és a csoport/flotta
hatókörben alapból összevont. `Cross-Cluster` és `Global-Search` mint **paneltípus megszűnik**;
mindkettő ugyanaz a panel, más hatókörrel. A képernyők maradnak — de mint a lista hatókör-állapotai,
nem mint önálló típusok.

### 5. A `diff` eszköz, nem panel

A `diff` **7 műveletből** nyílik, és **négy különböző képernyőn** jelenik meg: `13-YAML-and-Editor`
(mező-szintű, mentés előtt), `17-Helm` (revíziók között), `28-Cross-Cluster` (clusterek között),
`16-What-Would-Happen` (mi lenne). Négyszer megrajzolva, négy helyen.

**Javaslat:** **`DiffView` a hetedik panel-test**, és minden fenti a *forrását* adja meg (két
objektum, két revízió, két cluster, két időpont). Ez az egyetlen hely, ahol **új** komponenst
javaslok — mert ma négyszer van meg, csak nem komponensként.

---

## Amit nem szabad összevonni

Ezeket megnéztem, és **külön kell maradniuk**, hiába hasonlítanak:

- **`Events` és `Change-Stream`.** Ugyanaz a test, de más forrás és más kérdés: „mit jelentett a
  Kubernetes" kontra „mi változott, és mi lett tőle rossz". A `06` zárónotesze a korrelációs
  mondatra épül („megnevezi a változást, a tünetet és a nyolc percet köztük") — ezt az események
  listája nem tudja megadni, mert nincs benne a `spec` előtte-utána.
- **`Log` és `Terminal`.** Mindkettő szövegfolyam, de a terminál **elveszi a billentyűzetet**, és a
  `14-Terminal` egész terve erről szól. Ez a különbség nem kozmetikai.
- **`Settings` mint panel.** A `12` érvelése helyes és le is van rajzolva: a metrika-panel, ami nem
  tud kapcsolódni, és a beállítás, ami megjavítja — egymás mellett. Egy dialógus eltakarná azt,
  amit javítasz.
- **`Assistant`.** Külön szín, ami **benyúl más panelekbe** (a hivatkozott sorokra gyűrűt tesz).
  Ez az egyetlen ilyen a termékben, és pont ezért nem olvasztható semmibe.
- **`First-Run`, `Incident`, `Layouts`, `Time-Travel`, `Bulk-Action`.** Ezek **nem panelek**, hanem
  a keret állapotai vagy forgatókönyvei. A README is így sorolja őket. Nem redundancia.

**A `Kubeconfig` határeset.** Tartalmilag beállítás, és a `12-Settings` érve rá is illik. De az
`1. összevonás` alá tartozik: verdikt-oszlopos lista (öt vödör, negyven sor). **Legyen a `Settings`
egyik szekciója, ami egy verdikt-listát ágyaz be** — így egyik érv sem sérül.

---

## Ütközés azzal, ami már megépült

**A tervező a cluster-sáv törlését javasolja — én azt már megépítettem.**

A `29-Cluster-Rail` zárónotesze:

> „Nulla legitim szélesség. … A mai három névtelen pötty a bal szélen **46 px, amit minden képernyőn
> örökre elköltünk** azért, hogy megmutassunk valamit, amire egyetlen panelnek sem volt szüksége."

Az érv helytálló, és a helyettesítés is ki van dolgozva: az egyetlen pótolhatatlan feladat — *egy
cluster, amit nem nézel, bajban van* — átkerül a fülsávba egy chipre, ami **csak akkor létezik, amíg
igaz** („2 cluster kér téged"). A sáv `⌘\`-re nyílik három szélességben, és **húzás idejére magától
becsúszik**, mert az az egy gesztus akarja tényleg látni az összes célt.

Ez jobb, mint amit építettem. A `ClusterRail.svelte` marad, de **alapértelmezett szélessége 0**, és
kell mellé a fülsáv-chip. Ez nem sok munka, viszont **most kell eldönteni**, mert a `Rail.svelte`
már szerepel a komponensfában „három szélesség"-gel — tehát a tervező is így számolt.

---

## Rendrakás

1. **A `05-panelek/README.md` elavult.** `10-`-től kezdve olyan számozást ír le, ami nincs a
   lemezen: `10-Certificates`, `11-Helm`, `12-Metrics-Browser`, `13-Waste`, `14-Kubeconfig`,
   `15-Watch-Rules`, `16-Details`, `17-Describe`, `18-YAML`, `19-Terminal`, `20-Metrics`, `21-Diff`,
   `22-Impact-Preview`, `23-Drain-Simulation`, `24-Form-Editor` — **egyik sem létezik**. Ami a
   lemezen van: `09-RBAC`, `10-Details`, `11-Assistant`, `12-Settings`, `13-YAML-and-Editor`,
   `14-Terminal`, `15-Metrics`, `16-What-Would-Happen`, `17-Helm`, `18-OpenShift`, `19-Certificates`,
   `20-Waste`, `21-Kubeconfig`, `22-Watch-Rules`, `23-Describe`, `24-Node`. A README-t javítani kell,
   különben minden rá hivatkozó dokumentum rossz fájlra mutat.
2. **Hat prózasor a régi két tervben megdőlt.** A `04-Decisions` mindet megnevezi. A
   `kubescope-frame-design.dc.html` és a `kubescope-panel-design.dc.html` javítandó, hogy ne
   maradjon ellentmondó forrás — ez pont az a hiba, ami tőlem már kétszer kijött.
3. **A tervező a régi két tervet nem kapta meg**, és a értékeket a brief idézeteiből állította
   vissza. Ezt pótoltam (`brief/plans/`), de a következő körnek ezt tudnia kell.

---

## Mi következik ebből az építésre

A jelenlegi állás: a keret megvan (fa, osztás, húzás, munkaterületek), a panelek egyike sincs kész.
A fenti összevonások után a **panelsorrend** ez:

| # | mit | miért ez a sorrend |
|---|---|---|
| 1 | **`resource list`** teljesen, presetekkel | 48 megnyitás, és öt másik képernyő ebből lesz |
| 2 | **`Details`** a három nézetével | 40 megnyitás, és két másik képernyő ebből lesz |
| 3 | **`LogView`** | 8 megnyitás, és a spike már bizonyította |
| 4 | **`YamlView` + form** | a `13` szerint ez egy panel két nézettel, nem kettő |
| 5 | **`DiffView`** | négy helyen kell, egyszer megírva |
| 6 | **`Capacity`** | három képernyő egy panelben |
| 7 | a maradék | mind kicsi, ha az első hat áll |

**Az 1. és a 2. a termék fele.** Ha ez a kettő jó, minden más ráépül; ha nem, a többi harminc
képernyő sem ment meg semmit.

---

## Egy mondatban

A design **nem redundáns a rajzolás szintjén** — hat test, huszonhét komponens, tisztán faktorálva.
**A redundancia a paneltípusok szintjén van**: tizenkilenc típusból tizenegyet legfeljebb négy
művelet nyit meg, és ötöt közülük ugyanaz a „lista plusz verdikt-oszlop" minta ír le. Az öt
összevonás után **tizenkilenc típusból tizenegy marad**, a felület egyetlen bejárata sem tűnik el,
és a `resource list` presetjei viszik azt, amiért eddig öt külön panel járt.
