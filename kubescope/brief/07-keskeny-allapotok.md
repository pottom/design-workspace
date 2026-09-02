# KubeScope — Design brief: a keskeny állapotok

> A Claude Design bemenete. Ez a hetedik kör, és **szűk**: nem új panel, nem új szín, nem új
> komponens. Egyetlen kérdés, amit a 31 megrajzolt képernyő nem válaszol meg.
>
> Kezdd itt: `design/README.md` → „The known gap".

---

## 1. A feladat egy mondatban

**Mi történik a panelek testével, amikor keskenyebbek lesznek** — mert mind a 31 képernyő
1600 × 1000-en készült, és ez nem az a méret, amiben a terméket használni fogják.

## 2. Miért ez a feladat, és miért most

A króm megvan. A panelfejléc és a vezérlősáv degradációja **általános szabály, px-küszöbökkel**, és
minden panelre érvényes:

| sáv | mi megy, sorrendben | px |
|---|---|---|
| fejléc | a kapcsolat szava → a namespace → a cluster neve csonkul (8 karakterig) | 480 · 400 · 340 |
| fejléc | **a típusglif és a `⊞ ⋯ ✕` soha** | — |
| vezérlősáv | a stream szava → a darabszám mértékegysége → a keresőmező ikonra → a csipeszek összecsuknak | 520 · 460 · 380 · 340 |

**A testeké nincs meg**, és az minden panelnél más. Mértem: a 31 fájlból **tizenháromban egyetlen
degradációs jel sincs**, és ezek jórészt a legutóbb érkezett tizenöt panel.

```
kidolgozva (≥6)  01 04 05 06 28 29                       ██████ 6
érintve   (1–5)  02 03 07 08 10 13 15 16 17 18 22 25     ████████████ 12
semmi        (0) 09 11 12 14 19 20 21 23 24 26 27 30 31  █████████████ 13
```

Ez pontosan az a pont, amit a legelső brief a termék tétjeként nevezett meg:

> „Négy panel 1280 pixelen ~300 pixeles paneleket jelent. **Ez nem szélsőséges eset, hanem a
> mindennapi**, és a mostani felület pont ezen bukik el." — `brief/02-hol-tartunk.md` §5

## 3. A valódi szélességek — és nem a 320

Fontos, és eddig senki nem mondta ki. A `04-Decisions` 4. döntéséből (az új panel 40%-ot kap, a
küszöb 804 px) **következik**, hogy `⊞`-vel egy 1600 px-es mezőből pontosan ezek a panelszélességek
érhetők el:

```
1600  ·  958  ·  638  ·  572  ·  382
```

Ebből:

- **A 320 px ritka.** Csak kézi fogóhúzással érhető el, mert az osztás 804 alatt fület nyit helyette.
  Nem kell rá minden panelt megtervezni.
- **A 638 a mindennapi.** Két osztás egy 1600-as ablakban — ez a leggyakoribb elrendezés a
  termékben, és **egyetlen panel sincs rajta megrajzolva**.
- **A 382 a valódi alsó határ**, nem a 320.

**Kérés: a specimeneket ezeken a szélességeken rajzold, ne kerek számokon.** A 958 és a 638 a két
legfontosabb; a 382 az, ahol eldől, mi marad egyáltalán.

## 4. Amit ne tervezz újra

- A krómot (a fenti táblázat kész és érvényes).
- A hat döntést (`design/05-frame/04-Decisions.dc.html`).
- A tokeneket, a komponensfát, a primitíveket.
- **A `01-Resource-List` degradációját.** Az kész, és jó: a szerver `priority` mezője dönti el, mi
  esik ki — ez a szerver ítélete arról, mi számít, nem a miénk. **Ez legyen a minta** a többihez.

## 5. Amire válaszolnod kell

### 5.1 Előbb egy általános szabály

Ahogy a krómnak van egy grammatikája, **a testnek is kell egy**. Nem panelenkénti ízlésdöntések
sorozata, hanem néhány kimondott elv, amiből a huszonnégy panel levezethető. Például — ezek csak
kérdések, nem javaslatok:

- Mikor **esik ki** egy elem, és mikor **alakul át**? (Egy oszlop kiesik. Egy három hasábos
  elrendezés nem esik ki — mássá kell váljon.)
- Van-e olyan elem, ami **soha nem eshet ki** egy testből, ahogy a `⊞ ✕` sem a fejlécből?
- Mikor lesz egy **vízszintes** elrendezésből függőleges, és mikor lesz belőle **görgetés**?
- A **szám** és a **sáv** közül melyik megy előbb? (A `MUST NOT` szerint a százalék abszolút érték
  nélkül tilos — akkor keskenyen melyik marad?)

### 5.2 Aztán a tizenegy panel, ami nem magától szűkül

Három kategóriába estek, és csak a második és a harmadik igényel döntést:

**Magától szűkül — nem kell rájuk külön terv**, csak erősítsd meg a szabályt:
`01` lista · `03` események · `02` napló · `19` tanúsítványok · `20` pazarlás · `21` kubeconfig
· `22` watch-szabályok · `18` OpenShift

> Ezek `DataTable`-ek, tehát a `priority`-szabály viszi őket. **Egy kérdés viszont van:** a
> `19` / `20` / `21` egész értelme egy **számított verdikt-oszlop** („megújítás", „öt vödör", „a
> pár"). Ez az oszlop nyilván soha nem eshet ki — de akkor mi esik ki helyette 382 px-en, amikor a
> névoszlop és a verdikt együtt már nem fér el?

**Az elrendezési premisszájuk hal meg — ezekhez terv kell:**

| # | panel | mi dől össze |
|---|---|---|
| `10` | **Details** | „**három oszlop, nem hosszú görgetés**" — ez a panel egész érve. 638-on ez nem három oszlop |
| `16` | What would happen | három hasábos összehasonlítás (van · lenne · különbség) |
| `24` | Node | a node saját sávja **és** minden pod sávja, két jellel — két sávrendszer egymás alatt |
| `15` | Metrics | görbék eseményjelekkel; a jelölők a termék, nem a görbék — mi marad 382-n? |

**Nem tudjuk, mert N oszlopuk van — ezekhez döntés kell:**

| # | panel | a probléma |
|---|---|---|
| `09` | RBAC | 8 × N mátrix (erőforrás × verb). Mi lesz belőle keskenyen? |
| `17` | Helm | **clusteronként egy oszlop**, és tíz cluster is lehet. 638-on ez hány? |
| `11` | Assistant | saját szín, hivatkozás-csipeszek, idézetek — hosszú szöveg keskeny hasábban |
| `14` | Terminal | a PTY oszlopszáma **valódi következmény**: 382 px ≈ 40 oszlop. Mit lát a felhasználó, és mit mondunk neki? |

### 5.3 És a keret-képernyők keskenyen

`26` első indítás · `30` tömeges művelet · `31` incidens — mind 1600-on van. A `31` a termék
címlapja, **négy panel egyszerre**: rajzold meg azon a szélességen, ami a négy panelből tényleg
kijön (958 + 638, alattuk osztva), ne fél-fél képernyőn.

## 6. Amit át kell adnod

1. **Egy oldal az általános test-grammatikáról** — az 5.1 elvei, kimondva, a króm-táblázat
   párjaként, px-küszöbökkel ahol van értelme.
2. **A négy „premissza meghal" panel** (`10` `16` `24` `15`) mind a három szélességen: 958 · 638 · 382.
3. **A négy „N oszlop" panel** (`09` `17` `11` `14`) ugyanígy, plusz a döntés indoklása — mit adunk
   fel, és mit kapunk érte.
4. **A verdikt-oszlop kérdésére** (5.2 első blokk) egy szabály.
5. **A `31` incidens-képernyő** a valódi panelszélességeken.

## 7. A mérce

Akkor jó, ha **a 638 px-es panel ugyanolyan gondosan van megtervezve, mint az 1600-as** — nem
megcsonkítva, hanem sajátjaként. A terméket négy panelen fogják használni, tehát a 638 az
alapállapot, és az 1600 a kivétel.

És akkor jó, ha építés közben **egyszer sem kell kitalálnom semmit**. Ez az egyetlen mérce, ami
eddig is számított.
