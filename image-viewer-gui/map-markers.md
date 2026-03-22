# Térképjelölők

A „Térkép” fül a képeket GPS-koordinátáik alapján egy interaktív 2D-térképen jeleníti meg. Ez földrajzi áttekintést nyújt a felvételi munkamenetről, és segít a térbeli lefedettség vizualizálásában. Hasznos továbbá a képek első importálásakor is, hogy gyorsan eltávolíthassa azokat a képeket, amelyeket nem kell feldolgoznia.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## A Térkép fül eléréséhez

1. Nyisson meg vagy hozzon létre egy projektet az Chloros alkalmazásban
2. Importálja a GPS-metadatákat tartalmazó képeket
3. Kattintson a **Térkép** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> fülre a bal oldali sávban
4. A térkép jelölőket jelenít meg az egyes képek GPS-helyén

{% hint style="info" %}
**GPS szükséges**: Csak azok a képek jelennek meg a térképen, amelyek EXIF metaadatai tartalmaznak beágyazott GPS-koordinátákat. Győződjön meg arról, hogy a fényképezőgépén a felvétel készítése közben be van kapcsolva a GPS.
{% endhint %}

***

## Képek módosítása a Térkép fülről

A **Térkép**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> fülön ugyanaz a hozzáadási  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  és eltávolítás  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  gombokkal rendelkezik, mint a [**Fájlkezelő**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> fülön. Ugyanazt a projektfájl-táblázatot is megjeleníti, de más oszlopfejlécekkel:

### Fájlnév

* A fényképezőgép eredeti fájlneve
* Megtartja a fényképezőgép elnevezési konvencióját (pl. IMG\_0001.RAW)

### Szélesség

* A kép szélessége

### Hosszúság

* A kép hosszúsága

### Magasság

* A kép magassága

{% hint style="info" %}
A táblázat oszlopfejléceire kattintva a sorok adatai is rendeződnek
{% endhint %}

***

## Képjelölők

Minden GPS-adatokkal rendelkező képet egy jelölő jelöl a térképen:

### Jelölők megjelenítése

* A jelölők jelzik az egyes képek felvételének pontos GPS-koordinátáit
* A jelölők csoportosulhatnak, ha kicsinyítjük a képet
* Nagyítson rá a képek egyedi helyszíneinek megtekintéséhez

{% hint style="success" %}
SUPER-ZOOM: Amikor eléri a térképcsempe-szolgáltató maximális nagyítási szintjét, a csempe további nagyításkor tovább nagyul, így láthatja a közel egymáshoz lévő jelölőket.
{% endhint %}

### Előnézet egérmutatóval

* **Vigye az egérmutatót** bármely jelölőre, hogy megtekintse a kép miniatűr előnézetét
* Ez lehetővé teszi a gyors vizuális azonosítást anélkül, hogy elhagyná a térképnézetet
* Hasznos konkrét képek megkereséséhez egy nagy felvételi sorozatban

***

## Térképcsempe-szolgáltatók

{% hint style="success" %}
**Automatikus kiválasztás**: Az Chloros automatikusan kiválasztja azt a csempeszolgáltatót, amely a legjobb nagyítási szintet biztosítja az aktuális térképhelyzethez. Szükség esetén manuálisan is válthat a szolgáltatók között.
{% endhint %}

A Térkép fül két csempeszolgáltatót támogat a háttér térképképekhez:

### Google Maps

* Szabványos műholdas és térképfelvételek a Google-tól
* A legjobb általános világszintű lefedettséghez

### ESRI

* Műholdas és légi felvételek az ESRI ArcGIS-től
* Bizonyos régiókban gyakran nagyobb felbontású felvételeket biztosít

***

## Térképcsempe-típusok

Kiválaszthatja a térkép réteg típusát (balról jobbra):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="original">### Terep

Magassági profilokat és részletes térképcsempéket (utak stb.) jelenít meg

### Térkép

Szabványos (alacsonyabb sávszélességű) térképcsempéket jelenít meg részletekkel (utak stb.)

### Műholdas

Részletes (magasabb sávszélességű) műholdas térképcsempéket jelenít meg

### Hibrid

Műholdas térképcsempéket jelenít meg kiegészítő részletekkel (utak stb.)

***

## Térképnavigáció

### Nagyítás/kicsinyítés vezérlők

* **Nagyítás/kicsinyítés**: Használja az egér görgőjét vagy a nagyítás gombokat
* **Teljes képernyő**: Teljes képernyős térkép

### Képmozgatás vezérlők

* **Képmozgatás**: Kattintson és húzza az egérrel a térképen***

## Használati esetek

### Repülési útvonal megjelenítése

* A drónfelvételek lefedettségi területének megtekintése
* A képek lefedettségében lévő hiányosságok azonosítása
* A repülési útvonal végrehajtásának ellenőrzése

### Földi felmérés áttekintése

* A földi felvételek térbeli eloszlásának megtekintése
* A kalibrációs célképek helyének meghatározása a felmérési területhez viszonyítva
* További felvételi helyszínek tervezése

### Minőség-ellenőrzés

* A váratlan helyeken rögzített képek gyors azonosítása
* A GPS-pontosság ellenőrzése az adatkészletben
* A képek helyszíneinek összevetése a terepi jegyzetekkel

***

## Hibaelhárítás

### Nincsenek jelölők

**Lehetséges okok:**

* A képek nem tartalmaznak GPS-metadatákat
* A rögzítés során a kamera GPS-e ki volt kapcsolva
* Az EXIF-adatokat külső szoftver törölte

**Megoldás**: Ellenőrizze, hogy a fényképezőgépen engedélyezve van-e a GPS, és importálja újra az eredeti fájlokat

### A jelölők rossz helyen vannak

**Lehetséges okok:**

* A fényképezőgép GPS-ének gyenge volt a műholdas pozíciója
* A GPS eltérült a felvétel készítése közben

**Megoldás**: Ez általában a felvétel készítésének idejével kapcsolatos probléma; precíziós alkalmazásokhoz fontolja meg a PPK/RTK GPS használatát
