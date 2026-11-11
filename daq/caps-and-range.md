# Érzékelőburkolatok és kalibrált mérési tartomány

> Maguk a fedelek – hogy melyik érzékelőhöz melyik fedél tartozik, hogyan szerelhetők fel, és milyen az optikai viselkedésük – a **[DAQ felhasználói kézikönyvben](https://mapir.gitbook.io/daq)** vannak dokumentálva. Ez az oldal a felszerelt fedél *bejelentését* a Chloros
számára tárgyalja, ami biztosítja a korrekció helyességét.

Minden DAQ fényérzékelő gyári radiometrikus kalibrációja a *csupasz* érzékelőt írja le. A diffúzorra szerelt fizikai fedél megváltoztatja, hogy az érzékelő milyen fényt gyűjt össze, ezért a Chloros
a kalibrációs csomaghoz hozzáad egy gyárilag mért **fedélkorrekciós profilt**. A megfelelő fedél beállítása elengedhetetlen a kalibrált adatok megszerzéséhez – ez az oldal bemutatja, hogy mely fedelek léteznek az egyes modellekhez, hogyan kell beállítani őket, és mi is valójában az érzékelő kalibrált spektrális tartománya.

## A fedelek elérhetősége modellenként

| Fedélprofil (`cap_id`) | Fizikai fedél | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Napfény-koszinusz-korrektor sapka (**minden modellnél alapértelmezett**) | Igen | Igen | Igen |
| `fov_15` / `fov_45` / `fov_90` | Látómező-korlátozó kúpok (15° / 45° / 90°) | Igen | — | Igen |
| `fov_30` / `fov_60` | Látómező-korlátozó kúpok (30° / 60°) | Igen | — | — |
| `none` | Nincs felszerelve kupak | — | — | Igen |

Modellekhez kapcsolódó megjegyzések:

* **A DAQ-M-nek egyetlen kupakprofilja van: `sunshine_cosine`.** A „Bare-plus-Sunshine-cap” a termék meghatározása, és egy fedél nélküli DAQ-M-hez nincs szükség geometriai profilra.
* **A fedél nélküli DAQ-U valóban fedél nélküli** — egyáltalán nincs szüksége geometriai profilra, ezért nem létezik hozzá `none` profil.
* **A DAQ-E-n található `none` NEM egy „no-op” (hatástalan) profil.** A DAQ-E süllyesztett, üveggel borított diffúzorának megvan a maga saját, valós geometriai korrekciója, így a „no cap” (sapka nélkül) állapot maga is egy mért profil ezen a modellen.
* Egy **fedél nélküli DAQ-E semmilyen magasságban nem képes közvetlen napfényt mérni** — a Sunshine fedél a helyszíni konfiguráció. Ne tervezzen kültéri munkát fedél nélküli DAQ-E használatával.

A grafikus felhasználói felület (GUI) érzékelőnkénti beállításaiban (a „Light Sensors” fülön található fogaskerék ikon) a **Cap** legördülő menü a DAQ-U és DAQ-M modellek esetében a „None (bare sensor)” opciót is kínálja — ezen a két modellen a „bare” kifejezés a fenti megjegyzéseknek megfelelően egyszerűen azt jelenti, hogy nincs sapkakorrekció alkalmazva. Ezt csak akkor válassza, ha a sapkát fizikailag eltávolította.

## A sapka megadása — és miért fontos

**A megadott `cap_id` kódnak meg kell egyeznie az érzékelőn fizikailag található sapkával.** Sem az érzékelő, sem a szoftver nem képes felismerni a felhelyezett kupakot. A beállítás két dolgot határoz meg:

1. Az egyes spektrumokra alkalmazott **valós idejű korrekciót**.
2. Az **`.daq` felvételekbe beírt kupakjelölést**, amelyre a későbbi reflektancia-feldolgozás támaszkodik.

A Sunshine kupak **tervezés szerint**körülbelül**12-szeres** csillapítást biztosít, így a helytelen kupak megadásával készült felvételek a spektrumokat nagyjából ezzel a tényezővel torzítják. A kupakváltozásokat azonnal adja meg.

### A kupak beállítása

GUI: Light Sensors fül → fogaskerék ikon az érzékelő sorában → **Cap** legördülő menü. Minden modell alapértelmezett értéke az `sunshine_cosine`, ami egy feltételezés (a DAQ-U, DAQ-M és DAQ-E modellek fedél nélkül kerülnek forgalomba, és a felvétel addig azt feltételezi, hogy nincs fedél, amíg Ön nem választ egyet), és a kiválasztás a projekttel együtt megmarad.



CLI<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->
(a háttérprogramnak futnia kell):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

A CLI
szintaktikailag elfogadja a teljes `cap_id` listát (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); minden profilt a csatlakozáskor az érzékelő modelljéhez viszonyítva ellenőriznek, így egy elérhetetlen sapka-azonosító (például egy kizárólag E-típusú azonosító egy DAQ-U-n) egyértelmű hibaüzenettel jár, ahelyett, hogy helytelen korrekciót hajtaná végre. Ha semmit sem adnak meg, a háttérprogram alapértelmezett értéke az `sunshine_cosine`.

Python
SDK
: a csatlakozáskor adjon nevet a csatlakozónak az `connect_daq_sensor(cap_id="sunshine_cosine")` paranccsal, vagy futásidőben cserélje ki az `DAQSensorSession.set_cap("fov_45")` paranccsal; lásd az [SDK
-referenciát](../reference/sdk-reference.md#daq-sensor-sessions).

Haladó: a profilok az Chloros
-telepítés részeként az `daq/cap_profiles/<u|m|e>/<cap_id>.json` alatt találhatók, és felhasználónként felülírhatók az `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json` alatt.

A korlátoktól függetlenül azok az érzékelők, amelyeket még soha nem kalibráltak újra, automatikusan kapnak egy kis, a flottából származó sötét-eltolás-finomítást — ehhez nincs szükség felhasználói beavatkozásra.

## DAQ-E-S: rögzített diffúzor és dőlésszög

A **DAQ-E-S** egy DAQ-E, amelynek két, a Chloros
szempontjából fontos különbsége van. Minden egyéb (felismerés, csatlakozás, PTP, rögzítés, eszközbeállítások) pontosan úgy működik, mint egy DAQ-E esetében.

* **Diffúzorja rögzített, ezért nincs szükség fedélkorrekcióra.** A diffúzor már a készüléken volt, amikor a gyári kalibrálását elvégezték, így a kalibrálás már a valódi lefelé irányuló besugárzást mutatja. A Chloros
semmilyen sapka-profilt nem alkalmaz rá, függetlenül attól, hogy mit kérnek tőle: a neki elnevezett sapkát üzenettel figyelmen kívül hagyja, a felvételei szerint a sapka a modellből származik, és a feltételezett sapkára vonatkozó figyelmeztetés soha nem lép működésbe. A Projektbeállítások → DAQ fényérzékelő menüpontban a sorában nincs feltüntetve felső határ, és a projekt-szintű felső határ felülírása sem érinti. Ne alkalmazzon kézzel felső határt az adataira: a Sunshine-korrekció ennek hatására körülbelül 11-szer magasabb értéket jelezne.
* **3-tengelyes gyorsulásmérővel rendelkezik.** A diffúzor koszinusz-válasza csak vízszintes helyzetben érvényes, így egy megdőlt érzékelő más tünet nélkül téves besugárzási értéket jelez. Ezért minden DAQ-E-S-leolvasás tartalmazza a dőlés-, forgás- és emelkedési szögét, és az egyikből rögzített `.daq` ezeket minden spektrum mellé tárolja (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, …), így meg lehet különböztetni a felhőt a megdőlt tartótól.

Az egyes helyzetmezőkkel és a helyzetadat-folyamban fellépő hibák jelentésének módjával kapcsolatban lásd a [DAQ-E-S az CLI
-referenciában](../reference/cli-reference.md#daq-e-s--fixed-diffuser-and-tilt-sensor).

## A Sunshine cap teljesítménye (kültéri konfiguráció)

Számadatok, amelyekre eljárásokat építhet:

| Tulajdonság | Érték |
| --- | --- |
| Látómező | 180°-os félgömb alakú |
| Kozinuszos válaszhiba | ≤ ±4 % 60°-os beesési szögig; ≤ ±4,5 % 70°-ig |
| Alacsony napállás határértéke | ~15°-os napmagasság alatt nem ajánlott |
| Csillapítás | ~12× (tervezés szerinti) |
| A sapka újbóli felszerelésének ismételhetősége | ≈ 1,5 % |
| Mennyiségi besugárzási erősség | **≥ 15 másodperc** átlagolt mérési értékek (a műszer jellemzője, nem hiba) |

Bármely mennyiségi besugárzási erősség érték esetében – beleértve a reflexiós referenciaértékeket is – egyetlen képkocka helyett legalább 15 másodperc átlagolt mérési értékét kell használni.

## Kalibrált spektrális tartomány

| Tulajdonság | Érték |
| --- | --- |
| Spektrális mintavétel | 340–1010 nm, 5 nm-es lépésekben (135 pont) |
| Radiometrikusan kalibrált tartomány | **~374–974 nm** (szoftveresen érvényesítve) |

Az érzékelő a teljes 340–1010 nm-es rácsot jelenti, de a NIST-hez nyomon követhető radiometrikus erősítés ~374–974 nm-re terjed ki. A Chloros
**elutasítja az abszolút reflexió alapján történő felosztást** minden olyan kamerasáv esetében, amelynek spektrális súlyának kevesebb mint a fele esik ebbe a tartományba, és ahelyett, hogy kalibrálatlan eredményt adna, az `dls-uncalibrated-band-<nm>` kihagyási okot jelenti. A forgalomban lévő kameramodellek közül csak az F988 szűrő esik ezen a tartományon kívül; ez a modell helyette a reflexiós panel munkafolyamatot használja — lásd [Reflexiós munkafolyamatok](reflectance.md).

Az érzékelőmodellekről, transzportokról és érzékelő-azonosítókról lásd a [DAQ-áttekintést](README.md). A feldolgozás során a cap-bélyeg felhasználásáról lásd a [Felvétel és a .daq formátum](recording.md) című részt.
