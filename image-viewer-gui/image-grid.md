# Képrács

A képek projektbe történő importálása után azok rácsban rendezve jelennek meg a főterületen. A rácsban választhatod ki, **hogy az egyes képek melyik változatát szeretnéd megtekinteni** — a rács felett található gombokkal az összes miniatűr egyszerre váltható a forrásfájlok és az egyes feldolgozott változatok között.

## Miniatűrméret

A jobb felső sarokban található nagyítócsúszkával állíthatja be a képminiatűrök méretét. A csúszka **64 px és 1200 px** között mozog.

* **A Ctrl + egérgörgő** kombinációval szintén méretezheti a miniatűröket.
* A **Ctrl + `+`**/**Ctrl + `=`**és**Ctrl + `−`** billentyűkombinációk minden lenyomásra 4 px-rel növelik vagy csökkentik a méretet. A billentyűparancsok segítségével a méret a kisebbik végén 64 px-nél, a nagyobbik végén pedig annál a méretnél áll meg, amely pontosan két miniatűrt foglal el soronként az aktuális ablakban.
* A kiválasztott méret a projekthez kerül mentésre (`UI → Grid thumbnail size` az `project.json`-ben, alapértelmezés szerint `160`), így a projekt újbóli megnyitásakor visszaáll.

<figure><img src="../.gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>A miniatűr *felbontása* a miniatűr *méretétől* független beállítás: lásd a **Kijelző → Képminiatűr felbontása** menüpontot a [Projektbeállítások](../project-settings/project-settings.md) menüben (alapértelmezés szerint 512 px a hosszú oldalon). A méret azt határozza meg, hogy a csempe milyen nagynak jelenik meg; a felbontás pedig azt, hogy mennyi részletet tölt be a csempe kitöltéséhez.***

## A rács eszköztár

A rács feletti gombsor legfeljebb három csoportból áll, balról jobbra:

1. **Triggerenként / Kameránként** — csoportosítási mód. Csak a LATTICE-felvételeket tartalmazó projektekben jelenik meg.
2. **Kameraszűrő gombok**— egy-egy minden LATTICE-kamerához. Csak a**Kameraenként** módban jelenik meg.
3. **Exportálás/megtekintés mód gombok** — hogy melyik termék jelenjen meg az egyes miniatűrökben.

Ha az ablak túl keskeny ahhoz, hogy mindet befogadja, a csoportok jobbról balra legördülő menükbe csukódnak össze: először az exportálási/megtekintési gombok, majd a kamera gombok. Az összecsukott csoportból egyetlen gomb marad hátra, amelyen az éppen aktív választás szerepel, és ha az egérmutatót ráviszi, a teljes készlet lefelé csúszik. **A „Per Trigger” / „Per Camera” csoportok soha nem csukódnak össze.

<!-- SCREENSHOT-NEEDED: Image grid toolbar of a LATTICE array project at full width, showing all three button groups inline: Per Trigger / Per Camera, three camera filter buttons labelled "LATT-M3M (serial)", and the export/view buttons including TIFF, RAW (Original), RAW (Radiance), RAW (Reflectance). -->

*****

## Exportálási és megtekintési gombok

Ezek a gombok váltanak a rácsos miniatűrök között a képformátumok között. **Egy gomb akkor jelenik meg, amint a nevében szereplő termék létezik** — ami a forrásfájlok esetében az importáláskor azonnal megtörténik, nem pedig a feldolgozás után. Az Chloros újraszkenneli a projekt termékeit a futás közben, így a gombok a feldolgozás során jelennek meg, ahogy az egyes termékek elkezdenek a lemezre kerülni.

### Az alapgomb

A bal szélső exportáló gomb felirata **azt jelzi, amit ténylegesen importált**:

| Amit importáltál | Gomb felirata |
| --- | --- |
| Survey3 RAW+JPG | `JPG` |
| LATTICE-felvételek, a nyers képkocka mellett megjelenő előnézettel | `PNG` vagy `TIFF`, attól függően, hogy melyik előnézet jelenik meg |
| A LATTICE olyan felvételeket rögzít, ahol az alapfájl **a** nyers kép | *nincs gomb* — az `RAW (Original)` már megjeleníti azt a fájlt |

Vegyes projektben a felirat azt a kiterjesztést követi, amelyet a legtöbb kép használ.

### Termékgombok

| Gomb | Mit jelenít meg | Mikor jelenik meg |
| --- | --- | --- |
| **Célpontok** | Képek, amelyeken kalibrációs célpontot észleltek | Olyan futtatás után, amely célpontokat észlelt |
| **Reflektancia** | Kalibrált reflektancia-képek | Csak Survey3 projektekben — a LATTICE projektek helyette az `RAW (Reflectance)`-et használják, így a rács soha nem jelenít meg két reflektancia-gombot |
| **Fehér egyensúly** | A fehér egyensúlyú eredmény (RGB kamerák) | Feldolgozás után |
| **Vignettakorrekció** | A nem kalibrált, vignettakorrekcióval ellátott tartalék | Olyan futtatás után, amely során a reflektancia-kalibrálás nem volt alkalmazható, és a *Vignettakorrekció* be volt kapcsolva |
| **Érzékelő-válasz** | A kalibrálatlan, érzékelő-válasz alapérték | Ugyanaz, de kikapcsolt *Vignett-korrekcióval* |
| **`RAW (<INDEX> Index)`** | Egy gomb minden kiszámított indexhez | Indexekkel konfigurált futtatás után |
| **`<INDEX> LUT`** | Egy gomb minden színre leképezett indexhez | LUT-tal konfigurált futtatás után |
| **`<Index> <Index\|LUT> <NNN>`** | Egy gomb minden [Index/LUT Sandbox](index-lut-sandbox.md) exportfutáshoz | Amint befejeződik egy sandbox-export |

### LATTICE-szintű gombok

A LATTICE-felvételeket tartalmazó projektekben ezek is megjelennek, a terméknév helyett a szint nevével jelölve:

| Gomb | Szint |
| --- | --- |
| **RAW (Eredeti)** | Az importált forrás RAW-képkocka |
| **RAW (Sugárzás)** | Float32 spektrális sugárzás, W/m²/sr/nm |
| **RAW (Fényvisszaverődés)** | uint16 fényvisszaverődés, 32768 = ρ 1,0 |

Az `RAW (Original)` az importálás pillanatától elérhető — nincs szükség feldolgozásra. Ha egy LATTICE-importnak egyáltalán nincs alapgombja (minden rögzítés alapfájlja a nyers képkocka), a rács magától az első elérhető szintgombra mozog, hogy az eszköztár kiemelése egyezzen azzal, amit lát.

A két szintű Chloros-exportok **nem kapnak saját rácsgombot**:

* **Debayered** — az `RAW (Original)` nézet már debayered állapotban jelenik meg, így egy második gomb egy vizuálisan azonos képen csak felesleges lenne. Az `RAW (Debayered)` termék továbbra is lemezre kerül, és továbbra is kiválasztható a teljes képernyős réteg legördülő menüből.
* **Előnézet** — az RGB kameráknál az előnézet az `White Balanced` rétegként van regisztrálva, amelynek van gombja. Multispektrális kamerákon `RAW (Preview)` néven van regisztrálva, és a teljes képernyős réteg legördülő menüből érhető el.

{% hint style="info" %}
Ezek a szintgombok csak olyan projekteknél jelennek meg, amelyek ténylegesen tartalmaznak LATTICE képkockákat. Az Survey3 projektek ugyanazokat a belső rétegneveket regisztrálják, és a gombok kiszűrésre kerülnek ezeknél, így egy Survey3 rács megtartja a megszokott `JPG / Targets / Reflectance` készletét.
{% endhint %}

A rács miniatűrjének kattintásával megnyílik a teljes képernyős [Képnézegető](opening-an-image-full-screen.md) **azon a terméken, amelyet a rács ábrázol** — ha a rács beállítása `Targets`, a miniatűr az exportált célképet nyitja meg.

<figure><img src="../.gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: This GIF predates the LATTICE level buttons and the toolbar group separators. Reshoot on a LATTICE project cycling base -> RAW (Original) -> RAW (Radiance) -> RAW (Reflectance) -> an index button, so the new button set and the level names are visible. -->

***

## LATTICE-projekt csoportosítása: Triggerenként vagy kameránként

A tömbfelvételek ugyanazon pillanat több képét rögzítik különböző kameramodulokról. A csoportosítás határozza meg, hogy a rács hogyan rendezi őket egymás fölé. Mindkét mód teljes szélességű, összecsukható fejlécsávokat jelenít meg; **minden csoport kinyitott állapotban indul**, és az Chloros megjegyzi azokat, amelyeket bezár. Az összecsukott állapotot módonként külön követi a rendszer, így ha egy csoportot a „Kameraenként” módban bezár, az nem zár be semmit a „Triggerenként” módban.

### Per Camera (alapértelmezett)

Egy csoport kameramodulonként. A fejlécen látható a kamera modellje és sorozatszáma (`LATT-M3M — <serial>`), valamint a felvételek száma. A csoporton belüli képek időrendben vannak sorolva a rögzítési események szerint.

Ebben az üzemmódban az eszköztáron **kameránként egy kamera-szűrőgomb** is megjelenik, `MODEL (SERIAL)` felirattal. Minden kamera eleve kijelölve van; a gombra kattintva a kamera kijelölése megszűnik, és a csoportja eltűnik a rácsból. Ez a leggyorsabb módszer egy sáv áttekintésére egy teljes repülés során.

### Triggerenként

Egy csoport egy rögzítési eseményhez — az összes modul által ugyanazon a triggeren rögzített képkockák halmaza. A fejlécen látható a rögzítés ideje, a rögzítésben részt vevő kamerák száma, valamint a csoportban szereplő kameramodellek jelölése. A csoporton belüli csempék a kamerák sorozatszáma szerint vannak rendezve, így ugyanaz a sáv minden trigger esetében ugyanabban az oszlopban helyezkedik el.

<!-- SCREENSHOT-NEEDED: Image grid in Per Trigger mode for a 3-camera LATTICE array, showing two consecutive trigger groups with their header bars (chevron, capture timestamp, "3 cameras", and the three model badges) and one group collapsed to show the closed state. -->
A vegyes projektekben található, nem LATTICE-formátumú képek nem kerülnek csoportosításra – ezek a csoportok után egyszerű csempeként jelennek meg.

***

## A rácsos miniatűrök a GSD blokkméretet követik

Ha a Kép fül oldalsávjában beállított egy **GSD (px)** blokkméretet, akkor a rácsos miniatűrök is ugyanazzal a földfelszíni felbontással jelennek meg – nem csak a teljes képernyős nézetben. A 8-as blokkméret azt jelenti, hogy minden megjelenített pixel a forráspixelekből álló 8 × 8-as blokk átlaga, az alkalmazáson belül bárhol, ahol a kép megjelenik.

Mivel egy csempe eleve csak néhány száz pixel széles, a durva blokkméretek már jóval a teljes képernyős nézet előtt nem okoznak látható különbséget a rácson: egy 160 px-es csempébe rajzolt 4000 px-es keretnél már körülbelül 25 forráspixel jut egy megjelenített pixelre. A vezérlőelemről lásd a [Kép teljes képernyős megnyitása](opening-an-image-full-screen.md#gsd-block-size) című részt.

***

## Kapcsolódó oldalak

* [**Kép teljes képernyős megnyitása**](opening-an-image-full-screen.md) — a teljes képernyős néző, a kurzorértékek és a hisztogram
* [**Képrétegek**](image-layers.md) — a rétegek legördülő menüje a teljes képernyős nézőben
* [**Index/LUT Sandbox**](index-lut-sandbox.md) — index-vizualizációk létrehozása és exportálása
* [**Projektbeállítások**](../project-settings/project-settings.md) — az exportálási kapcsolók, amelyek meghatározzák, hogy mely termékek léteznek egyáltalán
