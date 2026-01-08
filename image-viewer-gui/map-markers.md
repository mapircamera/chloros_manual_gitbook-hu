# Térképjelölők

A Térkép fül a képeket interaktív 2D térképen jeleníti meg a GPS-koordináták alapján. Ez földrajzi áttekintést nyújt a felvételi munkáról, és segít a térbeli lefedettség vizualizálásában. Hasznos lehet a képek első importálásakor is, hogy gyorsan eltávolíthassa azokat a képeket, amelyeket nem kell feldolgoznia.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## A Térkép fül megnyitása

1. Nyisson meg vagy hozzon létre egy projektet az Chloros programban.
2. Importálja a GPS-metadatokkal rendelkező képeket.
3. Kattintson a **Térkép** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> fülre a bal oldali sávban.
4. A térkép minden kép GPS-helyzetén jelölőket jelenít meg.

{% hint style="info" %}
**GPS szükséges**: Csak azok a képek jelennek meg a térképen, amelyek EXIF metaadataiban beágyazott GPS-koordináták találhatók. Győződjön meg arról, hogy fényképezőgépén a felvétel készítése közben a GPS be van kapcsolva.
{% endhint %}

***

## Képek beállítása a Térkép fülön

A **Térkép** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> fülön ugyanazok a hozzáadási  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  és eltávolítási  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  fájl gombokkal rendelkezik, mint a [**Fájlböngésző**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> fül. Ugyanazt a projektfájl-táblázatot is megjeleníti, de más oszlopfejlécekkel:

### Fájlnév

* A fényképezőgép eredeti fájlneve
* Megőrzi a fényképezőgép névadási konvencióját (pl. IMG\_0001.RAW)

### Szélesség

* A kép szélessége

### Hosszúság

* A kép hosszúsága

### Tengerszint feletti magasság

* A kép tengerszint feletti magassága

{% hint style="info" %}
A táblázat oszlopcímkéire kattintva a sorok adatai is rendezhetők.
{% endhint %}

***

## Képmarkerek

Minden GPS-adatokkal rendelkező kép egy markerrel jelenik meg a térképen:

### Marker megjelenítése

* A jelölők az egyes képek felvételének pontos GPS-koordinátáit jelzik.
* A csoportosított jelölők kicsinyítéskor összevonhatók.
* Nagyítás esetén az egyes képek helyszínei láthatók.

{% hint style="success" %}
SUPER-ZOOM: Amikor eléri a térképcsempe-szolgáltató maximális nagyítási szintjét, a csempe további nagyításkor megnagyobbodik, így a közel egymáshoz lévő jelölők is láthatók.
{% endhint %}

### Előnézet az egérrel

* **Vigye az egérmutatót** bármely jelölőre, hogy megtekintse a kép miniatűr előnézetét.
* Ez lehetővé teszi a gyors vizuális azonosítást anélkül, hogy el kellene hagynia a térképnézetet.
* Hasznos nagy méretű felvételek között konkrét képek megkereséséhez.

***

## Térképcsempe-szolgáltatók

{% hint style="success" %}
**Automatikus kiválasztás**: Chloros automatikusan kiválasztja azt a csempe szolgáltatót, amely a legjobb nagyítási szintet biztosítja az aktuális térképhelyzethez. Szükség esetén manuálisan válthat a szolgáltatók között.
{% endhint %}

A Térkép fül két csempe szolgáltatót támogat a háttér térkép képekhez:

### Google Maps

* Szabványos műholdas és térképes képek a Google-tól
* A legjobb általános világszintű lefedettséghez

### ESRI

* Műholdas és légi felvételek az ESRI ArcGIS-től
* Bizonyos régiókban gyakran nagyobb felbontású képeket biztosít

***

## Térképcsempe típusok

Kiválaszthatja a térkép réteg típusát (balról jobbra):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="line">### Terep

Magassági profilokat és részletes térképkockákat (utak stb.) jelenít meg.

### Térkép

Szabványos (alacsonyabb sávszélességű) térképkockákat jelenít meg részletekkel (utak stb.).

### Műhold

Részletes (magasabb sávszélességű) műholdas térképkockákat jelenít meg.

### Hibrid

Műholdas térképkockákat jelenít meg további részletekkel (utak stb.).

***

## Térkép navigáció

### Nagyítás/kicsinyítés vezérlők

* **Nagyítás/kicsinyítés**: Használja az egér görgőjét vagy a nagyítás gombokat.
* **Teljes képernyő**: A térkép teljes képernyős megjelenítése.

### Panoráma vezérlők

* **Panoráma**: Kattintson és húzza az egérrel a térképen.

***

## Használati esetek

### Repülési útvonal vizualizálása

* A drónfelvételek lefedettségi területének megtekintése
* A képek lefedettségének hiányosságainak azonosítása
* A repülési útvonal végrehajtásának ellenőrzése

### Földi felmérés áttekintése

* A földi felvételek térbeli eloszlásának megtekintése
* A kalibrációs célképek helyének meghatározása a felmérési területhez viszonyítva
* További felvételi helyszínek tervezése

### Minőség-ellenőrzés

* Gyorsan azonosítsa a váratlan helyeken rögzített képeket.
* Ellenőrizze a GPS pontosságát az adatkészletben.
* Hasonlítsa össze a képek helyszíneit a terepi jegyzetekkel.

***

## Hibaelhárítás

### Nincsenek jelölők

**Lehetséges okok:**

* A képek nem tartalmaznak GPS-metadatákat.
* A felvétel készítése közben a kamera GPS-e ki volt kapcsolva.
* Az EXIF-adatokat külső szoftver törölte.

**Megoldás**: Ellenőrizze, hogy a kamera GPS-e engedélyezve van-e, és importálja újra az eredeti fájlokat.

### A jelölők rossz helyen vannak

**Lehetséges okok:**

* A kamera GPS-e gyenge műholdas jeleket vett.
* A GPS eltolódott a felvétel készítése közben.

**Megoldás**: Ez általában a felvétel készítésének idejével kapcsolatos probléma; precíziós alkalmazásokhoz fontolja meg a PPK/RTK GPS használatát.
