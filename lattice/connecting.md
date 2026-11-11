# Kamerák csatlakoztatása

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>A „Kamerák” fül, mielőtt bármit is csatlakoztatnánk</p></figcaption></figure>A

Chloros automatikusan felismeri a hálózaton lévő LATTICE kamerákat — a grafikus felület „Kamerák” füléről, az `chloros-cli lattice` fájlból vagy a Python SDK-ből. A kamera modelljének karakterlánca határozza meg az összes további lépést: a Chloros a kamera `DeviceUserID` + `DeviceSerialNumber` fájljaiból határozza meg az érzékelő profilját, a sávelrendezést és a gyári kalibrációt, így **kameránként nincs szükség semmilyen konfigurálásra**.

A csatlakozás előtt győződjön meg arról, hogy a gazdagép hálózata be van állítva — link-local címzés, jumbo keretek, valamint tömbök esetén a hálózati kártya vételi pufferének beállításai. Ez a hardveroldali beállítás, amelyről a LATTICE kézikönyvben találhat információt: [**Hálózati beállítás**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Csatlakozás a grafikus felhasználói felületről

Nyissa meg a **Kamerák** fület a Chloros oldalsávjában (a hardverfülek akkor jelennek meg, amikor a háttérprogram befejezte az indítást), vagy használja a főmenüt → **Csatlakozás a kamerához**. Mindkét esetben megnyílik a**Kamera(k) csatlakoztatása** párbeszédablak.

### A **Kamera(k) csatlakoztatása** párbeszédablak

A párbeszédablak megnyitásakor azonnal átvizsgálja a hálózatot („Hálózat átkutatása…”), és felsorolja az összes megtalált kamerát. Minden sorban látható a kamera **modellje**(pl. `LATT-M3M-L41-F550`),**sorozatszáma**és**IP-címe**.

* **Kattintson egy sorra a kijelöléséhez**(zöld kiemelés).**Több kamerát** is kiválaszthat, és egyszerre csatlakoztathatja őket — a Chloros egymás után csatlakoztatja őket.
* A **„Csatlakoztatva”** jelöléssel ellátott sorok már csatlakoztatva vannak, és nem választhatók ki újra.
* A **„Tömbben”** jelöléssel ellátott sorok egy jelenleg csatlakoztatott kameratömbhöz tartoznak. Először válassza le a rendszert, ha azt a kamerát önállóan szeretné használni.
* **Csatlakozás** — csatlakoztatja a kijelölt kamerákat; a gombon megjelenik a darabszám, pl. „Csatlakozás (3)”, ha egynél többet jelöl ki.
* **Újraolvasás** — újra elindítja a felderítést.
* **Bezárás** — bezárja a párbeszédpanelt.
* Ha a keresés eredmény nélkül zárul, a párbeszédpanelen a **„Nincs kamera a hálózaton”** felirat jelenik meg — lásd az alábbi [Hibaelhárítás](connecting.md#troubleshooting) részt.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>A „Kamera(k) csatlakoztatása” párbeszédablak — itt úgy látható, hogy nincs kamera a hálózaton</p></figcaption></figure>

### Első csatlakozás: kalibrációs csomag letöltése

Amikor egy adott kamerát **először** csatlakoztatnak egy gépre, a Chloros GigE-n keresztül letölti a kamera gyári kalibrációs csomagját (\~3,8 MB) magából a kamerából. Amíg ez fut, a párbeszédpanelen egy zöld **„Kalibrációs adatok letöltése a kameráról”**felirat jelenik meg, sorozatszámonkénti haladási sávval — kameránként körülbelül**70 másodperc**re számíthat. Hatnál több kamera esetén a haladási sorok a panel belsejében görgetődnek. A csomagot a gazdagép gyorsítótárába menti, így ugyanazon kamera későbbi csatlakoztatásakor a letöltés teljesen kihagyásra kerül (és a panel soha nem jelenik meg).

### Rendszerelemzés

A párbeszédpanel **Rendszerelemzés** gombja megvizsgálja a gazdagépet és a hálózatot (a művelet futása közben a felirat „Elemzés...” szöveget jelenít meg), majd diagnosztikai jelentést készít:

* **Gazdagép** — CPU-magok és RAM; GPU neve és memóriája, vagy „GPU: Nem észlelve”.
* **Hálózati interfészek** — az egyes hálózati kártyák neve, kapcsolati sebessége, MTU-ja (aktív esetben „jumbo” jelöléssel), fel/le állapota, valamint az, hogy USB-buszon található-e.
* **Kamerák**— sorozatszám, modell, IP-cím, valamint**hogy melyik hálózati kártyára van csatlakoztatva az egyes kamerák**.
* **Teljesítmény** — az aktuális és az ideális képkockasebesség (fps) kameránként a pixelformátum szerint; zöld „Potenciál: N×-szeres javulás lehetséges” sor jelenik meg, ha az ideális érték meghaladja az aktuálisat.
* **Figyelmeztetések és számozott ajánlások** — vagy „A rendszer a jelenlegi kameraszámhoz képest rendben van.”, ha nincs mit kijavítani.

Futtassa ezt, ha a felismerés vagy a streamelés váratlanul viselkedik — a párbeszédablak elhagyása nélkül azonosítja a legtöbb hálózati kártyával kapcsolatos problémát (helytelen MTU, a kamera rossz interfészen van, USB-adapter korlátai).

### Kameracsoport csatlakoztatása

Ha két vagy több kamerát **szinkronizált kameracsoportként**szeretne csatlakoztatni, használja inkább a kameracsoport-csatlakoztató varázslót (**Kameracsoport csatlakoztatása**): ez végigvezeti a mester/szolga kiválasztáson (amelyet egy szinkronkábel-érzékelő előre kitölt — bármely kábelezett kamera lehet mester, és az RGB `-FRGB` kamera előre kiválasztásra kerül, ha kábelezve van), a megjelenítési mód kiválasztásán (Különálló vs. Összevont mozaikok), valamint egy kameracsoport-beállítási képernyőn, ahol a véglegesítés előtt élőben látható a elérhető képkockasebesség (fps) és a vezetékes sávszélesség. A varázsló és a kamerarendszer-munkafolyamatok leírását a kézikönyv többkamerás rendszerekről szóló fejezete tartalmazza; a CLI-ben ennek megfelelője a [CLI-referencia](../reference/cli-reference.md) „LATTICE Camera First-Connect Workflow” című része.

## Csatlakozás a CLI-ről és az SDK-ről A

CLI-hez és az SDK-hez való hozzáféréshez fizetős Chloros+ szint szükséges, valamint be kell jelentkezni; ezt szerveroldalon érvényesítik (`401 AUTH_REQUIRED` bejelentkezés nélkül, `403 PLAN_UPGRADE_REQUIRED` az ingyenes szinten).

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

Teljes aláírások, opciók és rögzítési munkafolyamatok: [CLI-referencia](../reference/cli-reference.md) § `chloros-cli lattice`, [SDK-referencia](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Hogyan történik a kalibrálás a csatlakozáskor

Minden LATTICE kamera **a kamerán belül** hordozza a gyári kalibrációs csomagját, és a Chloros a kamera csatlakozásakor ellenőrzi a MAPIR felhőjét is:

| Helyzet   | Mit használ a Chloros                                                                                                                                                                                                           |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Online**| Az**adott sorozatszámhoz közzétett legújabb kalibráció** — a felhőben tárolt másolat elsőbbséget élvez a kamerán tárolt másolattal szemben. Az MAPIR által újrakalibrált vagy frissített kamera ezért automatikusan frissíti magát; nincs szükség felhasználói beavatkozásra. |
| **Offline**| A**kamerán található csomag**, változatlanul. A teljesen offline munkafolyamatok továbbra is működnek; egyszerűen nem veszik át az újabb kalibrációkat, amíg a kamera egyszer online nem lesz (vagy gyári állapotba nem állítják vissza).                                                  |

A felvétel készítésekor a ténylegesen alkalmazott együtthatók **be vannak fagyva az egyes képek XMP-metadatába**. Egy későbbi kalibrációs frissítés soha nem módosítja észrevétlenül a már rögzített képeket — egy régi felvétel újrafeldolgozásakor a kép XMP-jébe bejegyzett együtthatók kerülnek felhasználásra, nem pedig a mai legújabbak.

## Hibaelhárítás

* **„Nincs kamera a hálózaton”**— ellenőrizze a [Hálózati beállítások](https://mapir.gitbook.io/lattice-camera/setup/network-setup) link-local beállításait: a gazdagép hálózati kártyájának statikus címe `169.254.x.x/16`, a kamerák ugyanazon a kapcsolaton vannak, DHCP/átjáró nem várható. Ezután használja a**Rendszer elemzése**funkciót a csatlakozási párbeszédpanelen, hogy ellenőrizze, melyik hálózati kártyán látható (vagy nem látható) az egyes kamerák. Bármilyen kábelezési vagy hálózati kártya-cserét követően hajtsa végre az**Újrakeresés** műveletet.
* **Egy korábban működő rendszer nem hajlandó csatlakozni** (a panel kapui az `FRAMES WILL DROP` / `Reduce ROI to enable` hibakóddal jelzik) — egy hálózati kártya-illesztőprogram frissítése észrevétlenül visszaállította a vételi gyűrű beállításait. Állítsa be őket újra, vagy futtassa az `chloros-cli lattice network --fix` parancsot egy rendszergazdai jogosultsággal rendelkező terminálról; lásd [Hálózati beállítások](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Egy kamera „In Array” (Tömbben) feliratot jelenít meg** — ez egy csatlakoztatott tömbmunkamenethez tartozik. A kamera önálló használatához válassza le a tömböt.
