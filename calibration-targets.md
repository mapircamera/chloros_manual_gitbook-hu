---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibrációs célpontok

Az MAPIR különféle kalibrációs célpontokat kínál, amelyek széles körű alkalmazási területeket fednek le. Az alábbi képen látható kompakt T4-R50 modell 4 panelt tartalmaz, amelyek fényvisszaverődését 250–2500 nm tartományban mérték meg.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>A T4 diffúz referencia-célpontok a következő fényvisszaverődési görbéket mutatják, [adatok letöltése itt](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 fényvisszaverődés :: 250–2 500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 visszaverődés :: 400–1000 nm</p></figcaption></figure>A T4P diffúz referenciacélok a következő visszaverődési görbéket mutatják, [adatok letöltése itt](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P fényvisszaverődés :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P reflexió :: 400–1000 nm</p></figcaption></figure>A reflexiós grafikonon látható, hogy az értékek a hullámhossz (x-tengely) és a reflexió százalékos értéke (y-tengely) függvényében vannak ábrázolva. Amikor képet készítünk a kalibrációs célpontról, akkor a kamera egyes érzékelő sávjainak érzékenységi tartományán belül kapcsolatot hozunk létre a képpontérték és a reflexió százalékos értéke között.

Ez azt jelenti, hogy minden, a kameráinkkal rögzített képhez felhasználhatja a fényvisszaverő célpontjaink fényképét, például a [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) vagy a [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) fényvisszaverő célpontjaink fényképét használhatja a képek fényvisszaverődési kalibrálásához. A kalibrálás után a kép minden egyes képpontja megegyezik a százalékos fényvisszaverődéssel.

Az **Survey3** kimenetek esetében, ha a kalibrált képeket az Chloros formátumban szokásos JPG-ként vagy az TIFF formátumban mentjük, akkor a szórás százalékát úgy számoljuk ki, hogy a pixelértéket elosztjuk a képformátum bitmélységével. Tehát JPG esetén 255-tel, TIFF esetén pedig 65 535-tel kell osztani. Választhatja az Chloros fájlban a PERCENT kimeneti formátumot is, amely esetben minden pixel 0,0 és 1,0 közötti százalékos értéket fog felvenni (0%-tól 100%-os fényvisszaverődésig). Csak ne feledje, hogy egyes képkezelő alkalmazások nem fogadják el a százalékos (lebegőpontos) képeket, és ezek méretüknél fogva nagy tárhelyigényűek.

{% hint style="info" %}
**A LATTICE-reflektancia eltérő képpontskálát használ.** A LATTICE-reflektancia úgy kerül tárolásra, hogy DN 32768 = 100%-os reflektancia (nem 65535), és minden fájl tartalmaz egy XMP `Chloros:PixelScale` címkét, amely megadja a skáláját. Olvassa be a címkét, és osztja el vele az értéket, ahelyett, hogy állandót feltételezne – lásd [Kimeneti képformátumok](output-image-formats.md).
{% endhint %}

## Kalibrációs célpontok LATTICE kamerákkal

LATTICE kamerák esetén a kalibrációs célpont a reflexió esetében **opcionális**: az Chloros helyett a reflexiót a DAQ fényérzékelő által mért lefelé irányuló besugárzási intenzitáshoz (ρ = π·L/E) is lehet viszonyítani. A referenciát a reflexió-forrás beállítással választják ki (Projektbeállítások a grafikus felhasználói felületen; `--reflectance-source` az CLI-ben; `reflectance_source` az SDK-ben):

| Érték | Viselkedés |
| --- | --- |
| `auto` *(alapértelmezett)* | A minőségbiztosítási (QA) ellenőrzésen átment, a képkockán belüli célpont az **abszolút referencia**; ha nincs célpont, vagy a QA-ellenőrzés sikertelen, az Chloros a DAQ lefelé irányuló osztására tér vissza. |
| `target` | Szigorúan csak cél — nincs DAQ-helyettesítés. |
| `daq` | DAQ-hatályos — a lefelé irányuló mérés mindig a referencia. |

További célpont-viselkedés a LATTICE esetében:

* **Célpont-geometriák** — Az ArUco-jelöléssel ellátott panelek, a rögzített ROI-panelek és a szalagcélpontok egyaránt támogatottak; a geometria a projekt célpont-konfigurációjából származik.
* **Egységenként mért célpontadatok** — az `--target-reflectance-dir DIR` egy olyan könyvtárra mutat, amely az egységenként mért célpont-reflektancia-letapogatásokat tartalmazza (`<serial>.csv`, a célpont egységének sorozatszáma/QR-kódja alapján keresve). Ha nem találja, az Chloros a névleges T3/T4P spektrumokra tér vissza.
* **Időbeli rögzítés** — egy észlelt célpont kalibrálja a környező képkockákat, és a célpontok észlelései között rögzítve marad.

A jelzők teljes jelentése és példák az [CLI Referenciában](reference/cli-reference.md) találhatók (lásd „Termékenkénti exportkapcsolók”).

### F988

„Az F988 reflektancia kalibrálása a felvételen belüli reflektancia-panel segítségével történik: a sáv a DAQ fényérzékelő kalibrált tartományán kívül esik, ezért az Chloros a legutóbbi panelrögzítést alkalmazza, és azt a panel észlelései között megtartja.”

Ha az F988-at kizárólag DAQ-kalibrálással futtatják, az Chloros elutasítja az adott sáv DAQ-alapú reflektanciáját, és megindokolja az elutasítást (kihagyási ok: `dls-uncalibrated-band-988`); a panel-alapú munkafolyamat a támogatott eljárás.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
