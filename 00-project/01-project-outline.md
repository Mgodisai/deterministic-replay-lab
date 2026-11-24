---
name: Varga Tamás
neptun: UE6LI3
id: 2026-NP-04
github: https://github.com/Mgodisai/deterministic-replay-lab
github_project: https://github.com/users/Mgodisai/projects/2
---
# Determinisztikus visszajátszás és állapotszinkronizáció játékokban

A projekt egy 2D-s szimuláción alapuló demonstrátor és kísérleti mérőkörnyezet (Deterministic Replay Lab) megvalósítása .NET (C#) alapokon. A rendszer ahelyett, hogy minden képkockánál a teljes játékállapotot eltárolná, a rögzített felhasználói bemenetekből és eseményekből állítja elő újra a futást fix időléptékű (tick-alapú) szimulációs ciklus és kontrollált véletlenszám-generálás mellett. Az esetleges állapot-eltéréseket (desync) ellenőrző összegek (hash) segítségével ismeri fel, a ritkított állapot-pillanatképek (snapshotok) pedig a visszajátszásban való gyors pozicionálást (beletekerést) és a hibás állapotból történő helyreállítást (resync) biztosítják. A projekt végére a rendszerrel kísérleti úton mérhető a visszajátszás reprodukálhatósága, tárhelyigénye és futásidejű többletköltsége különböző szimulációs hosszok és zavarások mellett.

## Célok

- **Elsődleges cél:** Működő, bemenet-alapú determinisztikus visszajátszó demonstrátor és mérőkörnyezet készítése állapot-ellenőrzéssel és pillanatkép-alapú helyreállítással.
- **Célfelhasználók / érintettek:** Játék- és szimulációfejlesztők, a kísérleti méréseket végző hallgató és a szakmai bírálók.
- **Mérhető sikerkritériumok:** Zavarásmentes futásnál a visszajátszás 10 000 tick után is az eredetivel megegyező állapot-hash-t ad; szándékos állapotzavarás (perturbáció) esetén a rendszer észleli az eltérést és a legutóbbi pillanatképből helyreállítja a szinkront; a mérőmodul rögzíti és exportálja a tárhelyigényt, a pozicionálási időt és az ellenőrzés CPU-költségét.
- **Korlátok:** C# (.NET 8/9); megjelenítőtől független szimulációs mag; WPF asztali megjelenítő és diagnosztikai felület; a teljes világállapot minden képkockán történő mentése nem lehet az elsődleges visszajátszási mechanizmus; titkos kulcsok vagy személyes adatok nem kerülhetnek a repóba.

## Hatókör

### Benne van a hatókörben

- Fix időléptékű (tick-alapú) 2D-s aréna-szimuláció (mozgás, ütközések, játékosbemenet) és seed-alapú, kontrollált véletlenszám-generálás.
- Bemenetek és események tick-sorszámhoz kötött rögzítése és visszajátszása.
- Állapot-hash (checksum) számítása és az eredetitől való eltérés (desync) automatikus felismerése.
- Konfigurálható gyakoriságú állapot-pillanatképek (snapshotok) készítése, visszajátszás közbeni pozicionálás (seeking) és eltérés utáni helyreállítás (resync).
- Időzítési és numerikus perturbációk (hibainjektálás) előidézése, valamint a tárhelyigény, futási többletköltség és helyreállítási idő mérése.

### Nincs benne a hatókörben

- Hálózati többjátékos (multiplayer) kommunikáció és hálózati késleltetés-kompenzáció.
- Eltérő hardverarchitektúrák és operációs rendszerek közötti garantált, bitazonos cross-platform determinizmus.
- Komplex 3D-s megjelenítés, külső fizikai motorok és hangkezelés.

## Jegyzetek

A szimulációs szabályokat és a visszajátszó mechanizmust önálló osztálykönyvtárban valósítom meg, így a több ezer lépésből álló kísérleti mérések grafikus felület nélkül (headless módon), automatizált tesztekként is gyorsan lefuttathatók lesznek.