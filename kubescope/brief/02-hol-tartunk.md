# KubeScope — hol tartunk

> Kísérő a `00-kepernyo-brief.md`-hez. Röviden: mi van kész, mit dobtunk el, és mi az, amiben a
> terved futni fog.

## 1. Ami kész és marad

**A cluster-oldal.** 3 142 sor Rust, 35 teszt, plusz élő clusteren futó tesztek. Nulla UI-tudás van
benne — pont ezért éli túl a felület cseréjét.

Amit tud: kubeconfig beolvasása, cluster-próba (elérhetőség, RTT, verzió, platform, API-csoportok),
pod- és node-lista **élő watch**-csal (újracsatlakozás, `410 Gone`, lejárt resourceVersion kezelve),
egy objektum manifesztje, log-követés, és `exec` egy konténerbe. A figyelés **panelenként kulcsolt**:
a felület deklarálja, mit mutat, a háttér pedig ehhez igazítja a kapcsolatokat.

Ez a réteg fogja etetni a te komponenseidet. A mezők pontos listája a brief `5.2`-ben.

## 2. Amit eldobtunk, és miért

**A teljes felület.** Kb. 22 000 sor: egy saját widget-könyvtár (`kubescope-ui`), egy kirakat, és az
alkalmazás héja. Natív, azonnali módú GUI-val (egui) készült, ahol minden képernyőelemet kézzel
kellett megrajzolni — a görgetősávot, a beviteli mezőt, a lebegő menüt, a fókuszgyűrűt, a `…`-os
csonkítást.

**Nem azért dobtuk el, mert lassú volt.** Azért, mert a te terveid HTML-ben készültek, és valaki
kézzel fordította le őket egy rajzoló API-ra. Ez a fordítás minden körben veszített, és a végeredmény
a tervhez képest rosszul nézett ki. **Ez a fordítás most megszűnik**: az alkalmazás Tauri + React +
TypeScript lesz, tehát amit kiadsz, az lesz a kód.

A régi kód nincs kitörölve, egy `legacy/` mappában áll, amíg ki nem bányásztuk belőle azt a néhány
döntést, ami érték: oszlopkészletek, a Kubernetes-fázis → jelvény leképezés, a naplószint-felismerés,
és az osztott fa szabályai.

## 3. Ami megvan és bevált — a viselkedés

Ezek működnek, a felhasználó szereti őket, és **ne tervezd újra a gesztusaikat** — csak a kinézetüket:

- **Munkaterületek** i3 módjára: számozottak, `⌘1…0`-val válthatók, átnevezhetők, átrendezhetők.
- **Osztott fa**: egy munkaterületen panelek, vízszintesen és függőlegesen osztva, húzható
  osztásfogókkal, négyes mélységkorláttal.
- **Panel- és fülmozgatás** húzással, **élő próbával**: húzás közben a leendő elrendezés látszik, nem
  egy kiemelt zóna.
- **Panelenkénti hatókör**: nincs globális „aktuális cluster". Minden panel a sajátját hordozza, tehát
  két panel egymás mellett két cluster lehet.
- **Fülek** egy panelhelyen, ha több nézet osztozik rajta.

## 4. Két panelcsalád — ez a terved szempontjából számít

| család | típusok | a hatóköre |
|---|---|---|
| **kérdés-panel** | pod-lista, node-lista, eseménylista | választott, átállítható |
| **tárgy-panel** | napló, manifeszt, terminál | örökölt a forrássortól, **rögzített** |

Egy pod-lista megkérdezhető egy másik clusterről is — ugyanaz a kérdés, más válasz. Egy manifeszt
nem: az *ennek* az objektumnak *ebben* a clusterben a leírása. Átállítani nem ugyanannak a másik
nézete volna, hanem egy másik tárgy — vagyis egy másik panel.

## 5. Amit tudni érdemes a méretekről

- Alapértelmezett ablak: **1600×1000**. A legkisebb, amit támogatunk: **1280×800**.
- Egy panel minimuma **320×180** — ez alatt nem lehet tovább osztani.
- Négy panel 1280 pixelen ~300 pixeles paneleket jelent. **Ez nem szélsőséges eset, hanem a
  mindennapi**, és a mostani felület pont ezen bukik el.
- A régi cél 1080p-n 40 adatsor volt. Ez **megkérdőjelezhető** — ha a terved kevesebbet mutat, de
  olvashatóbbat, mondd meg, és vitassuk meg.
