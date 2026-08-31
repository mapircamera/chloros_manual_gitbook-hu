# Térképjelölők

A „Térkép” fül a képeket a GPS-koordinátáik alapján egy interaktív 2D-térképre jeleníti meg. Ez földrajzi áttekintést nyújt a felvételi munkamenetről, és az importálás után a leggyorsabb módja annak, hogy eltávolítsa azokat a képeket, amelyeket nem kíván feldolgozni.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## A „Térkép” fül megnyitása

1. Nyisson meg vagy hozzon létre egy projektet az Chloros alkalmazásban
2. Importálja a GPS-metadatokat tartalmazó képeket
3. Kattintson a bal oldali oldalsávon a **Térkép** <img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> fülre
4. A térkép minden kép GPS-helyén egy jelölőt jelenít meg

{% hint style="info" %}
**GPS szükséges**: csak azok a képek jelennek meg a térképen, amelyek EXIF-metaadatai GPS-koordinátákat tartalmaznak. A koordináták nélküli képek továbbra is a projektben maradnak, és a feldolgozásuk is normál módon zajlik – egyszerűen csak nincs hozzájuk jelölő.
{% endhint %}

***

## Képek szerkesztése a „Térkép” fülön

A **Térkép**<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> fülön ugyanazok a „Hozzáadás” <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> és „Eltávolítás” <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line"> fájlgombok találhatók, mint a [**Fájlkezelő**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> fülön. Ugyanazt a projektfájl-listát jeleníti meg, földrajzi oszlopokkal:

| Oszlop        | Tartalom                                                           |
| ------------- | ------------------------------------------------------------------ |
| **Név**      | A fényképezőgépről származó fájlnév                             |
| **Szélesség**  | Tizedes fokok, hat tizedesjegy                                |
| **Hosszúság** | Tizedes fokok, hat tizedesjegy                                |
| **Magasság**  | Méter, egy tizedesjegy — `-`, ha a kép nem tartalmaz magassági adatot |

{% hint style="info" %}
Kattintson bármely oszlopfejlécre a rendezéshez; kattintson újra a sorrend megfordításához.
{% endhint %}

{% hint style="warning" %}
**A magasság a tengerszint feletti magasság, nem a talaj feletti magasság.** Az érték a kép EXIF `GPSAltitude` címkéjéből származik, amely a tengerszintre vonatkozik. Ez nem a terep feletti repülési magasság, és az Chloros nem számítja ki belőle a talajmintavételi távolságot – egy tengerszint feletti magassága 300 m-es mező felett egy 100 m AGL magasságban repülő drón itt körülbelül 400 m-t rögzít. Használja az oszlopot a kiugró értékek kiszűrésére és a repülési magasság következetességének ellenőrzésére, ne pedig AGL-mérésként.
{% endhint %}

***

## Képjelölők

Minden GPS-adatokkal rendelkező képhez egy jelölő tartozik a koordinátáin.

### A jelölők megjelenítése

* A jelölők az egyes felvételekhez rögzített pontos koordinátákon helyezkednek el
* A közel egymáshoz lévő jelölők kicsinyítéskor vizuálisan átfedhetik egymást – nagyítson rájuk, hogy elkülönítse őket
* A kijelölt és kiemelt jelölők a többi felett jelennek meg

### Előnézet egérrel áthúzáskor

* **Húzza az egérmutatót** bármely jelölő fölé, és megjelenik a kép miniatűrje a fájlnévvel együtt
* **Kattintson**egy jelölőre a kép kijelöléséhez, és**rögzítse** a felugró ablakot – ez addig látható marad, amíg máshová nem kattint. Amíg a felugró ablak rögzítve van, más jelölők fölé viszve az egérmutatót az ablak nem tűnik el
* Ez a leggyorsabb módja annak, hogy egy nagy felvételsorozatból egy adott képet megtaláljon anélkül, hogy elhagyná a térképet

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>A „Térkép” fülön a projektben található összes földrajzi címkével ellátott kép megjelenik</p></figcaption></figure>### Szuper-nagyítás

{% hint style="success" %}
**SUPER-ZOOM**: amikor eléri a maximális nagyítást, amelyre a térképrészlet-szolgáltató rendelkezik képanyaggal, a további nagyítás nem áll meg, hanem tovább nagyítja a térképrészleteket, így szét tudja választani az egymásra szinte ráfekvő jelölőket.
{% endhint %}

* A szuper-nagyítás csak akkor lép működésbe, ha **elérte** a szolgáltató által az adott helyre megadott maximális nagyítási szintet, és a csempék betöltése befejeződött. Ezen alatt a nagyítás normál módon működik
* A tartomány **1× és 32×** között van a szolgáltató saját maximális értéke felett
* A sarokban található jelző a jelenlegi szuper-nagyítást százalékban mutatja, a mellette lévő **×** gomb pedig egy kattintással visszatér a normál nagyításhoz
* A kicsinyítés mindig átjut a térképig, így soha nem ragadhat meg a szuper-nagyításban
* Szupernagyítás közbeni nagyítás és eltolás esetén a keletkező eltolás visszakerül a térképre, így a középponttól eltolódott terület továbbra is csempéket kér, ahelyett, hogy üres maradna
* A jelölők vektoros elemekként kerülnek megrajzolásra, nem raszterizálva, így minden szupernagyítási szinten élesek maradnak

***

## Térképcsempe-szolgáltatók

{% hint style="success" %}
**Automatikus kiválasztás**: Az Chloros kiválasztja azt a csempeszolgáltatót, amely a képek helyétől függetlenül a legjobb nagyítási szintet kínálja. Bármikor manuálisan is válthatsz.
{% endhint %}

| Szolgáltató        | Megjegyzések                                                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Google Maps** | Széles körű világszintű lefedettség; mind a négy csempe típust támogat                                                                                                            |
| **Esri ArcGIS**| Bizonyos régiókban gyakran nagyobb felbontású légi felvételek. Az**Terrain** csempe típus nem elérhető az Esri esetében, és a gombja le van tiltva, amíg az Esri van kiválasztva |***

## Térképcsempe-típusok

Válassza ki a térkép réteg típusát a gombokkal (balról jobbra):

![](&lt;../.gitbook/assets/image (14).png&gt;)

| Típus                 | Megjelenít                                                                |
| -------------------- | -------------------------------------------------------------------- |
| **Terrain**          | Magassági árnyékolás térképészeti részletekkel (utak, feliratok). Csak Google       |
| **Map**              | Szabványos utcatérkép-csempék — a legkisebb sávszélességet igénylő opció              |
| **Műholdas**        | Részletes műholdas felvételek, feliratok nélkül — a legnagyobb sávszélességet igénylő opció |
| **Hibrid** (alapértelmezett) | Műholdas felvételek, amelyekre utak és feliratok vannak rajzolva                |

A Térkép fül **Hibrid** módban nyílik meg. A választásod átviszi a szolgáltatóváltást is, amennyiben a szolgáltató támogatja ezt.***

## Térképnavigáció

* **Nagyítás**: az egér görgője vagy a térképen található nagyító gombok
* **Elmozdítás**: kattintás és húzás
* **Teljes képernyő**: a teljes képernyős vezérlővel a térkép kitölti az egész ablakot***

## Használati esetek

### Repülési útvonal áttekintése

* A drónrepülés lefedettségi területének áttekintése egy pillanat alatt
* Azok a hiányosságok kiszűrése, ahol egy átrepülés kimaradt
* Annak ellenőrzése, hogy a repülés a tervezett útvonalat követte-e

### Földi felmérés áttekintése

* A földi felvételek eloszlásának megtekintése
* A kalibrációs célkeretek helyének meghatározása a felmérési területhez viszonyítva
* Döntés arról, hogy hol szükségesek további felvételek

### Minőség-ellenőrzés

* Keresse meg a váratlan helyen rögzített képeket, és távolítsa el őket a feldolgozás előtt
* Rendezés magasság szerint, hogy kiszúrja a rossz magasságban rögzített képeket, vagy azokat, ahol a GPS-jel gyenge volt
* Hasonlítsa össze a képek helyszíneit a terepi jegyzetekkel

***

## Hibaelhárítás

### Nem jelennek meg jelölők

**Lehetséges okok**

* A képek nem tartalmaznak GPS-metadatákat
* A felvétel készítése közben a fényképezőgépen ki volt kapcsolva a GPS
* Az EXIF-adatokat más szoftver eltávolította az importálás előtt

**Mit kell tenni**: ellenőrizze, hogy a fényképezőgépen be van-e kapcsolva a GPS, majd importálja újra az eredeti fájlokat. A „Térkép” fül fájltáblázatában ellenőrizheti, hogy egy adott fájl rendelkezik-e koordinátákkal – a koordináták nélküli képeknél ott nem jelenik meg sor.

### A jelölők rossz helyen vannak

**Lehetséges okok**: gyenge műholdas pozícióhatározás a felvétel készítésekor, vagy a GPS-jel eltérése a felvétel készítése közben.**Mit kell tenni**: ez a felvétel készítésekor felmerült probléma, nem pedig olyan hiba, amelyet az Chloros utólag kijavíthatna. A precíz munkavégzéshez használjon PPK/RTK GPS-munkafolyamatot – lásd a**PPK-korrekciók alkalmazása** beállítást a [Projektbeállítások](../project-settings/project-settings.md) menüpontban.

### A térkép üres, vagy a csempék betöltése leáll

A térképcsempék szolgáltatói online szolgáltatások. Ha a térképcsempék betöltése leáll, ellenőrizze a készülék hálózati kapcsolatát, majd próbáljon meg szolgáltatót váltani. Ha túlzottan nagyított a képet, nyomja meg a **×** visszaállítási gombot, hogy visszatérjen a normál nagyítási szintre, és hagyja, hogy a térkép újra kérje a térképcsempéket.***

## Kapcsolódó oldalak

* [**Képrács**](image-grid.md) — ugyanaz a képsorozat, mint a miniatűröknél
* [**Kép teljes képernyős megnyitása**](opening-an-image-full-screen.md) — egy kép részletes megtekintése
* [**Fájlok hozzáadása egy projekthez**](../processing-images-gui/adding-files-to-a-project.md) — a fülön található fájl hozzáadása/eltávolítása gombok
