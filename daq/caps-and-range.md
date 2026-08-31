# Kapcsok jellemzői és kalibrált mérési tartomány

> Maguk a kapcsok — azaz hogy melyik érzékelőhöz melyik kapcs tartozik, hogyan szerelhetők fel, valamint optikai viselkedésük — a **[DAQ felhasználói kézikönyvben](https://mapir.gitbook.io/daq)** vannak leírva. Ez az oldal a felszerelt fedél *beállítását* tárgyalja az Chloros számára, ami biztosítja a korrekció helyességét.

Minden DAQ fényérzékelő gyári radiometrikus kalibrálása a *csupasz* érzékelőt írja le. A diffúzorra szerelt fizikai fedél megváltoztatja, hogy az érzékelő milyen fényt gyűjt össze, ezért az Chloros a kalibrációs csomaghoz hozzáad egy gyárilag mért **fedélkorrekciós profilt**. A megfelelő sapka megadása elengedhetetlen a kalibrált adatok megszerzéséhez – ez az oldal bemutatja, hogy mely sapkák léteznek az egyes modellekhez, hogyan kell azokat megadni, és hogy valójában mi az érzékelő kalibrált spektrális tartománya.

## A fedelek elérhetősége modellenként

| Fedélprofil (`cap_id`) | Fizikai fedél | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Napfény-koszinuszkorrektor sapka (**minden modellnél alapértelmezett**) | Igen | Igen | Igen |
| `fov_15` / `fov_45` / `fov_90` | Látómező-korlátozó kúpok (15° / 45° / 90°) | Igen | — | Igen |
| `fov_30` / `fov_60` | Látómezőt korlátozó kúpok (30° / 60°) | Igen | — | — |
| `none` | Nincs felszerelve kupak | — | — | Igen |

Modellspecifikus megjegyzések:

* **A DAQ-M-nek egyetlen kupakprofilja van: `sunshine_cosine`.** A „Bare-plus-Sunshine-cap” a termékdefiníciója, és egy fedél nélküli DAQ-M-hez nincs szükség geometriai profilra.
* **A fedél nélküli DAQ-U valóban fedél nélküli** — egyáltalán nincs szüksége geometriai profilra, ezért nem létezik hozzá `none` profil.
* **A DAQ-E-n található `none` NEM egy no-op.** A DAQ-E süllyesztett, üveggel borított diffúzorának megvan a maga valódi geometriai korrekciója, így a „no cap” (sapka nélkül) állapot önmagában is egy mért profil ezen a modellen.
* Egy **fedél nélküli DAQ-E semmilyen magasságban nem képes mérni a közvetlen napfényt** — a Sunshine fedél a helyszíni konfiguráció. Ne tervezzen kültéri munkát fedél nélküli DAQ-E használatával.

A grafikus felhasználói felület érzékelőnkénti beállításaiban (a „Light Sensors” fülön található fogaskerék ikon) a **Cap** legördülő menü a DAQ-U és DAQ-M modelleknél a „None (bare sensor)” opciót is kínálja — e két modellnél a „bare” kifejezés a fenti megjegyzéseknek megfelelően egyszerűen azt jelenti, hogy nincs sapkakorrekció alkalmazva. Ezt csak akkor válassza, ha a fedelet fizikailag eltávolította.

## A fedél megadása — és miért fontos

**A megadott `cap_id` kódnak meg kell egyeznie azzal a fedéllel, amely fizikailag az érzékelőn van.** Sem az érzékelő, sem a szoftver nem képes felismerni a felszerelt kupakot. A beállítás két dolgot határoz meg:

1. Az egyes spektrumokra alkalmazott **valós idejű korrekciót**.
2. Az **`.daq` felvételekbe beírt kupakjelölést**, amelyre a későbbi reflektancia-feldolgozás támaszkodik.

A Sunshine fedél **tervezés szerint**körülbelül**12-szeres** csillapítást biztosít, így a helytelen fedél megadásával készült felvételek a spektrumokat nagyjából ennyivel torzítják. A fedélváltozásokat azonnal adja meg.

### A fedél beállítása

GUI: Light Sensors fül → fogaskerék ikon az érzékelő sorában → **Cap** legördülő menü. Minden modell alapértelmezett értéke az `sunshine_cosine` (minden DAQ-érzékelőre előre telepítve van a koszinuszkorrektor), és a kiválasztás a projekthez tartozik.

<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->

CLI (a háttérprogramnak futnia kell):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

Az CLI szintaktikailag elfogadja a teljes `cap_id` listát (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); minden profilt a csatlakozáskor az érzékelő modelljéhez viszonyítva ellenőriznek, így egy elérhetetlen kap-azonosító (például egy kizárólag E-típusú azonosító egy DAQ-U-n) egyértelmű hibajelzéssel jár, ahelyett, hogy helytelen korrekciót hajtaná végre. Ha semmit sem adnak át, a háttérprogram alapértelmezett értéke az `sunshine_cosine`.

Python SDK megjegyzés: Az `cap_id` **nem** egy SDK vezérlő — az `connect_daq_sensor()` / `DAQSensorSession` nem tesz közzé cap paramétert. Válassza ki a felső határt a fenti CLI parancsokkal vagy a GUI legördülő menüjéből; lásd az [SDK Referenciát](../reference/sdk-reference.md).

Haladó: a profilok az Chloros telepítőcsomagban az `daq/cap_profiles/<u|m|e>/<cap_id>.json` alatt találhatók, és felhasználónként felülírhatók az `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json` alatt.

A korlátoktól függetlenül azok az érzékelők, amelyeket még soha nem kalibráltak újra, automatikusan megkapnak egy kis, a flottából származó sötét-offset finomítást — ehhez nincs szükség felhasználói beavatkozásra.

## A Sunshine korlát teljesítménye (a kültéri konfiguráció)

Számadatok, amelyekre eljárásokat építhet:

| Tulajdonság | Érték |
| --- | --- |
| Látómező | 180°-os félgömb alakú |
| Kozinusz-válasz hiba | ≤ ±4 % 60°-os beesési szögig; ≤ ±4,5 % 70°-ig |
| Alacsony napállás határ | ~15°-os napmagasság alatt nem ajánlott |
| Csillapítás | ~12× (tervezés szerinti) |
| A sapka újbóli felszerelésének ismételhetősége | ≈ 1,5 % |
| Mennyiségi besugárzás | **≥ 15 másodperc** átlagolt mérési értékek (a műszer jellemzője, nem hiba) |

Bármely mennyiségi besugárzási érték esetében – beleértve a reflexiós referenciaértékeket is – egyetlen képkocka helyett legalább 15 másodperc átlagolt mérési értékét kell használni.

## Kalibrált spektrális tartomány

| Tulajdonság | Érték |
| --- | --- |
| Spektrális mintavétel | 340–1010 nm, 5 nm-es lépésekben (135 pont) |
| Radiometrikusan kalibrált tartomány | **~374–974 nm** (szoftveresen érvényesítve) |

Az érzékelő a teljes 340–1010 nm-es rácsot jelenti, de a NIST-hez nyomon követhető radiometrikus erősítés ~374–974 nm-re terjed ki. Az Chloros **elutasítja az abszolút reflexiós arány felosztását** minden olyan kamerasáv esetében, amelynek spektrális súlyának kevesebb mint a fele esik ebbe a tartományba, és ahelyett, hogy kalibrálatlan eredményt adna, az `dls-uncalibrated-band-<nm>` kihagyási okot jelenti. A forgalomban lévő kamera-SKU-k közül csak az F988 szűrő esik ezen a tartományon kívül; ez helyette a reflexiós panel munkafolyamatot használja — lásd [Reflexiós munkafolyamatok](reflectance.md).

Az érzékelőmodellek, transzportok és érzékelő-azonosítók tekintetében lásd a [DAQ-áttekintést](README.md). A feldolgozás során a cap-bélyeg felhasználásának módjáról lásd a [Felvétel és a .daq formátum](recording.md) című részt.
