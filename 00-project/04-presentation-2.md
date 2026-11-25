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

## A specifikáció összefoglalása

Mi van a specifikációban?

- 3 szereplő, **6 use case** (UC1–UC6)
- **39 funkcionális követelmény**, elfogadási kritériummal és prioritással
- üzleti szabályok és korlátok, nem funkcionális célok
- a felület leírása és wireframe-je
- kezdeti technikai javaslat és kísérleti terv

| Prioritás | Darab |
| --- | ---: |
| Kötelező (MVP) | 24 |
| Ajánlott | 11 |
| Lehetséges | 4 |

---

### A kulcsötlet

A futást nem a képkockákból, hanem a **bemenetekből** állítjuk elő újra.

- A szimuláció **fix lépésekben (tick)** fut: **Sₜ = Step(Sₜ₋₁, inputₜ)**
- A bemenetet **tick-sorszámmal** naplózzuk
- **K tickenként** hash-t, **N tickenként** pillanatképet mentünk
- A hash jelzi az eltérést, a pillanatkép az ugrást és a helyreállítást

A bemenetnapló marad a hiteles forrás, a hash és a pillanatkép csak gyorsítás és ellenőrzés.

---

### A legfontosabb szabályok

- a szimulációs magban **tilos**: rendszeridő, `System.Random`, szálak versenyhelyzete, rendezetlen bejárás
- az entitások és a hash **rögzített (azonosító szerinti) sorrendben** készülnek
- egyetlen **kanonikus bináris forma** a hash, a pillanatkép és a méret alapja
- a hash **valószínűségi** jelző, nem bizonyíték (tartalék: 128 bites hash)
- sérült fájl ≠ szimulációs eltérés: **két külön hiba**

---

### A garantált determinizmus határa

**Garantáljuk:** azonos build, operációs rendszer, architektúra, .NET futtatókörnyezet és Release profil mellett két külön folyamatban bájtszintű egyezés.

**Nem garantáljuk:** más hardveren, más futtatókörnyezetben, Debug és Release között. Ott az egyezés **kísérleti eredmény**, amit a dolgozatban dokumentálok.

---

## Fő felhasználói utak és követelmények

| UC | Szereplő | Cél |
| --- | --- | --- |
| UC1 | Operátor | Szimuláció rögzítése |
| UC2 | Operátor | Visszajátszás és ellenőrzés |
| UC3 | Operátor | Ugrás a felvétel adott pontjára |
| UC4 | Kutató | Hiba injektálása, észlelés, helyreállítás |
| UC5 | Kutató | Kísérlet futtatása, CSV export |
| UC6 | Ellenőrző | Kísérlet reprodukálása |

---

### Követelménycsoportok

| Csoport | Téma | Db |
| --- | --- | ---: |
| A | szimuláció és véletlen | 7 |
| B | rögzítés és fájl | 5 |
| C | visszajátszás és vezérlés | 7 |
| D | ellenőrzés, eltérés, helyreállítás | 9 |
| E | perturbáció | 3 |
| F | mérés és reprodukálhatóság | 6 |
| G | felület | 2 |

---

### Példa: egy követelmény és az elfogadási kritériuma

**FR-D7 – Automatikus helyreállítás** (Kötelező)

1. Észleléskor a futás megáll
2. Az **eredeti replay** utolsó, észlelés előtti pillanatképéből újraszámolunk, hiba nélkül
3. Ha a hash egyezik, az aktív állapotot lecseréljük, a függő hibahatásokat töröljük
4. A futás az eredeti naplóval folytatódik

**Célzott teszt:** K = 1, a bemenet t → t+1 eltolva, észlelés és helyreállítás t-nél, egyező hash t+1-nél is.

---

### Mit mérünk? (kiinduló hipotézisek)

| Cél | Érték |
| --- | --- |
| Egy tick + hash, 50 entitás, K = 1 | átlagosan 1 ms alatt |
| Ugrás, N = 100 | p95: 50 ms alatt |
| Tárhely a „minden tick" módhoz képest | legalább 80%-kal kisebb |
| Észlelési késleltetés | legfeljebb K tick |

A számok a 2. félév elején mért prototípus-adatok alapján **finomodnak**.

---

### A felület

Egy főablak, két terület és egy állapotsor:

- **bal:** 2D aréna, lejátszóvezérlő, idővonal (pillanatkép- és ellenőrzőpont-jelölésekkel)
- **jobb:** *Kísérlet* fül (seed, N, K, hiba típusa) és *Mérések* fül (tárhely, idők, eltérések, CSV)
- **alul:** aktuális tick, hash, mód, ellenőrzés állapota

---

## Kezdeti technikai javaslat

Két réteg, .NET 10 (C#):

1. **Replay-keretrendszer** (`Core`): nem tud a játékról, csak egy interfészen át éri el a szimulációt
2. **Szimuláció** (`Simulations`): az aréna, mint referencia

Ha a szimuláció lépése determinisztikus, a keretrendszer garantálja a reprodukálhatóságot és az eltérés felismerését.

---

### Az interfész (vázlat)

```csharp
interface ISimulation<TState, TInput>
{
    TState Init(SimulationConfig config, ulong seed);
    TState Step(TState state, TInput input);
    byte[] Serialize(TState state);
    TState Deserialize(ReadOnlySpan<byte> bytes);
}

interface IInputSource<TInput>
{
    TInput GetInput(long tick);   // billentyűzet, bot vagy replay napló
}
```

---

### Öt projekt

| Projekt | Szerep |
| --- | --- |
| `Core` | keretrendszer: ciklus, napló, PRNG, hash, pillanatkép, fájl, mérés |
| `Simulations` | aréna (+ egy minimális második szimuláció) |
| `App` | WPF felület, MVVM |
| `Cli` | grafikus felület nélküli kísérletfuttató |
| `Tests` | xUnit tesztek |

A `Core` **nem hivatkozik** sem az arénára, sem a WPF-re.

---

### Architektúra

```
Operátor / Kutató / Ellenőrző
      │                │
 App (WPF)        Cli (headless)
      └───────┬────────┘
              ▼
┌───────────── Core ──────────────┐
│ bemenet → tick-ciklus → napló   │
│ szerializáció → hash (K)        │
│              → pillanatkép (N)  │
│ perturbáció · mérés · CSV       │
└──────────────▲──────────────────┘
               │ ISimulation
          Simulations
```

---

### Technológiai irány

| Terület | Választás |
| --- | --- |
| Nyelv | C# / .NET 10 (LTS) |
| Számábrázolás | `double` (MVP), fixpont (Ajánlott) |
| Véletlen | saját xorshift / PCG, menthető állapottal |
| Hash | `XxHash64` a kanonikus bájtokon |
| Felület | WPF, MVVM (`CommunityToolkit.Mvvm`) |
| Teszt, mérés | xUnit, `Stopwatch`, CSV |

---

### Megvalósíthatóság: mit és hogyan igazolok?

| Feltételezés | Igazolás | Tartalék |
| --- | --- | --- |
| 50 000 tick bitazonos | korai prototípus, két példány, két processz | fixpontos számolás |
| Az ugrás elég gyors | mérés N = 10…500 mellett | adaptív pillanatkép |
| A hash elég olcsó | mikrobenchmark, K = 1 | nagyobb K |
| Nincs rejtett állapot | mezőlefedettségi teszt | kódellenőrzés |

---

### Előzetes kísérleti terv

1. **Költség:** 1 000 / 10 000 / 50 000 tick × N × K, hash nélküli kontrollal
2. **Észlelés:** ugyanazok az eltolások minden K-nál, négy helyzet az ellenőrzőponthoz képest
3. **Helyreállítás:** sikeres-e az újraszámolás a pillanatképből
4. **Ugrás:** négy távolság-osztály a pillanatképtől, 20 ismétlés, p95

A mátrix előzetes, a témavezetővel egyeztetem.

---

## A 2. félév megvalósítási terve

Fejlesztési sorrend, minden lépésnek mérhető kimenete van:

| # | Szakasz | Kész, ha |
| --- | --- | --- |
| 1 | Headless mag: tick-ciklus, PRNG, aréna, napló, replay fájl, hash | két futás azonos hash-t ad 1 000 / 10 000 / 50 000 tickig, két külön processzben is |
| 2 | Ugrás és helyreállítás: pillanatkép, perturbáció, észlelés | az FR-D7 célzott teszt és az ugrási tesztek zöldek |
| 3 | Mérések: CLI, CSV, kísérleti mátrix | egy paranccsal újrafuttatható, a determinisztikus oszlopok egyeznek |
| 4 | WPF demó | lejátszóvezérlő, idővonal, kísérleti és mérés panel |

---

### Ütemezés és tartalék

- Az **MVP** a 24 Kötelező követelmény; ezek készülnek el először
- Az **Ajánlott** és **Lehetséges** elemek csak utána jönnek (pl. fixpontos mód, második szimuláció, automatikus CSV-összevetés)
- Ha csúszás van, ezek maradnak el, a mérések és a determinizmus nem
- A részletes ütemterv és a mérföldkövek a GitHub Project táblán követhetők

---

### Hogyan ellenőrzöm a munkát?

- **automatikus tesztek:** determinizmus, pillanatkép, hash, perturbáció, hibás fájl, ViewModel
- **mérések:** headless, Release, bemelegítés, 5 ismétlés, medián
- **kézi ellenőrzés:** a felület rajzolása (cél: 30 FPS, 50 entitás)
- **újrafuttathatóság:** README-ben leírt egyparancsos futtatás tiszta környezetben

---

## Nyitott kockázatok és következő lépések

| Kockázat | Mit teszek ellene? |
| --- | --- |
| A `double` más futási módban eltérhet | kísérlet; csak alapműveletek; tartalék: fixpont |
| Rejtett állapot a szimulációban | mezőlefedettségi teszt, pillanatképből indított futás összevetése |
| A hash ütközhet | mezőlefedettség; tartalék: 128 bites hash |
| A WPF torzítja a mérést | headless mérés |
| Túl nagy hatókör | MVP-szűkítés, Ajánlott elemek későbbre |

---

### Kérdések a témavezetőnek

1. **.NET 10 (LTS)** használata, mert a .NET 8 támogatása 2026 novemberében lejárt
2. Az **MVP szűkítése** megfelelő-e (három hibatípus, egy helyreállítási módszer, `double`)
3. A **kísérleti mátrix** mérete és a mérési célértékek
4. **Külső, kész játék** adapterezése: hatókörön kívül hagyom

---

### Következő lépések

1. Témavezetői visszajelzés feldolgozása, a specifikáció frissítése
2. A 2. félév első lépése: **korai prototípus** a determinizmus igazolására (két példány, tickenkénti hash-összevetés)
3. A mérési protokoll és a célértékek finomítása az első mért adatok alapján
4. Ütemterv rögzítése a GitHub Project táblán
