---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibrációs célfelületek A

MAPIR számos alkalmazási területet lefedő, különféle kalibrációs célfelületeket kínál. Az alábbi képen látható kompakt T4-R50 modell 4 panelt tartalmaz, amelyek fényvisszaverődését 250–2 500 nm tartományban mérték meg.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>A T4 diffúz referencia-célpontok a következő fényvisszaverődési görbéket mutatják, [adatok letöltése itt](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 fényvisszaverődés :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 fényvisszaverődés :: 400–1000 nm</p></figcaption></figure>A T4P diffúz referencia-célpontok a következő fényvisszaverődési görbéket mutatják, [adatok letöltése itt](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P visszaverődés :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P reflexió :: 400–1000 nm</p></figcaption></figure>A visszaverődési grafikonon látható, hogy az értékek a hullámhossz (x-tengely) és a visszaverődés százalékos aránya (y-tengely) függvényében vannak ábrázolva. Amikor képet készítünk a kalibrációs célpontról, akkor létrehozunk egy összefüggést a képpontérték és a visszaverődés százalékos aránya között azon a spektrumon belül, amelyre a kamera egyes szenzorsávjai érzékenyek.

Ez azt jelenti, hogy minden, kameráinkkal rögzített képhez felhasználhatja a fényvisszaverődési célpontjaink fényképét, például a [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) vagy a [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) fényvisszaverő célpontjaink fényképét használhatja a képek fényvisszaverődés szerinti kalibrálásához. A kalibrálás után a kép minden egyes képpontja megegyezik a százalékos fényvisszaverődéssel.

A **Survey3** kimenetek esetében, ha a kalibrált képeket a Chloros-ban a szokásos JPG vagy TIFF formátumban mentik ki, akkor a reflektancia százalékát úgy számítják ki, hogy a képpont értékét elosztják a képformátum bitmélységével. Tehát JPG esetén 255-tel, TIFF esetén pedig 65 535-tel kell osztani. Chloros ban a PERCENT kimeneti formátumot is kiválaszthatja, ekkor minden pixel 0,0 és 1,0 közötti százalékos értéket fog felvenni (0%–100% fényvisszaverődés). Ne feledje azonban, hogy egyes képszerkesztő programok nem támogatják a százalékos (lebegőpontos) képeket, és ezek a fájlok tárolási szempontból nagy méretűek.

{% hint style="info" %}
**A LATTICE-reflektancia eltérő képpontskálát használ.** A LATTICE-reflektancia tárolásakor a DN 32768 = 100%-os reflektanciát (nem 65535-öt) vesz alapul, és minden fájl tartalmaz egy XMP `Chloros:PixelScale` címkét, amely megadja a skálát. Olvassa be a címkét, és osztja el vele az értéket, ahelyett, hogy állandót feltételezne — lásd [Kimeneti képformátumok](output-image-formats.md).
{% endhint %}

## Kalibrációs célpontok LATTICE kamerákkal

LATTICE kamerák esetén a kalibrációs célpont a reflexió mérése szempontjából **opcionális**: az Chloros e helyette a DAQ fényérzékelő által mért lefelé irányuló besugárzási intenzitáshoz (ρ = π·L/E) viszonyíthatja a reflexiót. A referencia a projekt reflektancia-forrás beállításával választható ki, amelyet az `chloros-cli process --reflectance-source` beállítással (a grafikus felületen nincs erre vonatkozó vezérlőelem, ezért a program a projektben tárolt értéket használja, alapértelmezés szerint az `auto` értéket — lásd [Projektbeállítások](project-settings/project-settings.md)):

| Érték | Viselkedés |
| --- | --- |
| `auto` *(alapértelmezett)* | A minőségbiztosítást (QA) sikeresen teljesítő, a képkockán belüli célpont az **abszolút referencia**; ha nincs célpont, vagy a minőségbiztosítás sikertelen, az Chloros a DAQ lefelé irányuló osztására tér vissza. Az a cél, amely jelzéssel (leesett panel vagy meghibásodott sáv) felel meg a minőségbiztosításnak, nem kerül automatikusan előnyben részesítésre: a cél, a cél+DAQ és a kizárólag DAQ láncokat a kihagyott paneleken mért pontszámuk sorrendjében próbálja ki a rendszer. |
| `target` | Szigorúan csak cél – nincs DAQ-helyettesítés. |
| `daq` | DAQ-domináns – a lefelé irányuló mérés mindig a referencia. |

További célpont-viselkedés a LATTICE esetében:

* **Célpont-geometriák** — Az ArUco-jelölésű panelek, a rögzített ROI-panelek és a szalagcélpontok egyaránt támogatottak; a geometria a projekt célpont-konfigurációjából származik.
* **Egységenként mért célpontadatok** — az `--target-reflectance-dir DIR` egy olyan könyvtárra mutat, amely az egységenként mért célpont-reflektancia-letapogatásokat tartalmazza (`<serial>.csv`, a célpont egységének sorozatszám/QR-kódja alapján keresve). Ha nem talál célt, az Chloros a névleges T3/T4P spektrumokra tér vissza.
* **Időbeli rögzítés** — egy észlelt cél kalibrálja a környező képkockákat, és a cél észlelései között rögzítve marad.

A jelzők teljes jelentése és példái a [CLI-referenciában](reference/cli-reference.md) találhatók (lásd „Termékenkénti exportkapcsolók”).

### F988

„Az F988 reflektancia kalibrálása a jelenetben található reflektancia-panel segítségével történik: a sáv a DAQ fényérzékelő kalibrált tartományán kívül esik, ezért a Chloros a legutóbbi panelrögzítést alkalmazza, és azt a panel észlelései között megtartja.”

Ha az F988-at kizárólag DAQ-kalibrálással futtatják, a Chloros elutasítja az adott sáv DAQ-alapú reflektanciáját, és megindokolja az elutasítást (kihagyási ok: `dls-uncalibrated-band-988`); a panel-alapú munkafolyamat a támogatott eljárás.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
