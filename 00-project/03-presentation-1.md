---
project_outline: "[[00-project/01-project-outline]]"
---

<!-- 5xl -->
# Determinisztikus visszajátszás és állapotszinkronizáció játékokban

Szerző: Varga Tamás  
Neptun: UE6LI3  
Témavezető: Novák Péter  
Projektazonosító: 2026-NP-04  

---

<!-- 2xl -->
[prmd.atom-forge.eu/docs](https://prmd.atom-forge.eu/docs)

---

## Probléma

Egy játék vagy szimuláció futását **vissza szeretnénk játszani**: megnézni, visszatekerni, ellenőrizni.

- Naiv megoldás: minden képkockán elmentjük a **teljes állapotot** → óriási tárhely
- Okosabb: csak a **bemeneteket** mentjük (mikor mit nyomott a játékos), és a futást **újraszámoljuk**
- Ez csak akkor működik, ha ugyanabból a bemenetből **mindig ugyanaz** jön ki

---

### Miért éri meg a bemenet-alapú megoldás?

Becslés: 10 000 tick, 50 szereplő

| Mit mentünk | Becsült méret |
| --- | ---: |
| Minden tick teljes állapota | ≈ 20 MB |
| Bemenetek + ritka pillanatképek + ellenőrző hash-ek | ≈ 0,3 MB |

→ nagyjából **98%-kal kisebb** (a pontos érték a mérésből jön)

---

### Miért nehéz? A determinizmus gyenge pontjai

Ugyanaz a bemenet is **más eredményt** adhat, ha a szimuláció függ:

- a **valós időtől** vagy a képkockasebességtől (FPS)
- a **véletlenszám-generátortól** (nem rögzített kezdőérték)
- a **lebegőpontos számolás** apró eltéréseitől
- a **feldolgozási sorrendtől** (pl. rendezetlen gyűjtemények)

Egy apró eltérés a futás során **felgyülemlik**, és a visszajátszás „szétcsúszik" (angolul: *desync*).

---

### Mit szeretnék megmutatni és megmérni?

1. A visszajátszás **hibátlanul reprodukálja** a futást
2. Az eltérést **gyorsan észrevesszük** (ellenőrző összeggel, hash)
3. Az eltérést **helyre tudjuk állítani** egy korábbi pillanatképből
4. Mennyibe kerül mindez: **tárhely, futásidő, ugrási idő**

---

## Célfelhasználók

A rendszer egyfelhasználós asztali alkalmazás, három használati szerep:

| Szerep | Mit szeretne? |
| --- | --- |
| **Operátor** | Játszik, rögzít, visszajátszik, tetszőleges pontra ugrik |
| **Kutató** | Zavarást (hibát) injektál, méréseket futtat, exportálja az eredményt |
| **Ellenőrző** | Betölti a replay-t, újrafuttatja a kísérletet, összeveti az eredményt |

---

### Érintettek

- **Játék- és szimulációfejlesztők:** akiknek a hibakeresést, tesztelést segíti a reprodukálható futás
- **Szakmai bírálók:** akik a módszert és a mérési eredményeket ellenőrzik
- **Én, mint kutató:** a kísérletek elvégzője

Nincs bejelentkezés, nincs jogosultságkezelés.

---

## Tervezett hatókör

**Benne van:**

- 2D aréna-szimuláció fix időlépéssel (tick) és seedelt véletlennel
- bemenetek rögzítése és visszajátszása
- állapot-hash, eltérés-felismerés
- pillanatképek, ugrás a felvétel adott pontjára, helyreállítás
- hibainjektálás és mérés (tárhely, többletköltség, helyreállítási idő)

---

### Ami kívül marad

- hálózati többjátékos mód
- bitazonos működés **eltérő gépek és operációs rendszerek** között
- 3D megjelenítés, külső fizikai motor, hang

---

### A 2. félévi első célkitűzés (MVP)

- **Kötelező** követelmények: 23 db (az összesen 39-ből)
- egy aréna, `double` számábrázolás, egy helyreállítási módszer
- két hibatípus: állapotzaj és elcsúsztatott bemenet
- kísérletek parancssorból, eredmény CSV-ben, egyszerű WPF felület
- a többi (Ajánlott, Lehetséges) csak ezek után

---

## Kezdeti felhasználói utak

Hat fő használati eset (UC1–UC6), egy tipikus útvonalon:

```
Rögzítés → Visszajátszás → Ugrás → Hiba injektálása → Helyreállítás → Mérés → Újrafuttatás
 (UC1)        (UC2)       (UC3)        (UC4)            (UC4)         (UC5)     (UC6)
```

---

### Rögzítés és visszajátszás (UC1, UC2)

1. Az operátor elindít egy futást, a rendszer **tick-enként naplózza a bemenetet**
2. Időnként pillanatképet és hash-t ment, a végén replay fájlba írja
3. Visszajátszáskor a rendszer **ugyanazt a bemenetet adja vissza**, és minden ellenőrzőponton összeveti a hash-t
4. Eredmény: **egyezik** vagy **az első eltérés ticke**

---

### Ugrás és hibakezelés (UC3, UC4)

- **Ugrás:** legközelebbi korábbi pillanatkép betöltése + a köztes lépések újraszámolása
- **Hiba injektálása:** a kutató szándékosan elront valamit
- **Észlelés:** a hash eltér az első ellenőrzőponton, ahol a hiba még látszik
- **Helyreállítás:** visszaállás az eredeti replay pillanatképére, újraszámolás, hash-ellenőrzés

---

### Mérés és reprodukálás (UC5, UC6)

- A kutató paraméterekkel (hossz, N, K, hiba típusa) lefuttatja a kísérletet **grafikus felület nélkül**
- Az eredmény **CSV**: konfiguráció, környezet, helyességi, tárhely- és időmutatók
- Az ellenőrző **egy paranccsal** újrafuttatja, és összeveti a hash-eket és méreteket (az idők gépfüggők)

---

## Következő lépések

**Kész:**

- projektleírás (outline)
- funkcionális specifikáció: 6 use case, 39 követelmény, szabályok, wireframe

**Következik:**

- 2. prezentáció: a specifikáció és a technikai javaslat
- 2. félév: headless mag → ugrás és helyreállítás → mérések → WPF felület

---

### Amit egyeztetni szeretnék

- a .NET 10 (LTS) használata a korábban tervezett .NET 8/9 helyett
- az MVP szűkítése (két hibatípus, egy helyreállítási módszer, `double`)
- a mérési terv és a célértékek (ezek kiinduló hipotézisek)
