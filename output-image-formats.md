---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Kimeneti képformátumok

Az Chloros a feldolgozott eredményeket négy fájlformátumban exportálja. Válassza ki a formátumot a Projektbeállításokban (GUI), az `--format` (CLI) vagy az `export_format` (SDK) segítségével. Az CLI és az SDK kizárólag az alábbi karakterláncokat fogadja el.

| Formátumkaraktersorozat | Kiterjesztés | Pixeltípus | Pixeltartomány | Megjegyzések |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(alapértelmezett)* | `.tif` | uint16 digitális szám | 0 – 65535 | Fotogrammetria / GIS esetén ajánlott. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100%-os fényvisszaverődés. Egyes alkalmazások nem tudják olvasni a lebegőpontos TIFF-fájlokat; a fájlok mérete nagyobb. |
| `PNG (8-bit)` | `.png` | uint8 digitális szám | 0 – 255 | Veszteségmentes tömörítés, alkalmas webes megtekintésre és vizualizálásra. |
| `JPG (8-bit)` | `.jpg` | uint8 szám | 0 – 255 | Veszteséges tömörítés, a legkisebb fájlméret. |

## A kimeneti fájlok helye

A fájlok a projektmappa alá kerülnek, kameránként, majd fájlformátum szerint csoportosítva:

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

A kamera mappája a LATTICE esetében `LATT-<sensor>-<lens>-F<filter>`, az Survey3 esetében pedig `<model>_<filter>` (pl. `Survey3N_RGN`). **Minden exportált termék megtartja a forrásfájl nevét – a terméket a mappa azonosítja, nem pedig a fájlnév kiterjesztése.** A teljes szabályrendszert lásd az CLI Referencia [A kimenetek helye](reference/cli-reference.md) című részében.

## LATTICE-termékek (rögzítési és exportálási szintek)

Egy LATTICE nyers képkocka egyetlen feldolgozási lépésben minden kért termékre felbomlik. Minden terméktípusnak megvan a saját kapcsolója (GUI jelölőnégyzetek, vagy CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, alapértelmezés szerint mind BE van kapcsolva):

| Szint | Tartalom | Adattípus |
| --- | --- | --- |
| `raw` | Közvetlenül az érzékelőből származó Bayer-adatok (mono kamerák: az egyetlen sáv). A feldolgozás mindig a nyers adatokból indul ki. | Rögzítéskor |
| `debayered` | Lineáris demosaik — 3 csatorna az M3C esetében, 1 csatornás szürkeárnyalatos az M3M esetében. | Lineáris DN |
| `radiance` | Abszolút spektrális sugárzás a teljes radiometrikus láncból, **W/m²/sr/nm**-ben. Mindig 32 bites TIFF (`tiff32/Radiance_Images/`) formátumban kerül rögzítésre, függetlenül a kiválasztott exportformátumtól. | float32 |
| `reflectance` | Fényvisszaverődés ρ, ahol **DN 32768 = ρ 1,0 (100%)**, ρ 2,0-ig terjedő tartalékkal. Pix4D-kompatibilis. | uint16 |
| `preview` | Kijelzőre kész renderelés: RGB = fehér egyensúly + gamma; multispektrális = hamis színű kiterjesztés. | 8 bites kijelző |

## A reflexió pixelesszámok kiolvasása

A reflexió egész számként van tárolva, és **az a DN-érték, amely ρ = 1,0-t (100%-os reflexiót) jelent, a forráskamerától függ**:

| Forráskamera | ρ = 1,0 megfelelő DN-érték | Hogyan lehet megállapítani |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (tartalék ρ 2,0-ig) | Az `Chloros:PixelScale=32768` XMP-címke be van illesztve a fájlba. |
| Survey3 | `65535` (1,0-nál levágva) | Nincs `Chloros:*` XMP-címke — ez a hiány jelzi a helyzetet. |

**Olvassa be az `Chloros:PixelScale` XMP-címkét, és ossza el vele**, ahelyett, hogy állandót feltételezne. A címke az uint16 tartományban van definiálva, így az átméretező kimeneti formátumokban is változatlan marad — először normalizálja vissza a tárolt adattípust uint16-ra (8-bites érték esetén ×257, float32 esetén ×65535).

{% hint style="warning" %}
**Egy eset tervezés szerint nem tartalmaz skálázást.** Amikor egy 8-bites forrásból származó rögzítést (BayerRG8) 8-bites TIFF formátumban írnak ki, a feldolgozási folyamat átméretezés helyett 0–255-re korlátozza az értékeket, így a fájlt nem jellemzi skála — az Chloros szándékosan kihagyja az `Chloros:PixelScale`-et ebben az esetben. Ha a címke hiányzik egy LATTICE reflektanciafájlból, ne feltételezzünk skálát; ehelyett exportáljuk újra 16-bites vagy 32-bites formátumban.
{% endhint %}

A teljes szabályrendszer (beleértve a MicaSense-kompatibilis címkéket is) megtalálható az [CLI Referencia](reference/cli-reference.md) **„Reflektancia-pixelek olvasása”** című részében.
