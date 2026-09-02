# KubeScope — Design brief: fogantyúk, gesztusok, kijelölés

> A Claude Design bemenete. Párja a `kubescope-design-brief-frame.md` (a keret), a
> `kubescope-design-brief-panes.md` (a panelek) és a két elkészült terv:
> `kubescope-frame-design.dc.html`, `kubescope-panel-design.dc.html`.
>
> **Nem új vizuális nyelvet kérünk, és nem új szerkezetet.** Mindkettő megvan. Ami hiányzik: a
> felület **megfogható részei** — amihez az ember hozzáér, ami visszajelez, és ami közben történik.

---

## 1. A feladat egy mondatban

Tervezd meg azt, ami **a kattintás és az eredmény között** van: a fogantyúkat, a húzás közbeni
állapotokat, és a kijelölés teljes nyelvtanát — mit lehet kijelölni, mit nem, hogy néz ki, és mikor
szűnik meg.

## 2. Miért ez a feladat

A két elkészült terv **statikus képeket** ad: így néz ki egy panel, ilyen a nyolc adatállapot, ez a
degradáció három szélességen. Ez pontos és használható volt — a keret ebből épült meg.

De a keret **mozog**. Panelt húzunk át a szomszédjára, osztásfogót tolunk, sorokat jelölünk ki,
fület váltunk. Ezekről a terv **méreteket ad, rajzot nem**:

> `SplitHandle` — 3 px rajz, 9 px kattintási sáv, húzás közben 2 px cián, dupla kattintásra 50/50.
>
> `PlacementPreview` — Húzás vagy „Hova…" közben: a cél-panel négy éle kiemelkedik, a leendő hely
> 10% cián kitöltést kap. Egyetlen alfás téglalap.

Két mondat, két widget. **Ebből nem lehet megépíteni, csak kitalálni** — és a keret jelenlegi
állapota pontosan ezt mutatja: kitalált fogantyúkat kitalált állapotokkal. Ránézésre látszik, hogy
nem tervezett.

Ugyanez a kijelölésnél. A terv egy mondatot ad rá (§5.6), és abban a próza és a kód **nem egyezik**
(lásd §7). Minden más — többes kijelölés, más paneltípusok, élettartam — nincs kimondva.

## 3. Amit ne tervezz újra

- **A tokenkészletet.** Kész, be van építve.
- **A szerkezetet.** Fejsáv · cluster-sáv · panelmező · állapotsor; a fa, a mélységkorlát, a
  minimumok. Kész.
- **A panelfejlécet és a vezérlősávot** (28 + 26 px). Kész, és a kód szintjén pontos.
- **A nyolc adatállapotot.** Kész.
- **A fókuszgyűrűt mint nyelvet.** 2 px cián, kifelé eső. Ezt **alkalmazni** kell az új helyeken,
  nem újratervezni.

## 4. A helyzet, amiben ez számít

Hat panel, három clusterből, incidens közben. Az ember **gyorsan** nyúl: megfog egy panelt, ráhúzza
egy másikra, elenged. Kijelöl nyolc podot, és mind a nyolcra ugyanazt akarja. Közben az adat alatta
**változik** — sor tűnik el, sor jön be.

Ebben a helyzetben minden bizonytalanság drága. „Hova fog kerülni?", „mit jelöltem ki?", „ez még
mindig ki van jelölve?" — ezekre a képernyőnek válaszolnia kell, mielőtt megkérdeznék.

---

## 5. Amire válaszolnod kell — a fogantyúk

### 5.1 Az osztásfogó

Három állapot kell, rajzolva: **nyugalom · rámutatás · húzás közben**.

A terv annyit mond: 3 px rajz, 9 px kattintási sáv, húzás közben 2 px cián, dupla kattintásra 50/50.
Amit nem mond:

- **Látszik-e nyugalomban egyáltalán?** A panelek között úgyis van 1 px keret. Ha a fogó ugyanolyan,
  akkor nincs jele, hogy megfogható; ha másmilyen, akkor hat panelnél öt vonal kiabál.
- **Mi történik rámutatáskor** — a 3 px vastagszik, világosodik, vagy csak a kurzor vált?
- **A kurzor.** `col-resize` / `row-resize` elég, vagy a keretnek saját kurzora van?
- **Húzás közben a fogó marad az egér alatt, vagy a panelek követik élőben?** Az élő követés
  drágább, de a keret amúgy is újrarajzol.
- **A dupla kattintás visszaállítja az 50/50-et — visszajelez valamivel?** Egy ugrás magyarázat
  nélkül baleset benyomását kelti.
- **Ütközés a minimummal:** ha a fogó eléri a 320 px-et, megáll — de *mutatja* is? A meg nem
  magyarázott ellenállás elromlásnak érződik.

### 5.2 A panel megfogása és elhelyezése

Ez a keret legkockázatosabb gesztusa, és **egyetlen mondat** van róla.

- **Miről fogható meg a panel?** A fejléc a fogantyú — de a fejlécben hat interaktív elem van
  (`cluster ▾`, típus, `ns ▾`, `⊞`, `✕`). Melyik pixel fog, melyik kattint? Mutassa a rajz.
- **Mikor kezdődik a húzás?** Küszöb kell, különben minden kattintás fél panelt mozdít. Hány pixel,
  és **mi jelzi, hogy elindult**?
- **Mi látszik az egér alatt húzás közben?** A megfogott panel a helyén marad és csak a cél
  világít? Vagy van kísértet? Ha van, mekkora, és mit tartalmaz — a fejlécét, egy glifet, semmit?
- **A cél megjelölése.** A terv szerint „a cél-panel négy éle kiemelkedik, a leendő hely 10% cián
  kitöltést kap". **Négy él plusz egy kitöltés — ez két jelzés egyszerre.** Kell mind a kettő?
  Rajzold le mind a négy irányra (bal · jobb · fel · le) **és** a „fülként" esetre, mert az az ötödik
  cél és egészen máshogy néz ki.
- **A visszautasított ejtés.** Ha a hely nem érvényes (saját magára, mélységkorlát fölött), **mit
  lát az ember?** Ma semmit — ez a legrosszabb válasz.
- **Az elengedés.** Ugrik, vagy átúszik? A design system 140 ms-os panel-időtartama ide való?

> **Amit ma csinál, és amit el kell dönteni:** a keret **élő próbát** rajzol — az egész elrendezést
> újraszámolja a művelettel, és a *teljes leendő képernyőt* mutatja halványan, nem egyetlen
> téglalapot. Ez azért lett így, mert a számított előnézet **hazudott**: ha egy panelt kiveszünk,
> az osztás, amiben volt, összecsuklik, és a cél megnő — az egytéglalapos előnézet ezt nem tudta
> megmutatni. **Ez a terv „egyetlen alfás téglalap" előírásával ütközik.** Döntsd el, és indokold: a
> pontosság vagy a csend a fontosabb.

### 5.3 A fülcsík

A terv: „Fülcsík a levél tetején, 22 px. Az aktív fül alsó éle 2 px cián." Ennyi. Kell:

- Fül nyugalomban · rámutatva · aktívan · húzás közben.
- **Mit visel egy fül?** Típusglif + cím? Csak glif? A panel neve hosszú (`coredns-64d64f99dd-7fjvd`).
- **Zárógomb a fülön?** Ha igen, mindig látszik vagy csak rámutatásra — és mi történik keskeny
  panelben, ahol a fülcsík amúgy is szűk?
- **Túlcsordulás:** hét fül egy 320 px-es levélen. Görgetés, összenyomás, vagy „+3"?
- **A fülcsík eltűnik-e egy fülnél?** (Valószínűleg igen — de mondd ki, mert a hely 22 px.)

### 5.4 A panel három állapota

A terv a **fókuszt** definiálja, kétféleképpen, és a kettő nem ugyanaz:

| forrás | amit mond |
|---|---|
| keret-terv, „Fókuszkeret" | 2 px cián keret **+ 4 px cián bal él** |
| panel-terv, `paneHeader` kódja | a **fejléc háttere** `rgba(63,224,208,.06)`, keret nincs |

**Melyik?** Vagy mind a kettő, más szinten? És hiányzik a harmadik állapot: **a rámutatás**. Hat
panelnél az egér folyton panel fölött van — ha ez semmit nem csinál, az rendben van, de ki kell
mondani, mert a hallgatás nem döntés.

### 5.5 A kurzorok

Egyetlen kurzor van megnevezve az egész tervben. Kell egy rövid, teljes lista: mikor `grab`, mikor
`grabbing`, mikor `col-resize`, mikor `text`, mikor `default` — mert a kurzor az egyetlen visszajelzés,
ami *a mozdulat előtt* érkezik.

---

## 6. Amire válaszolnod kell — a kijelölés

Ez a nagyobbik hiány. A terv egy mondatot ad rá, és abból is a felét a kód cáfolja.

### 6.1 Mit lehet kijelölni

A panelek **két családba** esnek (kérdés-panelek: pods, nodes, events; tárgy-panelek: logs, yaml,
shell, metrika). A kijelölés ma csak a kérdés-panelek soraira van kimondva. Kell:

- **Napló-panel:** kijelölhető-e egy sor? Egy tartomány? A napló szövegfolyam — a szövegkijelölés
  (`text`) és a sorkijelölés **ütközik**. Melyik nyer, és hogyan lehet a másikhoz jutni?
- **YAML-panel:** blokk kijelölhető? Egy kulcs? Ez a másolás miatt számít.
- **Metrika-panel:** időtartomány kijelölése (kihúzás a grafikonon) — ez kijelölés vagy szűrés?
- **Shell-panel:** itt a kijelölés a terminálé. A keret nem nyúlhat hozzá. Kimondva?
- **Az objektum-panel** (egy pod áttekintése): van-e egyáltalán kijelölhető dolga?

### 6.2 Többes kijelölés — van vagy nincs?

**A tervben egyetlen szó sem esik róla.** Nincs `⌘-katt`, nincs `⇧-katt`, nincs „többes".

Ez döntést kíván, mert nagy tétje van: a termék ígérete, hogy nyolc podra egyszerre lehet ugyanazt
csinálni. Ha van többes kijelölés, kell hozzá:

- A hozzáadó gesztus (`⌘`/`⇧` katt, `⇧`+nyíl, „mind" gomb?) és **a tartomány** gesztusa.
- **Hogy néz ki nyolc kijelölt sor** — ugyanaz a 8% cián nyolcszor egy összefüggő tömbben egészen
  mást ad ki, mint egyszer.
- **Hol látszik, hogy nyolc van?** Számláló a vezérlősávban? És hogy lehet elengedni mind?
- **Mi történik, ha a nyolcból három eltűnik** (a pod meghalt)? A kijelölés hallgatólagosan
  hatra fogy — ez incidens közben veszélyes, mert a következő művelet kevesebbre megy, mint hitted.
- **A művelet-sáv.** Ha nyolc sor ki van jelölve, megjelenik-e valami, ami megmondja, mit lehet
  velük? A terv szerint minden művelet a palettában és a jobbklikkben van — de nyolc sornál a
  „mit lehet" kérdés sürgetőbb.

Ha **nincs** többes kijelölés, azt is mondd ki, indoklással — a keret ma megcsinálta (`⌘`/`⇧`),
mert kézenfekvőnek tűnt, és ki kell derülnie, hogy ez a terv szerint helyes-e.

### 6.3 Hogy néz ki

A terv **próza és kód között ellentmond**:

| | próza (§5.6) | kód (`table()`) |
|---|---|---|
| kitöltés | 10% cián | `rgba(63,224,208,.08)` = 8% |
| 2 px bal él | „cián bal jel", a kijelölésé | `r.mark` — **külön csatorna**, státuszszín |

**Melyik a 2 px bal él?** Ha a kijelölésé, akkor a státusznak kell másik hely. Ha a státuszé, akkor
a kijelölésnek csak a kitöltés marad — és egy 8%-os kitöltés egy piros hátterű hibás soron
(`rgba(255,77,109,.05)`) **majdnem láthatatlan**. Ez a konkrét eset — kijelölt hibás sor — rajzot
kíván, mert incidens közben pont ez a sor érdekes.

Továbbá:

- **Kijelölés fókusz nélkül.** A terv kimondja, hogy a kettő külön csatorna, és egyszerre látszanak.
  De mi van, ha a **panel** veszti el a fókuszt? Marad a kijelölés teljes erővel, vagy tompul? Két
  panel, mindkettőben kijelölt sor, csak az egyik aktív — ezt le kell rajzolni.
- **A billentyűzetes „megálló"** (`↑ ↓`) és a kijelölés viszonya: a mozgás kijelöl is, vagy csak
  mozgatja a fókuszgyűrűt, és a `Space` jelöl ki?

### 6.4 Meddig él

A terv annyit mond, hogy a kijelölés **munkamenet-szintű**, mert „egy objektumra mutat, ami közben
megszűnhetett". Jó szabály. Ami hiányzik, az a **közben**:

- **Szűrés.** A kijelölt sor kiesik a szűrőből — megszűnik a kijelölés, vagy láthatatlanul megmarad?
  (A keret ma **eldobja**, mert egy nem látszó kijelölésre az ember rálép anélkül, hogy látná. Ez
  védhető, de nem tervezett.)
- **Cluster- vagy típusváltás a fejlécben.** Nyilván eldobja — de mondd ki.
- **A sor eltűnik** (a pod törlődött), miközben ki van jelölve. Csendben eltűnik? Vagy a sor helye
  megmarad egy pillanatra, áthúzva?
- **Rendezés.** A kijelölt sor elugrik a lista aljára. Görgessen utána a panel?

---

## 7. Ellentmondások, amiket dönteni kell

Ezeket menet közben találtuk, és mindegyik két, egyaránt védhető szabály között feszül.

| # | ütközés | ami ma van |
|---|---|---|
| 1 | **Fókusz:** 2 px cián keret + 4 px bal él *(keret-terv)* vs. cián fejléc-háttér *(panel-terv kódja)* | 1 px keret |
| 2 | **Fejléc-vezérlők:** öt gomb, 26 px, `⌕ ⇱ ⊞ ⋯ ✕` *(keret-terv)* vs. két gomb, 17 px, `⊞ ✕` *(panel-terv kódja)* | két gomb, 17 px |
| 3 | **Elhelyezés-előnézet:** „egyetlen alfás téglalap" vs. a teljes leendő elrendezés élő próbája | élő próba |
| 4 | **A `⊞` elhalványul 643 px alatt** (`320 + 3 + 320`, tehát felezés) vs. **az új panel 40%-ot kap** | a 40% szabály, a küszöb ebből számolva (804 px) |
| 5 | **Kijelölés bal éle:** a kijelölésé *(próza)* vs. `r.mark`, státuszcsatorna *(kód)* | nincs bal él |
| 6 | **Kijelölt sor kitöltése:** 10% *(próza)* vs. 8% *(kód)* | — |

**Kérés:** ahol a kód és a próza szétmegy, a **kód** legyen az alap, de mondd ki, melyiket
választottad, hogy a prózát javítani lehessen. Ne hallgatólagos legyen.

---

## 8. Amit át kell adnod

1. **Egy oldal, ami a fogantyúkat mutatja** — osztásfogó három állapotban, fülcsík négy állapotban,
   a panel három állapotában, kurzortáblázat.
2. **Egy oldal a húzás teljes ívéről** — megfogás → küszöb → mozgás → mind az öt cél (négy él +
   fülként) → érvénytelen cél → elengedés. Képkockákkal, ahogy a nyolc adatállapotot is
   végigrajzoltad.
3. **Egy oldal a kijelölésről** — egy sor, nyolc sor, kijelölt hibás sor, kijelölés inaktív panelen,
   kijelölés + fókuszgyűrű együtt, és a többes kijelölés összes kelléke (számláló, elengedés).
4. **Egy táblázat a kijelölés élettartamáról** — melyik esemény mit csinál vele, ugyanabban a
   formában, ahogy a perzisztencia-táblázat készült.
5. **A §7 hat ellentmondására egy-egy döntés**, indoklással és a vesztes opció megnevezésével —
   ahogy a Q1…Q9 döntések készültek. Ez a legfontosabb tétel: enélkül minden más találgatás marad.

## 9. A mérce

Akkor jó, ha **meg lehet építeni belőle anélkül, hogy egyszer is kitalálnék valamit** — és ha egy
hónap múlva, egy új paneltípusnál, a válasz „a fogantyú-oldal 3. sora", nem „csináld úgy, mint a
podoknál".

Az eddigi tervek mércéje ez volt, és tartották. Ez a rész maradt ki.
