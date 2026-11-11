# Grafikus felület: Projektek A

Chloros lehetővé teszi olyan projektek létrehozását, amelyeket később újra megnyithat. A projekt egy egyszerű mappa (a Projektmappa belsejében), amely a következőket tartalmazza:

* `project.json` — projektbeállítások, fájllista és megjelenítési beállítások
* `cameras.json` — a projekt megnyitása közben csatlakoztatott kamerák és érzékelősorok, beállításaikkal együtt
* `sensors.json` — a projekt megnyitása közben csatlakoztatott DAQ fényérzékelők, valamint a kamera↔érzékelő párosítások
* a felvételei, az `.daq` felvételek és a feldolgozott kimeneti mappák

Nincs saját fejlesztésű projektfájlformátum — a mappa és a benne található JSON-fájlok alkotják a projektet, ami egyben megkönnyíti a projektek másolását, archiválását, valamint a [CLI](CLI.md) vagy a [Python SDK](api-python-sdk.md) segítségével történő vezérlését.

Hacsak nem módosítod a [Projektbeállításokban](project-settings/project-settings.md), a projektmappa az `Documents\MAPIR\Chloros Projects` a Dokumentumok mappában (Linux esetén `~/Documents/MAPIR/Chloros Projects`).

{% hint style="warning" %}
**Windows: tartsa rövidnek a projektmappákat, hacsak a hosszú elérési utak nincsenek bekapcsolva.**A Windows alapértelmezés szerint kikapcsolt hosszú elérési utakkal szállít, és ilyenkor egyetlen fájl elérési útja sem lehet hosszabb 260 karakternél. A feldolgozott termékek a projektmappa alatt körülbelül 130 karakterrel helyezkednek el, ezért az 1.3.0-s verziótól kezdve a Chloros elutasítja a túl mélyen elhelyezkedő projektmappákat – akár létrehozáskor, megnyitáskor, akár a feldolgozás megkezdésekor –, ahelyett, hogy futna, de semmit sem írna. Az üzenet megadja a leghosszabb, még elfogadható projektmappát. Használjon rövidebb projektmappát, vagy kapcsolja be a Windows hosszú elérési útjait (a telepítő**Windows hosszú elérési útjainak engedélyezése** opciója, amely alapértelmezés szerint be van jelölve), majd indítsa újra a Chloros programot. Lásd: [A feldolgozás elindítása](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).
{% endhint %}

## Új projekt

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Válassza a „New Project” menüpontot a főmenüből, és adjon meg egy egyedi nevet a projektjének.

Ha mentett már projektsablonokat, a névmező alatt megjelenik egy **Select Template** legördülő menü — ha kiválaszt egyet, az új projekt a kiválasztott sablon beállításai alapján indul el. A sablonokat a [Projektbeállítások](project-settings/project-settings.md) menüpontban mentheti el: írjon be egy nevet a „Projektsablon neve” mezőbe, majd kattintson a mentés ikonra.

## Projekt megnyitása

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>A Projekt megnyitása menüpont a projektmappában található összes projektet felsorolja, alul pedig <strong>a Projektmappa megnyitása gomb</strong> található</p></figcaption></figure>

Válassza a „Projekt megnyitása” lehetőséget a projektmappában található meglévő projektek listájának megtekintéséhez. Ha nincs projekt, a másodlagos oldalsó menü nem nyílik meg. A fenti képen látható néhány, a grafikus felhasználói felületen létrehozott projekt (t1, t2, t3). A DATE\_TIME nevű projekteket a parancssori felület (CLI) hozta létre az alapértelmezett projektnevezési séma szerint. Bármelyik projektnévre kattintva megnyithatja azt.

A „Projektmappa megnyitása” gombra kattintva a számítógép fájlkezelője nyílik meg a projekt elérési útján. A projekt elérési útját a [Projektbeállítások](project-settings/project-settings.md) menüpontban állíthatja be.

Ha a projekt forrásképfájljainak bármelyike az utolsó megnyitás óta áthelyezésre vagy törlésre került, a Chloros nem egy üres rácsot nyit meg, hanem egy párbeszédpanelt jelenít meg, amely pontosan felsorolja, mely fájlok hiányoznak.

## Projekt másolása

Ez a funkció akkor érhető el, ha egy projekt megnyitva van. Válassza a „Projekt másolása” lehetőséget az aktuális projekt új név alatt történő másolásához – a Chloros javaslatot tesz a következő szabad névre (pl. „MyProject (2)”) –, és a másolat azonnal megnyílik.

## Fájlok hozzáadása

A projekt megnyitása után válassza a főmenüből az „Add Files” (Fájlok hozzáadása) menüpontot, hogy egyes képfájlokat adjon hozzá az aktuális projekthez. Ez megegyezik a fájlböngésző hozzáadási funkciójával, de a kényelem érdekében közvetlenül a főmenüből is elérhető.

## Mappa hozzáadása

A projekt megnyitása után válassza a főmenüből a „Mappa hozzáadása” lehetőséget, hogy képeket tartalmazó mappákat adjon hozzá az aktuális projekthez. Egyszerre több mappát is kiválaszthat. Az ismétlődő fájlokat a rendszer figyelmen kívül hagyja.

## Feldolgozás indítása / leállítása

Miután a fájlokat hozzáadta a projekthez, a főmenüben elérhetővé válik a „Feldolgozás indítása” opció. Ez ugyanazt a műveletet jelenti, mint a felső fejlécen található Lejátszás/Indítás gombra kattintás. A feldolgozás során a menüpont „Feldolgozás leállítása”-ra változik, hogy leállíthassa a feldolgozási folyamatot.

## Csatlakozás kamerához / Csatlakozás fényérzékelőhöz

A főmenü alján két hardveres gyorsgomb található, amelyek projekt megnyitása nélkül is elérhetők:

* **Csatlakozás kamerához** — megnyitja a [Kamerák fület](lattice/), hogy csatlakoztasson egy LATTICE kamerát vagy kamerasorozatot.
* **Csatlakozás fényérzékelőhöz** — megnyitja a [Fényérzékelők fület](daq/), hogy DAQ fényérzékelőt csatlakoztasson.

Ha projekt megnyitott állapotában csatlakoztatja a hardvert, az a projektbe kerül mentésre (lásd alább). Projekt nélkül a csatlakozások csak a munkamenetre vonatkoznak.

{% hint style="info" %}
A „Fájlok hozzáadása”, „Mappa hozzáadása” és „Feldolgozás indítása/leállítása” menüpontok csak akkor láthatók vagy használhatók, ha projekt van megnyitva, és fájlokat már hozzáadtak. Gyors hozzáférést biztosítanak azokhoz a műveletekhez, amelyek a Fájlkezelő oldalsávján és a fejléc gombjain keresztül is elérhetők.
{% endhint %}

## A projektek megjegyzik a hardvereket

Újdonság az 1.2.0-s verzióban: a projekt megőrzi a csatlakoztatott hardvereket, amíg nyitva van. A kamerákról és a kamerasorokról (a kameránkénti beállításokkal, nevekkel, színekkel és rácselrendezéssel együtt) pillanatfelvétel készül az `cameras.json` fájlba, a fényérzékelőkről (nevekkel, színekkel és kamerakapcsolatokkal együtt) pedig az `sensors.json` fájlokba – automatikusan, a munka során.

Amikor **újra megnyit** egy projektet, a Chloros nem érinti azonnal a hardvert. Mindkét rész akkor csatlakozik újra, amikor először megnyitja a hozzá tartozó lapot:

* A **Kamerák** lap megnyitásakor a program újra csatlakoztatja a mentett kamerákat és sorozatokat, és újra alkalmazza a mentett beállításokat.
* A **Fényérzékelők**fül megnyitásakor a mentett DAQ-érzékelők újra csatlakoznak, és a rendszer újra alkalmazza a mentett beállításaikat, beleértve a sapkát is. Az a szenzor, amelynek**None (csupasz szenzor)** beállítása van a Cap menüben, csupasz marad. A korábbi Chloros által mentett projekt a csupasz szenzort választási lehetőség nélküliként rögzítette, ezért a napfény-koszinusz-korrekcióval jelenik meg; állítsa be újra None-ra, és mentse el a projektet.

Így a projekt megnyitása – akár csak a képek böngészése vagy exportálása céljából – soha nem indítja el a kamerák adatátvitelét. Ha egy mentett eszköz nem található meg a fül megnyitásakor, egy párbeszédablak jelzi, mely eszközök nem érhetők el, így azokat újra csatlakoztathatja vagy eltávolíthatja.

## DAQ-felvételek és .daq fájlok egy projektben

* A projekt megnyitása közben készített `.daq` felvételek (a Fényérzékelők fülről vagy rögzítés közben) **automatikusan hozzáadódnak a projekthez**.
* Az importált `.daq` fájlok, valamint az összes projektfelvétel a [Projektbeállítások](project-settings/project-settings.md) **DAQ fényérzékelő** szakaszában szerepelnek, mindegyik a saját fényerő-korrekciós profiljával.
* A feldolgozás során a projekt `.daq` fájljai biztosítják a lefelé irányuló megvilágítást a reflexiós termékekhez — lásd a [Kimeneti képformátumok](output-image-formats.md) részt.

## Mentett projekt futtatása grafikus felület nélkül

A mentett projekt futtatható a grafikus felület (GUI) nélkül is:

* **CLI**: Az `chloros-cli project open / connect / capture / sensor / align / run` parancs a projektmappa elérési útját veszi alapul — lásd a [CLI-referenciát](reference/cli-reference.md).
* **SDK**: Az `chloros_sdk.open_project(path)` visszaad egy projekt-azonosítót; az `connect_all()` minden mentett kamerát, szenzortömböt és fényérzékelőt a mentett beállításokkal online állapotba hoz (a fényérzékelők esetében egy futó Chloros háttérprogramon keresztül) — lásd az [SDK-referenciát](reference/sdk-reference.md).
