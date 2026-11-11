# Felhasználói felület: Navigáció

A Chloros első indításakor elindul a feldolgozó háttérrendszere. Amint a háttérrendszer készen áll, megjelenik a bal felső sarokban található főmenü ikonja <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">, és a bal oldali oldalsávon feloldódnak a „Kamerák” és „Fényérzékelők” lapok (addig el vannak sötétítve).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

A felső fejléc balról jobbra a következőket tartalmazza:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Főmenü

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>A főmenüből a következő műveleteket hajthatja végre:

* **Új projekt**— új projekt létrehozása. Ha mentett projekt-sablonjai vannak, megjelenik a**Sablon kiválasztása** legördülő menü, így az új projekt a sablon beállításait veszi alapul.
* **Projekt megnyitása**— meglévő projekt megnyitása. A listában található egy**Projektmappa megnyitása** gomb, amely megnyitja a projektek mappáját a fájlkezelőben.
* **Projekt másolása** — az éppen megnyitott projektet új név alatt másolja (a rendszer egy szabad nevet javasol, például „MyProject (2)”), majd megnyitja a másolatot. _(a projekt megnyitása után látható)_
* **Fájlok hozzáadása** — egyes képfájlok hozzáadása az aktuális projekthez _(a projekt megnyitása után látható)_
* **Mappa hozzáadása** — egy vagy több képeket tartalmazó mappa hozzáadása az aktuális projekthez _(a projekt megnyitása után látható)_
* **Feldolgozás indítása / leállítása** — elindítja vagy leállítja a képfeldolgozási folyamatot _(a fájlok hozzáadása után elérhető)_
* **Csatlakozás a kamerához** — átugrik a [Kamerák fülre](lattice/), hogy csatlakoztasson egy LATTICE kamerát vagy kamerasorozatot. Nyitott projekt nélkül is működik.
* **Csatlakozás fényérzékelőhöz** — átugrik a [Fényérzékelők fülre](daq/), hogy csatlakoztasson egy DAQ fényérzékelőt. Nyitott projekt nélkül is működik.

{% hint style="info" %}
**Csak Windows**: A Chloros

Desktop GUI Windows

rendszeren érhető el. A Linux

-felhasználóknak a headless feldolgozáshoz a [CLI

](CLI.md) és a [Python

SDK

](api-python-sdk.md) dokumentációt kell megtekinteniük.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">

Lejátszás/Indítás gomb

Ha engedélyezve van, az indítás gomb elindítja a képfeldolgozási folyamatot.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line">

Haladási sáv <img src=".gitbook/assets/image (5).png" alt="" data-size="line">



Az ingyenes Chloros

módban, amely az összes fájlt egymás után dolgozza fel, a haladási sáv 2 szakaszt jelenít meg: Célfelismerés és Feldolgozás.

A fizetős, Chloros

+ licenccel rendelkező módban, amely az összes fájlt egyszerre dolgozza fel, a haladási sáv 4 szakaszt jelenít meg: Felismerés, Elemzés, Kalibrálás, Exportálás. Ha az egérmutatót a „Chloros

+” haladási sáv fölé viszi, megnyílik a kibővített, 4 szakaszból álló haladási sáv panel, így nyomon követheti a folyamatot. A felső haladási sávra kattintva a legördülő panel befagyasztódik, újabb kattintással pedig feloldódik.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Oldalsó menü

A bal oldali oldalsáv menüje különböző ikonokat tartalmaz, amelyekkel interakcióba léphet, a következő sorrendben fentről lefelé:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Projektbeállítások](project-settings/project-settings.md)

A Projektbeállítások fülön állíthatja be a projekt általános és feldolgozási beállításait. Ezeket a fájlok feldolgozásának megkezdése előtt állítsa be.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Fájlkezelő

Fájlok/mappák hozzáadása és eltávolítása a projektből. Az ismétlődő fájlokat a rendszer figyelmen kívül hagyja. Jelölje be a cél oszlopban a kívánt célképet, és a feldolgozás során a rendszer csak a bejelölt képeket veszi figyelembe célként, ami jelentősen lerövidíti a feldolgozási időt. A „Kép/Metaadatok” kapcsolóval válthat a kiválasztott képek miniatűr-rácsának és a részletes metaadat-táblázatnak a megjelenítése között.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Képnézegető](image-viewer-gui/opening-an-image-full-screen.md)

Ha rákattint egy képre a fő képnézegetőben, az teljes képernyős módban nyílik meg a Képnézegető fülön.

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Térképnézegető](image-viewer-gui/map-markers.md)

Tekintse meg képeit egy interaktív 2D-térképen a GPS-koordinátáik alapján. Támogatja a Google Maps és az ESRI térképlap-szolgáltatókat, és automatikusan kiválasztja a tartózkodási helyéhez legmegfelelőbb szolgáltatást. Vigye az egérmutatót a jelölőkre a képek miniatűr előnézetének megtekintéséhez.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Kamerák](lattice/)

Csatlakoztassa és vezérelje a LATTICE kamerákat élőben — egyenként vagy szinkronizált többkamerás rendszerként. A fülön láthatók az élő előnézeti csempék átfedésekkel és hisztogramokkal, a kameránkénti és rendszerenkénti beállítások, valamint a „Capture All” funkció által kiválasztott kamerákat és exporttípusokat meghatározó rögzítési beállítások. A funkció akkor érhető el, ha a háttérrendszer készen áll; a teljes útmutatót lásd a [LATTICE szakaszban](lattice/).

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Fényérzékelők](daq/)

Csatlakoztassa a DAQ fényérzékelőket — DAQ-U (USB), DAQ-M (Bluetooth) és DAQ-E (Ethernet) —, és tekintse meg azok élő, kalibrált spektrumdiagramjait W/m²/nm-ben. Innen rögzíthet `.daq` fájlokat a megnyitott projektbe, átnevezheti az érzékelőket, kiválaszthatja a sapkakorrekciós profilokat, és frissítheti a DAQ-E firmware-jét. A funkció akkor lesz elérhető, ha a háttérrendszer készen áll; a teljes útmutatót a [DAQ szakaszban](daq/) találja.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Hibaelhárítási napló

Problémák esetén ellenőrizze a naplót a hibaelhárítási kimenetekért. Másolja/töltse le a naplót, és küldje el a [MAPIR ügyfélszolgálatnak](https://www.mapir.camera/community/contact) segítségért.

Újdonság az 1.3.0-s verzióban: a napló fül tetején található hibaikon (**Hiba jelentése**) segítségével jelentést küldhet a MAPIR-nek, amely tartalmazza a leírását, legfeljebb 10 képet, a Chloros-naplókat és a rendszer adatait. A jelentés elküldéséhez be kell jelentkeznie. Lásd: [Hiba jelentése a napló fülről](processing-images-gui/monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Felhasználói bejelentkezés](chloros+-login.md)

A felhasználói bejelentkezési oldalsáv segítségével bejelentkezhet a Chloros+ fiókjába, hogy hozzáférjen a speciális funkciókhoz. Itt megtekintheti az alkalmazás aktuális verzióját, valamint beállíthatja a Chloros grafikus felületén és a CLI-ben megjelenő szöveg nyelvét.
