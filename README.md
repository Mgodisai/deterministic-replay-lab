# Deterministic Replay Lab (2026-NP-04)

Ez a repository a **Determinisztikus visszajátszás és állapotszinkronizáció játékokban** (témakód: `2026-NP-04`) című mérnökinformatikus BSc szakdolgozati és projektmunka dokumentációját és forráskódját tartalmazza.

## Projekt áttekintés

A **Deterministic Replay Lab** egy .NET (C#) alapú, fix időléptékű (tick-alapú) 2D-s szimulációs motor és kísérleti mérőkörnyezet. A rendszer célja annak vizsgálata és demonstrálása, hogyan valósítható meg a játékmenet bitazonos, reprodukálható visszajátszása (replay) pusztán a bemenetek naplózásával, hogyan detektálhatók a nemdeterminisztikus állapot-eltérések (desync) ellenőrző összegek (hash/checksum) segítségével, valamint milyen tárhely- és számítási idős kompromisszumokkal jár a ritkított pillanatképek (snapshotok) alkalmazása az időbeli pozicionálás (seeking) és az állapot-helyreállítás (resync) során.

## Projektdokumentáció (1. Félév - Projekt 1. tárgy)

A kurzus előírásainak megfelelően az 1. félév dokumentumai a `00-project/` mappában találhatók:

* **[00-application.md](./00-project/00-application.md)** – Az elfogadott projektjelentkezés és brief értelmezés.
* **[01-project-outline.md](./00-project/01-project-outline.md)** – A projekt céljai, hatóköre, sikerkritériumok és a GitHub Projects tábla hivatkozása.
* **[02-specification.md](./00-project/02-specification.md)** – Részletes funkcionális specifikáció, szerepkörök, use case-ek, követelmények és kezdeti technikai javaslat.
* **`03-presentation-1.md`** – Az 1. prezentáció PRMD forrása (probléma, célfelhasználók, hatókör).
* **`04-presentation-2.md`** – A 2. prezentáció PRMD forrása (specifikáció, technikai javaslat, 2. féléves terv).

## Repository Struktúra

```text
deterministic-replay-lab/
├── 00-project/                            # Az 1. félév kötelező dokumentumai és prezentációi
    ├── 00-application.md
    ├── 01-project-outline.md
    ├── 02-specification.md
    ├── 03-presentation-1.md
    └── 04-presentation-2.md
