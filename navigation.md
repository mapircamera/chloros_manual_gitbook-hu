# Felhasználói felület: Navigáció

Az Chloros és az Chloros (böngésző) első indításakor a program elindítja a háttérrendszerét. Amint az elkészül, megjelenik a bal felső sarokban található főmenü ikon <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

A felső fejléc balról jobbra a következőket tartalmazza:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Főmenü

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

A főmenüből a következőket teheti:

* **Új projekt** — új projekt létrehozása
* **Projekt megnyitása** — meglévő projekt megnyitása
* **Projektmappa megnyitása** — a projektmappa megnyitása a fájlkezelőben
* **Fájlok hozzáadása** — egyedi képfájlok hozzáadása az aktuális projekthez _(a projekt megnyitása után látható)_
* **Mappa hozzáadása** — képeket tartalmazó mappa hozzáadása az aktuális projekthez _(a projekt megnyitása után látható)_
* **Feldolgozás indítása / Feldolgozás leállítása** — a képfeldolgozási folyamat elindítása vagy leállítása _(a fájlok hozzáadása után engedélyezett)_

{% hint style="info" %}
**Csak Windows**: Az Chloros asztali grafikus felület elérhető az Windows verzión. Az Linux felhasználóknak az [CLI](CLI.md) és a [Python SDK](api-python-sdk.md) dokumentációt a headless feldolgozásról.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> Lejátszás/Indítás gomb

Ha engedélyezve van, az indítás gomb elindítja a képfeldolgozási folyamatot.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> Haladási sáv <img src=".gitbook/assets/image (5).png" alt="" data-size="line">Az ingyenes Chloros módban, amely az összes fájlt egymás után dolgozza fel, a haladási sáv 2 szakaszt jelenít meg: Célfelismerés és Feldolgozás.

A fizetős Chloros+ licencelt módban, amely az összes fájlt egyszerre dolgozza fel, a haladási sáv 4 szakaszt jelenít meg: Felismerés, Elemzés, Kalibrálás, Exportálás. Ha az egérmutatót az Chloros+ haladási sáv fölé viszi, megnyílik a kiterjesztett 4 haladási sáv panel, így követheti a folyamatot. A felső haladási sávra kattintva befagyasztja a legördülő panelt, újabb kattintással pedig feloldja a befagyasztást.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Oldalsó menü

A bal oldali sáv menüje különböző ikonokat tartalmaz, amelyekkel interakcióba léphet:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Projektbeállítások](project-settings/project-settings.md)

A Projektbeállítások fülön állíthatja be a projekt általános és feldolgozási beállításait. Ezeket a fájlok feldolgozásának megkezdése előtt állítsa be.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Fájlböngésző

Fájlok/mappák hozzáadása és eltávolítása a projektből. Az ismétlődő fájlokat a rendszer figyelmen kívül hagyja. Jelölje be a cél oszlopot a kívánt célképeknél, és a feldolgozás csak a bejelölt képeket veszi figyelembe célként, ami jelentősen felgyorsítja a feldolgozási időt. Az Image/Metadata kapcsolóval válthat a kiválasztott képek miniatűr rácsának és a részletes metaadat-táblázatnak a megjelenítése között.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Képmegjelenítő](image-viewer-gui/opening-an-image-full-screen.md)

Ha rákattint egy képre a fő képmegjelenítőben, az teljes képernyős módban nyílik meg a Képmegjelenítő fülön.

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [Térkép](image-viewer-gui/map-markers.md)

Tekintse meg képeit egy interaktív 2D-térképen a GPS-koordinátáik alapján. Támogatja a Google Maps és az ESRI térképszolgáltatókat, automatikusan kiválasztva a helyszínéhez legmegfelelőbb szolgáltatást. Vigye az egérmutatót a jelölőkre a képek miniatűr-előnézetének megtekintéséhez.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Hibakeresési napló

Problémák esetén tekintse át a napló hibakeresési kimeneteit. Másolja/töltse le a naplót, és küldje el a [MAPIR ügyfélszolgálatnak](https://www.mapir.camera/community/contact) segítségért.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Felhasználói bejelentkezés](chloros+-login.md)

A felhasználói bejelentkezés oldalsávján bejelentkezhet az Chloros+ fiókjába, hogy hozzáférjen a speciális funkciókhoz. Megtekintheti az alkalmazás aktuális verzióját, valamint beállíthatja a megjelenített szöveg nyelvét az Chloros GUI-ban és az CLI-ben.
