---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Kimeneti képformátumok A

Chloros a feldolgozott termékeket négy fájlformátumban exportálja. Válassza ki a formátumot a Projektbeállítások (GUI) menüben, az `--format` (CLI) vagy az `export_format` (SDK) parancsokkal. A CLI és az SDK pontosan az alábbi karakterláncokat fogadja el.

| Formátumkarakterlánc | Kiterjesztés | Pixeltípus | Pixeltartomány | Megjegyzések |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(alapértelmezett)* | `.tif` | uint16 szám | 0 – 65535 | Fotogrammetria / GIS esetén ajánlott. |
| `TIFF (8-bit)` | `.tif` | uint8 szám | 0 – 255 | Minden 8 bites felvételhez, amelyek csak 8 bites formátumokat kínálnak: az Chloros nem növeli a bitmélységet. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100%-os fényvisszaverődés. Egyes alkalmazások nem tudják olvasni a lebegőpontos TIFF-fájlokat; a fájlok mérete nagyobb. |
| `PNG (8-bit)` | `.png` | uint8 szám | 0 – 255 | Veszteségmentes tömörítés, alkalmas webes megtekintésre és vizualizálásra. |
| `JPG (8-bit)` | `.jpg` | uint8 szám | 0 – 255 | Veszteséges tömörítés, a legkisebb fájlméret. |

## A kimeneti fájlok helye

A termékek a projektmappa alá kerülnek, kameránként, majd fájlformátum szerint csoportosítva:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

A kamera mappája a LATTICE esetében `LATT-<sensor>-<lens>-F<filter>`, a Survey3 esetében pedig `<model>_<filter>` (pl. `Survey3N_RGN`). **Minden exportált termék megtartja a forrásfájl nevét — a terméket a mappa azonosítja, nem pedig a fájlnév kiterjesztése.** A teljes szabályrendszert lásd a [Kimeneti fájlok helye](reference/cli-reference.md) című részben a CLI-referenciában.

## LATTICE-termékek (rögzítési és exportálási szintek)

Egy LATTICE nyers képkocka egyetlen feldolgozási lépésben minden kért termékre felbomlik. Minden terméktípusnak megvan a saját kapcsolója (GUI jelölőnégyzetek, vagy CLI-ben az `--debayered` / `--preview` / `--radiance` / `--reflectance`, alapértelmezés szerint mind be van kapcsolva):

| Szint | Tartalom | Adattípus |
| --- | --- | --- |
| `raw` | Közvetlenül az érzékelőből származó Bayer-adatok (mono kamerák: az egyetlen sáv). A feldolgozás mindig a nyers adatokból indul. | Rögzítés szerinti |
| `debayered` | Lineáris demosaic — 3 csatorna az M3C esetében, 1 csatorna szürkeárnyalatos az M3M esetében: a kalibrációs csomag sötételtolódásának eltávolítása után élek figyelembevételével történő demosaic, linearitás-korrekció, síkkép-korrekció és színszétválasztás nélkül. Az `process` export minden feldolgozási módban ugyanazt a fájlt írja ki, amelyet a rögzítés ezen a szinten ment. | Lineáris DN |
| `radiance` | Abszolút spektrális sugárzási intenzitás a teljes radiometrikus láncból, **W/m²/sr/nm**-ben. Mindig 32 bites TIFF-ként kerül mentésre (`tiff32/Radiance_Images/`), függetlenül a kiválasztott exportformátumtól. | float32 |
| `reflectance` | Reflektancia ρ, ahol **DN 32768 = ρ 1,0 (100%)**, ρ 2,0-ig terjedő tartalékkal. Pix4D-kompatibilis. | uint16 |
| `preview` | Kijelzőre kész renderelés: RGB = fehéregyensúly + gamma; multispektrális = hamis színű kiterjesztés. | 8 bites kijelző |

### Tájolás

Az `raw` és `debayered` fájlok mindig az érzékelő **eredeti** tájolásában maradnak; a kamera tükrözés/forgatás beállítása csak a többi termékre (előnézet, sugárzás, visszaverődés, index) vonatkozik. A tömbfelvételek minden elemének fájljait natív tájolásban tartják meg, hacsak nem kérsz tájolást (`--orient` a CLI-ben, `orient=True` az SDK-ben). Minden fájl az XMP-ben rögzíti a pixeljeihez tartozó tájolást — `ChlorosRotation`, `ChlorosFlipHorizontal`, `ChlorosFlipVertical` —, míg az `ChlorosOrientationApplied` megadja, hogy a saját képpontjai hordozzák-e azt (`false` nyers és debayerezett fájlok esetén). A Chloros Image Viewer kizárólag a megjelenítés céljából alakítja át a nyers és a debayered fájlokat ezen rekord alapján; maguk a fájlok soha nem kerülnek átírásra. A rekord nélküli fájlok (amelyeket 1.3.0 előtt vagy más eszközzel írtak) a tárolt állapotukban jelennek meg.

## Clamp maszkok

A multispektrális M3C kamera sugárzási és visszaverődési adatai 3×3 sávos szétválasztáson mennek keresztül. Ha a szétválasztás egy sávban negatív értéket eredményez, azt az értéket nullára vágják (az élő nézetek pontosan ugyanígy vágják le az értékeket). Az 1.3.0-s verzió újdonsága, hogy az a termék, amelyben legalább egy pixel le lett vágva, egy kísérő **korlátozó maszkot** kap, amely pontosan megadja, mely pixelekről és sávokról van szó:

* **Hol:**egy `clamp_masks/` mappában**a**termékmappák**mellett**, soha nem azok belsejében: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`, és `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`.
* **Mi:** egy egycsatornás `uint8` TIFF-fájl a termék saját pixelrácsán (tartalmazza a tájolást, a ROI-t, a binninget és a tömb igazítást). A 0. bit (`mask & 1`) ott van beállítva, ahol a termék R-csatornájában a sáv levágásra került, az 1. bit (`& 2`) a G csatornát, a 2. bit (`& 4`) pedig a B csatornát jelöli. A levágott pixelek sávonkénti arányát is bejegyzik a termékbe `Chloros:BandRUnmixClampFraction`, `Chloros:BandGUnmixClampFraction` és `Chloros:BandBUnmixClampFraction` kódokkal.
* **Hogyan jelzi ezt a termék:** az `Chloros:UnmixClampMask` XMP-címke tartalmazza a maszk elérési útját a termék mappájához viszonyítva (`../clamp_masks/<product folder>/<name>_clampmask.tif`, mindig `/`-szel elválasztva), az `none`-et, ha egyetlen pixel sem lett levágva (nem íródott maszk), vagy az `unavailable`-et, ha a maszkot nem lehetett írni. A címke nélküli termékeknél soha nem futott le az unmix művelet (RGB- és monokróm kamerák, nyers és debayered szintek).

Mivel egyetlen termékmappa sem tartalmaz maszkot, egy olyan termékmappát, mint például az `Reflectance_Calibrated_Images/`, a Pix4D, a Metashape vagy más fotogrammetriai szoftver számára változatlan formában átadhatja, az almappákkal együtt — adja át a termékmappát, ne a fölötte lévő formátummappát. A Chloros saját importálója kihagyja a clamp maszkokat. A CLI külön számolja őket: `Image products written: N (plus M clamp masks)`.

## Indextermékek

Minden index a saját `<INDEX>_Index_Images/` mappájába kerül (pl. `NDVI_Index_Images/`).

* **Egész szám formátumok**(`TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`)**képenkénti lineáris nyújtást** tárolnak, nem pedig −1…1 leképezést: a DN 0 az adott képkocka legalacsonyabb indexértéke, a teljes skála pedig a legmagasabb. A fájl tartalmazza az `Chloros:IndexScaling=linear_minmax`, `Chloros:IndexMin` és `Chloros:IndexMax` fájlokat, így az `value = IndexMin + DN / full_scale × (IndexMax − IndexMin)`-et is. Mivel minden képkockának megvannak a saját végpontjai, a képkockákat értékekként kell összehasonlítani, nem pedig DN-ként.
* **`TIFF (32-bit, Percent)`** magukat az indexértékeket float32 típusúként tárolja, átméretezés nélkül — ez a legjobb választás repülésen átívelő kvantitatív munkákhoz.

Az 1.3.0-s verzió újdonsága, hogy egy indextermék **önmagát** írja le, ahelyett, hogy megismételné a forrás nyers adatcímkéit. GPS- és kamera-címkéit abból a termékből másolja át, amelyből kiszámították (a reflexiós termékből, ha van ilyen), majd:

| Címke | Érték |
| --- | --- |
| `Chloros:ProcessingLevel` | `index` (így a fájlt soha nem importálják újra felvételként) |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / a használt képlet |
| `Chloros:IndexLevel` | `reflectance` vagy `radiance` — miből számították ki az indexet (hiányzik, ha egyik sem) |
| `Chloros:IlluminationInvariant` | `True` a reflexióból számított index esetén, `False` a sugárzásból vagy az érzékelőértékekből számított index esetén |
| `Chloros:ChlorosPipelineVersion` | az indexet létrehozó Chloros-verzió |

`Chloros:PixelScale` eltávolításra került az indextermékekből — használja a fenti `IndexMin` / `IndexMax` értékeket. A teljes visszaolvasási recept a [CLI-referencia](reference/cli-reference.md) „Egy egész számú indexraszter kép-alapú kontrasztnyújtás” című részében található.

## A reflexió pixelértékeinek olvasása

A reflexió egész számként van tárolva, és **az a DN-érték, amely ρ = 1,0-t (100%-os reflexiót) jelent, a forráskamerától függ**:

| Forráskamera | ρ = 1,0 megfelelő DN-érték | Hogyan lehet megállapítani |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (tartalék ρ 2,0-ig) | Az `Chloros:PixelScale=32768` XMP-címke szerepel a fájlban. |
| Survey3 | `65535` (ρ 1,0-nál levágva) | Nincs `Chloros:PixelScale` címke — ennek hiánya jelzi a helyzetet. (A Survey3-termékek minden feldolgozási módban hordoznak más `Chloros:*` feldolgozási címkéket, például az `Chloros:CalibrationApplied`-et, a 1.3.0-s verziótól kezdve.) |

**Olvassa be az `Chloros:PixelScale` XMP-címkét, és ossza el vele**, ahelyett, hogy állandót feltételezne. A címke az uint16 tartományban van definiálva, így a kimeneti formátumok átméretezése során is `32768` marad — először normalizálja vissza a tárolt adattípust uint16-ra (8-bites érték esetén ×257-szeresre, float32 esetén ×65535-szeresre).

{% hint style="warning" %}
**8-bites felvételek.** Egy 8-bites felvétel exportált adatai 8-bitesek. Reflektanciáját újraméretezik és `Chloros:PixelScale=32768` címkével látják el, így a méretarány megmarad; A 8-bites DN-termékek, valamint az 1.3.0-ás verzió előtt írt reflektanciafájlok 0–255-re vannak korlátozva, és nem tartalmaznak skála-címkét. Ha a címke hiányzik egy LATTICE reflektanciafájlból, ne feltételezz skálát; ehelyett exportáld újra 16-bites vagy 32-bites formátumban.
{% endhint %}

A teljes szabályrendszer (beleértve a MicaSense-kompatibilis címkéket is) a [CLI-referencia](reference/cli-reference.md) **„Reflektancia-pixelek olvasása”** című részében található.
