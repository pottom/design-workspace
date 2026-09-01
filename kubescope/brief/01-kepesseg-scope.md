# KubeScope — Képesség-scope (v0.1)

> Munkacím. Ez a dokumentum a tervezés első fázisának eredménye: **mit fog tudni** az alkalmazás.
> Bemenete a design fázisnak (képernyők, navigáció) és az implementációs fázisnak (feature-enkénti bontás).
> A feature-azonosítók (pl. `RES-03`) stabilak — a design és a kód is ezekre hivatkozzon.

---

## 1. Cél

Egyetlen natív desktop alkalmazás, amely **több Kubernetes és OpenShift clustert** kezel egyszerre, és
olyan kérdésekre ad választ, amikre a meglévő eszközök (k9s, Lens, OpenShift Console, Grafana) külön-külön
vagy egyáltalán nem: **flottaszintű rálátás**, **időbeli visszatekintés** és **metrika–esemény korreláció**.

**Egymondatos pozicionálás:** *k9s sebesség, Lens-nél kisebb erőforrásigény, plusz olyan cross-cluster és
incidens-utólagos elemzési képességek, amik ma sehol nincsenek egyben.*

**A három megkülönböztető ék** (ezek adják a termék létjogosultságát, minden más ezek köré épül):

| Ék | Mit old meg |
|---|---|
| **Flotta-nézet** | Egy kérdés, N cluster. Keresés, diff, merged log, tömeges művelet. |
| **Time travel** | Lokálisan perzisztált cluster-állapot → „hogy nézett ki 3 órája?". A k8s Eventek 1 óra után elvesznek. |
| **Korrelált metrika** | Prometheus-görbe, ráültetve a pod életciklus-eseményekkel (restart, OOMKill, scheduling, image pull). |

## 2. Célfelhasználó

Platform engineer / SRE / DevOps, aki **3–20 clustert** felügyel vegyes környezetben (vanilla k8s + OpenShift),
és incidens közben vagy kapacitástervezéskor gyors, összesített képet akar. Terminál-közeli felhasználó:
a sűrű információt preferálja a látványos animációk helyett.

## 3. Nem célok

Ezek tudatosan kimaradnak — a scope védelme miatt fontos kimondani őket.

- **Nem** cluster-provisioning eszköz (nincs cluster létrehozás, node-pool kezelés, felhőszolgáltató API).
- **Nem** CI/CD vagy GitOps eszköz (nem helyettesíti az ArgoCD/Flux felületét, csak *olvassa* az állapotukat).
- **Nem** hosszú távú metrika-tároló (a Prometheus/Thanos marad a forrás, mi kérdezzük).
- **Nem** többfelhasználós/szerver oldali termék v1-ben (lokális desktop app, lokális adattárolással).
- **Nem** teljes YAML IDE (szerkesztés van, de nem versenyzünk a VS Code-dal).
- **Nem** mobil és nem böngésző-alapú (bár a WASM build későbbi opció maradhat).
- ~~**Nincs** beépített terminál emulátor v1-ben (lásd `ACT-05` — külső terminálba delegálunk).~~
  **Visszavonva (2026-08-30).** A beépített terminál bekerül a v1-be — lásd `ACT-05` és
  `kubescope-plan-m2.md` §0. A node-shell ezzel privilegizált műveletté teszi a terméket, ezért a
  `CONN-08` csak-olvasás kapcsoló, az `ACT-07` megerősítés és az `SEC-06` audit kötelező mögötte.

## 4. Technológiai keret

| Réteg | Választás | Megjegyzés |
|---|---|---|
| UI | `egui` / `eframe` | immediate mode, jól illik a folyamatosan frissülő cluster-cache-hez |
| Táblázatok | `egui_extras::TableBuilder` | virtualizált sorok, rendezhető/átméretezhető oszlopok |
| Grafikonok | `egui_plot` | metrikák és idővonal |
| K8s kliens | `kube-rs` (`watcher` + `reflector` + `Store`) | a `Store` a UI olvasási forrása |
| Async | `tokio` | háttér-runtime, a UI loop nem blokkolódik |
| Perzisztencia | SQLite (`rusqlite`) | time travel eseménynapló, mentett lekérdezések, beállítások |
| Prometheus | HTTP query API (`reqwest`) | instant + range query |

### 4.1 Workspace-felosztás

A felület teljesen egyedi, custom widgetekre épül. Ezeket külön crate-ben fejlesztjük, **nulla
Kubernetes-függőséggel** — így önállóan futtathatók, gyorsan fordulnak, és később más projektben is
újrahasznosíthatók.

```
kubescope-ui/       widget crate + téma — se tokio, se kube-rs
kubescope-gallery/  bin: widget kirakat, hamis adattal
kubescope-core/     k8s, prometheus, sqlite
kubescope/          bin: az alkalmazás
```

**Kötelező szabály a nulladik naptól:** a téma nem konstansokban él, hanem egy `Theme` structban, amit a
widgetek paraméterként vagy az egui `Memory::data` typemapjén keresztül kapnak meg. Ez az egyetlen olyan
döntés, amit utólag behozni drága, mert minden widget belsejét érinti — minden más általánosítás ráér.

A widgetek nem k8s típusokat fogadnak, hanem saját view-model structokat (pl. `PodRow`, `TimelineEvent`).
Ez teszi lehetővé, hogy a gallery hamis adattal működjön, és hogy ugyanaz a widget később élő watchból,
time travel visszajátszásból vagy merged logból is kaphasson adatot.

> A library-vé alakítás (generikus primitívek szétválasztása, publikálás, API-stabilitás) **nem v1 cél**.
> A fenti felosztás megtartja a lehetőséget, de nem kötelezzük el magunkat mellette.

---

## 5. Navigációs modell (eldöntve)

**Cluster-központú navigáció, hatókör-választóval.** A felhasználó először hatókört választ, és minden
további nézet ebben él.

Az „All" **nem külön mód, hanem egy cluster-csoport**, aminek a tagsága minden engedélyezett cluster.
Egyetlen cluster is csak egy egyelemű csoport. Így nincs két kódút, és a hatókör-választó egységes:

```
[ prod-eu ▾ ]   →   All  |  prod-eu  |  staging  |  aks-dev  |  ocp-prod  | …
```

Következmények, amiket minden nézetnek tiszteletben kell tartania:

- **Cluster oszlop**: csoport-hatókörben rögzített első oszlopként jelenik meg, egy cluster esetén elrejtve.
- **Namespace név szerint illeszkedik**: `All` + `payments` mind a nyolc cluster `payments` namespace-ét
  mutatja. Ez szándékos — ez teszi egy szűréssé a leggyakoribb kérdést.
- **Csoport-hatókörben kizárólag metadata-only watch.** Teljes objektumot csak a részletnézet nyit, és csak
  az adott clusteren. Ez szabály, nem optimalizáció.
- **Részleges eredmény látható**: ha 10-ből 2 cluster elérhetetlen, a lista megjelenik, de a fejléc jelzi,
  hogy hány forrásból. Csendben hiányos lista incidens közben veszélyes.
- **Csoport-hatókörben minden írási művelet flotta-művelet** (`XC-06`): tételes előzetes felsorolás arról,
  mire fog hatni, és külön jóváhagyás.

---

## 6. Képességtérkép

### A. Kapcsolat- és cluster-kezelés — `CONN`

| ID | Képesség |
|---|---|
| `CONN-01` | Kubeconfig beolvasása, több context felismerése, context-enkénti engedélyezés |
| `CONN-02` | Cluster-csoportok (pl. `prod`, `staging`) — a flotta-műveletek hatóköre |
| `CONN-03` | Kapcsolat-állapot jelzés clusterenként (elérhető / hitelesítési hiba / lassú / offline) |
| `CONN-04` | Exec-credential plugin támogatás (`aws eks get-token`, `gke-gcloud-auth-plugin`, `oidc-login`) |
| `CONN-05` | OpenShift OAuth token flow (nem sima kubeconfig — külön hitelesítési út) |
| `CONN-06` | Cluster-képesség felderítés: API discovery, CRD-lista, „ez OpenShift?", verzió |
| `CONN-07` | Proxy / bastion / SOCKS támogatás elzárt clusterekhez |
| `CONN-08` | Csak-olvasható mód clusterenként kikényszeríthető (véletlen prod-módosítás ellen) |

### B. Erőforrás-böngészés — `RES`

| ID | Képesség |
|---|---|
| `RES-01` | Namespace-navigáció, kedvencek, legutóbb használtak |
| `RES-02` | Beépített nézetek a core erőforrásokra (Pod, Deployment, StatefulSet, DaemonSet, Job, CronJob, Service, Ingress, ConfigMap, Secret, PVC, Node, Event…) |
| `RES-03` | Általános CRD-nézet: bármely CRD listázása, oszlopok az `additionalPrinterColumns` alapján |
| `RES-04` | Erőforrás-részletnézet: Status, Spec, Events, Owner-lánc, kapcsolódó objektumok |
| `RES-05` | Tulajdonosi fa (Deployment → ReplicaSet → Pod) és fordított navigáció |
| `RES-06` | YAML megtekintés szintaxiskiemeléssel, managed fields elrejtése |
| `RES-07` | Címke- és mezőszelektor szűrés, mentett szűrők |
| `RES-08` | Oszlopok testreszabása, mentett nézetek |
| `RES-09` | Secret értékek dekódolása explicit, naplózott felfedéssel (alapból maszkolt) |

### C. Cross-cluster / flotta — `XC`

| ID | Képesség |
|---|---|
| `XC-01` | Globális keresés N clusterben egyszerre (név, címke, típus szerint) — pl. minden `CrashLoopBackOff` pod |
| `XC-02` | Flotta-áttekintő: clusterenkénti egészségállapot egy képernyőn (node-ok, hibás podok, verzió, kapacitás) |
| `XC-03` | Erőforrás-diff két cluster között, mezőszintű kiemeléssel (konfigdrift-vadászat) |
| `XC-04` | Drift-figyelő: kijelölt erőforrás-halmaz eltéréseinek folyamatos követése környezetek között |
| `XC-05` | Merged log tail: ugyanaz az app N clusterből, timestamp szerint összefésülve |
| `XC-06` | Flotta-művelet: ugyanaz az akció N clusteren (pl. rolling restart), egyetlen megerősítéssel |
| `XC-07` | Verzió-mátrix: k8s/OCP verziók, operátorok, image tag-ek clusterenként egymás mellett |
| `XC-08` | Listanézet két megjelenítése csoport-hatókörben: **lapos** (clusterenként egy sor) és **összevont** (névre csoportosítva, „8 cluster / 2 eltér" jelzéssel). Az összevont sor a `XC-03` diff belépési pontja |

### D. Logok — `LOG`

| ID | Képesség |
|---|---|
| `LOG-01` | Élő log stream, virtualizált megjelenítés (nagy scrollback mellett is stabil FPS) |
| `LOG-02` | Több pod / több konténer egyidejű követése, forrásonkénti színkód |
| `LOG-03` | Szűrés és kiemelés (szöveg + regex), negatív szűrő |
| `LOG-04` | Strukturált (JSON) log felismerése, mezőkre bontása, mező szerinti szűrés |
| `LOG-05` | Előző konténer-példány logja (`--previous`) crash után |
| `LOG-06` | Szövegkijelölés, másolás, fájlba exportálás |
| `LOG-07` | Szünet / folytatás, időbélyeg-ugrás, „ugrás az esemény időpontjához" (kapocs a `TIME` felé) |

### E. Prometheus és metrikák — `MET`

| ID | Képesség |
|---|---|
| `MET-01` | Prometheus/Thanos endpoint konfigurálása clusterenként, automatikus felderítés OpenShiftben |
| `MET-02` | Beépített metrika-panelek podra, deploymentre, node-ra (CPU, memória, hálózat, restart) |
| `MET-03` | Ad-hoc PromQL szerkesztő, eredmény grafikonon és táblázatban |
| `MET-04` | **Esemény-annotáció a grafikonon**: restart, OOMKill, scheduling, image pull, deploy — a görbére ültetve |
| `MET-05` | Egyidejű időablak több panel között (egy helyen zoomolsz, minden együtt mozog) |
| `MET-06` | Cross-cluster metrika összehasonlítás egy grafikonon |
| `MET-07` | Aktív Alertmanager riasztások listája, erőforráshoz kötve |
| `MET-08` | `metrics-server` fallback, ha nincs Prometheus (élő `top`-szerű adat) |

### F. Time travel — `TIME`

| ID | Képesség |
|---|---|
| `TIME-01` | Watch-események lokális perzisztálása SQLite-ba, konfigurálható megőrzési idővel és hatókörrel |
| `TIME-02` | Idővonal-csúszka: a teljes UI visszaállítása egy múltbeli időpont állapotára |
| `TIME-03` | Erőforrás-változástörténet: mi változott, mikor, mezőszintű diff két revízió között |
| `TIME-04` | Eseménynapló megőrzése a k8s 1 órás ablakán túl |
| `TIME-05` | Incidens-csomag exportálása: kijelölt időablak állapota + eseményei + logjai, megosztható fájlként |
| `TIME-06` | Tárhely-kezelés: mennyit foglal, mit tartunk meg, ürítés |

### G. Diagnosztika — `DIAG`

| ID | Képesség |
|---|---|
| `DIAG-01` | „Miért Pending ez a pod?" — node-onkénti lebontás: melyik taint / affinity / kapacitás / topológiai megkötés zárta ki |
| `DIAG-02` | „Miért nem áll készen?" — probe-hibák, init konténer állapot, image pull hiba, mount hiba egy nézetben |
| `DIAG-03` | Rightsizing: requests/limits vs. tényleges Prometheus-használat, túl- és alulfoglalás kimutatása |
| `DIAG-04` | Node-nyomás nézet: allokált vs. kapacitás, evikciós kockázat, elaprózódás |
| `DIAG-05` | Restart- és OOMKill-toplista clusterenként és flottaszinten |
| `DIAG-06` | Elárvult erőforrások: nem használt PVC, ConfigMap, Secret, Service végpont nélkül |
| `DIAG-07` | Workload-egészség pontszám, a fenti jelekből összegezve |

### H. Biztonság és megfelelőség — `SEC`

| ID | Képesség |
|---|---|
| `SEC-01` | Fordított RBAC: „mit tud valójában ez a ServiceAccount / user?" — jogosultságok visszavezetve a Role/Binding forrásra |
| `SEC-02` | RBAC fordított irányban: „ki tud törölni podot ebben a namespace-ben?" |
| `SEC-03` | Tanúsítvány-idővonal: minden `tls` Secret, Ingress/Route cert, lejárati sorrendben, flottaszinten |
| `SEC-04` | Kockázatos beállítások kiemelése: privileged, hostNetwork, hostPath, root futtatás |
| `SEC-05` | Image-forrás áttekintés: mely registry-kből futnak konténerek, tag vs. digest |
| `SEC-06` | Az alkalmazás saját írási műveleteinek auditnaplója (ki, mit, mikor, melyik clusteren) |

### I. OpenShift-specifikus — `OCP`

| ID | Képesség |
|---|---|
| `OCP-01` | Natív nézetek: Route, Project, DeploymentConfig, ImageStream, BuildConfig |
| `OCP-02` | Build-követés: futó buildek, build logok, újraindítás |
| `OCP-03` | Operator / CSV / Subscription állapot, upgrade-státusz |
| `OCP-04` | SCC-nézet: melyik SCC-t kapja egy workload és miért |
| `OCP-05` | ClusterOperator-egészség (az OCP saját komponensei) |
| `OCP-06` | Route ↔ Ingress egységes megjelenítés, hogy a vegyes flotta egy nézetben legyen |

### J. Írási műveletek — `ACT`

| ID | Képesség |
|---|---|
| `ACT-01` | Skálázás, rolling restart, pod törlés, cronjob trigger/suspend |
| `ACT-02` | YAML szerkesztés és alkalmazás, diff-előnézettel a mentés előtt |
| `ACT-03` | Dry-run és szerveroldali validáció alkalmazás előtt |
| `ACT-04` | Port-forward kezelése (indítás, aktív forwardok listája, leállítás) |
| `ACT-05` | Beépített terminál emulátor: pod `exec` és node `debug` a felületen belül. A node-shell privilegizált podot indít a node host-névterében — `CONN-08` tiltja csak-olvasás módban, `ACT-07` megerősítést kér, `SEC-06` naplózza |
| `ACT-06` | Cordon / drain / uncordon node-okon |
| `ACT-07` | Megerősítő párbeszéd destruktív műveleteknél, prod clustereken szigorúbb (név begépelése) |
| `ACT-08` | Fájl másolás podba / podból |

### K. Alkalmazás-keret és UX — `APP`

| ID | Képesség |
|---|---|
| `APP-01` | Parancspaletta (`Ctrl+K`): erőforrás, cluster, művelet — minden egy helyről |
| `APP-02` | Billentyűzet-központú navigáció, vim-szerű mozgás opcionálisan |
| `APP-03` | Több panel / fül, egymás mellett különböző clusterek nézetei |
| `APP-04` | Munkamenet visszaállítása induláskor (megnyitott fülek, szűrők) |
| `APP-05` | Világos/sötét téma, sűrűségi beállítás |
| `APP-06` | Egyetlen bináris, telepítés nélkül futtatható, Linux/macOS/Windows |
| `APP-07` | Alkalmazáson belüli hibanapló és diagnosztika (mi ment félre a kapcsolatokkal) |

---

## 6. Kereszt-metszeti követelmények

Ezek nem feature-ök, hanem minden feature-re érvényes szabályok. A design és a kód is kösse magát hozzájuk.

**Teljesítmény és memória.** Ez a projekt legnagyobb technikai kockázata: 10+ clusteren mindent watchelni
reflectorral memória-robbanást okoz.
- Alapértelmezés: **metadata-only watch** (`PartialObjectMetadata`) a listanézetekhez, teljes objektum csak részletnézetben.
- **Lazy watch**: csak arra indul watcher, amit a felhasználó megnyitott; inaktivitás után leáll.
- Cél: 10 cluster mellett **< 500 MB RSS**, a UI **60 FPS** marad log-streamelés közben is.
- Minden listanézet virtualizált. Nincs olyan kód, ami képkockánként végigiterál egy 10 000 elemű listán.

**Biztonság.**
- Alapértelmezésben minden cluster csak-olvasható; az írást explicit engedélyezni kell (`CONN-08`).
- Secret értékek soha nem kerülnek a lokális adatbázisba (`TIME-01` hatóköréből kizárva).
- Az incidens-export (`TIME-05`) alapból maszkolja a Secret- és token-tartalmakat.

**Hibakezelés.** Egy elérhetetlen cluster nem blokkolhatja a többit: a nézetek részleges eredményt mutatnak,
a hiányzó forrás jelölve. Minden hosszú műveletnek van látható állapota és megszakíthatósága.

**Írási műveletek biztonsága.** Destruktív akció mindig megerősítést kér; flotta-művelet (`XC-06`) mindig
felsorolja előre, pontosan mire fog hatni.

---

## 8. Fázisterv

A fázisok egymásra épülnek, mindegyik végén használható eredmény áll.

A felület egyedi, ezért két előkészítő fázis előzi meg a funkcionális munkát: a design system, majd a
widgetek megépítése hamis adattal. Valódi működés csak azután jön, hogy a widgetek készen állnak.

| Fázis | Tartalom | Miért itt |
|---|---|---|
| **M-2 — Design system** | Token-lap (színek, térközök, sugarak, vonalvastagságok, tipográfia, animációs időzítések) + widget-katalógus. Nem képernyőterv, hanem elemkatalógus | A Claude Design kimenete. Ebből lesz a `theme.rs` |
| **M-1 — Widget gallery** | `kubescope-ui` + `kubescope-gallery`: minden widget megépítve, hamis adattal, élő paraméterezéssel és szélsőséges esetekkel (0 elem, 10 000 elem, hosszú nevek, hibaállapot) | Itt derül ki, mi működik egui-ban. Egyben ez kényszeríti ki a view-model típusok definícióját |
| **M0 — Váz** | `CONN-01/03/06`, `APP-05/06`, alap eframe-alkalmazás, tokio háttér-runtime, `Store` bekötése | Kapcsolódni tudunk és látjuk, hogy él |
| **M1 — Böngészés** | `RES-01/02/04/05/06/07`, `APP-01/02/03`, virtualizált táblázatok | Ettől kezdve napi eszköz egy clusterhez |
| **M2 — Logok** | `LOG-01…06`, `ACT-04` | A leggyakoribb művelet, és itt dől el a teljesítmény-architektúra |
| **M3 — Írás** | `ACT-01/02/03/06/07`, `CONN-08`, `SEC-06`, `RES-09` | A k9s-paritás nagyjából itt van meg |
| **M4 — Flotta** | `CONN-02`, `XC-01/02/03/05`, `RES-03` | **Első ék.** Innentől van olyan, amit más nem tud |
| **M5 — Metrikák** | `MET-01…05/07/08` | **Második ék.** Az esemény-annotáció (`MET-04`) a kulcsdarab |
| **M6 — Time travel** | `TIME-01…06` | **Harmadik ék.** A legnagyobb tárolási és adatmodell-munka, ezért hátul |
| **M7 — Elemzés** | `DIAG-01…07`, `SEC-01…05`, `XC-04/06/07` | A korábbi adatokra épül, önmagában nem állna meg |
| **M8 — OpenShift** | `OCP-01…06`, `CONN-05/07` | Párhuzamosítható M4-től, ha van OCP tesztkörnyezet |

> Megjegyzés: az OpenShift-támogatást szándékosan nem tettük az M0-ba, de a `CONN-06` (képesség-felderítés)
> már az elején számoljon vele, hogy a nézetek később cluster-típus szerint bővülhessenek.

---

## 9. Vizuális irány

A cél **futurisztikus, teljesen egyedi felület** — nem az egui alapértelmezett kinézete.

Egy feszültséget viszont fel kell oldani: a célfelhasználó incidens közben sűrű információt olvas, a
futurisztikus stílus pedig hajlamos helyet elvenni dekorációra. **A feloldás: a karakter a kromatikán és a
kereten legyen** (sötét alap, akcentszín, éles geometria, precíz vonalvezetés), a **tartalmi felület viszont
maradjon sűrű és nyugodt.** Ez korlát a design fázis felé, nem stílusjavaslat.

**Amit az egui olcsón ad:** egyedi keretek, sarokvágás, per-sarok lekerekítés, hexagon és egyéb formák,
sávos elválasztók, animált fókusz-vonalak, HUD-szerű overlay, egyedi betűtípus — mind a `Painter` + `Shape` API-val.

**Ami drága vagy nem natív:** valódi bloom/glow (nincs post-process; több rétegű, csökkenő alfájú stroke-kal
hamisítható, vagy `egui_wgpu` paint callback + saját shader), gradiens kitöltés (`Mesh` építése vertex-színekkel),
üvegszerű blur, per-glyph szövegeffekt. Ezek külön alprojektnek számítanak — az M-1 fázis döntse el, kell-e belőlük.

**Nyitott tétel:** ha a design egyedi betűtípust igényel, a licencet a beágyazás előtt ellenőrizni kell —
sok display font nem engedi a terjesztést.

---

## 10. Eldöntendő

**Az M-1 (widget gallery) előtt:**

1. **Lapos vagy összevont alapértelmezés** csoport-hatókörben (`XC-08`)? A lapos hibakereséshez jobb,
   az összevont drift-vadászathoz. Javaslat: lapos alapból, összevont kapcsolóval.
2. **Time travel megjelenése**: globális mód (az egész alkalmazás visszaáll egy időpontra) vagy nézeten
   belüli panel? A globális erősebb, de sokkal invazívabb — a widget-katalógust is érinti.
3. **Sűrűségi szint**: hány sor férjen el egy tipikus laptop-képernyőn? Ez konkrét szám legyen, mert a
   tipográfiát és a térközöket ez határozza meg.

**Az M0 előtt:**

4. **Milyen tesztkörnyezet áll rendelkezésre?** kind/minikube elég az M0–M3-hoz, de az M4 (flotta) és
   M8 (OCP) valós clustereket igényel. Ez befolyásolja a fázisok sorrendjét.

**Eldöntve:** navigációs modell (5. fejezet), workspace-felosztás (4.1), fázissorrend (8. fejezet).
