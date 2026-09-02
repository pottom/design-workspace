# KubeScope — építési katalógus

> **Ez az a lap, amiről építek.** Egy sor panelenként: melyik tervrajz, melyik test, milyen hatókör,
> mi van eldöntve, és mi hiányzik még hozzá. Ha valami itt nem szerepel, azt nem szabad kitalálni —
> vissza kell keresni a tervben, vagy kérdésként felvenni.
>
> Forrás: `design-workspace/kubescope/` — a `design/README.md` a navigáció, ez a végrehajtás.
> Frissítve: 2026-09-02.

---

## 1. A rétegek

| réteg | hol van | állapot |
|---|---|---|
| Mag (Rust) | `kubescope-core/` | **kész** — `catalogue` `resource` `table` `object` `runtime` `logs` |
| Keret (Svelte) | `app/src/frame/`, `App.svelte` | **kész** — fa, osztás, húzás, munkaterületek, perzisztencia |
| Formázás | `app/src/format/` | **kész** — idő, egység, csonkítás, státusz, naplósor, hiba, rendezés |
| Akciótérkép | `app/src/actions/map.ts` | **kész** — 32 teszt |
| Tokenek | `app/src/theme.css`, `theme.ts` | **kész** — `--ks-*`, a tervvel azonos nevek, semmi más nem ír színt |
| Panel-testek | — | **egyik sincs meg** |
| Primitívek | — | **egyik sincs meg** |

---

## 2. A hét test

A `PaneBody` egyetlen `switch`-e. **Ez a teljes lista** — minden panel ezek egyikét használja.

| test | mit tud | méret | kik használják |
|---|---|---|---|
| `DataTable` | virtualizált sorok, rendezhető/átméretezhető oszlopok | ~260 | a panelek fele |
| `LogView` | `Gutter` + szövegtest valódi szövegkijelöléssel | ~180 | Log |
| `YamlView` | `Gutter` + kiemelt szöveg, kulcs-alfa kijelölés | ~160 | YAML, Describe |
| `TerminalView` | xterm-host, **elveszi az összes billentyűt** | ~120 | Terminal |
| `MetricView` | görbék eseményjelekkel, vízszintes húzás = időablak | ~200 | Metrics |
| `DetailView` | szekcionált tények egy objektumról, semmi nem kijelölhető | ~140 | Details, Node, Helm, … |
| `DiffView` | **még nincs megrajzolva komponensként** — ma négy helyen szerepel | ~150? | YAML-editor, Helm, Cross-Cluster, What-Would-Happen |

`Gutter` közös a `LogView` és a `YamlView` között — a sor- és blokk-kijelölés ott lakik, hogy a
testben megmaradhasson a natív szövegkijelölés.

---

## 3. A panelek

`terv` = a képernyő fájlja a `design/06-panes/` alatt. `keskeny` = van-e kidolgozva a szűk állapot
(mérve: a `degrad`/`narrow`/`collapse` jelek száma a fájlban).

### Kérdés-panelek — a hatókör állítható

| # | panel | terv | test | keskeny | megjegyzés |
|---|---|---|---|---|---|
| 1 | **Resource list** | `01` | `DataTable` | **10** ✓ | a szerver `priority` mezője dönt — már beépítve (`A3`) |
| 2 | Events | `03` | `DataTable` | 3 ~ | ok szerint csoportosít, `47×` |
| 3 | Fleet | `04` | `DataTable` | **13** ✓ | egy verdikt-sor 30 px-en, alatta a tábla |
| 4 | Global search | `05` | `DataTable` | **10** ✓ | → a lista **flotta-hatóköre**, nem külön típus |
| 5 | Change stream | `06` | `DataTable` | 6 ✓ | a korrelációs mondat a lényeg |
| 6 | Resource monitor | `07` | `DataTable` | **1** ✗ | → `Capacity`, hatókör-választóval |
| 7 | Diagnostics | `08` | `DetailView` | **1** ✗ | → a `Details` szekciója |
| 8 | RBAC | `09` | `DataTable` | **0** ✗ | 8×N mátrix — keskenyen nincs kitalálva |
| 9 | Settings | `12` | vegyes | **0** ✗ | panel, nem dialógus — az érv jó |
| 10 | OpenShift | `18` | `DataTable` | 3 ~ | vezető mondat + operátorlista |
| 11 | Certificates | `19` | `DataTable` | **0** ✗ | → **preset** |
| 12 | Waste | `20` | `DataTable` | **0** ✗ | → `Capacity` rendezése; **piros nélkül** |
| 13 | Kubeconfig | `21` | `DataTable` | **0** ✗ | → a `Settings` szekciója, verdikt-listával |
| 14 | Watch rules | `22` | `DataTable` | 4 ~ | sárga a tiéd, piros az Alertmanageré, szürke a némított |

### Tárgy-panelek — a hatókör örökölt és rögzített

| # | panel | terv | test | keskeny | megjegyzés |
|---|---|---|---|---|---|
| 15 | **Details** | `10` | `DetailView` | **2** ✗ | „**három oszlop, nem hosszú görgetés**" — ez keskenyen összedől |
| 16 | Log | `02` | `LogView` | 3 ~ | a spike már bizonyította |
| 17 | YAML + form | `13` | `YamlView` + `DiffView` | **1** ✗ | egy panel két nézettel, élőben összekötve |
| 18 | Terminal | `14` | `TerminalView` | **0** ✗ | a keret **nem** nyúlhat a billentyűkhöz |
| 19 | Metrics | `15` | `MetricView` | **2** ✗ | a jelölők a termék, nem a görbék |
| 20 | Describe | `23` | `YamlView` | **0** ✗ | → a `Details` nézete |
| 21 | Node | `24` | `DetailView` | **0** ✗ | → `Capacity` node-hatókörrel |
| 22 | Helm | `17` | `DataTable` + `DiffView` | **1** ✗ | clusteronként egy oszlop — 10 clusternél keskenyen? |
| 23 | What would happen | `16` | `DiffView` | **1** ✗ | „kék = még nem igaz" — egy színszabály három panelre |
| 24 | Assistant | `11` | `DetailView` | **0** ✗ | saját szín, ami **más panelekbe nyúl** |

### Keret-képernyők — nem panelek

| terv | mi | keskeny |
|---|---|---|
| `25` Layouts | fülek, sablonok, mi marad meg | 1 |
| `26` First run | nincs cluster, nincs elrendezés | **0** |
| `27` Time travel | az egész felület fél órával ezelőtt — **a fül az óra** | **0** |
| `28` Cross-cluster | → a lista hatókör-módja, nem képernyő | 6 ✓ |
| `29` Cluster rail | három szélesség, **az egyik a nulla** | 7 ✓ |
| `30` Bulk action | nyolc objektum, három cluster, egy írás | **0** |
| `31` Incident | a négypaneles elrendezés végig, 14:02 → 14:31 | **0** |

---

## 4. Ami el van döntve, és nem kérdés többé

A `design/05-frame/04-Decisions.dc.html`-ből. **Ezeket nem szabad újratárgyalni építés közben.**

| # | szabály |
|---|---|
| 1 | **Fókusz** = fejléc-tint `rgba(63,224,208,.06)` + cián hajszálvonal a fejléc alján. **Nincs geometriaváltozás.** A gyűrű a panelen *belüli* vezérlőké marad |
| 2 | **Fejléc**: három gomb 17 px-en — `⊞ ⋯ ✕`. A szűrő a vezérlősáv mezője, a maximalizálás `⌘⏎` |
| 3 | **Elhelyezés-előnézet** = élő próba, króm-szinten, 45%-on; a cél-rekesz 100%-on; érvénytelen célon **befagy** |
| 4 | **Osztás**: az új panel 40%, küszöb **804 px**; alatta a `⊞` „nyisd fülként"-re vált, **soha nem szürkül** |
| 5 | **A sor 2 px bal éle a státuszé.** A kijelölés = kitöltés + **1 px keret a futam köré**, nem soronként |
| 6 | **Kijelölt sor**: 8% semleges soron; színezett soron **10%, kompozitálva, nem egymásra** + fehér szöveg. Fókuszálatlan panelen 4% / 40% |
| 7 | **Többes kijelölés van**: `⌘`/`⇧` katt, `⇧`+nyíl, `⌘A` a szűrt halmazon. A nyilak a kurzort mozgatják és **nem** jelölnek; a `Space` kapcsol |
| 8 | **A kijelölés soha nem fogy csendben**: a szűrő elrejtette sorok számolva, a halott sorok sírkőként maradnak, a művelet-sáv azt a számot írja ki, amit tényleg érintene |
| 9 | **Az osztóvonal soha nem színcsatorna.** Panel-keret 1 px `--ks-line`, fogó 3×20 px `--ks-line-strong`, mindkettő semleges |
| 10 | **Az osztásfogó nyugalomban is látszik**: 3×20 px a felezőponton. Nyugalom / rámutatás / húzás = egy tárgy három méretben (20 szürke → 30 cián → az egész él) |

### Amit a keret-króm degradációjáról tudunk

Általános szabály, **minden panelre érvényes** (a régi paneltervből, `brief/plans/`):

| sáv | mi megy, sorrendben | px |
|---|---|---|
| fejléc | a kapcsolat szava → a namespace → a cluster neve csonkul (8 karakterig) | 480 · 400 · 340 |
| fejléc | **a típusglif és a `⊞ ✕` soha** | — |
| vezérlősáv | a stream szava → a darabszám mértékegysége → a keresőmező ikonra → a csipeszek összecsuknak | 520 · 460 · 380 · 340 |

---

## 5. A nyitott hiány: a keskeny állapotok

**Ez a legnagyobb megmaradt kockázat.** A króm degradációja általános szabállyal meg van oldva; a
**testeké nincs**, és testenként más.

Mérés a 31 fájlon (`degrad` / `narrow` / `collapse` jelek):

```
kidolgozva (≥6)   01 04 05 06 28 29                            ██████ 6
érintve   (1–5)   02 03 07 08 10 13 15 16 17 18 22 25          ████████████ 12
semmi     (0)     09 11 12 14 19 20 21 23 24 26 27 30 31       █████████████ 13
```

**A 09–24 blokk — a legutóbb érkezett tizenöt panel — gyakorlatilag nincs keskenyen kidolgozva.**

### És a valódi szélességek nem 320 px

A 804 px-es osztásküszöbből következik, hogy `⊞`-vel 1600 px-ből ezek a szélességek érhetők el:

```
1600 · 958 · 638 · 572 · 382
```

A 320 px tehát **ritka** (csak kézi fogóhúzással), a **638 viszont a mindennapi** — és azt sem
rajzolta meg senki. A brief maga mondja ki, hogy ez a tét:

> „Négy panel 1280 pixelen ~300 pixeles paneleket jelent. **Ez nem szélsőséges eset, hanem a
> mindennapi**, és a mostani felület pont ezen bukik el."

### Három kategória, kockázat szerint

| kategória | panelek | mi történik keskenyen |
|---|---|---|
| **Magától szűkül** | Resource list, Events, Log, Certificates, Kubeconfig, Waste | oszlop esik ki `priority` szerint — a szabály megvan |
| **Át kell alakuljon** | Details (3 oszlop → ?), What-would-happen, Node, Metrics | az elrendezési premisszájuk hal meg, nem egy oszlop |
| **Nem tudjuk** | RBAC mátrix, Helm érték-rács, Assistant, Terminal | N oszlop N clusterre / N verbre — nincs rá szabály |

A második és harmadik kategória **tizenegy panel**, és mindegyikhez döntés kell, nem CSS.

---

## 6. Az építés sorrendje

| # | mit | miért ez | mit old fel |
|---|---|---|---|
| 1 | `DataTable` + `DataRow` + a kijelölés | 48 megnyitás; a panelek fele ezen ül | Resource list, Events, Fleet, Certificates, Waste, Kubeconfig, Watch rules, OpenShift |
| 2 | A **presetek**: oszlopkészlet + verdikt-szabály + vezető mondat | öt panel egy implementációval | Certificates, Kubeconfig, Waste, OpenShift, RBAC-lista |
| 3 | `DetailView` a három nézetével | 40 megnyitás | Details, Describe, Diagnostics |
| 4 | `LogView` + `Gutter` | a spike bizonyította | Log |
| 5 | `YamlView` (a `Gutter`-t újrahasználja) | | YAML, Describe |
| 6 | `DiffView` | ma négy helyen rajzolva, sehol komponensként | YAML-editor, Helm, Cross-cluster, What-would-happen |
| 7 | `MetricView`, `TerminalView` | külső könyvtárak (uPlot, xterm) | Metrics, Terminal |
| 8 | `Capacity` | három képernyő egy panelben | Resource monitor, Waste, Node |

**Az 1–3. a termék fele.** Amíg ez nincs meg, a többi nem számít.

---

## 7. Amit még el kell dönteni

Sorrendben, ahogy blokkolják az építést:

1. **A cluster-sáv alapértelmezett szélessége.** A terv 0-t javasol, plusz egy fülsáv-chipet („2 cluster
   kér téged"). Én 46 px-eset építettem. **Blokkolja:** a `Rail.svelte` és a fülsáv.
2. **A 638 px-es panel.** Mind a 31 képernyő 1600-on készült. **Blokkolja:** minden panelt, az
   elsőtől kezdve.
3. **A tizenegy panel, ami keskenyen átalakul** — külön kör kell rá, vagy panelenként döntés.
4. **A hat prózasor** a `brief/plans/` két fájljában megdőlt; javítani kell, hogy ne maradjon
   ellentmondó forrás.

---

## 8. Fájlnév-konvenciók

| hol | nyelv | minta |
|---|---|---|
| `brief/` | **magyar** | `NN-tema.md` |
| `design/` | **angol** | `NN-topic/`, benne `NN-Title.dc.html` |
| `docs/` (ez a repó) | magyar fájlnév, **angol kód és komment** | `kubescope-tema.md` |

Kód, komponensnév, CSS, teszt-üzenet: **mindig angol**.

---

## 9. A húsz primitív

`design/04-controls/04-Primitives.dc.html` — Svelte-nevekkel, minden állapot megrajzolva. **Minden
primitívhez tartozik egy `MUST NOT` sor**: az az egy mód, ahogy el szokták rontani. Ezeket
szó szerint idézem, mert pont ezek azok, amiket építés közben magamtól megsértenék.

| primitív | méret | mire jó |
|---|---|---|
| `Button` | 24 px | négy változat × hat állapot |
| `IconButton` | 17 / 24 px | a `⊞ ⋯ ✕` a fejlécben |
| `Field` | 24 / 22 px | szöveg, szám, keresés |
| `Select` | 24 px | a cluster- és hatókör-választók |
| `Toggle` · `Checkbox` | 14 / 12 px | logikai érték, két alakban |
| `Segmented` | 24 px | két–négy kizáró választás |
| `Slider` · `RangeSlider` | 20 px | egy érték, vagy kettő |
| `KeyCap` | 16 px | **minden** gyorsbillentyű a termékben |
| `StatusBadge` | 18 px | nyolc állapot, három csatorna |
| `Chip` | 18 px | mérések és elutasítások |
| `Meter` · `Progress` | 9 / 3 px | egy arány, és egy várakozás |
| `Sparkline` | 18 px | alak, nem leolvasás |
| `Spinner` · `Skeleton` | 11 px · sor | várakozás, kétféleképpen |
| `Tooltip` · `Toast` | 18 / 32 px | egy név, és egy nyugta |
| `Plot` | min 96 px | görbék + esemény-annotációk |
| `Bars` | soronként | kategória-összehasonlítás |
| `ScrollArea` | 10 / 8 px | **az egyetlen** görgetősáv |
| `Dialog` · `Sheet` | középre / élre | a két blokkoló felület |
| `CommandPalette` | 560 px | az egyetlen globális réteg |
| `Truncate` · `Mono` · `Timestamp` · `Rel` | — | négyféleképpen leírni egy értéket |

### A `MUST NOT` szabályok

Ezek a legértékesebb sorok az egész tervben. Sorrendben, a primitívek szerint:

- **`Button`** — soha ne legyen az egyetlen kapu egy destruktív művelethez. A destruktív változat
  *mindig* `ConfirmDialog`-hoz vezet; a szín figyelmeztetés, nem védelem.
- **`IconButton`** — soha ne szállítson `label` prop nélkül. A tooltip és az akadálymentes név
  onnan jön, és **öt névtelen glif a fejlécben pont az, amit ez a kör lecserél**.
- **`Field`** — soha ne validáljon gépelés közben egy szűrőben. A szűrő élőben alkalmazódik, és egy
  félig beírt kifejezés nem hiba.
- **`Select`** — soha ne natív `<select>`. Nem tud per-opció latenciát és státuszoszlopot vinni, és
  ez lenne az egyetlen OS-rajzolta felület az appban.
- **`Toggle`** — soha ne olyasmire, ami megerősítést kíván. **Mire elolvasod, már megtörtént.**
- **`Segmented`** — soha ne négynél több szegmens. Ötnél a címkék csonkulnak, és rosszabb `Select`
  lesz belőle, keresés nélkül.
- **`Slider`** — soha ne olyan értékre, aminek pontosnak kell lennie. A memórialimit `Field`.
- **`KeyCap`** — soha ne írjuk ki kézzel szövegként. Egy címkébe beírt gyorsbillentyű az első
  változtatás után ellent fog mondani a billentyűtérképnek.
- **`StatusBadge`** — soha ne kapjon színt közvetlenül. **Fázist képez le**; ha a hívó választ színt,
  ugyanaz a fázis két panelben két szín lesz.
- **`Chip`** — soha ne ússzon be. Egy csipesz, ami húzás közben felúszik, azután érkezik, mint a
  döntés, amit segíteni akart.
- **`Meter`** — soha ne írjon százalékot az abszolút érték nélkül. „98%" egy ki nem mondott
  összegből a leggyakoribb módja annak, hogy egy dashboard félrevezessen.
- **`Sparkline`** — soha ne rajzoljon nullvonalat hiányzó adatra. A lapos vonal azt állítja, hogy
  nem történik semmi — ez az ellentéte a „nem tudjuk"-nak.
- **`Spinner`** — soha ne fedjen le egy egész panelt. A töltő panel a krómját mutatja és a testét
  csontvázazza.
- **`Toast`** — soha ne *csak* toastként jelentsen hibát. A sikertelen írás megjelöli a panelt és az
  érintett sorokat is, mert egy elhalványuló toast magával viszi az egyetlen bizonyítékot.
- **`Plot`** — soha ne simítson és ne interpoláljon. **Az adathiány rés, nem görbe.**
- **`Bars`** — soha ne rajzoljon nulla hosszú sávot hiányzó mérésre. Az „off" vagy a „nem mérhető"
  **szó**, mert a 0 px-es sáv nullának olvasódik.
- **`ScrollArea`** — soha ne cseréljük saját görgetésre. Elveszik a trackpad-lendület és a Page Up,
  és ez billentyűzet-első termék.
- **`Dialog`** — soha ne olvasható tartalomra. **Az olvasás soha nem blokkol**: a részlet, a diff és
  a manifest panel, hogy kettő nyitva lehessen egyszerre.
- **`CommandPalette`** — soha ne mutasson műveletet a hatókör-címkéje nélkül. Egy címkézetlen
  flotta-művelet a panelműveletek listájában az, ahogy valaki `Enter`-rel újraindít nyolc clustert.
- **`Truncate`** — soha ne CSS-ellipszis egy névre. A `text-overflow` csak a **végét** tudja levágni,
  és pont az a hat karakter azonosítja a podot. *(Ez a `format/truncate.ts` `middle()`-je — már így van.)*

---

## 10. A mappák, és mi lakik bennük

`design-workspace/kubescope/design/`, olvasási sorrendben:

| mappa | mi | miért kell nekem |
|---|---|---|
| `01-directions/` | A · Instrument (választott), B · Ledger | a vizuális rendszer forrása |
| `02-prototype/` | ugyanaz kattinthatóan | a mozgás, amit a statikus lapok nem mutatnak |
| `03-action-map/` | 316 művelet · szabályok · nyolc kérdés · **overlay-grammatika** | mit nyit meg mi, és milyen réteg |
| `04-controls/` | `01-Controls` `02-Dropdowns` `03-Parts` **`04-Primitives`** | a widgetek — a 9. szakasz ebből van |
| `05-frame/` | fogantyúk · húzás · kijelölés · **döntések** · **komponensfa** · régiótérkép | a keret; a 4. szakasz ebből van |
| `06-panes/` | 31 képernyő | a 3. szakasz |
| `07-overlays/` | paletta+súgó, megerősítés+idő | a négy réteg |
| `08-icon/` | app- és tálcaikonok, öt méret | a Tauri bundle-höz |

**Amit ezen felül tudni kell:**

- Az **overlay-grammatika** (`03-action-map/04-Overlay-Grammar`) nyolc réteg-fajtát nevez meg: menü,
  hover-kártya, toast, popover, megerősítés, paletta, sheet, teljes ablak. A `07-overlays/` ebből
  **kettőt** rajzol meg részletesen — a maradék hat a grammatikából épül.
- A **megerősítésnek három foka van** (`07-overlays/02`): „csak csináld" · „mutasd a hatását" ·
  „írd be a nevét". És **hat jel** mondja meg, hogy a múltban vagy, nem egy banner.
- A **paletta és a súgó egy registry, két réteg** — a jobbklikk-menü ugyanannak a listának a
  kontextusra szűrt kivágata, nem külön kódút.

---

## 11. Amit kidobtunk, és miért — 2026-09-02

Az egui-korszak maradványai zavartak, nem segítettek. Ami ment:

| mi | méret | miért |
|---|---|---|
| `legacy/` | 9,9 MB | a teljes egui-felület. A README kilenc kibányászandó tételéből **nyolc már át van emelve** (`format/`, `frame/tree.ts`, `frame/workspace.ts`, `theme.css`), a kilencedik — a shell nem interaktív `2>/dev/null` mellett — pedig **már a `kubescope-core/src/exec.rs`-ben áll**, doc-kommenttel |
| `spike/` | 3,7 GB | eldobható mérés volt; a kérdést megválaszolta, az eredménye a `Core::logs_tail` |
| `kubescope-core/src/pod.rs` | 733 sor | pod-specifikus watch és sor-modell. Az `A1–A6` munka egész lényege, hogy **a pod nem különleges** — a `resource.rs` + `table.rs` bármelyik típust viszi |
| `kubescope-core/src/node.rs` | 224 sor | ugyanaz node-ra |
| `kubescope-core/src/watch.rs` | 362 sor | a pod/node-specifikus watcher. A benne lakó **`Stream`** generikus volt — külön modulba került (`stream.rs`), a `connect` pedig a `cluster.rs`-be, ahová tartozik |
| `pods` · `watch_pods` parancsok | — | a híd pod-specifikus ága. A felület már csak `table`/`watch_tables`-t hívott |
| `Pod` · `PodList` · `PodSubject` (TS) | — | ugyanaz a másik oldalon. A két `Subject` típus **eggyé olvadt** |
| `docs/KubeScope Design System v2.html` | 649 KB | az egui-korszak design systemje |
| `docs/kubescope-handoff-m1.md` | — | az egui M-1 átadó |
| `docs/*-design.dc.html` | 800 KB | a két régi terv — a design repóban él tovább (`brief/plans/`), ahol a hat megdőlt prózasora is meg van jelölve |
| `~/.claude/plans/docs-groovy-orbit.md` | 20 KB | **az egui M-1 terv, amit minden munkamenet elején a kontextusomba töltött.** Ez volt a legzavaróbb: egy `egui 0.36.1`-es widget-gallery építését írta le |

**A mag 3733 sorra fogyott**, és nincs benne két út ugyanarra.

### És a tokenek

A `--k-*` prefix a régi design systemből maradt, a terv `--ks-*`-ot használ. Ez fordítási réteg volt
minden panel átemelésénél — pont az a hely, ahol eddig kétszer elcsúsztam. **Átneveztem az egészet
`--ks-*`-ra**, hogy a `.dc.html`-ből másolt érték szó szerint működjön.

A paletta egyébként **egyezett** — `#0a0c10`, `#11141a`, `#242b35`, `#e7ecf3`, `#3ecf8e`, `#f0b429`,
`#4aa8ff`, `#1d2937` mind ugyanaz. Ami hiányzott:

- **`--ks-cy: #3fe0d0`** — az akcentus. Ez a fókusz, a kijelölés, az elsődleges gomb, a paletta
  színe, és **nem státusz**. Nálam a fókusz `--ks-info` (kék) volt, a terv szerint ciánnak kell
  lennie — javítva, a döntött szabállyal együtt (fejléc-tint + hajszálvonal, **semmi geometria**).
- `--ks-line-strong` (az osztásfogó fogója), `--ks-over` + `--ks-over-line` + három árnyék (lebegő
  felületek), `--ks-ask` (asszisztens), `--ks-past` (időutazás), `--ks-term` (a shell saját talaja),
  az öt művelet-család színe, és a szintaxis-hármas.
