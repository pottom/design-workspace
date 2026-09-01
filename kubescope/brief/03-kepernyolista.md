# KubeScope — mit fog tudni, és hol jelenik meg

> Kísérő a képernyő-briefhez. A `01-kepesseg-scope.md` a **szerződés** — mit tud majd az alkalmazás.
> Ez a dokumentum ugyanazt **képernyőnként** rendezi el: melyik paneltípus létezik, mi van benne, és
> melyik képesség hol jelenik meg. A tervezéshez ez a hasznosabb sorrend.

---

## 1. A mérce: k9s, és ami fölötte van

**A minimum, hogy mindent tudjon, amit a k9s.** Az a napi eszköz, amit a célfelhasználó ma használ,
és amitől el kell venni a helyét. Ha valami hiányzik belőle, az alkalmazás nem lesz elég jó ahhoz,
hogy váltson.

### 1.1 k9s-paritás — a lista, tételesen

| amit a k9s tud | nálunk |
|---|---|
| Bármely erőforrástípus listázása, rövidítéssel (`po`, `deploy`, `svc`) | `RES-02`, `APP-09` |
| CRD-k listázása, oszlopok a definícióból | `RES-03` |
| Namespace-váltás, context-váltás | `RES-01`, `CONN-01` |
| Élő, magától frissülő táblák | `RES-02` + a keresztmetszeti követelmények |
| Oszlop szerinti rendezés | `RES-02` |
| **Gyors szűrés gépelve, tagadással** | `RES-12` |
| Címke- és mezőszelektor | `RES-07` |
| **Describe** | `RES-10` |
| YAML megtekintés | `RES-06` |
| YAML szerkesztés és alkalmazás | `ACT-02`, `ACT-03` |
| *(a k9s ennyit tud — mi űrlapot is adunk hozzá)* | `ACT-12` |
| Napló, követéssel | `LOG-01` |
| Előző példány naplója | `LOG-05` |
| **Utolsó N sor / X perc** | `LOG-08` |
| Shell a konténerbe | `ACT-05` |
| **Attach a futó folyamathoz** | `ACT-09` |
| Port-forward, és a futók listája | `ACT-04` |
| Törlés, skálázás, rolling restart | `ACT-01` |
| **Rollout-történet és visszaállítás** | `ACT-10` |
| CronJob indítás / felfüggesztés | `ACT-01` |
| Node cordon / uncordon / drain | `ACT-06` |
| **Node debug shell** | `ACT-11` |
| **Többes kijelölés, tömeges művelet** | `RES-11` |
| Erőforrás-használat oszlopok (metrics-server) | `MET-08` |
| Tulajdonosi fa (a k9s XRay-je) | `RES-05` |
| Elárvult / gyanús erőforrások (a k9s Popeye-a) | `DIAG-06`, `DIAG-07` |
| Cluster-aktivitás áttekintés (a k9s Pulses-e) | `XC-02` |
| RBAC-nézetek | `SEC-01`, `SEC-02` |
| Secret dekódolás | `RES-09` |
| Saját oszlopok, mentett nézetek | `RES-08` |
| **Másolás vágólapra** | `RES-13` |
| Parancssor / paletta | `APP-01` |
| **Súgó: a teljes billentyűtérkép** | `APP-08` |
| **Saját parancsok billentyűhöz kötve** (pluginek) | `APP-10` |
| **Visszalépés a fúrás útvonalán** | `APP-11` |
| Témák, sűrűség | `APP-05` |

**Amit szándékosan nem veszünk át:** a beépített terheléstesztet (`benchmark`). Az egy másik eszköz
feladata, és semmi köze ahhoz, amiért ezt megnyitod.

### 1.2 Amit a k9s nem tud, és ettől lesz értelme

Öt dolog, és mindegyik a **több cluster** vagy az **idő** tengelyén nyer:

**Flotta.** Nyolc cluster egészsége egy képernyőn (`XC-02`). Globális keresés mindben egyszerre —
„hol van `CrashLoopBackOff`?" (`XC-01`). Ugyanaz a lista összevontan, „8 cluster / 2 eltér" jelzéssel
(`XC-08`). Verzió-mátrix (`XC-07`). Egy művelet N clusteren, egyetlen megerősítéssel (`XC-06`).

**Összefésült naplók.** Egy Deployment összes podja egyben, a rolling update alatt is szakadás
nélkül (`LOG-09`) — és ugyanez nyolc clusterből, időbélyeg szerint egymásba fűzve, forrásonként
színezve (`XC-05`, `LOG-02`). A JSON-sorok mezőkre bontva, választható mezőkkel oszlopként
(`LOG-04`).

**Helm, flottaszinten.** Mi van telepítve, milyen verzióban, melyik clusteren — és **hol tér el a
values** (`HELM-06`, `HELM-07`). A release-ek Secretként a Kubernetes API-ban vannak, tehát ehhez
`helm` bináris sem kell.

**Konfigdrift.** Mezőszintű diff két cluster között (`XC-03`), és folyamatos figyelés arra, hogy
elcsúsztak-e (`XC-04`).

**Metrika és esemény egy görbén.** A restart, az OOMKill, az ütemezés és a deploy **rá van ültetve** a
CPU- és memóriagörbére (`MET-04`). Ez az a nézet, ahol az „mi történt 14:02-kor?" kérdés egy
pillantás alatt megválaszolható.

**Time travel.** Az egész felület visszaállítható egy múltbeli pillanatra (`TIME-02`), a
változástörténet mezőszintű (`TIME-03`), az események túlélik a Kubernetes egyórás ablakát
(`TIME-04`), és egy incidens időablaka **exportálható egy fájlba** (`TIME-05`).

**A rövid hurok.** Három olyan pár, ahol ma három külön eszköz kell, és itt egy mozdulat lesz:
a monitor megmutatja, hogy 80 MB-ot használ 512 MB limittel — és **ugyanott át is írod** (`MET-12`
→ `ACT-13`); a diagnosztika megmondja, hogy egy liveness probe öli a podot — és **ugyanott
megjavítod** (`DIAG-02` → `ACT-14`); a drift megmutatja, hol tér el a values — és **ugyanott
visszaállítod** (`HELM-07` → `HELM-05`).

**Plusz a diagnosztika**, ami ma emberi fejben van: miért Pending ez a pod, node-onként lebontva
(`DIAG-01`); miért nem áll készen (`DIAG-02`); mennyivel van túlfoglalva (`DIAG-03`); mikor jár le a
tanúsítvány, flottaszinten (`SEC-03`).

---

## 2. A paneltípusok — a tervezés valódi egysége

Az alkalmazás **tiling**: munkaterületek, azokban osztott panelek, panelenként saját hatókör. Nincs
„képernyő" a szó hagyományos értelmében — **van panel-fajta**, és egy képernyő ezekből áll össze.

**Két család**, és ez dönti el a panel fejlécét: a **kérdés-panel** hatóköre átállítható (más
clusterről ugyanaz a kérdés), a **tárgy-panelé** örökölt és rögzített (egy megnevezett objektum egy
clusterben).

### 2.1 Kérdés-panelek

| panel | mit mutat | ami benne kell legyen |
|---|---|---|
| **Erőforrás-lista** | bármely típus sorai | oszlopok típusonként, rendezés, gyors szűrés, szelektorok, többes kijelölés, használat-oszlopok |
| **Események** | idősoros eseményfolyam | súly, ok, üzenet, érintett objektum, ismétlésszám; véges megőrzés, ezért ablak és nem darabszám |
| **Flotta** | minden cluster egészsége | node-ok, hibás podok, verzió, kapacitás, kapcsolat — clusterenként egy sor |
| **Kubeconfig** | mit ismerünk egyáltalán | contextek, clusterek, felhasználók — szerkeszthetően, kapcsolatpróbával mentés előtt, és kiírva, melyik fájlba ír |
| **Globális keresés** | egy kérdés N clusterben | találatok cluster szerint csoportosítva, részleges eredménnyel |
| **Diagnosztika** | egy kérdés, kifejtve | „miért Pending", „mi nem áll készen", rightsizing, elárvult erőforrások |
| **RBAC** | ki mit tud, és mit tudna | jogosultság visszavezetve a Role/Binding forrásra, mindkét irányban; **szerkesztés mátrixként** — API-csoport × erőforrás × ige, pipákkal —, és **mit engedélyez, mielőtt engedélyezed**, az eszkaláló jogokat kiemelve |
| **Tanúsítványok** | mi jár le és mikor | lejárati sorrend, flottaszinten |
| **Helm-release-ek** | mi van telepítve | chart- és app-verzió, revízió, státusz; flottaszinten ugyanaz a release N clusteren, az eltérés kiemelve |
| **Metrika-böngésző** | milyen metrikák léteznek | metrikák, címkéik és értékeik — ebből épül a lekérdezés |
| **Erőforrás-monitor** | ki mennyit eszik most | pod / konténer / node / namespace szerint, rendezhetően; a requests és limits a valós használat mellett, sparkline-nal |
| **Változásfolyam** | mi változott, és mikor | deploy, image-csere, ConfigMap, skálázás, RBAC — időrendben, cluster- és flottaszinten. **Ez lesz a leggyakrabban megnyitott panel**, mert ez az első kérdés, amikor elromlik valami |
| **Pazarlás** | mennyit foglalunk feleslegesen | namespace, cluster és flotta szinten: foglalt kontra használt |
| **Figyelőszabályok** | mire szóljon | saját szabályok a futó streamek felett, és a bekövetkezett riasztások listája |

### 2.2 Tárgy-panelek

| panel | mit mutat | ami benne kell legyen |
|---|---|---|
| **Részletek** | egy objektum | Status, Spec, feltételek, események, tulajdonosi lánc, kapcsolódó objektumok |
| **Describe** | ugyanaz, `kubectl` módjára | az olvasható összefoglaló — más kérdésre válaszol, mint a YAML |
| **YAML** | a tárolt objektum | kiemelés, összecsukás, keresés, managed fields elrejtve, szerkesztés diff-előnézettel |
| **Napló** | egy konténer **vagy egy egész workload** folyama | követés, szünet, keresés és kiemelés, előző példány, export. Két dolog, ami nem opcionális: **workload-szintű követés** (a Deployment összes podja egyben, a rolling update alatt is szakadás nélkül), és a **JSON-sorok mezőkre bontása**, választható mezőkkel oszlopként |
| **Terminál** | egy konténer shellje | `exec`, `attach`, node debug — és az, hogy elveszi a billentyűzetet |
| **Metrika** | egy objektum görbéi | CPU, memória, hálózat, restart — **eseményekkel a görbén** |
| **Diff** | ugyanaz két helyről | mezőszintű eltérés két cluster, két időpont vagy két Helm-revízió között — és **a saját múltjához**: „mi változott azóta, hogy jó volt?" |
| **Hatáskör-előnézet** | mit fog elrontani | egy írás előtt: mely Service-ek maradnak endpoint nélkül, mely PDB-ket sértené, mely útvonalak szűnnének meg. És fordítva: **miért akadt el egy eviction** |
| **Drain-szimuláció** | hova kerülnek a podok | ha lehúzod ezt a node-ot: hova ütemeződnek, és beférnek-e — requests, taintek, affinitások szerint |
| **Helm-release** | egy telepítés | values (megadott és számított), renderelt manifeszt, notes, revízió-történet, a hozzá tartozó erőforrások |
| **Lekérdezés-összerakó** | egy PromQL, épülés közben | metrika, címkeszűrők, aggregáció, ablak — és **a generált lekérdezés végig látszik**, szerkeszthetően |
| **Szerkesztő** | egy objektum, űrlapon | ConfigMap és Secret kulcs–érték párokként, Ingress és Route szabályonként, Deployment replikákkal és image-dzsel; **requests és limits a valós használattal egymás mellett**; liveness, readiness és startup probe; környezeti változók, feloldva mutatva, honnan jönnek — **és alkalmazás előtt mindig a YAML-diff** |

### 2.3 OpenShiftre külön

Nem külön alkalmazás és nem külön mód: **ugyanazok a panelfajták, más típusokkal.** A Route egy
erőforrás-lista, a Build egy tárgy-panel naplóval, a `ClusterVersion` egy részletnézet. Három hely
viszont saját nézetet kíván, mert saját kérdésre válaszol:

| panel | mit mutat |
|---|---|
| **Cluster-frissítés** | melyik csatorna, mi elérhető, hol tart az upgrade, **melyik ClusterOperator akasztotta meg** — és a MachineConfigPoolok: hány node kész, melyik ragadt be |
| **Operátorok** | CSV-k és Subscriptionök állapota, és a **jóváhagyásra váró InstallPlanek** — mert egy jóváhagyatlan InstallPlan csendben megállít egy operátort |
| **Kvóták** | `ResourceQuota`, `LimitRange`, `ClusterResourceQuota` — mennyi fogyott és ki fogyasztotta. Egy elfogyott kvóta Pendingben álló podként jelentkezik, tehát ide vezet a „miért Pending" egyik ága |

### 2.4 Ami nem panel

**A parancspaletta** (`APP-01`) — mindenhonnan elérhető réteg, nem panel.
**A megerősítő párbeszéd** (`ACT-07`) — a destruktív műveletek előtt, prod clusteren szigorúbb.
**Az időcsúszka** (`TIME-02`) — a *teljes felületre* hat, tehát a kereté, nem egy panelé.
**A súgó** (`APP-08`) — átfedő réteg, ami megmondja, mit tudsz itt és most.

---

## 3. Amit a tervnek meg kell oldania, és ma nincs megoldva

Ezek nem képernyők, hanem **kérdések**, amikre a terv kell hogy válaszoljon:

1. **Hogyan jön létre egy tárgy-panel egy sorból?** Egy pod sorára állva a napló, a YAML, a
   describe és a terminál mind értelmes következő lépés. Jobbklikk, billentyű, mindkettő — és hova
   kerül az új panel?

2. **Hogyan néz ki a többes kijelölés?** Nyolc pod megjelölve három panelben — mi jelzi, hogy
   „nyolc dolgot fogsz törölni", és hol.

3. **Hol lakik az idő?** A `TIME-02` az egész felületet visszaállítja. Ez a keret sávja, egy
   overlay, vagy valami harmadik — és **miből látszik ránézésre, hogy amit látsz, az nem a jelen**?
   Ez a legkockázatosabb elem az egész termékben: egy felület, ami múltbeli adatot mutat anélkül,
   hogy ez kiabálna, veszélyes.

4. **Hogyan mutat egy panel részleges eredményt?** Tíz clusterből kettő nem válaszol. A lista
   megjelenik, de valamiből látszania kell, hogy hiányos — és melyik forrás hiányzik.

5. **Mi történik, ha egy panel adata megállt?** A sorok igazak, csak régiek. Ez nem hibaállapot, és
   nem is normális — a kettő között van, és ma nincs rá vizuális nyelv.

6. **Hogyan fér el nyolc cluster egy sorban?** A flotta-panel a termék névjegye. Ha az nem szép és
   nem olvasható, az egész ígéret üres.

7. **Hogyan néz ki egy „mit fogsz elrontani" figyelmeztetés?** Nem hibaüzenet, mert még nem történt
   semmi — előrejelzés, amit el is lehet fogadni. Ez a hangnem ma nincs meg a készletben, és ez a
   különbség egy megerősítő ablak meg egy valódi biztonsági háló között.

8. **Miből látszik, hogy amit nézel, nem teljes?** Tíz clusterből kettő nem válaszol, vagy az adat
   négy perces. Ez nem hiba és nem is rendben — a kettő között van, és minden panelnek tudnia kell
   kimondani.
