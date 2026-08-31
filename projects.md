# Grafikus felület: Projektek

Az Chloros lehetővé teszi olyan projektek létrehozását, amelyeket később újra megnyithat. A projekt egy egyszerű mappa (a Projektmappa belsejében), amely a következőket tartalmazza:

* `project.json` — projektbeállítások, fájllista és megjelenítési beállítások
* `cameras.json` — a projekt megnyitása közben csatlakoztatott kamerák és érzékelősorok, beállításaikkal együtt
* `sensors.json` — a projekt megnyitása közben csatlakoztatott DAQ fényérzékelők, valamint a kamera↔érzékelő párosítások
* a felvételei, az `.daq` felvételek és a feldolgozott kimeneti mappák

Nincs saját fejlesztésű projektfájlformátum — a mappa és az abban található JSON fájlok alkotják a projektet, ami egyben megkönnyíti a projektek másolását, archiválását és a [CLI](CLI.md) vagy a [Python SDK](api-python-sdk.md) eszközről.

## Új projekt

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Válassza a „Új projekt” menüpontot a főmenüből, és adjon meg egy egyedi nevet a projektjének.

Ha mentett már projektsablonokat, a névmező alatt megjelenik egy **Sablon kiválasztása** legördülő menü – ha kiválaszt egyet, az új projekt a kiválasztott sablon beállításai alapján indul el. A sablonokat a [Projektbeállítások](project-settings/project-settings.md) menüpontban lehet elmenteni: írjon be egy nevet a „Projektsablon neve” mezőbe, majd kattintson a mentés ikonra.

## Projekt megnyitása

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>A Projekt megnyitása menüpont a projektmappában található összes projektet felsorolja, alul pedig <strong>a Projektmappa megnyitása gomb</strong> található</p></figcaption></figure>Válassza a „Projekt megnyitása” lehetőséget a projektmappában található meglévő projektek listájának megtekintéséhez. Ha nincs projekt, a másodlagos oldalsó menü nem nyílik meg. A fenti képen látható néhány, a grafikus felületen létrehozott projekt (t1, t2, t3). A DATE\_TIME nevű projekteket az CLI hozta létre az alapértelmezett projektnevezési séma szerint. Bármelyik projektnévre kattintva megnyílik az adott projekt.

A „Projektmappa megnyitása” gombra kattintva a számítógép Fájlkezelője a projekt elérési útján nyílik meg. A projekt elérési útját a [Projektbeállítások](project-settings/project-settings.md) menüpontban állíthatja be.

Ha a projekt forrásképfájljainak bármelyike áthelyezésre vagy törlésre került az utolsó megnyitás óta, az Chloros nem egy üres rácsot nyit meg, hanem egy párbeszédpanelt jelenít meg, amely pontosan felsorolja, mely fájlok hiányoznak.

## Projekt másolása

Ez a funkció akkor érhető el, ha a projekt megnyitva van. Válassza a „Projekt másolása” lehetőséget az aktuális projekt új név alatt történő másolásához — az Chloros javaslatot tesz a következő szabad névre (pl. „MyProject (2)”) —, és a másolat azonnal megnyílik.

## Fájlok hozzáadása

A projekt megnyitása után válassza a „Fájlok hozzáadása” menüpontot a főmenüből, hogy egyes képfájlokat adjon hozzá az aktuális projekthez. Ez megegyezik a fájlkezelő hozzáadási funkciójával, de a kényelem érdekében közvetlenül a főmenüből is elérhető.

## Mappa hozzáadása

A projekt megnyitása után válassza a főmenüből a „Mappa hozzáadása” menüpontot, hogy képeket tartalmazó mappákat adjon hozzá az aktuális projekthez. Egyszerre több mappát is kiválaszthat. Az ismétlődő fájlokat a rendszer figyelmen kívül hagyja.

## Feldolgozás indítása / leállítása

Miután a fájlokat hozzáadta a projekthez, a főmenüben elérhetővé válik a „Feldolgozás indítása” opció. Ez ugyanazt a műveletet jelenti, mint a felső fejlécen található Lejátszás/Indítás gombra kattintás. A feldolgozás során a menüpont „Feldolgozás leállítása”-ra változik, hogy leállíthassa a feldolgozási folyamatot.

## Csatlakozás kamerához / Csatlakozás fényérzékelőhöz

A főmenü alján két hardveres gyorsgomb található, amelyek projekt megnyitása nélkül is elérhetők:

* **Csatlakozás kamerához** — megnyitja a [Kamerák fület](lattice/), hogy csatlakoztasson egy LATTICE kamerát vagy kamerasorozatot.
* **Csatlakozás fényérzékelőhöz** — megnyitja a [Fényérzékelők fület](daq/), hogy DAQ fényérzékelőt csatlakoztasson.

Ha projekt megnyitott állapotában csatlakoztatja a hardvert, az a projektbe kerül mentésre (lásd alább). Projekt hiányában a csatlakozások csak a munkamenetre vonatkoznak.

{% hint style="info" %}
A „Fájlok hozzáadása”, „Mappa hozzáadása” és „Feldolgozás indítása/leállítása” menüpontok csak akkor láthatók vagy aktiválhatók, ha nyitva van egy projekt, és fájlokat már hozzáadtak. Gyors hozzáférést biztosítanak azokhoz a műveletekhez, amelyek a Fájlkezelő oldalsávján és a fejléc gombjain keresztül is elérhetők.
{% endhint %}

## A projektek megjegyzik a hardverét

Újdonság az 1.2.0-ás verzióban: a projekt megőrzi a csatlakoztatott hardvereket, amíg nyitva van. A kamerákról és a kamerasorokról (a kameránkénti beállításokkal, nevekkel, színekkel és rácselrendezéssel együtt) pillanatfelvétel készül az `cameras.json` fájlba, a fényérzékelőkről (nevekkel, színekkel és kamerakapcsolatokkal együtt) pedig az `sensors.json` fájlokba – automatikusan, a munka során.

Amikor **újra megnyit** egy projektet, az Chloros fájl nem érinti azonnal a hardvert. Mindkét rész újra csatlakozik, amikor először megnyitja azt a lapot, amelyhez tartozik:

* A **Kamerák** fül megnyitása újra csatlakoztatja a mentett kamerákat és tömböket, és újra alkalmazza azok mentett beállításait.
* A **Fényérzékelők** fül megnyitása újra csatlakoztatja a mentett DAQ-érzékelőket.

Így a projekt megnyitása pusztán a képek böngészése vagy exportálása céljából soha nem indítja el a kamerák adatfolyamát. Ha egy mentett eszköz nem található meg a fül megnyitásakor, egy párbeszédablak jelzi, mely eszközök nem érhetők el, így azokat újra csatlakoztathatja vagy eltávolíthatja.

## DAQ-felvételek és .daq fájlok egy projektben

* A projekt megnyitása közben készített `.daq` felvételek (a **Fényérzékelők**fülről vagy rögzítések során)**automatikusan hozzáadódnak a projekthez**.
* Az importált `.daq` fájlok, valamint az összes projektfelvétel a [Projektbeállítások](project-settings/project-settings.md) **DAQ fényérzékelő** szakaszában szerepelnek, mindegyik a saját fényerő-korrekciós profiljával.
* A feldolgozás során a projekt `.daq` fájljai biztosítják a lefelé irányuló megvilágítást a reflexiós termékekhez — lásd a [Kimeneti képformátumok](output-image-formats.md) részt.

## Mentett projekt futtatása grafikus felület nélkül

Egy mentett projekt a grafikus felhasználói felület (GUI) nélkül is futtatható:

* **CLI**: Az `chloros-cli project open / connect / capture / sensor / align / run` egy projektmappa elérési útján működik — lásd az [CLI Referencia](reference/cli-reference.md).
* **SDK**: Az `chloros_sdk.open_project(path)` egy projekt-azonosítót ad vissza; az `connect_all()` minden mentett kamerát és érzékelőt a mentett beállításokkal online állapotba hoz – lásd az [SDK referencia](reference/sdk-reference.md).
