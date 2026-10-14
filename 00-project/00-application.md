---
name: Varga Tamás
neptun: UE6LI3
id: 2026-NP-04
---
# Determinisztikus visszajátszás és állapotszinkronizáció játékokban

## A brief értelmezése

A feladat egy egyszerű 2D-s szimuláción alapuló demonstrátor és mérőkörnyezet elkészítése, amellyel a játékmenet pontos, reprodukálható visszajátszása (replay) vizsgálható. Ahelyett, hogy minden képkockánál a teljes világállapotot lementenénk, a rendszer a felhasználói bemeneteket és eseményeket naplózza fix időléptékekhez (tickekhez) kötve. Ahhoz, hogy a visszajátszás során ugyanaz az eredmény szülessen, kiküszöbölendők a nemdeterminisztikus tényezők, például a változó képkockasebességből adódó időzítési csúszások és a kontrollálatlan véletlenszám-generálás. A rendszer állapot-hash (checksum) számításával figyeli az esetleges szinkronvesztést (desync), ritkított pillanatképek (snapshotok) segítségével pedig lehetővé teszi a visszajátszásban való gyors pozicionálást és a hibás állapotból történő helyreállítást.

## Miért én lennék alkalmas erre a projektre?

Szoftverfejlesztőként kifejezetten érdekelnek az architektúrális problémák, az állapotkezelés és a teljesítményoptimalizálás. Ennél a témánál a fókusz nem a látványos 3D-s grafikán van, hanem a determinisztikus működésen, a szerializáción és a mérhető kompromisszumokon a tárhelyigény és a számítási idő között. Ez a feladat jól illeszkedik a gondolkodásmódomhoz, mert tiszta kódszerkezetet, jól definiált adatszerkezeteket és reprodukálható méréseket kíván meg.

## Releváns tapasztalat és előzmények

A napi munkám során (4 éve dolgozom C# fejlesztőként) C# és .NET környezetben fejlesztek, így magabiztosan mozgok a nyelvben, az objektumorientált tervezésben és az adatmodellezésben. Használtam már WPF-et asztali alkalmazásokhoz, és a Blazort is tanulmányozom, így a megjelenítő és diagnosztikai felület kialakítása nem jelent akadályt. Napi szinten használom a Gitet és a Visual Studio-t, a feladatokat pedig jól elkülöníthető, önállóan tesztelhető komponensekre bontva szoktam felépíteni.

## Tervezett megközelítés

Először egy megjelenítőtől független .NET osztálykönyvtárban valósítom meg a fix időléptékű (tick-alapú) 2D-s szimulációs magot és a saját, seedelhető véletlenszám-generátort. Ezt automatizált tesztekkel ellenőrzöm, hogy azonos bemeneti adatfolyam esetén több ezer lépés után is azonos állapotot kapjak. Ezután építem be a bemenetrögzítőt, az állapot-hash számítást és a konfigurálható gyakoriságú pillanatkép-mentést. Végül elkészítem hozzá a grafikus demonstrátor és vezérlő felületet, ahol vizuálisan követhető a szimuláció, bele lehet tekerni a visszajátszásba, szándékos hibákat (perturbációkat) lehet beinjektálni, és exportálni lehet a futásidejű mérési adatokat.

## Kezdeti terv

1. A szimulációs alapmodell, a bemeneti események és a pillanatkép (snapshot) adatformátumának meghatározása.
2. A specifikáció és a kezdeti technikai javaslat elkészítése.
3. A fix időléptékű szimulációs mag, a kontrollált RNG és a bemenet-alapú visszajátszó prototipizálása C#-ban.
4. Az állapot-hash ellenőrzés és a pillanatkép-kezelés (pozicionálás és helyreállítás) megvalósítása, lefedése egységtesztekkel.
5. A grafikus demonstrátor felület és a hibainjektáló (perturbációs) vezérlőpult elkészítése.
6. Kísérleti mérések lefuttatása különböző pillanatkép-sűrűségek és zavarások mellett, az eredmények kiértékelése.

## További információ

A kezdetektől Git repóban (`https://github.com/Mgodisai/deterministic-replay-lab`) vezetem a munkát, és issue-k segítségével követem a feladatokat. A projektben végig C# (.NET) technológiát használok; a szimulációs logikát külön osztálykönyvtárban valósítom meg, a megjelenítéshez és a diagnosztikai felülethez WPF-et, az automatizált tesztekhez pedig xUnitot használok.