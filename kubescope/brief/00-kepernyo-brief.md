# KubeScope — Design brief: képernyőtervek

> Ez a dokumentum a Claude Design bemenete. **Nulláról tervezz.** Nincs korábbi terv, amit folytatni
> kellene, és nincs készlet, amihez igazodni — csak ez a specifikáció. Képernyőterveket kérünk az
> egész alkalmazásra, és egy prototípust, amiben végig lehet kattintani.
>
> Volt egy korábbi felület. **Nem adjuk oda, és nem is akarjuk, hogy hasson rád** — rossz lett, és a
> tanulságait a `2.` fejezet írja le, kép nélkül, hogy a hibái ne horgonyozzanak le.
>
> 🔴 **A legfontosabb változás az előző körökhöz képest: amit kiadsz, az lesz a kódunk.** Az
> alkalmazás átáll **Tauri + React + TypeScript**re: helyi Rust folyamat végzi a Kubernetes-munkát, a
> felület webview-ban fut. Eddig HTML-ben terveztél, és én kézzel portoltam egy rajzoló API-ra —
> ezért néz ki rosszul, amit ma látsz. **Ez a fordítás megszűnik.** Írj úgy, mintha az
> implementációt írnád, mert az.

> ⚠️ **Ez a kör három versengő tervet kér, nem egyet.** A menete a `3.` fejezetben áll — előbb három
> irány ugyanazon a három képernyőn, aztán a nyertesből teljes terv. Ne ugorj a második körre.
>
> ⚠️ **A szerkezet eldőlt, és marad.** Az alkalmazás **tiling modellel** működik: számozott
> munkaterületek, azokban osztott fa, a fában panelek, panelenként saját hatókör. Ez a termék ötlete,
> nem egy megvalósítási részlet — **erre építs**. Amit kérünk, az a **kinézet**: hogy ugyanez a
> szerkezet végre jól nézzen ki, és hogy minden képességnek meglegyen benne a helye.

---

## 1. A feladat egy mondatban

Adj **három, egymástól gyökeresen különböző vizuális irányt** ugyanerre az alkalmazásra, ugyanazon a
három képernyőn bemutatva — hogy legyen miből választani —, és a kiválasztottból utána teljes
képernyőtervet és működő prototípust.

## 2. Öt csapda, amibe már beleestünk

Épült egyszer egy felület erre a specifikációra, és rossz lett. Nem mutatjuk meg — de a tanulságokat
igen, mert mind az öt ennek a **feladatnak** a csapdája, nem egy tervező hibája. Ha a terved egyiket
sem lépi meg, már jobb, mint amink volt.

1. **Minden azonos súlyú lesz.** Egy műszaki eszközben minden információ „fontos", és ha ezt
   komolyan veszed, a végén minden 12 pixeles szürke szöveg. **Kell egy elsődleges elem** minden
   képernyőn — mondd is meg, melyik az.
2. **A króm vízszintes csíkokban rakódik.** Panelfejléc, szűrősor, oszlopfejléc: három vonalka apró
   szöveggel, mielőtt egyetlen adatsor jönne — és négy panelnél tizenkettő. **Számold ki, mennyi a
   króm**, és mondd meg, mit adsz érte.
3. **A szín kimarad.** Sötét felületen könnyű mindent szürkére venni, és a végén egy nyolc pixeles
   pötty az egyetlen színes elem a képernyőn. A státusz megérdemel egy csatornát, de a
   **hierarchiának is kell egy**.
4. **A sűrűség és az olvashatóság szembekerül.** Negyven sor egy laptopon szép szám, de ha közben
   csonkolt neveket olvasol, semmit nem ér. Dönts, és mondd ki, mit választottál.
5. **Táblázatnak néz ki, nem eszköznek.** A cél nem egy adatrács, hanem valami, amit valaki éjjel
   kettőkor nyit meg, mert baj van, és **azonnal látja, hol**.

**A szerkezetet viszont ne tervezd újra.** A tiling — munkaterületek, osztott fa, panelek,
panelenkénti hatókör — bevált, és ez a termék lényege. Ami hiányzik, az a *kinézete*.

## 3. A kör menete — előbb három irány, aztán egy terv

### 3.1 Első kör: három irány

**Három terv, ugyanarra a három képernyőre.** Ugyanaz az adat, ugyanaz a méret, hogy
összehasonlíthatók legyenek:

1. **Egy pod-lista, egy panel a teljes munkaterületen** — a tábla, a hatókör, a szűrés.
2. **Az incidens: négy panel, három cluster** — lista + napló + esemény + terminál egyszerre. Ez a
   nehéz eset, és ez dönti el a legtöbbet.
3. **A flotta: 8–10 cluster egészsége** — ettől termék ez, és nem k9s-másolat.

**A három irány legyen tényleg különböző.** Nem három színséma ugyanarra a rácsra. Térjenek el
abban, ami számít:

- **mennyi a króm** — a keret mennyit vesz el, és mit ad érte;
- **mennyire sűrű** — hány sor fér el, és milyen áron;
- **honnan jön a hierarchia** — méretből, súlyból, színből, felületből vagy térközből;
- **mi a karakter** — mérnöki műszerfal, csendes dokumentum, vagy valami harmadik.

Mindegyikhez egy bekezdés: **mire jó, mit ad fel, és kinek a napja lesz tőle jobb.** És ha van
köztük olyan, amelyiket te magad nem választanád, mondd meg, melyik és miért — három egyformán
melegen ajánlott irány három rossz irány.

**Mindhárom nulláról induljon.** Nincs meglévő készlet, amihez igazodni kell, és nem is kapsz
ilyet. A betűtípus is a te döntésed — bármi, ami beágyazható; mondd meg, mit választottál és miért.

### 3.2 Második kör: a nyertes kidolgozása

Csak akkor kezdd, ha megmondtuk, melyik nyert. Akkor jön a `4.` fejezet teljes képernyőlistája, a
`6.` fejezet prototípusa, és a csiszolás — az addig fog tartani, ameddig kell.

---

## 4. Mit tervezz meg — a teljes termék, nem a megépült része

A `kubescope-scope.md` a képességtérkép. **Tervezz mindegyikhez képernyőt**, akkor is, ha még egy
sor kód sincs mögötte — pont az a lényeg, hogy lássuk, hova fognak kerülni:

| jel | terület | amit meg kell mutatni |
|---|---|---|
| `CONN` | kapcsolat, flotta | első indítás, cluster hozzáadása, 10 cluster egészsége egy képernyőn |
| `RES` | erőforrás-böngészés | listák, részletek (áttekintés / események / tulajdonosi lánc / YAML), CRD-k |
| `XC` | cross-cluster | ugyanaz az objektum N clusteren, eltérés-követés, összefésült listák, verzió-mátrix |
| `LOG` | logok | egy konténer folyama, **N cluster logja egymásba fésülve**, keresés, előző példány |
| `MET` | metrikák | görbe, események a görbére ültetve, requests/limits vs. valós használat |
| `TIME` | visszatekintés | **a teljes felület egy múltbeli időpontban**, mezőszintű diff, időablak exportja |
| `DIAG` | diagnosztika | „miért nincs ütemezve ez a pod", kapacitás, árva erőforrások |
| `SEC` | biztonság | „mit tud ez a ServiceAccount", tanúsítvány-lejáratok flottaszinten |
| `OCP` | OpenShift | Route, Build, SCC — ahol eltér a Kubernetestől |
| `ACT` | írás | rolling restart, skálázás, törlés — megerősítéssel, több clusteren egyszerre |
| `APP` | keret | parancspaletta, elrendezések mentése, billentyűzet |

**Nem kell mindet ugyanolyan mélységben.** Válaszd ki azt az **öt-hat képernyőt, ami eldönti a
termék karakterét**, azokat tervezd meg teljesen, a többit vázlat szinten — de mindegyik férjen bele
ugyanabba a rendszerbe, és látszódjon, hogy belefér.

## 5. Amit tudni kell, mielőtt tervezel

### 5.1 Ki használja és mikor

Platform- és SRE-mérnök, aki **egyszerre több clustert** tart életben. A tipikus megnyitás nem
böngészés, hanem: *„valami elromlott, hol?"* Éjjel, több ablak közt, sietve. A második leggyakoribb:
*„ugyanaz fut mind a nyolc clusteren?"*

**Ez a termék létjogosultsága a több cluster.** Egy clusterre a k9s gyorsabb. Amit ő nem tud: nyolc
cluster egy képernyőn, egymásba fésült logok, cross-cluster eltérés, és visszatekintés arra, milyen
volt a világ fél órája.

### 5.2 A valódi adat

Ne találj ki mezőket — ennyi van, és ennyiből kell képernyőt csinálni:

- **Cluster**: név (kubeconfig kontextus), apiserver URL, namespace, be van-e kapcsolva, írható-e,
  kapcsolat állapota (*nem kérdeztük · próbáljuk · él (RTT) · lassú (RTT) · nem enged be · nem
  válaszol · szüneteltetve*), verzió, platform (k8s/OpenShift), API-csoportok száma, CRD-k száma.
- **Pod sor**: namespace, név, fázis (Running/Pending/Succeeded/Failed/CrashLoop/Degraded/
  Terminating/Unknown), kész konténerek `1/1`, újraindulások, node, kor, ok.
- **Node sor**: név, állapot, szerep, verzió, kapacitás/allokáció, taintek, kor.
- **Esemény**: idő, súly, ok, üzenet, érintett objektum, hányszor.
- **Log sor**: idő, konténer, szint (ha kiolvasható), szöveg — és **melyik clusterből**, ha
  összefésült.
- **Folyam állapota**: tétlen · töltés · él · újracsatlakozik · leállt.

### 5.3 Amiben nem vagy szabad — és amiben igen

**Böngészőmotorra tervezel**, tehát a szokásos eszköztár mind rendelkezésre áll: `clip-path`,
árnyék, gradiens, `backdrop-filter`, SVG, animáció, egyedi betűtípus.

**Két dolgot tarts szem előtt.** Az egyik: rendszer-webview fut alatta (macOS-en WKWebView), tehát
maradj annál, amit a Safari is tud — ne építs olyanra, ami csak Chromiumban van. A másik:
teljesítmény. Tíz cluster, több ezer soros listák, folyamatos logfolyam. **Minden hosszú lista
virtualizált**, és semmi ne kényszerítsen ki elrendezés-újraszámolást minden képkockában.

**Ezekkel a könyvtárakkal fogunk dolgozni** — tervezz rájuk, ne ellenük:

| mire | mivel |
|---|---|
| terminál | `xterm.js` (saját vászon, saját betűméret-kezelés) |
| grafikon | `uPlot` (a `MET-04` eseményjelölői a vásznára rajzolva) |
| YAML | CodeMirror 6 (összecsukás, keresés, diff) |
| hosszú listák | TanStack Virtual, saját sorrajzolással |
| menü, párbeszéd | Radix primitívek, **saját bőrrel** — a fókuszcsapdáért és a billentyűzetért |

**Kötött az elrendezési modell:** munkaterületek, osztott fa, panelek, panelenkénti hatókör. A
panelek arányait, a króm mennyiségét, a fejléc magasságát, a szegélyek és elválasztók természetét
szabadon újratervezheted — a *modellt* nem.

**Minden más a tiéd:** szín, sűrűség, tipográfia, hierarchia, rétegzés, karakter, betűtípus. Egy
formai kérés: a **tokeneket CSS custom property-ként** add meg, mert abból lesz a témánk.

**Két dolog viszont követelmény, nem ízlés.** Az egyik: **a státusz soha ne csak színen múljon** —
kell mellé alak és szó is, mert a felhasználók egy része nem különbözteti meg a pirosat a zöldtől, és
mert az incidens a legrosszabb pillanat erre rájönni. A másik: **egy panel mindig mondja meg, melyik
clusterből való** — négy panel három clusterből a mindennapi eset, és egy tévesztés itt drága.

### 5.4 Ami már működik — a viselkedés, nem a kinézet

Megépült és **bevált**: munkaterületek (i3-szerű, számozott, `⌘1…0`), osztott fa panelekkel, húzható
panelek és fülek élő próbával, panelenkénti hatókör, valódi pod-tábla élő figyeléssel, terminál,
naplónézet, YAML-nézet.

**A gesztusok jók, és ne tervezd újra őket.** Panel nyitása, osztás, húzás, fülezés,
munkaterület-váltás: ezek működnek, és a felhasználó szereti őket.

**A kinézetük viszont teljesen a tiéd**, mert az a rossz: hol vannak a szegélyek, mennyi a króm, mi
a világos és mi a tompa, mitől látszik, melyik panel aktív, és mitől néz ki négy panel négy
paneljének, nem négy táblázatnak egymás mellett.

**A jelenlegi felület kódja megszűnik.** Ne kíméld semmiért — a Rust oldalból csak az adatréteg
marad, a felület a te tervedből épül újra.

## 6. Amit át kell adnod — a második körben

> Ez a fejezet a **második körre** vonatkozik. Az elsőben csak a `3.1` három képernyője kell,
> irányonként — de már Reactben és CSS-ben, mert abból lesz a kód.

**A második kör két dolgot kér: képernyőket és egy prototípust — Reactben és CSS-ben, olyan
minőségben, hogy elvihessük.** Nem mockot: komponenseket, amiket hamis adattal etetsz, és amiknek
később a Rust oldal adja ugyanazt az adatot. Az `5.2` mezőlistája a szerződés a kettő között.

### 6.1 Képernyőtervek

Teljes képernyők, **1600×1000-en** (az alapértelmezett ablakunk) és **1280×800-on** (a legkisebb, amit
támogatunk). Valódi kinézetű adattal — ne `Lorem ipsum`, hanem podnevek, namespace-ek, hibaüzenetek,
amilyenek tényleg vannak. Mindegyik a tiling modellben: **ez nem hat különálló képernyő, hanem hat
elrendezés ugyanabból az alkalmazásból.**

Minimum, teljes mélységben:

1. **Első indítás** — nincs cluster, nincs elrendezés. Mit lát, mi az első dolga.
2. **A flotta** — 8–10 cluster egészsége egy képernyőn, köztük kettő beteg.
3. **A munkanap** — egy cluster podjai, ahogy valaki tényleg nézi, kijelöléssel és következő lépéssel.
4. **Az incidens** — négy panel, három cluster, egymás mellett: lista + log + esemény + terminál.
5. **A visszatekintés** — ugyanaz a képernyő fél órával ezelőtt (`TIME-01`), és ahogy az ember odaáll.
6. **A cross-cluster kérdés** — „ugyanaz fut mindenhol?", eltéréssel kiemelve.

Vázlat szinten a maradék: metrika+esemény idővonal, diagnosztika, RBAC-válasz, tanúsítványok,
írási művelet megerősítéssel, parancspaletta, OpenShift-eltérések.

### 6.2 A prototípus — ez a kör lényege

**Kattintható, működő oldal**, amiben végig lehet menni. Nem videó, nem képsorozat: valódi
interakció. Ezeknek működniük kell benne:

- panel nyitása üresből, típusválasztással;
- egy panel kettéosztása, és a második megtöltése;
- cluster- és namespace-váltás egy panelen belül, és látszik, mi változik tőle;
- szűrés egy listán, a találatszámmal együtt;
- **egy sor kijelölése és abból a következő panel megnyitása** (log, YAML, terminál);
- egy panel, ami elvesztette a kapcsolatot — hogyan mondja meg, hogy amit mutat, már régi;
- váltás egy sűrűbb és egy levegősebb módra, ha a terved kínál ilyet.

A cél az, hogy **látni lehessen, hogyan működik**. Ha valamit nem lehet kattinthatóvá tenni, mutasd
állapot-sorozatként egymás mellett — de a fenti hat lehetőleg legyen élő.

### 6.3 És amit mellé mondj

- **Mit dobtál el a mostaniból, és miért.** Nem kell kímélni.
- **Hol vannak a komponenshatárok** — mi az újrahasznált elem, és mi az egy képernyőre szabott.
- **Mi a képernyő elsődleges eleme** — mire száll le a szem, és hogyan éred ezt el.
- **Hol tart a sűrűség**: hány adatsor látszik 1080p-n, és mit adtál fel érte.
- **Mi az, ami az eguival drága** a tervedben, ha van ilyen.

## 7. Bemenetek

Ebben a projektben csak ez a négy fájl van, és mind a négy kell:

| fájl | mi ez |
|---|---|
| `00-kepernyo-brief.md` | ez a dokumentum |
| `01-kepesseg-scope.md` | a teljes képességtérkép — **ez a szerződés** |
| `02-hol-tartunk.md` | a viselkedési modell: a tiling, a két panelcsalád, a valódi adat, a méretek |
| `03-kepernyolista.md` | **ugyanaz panelenként rendezve** — melyik paneltípus létezik és mi van benne. A tervezéshez ez a hasznosabb sorrend, és ez mondja meg, mit kell a k9s-től elvenni |
Ennyi, és szándékosan ennyi. **Nincs korábbi design system és nincs képernyőkép** — nem akarjuk,
hogy egy rossz megoldás lehorgonyozza a tiédet.

## 8. A mérce

Egy terv akkor jó, ha ránézve **eszköznek látszik, nem táblázatnak** — és ha a prototípusban végig
lehet kattintani egy incidenst anélkül, hogy bárhol meg kellene kérdezni, most mi történik.

És egy mondatban, amit a megrendelő kért: **a tiling modellre kell egy rendes, szép design.** A
szerkezet megvan; a kinézet hiányzik.
