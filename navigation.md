# Grafikus felület: Navigáció

Az Chloros első indításakor a program elindítja a feldolgozó háttérmodult. Amint a háttérmodul készen áll, megjelenik a bal felső sarokban a főmenü ikonja (<img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">), és a bal oldali oldalsávon feloldódnak a „Kamerák” és „Fényérzékelők” lapok (addig el vannak sötétítve).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

A felső fejléc balról jobbra a következőket tartalmazza:

### „<img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">” főmenü

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

A főmenüből a következő műveleteket hajthatja végre:

* **Új projekt**— új projekt létrehozása. Ha mentett projekt-sablonjai vannak, megjelenik a**Sablon kiválasztása** legördülő menü, így az új projekt a sablon beállításait veszi alapul.
* **Projekt megnyitása**— meglévő projekt megnyitása. A listában található egy**Projektmappa megnyitása** gomb is, amely megnyitja a projektek mappáját a fájlkezelőben.
* **Projekt másolása** — a jelenleg megnyitott projektet új név alatt másolja (a rendszer egy szabad nevet javasol, például „MyProject (2)”), majd megnyitja a másolatot. _(a projekt megnyitása után látható)_
* **Fájlok hozzáadása** — egyes képfájlok hozzáadása az aktuális projekthez _(a projekt megnyitása után látható)_
* **Mappa hozzáadása** — egy vagy több képeket tartalmazó mappa hozzáadása az aktuális projekthez _(a projekt megnyitása után látható)_
* **Feldolgozás indítása / leállítása** — elindítja vagy leállítja a képfeldolgozási folyamatot _(a fájlok hozzáadása után elérhető)_
* **Csatlakozás a kamerához** — átugrik a [Kamerák fülre](lattice/), hogy csatlakoztasson egy LATTICE kamerát vagy kamerasorozatot. Nyitott projekt nélkül is működik.
* **Csatlakozás fényérzékelőhöz** — átugrik a [Fényérzékelők fülre](daq/), hogy csatlakoztasson egy DAQ fényérzékelőt. Nyitott projekt nélkül is működik.

{% hint style="info" %}
**Kizárólag Windows**: Az Chloros asztali grafikus felület elérhető az Windows-en. Az Linux felhasználóknak érdemes megtekinteniük az [CLI](CLI.md) és [Python SDK](api-python-sdk.md) dokumentációt a képernyő nélküli feldolgozásról.
{% endhint %}

###<img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">

Lejátszás/Indítás gomb

Ha engedélyezve van, az indítás gomb elindítja a képfeldolgozási folyamatot.

###<img src=".gitbook/assets/image (4).png" alt="" data-size="line">

Haladási sáv<img src=".gitbook/assets/image (5).png" alt="" data-size="line">

Az ingyenes Chloros üzemmódban, amely az összes fájlt egymás után dolgozza fel, a haladási sáv 2 fázist jelenít meg: Célfelismerés és Feldolgozás.

A fizetős, licencelt Chloros+ üzemmódban, amely az összes fájlt párhuzamosan dolgozza fel, a haladási sáv 4 szakaszt jelenít meg: Észlelés, Elemzés, Kalibrálás, Exportálás. Ha az egérmutatót az Chloros+ haladási sáv fölé viszi, megnyílik a kiterjesztett, 4 szakaszból álló haladási sáv panel, így nyomon követheti a folyamatot. A felső haladási sávra kattintva befagyaszthatja a legördülő panelt, újabb kattintással pedig feloldhatja a befagyasztást.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Oldalsó menü

A bal oldali oldalsáv menüje különböző ikonokat tartalmaz, amelyekkel interakcióba léphet, a következő sorrendben fentről lefelé:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Projektbeállítások](project-settings/project-settings.md)

A Projektbeállítások fülön beállíthatja a projekt általános és feldolgozási beállításait. Ezeket a fájlok feldolgozásának megkezdése előtt állítsa be.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Fájlkezelő

Fájlok/mappák hozzáadása és eltávolítása a projektből. Az ismétlődő fájlokat a rendszer figyelmen kívül hagyja. Jelölje be a cél oszlopban a kívánt célképet, és a feldolgozás során a rendszer csak a bejelölt képeket veszi figyelembe célként, ami jelentősen lerövidíti a feldolgozási időt. A „Kép/Metaszolgáltatások” kapcsolóval válthat a kiválasztott képek miniatűr-rácsának és a részletes metadatatáblázatnak a megjelenítése között.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Képnézegető](image-viewer-gui/opening-an-image-full-screen.md)

Ha rákattint egy képre a fő képnézegetőben, az teljes képernyős módban nyílik meg a Képnézegető fülön.

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Térképnézegető](image-viewer-gui/map-markers.md)

Tekintse meg képeit egy interaktív 2D-s térképen a GPS-koordinátáik alapján. Támogatja a Google Maps és az ESRI térképlap-szolgáltatókat, és automatikusan kiválasztja a felhasználó tartózkodási helyéhez legmegfelelőbb szolgáltatást. A jelölők fölé viszve az egérmutatót megtekintheti a képek miniatűr előnézeteit.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Kamerák](lattice/)

Csatlakoztassa és vezérelje a LATTICE kamerákat élőben — egyenként vagy szinkronizált, többkamerás rendszerekként. A fül élő előnézeti csempéket jelenít meg átfedésekkel és hisztogramokkal, kameránkénti és rendszerenkénti beállításokkal, valamint a „Capture All” funkció által kiválasztott kamerákra és exporttípusokra vonatkozó rögzítési beállításokkal. A háttérrendszer készenléte után elérhető; a teljes útmutatót lásd a [LATTICE szakaszban](lattice/).

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Fényérzékelők](daq/)

Csatlakoztassa a DAQ fényérzékelőket — DAQ-U (USB), DAQ-M (Bluetooth) és DAQ-E (Ethernet) —, és tekintse meg azok élő, kalibrált spektrumdiagramjait W/m²/nm-ben. Innen rögzíthet `.daq` fájlokat a megnyitott projektbe, átnevezheti az érzékelőket, kiválaszthat sapkakorrekciós profilokat, és frissítheti a DAQ-E firmware-jét. A háttérrendszer elkészülte után érhető el; a teljes útmutatót a [DAQ szakaszban](daq/) találja.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> hibakeresési napló

Problémák esetén ellenőrizze a naplóban a hibakeresési kimeneteket. Másolja/töltse le a naplót, és küldje el az [MAPIR ügyfélszolgálatnak](https://www.mapir.camera/community/contact) segítségért.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Felhasználói bejelentkezés](chloros+-login.md)

A felhasználói bejelentkezési oldalsáv segítségével bejelentkezhet az Chloros+ fiókjába, hogy hozzáférjen a speciális funkciókhoz. Ezenkívül megtekintheti az alkalmazás aktuális verzióját, valamint beállíthatja a megjelenített szöveg nyelvét az Chloros grafikus felhasználói felületén és az CLI-ben.
