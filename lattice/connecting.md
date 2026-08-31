# Kamerák csatlakoztatása

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>A „Kamerák” fül, mielőtt bármit is csatlakoztatnánk</p></figcaption></figure>Az Chloros automatikusan felismeri a LATTICE kamerákat a kapcsolaton — a grafikus felület Kamerák füléről, az `chloros-cli lattice`-ről, vagy az Python SDK-ből. A kamera modelljének karakterlánca határozza meg az összes további lépést: az Chloros a kamera `DeviceUserID` + `DeviceSerialNumber` adataiból határozza meg az érzékelőprofilját, a sávelrendezést és a gyári kalibrálást, így **kameránként nincs szükség semmilyen beállításra**.

A csatlakozás előtt győződjön meg arról, hogy a gazdagép hálózata be van állítva – link-local címzés, jumbo keretek, valamint tömbök esetén a hálózati kártya fogadó puffereinek beállításai. Ez a hardveroldali beállítás, amelyről a LATTICE kézikönyvben találhat információt: [**Hálózati beállítás**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Csatlakozás a grafikus felhasználói felületről

Nyissa meg a **Kamerák**fület az Chloros oldalsávon (a hardverfülek akkor jelennek meg, amikor a háttérprogram befejezte az indítást), vagy használja a főmenüt →**Csatlakozás kamerához**. Mindkettő megnyitja a**Kamera(k) csatlakoztatása** párbeszédpanelt.

### A **Kamera(k) csatlakoztatása** párbeszédablak

A párbeszédablak megnyitásakor azonnal átvizsgálja a hálózatot („Hálózat átkutatása…”), és felsorolja az összes megtalált kamerát. Minden sorban látható a kamera **modellje**(pl. `LATT-M3M-L41-F550`),**sorozatszáma**és**IP-címe**.

* **Kattintson egy sorra a kijelöléséhez**(zöld kiemelés).**Több kamerát** is kijelölhet, és azokat egyszerre csatlakoztathatja — az Chloros sorrendben csatlakoztatja őket.
* A **„Csatlakoztatva”** jelöléssel ellátott sorok már csatlakoztatva vannak, és nem választhatók ki újra.
* A **„Tömbben”** jelöléssel ellátott sorok egy jelenleg csatlakoztatott kameratömbhöz tartoznak. Először válassza le a tömböt, ha azt a kamerát önállóan szeretné használni.
* **Csatlakozás** — csatlakoztatja a kijelölt kamerákat; a gombon megjelenik a szám, pl. „Csatlakozás (3)”, ha egynél többet jelöl ki.
* **Újra keresés** — újra elindítja a keresést.
* **Bezárás** — bezárja a párbeszédpanelt.
* Ha a keresés eredmény nélkül zárul, a párbeszédpanelen a **„Nincs kamera a hálózaton”** felirat jelenik meg — lásd az alábbi [Hibaelhárítás](connecting.md#troubleshooting) részt.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>A „Kamera(k) csatlakoztatása” párbeszédablak — itt úgy látható, hogy nincs kamera a hálózaton</p></figcaption></figure>### Első csatlakozás: kalibrációs csomag letöltése

Amikor egy adott kamerát **először**csatlakoztatnak egy gépre, az Chloros GigE-n keresztül letölti a kamera gyári kalibrációs csomagját (\~3,8 MB) magából a kamerából. Amíg ez fut, a párbeszédablakban egy zöld**„Kalibrációs adatok letöltése a kameráról”**panel jelenik meg, sorozatszámonkénti haladási sávval — kameránként körülbelül**70 másodperc**re számíthat. A csomagot a gazdagép gyorsítótárába menti, így ugyanazon kamera későbbi csatlakoztatásakor a letöltés teljesen kihagyásra kerül (és a panel soha nem jelenik meg).

### Rendszerelemzés

A párbeszédablak **Rendszerelemzés** gombja megvizsgálja a gazdagépet és a hálózatot (a művelet futása közben a felirat „Elemzés...” feliratot jelenít meg), majd diagnosztikai jelentést készít:

* **Gazdagép** — CPU-magok és RAM; a GPU neve és memóriája, vagy „GPU: Nincs észlelve”.
* **Hálózati interfészek** — az egyes hálózati kártyák neve, kapcsolati sebessége, MTU-ja (aktív esetben „jumbo” jelöléssel), fel/le állapota, valamint az, hogy USB-buszon található-e.
* **Kamerák**— sorozatszám, modell, IP-cím, valamint**hogy melyik hálózati kártyához tartozik az egyes kamerák**.
* **Teljesítmény** — a pixelformátumra vonatkozó aktuális és ideális képkockasebesség (fps) kameránként, zöld „Potenciál: N-szeres javulás lehetséges” sorral, ha az ideális érték meghaladja az aktuálist.
* **Figyelmeztetések és számozott ajánlások** — vagy „A rendszer a jelenlegi kameraszámhoz képest jól működik.”, ha nincs mit kijavítani.

Futtassa akkor, amikor a felismerés vagy a streamelés váratlanul viselkedik — a párbeszédablak elhagyása nélkül azonosítja a legtöbb hálózati kártyával kapcsolatos problémát (helytelen MTU, a kamera rossz interfészen van, USB-adapter korlátai).

### Kameracsoport csatlakoztatása

Ha két vagy több kamerát **szinkronizált kameracsoportként**szeretne csatlakoztatni, használja inkább a kameracsoport-csatlakoztató varázslót (**Kameracsoport csatlakoztatása**): ez végigvezeti a felhasználót a master/slave kiválasztáson (amelyet egy GPIO-vezetékvizsgáló előre kitölt), a megjelenítési mód választásán (Különálló vs. Kombinált mozaikok), valamint egy kameracsoport-beállítási képernyőn, ahol a véglegesítés előtt élőben látható a elérhető képkockasebesség (fps) és a vezetékes sávszélesség. A varázsló és a kameracsoport-munkafolyamatokról a kézikönyv többkamerás kameracsoportokról szóló szakasza szól; az CLI esetében ez a [CLI Referencia](../reference/cli-reference.md) „LATTICE kamera első csatlakoztatási munkafolyamat” című részében található.

## Csatlakozás az CLI és az SDK

Az CLI és az SDK eléréséhez fizetős Chloros+ szint szükséges, és be kell jelentkezni; ezt a rendszer szerveroldalon érvényesíti (`401 AUTH_REQUIRED`, ha nincs bejelentkezve, `403 PLAN_UPGRADE_REQUIRED` az ingyenes csomagban).

```bash
# List cameras on the network (vendor, model, serial, IP, MAC)
chloros-cli lattice info

# Single-camera smoke test: capture one frame (saves every applicable export type)
chloros-cli lattice capture -o output/

# Connect a synchronized array — same smart-prep flow as the GUI
chloros-cli lattice array-connect --serials 213800234,214000533
```

```python
import chloros_sdk

# Persistent live-camera session through the backend
with chloros_sdk.connect_camera("213800234") as cam:
    ...

# Array session (smart-prep: network probe, tier auto-pick, PTP, AE seeding, trigger config)
with chloros_sdk.connect_array(["213800234", "214000533"]) as array:
    ...
```

Teljes aláírások, beállítások és rögzítési munkafolyamatok: [CLI Hivatkozás](../reference/cli-reference.md) § `chloros-cli lattice`, [SDK Hivatkozás](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Hogyan történik a kalibrálás a csatlakozáskor

Minden LATTICE kamera **a kamerán** tárolja a gyári kalibrációs csomagját, és az Chloros a kamera csatlakozásakor ellenőrzi az MAPIR felhőjét is:

| Helyzet   | Mit használ az Chloros                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Online**| Az**adott sorozatszámhoz közzétett legújabb kalibráció** — a felhőben tárolt másolat elsőbbséget élvez a kamerán tárolt másolattal szemben. Az MAPIR által újrakalibrált vagy frissített kamera ezért automatikusan frissíti magát; nincs szükség felhasználói beavatkozásra. |
| **Offline**| A**kamerán tárolt kalibrációs csomag**, változatlan formában. A teljesen offline munkafolyamatok továbbra is működnek; egyszerűen nem veszik át az újabb kalibrációkat, amíg a kamera egyszer online állapotba nem kerül (vagy gyári beállításokra nem állítják vissza).                                                  |

A felvétel készítésekor a ténylegesen alkalmazott együtthatók **be vannak rögzítve az egyes képek XMP-metadatába**. Egy későbbi kalibrációs frissítés soha nem módosítja észrevétlenül a már elkészített felvételeket — egy régi felvétel újrafeldolgozásakor a kép XMP-jébe rögzített együtthatók kerülnek felhasználásra, nem pedig a mai legújabbak.

## Hibaelhárítás

* **„Nem találtak kamerákat a hálózaton”**— ellenőrizze a [Hálózati beállítások](https://mapir.gitbook.io/lattice-camera/setup/network-setup) link-local beállításait: a gazdagép hálózati kártyájának statikus címe `169.254.x.x/16`, a kamerák ugyanazon a kapcsolaton vannak, DHCP/átjáró nem várható. Ezután használja a**Rendszer elemzése**funkciót a csatlakozási párbeszédpanelen, hogy ellenőrizze, melyik hálózati kártyán látható (vagy nem látható) az egyes kamerák. Bármilyen kábelezési vagy hálózati kártya-változtatás után hajtsa végre az**Újrakeresés** műveletet.
* **Egy korábban működő rendszer nem hajlandó csatlakozni** (a tömbpanel kapui az `FRAMES WILL DROP` / `Reduce ROI to enable` kódokkal) — egy hálózati kártya-illesztőprogram frissítése észrevétlenül visszaállította a vételi gyűrű beállításait. Állítsa be őket újra, vagy futtassa az `chloros-cli lattice network --fix` parancsot egy rendszergazdai jogosultságú terminálból; lásd a [Hálózati beállítások](https://mapir.gitbook.io/lattice-camera/setup/network-setup) részt.
* **Egy kamera „In Array” (Tömbben) feliratot jelenít meg** — ez azt jelenti, hogy egy csatlakoztatott tömbmunkamenethez tartozik. A kamera önálló használatához válassza le a tömböt.
