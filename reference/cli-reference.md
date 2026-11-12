# Chloros CLI-referencia

**Verzió:**

1.3.0**Létrehozás:**

2026.07.29. 19:19 ·**Frissítés:**

2026.11.06.**Célközönség:** LLM-ek számára optimalizált; ember számára is olvasható.**Hatály:** Az `chloros-cli` minden felhasználói alparancsa, opciókkal és másolható példákkal.

Ez a dokumentum a MAPIR Chloros szoftverhez mellékelt `chloros-cli` parancssori eszköz teljes referencia-útmutatója. Szándékosan kimerítő, hogy egy LLM (vagy ember) a forráskód áttekintése nélkül is összeállíthassa az alábbi listákból bármely támogatott munkafolyamatot.

Ha csak a legfontosabb információkra van szüksége, ugorjon az alábbiakhoz:
- [Ötperces gyorsindítás](#five-minute-quickstart)
- [LATTICE kamera első csatlakozási munkafolyamat](#lattice-camera-first-connect-workflow)
- [DAQ-érzékelő első csatlakozási munkafolyamat](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Rögzítési módok, rögzítők és offline újrafeldolgozás](#capture-modes-recorders--offline-reprocess)

> **Chloros vezérlése AI-asszisztens segítségével?** A Claude Desktop, a Claude Code, a Cursor és más MCP-kliensek az `chloros-mcp`-en keresztül futtathatják a feldolgozást, a LATTICE kamerákat és sorozatokat, valamint a DAQ fényérzékelőket, az MCP-kiszolgálón keresztül, amely a Python SDK-vel együtt települ (Linux esetén a .deb csomagban, az `/usr/bin` fájlban): `claude mcp add chloros -- chloros-mcp`. A beállítások, opciók és eszközök az SDK-referencia [MCP-kiszolgáló (AI-asszisztensek)](sdk-reference.md#mcp-server-ai-assistants) című részében találhatók. A parancssorral rendelkező asszisztens ezen az oldalon szereplő minden parancsot közvetlenül is végrehajthat.

> **Változás az 1.3.0-ás verzióban** (amit egy meglévő 1.2.0-ás szkript vagy munkafolyamat észlel).
>
> **Változások, amelyekkel egy 1.2.0-ás szkriptnek esetleg meg kell birkóznia:**
>
> - **Az `--preset` érvényes.** `lattice capture`, `continuous`, `calibrate`, `multi-capture`, `multi-stream`, `align-calibrate`, `align-apply` és `align-stream` a 1.2-es verzióban eltávolításra került.0-ban eltávolították, és a kamera saját beállításai alapján működött. Most már beállítja az előre beállított profil formátumát, expozícióját, erősítését és képkockasebességét: az `--preset high_speed` 8 bites felvételt készít 5 ms-os időközönként, 6 dB-es erősítéssel, 60 fps sebességgel rögzít. Az `lattice cam-connect --preset` szintén megtartja az előre beállított érték formátumát (az 1.2.0-s verzióban a kamera 12 bitesre állt át az előre beállított érték alkalmazása után). Lásd [Általános beállítások](#common-options-shared-by-most-subcommands).
> - **Az M3M (mono) kamerán a Bayer-pixelformátum egyben a mélységet is jelenti.** Az előre beállított értékek olyan Bayer-formátumokat neveznek meg, amelyeket egy mono érzékelő nem tud rögzíteni: az 1.2.0-s verzióban a kamera elutasította a formátumot, és megtartotta a korábbi beállítást, így az `--preset high_speed` ott nem volt 8 bites. Most már ugyanolyan mélységű Mono formátumról van szó: az `high_speed` és az `--pixel-format BayerRG8` az `Mono8`-et, az `high_quality`, az `triggered` és az `BayerRG12` együtt az `Mono12`-et adja. Egy vegyes M3C + M3M tömbben minden kamera a saját családján belüli közös formátumot veszi át, és az `network-analysis` / `analyze-array --pixel-format` kódolja az egyes kamerákat az adott mélységben. Ugyanez fordítva is igaz: egy Mono formátumot kérő M3C az adott mélység Bayer-formátumát veszi át, így egy olyan tömb, amelynek vezérlője egy M3M, már nem nyitja meg újra M3C kameráit Mono. Lásd [Mono (M3M) egysávos kamerák](#mono-m3m-single-band-cameras).
> - **Az `lattice array-capture` exportfájlok alapértelmezés szerint igazodnak, akárcsak az alkalmazáséi.** Minden tag `debayered`, `display`, `radiance`, `reflectance` és indexfájljait a fő kamera képkockájához igazítják, és a tagok átfedési területére vágják (amely a kamerák eltolásai miatt kisebb, mint az érzékelő, `XMP-mapir:AlignmentJSON` `"applied": true`), így egy CLI-felvétel ugyanazokat a fájlokat tartalmazza, mint a Chloros-alkalmazás ugyanazon jelenetről készített felvétele; az `raw` natív formátumban marad. Korábban egy `--aligned` nélküli rögzítés minden fájlt natív méretben, torzítás nélkül mentett. Az `--no-aligned` ezt megtartja. Ugyanez az alapértelmezés vonatkozik az `--smart`-re, az `--continuous`-re és az `--interval`-re, valamint az SDK-re és az `chloros-mcp`-re is. Lásd [`array-capture` módok](#array-capture-modes).
> - **Az `lattice array-capture --compression` alapértelmezés szerint `none`.** A fájlok tömörítetlenül kerülnek írásra (teljes felbontású képkockánként körülbelül 6,3 MB) a leggyorsabb folyamatos sebesség elérése érdekében; az 1.2.0 verzió `deflate` formátumban írt (körülbelül 4,1 MB). A kisebb fájlokhoz adja meg az `--compression deflate` értéket. Mindkettő veszteségmentes.
> - **A PNG és a JPEG formátumok csak az `display` / `preview` és indexelt termékeket fogadják el.** Egy olyan `--format png` vagy `jpg` rögzítést, amelynek szintje meghaladja egy 8 bites fájl kapacitását, a rendszer elutasítja még a rögzítés megkezdése előtt (HTTP 400, kilépés `1`), ahogyan az alapértelmezett `--processing all` is, amely `--format png`/`jpg`-et tartalmaz. Lásd [Rögzítési exportszintek](#capture-export-levels-the-all-default).
> - **Az `lattice dls`-hez szükség van az `--mac`-re.**Nincs többé alapértelmezett MAC-cím (ez korábban egy egység címe volt); az `daq pool-discover --only ble` egy érzékelő címét találja meg.**`lattice dls --camera SERIAL`** új.
> - **További hibák esetén az `1` kilép.** Az `process` akkor lép működésbe, ha egy futtatás exportált adatai nem ellenőrizhetők, vagy elvesztette a képet, bár a többi termék leírásra került; egy rögzítési parancs bármely olyan fájllal, amelyet nem tudott írni (a többi fájl továbbra is mentésre kerül); `lattice continuous` és `multi-stream`, ha a meghajtó megtelt; `array-burst` / `array-record`, ha a felvevő hibát jelez; `array-build-video`, ha egy kért terméket kihagytak, vagy nem a kérésnek megfelelően készítettek el; `project burst` / `stream`, ha a rögzítés félúton meghiúsul; `project sensor log`, ha a felvétel nem tartalmaz adatot; `lattice exposure --set`, `gain --set` és `configure --set`, ha a kamera elutasítja az értéket. `lattice network-analysis --json` az összefoglalóval együtt lép ki (`1` az `needs_force_slip` esetében). Fordított esetben: egy olyan érzékelő `daq pool-disconnect` kódja, amely már nem szerepel a poolban, leállítja az `0`-et. Lásd [Kilépési kódok](#exit-codes).
> - **Egy lassú `lattice array-connect` soha nem bukik meg, ha a tömb csatlakoztatva van.** Legfeljebb 600 másodpercig vár (1.2.0-ban 180 másodperc; `project connect` 90 másodperc), közben a háttérrendszer lépéseit kinyomtatva, és az addig be nem fejezett csatlakozást a háttérrendszer visszavonja — minden kamerát visszaállít a rendszer az eredeti állapotába — így egy `Error:` (kilépés: `1`) azt jelenti, hogy nincs csatlakoztatva kameracsoport. Az 1.2.0-s verzióban a kamerák bekapcsolása után azonnal megkezdett csatlakozás túlléphette a várakozási időt, és a parancs sikertelensége után is csatlakoztathatta a sorozatot. A Ctrl+C billentyűkombináció most már megszakítja a csatlakozást. Lásd [Tartós kapcsolatok](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **Az `lattice array-connect` megőrzi a teljes érzékelőt.** `--width` / `--height` / `--binning` hiányában minden kamera a teljes érzékelőre van beállítva, és a képkockasebesség a számítógép és a hálózat által biztosított értékre csökken; a csatlakozási kimenet gyorsabb lehetőségeket sorol fel (binning vagy kisebb ROI) a várható sebességekkel együtt. Az 1.2.0 verzió kicsinyítette vagy binningelte a képkockákat a sebesség fenntartása érdekében. Lásd [`array-connect` beállítások](#array-connect-options).
> - **A lassú hálózat már nem utasítja el a kamerasorozatot.** Minden kamera a hálózati sávszélességének megfelelően van szabályozva, és a képkockasebesség ezzel arányosan csökken. Ha a kamerákat nem lehet ilyen lassan szabályozni, a kamerasorozat soros kibocsátási módban működik (`slip-emit-and-capture`) módban működik: minden képkocka megérkezik, de a kamerák nem egyszerre exponálnak. Az `--force-tier sim-capture-sim-emit` ragaszkodik az egyidejű exponáláshoz. Lásd: [Tömb fps és sorozatfelvételi modell](#array-fps--burst-model).
> - **`lattice network-analysis` / `analyze-array` meghatározza, hogy a connect milyen beállításokat alkalmaz**: `--pixel-format` hiányában a connect által beállított formátumok (Mono12 / BayerRG12, korábban BayerRG8) és a [tömörített 12 bites](#packed-12-bit-transport---packed) döntés. Ugyanez a parancs ezért eltérő képkockasebességet jelezhet, mint az 1.2.0-s verzióban.
> - **A relatív útvonalak a parancs futtatásának helyéhez viszonyulnak.** Az `-o` és a háttérprogram által megnyitott egyéb útvonalak (`--input`, `--burst-dir`, `--profile`, `--save`, `--mask`), egy receptlépés `output` elnevezése, valamint az `lattice index --live` alapértelmezett `output/` elnevezései az `chloros-cli` futtatásának helye szerinti mappához vannak rendelve. Az 1.2.0-s verzióban a háttérprogram a saját telepítési mappájában oldotta meg őket, ami egy telepített verziónál „Hozzáférés megtagadva” hibával járt. Lásd [Konvenciók](#conventions).
> - **Az `lattice index` minden formában a háttérprogramban fut**, akárcsak a többi `lattice` parancs, ezért először indítsa el a háttérprogramot. Az offline indexet (`--input`) kiszámítását a háttérprogram végzi, amely beolvassa a TIFF-fájlt, és az eredményeket annak mellé vagy az `-o` helyre írja, az `--list-presets` és az `--list-gradients` pedig a háttérprogram tábláit nyomtatja ki. Windows-en ezek most már működnek: korábban a Windows CLI minden `lattice index` formátumra – beleértve az `--live`-et is – az `Error: lattice_sdk not available` válasszal reagált. Linux rendszeren az offline index korábban a CLI-en belül futott háttérprogram és bejelentkezés nélkül; most már mindkettőre szüksége van, mint minden más CLI-parancsnak. Lásd [Index / Vegetációs matematika](#index--vegetation-maths).
> - **Új fájlok a termékek mellett.** Minden olyan sugárzási, visszaverődési és indextermék, amelyben a sávszétválasztás egy pixelt korlátozott, kap egy maszkot a termék mappája melletti `clamp_masks/` mappában, és a kinyomtatott eredményben megjelenik egy `clamp mask:` sor. Lásd: [Sávszétválasztási korlátozási maszk](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on).
> - **A kamera tájolása.** Egykamerás felvétel esetén az `raw` és az `debayered` fájlok az érzékelő által leolvasott formában kerülnek mentésre, és a kamera tükröződését/forgatását XMP-ben rögzítik; az `process` fájl ezt alkalmazza az általa létrehozott termékekre. A tömb tagfájljai natív formátumban maradnak, hacsak nem adja át az új `--orient` (`array-capture`, `array-burst`, `project capture` / `burst` / `stream`) paramétert. Lásd: [A kamera tájolása](#what-a-captures-folder-looks-like).
> - **Az 1-től eltérő exportszint-nevet a rendszer elutasítja.** Az `lattice capture --processing` és minden `--levels` lista elfogadja az `raw`, `debayered`, `radiance`, `reflectance` vagy `display` névvel (vagy egy olyan álnévvel, mint például `preview`; `--processing` és `all`). Bármely más név elutasításra kerül, mielőtt a kamerát megérintenék (HTTP 400 `UNKNOWN_LEVEL`, kilépés `1`); 1.2.0 elmentette az `raw` nevet, majd kilépett (`0`).
> - **Az `daq pool-set-cap`-hez szükség van az `--cap-id`-re.** Enélkül a parancs kilép `2`; az 1.2.0 eltávolította a fedélkorrekciót, így a szenzor fedél nélkül maradt. Az `--cap-id none` a csupasz érzékelőt kéri.
> - **Az `lattice resolution --binning-mode` eltűnt.** A binning mindig Átlag; a jelzőt továbbra is elfogadja, figyelmeztetést jelenít meg, de nem hajt végre semmilyen műveletet. Lásd [Érzékelő beállítása](#sensor-tuning).
> - **Az `process --debayer texture-aware` parancsot a LATTICE-felvételek esetén elutasítja a rendszer.** Egy LATTICE-felvételeket tartalmazó mappa esetén az `process` az importálás után kilép az `1`-ből, megnevezve, hogy hány felvétel van, mielőtt bármit is feldolgozna. A LATTICE-felvételek mindig a szabványos demosaic-kezelést kapják, ezért az 1.2.0-s verzió elfogadta az opciót, de figyelmen kívül hagyta; a Chloros-alkalmazás sem kínálja fel ezt számukra. Lásd [`chloros-cli process`](#chloros-cli-process).
> - **A kamerák visszatérnek az eredeti állapotukba — kivéve, ha az `--keep-settings`.** `lattice array-disconnect`, minden átmeneti parancs (`multi-capture` tömb nélkül, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`) és az `lattice continuous` parancsok minden kamerát visszaállítanak az eredeti állapotába: a kioldó- és szinkronvezeték-kábelezését, a PTP-jét, valamint minden olyan kamerabeállítást, amelyet a munkamenet során megváltoztattak (expozíció és erősítés, az AE-célérték, határértékek és csillapítás, fehéregyensúly, feketeszint, képpontformátum, binning és ROI, tükrözés, képkockasebesség). Az `--keep-settings` viszont az egyes kamerákbeállításait a kamera saját memóriájába menti, így azokkal indul el; az `--clear-settings` pedig visszaállítja azt a gyári beállításokra. Mindkét esetben a kamerák szabadon futnak tovább. A projekt a saját mentett beállításai alapján állítja be a kameráit, amikor csatlakoztatja őket. Lásd [Kamera memória](#camera-memory----keep-settings----clear-settings).
> - **Minden üzemmód ugyanúgy menti el a projektet.** A projekthez csatlakoztatott kamerán, mátrixon vagy fényérzékelőn végzett bármilyen módosítás – akár az alkalmazásból, az CLI-ből, az SDK-ből vagy az MCP-kiszolgálóról – elmentésre kerül a projekt `cameras.json` / `sensors.json` mappába, és a projekt bármelyik módban történő megnyitása beállítja az abban szereplő eszközöket. Korábban csak az alkalmazás mentette a projektet, és a következő mentés felülírta az összes többit. Lásd: [Egy projektrekord, minden mód](#one-project-record-every-mode).
> - **A több kamerából készült fájlok mindegyikét megnevezik.** Egy többsávos halmaz (`align-apply`, `align-stream`, `index --live --save-multiband`), több kamera sávjaiból készült élő index, valamint egy tömbfelvétel `composite/` képfájlja listaként hordozza az egyes forráskamerák sorozatszámát, modelljét és kalibrációs csomagját (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), az EXIF `SerialNumber` pedig a sorozatszámokat vesszővel elválasztva sorolja fel. Korábban egyáltalán nem tartalmaztak kamerát, vagy csak az első kamera sorozatszámát. Lásd [Több kamerából készült fájlok](#files-made-from-several-cameras).
> - **Csak egy `raw` fájl tartalmazza a kamera kalibrációs címkéit.** A MicaSense-kompatibilis készlet (`BlackLevel`, `RadiometricCalibration`, a csatornánkénti `Band{R,G,B}RadiometricCalibration`, az `MicaSenseVignetting*` készletek) és az `CalibrationJSON` hasznos adat a szenzor nyers értékeit írja le, ezért kizárólag az `raw` rögzítési fájlban találhatók meg. Minden egyéb fájl — egy rögzítés `debayered`, `display`, `radiance`, `reflectance` és indexfájlok, az `lattice continuous` és `multi-stream` képkockák, valamint minden `process` termék – hordozza a kamera azonosítóját (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...), valamint felvételkor a képkocka saját adatait (`XMP-mapir:FrameJSON`: expozíció, erősítés, hőmérséklet, képpontformátum, felvétel ideje). Az 1.2.0-s verzióban a nyers adatsort is tartalmazták, amelyet egy (Pix4D, Metashape) a nem nyers értékű pixelekre is alkalmazta. Az `process` a sorozatszám és az `CalBundleSha256` alapján határozza meg az ilyen fájl kalibrációját.
>
> **Egyéb változások:**
>
> - **A tömb automatikus képkockasebessége a tömb által szolgáltatott értékhez igazodik.** Az a sebesség, amely a triggerjeinek több mint 1%-át elveszíti, lecsökken – ezt nem egy, hanem több hosszú megfigyelés alapján ítélik meg –, és az a sebesség, amelyről lecsökkent, időnként újra megpróbálásra kerül. A felhasználó által beállított sebességet a rendszer nem módosítja.
> - **Az automatikus expozíció stabilabb.** Állandó fényviszonyok mellett változatlan marad, változás esetén másodpercek alatt stabilizálódik, figyelmen kívül hagyja a kis fényes területeket, és újra működésbe hozza az erős napfényben nem reagáló kamerát. Lásd [Hogyan viselkedik az automatikus expozíció](#how-auto-exposure-behaves).
> - **Az `process` csak a képeidet és felvételeidet számolja.** Ha újra futtatja egy olyan mappán, amely tartalmazza a projekt `camera_cal/` fájljait, a program már nem számolja azokat a kalibrációs fájlokat („96 képet talált” egy 90 képből álló készlet esetén), és egy olyan projekt, amely már felvette őket, eldobja azokat.
> - **`lattice multi-test` a kamerák bekapcsolási állapotából indul ki.** A futtatáshoz bekapcsolja a kamerák PTP-jét, és az első indítás előtt megvárja, amíg az óráik szinkronizálódnak és stabilizálódnak — minden kamera 120 másodpercig, a bekapcsolástól számított körülbelül 2 percig szinkronizálódik (az `--ptp-settle SECONDS` állítja be), mivel egy éppen szinkronizált óra körülbelül 2 percig akár ~200 us-os eltérést is mutathat — majd minden kamera PTP-jét visszaállítja az eredeti állapotába. Egy korábbi tesztből még mindig szinkronizált kamerára nem vár újra. Korábban egykapcsolós hálózaton, ahol egyetlen DAQ-E sem válaszolt, kikapcsolta a PTP-t, és minden kamera szabadon futó óráját olyan PTP-óraként értelmezte, amely nem szinkronizálódik (`PTP is not locked`, minden futtatásnál). Lásd [Többkamerás (átmeneti munkamenetek)](#multi-camera-transient-sessions).
> - **Egy `.daq` felvétel jelzi, hogy a kupakját választották-e ki, vagy feltételezték-e.** A DAQ-U, DAQ-M és DAQ-E készülékek fedél nélkül kerülnek forgalomba, és a Chloros nem tudja megállapítani, hogy van-e rajtuk fedél, ezért egy név nélküli fedél nélkül csatlakoztatott érzékelő alapértelmezés szerint az `sunshine_cosine` értéket kapja, ami egy *feltételezés*. Minden Chloros-felvétel (1.30-as sémakészlet) mostantól az `cap_id` mellett az `cap_id_source`-et is tárolja: `operator`, ha nevet adott a kupaknak (beleértve az `none`-et is), `auto_default`, ha az volt az alapértelmezett. Ha olyan felvételt dolgozunk fel, amelynél a kupakot feltételezték, és a projekt DAQ-kupakja „Auto” beállításon maradt, a fájl nevét tartalmazó `[daq-dls] WARNING` üzenet jelenik meg: egy napfény-korrekció nélküli DAQ-U körülbelül 30-szor magasabb értéket jelez. **Nevezze el a beállított korlátot** a csatlakozáskor, vagy állítsa be a projekt DAQ-korlátját, és a figyelmeztetés eltűnik.
> - **Nincs több pip konzolablak a Windows rendszeren.** Egy Python-tel felszerelt PC-n a háttérprogram és az egyes képfeldolgozók indításkor egyszer futtatták az `pip` parancsot, és mindegyik megnyitott egy konzolablakot. Ez már nem történik meg.
> - **Az élő `.daq` felvételek túlélik az áramkimaradást.**A felvétel minden commitkor naplóval és teljes lemezszinkronizálással kerül írásra, az érzékelő mögött, így egy lassú lemez a fájlt késlelteti, de soha nem az adatolvasást. Egy commit közben leállított felvétel (áramkimaradás, leállítás) mellett maradhat egy `<name>.daq-journal` fájl:**másolja össze a kettőt.** A Chloros az első fájlbeolvasáskor visszagörgeti a naplófájlt, így a felvétel a leállás előtt elküldött összes adattal nyílik meg (kb. az utolsó 3 másodperc veszik el); az 1.2.0-s verzióban egy rossz pillanatban bekövetkező áramkimaradás az egész fájlt olvashatatlanná tehette („az adatbázis lemezképe hibás”).
> - **A forgalmas lemezen történő felvétel már nem blokkolja az érzékelőt.** Az `daq pool-record` sorba állítja az olvasási műveleteket, és azokat egy saját szálon írja ki; az írásra nem kerülő olvasási műveletet (az `daq pool-list` az utolsó hibával az `N write error(s)`-et jelzi), ahelyett, hogy leállítaná az érzékelőt (egy írási művelet 8,4 másodpercet vett igénybe négy, TIFF-fájlokat mentő kamera mellett).
> - **A tömb nélküli `lattice multi-capture` már nem veszíti el a mentett képkockákat, ha a trigger időtúllépést okoz.** Egy kamera időtúllépésbe kerülő triggerje (`SC_ERR_TIMEOUT -1011`, négy kamerával egy 1 GbE kapcsolón megfigyelve) az adott képkockacsoportnál befejezi a futást: a már mentett csoportokat az `Saved:` jelzi, majd a hiba, exit `1`. Az 1.2.0-s verzióban ez egy HTTP-500 hiba volt lista nélkül, bár a fájlok a lemezen voltak.
> - **A szoftveresen vezérelt kamerák már nem hagyják ki minden második triggert.** A szoftveres trigger módban lévő kamera most sorba állítja az olvasás során beérkező triggert (vagy az Chloros megvárja, amíg készen áll), ahogyan azt a kábeles szinkronizálású kamerák már eddig is tették.
> - **Az `daq pool-discover` megnevezi a DAQ-E-S-t az `DAQ-E-S`** az `MODEL` oszlopában (és az `--json` esetében az `model`-ben). Minden Ethernet-érzékelőnél az `DAQ-E` felirat szerepelt; azok, amelyek körülbelül 2 másodpercen belül nem válaszolnak az azonosítási ellenőrzésre, továbbra is szerepelnek a listán, `DAQ-E` néven.
> - **Továbbfejlesztett LATTICE-indextermékek** (NDVI és egyéb indexek) az `process`-től kezdve, minden feldolgozási módban.
> - **OCN-kamerák: továbbfejlesztett sugárzási, visszaverődési és indextermékek napfényben.** A sávszétválasztás után flottakorrekciót alkalmaznak; az RGN-kamerák változatlanok maradnak, az OCN-értékek pedig eltérnek az 1.2.0-s verzióétól. Ez vonatkozik a korábbi verzióval készített OCN-felvételekre is, amennyiben azokat olyan számítógépen dolgozzák fel, amelyen megtalálható a felvétel készítésekor használt kalibráció (a Chloros megőrzi a kamera kalibrációját, miután a kamerát csatlakoztatták).
> - **Az `lattice continuous` rögzíti a nyers képkockát.** Az `tiff` futtatás demosaiced 3síkos képkockát írt, amelyekről az `process` nem tud elindulni. Képkockái most már az `raw` fájlban találhatók, amelyet egy tömbfelvétel ír: egy mozaiksík az érzékelő saját értékeivel, tömörítetlenül, a kalibrációs csomaggal együtt. Egy olyan szkript, amely három síkot olvas be ezekből a fájlokból, csak egyet olvas be.
>
> - **Minden rögzítés elmenti a felhasznált DAQ-leolvasást, ahogyan a grafikus felület is.** Az `lattice capture`, `lattice continuous` (képkockánként egy `.daq`), az `array-capture` és az `multi-capture` egy csatlakoztatott tömb (és az SDK) használatával írja az `.daq` fájlt és annak `.match.json` fájlját, amikor egy DAQ-ot kötnek vagy csoportosítanak, beleértve a kizárólag nyers adatokat tartalmazó rögzítéseket is; az 1.2-es verzióban0-ban nyers vagy debayered rögzítés esetén csak az `--force-daq` mentett egyet. Az `--no-daq` kikapcsolja ezt, az `--daq-tolerance-ms MS` beállítja az adott rögzítés illesztési tűrését, az új `daq pool-bind` parancs a kamerát vagy a tömböt a DAQ-hoz köti a parancssorból (CLI). A `.daq` beállítás esetén a rögzítés az `process` beállítás szerint csak a megnevezett kamerákhoz illeszkedik. A rögzítés most a DAQ következő leolvasására vár, hogy összeolvaszthassa az expozíció mindkét oldalán lévő leolvasási értékeket. Lásd: [A rögzítés által mentett DAQ-leolvasás](#the-daq-reading-a-capture-saves) és [Az expozíciótól távoli DAQ-leolvasás](#a-daq-reading-far-from-the-exposure).
> - **`process --indices NAME`** hozzáadja az indexet a projekthez az egyes kamerák sávjaihoz, beleértve egy új projektet is. Az 1.2.0-s verzióban az `Unknown index name(s) ignored` parancs figyelmen kívül hagyta azokat a neveket, amelyek még nem szerepeltek a projektben.
> - **`process` élő tömb mellett:**már nem indítja újra a kamerákat vagy DAQ-okat tartalmazó háttérprogramot, és legfeljebb 15 másodpercig vár egy lezáródó futtatásra. A**Ctrl+C** gombnyomással a háttérprogramban is leállítható a futtatás, nem csak a CLI-ben. Lásd [`process` megjegyzések](#notes).
> - **A Windows 260 karakteres elérési útkorlátját meghaladó mélységű projektmappa** esetén a rendszer azonnal elutasítja a műveletet, és a leghosszabb, a korlátba beleférő mappát használja, ahelyett, hogy egy semmit sem író futtatást hajtana végre. A Windows-telepítő mostantól felajánlja a „Windows hosszú elérési útvonalak engedélyezése” opciót és a Microsoft Defender kizárását a háttérprogram számára; mindkettő alapértelmezés szerint be van jelölve. Lásd: [Windows (.exe)](#windows-exe).
> - **`lattice array-capture --continuous`** a háttérprogram saját folyamatos rögzítését futtatja: 2 másodpercenként egy haladási sáv, összefoglaló a fájlonkénti felsorolás helyett, valamint nem nulla kilépési kód bármely le nem írt fájl vagy teli meghajtó esetén. Lásd: [`array-capture` módok](#array-capture-modes).
> - **Az `array-connect` utáni első `array-capture`** legfeljebb 20 másodpercig vár a tömb első képkockacsoportjára, ahelyett, hogy HTTP 503 „Array still warming up” hibával lépne ki.
> - **A tömb expozíciója egy ROI alatt:** egy kis ROI már nem korlátozza a tömb tagjának expozícióját az érzékelő szabadfutási periódusára (4,9 ms 768 × 304 felbontásnál). Lásd: [Expozíció a tömb tagján](#exposure-on-an-array-member).
> - **Az automatikus expozíció már nem csökkenti a sorozat automatikus képkockasebességét.** A kábelen szinkronizált sorozat a hálózat és a kamerák kiolvasási sebességének megfelelően működik; az automatikus expozíció kitölti az adott sebesség által biztosított időtartamot, majd növeli az erősítést. A periódusnál hosszabb kézi expozíció (`lattice exposure --set`, `configure --set`) esetén a rendszer nem utasítja el a kérését, hanem lelassítja a kioldást, hogy az illeszkedjen az expozícióhoz, és a képkockasebesség visszatér a normális szintre, amikor az elem visszatér az automatikus üzemmódba. Lásd [Expozíció egy rendszerelemnél](#exposure-on-an-array-member).
> - **A rendszerek kevesebb hiányos képkockacsoportot eredményeznek**, és kevesebb 503-as hibát („Nincs új, szinkronizált képkockacsoport”): minden szolga-kamera kioldási bemenete most már zavarmentes.
> - **Tömörített 12 bites adatátvitel** (`--packed`, alapértelmezett: `auto`) körülbelül egyharmadával növeli a teljes szenzoros tömb képkockasebességét. Most már az `array-connect --packet-size` van alkalmazva (az 1.2.0-s verzió elfogadta, de figyelmen kívül hagyta). Lásd [Tömörített 12 bites adatátvitel](#packed-12-bit-transport---packed).
> - **Olyan sorozat, amelynek kamerái a kikapcsolás-bekapcsolás után önállóan helyreállnak.** A kapcsoló kikapcsolása és bekapcsolása után az `array-connect` sorozata-újra megnyitja a kamerákat, amikor azok válaszolnak, és a PTP újra bekapcsolásával folytatja az adatfolyamot; az 1.2.0-ban végleg leállt, és új `array-connect`-re volt szükség. Az `array-status` az `ptp_enabled: False`-et (és az `ptp_off:`-et), miközben egy kamera PTP-kapcsolat nélkül olvas, ahol korábban a tömb csatlakozási beállítását jelentette. Lásd [Tartós kapcsolatok](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **Az `lattice array-record` felvételt készít, ha nincs megnyitva a grafikus felület.** Ha a grafikus felület kombinált nézete bezárva van, a háttérrendszer maga rendereli a nézetet: a rendszerNDVI-jét, vagy az új `--expr`-szel átadott indexet. Az a felvétel, amely nem ír keretet, most már hibaüzenetet ad (`no-frames: …`, kilépés `1`). Korábban csak akkor rögzített, amíg a GUI kombinált nézete nyitva volt, és egy olyan felvétel, amely semmit sem írt, akkor is kilépett az `0`-ből. Lásd [`array-record`](#array-record--combined-index-videogif-monitoring-grade).
> - **Egy olyan kamerát, amelyet a Chloros már birtokol, nem nyit meg másodszor.** Egy olyan parancs, amely maga nyitná meg, ehelyett HTTP 409 `CAMERA_HELD` hibakódot ad vissza, megnevezve azt, aki birtokolja, és azt a parancsot, amely felszabadítja. Ez vonatkozik egy olyan kamerára is, amelyet éppen egy másik parancs használ (például egy `multi-test` vagy `multi-capture`, amely megnyitja a kameráit, vagy egy `lattice capture`): a második parancsot elutasítja a rendszer, amíg az első be nem fejeződik. Az újbóli megnyitás leállíthatja a birtokos streamjét, vagy összeomlaszthatja a háttérrendszert. Lásd [Tartós kapcsolatok](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **Az átmeneti többkamerás parancsok minden kamerát kiszolgálnak, vagy meghiúsulnak.** `lattice multi-capture` csatlakoztatott tömb nélkül, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream` és `index --live` kilépnek az `1`-ből, megnevezve azt a kamerát, amelyik nem szállított képkockát. Az adatfolyamok kihagyják a hiányzó kamerát tartalmazó képkockacsoportot, és három egymást követő kihagyás után leállnak; az `multi-test` minden triggerre egy sort nyomtat, az `(no frame)` pedig azt jelzi, hol hagyta ki a kamera; az `align-stream` szintén kilép az `1`-et egy feladati hiba miatt. Ezután minden futtatás minden kamerát visszaállít arra az állapotra, ahogyan találta: a trigger és a szinkronvezeték bekötését, a PTP (`PtpEnable`) és minden olyan kamerabeállítást, amelyet a futtatás megváltoztatott — expozíció és erősítés, AE-cél, határok és csillapítás, fehéregyensúly, feketeszint, pixelformátum, binning és ROI, tükrözés, képkockasebesség — kivéve, ha `--keep-settings` / `--clear-settings` parancsot használja (lásd [Kamera memória](#camera-memory----keep-settings----clear-settings)). Korábban ezek a parancsok megőrizték az érkező képkockákat, majd kiléptek az `0` állapotból (az `align-stream` feladat-hibát jelezett, mint az `Note:`), és minden kamerát az `TriggerMode` `On` állapotban hagytak. Ezenkívül minden kamera expozícióját a képkocka mentése előtti jelenet alapján állították be (legfeljebb néhány másodpercre), a célponton egy csatlakoztatott tömb minden kamerát a következő beállításokkal indít: 40 multispektrális, 60 RGB-re (az `--brightness N` állítja be), és ezt tartja a futás során; ugyanezt teszi egy egyetlen `lattice capture` parancs is egy olyan kameránál, amelyet a Chloros nem tart fenn. Korábban minden kamera első képkockáját a kamera saját expozíciós határértékénél vették fel, körülbelül 100 ms-nál, függetlenül a fényviszonyoktól, és gyakran túlexponáltak voltak. Lásd [Többkamerás (átmeneti munkamenetek)](#multi-camera-transient-sessions).
> - **Az átmeneti többkamerás parancsok úgy osztják meg a hálózatot, ahogyan az `array-connect` teszi, és csak az `--serials`-et nyitják meg.** Minden kamera a gazda-kapcsolatból rá eső részesedéshez igazodik, így négy kamera fut egy 1 Gbps-es kapcsolaton; korábban a négy kamera összesen 125 MB/s-ot igényelt, mind a 1 GbE-t, és minden képkockánál csomagvesztés történt. A hálózat által nem kezelhető tömböt a rendszer elutasítja, mielőtt bármelyik kamera elkezdene streamelni. Az `--serials` most kiválasztja a kamerákat, az első pedig elindítja a triggert; korábban minden parancs megnyitotta a hálózaton lévő összes kamerát, függetlenül attól, hogy az `--serials` mit mondott. A GVSP-csomagvesztésként jelentkező hiba most már ezt jelzi, ahelyett, hogy az M8 szinkronkábelre utalna. Lásd [Többkamerás (átmeneti munkamenetek)](#multi-camera-transient-sessions).
> - **`lattice array-disconnect` a kamerákat szabadon futó állapotban hagyja, ahogyan találta.** `TriggerMode`, `Off`, valamint minden egyéb beállítást úgy hagy, ahogyan a csatlakozás során talált – a kioldó és a szinkronvonal beállításait (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`), a PTP, valamint a munkamenet során módosított kamera-beállítások – így egy egyetlenkamera-parancs a korábbi tagon egyszerre letölti a képkockákat, és a mester már nem marad az `TriggerSource`, `Software` állapotban. Az `--keep-settings` a munkamenet beállításait az egyes kamerák memóriájába menti, az `--clear-settings` pedig visszaállítja azt a memóriát (lásd [Kamera memória](#camera-memory----keep-settings----clear-settings)). Az `lattice continuous` emellett az általa élesítettnek talált kamerát szabad futásra állítja a futás idejére (majd utána visszaállítja), és egy keret nélküli futás esetén kilép az `1`-ből.
> - **`lattice configure --get` / `lattice exposure`** kinyomtat egy `note:`-et a kameravalódi expozíciójával vagy erősítésével. Egy elutasított `lattice exposure --set` / `configure --set` megadja a kamera által elfogadott tartományt, és visszaállítja az automatikus módot, ahelyett, hogy kikapcsolná az automatikus expozíciót. Lásd [Érzékelő-beállítás](#sensor-tuning).
> - **Új kamera-vezérlők:**`lattice highlights` (a fényes fehér felületek lefelé húzása az RGB-képernyőn) és `lattice exposure --brightness-auto`.**`lattice multi-detect`** egy próbakör alatt megtalálja a kábelezett kamerákat, és alapértelmezésként a főkamerát az RGB-kamerára állítja.
> - **Új DAQ-parancsok:** `daq pool-bind`; `pool-enable-ptp`; valamint a DAQ-E saját beállításaihoz: `pool-info`, `pool-set-network`, `pool-set-auto-stream`, `pool-reboot` és `pool-update-firmware`. Lásd a [DAQ-E eszközbeállításokat](#daq-e-device-settings).
> - **A háttérrendszer újra csatlakoztatja az adatátvitelt leállító DAQ-E-t.** Amikor egy poolba tartozó DAQ-E adatfolyama elhallgat (a készüléket tápláló kapcsoló újraindítása, a kamera kapcsolat megszakad), a háttérrendszer a korábbi beállításokkal csatlakoztatja újra, és a hiányzás ideje alatt egyre hosszabb időközönként próbálkozik újra. A rajta folyó felvétel ugyanabban az `.daq` fájlban folytatódik, a szünetet a fájlban jelölve, az `pool-record --stop` fájl pedig felsorolja az egyes szüneteket. Az 1.2.0-s verzióban az érzékelő addig maradt csendben, amíg az `pool-disconnect` + `pool-connect` fájlok meg nem jelentek, amelyek egyben a felvétel végét is jelentették. Lásd [A DAQ-E, amely leáll az adatátvitellel](#a-daq-e-that-stops-delivering).
> - **`time-sync status`** jelzi, hogy a PTP grandmaster csomagjai eljutnak-e a hálózatra (`health`), és ha nem, akkor a stderr-re ír egy `WARNING:` üzenetet. Lásd [`time-sync`](#chloros-cli-time-sync).
> - **Linux .deb:** növeli a kernel socket-fogadó puffereinek méretét, amelyekre a kamerák adatfolyamainak feldolgozásához szükség van (a kernel alapértelmezett beállításainál a tömbök képkockákat és képkockasebességet veszítenek), hozzáad egy háttérszolgáltatást, amely a felhasználó nevében fut, és az `lattice network` ellenőrzi a puffereket. A CLI soha nem állítja le a szolgáltatás által futtatott háttérszolgáltatást, és a frissítés újraindítja az engedélyezett háttérszolgáltatásokat. Lásd: [A háttérszolgáltatás futtatása szolgáltatásként](#running-the-backend-as-a-service-headless) és [Linux-gépek — socket-fogadó pufferek](#linux-hosts--socket-receive-buffers) című szakaszokat.
> - **Az `lattice array-capture` jelzi, ha nem sikerült az igazítás:**minden olyan ok, ami miatt egy rögzítés igazítás nélkül került mentésre (nincs igazítási profil, különböző felbontású elemek, kábellel nem szinkronizált tömb, a profil nem rendelhető hozzá egy kerethez), `Alignment:` sor formájában jelenik meg.**`chloros-cli daq help`** az `daq --help` (az `2`-ből lépett ki), és Linuxon a súgó felsorolja azokat a közvetlen hardveres `daq` parancsokat , amelyeket a build tartalmaz.
> - **Jetson:** a csomag neve `chloros_1.3.0_arm64_jp6.deb`, a JetPack 6-hoz, és a GPU-feldolgozása az Orinon fut a JetPack CUDA PyTorch-jával. Lásd [Linux arm64 — Jetson](#linux-arm64--jetson-jetpack-6).

---

## Konvenciók

- Minden parancs előtagja `chloros-cli`. Windows rendszeren a bináris fájl neve `chloros-cli.exe`; Linux/Jetson rendszeren pedig `chloros-cli`.
- Az opcionális argumentumok `--flag` formában szerepelnek. A kötelező pozíciós argumentumok zárójelek nélkül szerepelnek.
- Ha alapértelmezett érték van megadva, a jelző kihagyása esetén az az érték kerül felhasználásra.
- A CLI egy egyszerű HTTP-kliens a Chloros-háttérrendszeren keresztül (az `127.0.0.1:5000` szerveren, amelyet az asztali alkalmazás is használ). **Mely parancsok indítják el:**
  - Az alapvető parancsok — `process`, `login`, `logout`, `status`, `export-status`, `time-sync` és `selftest` — futó háttérszolgáltatást használnak, vagy elindítanak egyet, ha nincs futó, és leállítják, amikor befejezik a munkát.
  - **Az `lattice`, `project` és `daq` parancsok soha nem indítanak el háttérszolgáltatást.** Ezekhez már futó háttérprogramra van szükség: a Windows-as asztali alkalmazásé, a Linux [háttérszolgáltatásá](#running-the-backend-as-a-service-headless), vagy egy olyanra, amelyet Ön indít el (lásd [A háttérprogram elindítása](#start-the-backend)). Ha nincs ilyen, akkor az `Error: Chloros backend not reachable at http://127.0.0.1:5000` üzenetet jelenítik meg a futtatáshoz szükséges paranccsal együtt, majd kilépnek (`1`).
- Az `CHLOROS_BACKEND_URL=<url>` a **`lattice`**,**`project`**és**`daq pool-*`** parancscsaládokat egy másik háttérrendszerre (az `lattice network-analysis` helyette az `--backend-url`-et veszi igénybe). Az alapvető parancsok szándékosan rögzítik az `http://127.0.0.1:<port>`-et, és figyelmen kívül hagyják (az IPv4-literál elkerüli a Windows `localhost`→`::1` ~2 másodperces kérésenkénti teljesítménycsökkenését). Lásd [Környezeti változók](#environment-variables).
- **A relatív útvonalakat az `chloros-cli` futtatásának helye szerinti mappa alapján értelmezi a rendszer.** A háttérfolyamat az, amely megnyitja a kimeneti és bemeneti mappákat, ezért az útvonal elküldése előtt a CLI abszolút útvonalat alakít ki belőle: `-o` / `--output` (beleértve az alapértelmezett `output`-et is), `--input`, `--burst-dir`, `--profile`, `--save` és `--mask`, valamint a [recept](#recipe-dsl) lépés `output` (alapértelmezett: `output`), valamint az `lattice index --live` alapértelmezett `output` mappája. Az `-o captures/` ezért az `<current folder>/captures/`-be ír, és a kinyomtatott útvonalak abszolútak. Az `~` kibontásra kerül. Távoli háttérrendszer esetén (ahol az `CHLOROS_BACKEND_URL` egy másik gépet jelöl) az elérési útvonalakat a beírt formában küldik el, és azok a célgépen kerülnek feloldásra.
- Minden SDK/CLI híváshoz (gépenként egyszer futtassa az `chloros-cli login` parancsot; a gyorsítótárban az `~/.chloros/` tárolja).
- A példák Linux-útvonalakat használnak; Windows rendszeren cserélje ki az `/home/user/...`-et az `C:/Users/.../...`-re. A DAQ-Usoros portja Windows rendszeren `COM3` formátumú, Linux rendszeren pedig `/dev/ttyUSB0` / `/dev/ttyACM0` formátumú.
- Az összefoglalók a kölcsönösen kizáró jelzőket az `{--a | --b}` formában, az opcionális jelzőket pedig `[...]` formában írja.

---

## Legfelső szintű összefoglaló

```
chloros-cli [global options] COMMAND [command options]
```

### Globális beállítások

| Jelző | Leírás |
| --- | --- |
| `--backend-exe PATH` | Felülírja az automatikusan felismert háttérprogramot. |
| `--port N` | A háttérprogram HTTP-portja (alapértelmezett: `5000`). Csak a magparancsok használják (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`); az `lattice`, az `project` és az `daq` az `CHLOROS_BACKEND_URL`-et használja (alapértelmezett `http://127.0.0.1:5000`). |
| `-v, --verbose` | Részletes kimenet engedélyezése. |
| `--restart` | A háttérfolyamat kényszerű újraindítása (minden futó `backend_server.py`-et leállít). A systemd szolgáltatás által futtatott háttérfolyamat (Linux .deb) által futtatott háttérprogramot soha nem állítja le: a parancs ehelyett az `sudo systemctl restart <unit>` üzenetet írja ki, majd kilép az `1` parancsból. |
| `--version` | Kijelzi a verziót (`Chloros CLI 1.3.0`). |
| `--help` | Megjeleníti a legfelső szintű súgót. |

### Parancsok indexe

| Parancs | Cél |
| --- | --- |
| [`process`](#chloros-cli-process) | A Survey3- vagy LATTICE-felvételeket tartalmazó mappa teljes feldolgozása. |
| [`login`](#chloros-cli-login) | Hitelesíti ezt a gépet egy Chloros+ fiókkal. |
| [`logout`](#chloros-cli-logout) | Törli a gyorsítótárban tárolt hitelesítő adatokat. |
| [`status`](#chloros-cli-status) | Az aktuális licenc- és hitelesítési állapot megjelenítése. |
| [`export-status`](#chloros-cli-export-status) | A Thread-4 exportálásának élő állapotát jeleníti meg egy `process` futás során. |
| [`language`](#chloros-cli-language) | A CLI-kijelző nyelvének beállítása vagy felsorolása (38 támogatott nyelv). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Alapértelmezett projektmappa (megosztva a GUI-val). |
| [`update`](#chloros-cli-update) | Chloros-frissítések keresése és telepítése (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Rendszerdiagnosztika + gyors tesztek. |
| [`time-sync`](#chloros-cli-time-sync) | PTP grandmaster állapot / vezérlés. |
| [`lattice`](#chloros-cli-lattice) | LATTICE kamera vezérlés és felvétel (több mint 45 alparancs). |
| [`daq`](#chloros-cli-daq) | DAQ spektrális érzékelő vezérlése (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Mentett Chloros-projekt megnyitása és futtatása (kamerák + DAQ-ok). |

---

## Telepítés

Az `chloros-cli` minden támogatott platformon a Chloros asztali telepítőjében található — nincs külön CLI-letöltés. A platformcsomag telepítésével az `chloros-cli` bekerül az `PATH`-be az asztali alkalmazás és az általa vezérelt háttérprogram mellett.

Legfrissebb letöltések: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> A telepítő kényelmi indító szkripteket is tartalmaz (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`), amelyek egy azonnal használható CLI-parancssort nyitnak meg.

### Windows (.exe)

1. Töltse le a Windows-telepítőt a letöltési oldalról.
2. Futtassa az `Chloros Installer 1.3.0.exe` fájlt, és kövesse a varázsló utasításait. Az alapértelmezett telepítési útvonal az `C:\Program Files\Chloros\` (a CLI-parancssor az `C:\Program Files\Chloros\cli\` mappába kerül, amelyet a telepítő hozzáad a PATH változóhoz).
3. Nyisson meg egy új terminált (`cmd.exe`, PowerShell vagy Windows Terminal), hogy a rendszer felismerje a frissített `PATH` fájlt.

```powershell
chloros-cli --version
```

A telepítő automatikusan hozzáadja az `chloros-cli.exe` mappát a rendszer `PATH` mappájához, és csomagolja a LATTICE kamera illesztőprogramjait. A háttérprogram az `C:\Program Files\Chloros\backend\chloros-backend.exe` (a telepítési mappa `backend` mappája), amely nem található az `PATH`-en.

**Rendszerbeállítások.** A telepítő *További feladatok kiválasztása* oldalán a **Rendszerbeállítások**alatt két lehetőség található, amelyek mindkettő**alapértelmezés szerint be van jelölve**:

| Lehetőség (a telepítőben szereplő megfogalmazás szerint) | Mit csinál | Csendes telepítés kikapcsolása |
| --- | --- | --- |
| **Windows hosszú útvonalak engedélyezése** (lehetővé teszi, hogy a projektmappák 260 karakternél mélyebben helyezkedjenek el; ajánlott) | A Windows rendszer egészére vonatkozó `LongPathsEnabled` értéket 1-re állítja. A Windows alapértelmezés szerint kikapcsolt állapotban szállítja, és ilyenkor egyetlen elérési út sem haladhatja meg a 260 karaktert; a Chloros-termékek körülbelül 130 karakterrel a projektmappa alatt helyezkednek el, ezért a mélyebben elhelyezett projektmappákat a rendszer elutasítja (lásd [Hibaelhárítás](#hibakeresési-tippek)). A telepítés után elindított programokra vonatkozik. Az eltávolításkor is bekapcsolva marad, mert más programok is támaszkodhatnak rá. | `/MERGETASKS="!longpaths"` |
| **A Chloros-háttérprogram kizárása a Microsoft Defender vizsgálatából**(sokkal gyorsabb kamerás rögzítés; ajánlott) | Hozzáad egy Defender**folyamat**-kizárást az `chloros-backend.exe` számára a telepítési mappában, így a valós idejű védelem nem vizsgálja át a háttérprogram által írt minden fájlt, függetlenül attól, hová írja azokat. Egyéb programok nem kerülnek kizárásra; a kizárás az eltávolításkor eltávolításra kerül. Ha a Defender nincs telepítve, vagy a házirend által le van zárva, a lépés észrevétlenül meghiúsul, és a telepítés folytatódik. | `/MERGETASKS="!defenderexclusion"` |

**Miért fontos a Defender opció:** magas rögzítési sebesség mellett a rögzített fájlok valós idejű ellenőrzése a rögzítési sebesség körülbelül 30 %-át veszi igénybe (9 kamerás folyamatos rögzítés esetén mért érték: 77,7 rögzítés/másodperc kizárva, 52,7 átvizsgálva). Az `lattice array-capture --continuous` figyelmeztetést jelenít meg, ha azt észleli, hogy a Defender a rögzítés során egy vagy több CPU-magot használ; futtassa újra a telepítőt a bejelölt opcióval.

A háttérben futó telepítés (`/SILENT` vagy `/VERYSILENT`) mindkettőt alkalmazza, hacsak nem választja ki, pl. `"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`.

**A Windows-tűzfal.** A telepítő minden hálózati profilhoz mindig két bejövő szabályt ad hozzá az `chloros-backend.exe`-hez, az eltávolító program pedig eltávolítja azokat: az `Chloros Backend (UDP-In)`-et a kameraképek, a PTP, mDNS és a DAQ-E adatfolyam számára, valamint az `Chloros Backend (TCP-In)`-et, amely a helyi alhálózatra korlátozódik, a DAQ-E firmware-frissítéshez, amely során az érzékelő visszacsatlakozik erre a számítógépre a kép begyűjtése érdekében (lásd [DAQ-E eszközbeállítások](#daq-e-device-settings)). A háttérrendszer HTTP-API-ja továbbra is az `127.0.0.1`-hez van kötve.

### Linux amd64 (.deb)

Ubuntu 22.04 LTS vagy újabb verziókhoz, illetve Debian-alapú x86_64 munkaállomásokhoz.

> **Az Ubuntu 20.04 nem támogatott.** A csomag függőségi listája abból származik,
> amire a háttérprogram ténylegesen hivatkozik, és ez magában foglalja az `libc6 (>= 2.35)`-et is;
> a focal a glibc 2.31-et szállítja. Az `apt` megakadályozza a telepítést, ahelyett, hogy
> futásidőben hibaüzenetet adna. A lista említi az `libgl1`-et is, amelyet a csomagban lévő OpenCV betölt, és amely egy minimális
> szerverképből hiányozhat; az `apt install` tartalmazza.

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

A .deb a következőket telepíti:
- `chloros-cli`-tól `/usr/bin/chloros-cli`-ig, valamint `chloros-backend`-et (elindítja a háttérprogramot) az `/usr/bin/chloros-backend`-ig
- A lefordított háttérprogramot és a CLI-t, kicsomagolva, az `/usr/lib/chloros/backend/` és az `/usr/lib/chloros/cli/`-be
- A LATTICE kamerameghajtók
- Zajszűrő modellek, kalibrációs csomagok és a frissítési csatorna konfigurációja
- A rendszerhez tartozó Python SDK: `python3` (`import chloros_sdk`). Az `apt install` a saját követelményeit is magával hozza: az `python3-numpy` és az `python3-requests` fájlokat (ajánlott, de nem kötelező: a fenti programok önállóak; az `dpkg -i` kihagyja őket, a telepítési kimenet pedig felsorolja a hiányzó elemeket). Virtuális környezetben: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
- `/etc/sysctl.d/60-chloros-gige.conf`, amely a kamera adatfolyamaihoz a rendszermag socket-fogadó puffereinek méretét adatfolyamokhoz 32 MB-ra növeli a kernel socket-fogadó puffereit, és azokat azonnal alkalmazza (a már magasabbra beállított értéket nem módosítja). A csomag eltávolításával a fájl is eltávolításra kerül. Lásd [Linux-gépek — socket-fogadó pufferek](#linux-hosts--socket-receive-buffers).
- Két opcionális systemd-szolgáltatás a háttérprogramhoz, egyik sem engedélyezve (következő szakasz)

#### A háttérprogram futtatása szolgáltatásként (headless)

Szolgáltatás nélkül a háttérprogram (és vele együtt a PTP grandmaster) csak addig fut, amíg valami elindítja: `chloros-backend` egy terminálban, egy SDK-parancsfájlban, vagy egy core `chloros-cli` parancs futási ideje alatt. Ha egy headless gépen állandóan aktív háttérprogramot szeretne, engedélyezze a szolgáltatást **a saját felhasználói fiókjával**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

Az `chloros-backend@<user>.service` parancs a háttérprogramot az adott felhasználóként futtatja. Az általa írt rögzítések az adott felhasználóéi, a részletes naplója az ő `~/.cache/chloros/logs` fájljában található, és az ő `chloros-cli login` (`~/.chloros`) fiókját használja, ezért először jelentkezzen be egyszer azzal a felhasználóval. A PTP-nek nincs szüksége root jogosultságra. Mint minden telepített háttérprogram, ez is kizárólag az `127.0.0.1:5000` porton hallgatózik.

Az `chloros-backend.service` **root** jogosultsággal futtatja a háttérprogramot. Még mindig telepítve van, így az a gép, amelyen engedélyezték, továbbra is fut, de minden, amit egy root háttérprogram ír, a root tulajdonát képezi, így egy felhasználó nem tudja törölni a saját rögzítéseit. Ahhoz, hogy átváltson a saját felhasználójára:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

A root szolgáltatás részletes naplója az `/var/log/chloros` fájlban található, amelyet az `adm` csoport tagjai olvashatnak (a szolgáltatás beállítja az `CHLOROS_LOG_DIR` értéket; lásd [Környezeti változók](#environment-variables)). A csomag telepítésekor a rendszer kinyomtatja az `enable` parancsot a felhasználóneveddel kitöltve, valamint a váltást is, ha a root szolgáltatás engedélyezve van.

**Engedélyezz egy háttérmodult.** Egyszerre csak egy háttérmodul futhat. Egy második engedélyezett egység (az `chloros-backend@<user>.service` mellett az `chloros-backend.service`, vagy két felhasználó `@` egységei) nem indul el, és az `failed` kilépési állapot 98-cal marad; a systemd nem indítja újra. Az `systemctl status` egység megmagyarázza, miért: `Another instance is already running. Exiting.`, majd azt, hogy mi tartja fenn a háttérmodult (az 5000-es port, vagy a zárolási fájl és a hozzá tartozó felhasználó), majd az `systemd will not restart <unit> (exit status 98)`. Újraindítás után bármelyik egység futhat. A váltáshoz tiltsa le a nem használtat, a fentiak szerint.**A CLI** soha nem állítja le a szolgáltatás háttérfolyamatát. Az `chloros-cli` úgy használja, ahogy van, ahogyan a GUI és az SDK is: az `process` ezen fut, és a legutóbbi helyett a saját projektjét tölti be. Ahol az CLI-nek egyébként újra kellene indítania a háttérprogramot (`--restart`, vagy egy `process`, amelynek importja nem talált semmit), megnevezi az egységet, kinyomtatja az `sudo systemctl restart <unit>`-et, majd kilép az `1`-ből, mert egy ilyen módon leállított háttérszolgáltatás leállt állapotban maradna. Az `GET /api/status` az egységet `service` néven nevezi meg (`null` esetén, ha a háttérszolgáltatáson nem fut szolgáltatás).**Frissítés és eltávolítás.** A csomag frissítése leállítja az összes háttérszolgáltatást, majd újraindítja az összes engedélyezett szolgáltatást, először az `@<user>` egységeket (azt, amelyet kézzel indítottak el, de soha nem engedélyeztek, leállt állapotban marad, akárcsak újraindítás után). Ha egynél több van engedélyezve, csak egy fut, a többi pedig a fentiek szerint hibás állapotban marad; a frissítés során az `Chloros: N backend units are enabled: …` és az `Keep one enabled: sudo systemctl disable --now <unit>` kódok jelennek meg. Az eltávolítás leállítja és letiltja őket. A törléssel az `/var/log/chloros` is eltávolításra kerül.

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Ugyanaz az elrendezés, mint az amd64 .deb fájlnál, tartalmazza a fogadó puffereket és a [háttérszolgáltatásokat](#running-the-backend-as-a-service-headless), a **JetPack 6** (L4T R36) rendszeren futó Jetson Orin / Orin NX / Orin Nano készülékekhez. A PyTorch a JetPack 6 Orinra készült CUDA-verziója, így a GPU-feldolgozás – beleértve a textúra-érzékeny debayert is – a Jetson GPU-ján fut. A csomag maga nem tartalmazza a CUDA-t: a JetPack 6 által telepített CUDA- és cuDNN-könyvtárakat használja (`sudo apt install nvidia-jetpack` olyan képen, amelyeken ezek hiányoznak). Ha nincs használható CUDA-GPU, a feldolgozás a CPU-n fut, és a textúra-érzékeny feldolgozás a standard debayert használja (lásd [`process`](#chloros-cli-process)).

### Egyszeri hitelesítés gépenként

Minden platformon egyszeri Chloros+ bejelentkezés szükséges ahhoz, hogy az SDK/CLI-hívások működjenek:

```bash
chloros-cli login user@example.com 'YourPassword'
```

A hitelesítő adatok az `~/.chloros/user_session.json` fájlban kerülnek tárolásra.

### A háttérprogram elindítása

Az `lattice`, `project` és `daq` parancsok egy már futó háttérrel kommunikálnak; soha nem indítanak el egyet sem (lásd [Konvenciók](#conventions)). Használatuk közben tartson egyet futásban:

| Platform | Hogyan |
| --- | --- |
| Windows | Nyissa meg a Chloros asztali alkalmazást (a háttérprogramja a CLI-t is kiszolgálja), vagy futtassa az `"C:\Program Files\Chloros\backend\chloros-backend.exe"` parancsot egy külön terminálban (a telepítési mappa `backend` mappájában), és hagyja nyitva. |
| Linux / Jetson | Futtasd az `chloros-backend` parancsot egy külön terminálban, vagy tarts egyet folyamatosan futtatva a [háttérszolgáltatással](#running-the-backend-as-a-service-headless). |

Egyszerre csak egy háttérszolgáltatás fut; minden kliens (a grafikus felület, a CLI, az SDK-parancsfájlok) ezt osztja meg. Az alapvető parancsok (`process`, `login`, `status`, …) szintén ezt használják, és elindítanak egyet, ha éppen nincs futó.

**A háttérprogram a parancsok között is tovább fut**, így a következő parancs nem indítja el újra. A háttérfolyamat leáll, ha kijelentkezik (`chloros-cli logout`, kivéve, ha a Chloros-alkalmazás nyitva van, vagy futás vagy offline összeállítás van folyamatban), ha futtatja az `chloros-cli backend stop` parancsot, vagy önmagától 30 perc elteltével, ha semmi sem használja: nincs parancs, szkript vagy alkalmazás, nincs futtatás vagy offline összeállítás, nincs csatlakoztatott kamera vagy fényérzékelő. Az `backend stop` elutasítja a leállítást, amíg futtatás vagy offline összeállítás folyik, és megnevezi azt; az `--force` azonnal leállítja, és az adott futtatás vagy összeállítás elveszik (a sorozat megmarad, és az `array-build-video` újra összeállítja). Állítsa be az `CHLOROS_BACKEND_IDLE_MIN` értéket, mielőtt a háttérszolgáltatás elindulna, hogy megváltoztassa a 30 perces időtartamot (`0`: soha); a [háttérszolgáltatás](#running-the-backend-as-a-service-headless) soha nem áll le magától. Az `chloros-cli backend status` megmutatja, hogy fut-e valamilyen, mi használja azt, és fut-e bármilyen offline összeállítás. Az `process` megnyitja a projektjét a futó háttérszolgáltatásban, kivéve, ha a Chloros-alkalmazás nyitva van egy olyan projekttel, amelyet megnyitott: ebben az esetben elutasítja a műveletet, és először az alkalmazásból kell feldolgoznia, vagy be kell zárnia azt a projektet. Egy korábbi `process` vagy egy SDK-parancsfájl által nyitva hagyott projektet egyszerűen felülírja.

### A telepítés ellenőrzése

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

`selftest` csak akkor lép ki az `0`-ből, ha mind a hét ellenőrzés sikeres. Az utolsóhoz CUDA GPU (NVIDIA GPU, vagy egy Jetson Orin-é), valamint a zajszűrő modellekre, így egy ilyen nélküli számítógépen az `Failed: CUDA + Denoiser` kódot jeleníti meg, és kilép az `1`-ből; ott a kizárólag GPU-ra épülő, textúraérzékeny debayer kivételével minden más továbbra is működik.

> **Chloros+ előfizetés szükséges.**A CLI-hez aktív Chloros+ csomag szükséges. A**Copper**a Chloros+ belépő szintje — minden fizetős Chloros+ szint CLI/SDK hozzáférést biztosít; csak az ingyenes**Iron** szint nem. (Csomag-azonosító leképezés: `0`=Iron/free, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Frissítés az [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing) oldalon.
>
> Ezt a korlátot nem csupán a CLI, hanem a háttérrendszer is érvényesíti: egy fizetett csomaggal nem rendelkező, SDK-vel vagy CLI-vel jelölt kérés `403 PLAN_UPGRADE_REQUIRED` hibakóddal kerül elutasításra, függetlenül attól, hogy az `chloros-cli`-ből, a Python SDK-ből vagy egy saját fejlesztésű HTTP-kliensből érkezik-e. A kijelentkezett felhasználó helyette az `401 AUTH_REQUIRED` hibakódot kapja. Az elérhetőség a csomag türelmi ideje alatt offline módban is működik (havonta 30 napig, éves csomag esetén a lejáratig) , és a tárgyidőszak lejárta után megszűnik; az `chloros-cli status` továbbra is működik, így az ok látható marad (ez az egyetlen SDK/CLI útvonal, amely mentesül a szintkorlátozás alól — `GET /api/license-status`).

---

## Ötperces gyorsindítás

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

Az `process` (2. lépés) elindít egy háttérfolyamatot a saját futtatásához, ha még nincs futó, és leállítja azt, amikor befejeződik, ezért indítson el egyet a 4. lépéstől kezdve. Lásd: [A háttérfolyamat elindítása](#start-the-backend).

---

## `chloros-cli process`

Feldolgozza a képeket tartalmazó mappát a teljes Chloros-folyamaton keresztül (célfelismerés → kalibrálás → vignettálás → reflexió → index export).

### Áttekintés

```
chloros-cli process INPUT [OPTIONS]
```

### Pozíciós argumentumok

| Argumentum | Leírás |
| --- | --- |
| `INPUT` | Az `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) vagy `.dng` fájlokat tartalmazó bemeneti mappa elérési útja. |

### Általános beállítások

| Beállítás | Alapértelmezett | Leírás |
| --- | --- | --- |
| `-o, --output PATH` | egy új mappa, amelynek neve `-n` az alapértelmezett projektmappában | A projektmappa. A projekt nevét ez alapján kapja, és egy olyan mappa, amelyben már található egy `project.json` nevű fájl, a `_1` / `_2` nevű testvérmappát kap, ahelyett, hogy felülíródna. Az alapértelmezett projektmappa az, amelyet az `set-project-folder` (vagy a grafikus felület) választott, egyébként az `Documents/MAPIR/Chloros Projects`. |
| `-n, --project-name NAME` | automatikus (időbélyeg) | Projektnév, kizárólag **az `-o` nélkül** használható (az `-o` használata esetén a projekt a mappa nevét veszi fel). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` a Chloros+ neurális debayer-t használja a Survey3 felvételekhez; lassabb, de jobb minőségű. Kizárólag a Survey3-hoz használható: A LATTICE-felvételek mindig a standard demosaic-ot használják, így egy LATTICE-felvételeket tartalmazó mappa esetén az `process` az importálás után elutasítja az `texture-aware`-et, egy hibaüzenettel, amely megnevezi, hány ilyen van, majd kilép az `1` fájlt, mielőtt bármit is feldolgozna (a Chloros-alkalmazás ezekhez sem kínál Texture Aware funkciót). Ehhez CUDA GPU szükséges (Windows / Linux x64 rendszeren NVIDIA GPU, Jetsonon az Orin GPU-ja): ennek hiányában a futtatás az `standard`-et használja, és a háttér egy `[DEBAYER] … needs a CUDA GPU …` sort jegyzőkönyvez, amely megmagyarázza az okot (az `--verbose`-szel együtt látható). |
| `--vignette / --no-vignette` | `--vignette` | Vignettakorrekció. |
| `--reflectance / --no-reflectance` | `--reflectance` | Reflektancia-kalibrálás (ha található, panelcélpontot használ, LATTICE esetén NIST sorozatszám-alapú kalibrálást). A LATTICE multispektrális esetében ez egyben a reflektancia **szorzat** kapcsolóként is szolgál — lásd [Termékenkénti exportkapcsolók](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | ki | PPK GNSS-korrekciók alkalmazása a sidecar fájlokból. |
| `--exposure-pin-1 CAMERA_MODEL` | nincs | Az expozíciós 1. csaphoz csatlakoztatott kameramodell, a PPK időszinkronizáláshoz. |
| `--exposure-pin-2 CAMERA_MODEL` | nincs | A 2. expozíciós csatlakozóhoz kapcsolt kameramodell, a PPK időszinkronizáláshoz. |
| `--recal-interval SECONDS` | `0` | Minimális újrakalibrálási intervallum, másodpercben (a projekt *Minimális újrakalibrálási intervalluma*). |
| `--timezone-offset HOURS` | `0` | A fényérzékelő időzóna-eltérése az UTC-től, órában kifejezve (a projekt *Fényérzékelő időzóna-eltérése*). |
| `--format FORMAT` | `TIFF (16-bit)` | Az alábbiak közül az egyik: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)` közül az egyik. Az Chloros soha nem növeli a kép bitmélységét: a teljes egészében 8 bites képek esetében a nagyobb mélységű formátum `TIFF (8-bit)` néven kerül mentésre, és a futtatás során ez egyszer szerepel. Egy olyan projektben, amely 8 bites és 12/16 bites felvételeket kever, minden felvétel a saját mélységében vagy annál alacsonyabb mélységben kerül exportálásra. A Radiance mindig 32 bites lebegőpontos. |
| `--indices NAME [NAME ...]` | nincs | Vegetációs indexek előre beállított név szerint (`NDVI`, `GNDVI`, `OSAVI`, `SAVI`, `EVI`, `GCI`, …), kis- és nagybetűk közötti különbséget nem tesz. Mindegyik a kamera sávjai alapján kerül hozzáadásra a projekthez, egy új projekt is beleértve. Az a név, amely nem előre beállított, vagy amelyet a kamera sávjai nem tudnak létrehozni (az `NDRE`-hez RE és NIR sávokra van szükség, amelyeket egyetlen kamera sem biztosít), figyelmeztetésben kerül megjelölésre, és nem kap indexképet. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | A LATTICE TIFF-ek feldolgozási folyamatának belépési pontjának kényszerítése (a Survey3.raw fájlra ez nincs hatással). Emellett ez a menekülési út, amely lehetővé teszi, hogy egy **nyers adat nélküli** rögzítést egyáltalán feldolgozzanak — lásd [Hogyan néz ki egy rögzítési mappa](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | be | A lineáris debayeringelt termék kimenete (`Debayered_Images`). Lásd [Termékenkénti exportkapcsolók](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | be | A képernyőn megjelenő előnézet kimenete (`Preview_Images`): RGB = fehéregyensúly (DAQ-megvilágító, ha elérhető, egyébként szürke-világ) + gamma; multispec = hamis színű kiterjesztés. |
| `--radiance / --no-radiance` | be | Float32 sugárzási érték (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referencia a LATTICE reflexiós termékhez: `auto` = a minőségbiztosításon átment, a keretben lévő célérték az abszolút referencia (a minőségbiztosítási jelzővel ellátott célérték versenyez a DAQ-val a mért pontszám tekintetében), DAQ-lefelé irányuló (ρ = π·L/E) tartalék; `target` = szigorú (nincs DAQ-helyettesítés); `daq` = DAQ-meghatározó. Lásd [Termékenkénti exportkapcsolók](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | nincs | Az egységenkénti **mért** célreflektancia-letapogatások könyvtára (`<serial>.csv`); hiány esetén a névleges T3/T4P spektrumokra vált vissza. |
| `--array-alignment / --no-array-alignment` | be | LATTICE-tömbök: minden feldolgozott termékre (debayered / preview / radiance / reflectance / index) alkalmazza az egyes felvételek `Chloros:Alignment*` XMP-jében megjelölt modul-modul igazítást. A címkék nélküli képek esetében nincs művelet. |
| `--array-alignment-crop / --no-array-alignment-crop` | kivágás | Az igazított exportokat a rács közös átfedési területére vágja ki, így minden modul ugyanazt a lábnyomot használja; az `--no-…` megőrzi a teljes érzékelő felületét (a forráson kívül fekete kitöltéssel). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Újramintavétel az igazítási torzításhoz. Az `nearest` megőrzi a forrás pontos dinamikatartományát (a radiometrikus értékek pixelek közötti keveredése nélkül). |

### Célpont-felismerési beállítások

| Jelző | Leírás |
| --- | --- |
| `--min-target-size PIXELS` | A detektor minimális kalibrációs célpontmérete pixelben. |
| `--target-clustering 0-100` | A célpontok csoportosításának százalékos aránya (alapértelmezett: `60`). |
| `--target / --targets` | Kalibrációs célpontok keresése kizárólag a bemeneti fájl `target` vagy `targets` almappájában (gyorsabb). |

### Példák

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### Termékenkénti exportkapcsolók (LATTICE multispektrális)

A LATTICE feldolgozás **egy lépésben** **minden alkalmazható termékre**kiterjed. A négy típusonkénti kapcsoló — `--debayered`, `--preview`, `--radiance`, `--reflectance` —**alapértelmezés szerint mind be vannak kapcsolva**; az `--no-<type>` űrlapot használja, ha valamelyiket ki szeretné kapcsolni. Az RGB-mesterkamerák kizárólag debayerezett + előnézeti adatokat adnak ki (nincs sávonkénti sugárzás/visszaverődés), így az `--radiance`/`--reflectance` ezeknél hatástalanok. A kapcsolókat a Survey3-as `.raw` esetében figyelmen kívül hagyják (amely a szabványos visszaverődés/célútvonalat követi). *(A régi `--radiometric-output {reflectance,radiance,sensor-response}` jelzőt **eltávolították**, és ezekkel a kapcsolókkal helyettesítették; már nincs `sensor-response` szint.)*

| Termék | Kimenet | Szükséges-e DAQ lefelé irányuló adatgyűjtés? |
| --- | --- | --- |
| `--debayered` | Lineáris demosaic (`Debayered_Images`): ugyanaz a fájl, amelyet egy rögzítés az `debayered` szintjén ment — a kalibrációs csomag sötételtolása (DSNU) a nyers adatról, majd az OpenCV élekkel számoló demosaic-ja; nincs linearitási LUT, síkkép vagy színkeverés. | Nem. |
| `--preview` | Képernyő-előnézet (`Preview_Images`): RGB = fehér egyensúly + gamma; multispektrális = hamis színű kiterjesztés. | Nem. |
| `--radiance` | float32 W/m²/sr/nm a teljes radiometrikus láncból (`Radiance_Images`). | Szám |
| `--reflectance` | uint16 fényvisszaverődés ρ (`32768` = 1,0), Pix4D-kompatibilis. | **Igen**, kivéve, ha egy minőségbiztosításon átment, a képkockán belüli célpont rögzíti (lásd alább). |

`--reflectance-source` kiválasztja a reflexió-referenciát:**`auto`**(alapértelmezett) egy minőségbiztosításon átesett, a képkockán belüli célt tesz**abszolút referenciává**— a célponttal rögzített empirikus vonalláncokat a kiválasztott paneleken keresztértékelésnek vetik alá, és a mért győztes értéket sávonként alkalmazzák: egy olyan sáv, amelyet a DAQ nem tud kalibrálni, vagy amelynek DAQ-meredekség-illesztése sikertelen, a célpont saját vonalát veszi át (az `ReflectanceElBandMethods` jelzi, melyikről van szó) — egy olyan cél, amely jelzéssel felel meg a minőségbiztosításnak (kihagyott panel, hibás sáv), szintén versenyez a DAQ-osztással az adott mért pontszám alapján, és az a képkocka, amelytől alulmarad, az `ReflectanceChain=daq` jelölést kap — a DAQ lefelé irányuló elválasztási értékére (ρ = π·L/E) tér vissza, ha nincs célpont jelen, vagy a minőségbiztosítás sikertelen;**`target`**szigorú (nincs DAQ-helyettesítés);**`daq`**a DAQ-vezérelt viselkedést választja. A célpont geometriája (ArUco / rögzített ROI / sáv) a projekt célpontkonfigurációjából származik; az**`--target-reflectance-dir DIR`**az egységenkénti**mért** letapogatásokat (`<serial>.csv`) tárolja, amelyeket a célkészülék sorozatszámának/QR-kódjának alapján keres ki, a névleges T3/T4P spektrumok pedig tartalékértékekként szolgálnak.

A DAQ-reflektancia útvonal automatikusan meghatározza az **időbélyeggel egyező lefelé irányuló sugárzást**egy rögzített**`.daq`**(DAQ-U/M/E)**vagy a képek mellett található DAQ-M natív `.csv`**fájlból. Ha egy kamera- vagy DAQ-kalibrációs csomag nincs helyileg tárolva, a feldolgozási folyamat**az első használatkor automatikusan letölti az AWS-ről** (egyszeri internetkapcsolat szükséges; az `~/.chloros/` fájlban tárolódik).**Kinek a mérési értéke ez?**A rögzítés által mentett `.daq` fájl az általa leírt kamera(k) mérési értéke – amelyet a mellette található `<file>.daq.cameras.json` kiegészítő fájl nevez meg (`lattice capture`, `multi-capture`, `array-capture` és a grafikus felület is; egy nyers sorozat a manifesztjében megnevezi az egyes elemek értékeit) — és az `process`**kizárólag az említett kamerák képeihez**. Egy önálló kamera, amelynek nincs saját leolvasási értéke, ezért már nem veszi át egy másik kamera DAQ-ját; rendeljen hozzá egy DAQ-ot, vagy készítsen felvételt. Egy DAQ saját felvétele (`daq pool-record`, vagy a GUI-ban elindított felvétel), illetve bármely `.daq`, amelyben nincs ilyen sidecar (beleértve a régebbi rögzítéseket is), továbbra is minden kamerát kiszolgál.

#### Reflektancia-pixelek kiolvasása (Pix4D / Metashape / saját szkriptek)

A reflektancia egész számú DN-ként van tárolva, és **az a DN-érték, amely ρ = 1,0-t jelent, a forráskamerától függ**:

| Forrás | ρ = 1,0 értéke | Hogyan lehet megállapítani |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (tartalék ρ 2,0-ig) | Az XMP-fájlban szerepel az `Chloros:PixelScale=32768` jelölés. |
| Survey3 | `65535` (1,0-nál levágva) | Nincs `Chloros:*` XMP-címke — ez a hiány *maga* a jel. |

**Olvassa be az `Chloros:PixelScale` értéket, és ossza el vele**, ahelyett, hogy állandót feltételezne. A címke az uint16 tartományban van definiálva, így az átméretező kimeneti formátumokban is `32768` marad — `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)` és `TIFF (32-bit, Percent)` mind önleíróak (először normalizálja vissza a tárolt adattípust uint16-ra: 8-bites értékek esetén ×257-re, lebegőpontos értékek esetén ×65535-re).

> **A régebbi verziókból származó fájlok nem tartalmazhatnak skálát.** Egy 8-bites reflektancia-TIFF-fájlt átméreteznek, és a többihez hasonlóan `Chloros:PixelScale` jelöléssel látják el. Az 1.3.0-ás verziók helyette 0..255-re *vágták* le, így minden ρ≈0,008 feletti érték 255-re lett kiegyenlítve, és kihagyták mind az `Chloros:PixelScale`, mind az `MicaSense:RadiometricCalibration` tupelt. **Ha a címke hiányzik egy LATTICE reflektanciafájlból, ne feltételezzünk skálát — dolgozzuk fel újra**, ahelyett, hogy olyan pixeleket osztanánk, amelyek soha nem voltak oszthatók.

#### Indexpixelek olvasása (az NDVI és az egyéb vegetációs indexek)

**Egy egész számú indexraster kép-specifikus kontrasztnyújtás, nem pedig −1…1-es leképezés.** Az `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` és `JPG (8-bit)` indextermékek — beleértve a 8 bites TIFF-fájlt is, amelybe a 8 bites felvétel kerül írásra — tárolják az

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

ahol az `IndexMin` / `IndexMax` **az adott képkocka saját legalacsonyabb és legmagasabb véges indexértékei**. A DN 0 nem −1, a teljes skála pedig nem +1.

A Chloros mindkét végpontot megjelöli, így a leképezés inverzibilis marad:

| Címke | Jelentés |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — az alábbi konvenció szerint |
| `Chloros:IndexMin` | a DN 0-ként tárolt indexérték |
| `Chloros:IndexMax` | a teljes skálaként tárolt indexérték |

Olvassa be őket, és fordítsa meg őket, normalizálva a tárolt DN-t 0…1-re a tárolt adattípus maximális értékével, hogy egy kifejezés minden egész számformátumot lefedjen:

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

Az olvasáshoz nincs szükség az `-config pix4d.config`-re — az exiftool önállóan olvassa be ezeket az XMP-tulajdonságokat. A konfiguráció csak az *íráshoz* szükséges.

> **`IndexMin == IndexMax`** azt jelenti, hogy a képkocka minden pixelje ugyanazt az indexértéket tartalmazta. A raszter a skála közepén van kitöltve, és nincs mivel osztani — vegyük a végpontot.

> **Ha nincs `Chloros:Index*` címke, az azt jelenti, hogy nincs megadott leképezés.** A LUT-export egy színezett renderelés, amelynek pixeljei a LUT küszöbérték-ablakához igazított színátmenetek, és egy olyan képkockának, amelyben nincs véges indexű pixel, nincs megadható tartománya. Semelyik esetben se feltételezzünk leképezést – ugyanaz a szabály, mint a hiányzó `PixelScale` esetében.**Mivel a nyújtás képenkénti, egy repülés két képkockája nem összehasonlítható DN-ként**— mindegyik a saját végpontjait hordozza, így ugyanaz az NDVI különböző képeken különböző DN-értékeket eredményez. Egy repülésen átívelő kvantitatív munkákhoz (zónás statisztikák, rögzített küszöbértékek, idősorok) exportálja a**`TIFF (32-bit, Percent)`**fájlt: az indexraszterek ott float32 típusú**indexértékekként** kerülnek rögzítésre, átméretezés és címkék nélkül. A formátum nevével ellentétben az indextermékek nem ×100-asok — ez a méretezés csak az egész szám típusú reflektanciára vonatkozik.

#### Unmix-korlátozó maszk (mely pixeleken korlátozták egy sávot)

A 3×3-as spektrális szétválasztás negatívvá teheti egy sávot — ott, ahol a fény alig tartalmazza azt a sávot, vagy ahol a jelenet kívül esik a kamera szétválasztási mátrixa által leírt tartományon. Ez nem valós érték, ezért az export 0-ra rögzíti — és az élő nézet is ugyanazt a 0-t mutatja. Egy sáv néhány képponton vagy szinte az összes képponton korlátozódhat; a maszk mindkettőt rögzíti. Az `Chloros:Band{R,G,B}UnmixClampFraction` azt mutatja, hogy a képkocka mekkora részén történt ez; a korlátozó maszk pedig azt, hogy **melyik pixeleken**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— egy `clamp_masks/` mappában**a termék**mappája**mellett**, soha nem benne (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`; egy `process` exportból származó `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`) — minden sugárzási, reflektancia és index termék esetében, amelyben legalább egy pixel korlátozásra került: a rögzítések és az `process` exportok egyaránt (valamint az `array-build-video --save-tiffs` képkockánkénti TIFF-jei). Egycsatornás `uint8`, a termék saját pixelrácsán (tájolás, ROI, binning és tömbigazítás is beleértve). Minden maszk megnevezi a termék kameráját (`Make` / `Model` / sorozatszám), a rögzítés idejét és az objektív címkéit, valamint felsorolja azokat a sávokat, amelyeket a bitjei jelölnek (`XMP-Camera:BandName`, 0. bit elsőként); az `ImageDescription` és `Software` jelölésekkel clamp maszknak minősül, és nem tartalmaz kalibrációt.
- **Bitek**: 0. bit = a termék R-csatornájának sávja (`Chloros:BandRName`), 1. bit = G (`BandGName`), 2. bit = B (`BandBName`). A beállított bit azt jelenti, hogy az adott sávban a pixel értéke 0-ra lett korlátozva — ezt, valamint az abból épített bármely indexet alacsony megbízhatóságúnak kell tekinteni.
- **`Chloros:UnmixClampMask`** jelölés szerepel minden olyan terméken, amelyen az unmix futott: a maszk elérési útja a termék saját mappájához viszonyítva (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, mindig `/`-el elválasztva — csatlakoztassa a kép mappájához a maszk megnyitásához), vagy `none`, ha egyetlen pixel sem lett korlátozva. Ha nincs címke, az azt jelenti, hogy a terméknél soha nem futott le az unmix (RGB- vagy monokróm kamerák, nyers/debayered szintek).
- Az `lattice capture` / `array-capture` a maszk elérési útját a mentett fájl alá írja (`clamp mask: …`) alá, és a rögzítési eredmények listája a képkocka `clamp_mask`-ként tünteti fel.
- Az `lattice index --live --save-multiband` a többsávos halmazhoz forráskameránként egy maszkot ír — lásd [`lattice index` — többsávos halmaz maszkok](#lattice-index--multiband-stack-masks). A videóhoz nincs maszk: az `array-build-video` ehelyett az egyes képkockák clamp-hányadát jelenti (lásd [`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst)).

Mivel egyetlen termékmappa sem tartalmaz maszkot – semmilyen szinten –, egy termékmappa (`radiance/`, `reflectance/`, egy `process` export `*_Images/` mappája) átadható a Pix4D-nek, a Metashape-nek vagy más fotogrammetriai szoftvernek úgy, ahogy van — az almappákkal együtt —, és a szoftver kizárólag képfájlokat fogad el. Adja át neki a termékmappát, ne a fölötte lévő mappát (az ugyanis tartalmazza az `clamp_masks/` mappát és minden egyéb exporttípust is). Az Chloros saját importálási funkciója kihagyja a maszkokat. Maszk olvasása: az `(mask & 1) != 0` azt jelenti, hogy „az R-csatorna sávja itt korlátozva volt”, az `& 2` a G-t, az `& 4` pedig a B-t jelenti.

#### Az EXIF-adatok átvitelre kerülnek az exportba

Az `process` a forrásfelvétel **GPS-blokkját és annak ExifIFD-jét** minden termékre, így az
export során az `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` és
`CameraSerialNumber` fájlokat a georeferenciálás mellett.

**Az `FocalLength` nem opcionális a fotogrammetria szempontjából.** A Pix4D a földi mintavételi távolságot a
fókusztávolság és a magasság alapján számítja ki; a címke hiányában rendkívül hibás méretarányra tér vissza. Egy
49 felvételből álló narancsligeti repülés során a hiányzó címke miatt egy 411 m × 160 m-es területből egy rekonstruált
47,8 km × 13 km-es területté – egy 455 MP-os ortofotóvá, amelyen többnyire „nodata” értékek szerepeltek, és amelyet ezután mozaikolási problémának és
BigTIFF-problémának tulajdonítottak, mielőtt bárki ellenőrizte volna a GSD-t. Ha az ortofotója hihetetlenül nagy
méretarányú, először futtasd le az `exiftool -FocalLength` parancsot az exportált terméken.

A másolat szándékosan **nem** az `-all:all`: az IFD0'szerkezeti címkéi megszakítják a LATTICE kimenetét, amikor
másolják őket, az `ExifImageWidth` / `ExifImageHeight` fájlokat pedig azért zártuk ki, mert azok a
*forrás* felvételt írják le – egy átméretezett export egyébként olyan méreteket hordozna,, amelyek ellentmondanak a saját raszterének. Az XMP-t nem másolják, hanem közvetlenül írják, mert az ExifTool
elveti az azonos meghívású XMP-címkéket, amikor az XMP-blokkot másolják (ami elveszítené a MAPIR kalibrációs címkéket).

### A kimenetek helye

A termékek **a projektmappa alá kerülnek, kameránként, majd fájlformátum szerint csoportosítva**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

A LATTICE kameramappája az `LATT-<sensor>-<lens>-F<filter>` (megfelel a felvétel EXIF
`Model`), a Survey3 esetében pedig `<model>_<filter>` – két olyan kamera, amely ugyanazt az érzékelőt és szűrőt használja, de
objektívjük eltérő, külön fa struktúrában marad, mivel a vignettálás, a látómező és a torzítás is eltérő. A formátum
mappa a következő: `--format`: `tiff16`, `tiff8`, `png8`, `jpg8`, vagy `tiff32` az
`TIFF (32-bit, Percent)` esetében.

> **Minden exportált termék megtartja a FORRÁS fájl nevét.** Az
> `capture_…_raw.tif` sugárzási adatainak exportja továbbra is `capture_…_raw.tif` néven szerepel — csupán az
> `tiff32/Radiance_Images/` mappában található. **A terméket a mappa azonosítja, nem a fájlnév**, így a
> `*radiance*.tif` névre történő globális keresés nem talál semmit; inkább a könyvtárat használja a kereséshez.

### Fényérzékelő-felvételek — kalibrált `.daq` + `.csv`

Az `process` a bemeneti mappában található `.daq` felvételeket is feldolgozza, és ehhez **nincs**
képanyagot igényel ehhez: egy önállóan repült DAQ-U / DAQ-M / DAQ-E már önmagában is teljes
felvételt jelent, és egy kizárólag `.daq` fájlokat tartalmazó mappa is érvényes bemenetnek számít.

A DAQ-ot **kalibrálás nélkül** is lehet rögzíteni – ezt teszik alapértelmezés szerint a nyilvános
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) felvevők
(`record_daq.py`) alapértelmezés szerint: a nyers érzékelőértékeket írják ki, és időbélyeggel látják el a fájlt, így a
Chloros **soros porton keresztül** (először a helyi gyorsítótárból,
majd a MAPIR-felhőből), és alkalmazza azt. Az `process` visszírja az eredményt:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

Az `.csv` minden leolvasáshoz egy sort tartalmaz: UTC időbélyeg, integrációs idő, teljes teljesítmény,
fotopikus/skotopikus lux, PPFD (és annak kék/zöld/vörös bontása), csúcshullámhossz, majd a
teljes spektrumot az érzékelő saját hullámhossz-rácsán. Az `.daq` fájl újraimportálásra kerül anélkül, hogy
másodszor is kalibrálni kellene.

Az `<name>` a felvétel saját fájlneve. Két, különböző mappákban található felvétel megoszthat
egyet – a két kamera által ugyanazon a
milliszekundumon belül rögzített, felvételenkénti mérési értékeket (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) –, és mindkettő megkapja a saját kimeneti adatait: az első megtartja az
az `<name>` nevet, a következő pedig a nevet **követően** hozzáadja a megkülönböztető mappákat
(`<name>__<capture>_daq_calibrated.daq`), így a felvétel dátuma és ideje továbbra is az első helyen szerepel.

Siker esetén a futtatás az `Light-sensor products written: N (calibrated .daq + .csv)` eredményt jelenti.
A zárójelben szereplő szöveg leírja, hogy mi került ténylegesen írásra, így a következőképpen olvasható:
`(RAW COUNTS — this sensor has no calibration bundle)` egy csomag nélküli érzékelő esetében, és
`(N calibrated, M raw counts)` egy mindkettőt tartalmazó mappa esetében. A háttérrendszer saját
`[DAQ-EXPORT]` és `[RUN-SUMMARY]` fejlécének megfogalmazása ugyanúgy alakul ki — a
három közül egyik sem nevezheti kalibráltnak a nyers exportot.

Egy olyan DAQ-U / DAQ-M / DAQ-E felvétel, amelynek kalibrációs csomagja nem tölthető le – mert
offline állapotban van, vagy az adott érzékelőhöz nincs kalibrációs fájl –, **ok megjelölésével kihagyásra kerül** egy
`[DAQ-EXPORT]` sorban, és soha nem kerül kiírásra „kalibrált” fájlként, amely nyers számlálási adatokat tartalmaz.
Csatlakozzon az internethez, és futtassa újra a felvételt. Az ok az, amelyet az olvasó ténylegesen
megállapított az adott fájl esetében (olvashatatlan sémakészlet, nincs csomag, írási hiba), és a futtatás
összefoglalója **különálló** okokat sorol fel — ha húsz fájlt egy ok miatt hagytak ki, az egy
okként jelenik meg, nem pedig húsz ismétlésként. A CLI a futtatás végén megnevezi az egyes kihagyott felvételeket
és azok okát:

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

Egy olyan felvételeket tartalmazó mappa, amelyből **egyetlen** sem kerül exportálásra, hibaüzenetet eredményez: lásd
[A képeket nem író futtatás hibaüzenetet eredményez](#a-run-that-writes-no-images-fails).

Az a felvétel, amelynek neve `_calibrated` vagy `_raw` (`field_raw.daq`) végződéssel zárul, **nem**nem**importálható**: ezeket a neveket az `process` adja a saját kimeneteinek, és egy kimenet, amelyet
felvételként olvasna vissza a rendszer, kétszer lenne számolva. A CLI ezt jelzi, és megnevezi a fájlt —
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
— ahelyett, hogy üres mappát jelentené. Ilyen felvételt nevezzen át feldolgozás előtt.

#### A DAQ-A felvételek nyers számlálási adatokként kerülnek exportálásra

A **DAQ-A** család a sorozatszámonkénti csomagrendszer bevezetése előtt jelent meg, és nincs kalibrációs csomagja,
amelyet be kellene tölteni – helyette a terepen, egy reflektancia-célponttal kalibrálják, ezért
soha nem is volt rá szüksége. Ha ezeket a felvételeket elutasítottuk volna, akkor egyáltalán nem lett volna módjuk
az adatok kinyerésére, ezért **más néven** exportálódnak:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Egy másik fájlnév, nem pedig a fájlon belüli jelölés, mert az adatnak meg kell maradnia,
amikor puszta névként továbbítják e-mailben. Az `.csv` fejlécben az
`raw spectral sensor counts (NOT irradiance)` szerepel, és figyelmeztet arra, hogy az értékek
**a** fájlon belül összehasonlíthatók – pontosan erre használja őket a célalapú kalibrálás –,
de nem az érzékelők között. A teljesítményfüggő fotometrikus oszlopok (teljes teljesítmény, fotopikus és
skotopikus lux, PPFD) **NULL** értékkel kerülnek rögzítésre, ahelyett, hogy a számlálásokból integrálnák őket, és a futtatási
összefoglalóban az `RAW COUNTS` szerepel, így a naplóban „exportált” felirat a naplóban nem értelmezhető besugárzási intenzitásként.

A régebbi **v1.01 / v1.02** felvételek (amelyeket a DAQ-A-SD ír) nem tartalmaznak olvasásonkénti időpontot,
csak a fájl írási idejét. A kép↔lefelé irányuló sugárzás-egyeztető továbbra sem fogadja el őket – egy
képkocka és az írási idő összevetése láthatatlanul hibás lenne –, de az exportáló program beolvassa őket, és
a CSV-fájlban az `clock=daq_created_on` formátumban jelenik meg, így a termék jelzi, melyik órához tartozik.

### Megjegyzések

- Az `process` automatikusan felismeri, hogy a mappa Survey3, LATTICE vagy vegyes típusú-e.
- Az előrehaladás Server-Sent Events-en keresztül történik; a CLI szálankénti élő előrehaladást jelenít meg (Felismerés, Elemzés, Feldolgozás, Exportálás).
- Linux/Jetson esetén a CLI ellenőrzi a swap-területet, és nagy mappák feldolgozása előtt figyelmeztetést jelezhet. A textúraérzékeny debayer alacsony fogyasztású Jetson-készülékeken (Nano, Orin Nano) automatikusan GPU-frekvencia-korlátozást is alkalmaz.
- Siker esetén a futtatás jelentést ad arról, hogy hány képi terméket írt ki (`Image products written: N`, illetve `(plus M clamp masks)`, ha az unmix bármely pixelt korlátozott).
- **Az `process` biztonságosan megosztja a futó háttérprogramot.** A már futó háttérprogramot csak akkor indítja újra automatikusan, ha az a korábbi projekt betöltése után tétlen állapotban van, és nem tart fenn kamerákat, tömböket vagy DAQ-okat. Azokat a háttérfolyamatokat, amelyek bármelyiket tartalmazzák (a tömböt streamelő grafikus felület, egy SDK-parancsfájl, egy DAQ-felvétel), változatlanul használja, soha nem indítja újra, így az `process` egy élő tömb mellett is futtatható. Ha a háttérfolyamat folyamatban lévő futást jelent, az `process` legfeljebb 15 másodpercig vár a befejezésére (egy éppen befejeződött futtatás törlése körülbelül egy másodpercet vesz igénybe), majd elutasítja a műveletet az `Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).` hibakóddal. A globális `--restart` jelző továbbra is új háttérrendszert kényszerít, és lezár minden rajta futó kamera-, tömb- vagy DAQ-munkamenetet. A Linux systemd szolgáltatás által futtatott háttérrendszert soha nem indítják újra: az `process` használja azt, és az `--restart` ehelyett az `sudo systemctl restart <unit>` üzenetet írja ki (lásd [A háttérprogram szolgáltatásként való futtatása](#running-the-backend-as-a-service-headless)). Ugyanez vonatkozik a Chloros-alkalmazás saját háttérprogramjára is: az alkalmazás megnyitott állapotában az `process` használja azt, és az alkalmazás projektje helyett a sajátját tölti be (ha helyette egy háttérprogramot indítanánk, az kizárná a megnyitott alkalmazást).
- **A futás közbeni feldolgozás megtartja a saját projektjét.** Amíg egy feldolgozási futtatás folyamatban van — amelyet az alkalmazás, az `process`, egy SDK-parancsfájl vagy az `chloros-mcp` indított el —, a háttérprogram nem engedélyezi fájlok létrehozását, megnyitását, azokba való importálást, hozzáadását vagy eltávolítását, áthelyezését, törölni vagy módosítani a projekt feldolgozási beállításait, HTTP 409 `PROCESSING_IN_PROGRESS` és `A processing run is in progress on project '<name>', so ... now is refused` hibakóddal; egy `process`, amely ezt megkapja, kilép az `1`-et kap, kilép az `1`-ből. A megjelenítési beállítások (miniatűr méret) továbbra is mentésre kerülnek. A feldolgozási mód és a projekt célreflektancia-beállításai ugyanúgy elutasításra kerülnek (a futtatás megtartja azt a módot, amelyben elindult). A futás közben a projektbe mentett felvétel (az alkalmazásból, a CLI-ből, az SDK-ből vagy az `chloros-mcp`-ből) a szokásos módon mentésre kerül, de csak a futás befejezésekor kerül be a projekt fájllistájába; ezt jelzi a visszajelzés is (`project_add`), akárcsak a projektbe beillesztett, leállított fényérzékelő-felvétel. Korábban egy második kliens a futás közben átválthatott a megnyitott projektre, és a futó futás átkerült a másik projektbe.
- **A Ctrl+C a háttérben állítja le a futást, nem csak az CLI-ben.** A CLI megkéri a háttérprogramot, hogy állítsa le a feldolgozást, kiírja az `Waiting for the backend to stop (Ctrl+C again to stop waiting)...` üzenetet, és legfeljebb 120 másodpercig vár az `Processing stopped.` üzenetre, így az `chloros-cli process … && next` nem indítja el az `next` parancsot egy olyan háttérrendszeren, amely még mindig a futtatással van elfoglalva. Egy második Ctrl+C befejezi a várakozást. Az `process` ezután kilép az `130`-ből, ahogyan minden megszakított parancs teszi.

#### A képeket nem író futtatás sikertelen

Ha termékeket kért, és a futtatás **egyet sem** írt ki — csak az `project.json` és
az `calibration_data.json` —, az `process` ezt hibaként kezeli: kinyomtatja az
`Processing finished but wrote no image products.` üzenetet, és **nem nulla értékkel lép ki**, így egy szkript
felismerheti a hibát. Az üzenet megnevezi a projektmappát és a szokásos okokat:

- a bemeneti mappát nem ismerte fel rögzítésként (ellenőrizd az elrendezést és az `--input-level`-et), vagy
- minden kért terméket kihagytak, mivel azok az adott kamerákra nem alkalmazhatók (pl.
  RGB-es kameráktól sugárzási/visszaverődési értékeket).

Futtassa újra az `--verbose` parancsot, és ellenőrizze a háttérnaplóban az `[LATTICE-EXPORT]` / `[EXPORT-CHECK]` sorokat,
amelyek kameránkénti kihagyásokat magyarázzák, amelyek egyébként nem jelennek meg a CLI kimenetében.

Egy szándékosan kizárólag metaadatokra korlátozott futtatás — minden termék kikapcsolva és `--indices` nélkül — továbbra is
**sikeres**, mivel ebben az esetben az üres kép kimenet a helyes eredmény.

#### Az a futtatás, amelynek exportált adatai nem ellenőrizhetők, sikertelennek minősül

A futtatás után az `process` az ExifTool segítségével ellenőrzi az exportált fájlok címkéit. Ha az ExifTool nem futtatható, akkor
a következő üzenetet írja ki: `Export ellenőrzés MEGHIÚSULT: Az ExifTool nem futtatható, ezért az M exportált fájlból N címkéje
nem kerültek ellenőrzésre (lásd az [EXIFTOOL] sorokat a háttérnaplóban)` and `A kimeneti fájlok létrejöttek és
a helyükön maradnak; futtassa újra a folyamatot az ellenőrzéshez.`, does not print `A feldolgozás befejeződött!` üzenetet ír ki, és **nem nulla kóddal lép ki**. A
kimeneti fájlok megmaradnak.

#### Melyik felvételeket veszi figyelembe egy futtatás

Az `process` futtatása előtt befejezett felvételek szerepelnek a kimenetben. A futtatás során mentett felvételek nem; az `process`
figyelmeztet, hogy hányat hagyott ki, és az újrafuttatás során azokat is belefoglalja.

#### A futtatás sikertelen, ha egy kép elveszett

Ha egy futás során egy kép meghiúsult, az `process` kinyomtatja az `N image(s) failed and were not exported` üzenetet a
leggyakoribb ok megjelölésével, nem nyomtatja ki az `Processing complete!` üzenetet, és **nem nulla kóddal lép ki**. A többi termék
elkészül és a helyén marad, a háttérnapló pedig minden hibát rögzít.

Ugyanez vonatkozik a **kizárólag fényérzékelővel végzett futtatásra** is: egy `.daq` felvételeket tartalmazó mappa definíció szerint nem tartalmaz exportálható képanyagot,
és a futtatást a helyette írt, kalibrált `.daq` / `.csv` alapján értékelik.
Ha **semmit** sem írt – minden felvételt kihagyott –, akkor kiírja az
`Processing finished but wrote no light-sensor products.` kódot, minden felvételt elnevez a
kihagyás okával, és **nem nulla értékkel lép ki**.

---

## `chloros-cli login`

Hitelesítse ezt a gépet egy Chloros+ felhőfiókkal. A hitelesítő adatok biztonságosan tárolódnak az `~/.chloros/user_session.json` fájlban.

```
chloros-cli login EMAIL PASSWORD
```

### Példák

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (a jelszó egy részének elhagyásával vagy másolásával). 401-es hiba esetén a CLI automatikusan újrapróbálja az `$$` parancsot-után, majd a jelszó duplikátumaitól megtisztított felével; ha az újrakísérlet sikeres, bejelentkezik, és kinyomtatja a következő alkalommal használandó helyes egyszerű idézőjel-szintaxist.

> **Headless/parancsfájl-alapú használat: a gyorsítótárban tárolt munkamenet hiánya bejelentkezési parancssort jelent.** A core parancsok (`process`, `status`, `export-status`, `time-sync`, …) futtatása gyorsítótárazott munkamenet nélkül az `Email:` / `Password:` megadását kéri a stdin-en, mielőtt továbbhaladna. Ha a stdin zárva van (cron, systemd-egység), a program azonnal hibaüzenetet ad az `login cancelled` kóddal; ha a stdin egy nyitott terminál vagy cső, akkor a program várja a bemenetet. Futtassa az `chloros-cli login EMAIL PASSWORD` parancsot egyszer minden gépen, mielőtt headless feladatokat ütemez.

---

## `chloros-cli logout`

Törli a gyorsítótárban tárolt munkamenetet, és a következő híváskor új bejelentkezést kényszerít.

A feldolgozás futása közbeni kijelentkezés nem állítja le a folyamatot: a futás abban a feldolgozási módban fejeződik be, amelyben elindult, a következő futás pedig a szokásos feldolgozási módot használja. Ezt jelzi az `logout` (`The processing run in progress finishes in <mode> mode; the next run uses standard`), és az alkalmazás is ezt jelzi, amikor kijelentkezik .

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Megjeleníti az aktuális licencszintet (Iron/Copper/Bronze/Silver/Gold), a hitelesített felhasználót és az eszközhoz kötött fiókok számát.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

A Thread-4 exportálás aktuális állapotának lekérdezése. Biztonságosan hívható **a**`process` futása**alatt** egy másik shellből.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Beállítja a CLI megjelenítési nyelvét (38 nyelv támogatott, beleértve a CJK, RTL és indiai nyelveket is). Azoknál a régebbi konzoloknál, amelyek nem tudják megjeleníteni a szkriptet, zökkenőmentesen angolra vált.

```
chloros-cli language [LANG_CODE] [--list]
```

### Példák

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Projektmappa-parancsok

Ezek az alapértelmezett projektmappa helyét kezelik (amelyet a grafikus felülettel is megosztanak).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

Az `get-project-folder` parancs kinyomtatja azt a mappát, ahová az új projektek kerülnek. Ha a beállított mappa már nem létezik (törölték, átnevezték, vagy egy nem csatlakoztatott meghajtón található), a CLI, akárcsak a grafikus felület, az alapértelmezett mappára vált vissza: az alapértelmezett mappa kinyomtatásra kerül a stdout-ra, a hiányzó mappát megnevező figyelmeztetés a stderr-re kerül, és a kilépési kód `0` lesz.

---

## `chloros-cli update`

Csak Linux/Jetson esetén. Ellenőrzi az `version_url` verziót az `/etc/chloros/update.conf`-hez képest, és ha létezik újabb verzió, letölti a megfelelő `.deb` fájlt, és azonnal telepíti azt az `sudo dpkg -i` paranccsal (megerősítést nem kér; az `sudo` parancs esetén a rendszer jelszót kérhet). Jetson rendszeren csak a JetPack 6 rendszer számára kínálja fel a frissítést. Windows-en a parancs nem hajt végre semmilyen műveletet, és a következő üzenetet jeleníti meg: `up to date`: frissítés új telepítővel.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Linux/Jetson rendszeren a CLI **minden indításkor automatikus frissítésellenőrzést** is futtat (nem blokkoló, soha nem késlelteti a parancsot): kiolvassa az `/etc/chloros/update.conf` üzenetet, az eredményt 1 órára gyorsítótárba menti az `~/.chloros/update_cache.json` üzenetben, és kiírja az `Update available: vX.Y.Z / Run: chloros-cli update` üzenetet, ha újabb verzió létezik. Hiba esetén és Windows rendszeren csendben kihagyja; az `CHLOROS_DISABLE_UPDATE_CHECK=1` kikapcsolja. Az 1 órás gyorsítótár miatt az egy órán belül kiadott verzió még mindig naprakésznek tűnhet.

---

## `chloros-cli selftest`

7 lépésből álló gyors tesztet futtat: verzió, port elérhetősége, háttérprogram indítása, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), zajszűrő modell jelenléte, a CUDA+zajszűrő készenléte. Futó háttérprogramot használ, vagy a teszt céljából elindít egyet, majd utána leállítja. Kinyomtatja az `N/7 checks passed` üzenetet és a sikertelen ellenőrzéseket, és csak akkor lép ki az `0` állapotból, ha mind a hét ellenőrzés sikeres; a CUDA+zajszűrő ellenőrzés sikertelen egy olyan számítógépen, amelyen nincs CUDA GPU (lásd [A telepítés ellenőrzése](#verify-the-install)).

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTP grandmaster állapot és vezérlés. A Chloros-gazdagép futtatja a PTP grandmastert; a LATTICE kamerák és a DAQ-E egységek ehhez kapcsolódnak az eszközök közötti időbélyegek érdekében (körülbelül 1 ms-os késleltetéssel az eszközök között). A PTP **időbélyegeket** hangol össze: a kamerasor kamerái azért exponálnak egyszerre, mert az M8 hardveres trigger indítja el őket (a modulok között 100 µs alatt), nem pedig a PTP miatt. Az alparancs nélküli `chloros-cli time-sync` az `status` parancsot futtatja.

| Alparancs | Leírás |
| --- | --- |
| `status` | Megjeleníti a grandmaster állapotát, a BMCA prioritásokat, az óraazonosítót, valamint azt, hogy a csomagjai eljutnak-e a hálózatba: az `health` az `OK` (`STOPPED`, ha a grandmaster nem fut), vagy az `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` egy `health_detail`-szel, majd egy `WARNING:` sor kerül kinyomtatásra a stderr-re (a stdout továbbra is JSON formátumú marad). Az `sync_sent` / `announce_sent` csak a kilépett üzeneteket számolja; az `*_send_errors` és az `last_send_error` pedig azokat, amelyek nem léptek ki. Miután egy kamera kapcsolata megszakad, majd helyreáll, a grandmaster önállóan újjáépíti a socketjeit (`rebinds`, `last_rebind`). |
| `peers` | A Delay_Req-en keresztül észlelt szolgaeszközök listája (kamerák + DAQ-E érzékelők). |
| `cameras` | Kameránkénti PTP-állapot (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | A grandmaster folyamat újraindítása. |
| `set-priority --priority1 N --priority2 N` | A BMCA prioritások felülírása. |

A grandmaster a kamera hálózati kártyáin keresztül (bármelyik adapteren, amelynek címe tartalmazza az `169.254.x.x` címet). Ha egyik sem elérhető, akkor csak az alapértelmezett útvonal adapterére tér vissza, amennyiben az vezetékes, soha nem Wi-Fi-re vagy virtuális adapterre (Tailscale, WSL), mivel egyetlen kamera vagy DAQ-E nem áll ilyen mögött. Ha nincs mit bejelentenie, akkor socketek nélkül fut (`health` `NO_SOCKETS`), és a következő kamera vagy DAQ csatlakozáskor újraépíti azt az akkor aktív hálózati kártyán; az `time-sync restart` ezt azonnal megteszi.

A grandmaster saját folyamatban fut, a háttérprogram mellett (`own_process` / `process_pid` az `time-sync status`-ben), magas prioritással a Windows rendszeren (`process_priority`), így egy leterhelt háttérprogram (feldolgozási futtatás, export, több parancs egyszerre) nem késleltetheti az általa rögzített időpontokat; korábban egy ilyen késleltetés egyszerre eltolhatta minden kamera óráját. Ez a folyamat a háttérprogrammal együtt ér véget; ha az nem indítható el, a főóra a háttérprogramon belül fut, mint korábban (`own_process` false, és a háttérprogram naplója is ezt jelzi).

### Példák

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

LATTICE kameravezérlés. Minden alparancs a Chloros háttérrendszeren keresztül fut; a háttérrendszer kezeli a kamerakészletet, így a következő CLI-hívások ugyanazt a nyitott kezelőt használják újra.

### Általános opciók (a legtöbb alparancsban közös)

| Jelző | Leírás |
| --- | --- |
| `-d, --device N` | Kamera index (alapértelmezett: 0). |
| `-s, --serial SN` | Konkrét sorozatszám; felülírja az `--device` parancsot. |
| `--serials SN1,SN2,…` | Vesszővel elválasztott sorozatszámok többkamerás működéshez. A [többkamerás átmeneti parancsok](#multi-camera-transient-sessions) csak ezeket a kamerákat nyitják meg, az első indítja el a kioldást. |
| `--all` | Minden felismert kamerán végrehajtja a műveletet. |
| `--exposure US` | Expozíciós idő mikroszekundumokban. |
| `--gain DB` | Erősítés dB-ben. |
| `--pixel-format FMT` | pl. `BayerRG8`, `BayerRG12`; `Mono8`, `Mono12` egy M3M-en. A név határozza meg a bitmélységet a kamera saját családján belül: egy M3M-en egy Bayer-név határozza meg a mélység Mono formátumát (az `BayerRG8` beállítja az `Mono8`-et), egy M3C-n pedig egy Mono-név a Bayer-formátumot (az `Mono12` beállítja az `BayerRG12`-et). |
| `--width N` / `--height N` | Képméretek. |
| `--preset {default,high_quality,high_speed,triggered}` | Beállítási előre beállított értékek alkalmazása: pixelformátum, expozíció, erősítés és képkockasebesség (`high_speed`: `BayerRG8`, 5 ms, 6 dB, 60 fps; `high_quality`: `BayerRG12`, 20 ms, 0 dB; `lattice presets` felsorolja őket). M3M-en a formátum ugyanolyan mélységű Mono: az `high_speed` rögzíti az `Mono8`-et, `high_quality`-et, valamint az `triggered`-et és az `Mono12`-et. A hozzá tartozó jelző (`--exposure`, `--gain`, `--pixel-format`) felülírja az előre beállított értéket, és ha az előre beállított érték nem tartalmazza a beállítást, akkor a kamera saját beállítása marad érvényben. Minden szabadműködésű, kivéve az `triggered`-et, amely a 2. vonalon jelentkező hardveres élre érzékenyen állítja be a kamerát — ha semmi sem vezérli azt a vonalat, a kamera örökké várni fog, ahelyett, hogy felvételt készítene. |
| `-o, --output DIR` | Kimeneti könyvtár (alapértelmezett: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | GVSP-csomagméret. Az `auto` ICMP+GVSP-próbákat futtat; `jumbo` = 9000; `standard` = 1500. |

### LATTICE kamera első csatlakozási munkafolyamat

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Alparancs-referencia

#### Felderítés és információk

| Alparancs | Cél |
| --- | --- |
| `lattice info` | A csatlakoztatott kamerák listája (gyártó, modell, sorozatszám, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | A gazdaszerver elemzése az optimális kamerakonfiguráció érdekében. Az `--no-discover` kihagyja a kamerák felderítését (gyorsabb, kizárólag a hálózati kártyát elemzi). |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | A hálózati kártya beállításainak ellenőrzése/javítása (Linux rendszeren a kernel socket-fogadó puffereinek ellenőrzése is — lásd [Linux-gépek](#linux-hosts--socket-receive-buffers)); a sávszélesség és a képkockasebesség (FPS) becslése. |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | Stabil sémájú háttérhálózat-képesség + tömbajánlás (visszaadja az `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). Az `--width` / `--height` alapértelmezés szerint a teljes érzékelőt veszi, `2048` × `1536`. Az `0`-et az `ok`, az `auto_capped_fps` és `auto_shrunk`, valamint az `1`-et az `needs_force_slip` és az `error` helyett, `--json` jelenlétével vagy anélkül. A háttérrendszerhez az `--backend-url` értéknél érkezik (alapértelmezés szerint `http://127.0.0.1:5000`), nem pedig az `CHLOROS_BACKEND_URL` értéknél. Az `auto_capped_fps` megtartja a kért felbontást, de korlátozza a cél fps-t — olvassa be az `recommended.recommended_target_fps`-et, és adja át csatlakozási célként; kezelje sikernek, nem pedig hibaként. Az `auto_shrunk` csak akkor jelenik meg, ha a kért képkocka semmilyen képkockasebességgel nem szállítható; a connect soha nem kicsinyíti önállóan a képkockákat. |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | „Mi lenne, ha” elemzés kamerák megnyitása nélkül (`--width` / `--height` alapértelmezett `2048` × `1536`). Mind ez, mind az `network-analysis` azt vetíti előre, amit a connect fog streamelni: `--pixel-format` nélkül a connect által alkalmazott kameránkénti formátumok (Mono12 / BayerRG12), míg az `--packed auto` (alapértelmezett) esetén a [tömörített 12 bites](#packed-12-bit-transport---packed) döntés, amelyet a connect ezen a számítógépen hozna — így a megadott fps az a sebesség, amellyel a tömb fut. Mindkettő kinyomtatja a csomagolási döntést. **Az `--n-active` a hálózaton lévő kamerák teljes száma, nem csak ennek a tömbnek a száma**— növelje meg, ha önálló kamerák egyszerre továbbítanak, vagy ha a hálózati sávszélességet olyan igény alapján számolják ki, amely alulbecsüli azok számát (alapértelmezett: `len(--models)`). Mindig kinyomtatja az összesített `Wire budget:` (igényelt MB/s vs. ütközésmentes felső határ) és `Max cameras:` sorokat, és jelzi az `** OVER-SUBSCRIBED**`-et, ha a tömb túlterheli a hálózatot — lásd [Tömb fps és burst modell](#array-fps--burst-model). |
| `lattice gpu` | A GPU állapotának megjelenítése. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | A kamera firmware-jének ellenőrzése vagy frissítése. A helyi `.fwa` kiválasztás rögzítve van: az `firmware/<MODEL_PREFIX>/`-ben található, a build `MIN_FIRMWARE_VERSION`-jéhez illeszkedő fájl akkor kerül feltöltésre, ha jelen van (csak a legmagasabb verzió, mint tartalék), így a lemezen tárolt újabb gyártói kép addig inaktív marad, amíg azt a rögzítést fel nem oldják — a szándékosan újabb kiadások az aláírt AWS-manifeszt révén jutnak el az egységekhez, ami az újabb verziók esetén előnyösebb. |
| `lattice presets [--apply NAME]` | A kamera előre beállított értékeinek felsorolása vagy alkalmazása. Az `--apply` meghatározza az előre beállított értékek nevét (pixelformátum, expozíció, erősítés, képkockasebesség, kioldás), és kinyomtatja azokat a beállításokat, ahogyan a kamera azokat az alábbiak után olvassa be: egy M3M-en az előre beállítás mélységének Mono formátuma; az a képkockasebesség, amelyen fut, és amely alacsonyabb lehet, mint amit az előre beállítás kér (az `high_speed` 60 fps-t kér; egy 1 Gbps-es kapcsolaton lévő teljes képkocka-kamera körülbelül 35-öt futtat). Minden egyéb beállítás, amelyet az „open” parancs írt, a talált értékek szerint kerül visszaállításra. Az `--apply default` parancs a kamerát gyári alapértékekre állítja vissza: gyári beállítások szabadon futó automatikus expozícióval a Line2-n, a kapcsolat átviteli sebességének korlátozása és a hibajavítás kikapcsolva, egy RGB-kamera kalibrált fehéregyensúlyának megőrzésével (más típusoknál egységes érték). Semmi sem kerül mentésre a kamera memóriájába, így a következő bekapcsoláskor a saját memóriájából indul, mint korábban. A rögzítés vagy a csatlakozás `--preset` parancsaként az `default` nem előre beállított érték. |
| `lattice status` | Élő kameraállapot. |

#### Felvétel

| Alparancs | Cél |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | Egyetlen képkocka. **Alapértelmezés szerint minden exporttípust elment**(`--processing all`); lásd [Rögzítési exportszintek](#capture-export-levels-the-all-default). Az `--processing` az `all`, `raw`, `debayered`, `radiance`, `reflectance` vagy `display` neveket (az `preview` név az `display` esetében elfogadott); bármely más nevet elutasít a kamera megérintése előtt (kilépés: `1`). Az `--levels` egy explicit részhalmazt ment el (felülírja az `--processing`-et). A felvételhez megnyílik egy olyan kamera, amelyet Chloros nem tart kézben, és expozícióját először a jelenet alapján állítják be (legfeljebb néhány másodpercig), ahogyan azt a [tranziens parancsok](#multi-camera-transient-sessions) is teszik; amikor a felvétel véget ér – függetlenül attól, hogy hogyan ér véget –, azt a kamerát visszaállítják az eredeti állapotába (formátum, expozíció, színfeldolgozási folyamat, minden olyan beállítás, amelyet a felvétel megváltoztatott – a [Többkamerás (átmeneti munkamenetek)](#multi-camera-transient-sessions) alatt található lista). A felvétel olvasásához használt DAQ-adatokat `.daq` sidecar fájlként menti el, amely tartalmaz egy kizárólag nyers adatokat tartalmazó felvételt; az `--no-daq` / `--daq-tolerance-ms` fájlokkal lehet módosítani ([A rögzítést olvasó DAQ elmenti](#the-daq-reading-a-capture-saves)). Az `--force-daq` továbbra is elfogadott. `--jpeg-quality` = JPEG-minőség 1–100 (alapértelmezett: 95).**A megnyitott projektbe mentett fájlok csatlakoznak ahhoz.** Akárcsak az alkalmazásban, egy olyan rögzítés, amelynek fájlai a Chloros-ban megnyitott projekt mappájába kerülnek, hozzáadja azokat a projekt fájllistájához; a máshová mentett fájlokat nem érinti. A parancs egy sort jelenít meg, amely ezt jelzi (`Project: ...`), vagy – ha éppen feldolgozás folyik – azt, hogy a fájlok visszatartásra kerülnek, és a feldolgozás befejezése után csatlakoznak a projekthez. Ugyanezt teszi az `multi-capture`, az `array-capture` (`--smart` is) és az `project capture` parancsok is. |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | Adatfolyamot ír a lemezre a Ctrl+C billentyűkombinációig. Az `tiff` futtatásképkockái alkotják az `raw` fájlt, amely egy tömb, amit a kamera rögzít: az érzékelő mozaikja, egy sík a saját értékeivel, tömörítetlenül, a kalibrációs csomaggal (egy 2048 x 1536-os, 12 bites képkocka 6,0 MiB, plusz a csomag, függetlenül a jelenettől), így az `process` ezekből indul ki. Az `png` és az `jpg` 8 bitet tartalmaz, így ezek a sorozatok megtekintésre szánt, demosaicingolt képkockák, amelyekből az `process` nem indul ki. DAQ-hoz kötve (`daq pool-bind`) vagy pooling esetén minden képkocka megkapja a saját `.daq` és `.match.json` fájljait a sorozat `daq/` mappájában, mivel az `lattice capture` egyet ment; az `--no-daq` ezt kikapcsolja, és a leálláskor kinyomtatja, hány volt. Minden képkockát egyszer ír le, a teljes metaadatokkal együtt; Ctrl+C billentyűkombinációval a már rögzített képkockák a kamera elengedése előtt kerülnek írásra. A program leáll, amikor a meghajtó megtelik, kinyomtatja az `Failed:` / `Disk full:`-et írja ki, majd kilép az `1`-ből; minden olyan képkocka, amelyet nem tudott írni, szintén kilép az `1`-ből. A hardveres kioldásra beállított kamera (`TriggerMode` `On`) a futás idejére szabad futásra van állítva. A futás befejezésekor a kamera visszaáll a talált állapotába — a kioldási beállításai és minden, a futás során megváltozott beállítás (expozíció, erősítés, formátum, …) – kivéve az `--keep-settings` esetét, amely a futás beállításait a kamera saját memóriájába menti (szabad futás), vagy az `--clear-settings` esetét, amely a memóriát gyári beállításokra állítja vissza. Az a futtatás, amelyik nem rögzít képkockát, hiba: ha 10 másodpercig nincs képkocka, akkor magától leáll, kinyomtatja az `No frames:` üzenetet az ok megjelölésével, majd kilép (`1`). Azokat a képkockákat, amelyekkel az írók nem tudtak lépést tartani, elveszettnek számítják, és a leálláskor a rendszer kinyomtatja az `Dropped:` üzenetet az ok megjelölésével (a kamera képkockasebessége, az íróké és a soré); alacsonyabb képkockasebesség vagy 8 bites képpontformátum esetén mindet megőrzi. A parancs 2 másodpercenként jelentkezik be a háttérrendszerbe; ha a felvétel leállítása nélkül leáll (`kill -9`, memóriahiány), a háttérrendszer az utolsó bejelentkezés után 30 másodperccel leállítja, és visszaállítja a kamerát (`stop_reason` `client_gone`, és egy sor a háttérrendszer naplójában). Az `multi-stream` és az `align-stream` ugyanezt végzi. Ha a kamera megnyitása hosszabb ideig tart, mint amennyit a parancs vár (60 másodperc; 180 másodperc az `multi-stream` / `align-stream` esetében) — a kamera még mindig indul —, a háttérrendszer visszavonja a műveletet: a kamerát visszaállítják és bezárják, és egyetlen felvétel sem marad futásban (HTTP 504 `CLIENT_DEADLINE_PASSED`). |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Böngészőalapú élő MJPEG-előnézet: csatlakozik a kamerához, kinyomtatja a stream URL-jét, és megnyitja azt a böngészőben. Az `--ae-damping` beállítja az automatikus expozíciócsillapítást (0,4–100). |

##### Élő adatfolyamok (MJPEG; RTSP nélkül)

A Chloros **MJPEG formátumban HTTP-en**(`multipart/x-mixed-replace`, határérték: `frame`) formátumban a háttérrendszerből.**Az RTSP nem támogatott.** Nyissa meg az URL-t egy böngészőben, vagy olvassa be bármely MJPEG-klienssel (az OpenCV `cv2.VideoCapture(url)` funkciója, ffmpeg, VLC). A kamerát először a háttérrendszerhez kell csatlakoztatni (`lattice cam-connect`, `lattice array-connect`, `lattice viewer` vagy a grafikus felületen keresztül).

| URL (`http://127.0.0.1:5000` + …) | Adatfolyam |
| --- | --- |
| `/api/camera/stream/<serial>` | A képernyőn megjelenő előnézet, legfeljebb kb. 30 fps. Tömb elem esetén az `?max_h=` (alapértelmezés: `720`) és az `?jpeg_q=` (alapértelmezés: `80`) állítja be a magasságot és a JPEG-minőséget. Ez az az URL, amelyet az `lattice viewer` megnyit. |
| `/api/camera/<serial>/stream-annotated` | Ugyanaz az előnézet, a grafikus felhasználói felület átfedéseinek megjelenítésével: `zebra=1`, `crosshair=1`, `grid=3x3` (vagy `9x9`), `histogram=1`, `peaking=1`, `spot=1`; lekérdezés nélkül a projektben az adott kamerához mentett átfedési beállítások. |
| `/api/camera/<serial>/stream-reflectance` | sávonkénti 8 bites reflexió: ρ = pixel ÷ az`X-Chloros-Pixel-Scale` fejlécének része (255, hacsak az `refl_max` nem van beállítva). `fps` (alapértelmezett: `10`), `jpeg_q` (alapértelmezett: `95`), `max_h` (alapértelmezett: `0` = natív), `band=<name>`, `align=1`. JPEG, tehát veszteséges. HTTP 409-es hibakóddal elutasítva, ha a kamerának nincs reflektanciája (RGB-kamera, nincs kalibrációs csomag, nincs DAQ-leolvasás). |

- **Képkockák összehangolása a kamerák között.**Minden**tömb elem** által elküldött rész mindhárom adatfolyamon az `X-Chloros-Group-Id` (a tömb szinkronizált csoportjának számlálója), `X-Chloros-Frame-Id` és `X-Chloros-Timestamp-Ns` fejléceket hordoz. A különböző kamerákból származó, azonos `X-Chloros-Group-Id` értékkel rendelkező részek ugyanabból a kiváltóból származnak. Egy önálló kamera nem tartozik csoporthoz, és nem küld csoportazonosítót. A böngészők figyelmen kívül hagyják az egyes részek fejlécét; azokat az a kliens olvassa be, amely maga elemzi a többrészes adatfolyamot.
- **Szüneteltetve, amíg a tömbnézet nyitva van.** A tömb egyik elemének `/api/camera/stream/<serial>` azonosítója statikus „paused” képkockát küld, amíg a tömb egyesített élő adatfolyama nyitva van (például a grafikus felhasználói felület tömbnézete), és folytatja a működést, amikor az bezárul.

#### Érzékelő beállítása

| Alparancs | Cél |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | Bármely GenICam-csomópont olvasása/írása. `--set ExposureTime=…` (`Gain=…`) az `ExposureAuto` (`GainAuto`) kapcsolók nélkül, amelyek először kikapcsolják az automatikus módot, ahogyan az `lattice exposure --set` teszi; ha a kamera elutasít egy értéket, akkor az `Failed to set <node>: …` üzenetet ír a stderr-re, és kilép az `1`-ből. Az `--get` és az alapértelmezett csoportos nézet az `note:`-et az `ExposureTime` / `Gain` alatt jeleníti meg, amíg az automatikus mód fut — lásd [Az expozíció és az erősítés leolvasása automatikus módban](#reading-exposure-and-gain-under-auto-mode). |
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | Expozíció és AE. A `--set` expozíciót vagy a `--brightness` célt a rendszer a sajátodként kezeli (a Chloros automatikus expozícióvezérlése nem avatkozik bele); az erősítés változatlan marad, így automatikus üzemmódban a rendszer addig állítja be a célérték felé, amíg be nem állítod az `lattice gain` értéket. Az `--auto` visszaadja az expozíciót, az `--brightness-auto` pedig visszaadja a célt a Chloros automatikus célbeállításának. Jelölők hiányában kiolvassa az expozíciós csomópontokat, ugyanazzal az `note:` értékkel, mint az `configure --get`. Ha a kamera elutasít egy `--set` értéket, akkor az `Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)` üzenetet írja a stderr-re — az automatikus mód, amelyben az írás ki volt kapcsolva, helyreáll, ahelyett, hogy kikapcsolt állapotban maradna — nem ír ki `Exposure: … us (auto OFF)` sort, és kilép `1`. |
| `lattice gain [{--auto \| --off \| --set DB}]` | Erősítés és automatikus erősítés. Ha a kamera elutasítja az `--set` értéket, akkor az `Failed to set Gain: …` üzenetet írja a stderr-re, nem írja ki az `Gain: … dB (auto OFF)` sort, és kilép az `1`-ből. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | Érzékelő ROI és binning. A binning mindig **Átlag** — lásd alább. |
| `lattice format [--set FMT] [--list]` | Pixelformátum. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Hardveres/szoftveres indítás. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (jelzők hiánya = egyszeri fehér egyensúly) | Fehér egyensúly műveletek. Csak RGB/Bayer kamerák esetén; mono M3M esetén nem hajt végre műveletet (kihagyva). Az egyszeri művelet megtartja a kiszámított fehér egyensúlyt (`BalanceWhiteAuto` és az erősítéseket); minden egyéb beállítás, amelyet az open írt, visszaáll a talált értékre, és mindez akkor is, ha az egyszeri művelet nem futtatható. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | RGB-kijelző színfeldolgozási folyamat. Az `natural` (alapértelmezett) az olcsó élő feldolgozás; az `enhanced` hozzáadja a színszegély-eltávolítást, az élénkséget és a CLAHE helyi kontrasztot a teljes „hub-parity” megjelenés érdekében, körülbelül kétszeres képkockánkénti feldolgozási költséggel, így alacsonyabb **élő** képkockasebességet eredményez – a mentett felvételek mindkét esetben mindig a teljes feldolgozást kapják. Csak RGB/Bayer kamerák esetén; mono M3M esetén kihagyva. |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | Élő előnézeti telítettség/kontraszt (RGB-szűrős kamerák); az `100` semleges, az `--reset` mindkettőt 100-ra állítja. A mentett felvételeket ez nem érinti. Mono M3M esetén kihagyva. |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]` (nincs jelző = olvasás) | **Fényes részek** megjelenítése (RGB-szűrős kamerák): a legfényesebb, legfehérebb pixelek tompítása — pl. egy ragyogóan fehér felület, amely lapos fehér foltként jelenik meg. `0` = ki, `-100` = legerősebb. Az élő előnézet és a kijelzőszintű felvételek ezt követik; a sugárzás/visszaverődés soha nem, és nem tudja helyreállítani a túlexponált szenzordatait (ehhez csökkentse az expozíciót). Kihagyva a monokróm M3M-en. |
| `lattice filter [--list]` | Az Chloros által ismert szűrőmodellek felsorolása (`RGN-IMX265`, `OCN`, `NGB`, …). A kamera szűrőjét a modellkódból olvassa ki, és nem állítható be: az `--set NAME` csak megkeresi a modellt, majd kilép az `1`-ből az `setting the filter model is not supported`-szel. |
| `lattice power [--sleep]` | A tápellátási/hőmérsékleti csomópontok ellenőrzése; az alacsony fogyasztású készenléti állapot be- és kikapcsolása. Az `--sleep` alvó állapotba helyezi a kamerát (a kapcsolat átviteli sebesség-korlátozása bekapcsolva (`DeviceLinkThroughputLimitMode` `On`) a legalacsonyabb szinten, az energiatakarékos üzemmód bekapcsolva, ha a kamerának van ilyen) minden egyéb beállítását az eredeti állapotba állítja vissza; a sikertelen alvó üzemmód teljes egészében visszaáll. Az ébresztés a kapcsolatot olyan állapotba hozza vissza, amilyenben az alvó üzemmód találta (a korlát, az üzemmód és a csomagkésleltetés `GevSCPD`, amelyet a kamera 0-ra tart, amíg a korlát be van kapcsolva; a feljegyzést ezen a számítógépen tárolják, így a háttérrendszer újraindulása utáni ébresztés is megtalálja), vagy ha egy olyan kamera, amelyet ez a számítógép nem állított alvó üzemmódba, de alvónak tűnik (a korlátja a legalacsonyabb szinten van), akkor a korlátot kikapcsolják, és az `GevSCPD` értéket 0-ra állítják 80-nál (gyári beállítások); a küszöbérték alsó határ felett történő beállítása változatlan marad. A szonda és az ébresztés visszaállítja azt, amit a nyitásuk írt. |

> **A binning mindig Átlagos.** Az `lattice resolution --binning N` beállítja az Átlagos binninget, és
> nem kínál üzemmód-váltást. Az összegző binning minden DN-t N²-vel szoroz meg, ami nagyjából két f-stop
> fényerő-tartalékot vesz igénybe binning-lépésenként; az Chloros ezt a tényezőt visszaszámítja a
> radiometrikus lánc előtt, de az összegző binning által az érzékelő felső határáig hajtott pixelek levágódnak, és nem
> állíthatók vissza. Ezzel nem nyerünk semmit, amit az „Átlag” ne nyújtana – az index skála-invariáns, és a sugárzási
> skalárokat az „Átlag” skálára kalibrálják. Ha egy kamera már „Összegzés” beállítással érkezik (amit
> egy másik eszköz hagyott ott, vagy gyári visszaállítás eredményeként), a Chloros egy csomópont visszaolvasásából észleli ezt, és
> kompenzálja; az `lattice resolution --binning N` futtatása visszaállítja az Average beállításra. Az 1.2.0
> `--binning-mode` jelzőt továbbra is elfogadja, így a régebbi szkriptek is futnak, de csak figyelmeztetést jelenít meg:
> a binning „Átlag” marad.

> **A binning, a képpontformátum és a kioldás módosítása a kamerán nem marad meg.** Ha a kamerán a
> háttérprogram nyitva marad és adatfolyamot továbbít (`cam-connect`, a grafikus felület), az `lattice resolution --binning`
> nem lép hatályba (azt jelzi, hogy a binning továbbra is érvényben van, `Binning: 1x1`), és
> az `lattice format --set` és az `lattice trigger --mode` parancsokat a kamera elutasítja
> (`SC_ERR_ERROR -1001`). Először engedje fel (`lattice cam-disconnect -s <SN>`), vagy állítsa be
> őket a csatlakozáskor (`cam-connect --pixel-format …`, `array-connect --binning …`).

##### Az expozíció és az erősítés leolvasása automatikus módban

Amíg a kamera `ExposureAuto` (`GainAuto`) csomópontja fut, az `ExposureTime` (`Gain`) csomópontja a **legutóbb írt értéket** tárolja, nem pedig azt, amit a kamera éppen csinál. Egy olyan kamera esetében, amelyet a háttérprogram nyitva tart (`cam-connect`, `array-connect`, a grafikus felület), az `lattice configure --get`, az alapértelmezett `lattice configure` nézet és az `lattice exposure` (jelzők nélkül) ezért kinyomtatja a csomópont értékét, és alatta egy megjegyzést a kamera legújabb képkockájával:

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

Olvassa ki a képkocka értékét a megjegyzésből; a csomópont értéke csak az utolsó írási érték. HTTP-en keresztül ugyanaz az `POST /api/camera/<serial>/read-nodes` válasz változatlanul hagyja az `nodes` értéket, és hozzáadja az `frame_values` (`{"ExposureTime": <us>, "Gain": <dB>}`, a legújabb képkocka) és az `notes` (a fenti mondat) értékeket. Mindkettő csak akkor jelenik meg, ha az automatikus mód nem `Off`, és ismert egy képkocka; az olvasás céljából megnyitott kamera nem rendelkezik képkockával, és egyik sem jelenik meg. Az `lattice gain` (jelzők nélkül) csak a csomópontot nyomtatja ki — a megjegyzéshez használja az `lattice configure --get Gain`-et.

##### Expozíció egy tömb elemén

Amíg egy tömb adatfolyamot továbbít, minden eleme **`AcquisitionFrameRateEnable` parancsot**futtatja, a tömb trigger-frekvenciájánál kissé magasabb képkockasebességgel: periódusa a trigger-periódus mínusz 15 % (legalább 3 ms), vagy a kamera saját maximális sebessége, ha a trigger már közel van ahhoz. Az `configure --get` egy elemre ezért az `AcquisitionFrameRateEnable` értéket `True`-ként, az `AcquisitionFrameRate` értéket pedig ugyanilyen beállítás mellett olvassa be; az engedélyezés kikapcsol, amikor a tömb leáll.**Ne írja be ezt a két csomópontot egy tömb tagjára** — a tömb ezeket a triggerével szinkronban tartja, és újraírja őket.

Ez a beállítás egyben a tag expozíciós felső határa is: körülbelül 85 ms 10 fps-es trigger esetén (100 ms-os periódus), **bármilyen ROI vagy binning** mellett. Az 1.3.0-ás verzió előtt egy kis ROI a tag expozícióját a szenzor szabadfutási periódusára korlátozta (4,9 ms 768 × 304-es felbontásnál), függetlenül attól, hogy a mátrix milyen lassan váltott ki.**A felső határnál hosszabb, kézzel megadott `ExposureTime` érték lelassítja a kioldást, hogy az illeszkedjen hozzá**, ha azt `lattice exposure --set` vagy `configure --set` parancsokkal (vagy a grafikus felületről) írják be. A rendszer először megnyitja az adott elem felső határát az értékre, és hagyja, hogy a lassabb sebesség néhány kioldási periódusig futjon, így az írás néhány másodperccel tovább tarthat; a kioldás ezután nem gyorsabban történik, mint amennyit az expozíció megenged. Automatikus sebességű rendszeren a korábban futott sebesség visszatér, miután minden olyan elem, amelynek kézi expozíciója lelassította a rendszert, visszatért az automatikus expozícióra. A kamerával magával nem rögzíthető értéket továbbra is elutasítja, a hibaüzenetben feltüntetve az általa elfogadott tartományt, és a tag automatikus expozíciója visszaáll, ha azt kizárólag az adott írás miatt kapcsolták ki. Az `apply` expozíciós recept nem lassítja a kioldást: azt a felső határértéknél tartja. A Chloros soha nem rövidíti le a kézi expozíciót a gyorsabb kioldáshoz való igazodás érdekében; ha egy rögzített sebesség (`array-connect --target-fps F`) gyorsabb, mint amit az expozíció megenged, akkor az adott tag számára elegendő ideig fennmarad a maximális érték, és az a tag nem vesz fel gyorsabb kioldásokat, mint amit az expozíciója megenged.

#### Kalibrálás és érzékelők

| Alparancs | Cél |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibrálás egy reflektancia-célpont alapján. Legfeljebb `--attempts` képkockát (alapértelmezés: `30`) keresnek hozzá; egy hiányos keret (GVSP-csomagvesztés) egy kísérletet vesz igénybe, nem a futást. Hibaüzenetet ad `no target detected in N attempts` esetén, vagy `no complete frame in N attempts: <last error>` értéknél, ha egyetlen keret sem érkezett meg teljesen. |
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | A DAQ-M spektruma és sávmagnitúdói, amelyeket a háttérrendszeren keresztül Bluetooth-on (BLE) olvasnak ki. Az `--mac` az érzékelő saját BLE MAC-címe, és a három művelet bármelyikéhez szükséges: nincs alapértelmezett érték, mivel minden egységnek megvan a sajátja (az `daq pool-discover --only ble` felsorolja őket); ennek hiányában a parancs kilép (`1`). Művelet hiányában a súgót jeleníti meg. Az `--camera SERIAL` a kinyomtatott sávértékeket a szűrőmodell egyenes görbéje helyett az adott kamera saját, mért spektrális válaszával integrálja — RGB-kamerák esetében, amennyiben a kamera kalibrációs csomagja betöltődött (csatlakoztatva van) vagy a számítógépen gyorsítótárban van. A fejléc jelzi, kinek a görbéjét használták: `Band magnitudes (camera <SERIAL>):`, vagy `Band magnitudes (model <filter>):`, ha a kamerának nincs sajátja (nem RGB-kamera, vagy nincs betöltve vagy gyorsítótárban tárolva a csomag), és helyette a fleet görbét használták. HTTP-en keresztül az `POST /api/dls/cli-read` fogadja az `camera_serial`-et, és visszaadja az `band_magnitudes_source`-et. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Vignettakorrekció alkalmazása a meglévő képekre, mindegyikhez a saját objektívjének síkmezőjével (az EXIF Model mezőből). Az `--lens-model` olyan képekre alkalmazható, amelyek Model mezője nem nevez meg kamerát; az olyan képet, amelynek Model mezője ezzel ellentmond, a program kihagyja. |

#### Többkamerás (átmeneti munkamenetek)

Egy átmeneti parancs egy futtatásra megnyitja a kamerákat, és a futtatás végén bezárja őket: az `multi-capture` állandó tömb csatlakoztatása nélkül, az `multi-stream` és az `multi-test` itt, valamint az alábbiakban az `align-calibrate`, `align-apply`, `align-stream` és `index --live`. Ezek a szabályok mindegyikre érvényesek:

- **Az `--serials SN1,SN2,…` választja ki a kamerákat.** Csak azok nyílnak meg, és az első indítja el a kioldást (az `align-calibrate` és `--reference` esetében, ha megadva vannak). Ha egy sorozatszám nem szerepel a hálózaton, a futtatás meghiúsul, mielőtt bármelyik kamera kinyílna. Az `--serials` hiányában a hálózaton lévő összes kamera kinyílik. Egy olyan kamera, amelyet a Chloros már tart (a grafikus felületen, az `cam-connect`, `array-connect` parancsokkal), elutasítja a futtatást, függetlenül attól, hogy az `--serials` megnevezi-e vagy sem: mert ugyanazt a hálózatot használja, amelyre a futtatás be van állítva. (Az `index --live` az `--serials` nélkül ehelyett a profiljában engedélyezett kamerákat nyitja meg: lásd [Index / Vegetációs matematika](#index--vegetation-maths).)
- **A kamerák úgy osztoznak a hálózaton, ahogyan az `array-connect` osztja meg azt.** Minden kamera a gazda-kapcsolat által továbbított adatmennyiségből rá eső részhez igazodik (ez egy csomagok közötti késleltetés, nem pedig a kamera átviteli sebességének korlátja, amelynek 31,25 MB/s-os alsó határát négy kamera 1 GbE-n máris túllépi), így négy kamera fut egy 1 Gbps-es kapcsolaton. Amikor a kamerák nem tudnak egyszerre küldeni a leglassabb általuk elérhető sebességgel, egymás után aktiválódnak, mindegyik a saját kapcsolatával, és a parancs kiírja: `Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together`. Ha egyszerre akár csak egy kamera sem fér el, a futtatás elutasításra kerül, mielőtt bármelyik kamera streamelni kezdene: `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`, HTTP 409, kilépés `1`.
- **Minden kamerának át kell adnia az adatokat, különben a futtatás sikertelen.** Ha egy kamera nem ad át képkockát, az hiba: a parancs kiírja az `Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …` üzenetet, ahol az `<why>` az adott kamera letöltési hibája, majd a hiba okát jelzi, és kilép az `1` üzenettel. Ha egy képkocka GVSP-csomagvesztéssel érkezik (`IncompleteBufferError: GVSP packet loss …`), az azt jelenti, hogy a hálózat nem bírja el a kamerák összesített sávszélességét (`Every missing frame arrived with GVSP packet loss: …`): ellenőrizze a gazdagép és a kapcsoló felkapcsolási sebességét (`lattice network`), használjon jumbo kereteket, vagy kevesebb kamerát nyisson meg az `--serials` paranccsal. Az a kamera, amely semmit sem továbbított, általában nem kapja meg a master indítójelét: ellenőrizze az M8 szinkronkábelét, majd futtassa az `lattice multi-detect` parancsot. Az egyes parancsok működése egy kamerát hiányzó keretcsoport esetén a megfelelő sorban található.
- **Minden kamerát úgy állít vissza, ahogyan megtalálta.** A futtatás befejeztével minden kamera visszakapja azt az állapotot, amelyben a futtatás megnyitásakor volt: `TriggerMode`, `TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, valamint az `LineMode` / `LineSource` az `Line2`-en és az `Line3`-en (és a trigger-bemeneti zavarmentesítés egy tömb adja a szolga vonalát, `LineFilterSelector` / `LineFilterWidth`); az `PtpEnable` (és az azzal beállított `PtpSlaveOnly` / `PtpDomainNumber`); `ExposureAuto` / `GainAuto` (valamint az `ExposureTime` / `Gain`, ahol kézi értékre volt beállítva); `TargetBrightness`, `ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`, `GainAutoUpperLimit`, `ExposureAutoDamping`; `BalanceWhiteAuto` (és minden olyan `BalanceRatio`, ahol a fehéregyensúly kézi volt); `BlackLevelAuto` / `BlackLevel`; `PixelFormat`, binning, `Width` / `Height` / `OffsetX` / `OffsetY`; `GammaEnable` (és az `Gamma`, ahol a gamma be volt kapcsolva), `LUTEnable`, `ColorTransformationEnable`; `ReverseX` / `ReverseY`; `UserSetSelector`; a csomagméret (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`); a ritmus, `DeviceLinkThroughputLimitMode` (és `DeviceLinkThroughputLimit`, ahol a mód `On` volt), a csomagok közötticsomók közötti késleltetés `GevSCPD` és a kezdeti eltolás `GevSCFTD`, amelyet egy tömb ad meg minden kamerához; a blokkadatok, amelyeket minden adatfolyam bekapcsol (`ChunkModeActive`, mindegyik `ChunkEnable` a saját `ChunkSelector` alatt); `AcquisitionFrameRateEnable` (és az `AcquisitionFrameRate`, ahol az engedélyezés be volt kapcsolva — ha ki van kapcsolva, az érték csak olvasható, és a rendszer a ritmus által megengedett legnagyobb értéket olvassa ki, így ez nem egy beállítás). Csak a különbségek kerülnek írásba: a kamera először leállt, a lépésszám és a képkockasebesség az expozíció előtt, amelyhez a képkocka-periódusukra van szükség (a kamera az `ExposureTime`-et egy képkocka-periódusra korlátozza). Az `--keep-settings` esetén a trigger és a szinkronvonal vezetékei visszakerülnek, a kamera szabadon fut, a többi adat pedig a futási beállításoknak megfelelően a kamera saját memóriájába kerül; az `--clear-settings` a gyári beállításokat menti oda.
- **Az expozíciót az első képkocka rögzítése előtti jelenet alapján állítják be.** A futtatás szinkronizált képkockacsoportokat vesz fel, majd elveti azokat, miközben az egyes kamerák expozícióját és erősítését ezekből a képkockákból állítja be. A cél a kamera célpontjának átlagos fényessége, úgy, hogy a képkocka legfeljebb 1%-a legyen túlexponált; egy erős fénycsúcsokkal rendelkező jelenet esetén a célpont elérése előtt leáll a folyamat. Ez általában 2–6 képkockacsoportot vesz igénybe, és legfeljebb 5 másodpercet.
  - Minden kamera a csatlakoztatott tömb által megadott célpontra irányul a szűrőjével: TargetBrightness **40**multispektrális kamerán (RGN, OCN, NGB) és**60** RGB-kamerán. Ez az 80-as expozíciós célérték fele (RGB esetén háromnegyede), így egy fényes reflektancia-panel esetében kisebb a csúszás valószínűsége. Az `--brightness N` (0-255) helyette minden kamera célértékét beállítja (az `multi-test`-nek nincs `--brightness`-e: az automatikus expozíciója ki van kapcsolva). A háttérprogram naplója megnevezi az egyes kamerák célértékeit: `[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`.
  - Elsőként az expozíció kerül beállításra, a kamera saját automatikus expozíciós határértékéig. Az erősítés csak a fennmaradó részt fedi le.
  - A meghatározott expozíció és erősítés a teljes futás során megmarad, így az `multi-stream` minden képkockája azonos expozícióval készül. Az a felvétel, amelynek a változó fényviszonyokat kell követnie, egy kapcsolódó tömbbe (`array-connect`) tartozik, amelynek automatikus expozíciója folyamatosan alkalmazkodik.
  - Az `--exposure` és az `--gain` pontosan a megadott értékeken maradnak. Ha mindkettő meg van adva, egyetlen képkocka sem kerül elvetésre.
  - A futtatás befejeztével minden kamera automatikus expozíciója újra bekapcsolódik.
  - A háttérprogram naplója jelenti az eredményt: `[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …` vagy `exposure NOT solved after …` az ok megjelölésével, valamint `(not settled)` minden olyan kameránál, ahol ez nem történt meg. Ebben az esetben a képkockák továbbra is rögzítésre kerülnek.
  - Az `CHLOROS_TRANSIENT_AE_SETTLE_S` megváltoztatja az 5 másodperces korlátot; az `0` kikapcsolja ezt (lásd [Környezeti változók](#environment-variables)).

**Amíg az Chloros bármelyik kamerát** (a grafikus felületet, az `cam-connect` parancsot vagy egy csatlakoztatott kameracsoportot) tartja, az `multi-stream`, `multi-test`, `multi-detect`, `align-apply`, `align-stream` és `index --live` nem hajlandók futni, mert a hálózaton lévő összes kamerát megnyitják, és egy már megnyitott egy foglalt kamera megnyitása összeomlaszthatja a háttérrendszert. (Az `index --live` megtagadja a futást, ha a megnyitni kívánt kamera foglalt: ez lehet az `--serials` vagy a profilhoz tartozó kamerák, illetve bármely olyan profilhoz tartozó kamera, amelyik nem nevez meg egyet sem.) A háttérrendszer HTTP 409-es hibakóddal válaszol az `CAMERA_HELD` parancsra, felsorolva az összes lefoglalt kamerát és a felszabadító parancsokat (`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`), majd a parancs kilép `1`. Ha egy tömb csatlakozik, az `array-capture --continuous` rögzíti azt az `multi-stream` helyére, az `array-capture` (alapértelmezés szerint igazítva) egy igazított halmazt ment az `align-apply` helyére; az `multi-capture` és az `align-calibrate` a kameráikat tartalmazó csatlakoztatott tömböt használja ahelyett, hogy megnyitná őket.

| Alparancs | Cél |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | Az összes szinkronizáló szerepet betöltő kamerát felsorolja; az `--serials` parancs használatával csak azokat, az első pedig az `MASTER` parancs használatával. |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | Minden kamerából egy szinkronizált képkocka. **Alapértelmezés szerint minden exporttípust**elment, ha egy állandó tömb csatlakozik, és a DAQ, amelyik beolvassa, ([A rögzítést beolvasó DAQ elmenti](#the-daq-reading-a-capture-saves)); az átmeneti, tömb nélküli tartalék**csak debayerezett** (a többihez először futtassa az `array-connect` parancsot). Az `--brightness` kizárólag a tartalék üzemmódra vonatkozik: a csatlakoztatott sorozat megtartja saját automatikus expozícióját. A tartalék üzemmód nem menti az `.daq` (az `--no-daq` / `--daq-tolerance-ms` ott nincs hatással), és az `multi-capture` nem veszi figyelembe az `--force-daq`-et. A tartalék megoldásban az a kamera, amelyik nem szolgáltat képkockát, meghiúsítja a rögzítést (`Error: multi-capture failed: no frame from …`, kilépés `1`), és az adott képkockacsoportból egyetlen fájl sem kerül mentésre. |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | Szinkronizált képkockák streamelése (átmeneti), `--fps` alapértelmezett `3`, amíg Ctrl+C vagy `--count` csoportok nem következnek. Az a képkockacsoport, amelyhez a kamera nem szállított semmit, nem kerül írásra, és három egymást követő csoport véget vet a streamnek. Ha bármelyik csoportot kihagyták, a futtatás az `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)` kinyomtatásával ér véget, valamint az `Error: the stream stopped: …` kinyomtatásával, amely megnevezi a kamerát, amikor az befejezte a streamet, majd kilép az `1`X parancsot. Megáll, ha a meghajtó megtelik, kinyomtatja az `Failed:` / `Disk full:` parancsokat, majd kilép az `1` parancsra; bármely olyan fájl, amelyet nem tudott írni, szintén kilép az `1`-ből. |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | GPIO szinkronizációs időzítés teszt (átmeneti), `--count` alapértelmezett `10` kiváltók. Kinyomtatja, hogyan van megosztva a hálózat (`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)`, valamint az `Serial emit:` sor, amikor a kamerák egymás után aktiválódnak, és az időbélyegek tartalmazzák az eltolódást), egy `Sync group: master SN:<serial> -> SN:<serial>, …` sor szinkronizációs csoportonként (`(no slaves: not synced)` egy szinkronkábel nélküli kamera esetében, amely önállóan aktiválódik), majd egy sor minden egyes kiváltáshoz: `[  0] SN:<serial>=<timestamp> \| …`, az `SN:<serial>=(no frame)` pedig olyan kamerára vonatkozik, amely az adott kiváltáskor nem szolgáltatott adatot. Ezután `Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):` és egy `SN:<serial>: mean <±x> us, jitter <y> us` sor kameránként, minden kamera a mesterhez viszonyítva egy órajel alatt: egy olyan trigger, amelynek időbélyegei keverik a kamerák eszközóráit és a PTP-t (PTP ki van kapcsolva vagy még záródik) kihagyásra kerül, ezeket az `Note:` számolja. Bármely kihagyott képkocka a teszt meghiúsulását jelenti (`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`, kilépés: `1`); három egymást követő hiányos trigger esetén a teszt idő előtt véget ér. A teszt akkor is sikertelen (kilépés `1`), ha kevesebb mint két trigger esetén minden kamera egy órajelre van állítva (`… the clocks cannot be compared`), vagy ha az órajelek soha nem egyeznek: a PTP-órák több mint 1 ms-mal (plusz az esetleges soros rétegbeli eltolódás) eltérnek a mestertől (`… PTP is not locked`), vagy az eszközórák eltérése a trigger-ek között ennél nagyobb mértékben változik. Elindítja a PTP grandmastert, ha az nem fut, bekapcsolja minden kamera PTP-jét (ahogyan az `array-connect` teszi, függetlenül a hálózat állapotától), és az első trigger előtt megvárja, amíg minden kamera PTP-órája rögzül a grandmasterhez és stabilizálódik: `PtpStatus` Szolga egy főóra alatt, attól 100 us-on belül és 3 másodpercig stabilan, és minden kamera így rögzülve, szünet nélkül, 120 másodpercig — körülbelül 2 percig a bekapcsolástól. Egy éppen rögzített óra akár ~200 us-os eltérést is mutathat körülbelül 2 percig azután, hogy „Slave” értéket olvasott, és a kamera saját `PtpOffsetFromMaster` értéke ezt nem jelzi; 120 másodperc alatt a kamerák ~10 us-os eltérésen belülre kerülnek a stabilizált órához képest. Egy kamera még mindig szinkronban van egy korábbi teszt óta, amely a háttérrendszer szemszögéből nézve bekapcsolva hagyta a PTP-t, azt a szinkronizálást veszi alapul, így egymást közvetlenül követő tesztek esetén nem vár újra; az a kamera, amelyet a teszt bekapcsol, vagy amely PTP-vel be van kapcsolva, de amelynek szinkronizálását ez a háttérrendszer még nem látta (máshol kapcsolták be, vagy a háttérrendszer újraindítása előtt), kivárja a 120 másodpercet, és mindkét, éppen szinkronizálódott kamera a saját idejét várja ki. Az a kamera, amelyik kilép a Slave állapotból, túllépi a 100 us-t, vagy a várakozás alatt egymást követő 3 leolvasáson (1,5 s) keresztül egy másik grandmastert követ, újraindítja a számlálását; egy vagy két ilyen leolvasást rögzítenek (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`), és a számlálás folytatódik; az olvashatatlan grandmaster-azonosítót ismeretlenként kezeli, nem pedig változásként. Ha minden kamera számlálása egyszerre (egy másodpercen belül) újraindul, az a grandmaster idejének eltolódását jelenti, nem a kamerákéét: a háttérrendszer naplója az `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …` bejegyzést tartalmazza, az `ptp_lock` az `grandmaster_steps`-ben sorolja fel, majd egy lejáró várakozás hibaüzenete az `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`-szel kezdődik. A határérték nem kerül meghosszabbításra emiatt. Az `--ptp-settle SECONDS` beállítja a futás lezárási idejét (0–600; az `0` esetén nincs lezárási várakozás). A várakozás a leállási idő plusz 90 másodpercre van korlátozva (alapértelmezés szerint 210 másodperc, legalább 90 másodperc), és a parancs 240 másodperc plusz a leállási idő (alapértelmezés szerint 360 másodpercig). Kinyomtatja az `PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)` üzenetet, és a háttérrendszer naplójában 5 másodpercenként megjelenik egy sor, amely tartalmazza az egyes kamerák állapotát és eltolását, valamint azt, hogy mióta vannak rögzítve és mennyi idő van még hátra: `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`. Minden kamera kikapcsolt PTP-vel indul el; ezt követően az órák körülbelül 10 másodperc alatt szinkronizálódnak. Utána minden kamerát visszaállítanak arra az állapotra, ahogyan a teszt megtalálta – a PTP-jét, az automatikus expozíciót és az erősítést, amelyeket a teszt a futás ideje alatt kikapcsolt állapotban tart, a többi beállítást pedig (hacsak nem `--keep-settings`) — így egy PTP-vel bekapcsolt állapotban hagyott kamera (`lattice configure -s <serial> --set PtpEnable=True`, vagy egy `--keep-settings` kóddal futtatott teszt) szinkronizált állapotban marad, és miután ez a háttérprogram látta, hogy a stabilizálási idő alatt szinkronizált volt, egy későbbi teszt már nem vár rá. Ha az órák a várakozás alatt nem stabilizálódnak, a teszt a kiváltás előtt meghiúsul (`PTP: NOT locked …`, kilépés: `1`) és megadja, mit kell tenni: egy olyan kamera, amely soha nem záródott be (`PtpStatus Listening`), azt jelenti, hogy a grandmaster nem éri el (`time-sync status`); az, amelyik folyamatosan kiesett, a bezáródás időtartamával van megnevezve (`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`); az órák még mindig konvergálnak: ellenőrizze az `time-sync status`-et, és hogy a teszt előtt rögzüljenek, kapcsolja be a PTP-t, és hagyja bekapcsolva (`lattice configure -s <serial> --set PtpEnable=True` minden kamera esetében, vagy futtassa a tesztet az `--keep-settings` paranccsal; a teszt visszaállítja az eredeti állapotot, így bekapcsolva marad), majd futtassa le. |
| `lattice multi-detect [--line LINE] [--json]` | Keresse meg az M8 szinkronvonalon lévő kamerákat és az alapértelmezett mestert: az RGB-kamerát (a modell neve `-FRGB`-re végződik), ha az bekötve van, egyébként a soros csatlakozás szerint az első bekötött kamerát. Az M8-as kábelek egy csatlakozóponton futnak össze, így **bármelyik bekötött kamera lehet master** — a rányomtatott `MASTER` csak alapértelmezett érték, nem a bekötés tényét jelzi. Minden kamera sorban vezérli a vonalat, és a többi kamera impulzusát ellenőrzi; a bekötött csoportban már megtalált kamerát nem vizsgálja újra, így egy teljesen bekötött rendszer egy kör alatt átvizsgálható. Ha nincs `--line`, akkor az `Line2`-et vizsgálja, majd az `Line3`-et. Kinyomtatja az `MASTER: SN:…`-et, egy-egy `SLAVE: SN:…`-et minden további bekötött kamera után, valamint az `(probed on <line>)`-et; az `--json` hozzáadja a teljes eredményt (`master`, `slaves`, `no_cable` — azok a kamerák, amelyeket az impulzus nem ért el — `line`, valamint kameránként egy-egy `results`). A folyamat befejezése után minden kamera visszakapja a korábbi trigger- és szinkronvonal-beállításait (ugyanazok a csomópontok, mint fent). |

#### Igazítás

| Alparancs | Cél |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — plusz detektor/illesztő gombok `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, RANSAC paraméterek `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, többképkombináció `[--averaging mean\|median\|inlier_weighted]`, geometriai korlátok `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, térbeli korlátozás `[--roi X0,Y0,X1,Y1] [--mask PATH]`, valamint az egyes-szolga felülírások `[--per-cam-override SN:KEY=VALUE]` (ismételhető) | Igazítási profil kiszámítása élő kamerákból. Alapértelmezett értékek: `--method orb`, `--model rigid`, `--frames 3`, `--save alignment_profile.json` (abban a mappában, ahonnan futtatja), `--max-features 2000`, `--ratio-threshold 0.75`, `--matcher bf`, `--knn-k 2`, `--ransac-threshold-px 3.0`, `--ransac-iters 2000`, `--ransac-confidence 0.99`, `--averaging mean`. (Az `project align calibrate`-nek saját alapértelmezett beállításai vannak; lásd [a hozzá tartozó táblázatot](#project-align-calibrate-options).) Az `--prefilter` alapértelmezett értéke az `gradient` (éltérkép; megegyezik a GUI/tömb-igazítóval — az élek a spektrális sávokon át megmaradnak). Az `--matcher flann` ~5000 jellemző felett térül meg; az `--averaging median` robusztus egy hibás felvétel esetén, az `inlier_weighted` az egyezések száma alapján súlyoz; az `--lock-scale` a legközelebbi forgatásra vetíti (méretarány nélkül), `--lock-axis` egy transzlációs komponenst nullára állít; az `--mask` minden kamerára vonatkozik (kameránkénti beállításokhoz használja az `--per-cam-override`-et, pl. az `--per-cam-override 214701292:method=phase`-et); `--rms-threshold-px` elutasítja olyan kalibráció mentését, amelynek újraprojekciós RMS-értéke meghaladja a küszöbértéket. `--method manual` a pontjait az `--points PATH`-ből veszi: egy JSON-objektumból, az `{"<serial>": [[x, y], ...], ...}`-ből, amely megadja az egyes kamerák pixelkoordinátáit ugyanazon jellemzőkhez, beleértve a referenciakamerát is (kameránként legalább 2); az `--frames` mindegyikét ezekből a pontokból számítja ki. Ha a kamerákat egy összekapcsolt tömb tárolja, a kalibrálás a tömb élő képkockáit használja, és a profil egyben a tömb felvételeinek igazításává is válik. Ellenkező esetben megnyit egy [ideiglenes tömböt](#multi-camera-transient-sessions): ha bármelyik `--frames`-ben szereplő kamera nem szolgáltat képkockát, a kalibrálás sikertelen lesz (kilép az `1`-ből, megnevezve azt), és profil nem kerül mentésre. |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | Egy igazított többsávos képkocka rögzítése. Az `--bit-depth` alapértelmezés szerint a kamerához igazodik; `--no-crop` megtartja a teljes képkockát (feketével kitöltve); `--interpolation` (alapértelmezett: `linear`) és `--border-mode`/`--border-value` (alapértelmezett: `constant`/0) vezérli a CPU-torzítást — a GPU-útvonal minden esetben bilineáris. Megnyit egy [átmeneti tömböt](#multi-camera-transient-sessions): ha egy kamera nem szolgáltat képkockát, a rögzítés sikertelen lesz (kilép az `1`-ből, megnevezve azt), és semmi sem kerül írásra. |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | Stream-igazított többsávos képkockák (ugyanazok a warp-szabályozók, mint az `align-apply`-nél), `--fps` alapértelmezett `3`. Megnyit egy [átmeneti tömböt](#multi-camera-transient-sessions): egy olyan képkockacsoport, amelyhez a kamera nem szállított semmit, nem kerül mentésre, és három egymást követő ilyen eset véget vet a streamnek. Ha bármelyik csoportot kihagyták, a futtatás az `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)` és az `Error: …` kinyomtatásával ér véget, majd kilép az `1` parancs kiírásával. Bármely más hiba miatt véget érő adatfolyam szintén kiírja az `Error: …` parancsot, majd kilép az `1` parancs kiírásával. |
| `lattice align-info --profile PATH [--json]` | Profiladatok megjelenítése. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Rétegek sorrendjének módosítása. |

#### Index / Növényzet-számítások

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**Az `--channel` egy előre beállított szimbólumokat rendeli a sávokhoz** (NDVI: `red`, `nir`), egy-egy `--channel SYMBOL=BAND`.

- **`--live`: csak olyan szimbólumra vonatkozik, amelyre a sorozat nem tud választ adni.** Az `--input`-hez hasonlóan az előre beállított szimbólum, amelyet kihagy, a halom azon egyetlen sávját veszi át (a profil engedélyezett kameráinak sávjai közül), amelynek neve azzal kezdődik. Ezt minden kamera megnyitása előtt ellenőrizzük: ha van olyan szimbólum, amelyre egyetlen sáv sem, vagy több sáv válaszol, azt elutasítjuk, megnevezve azokat, majd kilépünk az `1`-ből. Egy egyéni `--formula`-hez minden szimbólumhoz szükség van egy `--channel`-re.
- **Az `--live` termékek megadják, kinek a termékei.** Az `index.tif` és az `index_LUT.png` tartalmazza a köteg kamerájának adatait (gyártó, modell, sorozatszám), a felvétel időpontját (a referencia-kamera expozíciós ideje, UTC) és objektívét, és mindkettő az indexet nevezi meg egyetlen sávjaként (`XMP-Camera:BandName` `NDVI`; a színes PNG-fájl az indexet nevezi meg, nem a színcsatornáit), ahogyan az `--input` termék is teszi. A több kamera sávjából származó index viszont mindegyiket megnevezi, és az `Model` neve `LATTICE` — lásd [Több kamerából készült fájlok](#files-made-from-several-cameras).
- **`--input`: csak olyan szimbólumra vonatkozik, amelyre a fájl nem tud válaszolni.** A kihagyott szimbólum a fájl azon sávját veszi át, amelynek neve azzal kezdődik: az `red` az `Red_660`-et vagy az `Red_660_RGN`-et veszi fel, az `nir` pedig az `NIR_850`-et. A nevek magából a fájlból származnak: a stack saját sávlistájából, vagy egy Chloros sugárzási/reflektancia-felvétel vagy az `process` termék esetében az `Chloros:Band{R,G,B}Name` / `XMP-Camera:BandName` címkékből. Tehát az `lattice index --input <reflectance capture> --preset NDVI`-hez nincs szükség `--channel`-re.
- **Két, egy szimbólumhoz tartozó sáv között nem történik kitalálás** (egy 9 kamerás stacknek két NIR-sávja van): nevezd meg azt. Egy olyan szimbólumot, amelyhez semmi sem tartozik, elutasítjuk az `channel_map missing entries` kóddal, majd kilépünk az `1` kódra.**A sorozatban szereplő sávnevek** a kamera szűrőjét hordozzák: egy M3C kamera sávjai az `<band>_<filter>` — `NIR_850_RGN`, `Green_550_RGN`, `Red_660_RGN` egy RGN-en; `NIR_808_OCN`, `Cyan_490_OCN`, `Orange_615_OCN` egy OCN-en; `Blue_475_RGB`, `Green_550_RGB`, `Red_625_RGB` egy RGB-en; `Blue_475_NGB`, `Green_550_NGB`, `NIR_850_NGB` egy NGB-en — és egy M3M kameraegysávja az `Mono_<filter>` (`Mono_660`, `Mono_850`). Az `lattice align-info --profile PATH` egy profil sávjait sorolja fel.

Teljes zászlóhalmaz: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (ismételhető), `--capture-level raw|debayered|radiance|reflectance|unknown` (felülírja a forrás TIFF-ben rögzített rögzítési szintet; alapértelmezés: TIFF-metadatokból olvasva, visszaesve egy Chloros-fájl `Chloros:ProcessingLevel` értékére, így egy reflektancia-rögzítés `reflectance` értéket jelent), `--output PATH` / `-o` (offline: egy elérési út; képkiterjesztéssel a fájlok neve a képkiterjesztés nélküli névből származik — az `-o ndvi_FRGN.png` az `ndvi_FRGN.tif` és az `ndvi_FRGN_LUT.png` fájlokat írja — kiterjesztés nélkül pedig előtagként szolgál, például `-o ndvi` -> `ndvi.tif`, `ndvi_LUT.png`; alapértelmezés szerint a bemenet mellett, mint például az `<input name>_<preset>`; az `--live` mellett, de az `-o` nélkül, egy `output/` mappa a parancs futtatásának helye alatt), `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Az `--live` fájl esetében az igazítási torzító gombok is érvényesek: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`, `--brightness N` (a kamerák automatikus expozíciós célértéke, lásd az átmeneti szabályokat), `--serials SN1,SN2,…`. Az `--live` megnyit egy [átmeneti tömböt](#multi-camera-transient-sessions): ha egy kamera nem szolgáltat képkockát, a futtatás sikertelen lesz (kilép az `1`-ből, megnevezve azt), és sem a verem, sem az index nem kerül írásra. Megnyitja az `--serials`-et (az első indítja el a triggert), egyébként a profil azon kameráit, amelyek `layer_order` bejegyzése engedélyezve van, először a referenciát — a referenciát akkor is, ha sávjai letiltva vannak, mivel a többi ahhoz van igazítva , és ez indítja el a triggert. Egy olyan kamera, amelynek sávjai le vannak tiltva, soha nem kerül kalibrálásra, így egy, a profilban letiltott RGB-kamera már nem bukik meg a reflexiós indexnél (`no-absolute-radiance-scale`). Egy olyan profil, amely nem nevez meg kamerákat, a hálózaton lévő összes kamerát megnyitja, mint korábban.

Minden űrlap a háttérprogramban fut, ezért annak futnia kell: indítsa el a Chloros GUI-ból, vagy azzal a paranccsal, amelyet a CLI kiad, ha nem éri el a háttérprogramot. Offline állapotban a háttérprogram beolvassa az `--input` fájlt, és maga írja a lebegőpontos TIFF-et és a színezett PNG-et, így egy relatív `--input` vagy `-o` fájl a parancs futtatásának helye szerinti mappához van rendelve (lásd [Konvenciók](#conventions)). Mindkét offline termék tartalmazza a bemeneti fájl kamerájának adatait, ahogyan az `process` indexexport is: gyártmány / modell / sorozatszám, GPS, az objektív címkéi, a felvétel ideje (a bemeneti fájl saját ideje; ha az EXIF-adatok nem tartalmaznak ilyet, akkor a rögzített expozíciós pillanat vagy a fájlnév időbélyege, UTC-ben), valamint a MAPIR-kalibrációs azonosítót — soha nem a nyers-DN-kalibrációt, amely nem vonatkozik az indexre —, továbbá az `Chloros:ProcessingLevel=index`-et és az indexkifejezést; a lebegőpontos TIFF-név egy sávot jelöl, az előre beállított értéket. Az `--list-presets` és az `--list-gradients` kinyomtatja a háttérprogram előre beállított és gradiens tábláit. Ha a háttérprogram elutasít egy képletet, vagy a TIFF-ben nincs meg egy `--channel` sáv, akkor az `1` a háttérprogram indoklásával lép ki.

> **Az `--channel` szimbólumoknál a kis- és nagybetűk megkülönböztetése érvényes.** A szimbólum oldalának pontosan meg kell egyeznie az előre beállított érték csatornevével (az előre beállított értékek kisbetűket használnak, pl. NDVI = `red`,`nir` — ellenőrizze az `--list-presets`-et), a sávoldalnak pedig meg kell egyeznie az igazított halmaz egyik sávnevével (vagy offline módban 0-tól kezdődő sávindexnek kell lennie). Az `--channel red=Red_660_RGN --channel nir=NIR_850_RGN` működik; az `--channel RED=660` hibaüzenettel bukik, `channel_map missing entries` hibával.

#### `lattice index` — többsávos halmazmaszkok

Az `--capture-level radiance` vagy az `reflectance` parancsoknál az egyes kamerák 3×3-as szétválasztása egy sávot 0-ra korlátozhat értékre korlátozhatja (lásd [Unmix korlátozó maszk](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)). Egy többsávos halmaz több kamera sávjait tartalmazza, és minden kamera a sajátját korlátozza, így az `--live --save-multiband` **forráskameránként egy maszkot** ír a halmaz mellé:

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

- Minden maszk egycsatornás `uint8` a **halmaz** képpontrácsán (ugyanazon igazítási torzítás és kivágás révén), az adott kamera saját bitjeivel: 0. bit = az R csatorna sávja, 1. bit = G, 2. bit = B. A kamerák *c* csatornájából vett halmazsávot ott korlátozzák, ahol az adott kamera *c* bitje (R = 0, G = 1, B = 2) be van állítva.
- Minden sáv beágyazott eredetadata megnevezi a kamera maszkját (`unmix_clamp_mask`, a stack mappájához viszonyítva, vagy `none`) és az olvasandó bitet (`unmix_clamp_mask_bit`).
- Az a kamera, amelynek unmix művelete semmit sem korlátozott, nem kap fájlt; a monokróm vagy RGB-kamera nem futtat unmix műveletet, és szintén nem kap fájlt. A CLI a stack alatt kinyomtatja az egyes maszkokat (`clamp mask SN…: …`); a válasz `clamp_masks` néven sorolja fel őket.
- Ugyanazon futtatás indexTIFF-je / PNG-je ugyanabból a veremből, ugyanazon a rácson kerül kiszámításra, így a maszkok ezt is leírják. Az `--capture-level raw` nem futtat unmix műveletet, és nem ír maszkot.
- A halmaz és az index közvetlenül az `--output` mappában található, így a maszkok az `--output` saját `clamp_masks/` mappájába kerülnek — soha nem az Ön által megnevezett mappa fölé. Ez a mappa különböző típusú fájlokat tartalmaz (stack, index, színezett render): válaszd ki belőle a szükséges fájlokat, ne add át az egész mappát a fotogrammetriai szoftvernek.

#### Állandó kapcsolatok (Smart-Prep, GUI-hez hasonló folyamat)

Ezek a parancsok a háttérpoolban nyitva tartják a kamerákat a különböző CLI-hívások között.

| Alparancs | Cél |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Egy kamera hozzáadása a poolhoz (egykamerás, nem tömb). A [közös opciók](#common-options-shared-by-most-subcommands) `--exposure`, `--gain`, `--preset`, `--pixel-format`, `--width` / `--height` és `--packet-size` (`standard` = 1500, `jumbo` = 9000, vagy bájtok; `auto` szondák) csatlakozáskor kerülnek alkalmazásra (egy `--exposure` vagy `--gain` kapcsoló kikapcsolja az automatikus módot). Visszatér, miután a kamera adatfolyama elindult (erre körülbelül 15 másodpercet vár, majd 10 másodpercet az első teljes pufferre), és kinyomtatja az `stream: streaming` vagy az `stream: no_frames -- connected, no frames yet: …` értéket, ha az adatfolyam nem szállított semmit — jumbo-csomagokat használó kamera esetén, azzal a tanáccsal, hogy az `--packet-size standard` paranccsal próbálkozzon újra. Amíg az egyik kamera csatlakozik (az első csatlakozáskor letölti a kalibrációs adatait, körülbelül 70 másodpercig tart), egy másik, kamerát igénylő parancs legfeljebb 5 másodpercig vár rá, majd HTTP 503 hibával bukik meg (`CAMERA_POOL_BUSY`), megnevezve a csatlakozó kamerát; egy pillanat múlva futtassa újra. Az `lattice status` és a kameralista soha nem vár: egy csatlakozás közben lévő kamera csatlakozóként jelenik meg, a fázisával együtt. |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | Felszabadítás. A kamera visszaáll a csatlakozáskor talált állapotába (kioldó vezetékek, PTP, beállítások); az `--keep-settings` a beállításait a saját memóriájába menti, az `--clear-settings` pedig azt a gyári beállításokra állítja vissza (lásd [Kamera memória](#camera-memory----keep-settings----clear-settings)). Egy tömb tagjaként szereplő kamera felszabadítja az egész tömböt. |
| `lattice cam-list` | A kamerák felsorolása a poolban. |
| **`lattice array-connect`**|**Tartós, szinkronizált tömb csatlakoztatása (AJÁNLOTT belépési pont).** A teljes GUI smart-prep folyamatot futtatja. 9 kamera esetén körülbelül egy percig tart; a kamerák bekapcsolása után azonban több percet is igénybe vehet. Várakozás közben kinyomtatja az egyes lépéseket, amelyeket a háttérprogram elér, például `    <s> s: <step>` (például `    41 s: open SN:214000535 (4 of 9)`), a háttérrendszer naplójában pedig lépésenként egy időjelzés szerepel (`[array-connect <id>] <step>: <s> s …`), valamint egy utolsó sor a leglassabb lépésekkel. Legfeljebb 600 másodpercig vár. Ha addig nem jön létre a kapcsolat, a háttérrendszer visszavonja a műveletet: minden megnyitott kamerát visszaállít az eredeti állapotába, és bezárja, a parancs kinyomtatja az `Error:` sort az elért lépéssel, majd kilép (`1`), és egyetlen tömb sem csatlakozik (HTTP 504 `CLIENT_DEADLINE_PASSED`). A Ctrl+C ugyanúgy megszakítja a csatlakozást (`Interrupted: …`); ha a tömb már csatlakozott, a sor ezt jelzi, és megnevezi az `array-disconnect` parancsot, amely felszabadítja azt. Egy várakozás közben leállított parancs (`kill -9`), a kapcsolat visszavonásra kerül, ha a háttérrendszer 30 másodpercig nem kap tőle jelzést. Egy olyan tömb, amely aktív, de még nem rendelkezik szinkronizált keretcsoporttal, továbbra is csatlakoztatottként jelenik meg, adatfolyama pedig az `no_frames`. |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | Tömb felszabadítása. Minden kamera szabadon futó állapotban marad (`TriggerMode` `Off`), minden egyéb beállítás pedig a kapcsolat felépítésekor megállapított állapotban marad — a trigger és a szinkronvonal beállításai (a master`TriggerSource` visszaáll a korábbi állapotába, a szinkronvonal pedig visszaáll bemeneti állapotba), a PTP-jét, valamint a munkamenet által módosított kamera beállításokat — így egy tagon végrehajtott egykamerás parancs azonnal hat. Az `--keep-settings` minden kamerabeállításait a munkamenet befejezésekor a kamera saját memóriájába; az `--clear-settings` visszaállítja azt a gyári beállításokra. Lásd [Kamera memória](#camera-memory----keep-settings----clear-settings). |
| `lattice array-list` | Csatlakoztatott rendszerek listája. |
| `lattice array-status [--array-id ID]` | Élő képkockasebesség (fps), PTP, utolsó hiba. Az `ptp_enabled` csak akkor az `True`, ha minden tag PTP-t olvas; az `ptp_off:` megnevezi azt a tagot, amelyik nem. Az `state:` sor az `restarting (applying settings)` értéket jeleníti meg a beállítások újraindítása során, az `recovering (…)` értéket pedig akkor, amikor egy áramellátását elvesztett tagot újra megnyitnak (lásd az alábbi megjegyzést), az `stalled (…)` a rögzítési ciklus futása közben, de keretcsoport közzététele nélkül, az `stopped (…)` pedig akkor, ha a rögzítési ciklus feladta a próbálkozást (csatlakoztassa újra a tömböt). |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Egy szinkronizált rögzítés az élő tömbből — Egyszeri / Folyamatos / Intervallum / Leggyorsabb. **Alapértelmezés: `all`**(kameraenként egy fájl az adott exporttípus szerint),**igazítva** az alkalmazás rögzítéséhez (`--no-aligned` natív fájlok esetén). A kihagyott kamerák (pl. RGB, amelyet kizártak a sugárzás/visszaverődésből) az `Skipped: SN:<serial> (<reason>)` kóddal kerülnek jelentésre; a rögzítést leolvasó DAQ adatai együtt kerülnek mentésre, egy kizárólag nyers adatokat tartalmazó rögzítés is szerepel, és az `DAQ: <path>` kóddal kerül jelentésre ([A rögzítést leolvasó DAQ elmenti](#a-felvételt-olvasó-daq-mentése)). Lásd [Felvételi módok, rögzítők és offline újrafeldolgozás](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | Az élő kombinált index nézet rögzítése videóba/GIF-be (megfigyelési minőségű; nyitott GUI-nézet hiányában a háttérrendszer rendereli a tömb NDVI-jét, vagy `--expr`). |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | Nagy képkockasebességű nyers Bayer-sorozatfelvétel (elemzési minőségű; offline újrafeldolgozás). Az `array-record` és az `array-capture --continuous` parancsokhoz hasonlóan minden lekérdezéskor jelentkezik a háttérprogramnál: leállítás nélkül megszakítva (`kill -9`), a háttérrendszer az utolsó jelentkezés után 30 másodperccel leállítja (`stop_reason` `client_gone`; a sorozat manifesztje véglegesítve lesz, és megkezdődik a kombinált index létrehozása, ahogyan a leállításkor is történik), és a tömb továbbra is csatlakoztatva marad. |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | A mentett nyers sorozat újrafeldolgozása kalibrált videó(k)vá. |

> **Az a kamera, amelyik áramellátását vagy kapcsolatát elveszíti, magától visszatér.** Amikor egy tag nem válaszol (a kapcsoló áramellátásának kikapcsolása és újraindítása minden általa táplált kamerát újraindít), a rendszer megvárja, majd újra megnyitja, amikor az újra válaszol, és visszaadja neki az `array-connect` által beállított értékeket: PTP, a trigger és szinkronizációs vonal, hálózati ütemezés, képpontformátum és képkockaméret, valamint automatikus expozíció. Ezután új `array-connect` beállítás nélkül folytatja az adatátvitelt. Eközben az `array-status` az `state: recovering (...)` értéket jelzi, az `last_error` pedig megadja, melyik kamerára vár; ezt követően az `re-opened:` értéket írja ki. Az újraindított kamera PTP kikapcsolt állapotban tér vissza, így amíg újra nem nyitják meg, az `ptp_enabled` az `False`, az `ptp_off:` pedig megnevezi azt.

> **Egy visszatartott kamerát soha nem nyitnak meg másodszor.** A második megnyitás leállítja a visszatartó adatfolyamát, átírja a beállításait, és összeomlást okozhat a háttérrendszerben. Tehát egy olyan kamerán, amelyet a pool tart fenn (a GUI vagy az `cam-connect`), egy tömb elemén, vagy egy olyan kamerán, amelyet egy háttérfeladat tart fenn (egy futó `lattice continuous`, `multi-stream` vagy `align-stream`, saját vagy egy másik kliensé), az `lattice continuous`, `lattice capture`, `cam-connect`, `power --sleep`, `calibrate`, egy-shot `white-balance` és `presets --apply` elutasítják HTTP 409 kóddal az `CAMERA_HELD`-et, megnevezve a tulajdonost és a felszabadítási parancsot (`chloros-cli lattice cam-disconnect -s <SN>`, `chloros-cli lattice array-disconnect --array-id <ID>`, vagy egy feladat esetében annak azonosítóját és azt, aki elindította), majd kilép `1`. Ugyanez vonatkozik egy olyan kamerára is, amelyet éppen egy másik parancs használ (egy `multi-test` vagy `multi-capture`, amely megnyitja a kameráit, egy másik egyszeri művelet), az itt szereplő minden parancs esetében: a hibaüzenet azt jelzi, hogy várni kell, amíg az adott parancs befejeződik. Az `lattice power` – `--sleep` nélkül – a tartó saját kezelőjén keresztül olvassa be az adatokat, ha van ilyen (a pool, egy tömb, egy feladat nyitott kamerája), és elutasításra kerül, ha a kamerát egy másik parancs használja. Minden `CAMERA_HELD` válasz tartalmazza a kódot mind az `code`, mind az `error_code` mezőkben. Az összes kamerát megnyitó parancsokról lásd [Többkamerás (átmeneti munkamenetek)](#multi-camera-transient-sessions).

##### `array-connect` opciók

| Jelző | Alapértelmezett | Leírás |
| --- | --- | --- |
| `--serials SN1,SN2,…` | az összes LATTICE kamera automatikus felismerése (legalább 2 szükséges) | Az első soros eszköz a MASTER. Ha elhagyják, a felismerés a LATTICE (M3C / M3M) modellekre szűri a keresést, és mindet csatlakoztatja. |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO szinkronizáló vonal. |
| `--target-fps F` | automatikus | A MASTER trigger tűzgyakorisága. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | automatikus | A szintválasztó felülírása. |
| `--wire-ceiling-mbps MB_PER_S` | automatikusan felismert | **A gazdagép folyamatos vezetékes sávszélessége, MB/s-ban — ez az a szám, amelytől az egész tömb-allokáció függ.** Csökkentse, ha a tömb GVSP-sérült kereteket jelent: az automatikus érték a hálózati kártya által jelzett kapcsolat sebességéből származik, amely túlbecsüli az USB-adaptereket, a keskeny PCIe sávokat és a forgalmas megosztott hálózatokat. A projekt tömb-rögzítési blokkjában tárolódik, így egy újbóli megnyitás, CLI- vagy SDK-újracsatlakozás visszaállítja. Lásd [Tömb állapota](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | nincs (teljes érzékelő) | Hardveres binning (mindig Átlag). |
| `--width N` / `--height N` / `--pixel-format FMT` | teljes érzékelő; kameránkénti formátum | Minden kamerához kifejezett képkockaméret vagy pixelformátum, a teljes érzékelő és a kameránkénti formátumválasztás (Mono12 / BayerRG12) helyett. |
| `--no-recommend` | ki | A hálózati elemzési lépés kihagyása (ekkor minden kamera megtartja a jelenlegi méretét, ahelyett, hogy a teljes érzékelőre lenne beállítva). |
| `--no-ptp` | ki | PTP letiltása (ekkor a kamerák közötti időbélyegek **nem** összehasonlíthatók). |
| `--packed {auto,on,off}` | `auto` | 12 bites képkockák csomagolása és kicsomagolása ezen a számítógépen — több képkocka másodpercenként teljes szenzor mellett, azonos adatokkal. Az `auto` csak akkor csomagol, ha ez előnyös, és a számítógép kapacitása megengedi; az `off` soha nem csomagol. Lásd: [Csomagolt 12 bites átvitel](#packed-12-bit-transport---packed). |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | A tömbben lévő minden kamera GVSP-csomagmérete. `auto` Az  lehetővé teszi, hogy minden kamera felderítse a hálózatot; `jumbo`  = 9000, `standard`  = 1500, vagy egy bájtban megadott méret rögzíti azt. Az 1.3.0-ás verzió előtt az `array-connect` elfogadta ezt a jelzőt, de figyelmen kívül hagyta. |

> **Alapértelmezés szerint teljes érzékelő.** `--width` / `--height` / `--binning` /
> `--pixel-format`, az `array-connect` minden kamerát teljes szenzorra állít — nincs
> binning, nincs ROI-kivágás — és hagyja, hogy a képkockasebesség a számítógép és a
> hálózat által biztosított szintre csökkenjen, ahelyett, hogy a képkockákat kicsinyítené a sebesség fenntartása érdekében. A
> hálózati elemzés sorában a várható sebesség látható, a következő sorban pedig
> a gyorsabb lehetőségek (binning vagy kisebb ROI) szerepelnek a várható sebességükkel; a
> az `--binning` vagy az `--width`/`--height` parancsot az egyik kiválasztásához. A „connect” parancs önmagában soha nem végez binninget vagy kivágást
> .

##### Tömörített 12 bites átvitel (`--packed`)

Teljes szenzorhasználat mellett a tömb képkockasebességét az határozza meg, hogy az egyes kamerák milyen gyorsan tudják elküldeni
a képkockáikat. A 12 bites formátumok (`BayerRG12`, `Mono12`) általában 16 bites
szavakként kerülnek továbbításra; a **tömörített** átvitel ugyanazt a 12 bitet 1,5 bájtban küldi el pixelenként, és
ezen a számítógépen kicsomagolja őket, így minden képkocka továbbítása 25%-kal rövidebb időt vesz igénybe —
teljes szenzor mellett **másodpercenként körülbelül 33%-kal több képkocka** (mért érték: 10,7 → 14,0 fps
egy 9 kamerás rendszerben). **A mentett adatok megegyeznek**: ugyanazok az értékek, ugyanazok a
16 bites fájlok, ugyanazok a metaadatok.

- Az `auto` (alapértelmezett) csak akkor tömörít, ha ez előnyös **és** ez a számítógép képes
  a kicsomagolásra. A csatlakozáskor a háttérrendszer megméri saját kicsomagolási sebességét, a CPU-magok számát,
  az aktuális CPU-terhelést és a hálózati sávszélességet, így egy szabad magokkal rendelkező asztali számítógép **teljes szenzorhasználat mellett** tömörít,
  míg egy leterhelt vagy alacsony teljesítményű számítógép kicsomagolt formában továbbítja az adatokat. A kis ROI-k és a binningelt képkockák
  kevés előnyt.
- `off` soha nem tömörít — használja a tömörítés kikapcsolására diagnosztika közben, vagy olyan
  olyan számítógépen, amelynek az összes CPU-kapacitását más feladatokra kell fordítania.
- `on` mindig 12 bites formátumokat tömörít.

A connect kimenet megmutatja a döntést és annak indoklását (`tömörített 12 bites: (auto:
becsült 11,0 -> 14,4 fps, …)`); `/api/camera/array/status` jelentések
`packed_transport` és `packed_decision`. Ha a zászló minden alkalommal történő átadása helyett
gép-szintű alapértelmezést szeretne beállítani, állítsa be az `CHLOROS_ARRAY_PACKED12=off` (vagy
`on`) értéket a háttérprogram környezetében — egy kifejezett `--packed on|off` beállítás továbbra is elsőbbséget élvez.
A 8-bites és 16-bites formátumok soha nem kerülnek tömörítésre.

### Egy projektfájl, minden üzemmód

Egy projektmappa tartalmazza az összes eszköz beállításait: `cameras.json` — minden kamera beállításait (`cameras[].settings`, egy kamerára és a tömb minden tagjára vonatkozóan), valamint minden tömb tömbszintű beállításait (`arrays[].capture`: formátum, ROI, binning, triggerfrekvencia, vezeték-kapacitás) — és az `sensors.json`, minden fényérzékelőé (integrációs idő, képkockaátlag, felső határ). Az alkalmazás, a CLI, az SDK és az MCP-kiszolgáló mind ugyanazokat a fájlokat olvassa és írja, ugyanúgy:

- **Egy projekthez csatlakoztatott eszköz abba kerül mentésre, és minden változást abba rögzít.** A „projekthez csatlakoztatva” azt jelenti, hogy a háttérben (az alkalmazásban) nyitva van egy projekt, vagy — az SDK-ből — egy `ChlorosProject`-en keresztül, amely megnevezi a saját mappáját. Egy olyan kamera, sorozat vagy fényérzékelő, amelyet a projekt még nem tartalmaz, hozzáadódik a fájljaihoz, amikor csatlakozik (`lattice cam-connect`, `array-connect`, `daq pool-connect`, az alkalmazás, az SDK, `chloros-mcp`), a beállításaival együtt. Bármelyik üzemmódból (`lattice exposure`, `gain`, `configure --set`, `resolution`, egy sorozatbeállításai és kioldási gyakorisága, a fényérzékelő fedele, az alkalmazás vezérlőelemei) egy pillanat múlva el lesz mentve az adott projekt fájljaiba. A fényképezőgépen beállított expozíció, erősítés vagy AE-célérték rögzítve marad, így a projekt újbóli megnyitásakor azok ismét érvényesek lesznek.
- **A projekt megnyitásakor az eszközök beállításai a projektből kerülnek beállításra**, minden üzemmódban: a saját beállítások nélkül csatlakoztatott fényképezőgép, sorozat vagy fényérzékelő a projektben mentett beállításokat kapja meg (`lattice cam-connect` / `array-connect` egy megnyitott projekt esetén, `chloros-cli project connect`, `open_project().connect_all()`, az alkalmazás újbóli megnyitásakor). Egy új azonosítóval csatlakoztatott sorozat frissíti a projekt által ugyanazokkal a kamerákkal mentett sorozatot; egy más kamerákkal rendelkező sorozat pedig mellé kerül hozzáadásra.
- **Az alkalmazás saját mentése soha nem írja felül az eszköz aktuális beállításait** azok régebbi másolatával, és megőrzi az egyes elemekre vonatkozó felülírásokat, amelyeket egy CLI- vagy SDK-hívó mentett el.
- **A beállított képpontformátum az, amelyet a projekt megőriz.** A Tömbbeállításokban, az `--pixel-format` vagy az `connect_array(pixel_format=)` parancsokkal beállított formátum – beleértve a 8 biteset is – pontosan mentésre kerül és újracsatlakozáskor is pontosan alkalmazásra kerül. Ha nincs beállítva, minden kamera az alapértelmezett 12 bites beállítással csatlakozik (`BayerRG12`, `Mono12` monokróm érzékelő esetén). A formátum neve a bitmélységet jelenti; a család az érzékelőé, így egy színes kamerát soha nem állítanak be monó formátumra, egy monó kamerát pedig soha nem állítanak be Bayer-formátumra, és az `[PixelFormat <serial>] ... written` sor ezt jelzi, ha a formátumot leképezték. Egy 8-bites rögzítés exportált adatai 8-bitesek, beleértve a reflexiót is: átméretezve és az `PixelScale` azonosítóval ellátva.
- Egy projekthez nem csatlakoztatott eszköz nem rögzít semmit.
- **A felvétel az azt követően létrehozott projektet követi; a bezáráskor a projektjét megtartja.**Amikor egy projektet hoznak létre — egy SDK-t (`create_project()`, `chloros-mcp` vagy az alkalmazás**Új projekt**— vagy lemásolásakor (az alkalmazás**Másolás**funkciója, amely megnyitja a másolatot), miközben a felvétel még mindig a korábban megnyitott projektbe ír, a felvétel folytatódik, és az, amit ír, az új projektbe kerül, annak ugyanazon a helyén (alul). Meglévő projekt megnyitása — `chloros-cli process`, `project devices --reconnect`, `ChlorosProject.process()`, az alkalmazás**Projekt megnyitása** gombja — esetén a felvétel továbbra is fut abban a projektben, amelybe éppen rögzít: ott fejeződik be, és a fájljai bekerülnek az adott projekt fájllistájába (az alkalmazás minden felvételt leállít, mielőtt megnyitna egy projektet).
  - `array-record` és `array-burst`: a videó, illetve az összes képkockát tartalmazó sorozatmappa teljes egészében oda kerül, amikor a felvétel leáll (az `--duration`, Ctrl+C). Az `Burst dir` az új fájl, az `--build` és az `array-build-video` pedig abból indulnak ki. Ha a meghajtó nem engedi a fájl áthelyezését, a felvétel ott marad, ahová íródott; avégén található napló megmagyarázza az okát, ahogyan a leállítás HTTP-válasza is (`relocation_error`). A leállított sorozat nyers képkockái és az `.daq` felkerülnek annak a projektnek a fájllistájára, amelybe végül bekerül; egy videó (egy `array-record`, egy sorozatból összeállított videó) nem kerül fel a projekt fájllistájára.
  - `array-capture --continuous`: a módosítás utáni minden felvétel az új projektbe kerül; a módosítás előtt mentett felvételek a helyükön maradnak.
  - `daq pool-record` és `project sensor log` (valamint az alkalmazás saját fényérzékelős felvétele): az `.daq` az új projekt`light_sensor` mappájába kerül, amikor a felvétel leáll, és csatlakozik az adott projekt fájllistájához.

  A nyitott projekten kívüli mappába történő felvétel (egy máshol található `-o`) ott marad, ahol van, az `lattice continuous` és az `multi-stream` pedig megtartja a nekik kijelölt mappát.
- **Az alkalmazásban az új projekt létrehozása után készült rögzítések abba kerülnek.** Egy projekt létrehozása vagy másolása során minden rögzítési folyamat és felvétel továbbra is fut, és az azt követően készült minden rögzítés (Minden rögzítése, Rögzítés indítása) az új projektbe kerül, és csatlakozik annak fájllistájához. Egy meglévő projekt megnyitása az alkalmazásban először mindet leállítja.
- **A projekt felvétel közben nem mozgatható.**A nyitott projekt másik munkakönyvtárba való áthelyezése (az alkalmazás**Beállítások → Munkakönyvtár** menüpontja; CLI-parancs nem mozgatja a projektet, ezt csak az `/api/move-project`-re küldő HTTP-kliens teszi) HTTP 409 `PROJECT_RECORDING` hibával kerül elutasításra, amíg egy tömbfelvétel vagy sorozatfelvétel, egy tömbfolyamatos rögzítés, egy `lattice continuous` / `multi-stream` feladat vagy egy fényérzékelős felvétel ír bele: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.` Az alkalmazás ugyanúgy elutasítja, még mielőtt mappát kérne, miközben a saját „Capture All”, „Start Capturing” vagy fényérzékelős felvétele fut.

Ez független attól, hogy a kamerák milyen állapotban maradnak a kapcsolat megszakításakor (lásd [Kamera memória](#camera-memory----keep-settings----clear-settings)): egy projekt megnyitásakor maga állítja be a kameráit.

### Kamera memóriája — `--keep-settings` / `--clear-settings`

Ha egy tömb csatlakozik, a kamerái átveszik a tömb beállításait, amelyeket ezután módosíthat. Amikor a Chloros elengedi a kamerákat — `array-disconnect`, egy átmeneti parancs vége (`multi-capture` tömb nélkül, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`), az `lattice continuous` leállítása, egy olyan kamera `lattice capture` végének lezárása, amelyet nem tart kézben — mindegyiket visszaállítja arra az állapotra, ahogyan a csatlakozás előtt talált: a kioldó- és szinkronvezeték-kábelezését, a PTP- és a kamera beállításait (a teljes lista a [Többkamera (átmeneti munkamenetek)](#multi-camera-transient-sessions) alatt található).

A kamerák projektön kívüli használatához e parancsok bármelyikénél két jelzővel a kamera **saját memóriájába** írhatók azok a beállítások, amelyekkel a kamera bármely szoftverben bekapcsoláskor rendelkezik:

- **`--keep-settings`** elmenti az egyes kamerák beállításait úgy, ahogyan azok a munkamenet végén a memóriában voltak (UserSet2, amely a rendszerindítási beállításokká válik). A kamerák továbbra is szabadon futnak (`TriggerMode` `Off`), a szinkronvezeték-kábelezés pedig a talált állapotban marad. Az a kamera, amelynek nincs UserSet2-je, a UserSet1-be menti el a beállításokat; ezt a számítógép megjegyzi, így a Chloros-rendszer önjavító funkciója — ami visszaállítja a Chloros alapértékeitől eltérő UserSet1-et — nem avatkozik bele.
- **`--clear-settings`** visszaállítja az egyes kamerák memóriáját a gyári beállításokra: Chloros alapértékeire, amelyeket a rendszer önjavítása — gyári alapértékek az `TriggerMode`, `Off`, `TriggerSource` és `Line2` parancsokkal, automatikus expozíció, az RGB-kamera kalibrált fehéregyensúlya megmarad — a UserSet1-ben, amely a rendszerindítási készletet alkotja.
- A kettő nem kombinálható. Minden kamera kap egy sort: `SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` vagy `SN:…: camera memory reset to the factory settings.`; a válaszban szereplő `camera_memory` minden kamerára ugyanazt tartalmazza.

Az alkalmazás nem kínálja ezeket a jelzőket.

**Minden kilépési útvonal visszaállítja a kamerákat.**Ugyanaz a „talált állapotba való visszaállítás” fut le minden kilépési útvonalon, nem csak a fenti parancsok esetén: `cam-disconnect` (és az alkalmazás**X**kamerája); az alkalmazás**X** parancsa egy tömbön vagy bármely tömbtag során (egy tömb nem futhat tag nélkül, így az egész tömb felszabadul); az alkalmazás bezárása (ezzel a háttérrendszernek legfeljebb 20 másodpercet ad minden kamera, tömböt és a futó `lattice continuous` parancsot visszaállítsa, mielőtt leállítaná; egy ekkor még futó parancs — egy átmeneti parancs, amely megnyitja kameráit vagy várja azok PTP-óráit, egy egyszeri `lattice capture`, `power`, `white-balance` vagy `presets --apply` — a következő biztonsági ponton leáll, először a kameráit állítja vissza, legfeljebb 8 másodpercig, és HTTP 503 `SHUTTING_DOWN` választ ad; egy még futó connect-tömb — az alkalmazás connect-je, az `lattice array-connect` — a következő kamera előtt leáll, és legfeljebb 10 másodpercre visszaállítja a korábban megnyitott kamerákat, és HTTP 503-as hibakódot ad vissza: `SHUTTING_DOWN`; és egy olyan, amely a leállás közben érkezik, nem nyit meg semmit: HTTP 503 `SHUTTING_DOWN`); valamint egy CLI-parancs, amelyet megszakítanak, vagy amelynek terminálja bezárul (a SIGTERM / SIGHUP leállítja a feladatát — és visszaállítja a kameráit — mielőtt kilépne (`143` / `129`, hasonlóan a Ctrl+C-hez). Egy `lattice continuous`, `multi-stream` vagy `align-stream`, amelyet azonnal leállítanak (SIGKILL) – és amely semmit sem tud leállítani –, feladatát a háttérrendszer állítja le, és kameráit az utolsó bejelentkezés után 30 másodperccel után. Az összeomló vagy erőszakosan leállított háttérprogram nem tud semmit visszaállítani; a teljes kikapcsolás és újraindítás minden kamerát a saját memóriájába állít vissza, és a következő Chloros-kapcsolat újrakábelezi az általa élesítettnek talált kamerát.**A projekt maga állítja be a kameráit.** Amikor egy tömb csatlakozik egy megnyitott projekthez, minden tag megkapja a projekt által számára elmentett beállításokat – egy expozíciót, erősítést vagy AE-célt, amelyet a rendszer számára rögzít (a sajátodként kezelve, így az automatikus expozíció nem módosítja), valamint a fehéregyensúlyi módját (a Chloros korrekciója; a kamera saját fehéregyensúlya kikapcsolt marad) – függetlenül attól, hogy mi van a kamera memóriájában.

### Smart-AE / Smart-Capture

A LATTICE-rendszerek csatlakozás után azonnal folyamatos AE-t futtatnak a háttérben, de egy újonnan beállított jelenetnek egy pillanatig tart, amíg konvergál. Az `array-capture --smart` a **készre csomagolt kényelmi megoldás**: megvárja, amíg az AE minden kamerán beáll a rendszerben, majd elindítja a felvételt. Használd, ha a munkamenet közben váltasz jelenetet.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Az állandósodási szabály alapértelmezés szerint konzervatív: 5 másodperces időkorlát, 1,5 másodperces stabilitási ablak, ±5 %-os expozíciós szórás-tolerancia. Ha az automatizálástól eltérő viselkedésre van szükséged, az SDK-n (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) keresztül állítsd be.

#### Hogyan működik az automatikus expozíció?

- **Először az expozíció, utoljára az erősítés.** Ha az erősítés automatikus üzemmódban van, az erősítés 0 dB-en marad, amíg az expozíció el nem éri a megengedett leghosszabb értéket.
- **A kézi expozíció esetén az erősítés „Auto” állapotban marad**, amíg meg nem változtatja az erősítést, vagy kézi üzemmódra nem állítja. Visszatérve az „Auto” üzemmódba, az erősítés határa a korábbi érték marad.
- **A kis fényes területek nem sötétítik el a képet.** A képkocka legfeljebb 1%-át kitevő tükröződést a rendszer figyelmen kívül hagyja, ismert kalibrációs célpanelek esetében pedig a panel legfeljebb 0,5%-át.
- **A DAQ expozíciós felső határa csak akkor érvényes, ha kalibrációs célpont látható.** Ez megakadályozza a fehér panel túlexponálását; ha nincs látható panel, a kamera a jelenetnek megfelelően állítja be az expozíciót.
- **Az állandó fényviszonyok mellett az expozíció és annak határa változatlan marad, míg a fényviszonyok jelentős változása után a kép néhány másodpercen belül stabilizálódik.** Az állandó fényviszonyok mellett az expozíció és annak határa változatlan marad, míg a fényviszonyok jelentős változása után a kép néhány másodpercen belül stabilizálódik. A túlvilágítás már nem korlátozza véglegesen az expozíciót, és az érdeklődési terület (ROI) megváltozása esetén a rendszer újra megtalálja a megfelelő expozíciót.
- **A fényerő-célérték elérhető tartományban marad.** Nem emelkedik, amíg a kamera az expozíciós határértékén van, és az erősítése nem növelhető, így a visszatérő fény nem okoz túlexponálást a képkockán.
- **Az a kamera, amelynek automatikus expozíciója nem reagál, a jelenlegi fényviszonyokhoz igazodó szinten marad.** Erős napfényben a kamera a legrövidebb expozíciós időre állhat át, és ott is maradhat sötét képpel, miközben minden beállítása normális értéket jelez. Az Chloros figyeli a képet, a fényviszonyokhoz igazodó szinten tartja a kamerát, és elengedi, amikor a fényviszonyok megváltoznak. Egy sötét vagy eltakart fényérzékelő ezt nem zavarja.

### Felvételi exportszintek (az `all` alapértelmezés)

Ezen kiadás, az `lattice capture`, `lattice multi-capture` és `lattice array-capture` **alapértelmezés szerint az `--processing all`** — exporttípusonként egy mentett fájl, amely minden kamerára vonatkozik, összhangban a grafikus felhasználói felület „Minden rögzítése” funkciójával. A szintek a következők:

| Szint | Kimenet | Vonatkozik |
| --- | --- | --- |
| `raw` | Egycsatornás Bayer (mono kamerák: az egyetlen sáv) közvetlenül az érzékelőből. | Minden kamera. |
| `debayered` | 3-csatornás színes, **RGB**csatornasorrendben mentve (TIFF RGB fotometrikus értelmezéssel; a PNG és JPEG fájlok szintén RGB formátumban nyílnak meg): a kalibrációs csomag sötételtolása (DSNU) levonásra kerül a nyers adatból, majd az OpenCV éltudatos-érzékeny demosaic – nincs linearitási LUT, síkkép-korrekció vagy színszétválasztás (monokróm kamerák: 1-csatornás szürkeárnyalatos). A Python SDK által visszaadott, memóriában demosaicolt képkockák (`grab()`, `frame_stream()`)**BGR** sorrendűek, az OpenCV sorrendje szerint; csak a mentett fájl RGB. | Minden kamera. |
| `radiance` | float32 W/m²/sr/nm a teljes radiometrikus láncon keresztül. | Csak multispektrális (M3C/M3M) — **kihagyva az RGB-szűrős kamerák esetében**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), Pix4D-kompatibilis. | Csak multispektrális, és **csak akkor, ha DAQ van hozzárendelve + a kamera kalibrálva van**; ellenkező esetben kihagyva. |
| `display` | Teljes GUI-előnézeti lánc (CCM + WB + gamma a kamera profilja szerint). `preview/` mappába mentve. Az `lattice capture` az `preview` nevet is elfogadja; az `array-capture` / `multi-capture` csak az `display` nevet fogadja el. | Minden kamera. |

Adjon meg egyetlen szintet, hogy csak azt mentse el (`--processing debayered`). Ha az `all`-et kéri, az adott kamerára nem vonatkozó szinteket a rendszer kihagyja (és jelentésre kerülnek), de hibaüzenet nem jelenik meg — egy csatlakozatlan vagy kalibrálatlan kamera is megkapja az `raw` / `debayered` / `preview` értékeket.

#### A felvételt olvasó DAQ mentése

Amikor egy DAQ-t a kamerához vagy a kameracsoporthoz rendelnek, vagy poolba helyeznek, a felvétel olvasásához használt lefelé irányuló mérési adatokat egy **`.daq`** fájlba a felvételek mappa `daq/` almappájába, az `.match.json` (az érték és az expozíció közötti időeltolódás) és az `.cameras.json` (melyik felvételhez tartozik) értékekkel, és egy `DAQ:` sorban kerül jelentésre. Ez minden szintre érvényes, beleértve a nyers és a debayered felvételeket is, így bármelyik felvétel később újra feldolgozható sugárzási, visszaverődési vagy indexértékre. Ugyanaz a fájl, ugyanazzal a névvel és kiegészítő fájlokkal, függetlenül attól, hogy a felvétel a grafikus felhasználói felületről (GUI), a parancssorból (CLI) vagy az SDK-ből származik-e. Ha egyáltalán nincs DAQ-leolvasás, akkor nincs `.daq`, és egy `multi-capture`, amelyhez nincs kapcsolódó tömb (a tranziens tartalék) nem ment semmit.

| Jelző | Hatás |
| --- | --- |
| `--no-daq` | Ne mentse el az értéket olyan rögzítésnél, amelynek szintjei nem használják azt (`raw`, `debayered`). Az `radiance`, `reflectance` vagy `display` szint azonban továbbra is írja, mert az újbóli importáláshoz szükség van arra az értékre, amellyel a termék készült. |
| `--daq-tolerance-ms MS` | Milyen messze lehet az érték az expozíciótól, és még mindig a tűréshatáron belülnek számít (alapértelmezés: 500). Csak az értékelés változik (az `.match.json`, `in_tolerance` és `tolerance_ms` esetében, valamint a reflexió esetében az `DlsInTolerance`), soha nem a felhasznált mérési érték. Lásd [Expozíciótól távol eső DAQ-mérési érték](#a-daq-reading-far-from-the-exposure). |
| `--force-daq` | A régebbi módszer az `.daq` lekérdezésére kizárólag nyers adatokat tartalmazó rögzítés esetén. Jelenleg ez az alapértelmezett beállítás, és az `lattice capture` és az `array-capture` továbbra is elfogadja ezt a jelzőt (az `multi-capture` viszont nem). Az `--no-daq` felülírja ezt. |

Melyik DAQ: a kamerához vagy a sorozathoz kötött (a grafikus felület fényérzékelő-kiválasztása, egy projekt mentett kötése vagy az `chloros-cli daq pool-bind`), egyébként a háttérrendszer legfrissebbje. Egy nyers sorozat (`array-burst`) továbbra is minden egyes leolvasáshoz saját `.daq` fájlt tárol a burst mappában, mint korábban.

**A PNG és a JPEG formátumok csatornánként 8 bitet tárolnak, ezért csak az `display` / `preview` fájlokat és az indextermékeket veszik át.** Az olyan `--format png` vagy `jpg` fájlokat, amelyek szintje meghaladja a tárolási kapacitást, a rendszer még a rögzítés megkezdése előtt elutasítja, megnevezve a szintet és a TIFF formátumot, amely képes azt tárolni:

- `radiance` (float32) és `reflectance` (uint16, `32768` = 1,0) — mindig, így az `--processing all` (az alapértelmezett) az `--format png`/`jpg` értékekkel szintén elutasításra kerül: használja az `--format tiff` vagy az `--levels display` értéket.
- Az `raw` és az `debayered` értékek olyan kamerától, amely 8 bitnél többet továbbít (a 12 bites M3C / Mono12 M3M képkockák) — a CLI nem látja a kamera bitmélységét, ezért a háttérrendszer elutasítja ezeket (HTTP 400). Az egyetlen`lattice capture --processing debayered` 8 bites képkockát rögzít PNG/JPEG formátumban, és nem kerül elutasításra, ahogyan a tartós tömb nélküli `multi-capture` sem (8 bites debayered).

Minden rögzítési útvonal ugyanúgy válaszol (`error_code` `FORMAT_CANNOT_HOLD_LEVEL`): `lattice capture`, `multi-capture`, `array-capture`, `project capture` / `burst`, valamint a mögöttük lévő HTTP-útvonalak.

### Hogyan néz ki egy rögzítési mappa?

Minden exporttípus az `-o` mappa alatt egy **saját almappába**kerül, így egy többszintű rögzítés során a típusok soha nem keverednek. Az ábra az**`all` által egy kameráról készített alapértelmezett rögzítést mutatja az `lattice capture` alatt** — kameránként több szint van, így minden fájl `_<level>` kiterjesztéssel rendelkezik:

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

Hogyan változnak a fájlnevek:

- **Kamera mező — sorozatszám, majd szűrő:** minden rögzítési fájl a felvételt készítő kamera nevét `SN<serial>-<filter>` formában tartalmazza, ahol az `<filter>` a kamera modelljének szűrő része: `SN261902142-F450` egy `M3M-L87-F450` esetében, `SN213800234-FRGN` az `M3C-L87-FRGN` esetében, `SN214701292-FRGB` egy RGB-kamera esetében. A szűrő a sorozatszámot az `-`-hez csatolja, nem pedig az `_`-hez, így a név ugyanazokat az `_`-szel elválasztott mezőket tartalmazza, mint korábban (előtag, dátum, idő, milliszekundumok, kamera, szint). Azok a kamerák, amelyek modellje nem olvasható ki (törölt DeviceUserID), egyszerűen `SN<serial>` néven szerepelnek. A korábbi kiadásokból származó felvételek nevében nincs szűrő; az Chloros mindkettőt elolvassa.
- **Előtag:** Az `lattice capture` fájlokat `capture_…`; `array-capture`, `multi-capture` néven nevezi el egy csatlakoztatott tömb esetén, az `project capture` / `burst` / `stream` nevű sorozatból származó fájlokat pedig `sync_<ts>_SN<serial>-<filter>…` néven.
- **Szint utótag csak több szint esetén.**Az a kamera, amely**több**szintet ment egy felvételben (`all` vagy `--levels a,b`), hozzáadja az `_<level>` utótagot. Az a kamera, amely**egy** szintet ment, megtartja az egyszerű nevet, utótag nélkül — az `--processing reflectance` fájl `reflectance/sync_<ts>_SN<serial>-<filter>.tiff` néven kerül mentésre — így az egyszintű felvételekhez írt szkriptek továbbra is működnek. A szintet továbbra is a mappa adja meg.
- **A korlátozó maszkok a termék nevét követik:** `<product name>_clampmask.tif`. Egy szintű reflektancia-felvétel esetén az `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif` név íródik; egy `all` felvétel esetén az `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif` név íródik.
- **A kiterjesztés** az `--format` után következik: az alapértelmezett `tiff` az `.tiff`-et adja, az `--format tif` az `.tif`-et eredményez (a korlátozó maszkok mindig `.tif`).
- **Egy kivétel:** az `lattice capture --processing debayered` (vagy az `--levels debayered`) egy gyors, egyetlen-képkockás útvonalat, és az `capture_<ts>_SN<serial>-<filter>.tiff`-et közvetlenül az `-o`-be írja — alkönyvtár nélkül.
- **A streamek és a burstok** ugyanazt a mezőt hordozzák: az `lattice continuous` fájl az `frame_<seq>_SN<serial>-<filter>.tiff`, az `lattice multi-stream`, az `stream_<seq>_SN<serial>-<filter>_<frame id>.tiff` fájlokat, valamint egy nyers sorozatot, az `<burst>_f<frame>_SN<serial>-<filter>_raw.tiff` nevű nyers sorozatot.

Az `<ts>` a rögzítés időbélyege, az `<serial>` pedig a kamera sorozatszáma, így egy szinkronizált csoport
időbélyegét a kamerák között megosztják. **Figyeljük meg az egyetlen aszimmetriát:** az `display` szint egy
`preview/` nevű mappában van tárolva, míg a többszintű fájlok az `_display` nevet — a mappa és az utótag csak
ezen a szinten különbözik. Az ismeretlen szintek a saját nevüket viselő mappába kerülnek, és ha az almappa
nem hozható létre, a fájl a kimeneti gyökérmappába kerül, ahelyett, hogy elveszne.

**A felvételek mappájának újrafeldolgozása:**irányítsd az `chloros-cli process` fájlt a**captures gyökérkönyvtárra**
(`output/`). Az `process` általában csak a megadott mappát importálja, de ha abban a mappában nincsenek
képet tartalmaz, de almappákkal rendelkezik, akkor automatikusan lefelé halad – így a szint alkönyvtárak és az
`daq/` almappa is egy lépésben felvételre kerül. A rögzítés minden szintje egyetlen képként kerül importálásra,
a többi szint pedig módként érhető el, nem pedig szintenként egy-egy kép formájában.

A **szint almappa** közvetlen megnevezése (pl. `output/raw/`) szintén működik. Ez esetben az `daq/`
mappa hátramarad, ezért másolja át vagy mutasson rá a DAQ-olvasmányra, amikorradiometrikus
terméket származtat az `raw/` fájlból – ellenkező esetben az időbélyeg-egyeztetésnek nincs mihez viszonyítania.

**A feldolgozás mindig az `raw` fájlból indul.** Minden felvételen belül a nyers képkocka a feldolgozási folyamat forrása;
az `debayered`, `radiance`, `reflectance` és `preview` megjeleníthető módokként jelennek meg, de soha nem kerülnek
vissza a feldolgozási folyamatba. Egy származtatott termék újrafeldolgozása újra alkalmazná a vignettát, a CCM-et és
a sugárzási számításokat, amelyek már be vannak égetve a képpontjaiba, ezért az Chloros inkább elutasítja, mintsem
hogy kétszer dolgozza fel. Két tudnivaló következmény:

- Az `index/` és `composite/` renderelések **soha** nem kerülnek feldolgozásra. Ezek kimeneti fájlok, nem rögzítések —
  egy NDVI LUT-rendernek nincs értelmezhető sugárzási értelmezése.
- Egy olyan rögzítések mappa, amelyet **az `raw` nélkül** exportáltak (pl. `array-capture --processing reflectance`),
  nincs érvényes feldolgozási forrása. Ezek a rögzítések normálisan importálhatók és megjeleníthetők, de az `process`
  kihagyja őket, és ezt jelzi:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Ha valóban szükség van egy származtatott termék továbbítására — például egy
  `demosaic` beállítással rögzített hub-munkamenetet, vagy egy régi mappát –, az `--input-level {raw,debayered,processed}` kényszeríti a belépési
  pontot, és felülírja az átugrást. Ez a jelző egy szándékosan beépített menekülési út; az `auto` (az alapértelmezett)
  soha nem dolgozza fel azokat a rögzítéseket, amelyek nem tartalmaznak nyers adatot.

**Kalibrációs címkék: csak az `raw` esetén.** A felvétel `raw` fájlja tartalmazza a kamera kalibrációs csomagját: az `CalibrationJSON` hasznos adatot és a MicaSense-kompatibilis címkéket (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), amelyek az érzékelő nyers értékeit írják le. Minden második szint fájlja (`debayered`, `display`, `radiance`, `reflectance`, az `index/` render) és minden `lattice continuous` / `multi-stream` képkocka hordozza a kamera azonosítóját (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...), valamint a képkocka saját adatait, az `XMP-mapir:FrameJSON` fájlt (expozíció, erősítés, hőmérséklet, pixelformátum, rögzítési idő, DAQ-egyezés), mivel minden `process` termék hordozza az azonosítót. Az `process` a sorozatszám alapján keresi meg az ilyen fájl kalibrációját, az `CalBundleSha256` pedig a Chloros kalibrációs gyorsítótárában, és egyszer letölti a csomagot, ha a számítógépen még nincs meg.

#### Több kamerából készült fájlok

Több kamera sávjaiból készült fájl – többsávos réteg (`align-apply`, `align-stream`, `index --live --save-multiband`), több kamerából készült élő index (`index.tif`, `index_LUT.png`), egy tömbfelvétel `composite/` képe — felsorolja az összes benne szereplő kamerát:

- **Kameránkénti listák, sávsorrendben** (az összetett kép: háttérréteg, majd előtér; egy indexréteg, amelyen szerepelnek azok a kamerák, amelyek sávjaiból származik): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (az a kalibrációs csomag, amelyen az egyes kamerák sávjai átmentek, egyébként — egy `align-apply` / `align-stream` halmaz, amelynek sávjai egyiken sem mentek keresztül — a kamera azon csomagja, amelyet a Chloros tárolt a gyorsítótárban, és amelynek nevét a rögzítési fájlok viselik), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — párhuzamos listák, amelyek *n*-edik eleme mindegyikben ugyanazt a kamerát írja le; `-`, ahol egy kamerának nincs értéke (nincs kalibrációs csomagja, ismeretlen objektívje van). Minden kamera **egy** elem: amennyiben a csomagja egy mezőhöz több értéket is tartalmaz — például a csomag halogénlámpa-tanúsítványai —, azokat `;` (`F-1479-1;F-1480-1;F-1481-1`) jelöléssel kötik össze, soha nem vesszővel, így a lista kameránként egy tételt tartalmaz, függetlenül attól, hogyan van kinyomtatva.
- **EXIF `SerialNumber`** ugyanazokat a sorozatszámokat tartalmazza, vesszővel elválasztva (`214000535,214000551`).
- **Nem** tartalmaz egyetlen kamera azonosítóját sem (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): ezek egy-egy kamerát és annak csomagját nevezik meg, és az `process` ezekből határozná meg a fájl kalibrációját. A több kamera sávjaiból számított index (`index.tif`, `index_LUT.png`) az EXIF-adatokban a következőket tartalmazza: `Model` `LATTICE` (a rendszer), nem pedig egyetlen kamera `LATT-…` modelljét.

Egy **egyetlen** kamera sávjaiból álló fájl — sávonkénti TIFF-fájl (`--per-band`), egy `align-apply --per-camera` képkocka, egy egyetlen kamera képhalmaza vagy élő indexe — hordozza az adott kamera saját azonosító címkéit, ahogyan minden más rögzített fájl is. Az `align-apply` halmaza, sávonkénti és kameránkénti fájljai, valamint minden `align-stream` halmaz és annak sávonkénti fájljai a csoportjuk rögzítési idejét hordozzák (a referenciakamera expozíciója, UTC), és egy monokamerás `--per-camera` képkocka egy sík, az `--bit-depth`-hez hasonlóan, mint a képcsomag.**Kamera tájolása (tükrözés / forgatás).** A kamera tájolási beállítása (a grafikus felhasználói felület Tükör H /
Mirror V / rotation) határozza meg, hogy a rögzítés mit ment el:

- **Az `raw` és az `debayered` pontosan úgy kerülnek mentésre, ahogyan az érzékelő leolvasta őket**, függetlenül a beállítástól. Egy
  Elfordított vagy tükrözött Bayer-mozaik eltérő színszűrő-mintázatot eredményezne, és a kalibrációs
  térképek (sötét eltolás, sík mező) az érzékelő saját rácsán helyezkednek el.
- **Az `preview`/`display`, `radiance`, `reflectance` és az `index/` renderelt képek orientáltak** —
  egyszer elfordítják őket, miután az összes kalibrációs számítás megtörtént, így úgy nyílnak meg, ahogyan a kamera beállította a képet. Az
  `clamp_masks/…_clampmask.tif` fájljaik ugyanazon a rácson helyezkednek el.
- **Minden felvételi fájl rögzíti a beállítást** az XMP-ben: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, az óramutató járásával megegyező irányban,
  a tükrözések után), valamint az `ChlorosOrientationApplied` – `false` a nyers és a debayered (natív pixelek) fájlokban,
  `true` az orientált termékeken. Az `ChlorosProcessing` felsorolja az adott fájl
  pixeljeire alkalmazott műveleteket (`none` natív esetben). Ez magában foglalja az `lattice continuous` által továbbított minden képkockát (a
  kamera saját beállítása, alkalmazás nélkül) és az `lattice multi-stream` minden képkockáját (az identitás:
  tagképkockái natívak, mint egy tömbfelvétel esetében). Az `lattice align-stream` / `align-apply`
  halmazok a fő kamera beállításait rögzítik, alkalmazás nélkül: sávjaikat a fő kamera
  képkockájához igazítják, de nem forgatják el.
- **Az `process` a rögzített tájolást alkalmazza**, amikor egy nyers adatot feldolgoz: először a sötét eltolást, a sík
  mezőt, a demosaikot és a keverés feloldását az érzékelő koordinátarendszerében, majd a mátrix-igazítási torzítást, végül a
  tájolást végül — így az előnézet, a sugárzás, a visszaverődés és az index termékek pontosan úgy jelennek meg, mint
  az élő felvételnél, és rögzítik az `ChlorosOrientationApplied=true`-et. A nem kalibrált tartalék
  termékek (`Sensor_Response_Images`, `Vignette_Corrected_Images`) ugyanúgy vannak tájolva, a
  a síkkép-korrekciót. A debayerezett termék natív formátumban marad.
- **Az Chloros képnézegető a nyers vagy debayerezett fájlt a felvétel szerinti irányban jeleníti meg** (csak megjelenítés céljából —
  a fájl nem változik), így úgy néz ki, mint a termékei; a miniatűr, a teljes méretű nézet, a
  kurzor kijelzése és az Index/LUT sandbox mind ugyanazt a forgatást használja. A rekord nélküli fájl, valamint
  egy (már orientált) termék a tárolt formában jelenik meg. A sandbox-export rögzíti a
  pixelek által hordozott tájolást (`ChlorosOrientationApplied=true`; az azonosító egy beállítás nélküli réteg esetében).
- **A rekord nélküli felvételeket natívként dolgozza fel** (változatlan viselkedés). Az 1.3.0-ás verzió előtt egy
  tükrözéssel vagy forgatással beállított, egykamerás felvétel maga forgatta el a nyers és a debayerezett fájljait;
  a Chloros nem tudja megkülönböztetni az ilyen, már-elforgatott nyers fájlt nem tudja megkülönböztetni a natívtól, és nem is próbálja kitalálni, ezért
  a tárolt állapot szerint dolgozza fel. A kalibrációs térképek az érzékelő saját rácsán vannak, ezért ilyen
  felvételek esetén használjon egy 1.3.0-val készítettet, ha a kalibráció fontos.
- **A tömbök alapértelmezés szerint egyetlen tagfájlt sem forgatnak el** (az igazítási torzítás és az `Alignment*` XMP fájl a
  kamerák saját, el nem forgatott képkockáin található), és rögzítik az identitást. Az `array-capture --orient` az igazítási torzítás után minden
  tag termékeit elforgatja: a **fő** orientációja szerint, amikor azokat
  a keretébe torzították (így a tagok továbbra is egymáshoz igazodnak), egyébként a saját kamerájuk orientációja szerint. Minden
  fájl rögzíti annak a képkockának a tájolását, amelyben a saját képpontjai találhatók — egy nyers fájlban a saját kamerája tájolását —
  és egy olyan nyers fájl, amely az igazítási transzformációt is tartalmazza, rögzíti az `ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation` értékeket: a mesterét, ha a termék
  a mester képkockájába. Az `process` minden terméket a pixeljeihez tartozó értékkel forgat el:
  az igazítottat, amikor a nyers fájlt torzítja (`--array-alignment`, az alapértelmezett — az élő igazítású
  felvétel termékei), a kamera sajátja, ha nem igazítja (`--no-array-alignment` – az élő
  nem igazított felvételeké). Ugyanez az opció létezik az `array-burst`, `project capture` / `burst` /
  `stream`, a receptek és az SDK esetében is.

### Kihagyott felvételek vegyes szűrőjű rendszerekben

Ha egy sorozatban keverjük az RGB- és multispektrális kamerákat, az `array-capture --processing radiance` (vagy az `reflectance`) elmenti a multispektrális képkockákat, és **kihagyja** az RGB-kamerák felvételeit — a Bayer-mátrixonkénti sugárzásérték szélessávú érzékelő esetén nem releváns. A CLI kifejezetten kinyomtatja az egyes mentett fájlokat (az exportálási szintjükkel együtt), az egyes `.daq` bejegyzéseket, valamint az egyes kihagyásokat, így a fájlok száma nem meglepő:

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

A kihagyás okát jelző tokenek az `<level>-not-applicable-to-rgb-cam` mintát követik. Az a kamera, amelynek kalibrációs csomagja nincs betöltve, kihagyja az `radiance` / `reflectance` fájlokat az `<level>-needs-calibration (connect the camera so its cal pack loads)`-re, és ha nincs használható DAQ-reflektancia-érték, akkor az `reflectance-skipped-no-dls`-re ugrik. A reflektancia az `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`-re is átugorhat, valamint az `dls-uncalibrated-band-<nm>` esetén is, ha a sáv nagy része a DAQ fényérzékelő radiometrikusan kalibrált tartományán (~374–974 nm) kívül esik — a forgalomban lévő SKU-k közül csak az F988, amelynek támogatott útvonala a reflektancia-panel munkafolyamat.

Használja az `--processing debayered` (vagy `display`) beállítást, hogy minden kamerát bevonjon a szűrőtípustól függetlenül, vagy az alapértelmezett `all` beállítást, hogy egy lépésben megkapja az összes alkalmazható szintet kameránként.

### Az expozíciótól távol eső DAQ-érték

Minden egyes `.daq`, amelyet egy felvétel ment, az adott expozícióhoz tartozó DAQ-érték, a mellette lévő `.match.json` pedig rögzíti, hogy a kettő milyen messze volt egymástól (`gap_ms`), és hogy ez a tűréshatáron belül van-e (`in_tolerance`, 500 ms, hacsak a rögzítési beállítás másként nem határozza meg: `--daq-tolerance-ms`). A tűréshatáron kívüli értéket is felhasználják, így egyetlen képkocka sem vész el, és azt megjelölik: a reflexiós fájl XMP-adatmezője tartalmazza az `DlsInTolerance=False` és az `DlsMatchGapS` értékeket. Az `lattice capture`, `multi-capture` és `array-capture` kódok szintén kinyomtatják az érintett `.daq` kód alatt, a DAQ'adatfolyam-hibájával együtt, amikor ezt elmagyarázzuk:

**Hogyan illeszkedik egy leolvasás az expozícióhoz.** Egy rögzítés az expozíció utáni első DAQ-leolvasásra vár, hogy össze tudja keverni a leolvasásokat mindkét oldalán (`method` `interpolated`). A DAQ-E nappal 0,4 másodpercenként, beltérben pedig 1–2 másodpercenként küld mérési értéket, így a várakozási idő körülbelül másfél ilyen intervallummal hosszabbítja meg a parancs futási idejét, nem pedig az expozícióét. Az a DAQ, amelyik leállt az adatküldéssel, nem kerül megvárásra. A DAQ-E leolvasási értéke az előző leolvasás óta eltelt időtartam alatt átlagolt fényérték, ezért az adott időtartam közepére kerül: az `gap_ms` értéket az expozíciótól az adott közepéig mérik, az `bracket_stamps_ns` pedig a leolvasási értékek saját időbélyegeit adja meg. Az `array-capture --continuous` várakozik anélkül, hogy lassítaná a rögzítést: a nyers fájljait azonnal írja, az `.daq` értékét pedig az érték beérkezésekor. Az `--interval` esetén minden átfutás úgy várakozik, mintha egyetlen felvételről lenne szó.

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

Ha az adott leolvasásból nem írtak reflektanciafájlt, az első sor helyette az `saved anyway; its .match.json says in_tolerance: false`-szel végződik.

A rögzítés továbbra is az `0`-szel zárul: minden fájl írásra került, és feltünteti a tartalmát. Az a DAQ, amelynek adatfolyama leállt, továbbra is az utolsó leolvasási értéket szolgáltatja, ezért az adott rögzítés reflektanciájára való támaszkodás előtt ellenőrizze azt az `chloros-cli daq pool-list` segítségével. A háttérrendszer újra csatlakoztatja azt a poolba tartozó DAQ-E-t, amely leállt az adatátvitelben (lásd [Egy adatgyűjtő, amely leáll az adatátvitellel](#a-daq-e-that-stops-delivering)). A tűréshatáron kívüli mérési érték csak az `DAQ:` sorát írja ki, kivéve, ha megőrzött mérési értékről van szó (lásd alább).

**A DAQ leállása után megőrzött mérési érték.** Ha egyetlen DAQ sem tud mérési értéket szolgáltatni egy expozícióhoz (a lekötött DAQ nem elérhető, vagy egyetlen poolba tartozó DAQ sem rendelkezik ilyennel, pl. az `daq pool-disconnect` után), a rögzítés újrahasznosítja az Chloros által utoljára rögzített értéket, és az `.match.json` kódja ezt az értéket az adott expozícióhoz rendeli: `method`, `retained`, `gap_ms`-et az adott expozícióból a mintákra, amelyekről az értéket mérték, az `in_tolerance`-et abból a résből, valamint az `retained_from_epoch_ns`-et, azaz azt az expozíciót, amelyhez az értéket először illesztették. Egy olyan rögzítés, amelyet jóval a DAQ leállása után készítettek, ezért a fenti sorokat nyomtatja ki, és még egy, amely akkor is kinyomtatódik, ha az érték még a tűréshatáron belül van:

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

Az `.daq` továbbra is megnevezi azt a DAQ-ot, amely az értéket rögzítette, valamint a kalibrációt és a fedelet, amelyek mellett az érték felvételre került, így egy újbóli importálás során tudni lehet, melyik DAQ, kalibráció és fedél eredményezte azt.

### Azok a fájlok, amelyek írása sikertelen

Az a kamera, amelynek fájlja nem írható – teli meghajtó, jogosultsági hiba –, nem akadályozza meg a többi kamera fájljainak mentését. Minden rögzítési parancs (`lattice capture`, `multi-capture`, `array-capture`; `project capture`, `burst`, `stream`; valamint egy recept `capture`, `burst` és `stream` parancsai) minden sikertelen fájlt kinyomtat a stderr-re, majd **kilép az `1`-ből**, annak ellenére, hogy a többi mentésre került:

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

Minden, a rögzítéshez tartozó fájl szerepel, beleértve az egyetlen-képkockás, debayered `lattice capture --levels debayered` is. Az a `.daq` olvasás, amelynek `.match.json` eredetét igazoló sidecar fájlja nem írható, `[match]` néven szerepel (maga az `.daq` fájl elmentésre került).

Teljes meghajtón az ismétlődő rögzítések (`array-capture --continuous` / `--interval`, `project burst` / `stream`, egy recept `burst` / `stream`) **az adott futtatási lépésnél leáll**, ahelyett, hogy a futtatás hátralévő részében semmit sem írna ugyanarra a meghajtóra. Bármely más hiba minden olyan futtatási lépésnél jelentésre kerül, ahol előfordul, és a futtatás a végéig folytatódik, majd kilép az `1`. `array-capture --continuous` (a háttérrendszer folyamatos rögzítése) nem sorol fel fájlokat, a sikerteleneket sem: azokat a haladási és összefoglaló sorokban számolja (`N failed`, `N file(s) failed to write.`), majd kilép az `1`-ből — lásd [Folyamatos mód](#continuous-mode---continuous).

A rögzítők — `array-burst` és `array-record` — jelentik a saját hibáikat. Az `array-burst` leáll, amikor a meghajtó megtelik, vagy közvetlenül azelőtt. Mindkettő tovább működik bármely más sikertelen írási kísérlet után is. Amikor bármelyikük hibát jelent, az a számlálók után kerül kinyomtatásra, és a parancs **kilép az `1`-ből**:

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

Az `Disk low:` az `Disk full:` helyett azt jelenti, hogy a sorozat a meghajtó megtöltése előtt leállt. A leállás előtt rögzített keretek megmaradnak. Az `array-burst --build` kihagyja a felépítést, amikor elfogy a hely, mert a videók ugyanarra a meghajtóra kerülnének. Először szabadítson fel helyet, majd futtassa az `array-build-video --burst-dir …` parancsot. Bármely más hiba után a felépítés továbbra is fut.

Az `array-record` hiba, amely egyetlen képkockát sem írt, szintén sikertelen felvételt jelent (`Recorder error: no-frames: the combined view published no frame to record …`, lépjen ki az `1`-ből) — lásd [`array-record`](#array-record--combined-index-videogif-monitoring-grade).

---

## Rögzítési módok, rögzítők és offline újrafeldolgozás

Mindezek egy **állandó tömbön** működnek (először futtassa az `array-connect` parancsot). Ezek a GUI rögzítési paneljét tükrözik.

### `array-capture` módok

Az `array-capture` egy egyetlen parancs, amely négy exponálási módot és egy sor exportálási kapcsolót tartalmaz:

| Mód | Jelző | Viselkedés |
| --- | --- | --- |
| **Egyszeri** *(alapértelmezett)* | (nincs) | Egy szinkronizált rögzítési csoport, majd kilépés. |
| **Folyamatos** | `--continuous` | Egymást követő rögzítések, amíg az `Ctrl+C`, `--count N` rögzítések, `--duration S` rögzítések, vagy a teljes meghajtó megtöltése. A háttérprogram saját folyamatos rögzítését futtatja (a grafikus felület Folyamatos módja) — lásd az alábbi [Folyamatos mód](#continuous-mode---continuous) részt. |
| **Intervallum** | `--interval S` | Egy futtatás minden `S` másodpercenként (az egyes futtatások kezdetétől számítva), azonos határok mellett. |
| **Leggyorsabb** | `--fastest` | Csak nyers-csak + a hozzárendelt DAQ-érték + a kombinált indexű kompozit; kihagyja a sugárzási/visszaverődési/megjelenítési számításokat, így a képkocka gyorsan megjelenik. Magában foglalja az `--processing raw --force-daq`-et. A mentett `.daq`-et később kalibrált termékekké dolgozza fel. |

Exportálási kapcsolók (bármely móddal kombinálhatók; mindegyik megosztja a GUI/SDK végpontot):

| Jelző | Hatás |
| --- | --- |
| `--array-id ID` | A rögzítendő tömb (hagyja ki, ha csak egy van csatlakoztatva). |
| `-o, --output DIR` | Kimeneti mappa (alapértelmezés: `output`, a parancs futtatásának helye alatt). |
| `--format {tiff,tif,png,jpg,jpeg}` | Fájlformátum (alapértelmezett: `tiff`). A PNG/JPEG formátumok csak az `display` és az index renderelést fogadják el; lásd [Rögzítési exportszintek](#capture-export-levels-the-all-default). |
| `--processing LEVEL` | Egyetlen exportszint (`raw`, `debayered`, `radiance`, `reflectance`, `display`), vagy `all` (alapértelmezett). |
| `--levels L1,L2,…` | Az exporttípusok explicit részhalmaza (pl. `raw,radiance,reflectance`); **felülírja az `--processing`-et**. |
| `--aligned` / `--no-aligned` | **Alapértelmezés: igazítva, ahogyan az Chloros-alkalmazás ugyanazon tömböt rögzíti**(a tömbönkénti**Igazított** rögzítési beállítás, amely be van kapcsolva, hacsak ki nem kapcsolod). Minden tag nem nyers formátumú exportja (`debayered`, `display`, `radiance`, `reflectance`, az index-render) a fő kamera képkockájához igazodik, és a tagok közös átfedési területére van kivágva, így a fájlok pixel-regisztráltak, és méretük, kivágásuk, valamint az `Alignment*` címkék megegyeznek az alkalmazáséval — kisebbek, mint az érzékelő (vagy az ROI-nál) a tagok eltolásai miatt; ROI esetén a transzformációt az ablakra alapozzák át. Az `XMP-mapir:AlignmentJSON` megadja az `"applied": true` értéket, és kiadja az `crop_roi`-et. A nyers fájl torzításmentes marad a natív méretben, de a transzformációt a metaadatokban hordozza (`"applied": false`). Az `--no-aligned` minden fájlt natív méretben, torzításmentesen ment. Az `--aligned` az alapértelmezett beállítás. Függetlenül attól, hogy a torzítás alkalmazásra kerül-e vagy sem, minden tagfájl rögzíti a kamerájának `Alignment*` transzformációját, amint a tömb rendelkezik igazítási profillal — kivéve azokat a kamerákat, amelyeket a profil nem igazít (kihagyásra kerülnek, vagy sikertelen megoldás miatt identitásra kényszerülnek), valamint a referenciakamerát, ha más kamera nincs igazítva: ezek nem hordoznak `Alignment*` címkéket. Ha a tömbhöz nincs profil, a felvétel igazítás nélkül kerül mentésre, és a CLI az okot egy `Alignment:` sorban jeleníti meg az okot, ahogyan minden más olyan esetben is, amikor a torzítás nem alkalmazható (különböző felbontású elemek, kábellel nem szinkronizált tömb, olyan képkocka, amelyre a profil nem térképezhető fel). A tömb automatikus igazítása létrehozza a profilt, miután az expozíció stabilizálódik; az `align-calibrate` pedig igény szerint hoz létre egyet. |
| `--orient` | Fordítsa el az egyes elemek **termékeit**— `display`, `radiance`, `reflectance`, az `index/` render és az `composite/` — a kamera tájolásához igazítja az igazítási torzítás után: a termék a**master**tájolása szerint a master képkockájába torzított termék, így az elemek továbbra is egymással regisztrálva maradnak; egy igazítatlan termék (`--no-aligned`, vagy egy olyan felvétel, amelyre a torzítás nem alkalmazható) a saját kamerája tájolása szerint.**Alapértelmezés: ki — minden elemfájl natív marad**, ahogyan a tömbös felvételeknél mindig is. Az `raw` és az `debayered` mindkét esetben natív formátumban marad. Minden fájl rögzíti a termékei által hordozott tájolást (lásd [Kamera tájolása](#what-a-captures-folder-looks-like)); az `Alignment*` XMP fájl a natív keret transzformációját tartalmazza, amelyet a tájolás előtt alkalmaznak, így az `process` (torzítás, majd orientáció) reprodukálja az élő terméket. |
| `--index` / `--no-index` | A kameránkénti növényzet-index átfedés mentése / kihagyása, amennyiben ilyen van beállítva. Alapértelmezés: megjelenítése. |
| `--no-daq` | Hagyja ki az `.daq` sidecar fájlt, ha egyetlen kiválasztott szint sem használja a DAQ-ot (csak nyers adatokat tartalmazó felvétel). Alapértelmezés szerint az érték mentésre kerül, így a képkockák offline módon újrafeldolgozhatók reflektancia/index formátumba. Az `--force-daq` beállítás továbbra is elfogadott (ez az alapértelmezett). |
| `--daq-tolerance-ms MS` | A DAQ-egyezési tűréshatár ezeknél a rögzítéseknél (alapértelmezés szerint 500 ms). |
| `--smart` | Várja meg, amíg az AE minden kamerán stabilizálódik, mielőtt elindítaná a rögzítést (lásd [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | TIFF-pixeltömörítés. `none` (alapértelmezett az 1.3.0 verziótól; az 1.2.0 verzióban `deflate` volt) = tömörítetlen — az érzékelő pixeljei pontosan úgy, ahogyan kiolvasásra kerültek, ~6,3 MB teljes felbontású képkockánként, és a leggyorsabb folyamatos sebesség; `deflate` = veszteségmentes zlib L1 + vízszintes prediktor, ~4,1 MB képkockánként, lassabb. Mindkettő veszteségmentes, és importáláskor azonos módon olvasható. |

> **Egyszer írt TIFF + a folyamatos sebességű modell.**Minden fájlt**egyszer**írnak: pixelek, az XMP-csomag, az IFD0 Make/Model/CameraSerialNumber és az EXIF `ExposureTime` / `SerialNumber` címkék egy lépésben, utána semmi sem írja felül őket. Az `--continuous`X egymást követő, szinkronizált csoportokat rögzít — minden rögzítés az előzőt követő csoportot veszi át, beleértve azt is, amelyik már megjelent, miközben az adott rögzítés még mentés alatt volt — így a tömb sebességével fut. A DEFLATE lefoglalja a Python GIL-t, így a tömörített írások**nem**párhuzamosodnak a kameránkénti író szálak között; a szenzor sebességén történő folyamatos, teljes felbontású rögzítéshez tömörítetlen írásokra van szükség (ez az alapértelmezett)**és** NVMe-osztályú lemezre van szükség (~600 MB/s folyamatos sebesség egy 9 kamerás tömbnél ~10 fps-nél). Ugyanez a beállító gomb `compression` néven jelenik meg az `POST /api/camera/array/capture`-en.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### Folyamatos mód (`--continuous`)

Az `array-capture --continuous` (az `--interval` vagy az `--smart` nélkül) átadja a feladatot a **háttérrendszer saját folyamatos rögzítésének** — annak a munkamenetnek, amelyben a grafikus felhasználói felület Folyamatos módja fut. A háttérrendszer minden szinkronizált csoportot önállóan ment, két rögzítéssel párhuzamosan, és a csoportok között nem érkezik kérés a CLI-től, így lépést tart egy gyors rendszerrel, ahol a rögzítésenkénti kérés lemarad (9 kamerás rendszerben, 512 pixeles ROI-nál mért érték: 73–77 rögzítés/másodperc). A fenti összes exportálási beállítás minden egyes rögzítésre vonatkozik.

- **Határok:** Az `--count N` N rögzítés után leáll (egy rögzítés = egy szinkronizált csoport: minden tag fájljai); az `--duration S` pedig S másodperc elteltével. Ha egyik beállítás sincs megadva, a program addig fut, amíg az `Ctrl+C` beállítás nem lép életbe, a meghajtó meg nem telik, hiba nem lép fel, vagy a rendszer le nem áll. Az `Ctrl+C` leállítja a háttérprogram munkamenetét, majd kinyomtatja az összefoglalót.
- **Haladás** 2 másodpercenként, `, N failed` és `, N groups skipped` hozzáfűzve, ha nem nulla (az elhagyott csoport az a csoport, amelyet a tömb közzétett, de amelyről nem történt rögzítés):

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **Nincs fájlonkénti felsorolás.** A futtatás egy összefoglaló sorral zárul — rögzítések, sebesség, fájlok és a kimeneti mappa —, amelyet a nem nulla érték esetén az `N file(s) failed to write.` / `N group(s) skipped.` követ:

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **Leáll**, ha a meghajtó megtelik (`Continuous capture stopped: the drive is full.`), illetve hiba esetén vagy a tömb eltűnése esetén (`Continuous capture stopped: <reason>.`) (mindkettő a stderr-en jelenik meg).
- **Kilépési állapot:** `0`, ha a rögzítéseket elmentette és minden fájl írásra került, beleértve az `Ctrl+C` fájlt is; `1`, ha bármely fájl írása sikertelen volt, a meghajtó teljessége, hiba, a tömb megszűnése vagy a rögzítés teljes hiánya esetén.
- **Microsoft Defender:** ha a Defender egy vagy több CPU-magot használ fel a rögzítés által írt fájlok vizsgálatára, a futtatás egyszer kinyomtatja a következő üzenetet a stderr-re: `Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.` Lásd a telepítő [Rendszerbeállítások](#windows-exe) című részét.
- Egyszerre csak egy folyamatos rögzítés lehetséges tárolócsoportonként; a másodikat a rendszer elutasítja az `a continuous capture is already running for this array` hibaüzenettel. HTTP-en keresztül a munkamenet `POST /api/camera/array/continuous/start` (az `/api/camera/array/capture` test, valamint az `max_captures` / `duration_s`), `GET /api/camera/array/continuous/status?array_id=ID` és `POST /api/camera/array/continuous/stop`.

**Interval**(`--interval S`, `--continuous`-szel vagy anélkül) és**`--continuous --smart`** megtartják a kérésenkénti ciklust: egy rögzítési kérés minden futtatásnál — az Interval esetében minden ütemezett időpontban, az `--smart` esetében az AE leállása után — az egyes futtatások fájljait az Egyedi módhoz hasonlóan sorolja fel, a végén egy `Done — N capture pass(es).` sorral, és az `--count N` számolja a futtatásokat.

### `array-record` — kombinált indexű videó/GIF (felügyeleti minőségű)

Rögzíti a tömb **élő kombinált index nézetét** egy `.avi` fájlba (és opcionálisan egy `.gif` fájlba). A grafikus felület kombinált nézetének megnyitásakor a program a megjelenített nézetet rögzíti. Ha nincs megnyitva, a háttérprogram maga rendereli a nézetet: az `--expr`-szel átadott indexet, vagy alapértelmezés szerint a tömb NDVI-jét, miután a tömb összehangolásra került. Az alapértelmezett NDVI a 660 nm-hez és 850 nm-hez legközelebb eső vörös és NIR sávokat használja, ahogyan az élő nézet is. A felvétel közben megnyitott GUI-nézetet addig rögzíti, amíg az nyitva marad. 2 másodpercenként ellenőrzi az előrehaladást, és leáll az `--duration`, `Ctrl+C` kódra, vagy amikor a felvevő önállóan befejezi a működést.

Az a felvétel, amelyik nem ír ki egyetlen képkockát sem, sikertelen: a felvevő az `no-frames: …` hibakódot jelenti (lehet, hogy a képkockasorozat még mindig igazodik, vagy az expozíciója még nem állt be), és a parancs **kilép az `1`** hibakóddal.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| Jelző | Alapértelmezett | Leírás |
| --- | --- | --- |
| `--array-id ID` | csak tömb | Cél tömb (hagyja ki, ha csak egy van csatlakoztatva). |
| `-o, --output DIR` | `output` | Kimeneti könyvtár. A relatív elérési út a parancs a parancsot futtatja. |
| `--fps F` | `10` | Felvételi képkockasebesség. |
| `--duration S` | amíg Ctrl+C | Automatikus leállítás `S` másodperc elteltével. |
| `--gif` | ki | Animált GIF-fájl írása is. |
| `--gif-only` | ki | Csak GIF-et írjon (nincs `.avi`). |
| `--expr EXPR` | a tömb NDVI-je | Indexkifejezés, amelyet akkor kell rögzíteni, ha nincs megnyitva GUI-nézet, a tömb sávneveiben (pl. `'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'`; lásd [sávnevek](#index--vegetation-maths)). A rövid színnév (`NIR`, `Red`) csak akkor működik, ha a tömb egyik sávja rendelkezik vele. Nyitott GUI-nézet esetén a rendszer a megjelenített értéket rögzíti. |

### `array-burst` — nyers Bayer-formátumú, nagy képkockasebességű sorozatfelvétel (elemzési minőségű)

Közvetlenül olvassa be a rögzítési ciklus szinkronizált csoportjának pufferét — **nincs szükség kalibrációs láncra, exiftoolra vagy élő nézetre** — így a kamera teljes képfelvételi sebességén fut. Raw képkockákat ír + képkockánkénti manifesztet + egy `.daq` fájlt minden egyes DLS-leolvasáshoz az `<output>/bursts/<base>/` alatt. Feldolgozza offline módban (következő parancs), vagy adja meg az `--build` parancsot, hogy a leálláskor azonnal elvégezze.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Jelző | Alapértelmezett | Leírás |
| --- | --- | --- |
| `--array-id ID` | csak tömb | Cél tömb. |
| `-o, --output DIR` | `output` | Kimeneti könyvtár (a burst az `<DIR>/bursts/<base>/`-be kerül). |
| `--duration S` | Ctrl+C-ig | Automatikus leállás `S` másodperc elteltével. |
| `--max-frames N` | korlátlan | Automatikusleállás `N` nyers képkocka után. |
| `--orient` | ki | A build által ebből a sorozatból létrehozott termékek a kamera tájolása szerint fordulnak el — a kameránkénti termékek a saját kamerájuk, a kombinált indexet pedig a vezető kamera tájolása szerint – ahogyan az `array-capture --orient` is teszi. A nyers képkockák natív formátumban maradnak; a sorozat `manifest.json` fájlja rögzíti minden elem tájolását és az opciót. |
| `--build` | ki | A leállítás után azonnal dolgozza fel újra a sorozatot (ugyanúgy, mint az `array-build-video`). |
| `--products …` | `combined:index` | Az `--build` használatával: mely videó(k) készüljenek (lásd alább). |
| `--fps F` | `10` | Az `--build` beállítással: a kimeneti videó képkockasebessége (fps). |
| `--save-tiffs` | ki | Az `--build` használatával: mentse el a képkockánként kalibrált TIFF-fájlokat is (mindegyik a [unmix clamp mask](#unmix-clamp-mask-which-pixels-a-sáv-lett-korlátozva) jelöléssel, amikor egy pixel korlátozásra került; a fordítási eredmény ezeket `clamp_masks` néven sorolja fel). |
| `--gif` | ki | Az `--build` használatával: animált GIF-eket is ír. |

### `array-build-video` — mentett sorozat offline újrafeldolgozása

Időben összehangolja az egyes nyers képkockákat a legközelebbi mentett `.daq`-értékkel, és **ugyanazon a sugárzási / visszaverődési / index láncon** futtatja át őket, mint az importálási folyamat, így egy vagy több videót renderel.

Az `--products` egy vesszővel elválasztott lista `kind:level` elemekből, ahol `kind` ∈ `per_cam` | `combined`, és `level` ∈ `radiance` | `reflectance` | `index`. Egy önálló `level` (`kind:` nélkül) alapértelmezés szerint `per_cam`. Az alapértelmezett érték `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Jelző | Alapértelmezett | Leírás |
| --- | --- | --- |
| `--burst-dir DIR` | (kötelező) | A burst mappa elérési útja (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | `kind:level` lista, a fentihez hasonlóan. |
| `--fps F` | `10` | A kimeneti videó képkockasebessége (fps). |
| `--save-tiffs` | ki | A videó(k) mellett képkockánként kalibrált TIFF-fájlokat is ment. |
| `--gif` | ki | Animált GIF-fájlokat is írjon. |

**A videó nem tartalmaz korlátozó maszkot** (egy képsorozat, 8 bites). Ehelyett a fordítási eredmény tartalmazza az `clamp_fractions` értéket: videónként, minden képkocka pixeljeinek azt a részét, amelyet a keverés elválasztott az R / G / B csatornájában lévő sávban — az `Chloros:Band{R,G,B}UnmixClampFraction` értéket, amellyel az adott képkocka sugárzási/visszaverődési szorzatát megjelölnék (a kombinált indexben, minden részt vevő kamera esetében). A futtatás termékenként a legrosszabb képkockát egy `Clamp:` sorban nyomtatja ki. Az `--save-tiffs` hozzáadja a TIFF-fájlokhoz tartozó pixelenkénti maszkokat, az `clamp_masks/<level>/` fájlban, a TIFF-ek `<level>/` mappája mellett.**A megszakított összeállítás nem hagy hátra sérült videót.** Minden videó `<name>.partial.avi` néven kerül írásra, és csak akkor kapja meg végleges nevét, amikor elkészül, így egy régebbi, azonos nevű videó addig megmarad, amíg az azt felváltó videó teljes nem lesz. Futtassa újra a buildet, hogy kicserélje a hátramaradt `.partial.avi` fájlt.

A futtatás kinyomtatja az összes létrehozott videót, majd minden `Skipped:` terméket (olyan kért kamera/szint – vagy kombinált index –, amely nem eredményezett videót, az ok megjelölésével – pl. nincs kalibrációs csomag, nincs DLS-leolvasás a reflexióhoz, vagy nincs vörös/NIR sávpár a kombinált indexhez), valamint minden `Note:`-et. Egy sugárzási vagy reflexiós adatot kérő RGB-kamera helyett a kijelzővideóját kapja meg, fehér egyensúlyozva, ha a kamera kalibrációs csomagja elérhető; az anélkül renderelt kamerát egy `Note:`-ben nevezik meg. **Nem nulla értékkel lép ki**, ha egy kért terméket kihagytak, vagy nem a kérésnek megfelelően állítottak elő (a konfigurált indexkifejezést nem értékelik ki offline módban); az elkészített videókat továbbra is írja és sorolja fel.

> **Válassza ki a megfelelő rögzítőt.** Az `array-record` *megfigyelési minőségű* — a renderelt élő kompozíciót rögzíti: a grafikus felhasználói felület (GUI) kombinált nézetét, vagy ha nincs megnyitva a GUI, akkor a háttérrendszer saját NDVI (vagy `--expr`) nézetét. Az `array-burst` → `array-build-video` *elemzési minőségű* — teljes sebességgel menti az érzékelő nyers adatait, majd utólag rekonstruálja a kalibrált sugárzási/visszaverődési/index videókat, élő nézet nélkül.

### Mono (M3M) egycsatornás kamerák

Az **M3M**sorozat a Bayer**M3C**monokróm változata: kameránként egy keskenysávú interferencia-szűrő (`M3M-<lens>-F<wavelength>`, pl. `M3M-L87-F685`), így az érzékelő**egyetlen szürkeárnyalatos sávot** ad ki, Bayer-mozaik nélkül. Nincs mit demosaikolni, nincs csatornák közötti átvitel, amit szét kellene választani, és nincs fehér egyensúlyt beállítani — az egész RGB-megjelenítési színfeldolgozási folyamat egyszerűen nem vonatkozik rá.

Mit jelent ez a CLI-n:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**felismeri a monokróm kamerát, és**egy soros üzenettel továbbhalad**, ahelyett, hogy értelmetlen beállításokat adna ki. Ugyanebben a munkamenetben egy RGB/Bayer M3C kamerával továbbra is normálisan működnek.
- **A Bayer-pixelformátum a mélységet jelenti.** Az M3M csak monokróm formátumokat kínál, így az `--pixel-format` vagy az `--preset` parancsokból származó Bayer-formátum ugyanolyan mélységű monokróm formátumot állít be: az `BayerRG8` és az `--preset high_speed` az `Mono8`-et eredményezi; az `BayerRG12`, az `high_quality` és az `triggered` az `Mono12`-et eredményez; a tömörített név tömörített marad (az `BayerRG12p` az `Mono12p`-et adja). Az `lattice format` fájlok a Mono formátumot jelölik. Az M3C és M3M kamerákat keverő tömb ugyanúgy egy formátumot vesz fel ugyanúgy: minden M3C-n a Bayer, minden M3M-en a Mono mélység. A név a bitmélységét jelenti: ez fordítva is igaz, így egy M3C-t soha nem helyeznek Mono formátumba (az `--pixel-format Mono12` az `BayerRG12`-nek felel meg egy M3C-n, függetlenül attól, hogy melyik kamera a tömb vezérlője), az `lattice format --set` pedig ugyanúgy leképezi, és kinyomtatja a beállított formátumot.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** továbbra is működnek — a sugárzás és a visszaverődés *sávonkénti* radiometrikus térképek, és egy sávra tökéletesen jól definiáltak. A monokróm képkockák **identitás** érzékelő-válasz mátrixot hordoznak (nincs 3×3-as keverés), így a sík érintetlenül halad át a kalibrációs számításokon.
- **Egyetlen monokróm kamera nem képes növényzetindexet előállítani.**Az NDVI/NDRE/stb. indexekhez legalább két sávra van szükség (pl. Red + NIR). Ahhoz, hogy monokróm hardverből indexet kapjunk,**több**, különböző hullámhosszúságú M3M kamerát kell különböző hullámhosszúságokra irányítani, azokat egy többsávos képcsomagba összehangolni, és *azt* indexelni:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live` maga indítja el a kamerákat (egy [ideiglenes munkamenet](#multi-camera-transient-sessions)), ezért először ne csatlakoztasson semmit: `--serials`, különben a profilban engedélyezett kamerák (lásd [Index / Vegetációs matematika](#index--vegetation-maths)). Amíg az Chloros bármelyik kamerát tartja (a GUI, `cam-connect`, `array-connect`) HTTP 409-es hibakóddal (`CAMERA_HELD`) elutasítja a kérést, és kilép (`1`); a kamerákat az `chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all` parancsokkal lehet felszabadítani.

Az `--channel` szimbólumoknak **pontosan** meg kell egyezniük az előre beállított csatornák neveivel (a kis- és nagybetűk megkülönböztetése érvényes; az NDVI-k kisbetűs `red`, `nir` — lásd: `--list-presets`), és a sávoldali névnek az igazított halmazban szereplő sávra kell utalnia (offline módban a 0-tól kezdődő sávindexek is elfogadottak, pl. `--channel red=0 --channel nir=1`).

A halmazon belüli megkülönböztető jel a modell karakterláncban szereplő `M3M` token (ez soha nem jelenik meg egy `M3C` karakterláncban), amely a GUI/SDK számára `is_mono` néven jelenik meg.

---

## Gazdagép hálózati kártyájának beállítása és hangolása (LATTICE-rendszerek)

A LATTICE kamerák a gazdagép Ethernet-adapterén keresztül továbbítják a GVSP-t, ezért többkamerás rendszerek esetén az adapter **illesztőprogramja**és**a vételi gyűrű mérete** ugyanolyan fontos, mint a kapcsolat sebessége. A helytelen beállítások `FRAMES WILL DROP` / `Reduce ROI to enable` hibaüzenetként jelennek meg a Rendszerbeállítások panelen (valamint az `lattice network-analysis` / az SDK `analyze_array_network()` részében), még akkor is, ha maguk a kamerák rendben működnek.

### USB 10GbE adapterek — Realtek RTL8157 („Realtek USB 10GbE Family Controller”)

Az alábbi beállítások a Windows-es illesztőprogram tulajdonságai. Linux rendszeren az USB-adapternek is van sebességkorlátja (400 MB/s x86_64-en, 200 MB/s az NVIDIA Jetsonon) (lásd [Tömb fps és burst modell](#array-fps--burst-model)), és a kernel socket fogadó pufferei minden adapter esetében fontosak — lásd [Linux-gépek](#linux-hosts--socket-receive-buffers).

| Elem | Szükséges érték | Miért fontos |
| --- | --- | --- |
| **Illesztőprogram-verzió**|**≥ v10.67 (2026. január)**, INF `rtump64x64sta.inf` | A régi**2016-os**illesztőprogram (v10.65, `rtump64x64.inf`) nem kezeli megfelelően a kikapcsolást és a hibajelentéseket,**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**hibát okozva leállításkor/újraindításkor/alvó üzemmódba lépéskor. Az átmenet lefagy (~5 perces időtúllépés), a felhasználó kénytelen erőszakosan kikapcsolni a rendszert, és az ismételt nem szabályszerű leállítások**megrongálják a WMI-tárat**(a PowerShell és az eszközök az `Invalid class` hibával kezdenek meghibásodni), valamint**megakadályozzák az USB-stack működését** a következő indításkor (a hálózati kártya nem aktiválódik; az USB-meghajtók felsorolása leáll). Frissítsen a realtek.com webhelyről (vagy a dongle gyártójától), mielőtt a tiszta újraindításokra támaszkodna. |
| **Fogadó pufferek**— kulcsszó: `ReceiveBufferLen` |**256**(az illesztőprogram maximuma) | A hálózati kártya RX gyűrűje. Az illesztőprogram alapértelmezett**32**értéke csak ~0,26 MB használható gyűrűt hagy — ami túl kicsi egy többkamerás adatcsomaghoz —, így a tömbpanel `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` hibát jelez, és blokkolja a csatlakozást.**256**a gyűrű mérete nagy (**~13,5 MB, a laboratóriumi 10 GbE-s gazdagépen mért érték**), ami valódi tartalékot biztosít az RX-csatornának a többkamerás GVSP-burstokhoz. (Hogy egy adott konfiguráció valóban *csatlakozik-e*, azt két ellenőrzés dönti el — a **drain-aware**beengedési ellenőrzés és az**aggregált túljegyzés** ellenőrzés – nem pedig egy nyers burst-vs-gyűrű összehasonlítás; lásd [Tömb fps és burst modell](#array-fps--burst-model).) |
| **Fogadó URB-k**— kulcsszó `PendingReceives` |**64** (max.) | Folyamatban lévő USB-kérésblokkok; a burst-ek felszívásához a fogadó pufferekkel együtt növelendő. |
| **Jumbo Frame** — kulcsszó `*JumboPacket` | **9014** | Szükséges a 9000 bájtos GVSP-csomagokhoz (6× kevesebb csomag/keret, mint 1500-nál). |

> ⚠️ **A hálózati kártya-illesztőprogram frissítése VISSZAÁLLÍTJA ezeket a speciális tulajdonságokat az alapértelmezett értékekre.**Az adapter illesztőprogramjának frissítése vagy cseréje után**újra kell alkalmazni** az `ReceiveBufferLen=256` és az `PendingReceives=64` beállításokat, különben a tárcsa panel újra lezárul, még akkor is, ha „a hardverben semmi sem változott”. Ez a legfőbb oka annak, hogy egy korábban működő rendszer hirtelen nem hajlandó csatlakozni.

Alkalmazza **feljogosított** PowerShell-ből (cserélje ki az adapter nevét, pl. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **Az `lattice network --fix` az USB 10 GbE adaptereket fedi le.** Most már felismeri az adapter típusát, és beállítja a megfelelő receive-ring kulcsszót: `*ReceiveBuffers`→2048 a PCIe hálózati kártyákhoz (Intel I219, stb.), vagy `ReceiveBufferLen`→256 + `PendingReceives`→64 a Realtek **USB** 10 GbE vezérlő esetében (amely nem teszi elérhetővé az `*ReceiveBuffers` értéket). A célértékeket az egyes illesztőprogramok által jelentett maximális értékre (`NumericParameterMaxValue`) korlátozzuk, így soha nem írnak a tartományon kívüli értéket. Futtassa **feljogosított** terminálból; mint minden rendszerleíró adatbázis-alapú beállításnál, a módosítás az adapter újraindítása vagy a rendszer újraindítása után lép hatályba. A fenti manuális `Set-NetAdapterAdvancedProperty` parancsok továbbra is kiváló alternatívát jelentenek — újraindítás nélkül, azonnal hatályba lépnek (az adaptert újra kötik).

### Linux-alapú rendszerek — socket-fogadó pufferek

Linux rendszereken – beleértve az NVIDIA Jetsont is – a kernel socket-fogadó pufferei ugyanolyan fontosak, mint a hálózati kártya (NIC) fogadógyűrűje. A kernel alapértelmezett beállításainál (`net.core.rmem_max` / `net.core.rmem_default` = 212 992 bájt) a kamera GVSP-csomagjainak robbanásszerű áramlása túlcsordítja a stream socketet, mielőtt aChloros
beolvasná azokat. A csomagokat a rendszermag eldobja, miközben a hálózati kártya maga nem jelenti a veszteséget, és a rendszer elveszíti a képkockacsoportokat, majd csökkenti a képkockasebességet. Egy USB 10 GbE adapterhez csatlakoztatott, 4 kamerából álló rendszeren végzett mérések szerint mindkét érték 32 MB-ra történő emelése a csatlakozás után a rendszer képkockasebességét 4,96-ról 7,52 fps-re, a folyamatos nyers rögzítést pedig 4,7-ről 7,6 rögzítés/másodpercre növelte, anélkül, hogy a kernelben csomagok vesztek volna el.

- **A .deb fájl állítja be őket.** A csomag telepítésekor az `/etc/sysctl.d/60-chloros-gige.conf` értékek kerülnek beírásra (mindkettő az `33554432` értékre), és azok azonnal alkalmazásra kerülnek. A már magasabbra beállított értéket nem módosítja.
- **Az `lattice network` ellenőrzi őket.** Ha bármelyik érték alacsonyabb, az *Talált problémák* alatt felsorolja az `socket receive buffers too small for GigE Vision (…)` értéket, és a háttérrendszer naplója ugyanazt a figyelmeztetést tartalmazza az első kamera vagy kameracsoport csatlakozásakor.
- **Kézi emelésük** (a következő újraindításig; a csomag fájlja minden indításkor alkalmazza őket):

  ```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  Az `lattice network --fix` szintén megnöveli és megőrzi őket, de csak akkor, ha a háttérprogram rootként fut. Az a háttérprogram, amely a felhasználóként fut (amelyet az `chloros-backend` vagy [`chloros-backend@<user>`] parancsot használva indított) nem tudja módosítani a kernel beállításait, és ehelyett az `sudo sysctl` sort írja ki.

**Az automatikus adapterbeállítás csak a csatlakoztatott adaptereket érinti.** Az első kamera csatlakozásakor a rootként futó háttérprogram beállítja azokat a vezetékes adaptereket, amelyek vételi gyűrűje kicsi. Most már csak azokat az adaptereket hangolja, amelyekhez kapcsolat van, így egy kihúzott beépített port megtartja az MTU-ját, és a fogadó puffereket nem módosítja. Az `lattice network` már nem jelzi problémaként a kihúzott adapterMTU-ját nem jelzi problémaként. Az `lattice network`, `--fix`, `lattice probe` és az adapter-beállítás kizárólag a vezetékes Ethernet-portokat veszi figyelembe, nem pedig a Jetson CAN-buszát (`can0`) vagy az USB-eszközmódú kapcsolatokat (`usb0`, `usb1`), amelyeket a Jetson a gazdaszámítógép számára biztosít; a kamerákhoz használt USB-Ethernet-adapter viszont továbbra is számít.

### Hálózati alapismeretek (minden LATTICE-kapcsolat)

- **Címzés:** link-local `169.254.0.0/16` (GigE Vision LLA). A gazdaszámítógép statikus `169.254.x.x/16` címet kap; a kamerák és a DAQ-E ugyanebben a tartományban önállóan osztják ki a címeket. Nincs szükség DHCP-re/átjáróra.
- **Csomagméret:**előnyben részesítjük a jumbo (9000) méretet, de hagyjuk, hogy az automatikus felderítés találja meg – a GVSP-próbával átlép a kamera 1500 bájtos ICMP-korlátján, így ott állapodik meg a jumbo méretnél, ahol a kábel valóban továbbítja.**Minden kamerát egyszer mérnek meg háttérszerver-munkamenetenként, majd újra, ha a kapcsolata megváltozik (másik adapter, másik kapcsolatsebesség, vagy a kapcsolat megszakadása, például egy kapcsoló cseréje miatt); az újracsatlakozáskor az eredményt újra felhasználják. Ha a kapcsoló mögött változás történt, de a kapcsoló továbbra is működik, indítsa újra a háttérszervert az újraméréshez.**Az első csatlakozás ennél az ellenőrzésnél esetenként néhány percet is igénybe vehet, és a napló ezt jelzi, ha így történik. Csak akkor állítsd be a `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` értéket, ha jobban tudod, mint a szonda, és inkább a parancsonkénti beállítást részesítsd előnyben az állandóval szemben: a beállítás kihagyja a szondát, így ha az útvonal valójában nem képes 9000-et továbbítani,**minden** rögzítés időtúllépést eredményez az `SC_ERR_TIMEOUT -1011` használatával (lásd [Környezeti változók](#environment-variables)).
- **Az RX-gyűrű mérete az `ReceiveBufferLen` értékével arányosan változik:**az alapértelmezett `32` értéknél a használható gyűrű mérete ~0,26 MB (túl kicsi bármilyen többkamerás sorozatfelvételhez); a maximális `256` értéknél nagy (~13,5 MB, a laboratóriumi 10GbE-s gazdagépen mérve), ami valódi tartalékot biztosít. Hogy egy konfiguráció csatlakozik-e, azt a sávszélesség-figyelembe vevő belépési ellenőrzés**és** az alábbi összesített túljegyzés-ellenőrzés határozza meg — nem pedig a nyers sorozat és a gyűrű közötti összehasonlítás.

### Array fps és burst modell

Hogyan kell értelmezni az Array Settings panelt (és az `lattice analyze-array`-et / az SDK `analyze_array_network`-ét):

- **A burst-értékeket kameránként, az egyes kamerák tényleges pixelformátumában összegzik.**A Mono**M3M**kamerák**Mono12 (2 B/px)**formátumban streamelnek, vagy az `--pixel-format` használatával a mélységi adatok Mono formátumában (az `BayerRG8` az `Mono8`, 1 B/px); az**M3C**Bayer kamerák 8- vagy 12-bites adatfolyamot adnak (az M3C firmware csendben BayerRG12-t ad ki, még akkor is, ha BayerRG8-at kérnek). Tehát egy 4 kamerás, teljes felbontású képkocka mérete**~12,6 MB, ha mind 8 bites, de ~25 MB, ha három 12 bites mono kamera van**. A vetítés a modell alapján (azonosító gyorsítótár) határozza meg az egyes kamerák formátumát, így a sorozatfelvétel pontosan megfelel annak, amit a vezeték ténylegesen továbbít — nem pedig egy általános BayerRG8-feltételezésnek.
- **Egy USB-Ethernet-adapter sebessége 200 MB/s-ra van korlátozva, függetlenül a névjelzésétől.** Az a hatékonysági táblázat, amely a kapcsolati sebességet tartós értékre konvertálja, a PCIe-ből származik; egy USB-hálózati kártya ugyan az *Ethernet* kapcsolati sebességét hirdeti, de az USB-busz és az illesztőprogram korlátozza. Egy USB 10 GbE dongle ~1063 MB/s „tartós” sebességet – egy olyan értéket, amelyet soha nem vizsgáltak –, és az ebből adódó sebességszabályozás a keretek 6–18 %-át megrongálta, miközben továbbra is megfelelő cél-fps-értéket jelentett. Az USB-n keresztül csatlakoztatott hálózati kártyák sebességét most már abszolút értékben korlátozzák: **200 MB/s**-ra Windows-en és az NVIDIA Jetsonon, **400 MB/s**-ra Linux x86_64 rendszeren (Linux alatt egy USB 2.0-porton csatlakoztatott adapter **30 MB/s**-ra van korlátozva, és a műszaki adatok szerint azt USB 3-as porthoz kell csatlakoztatni) (a korlátot a gazdagép USB-busza és illesztőprogramja határozza meg, így nem arányos a névleges értékkel; egy USB 1 GbE adapter ~80 MB/s sebességet ér el, és ez nem érinti). Linux rendszeren az adaptert a sysfs azonosítja (visszaesve az `ethtool -i`-ről az `bus-info`-re). Minden adatot megmértünk: Windows rendszeren ugyanaz a 4 kamerás rendszer 200 MB/s felett a képkockák 6–10 %-át rontotta el; Linux x86_64 rendszeren ugyanaz az adapter 480 MB/s-ig minden sebességhatárnál hibamentesen működött (273 MB/s átviteli sebességet ért el, ami a kamerák saját korlátja). NVIDIA Jetson rendszeren ugyanaz az adapter csak 200 MB/s sebességnél működött hibamentesen, e felett pedig időnként képkockákat veszített. Az `wire_ceiling_source` a képességi leírásban ezt szóban is megemlíti, az `nic_is_usb` pedig jelzi. Mindkét esetben felülírható az `--wire-ceiling-mbps` parancs segítségével felülírható.
- **A belépés a drain-t veszi figyelembe, nem a teljes burst-vs-ring arányt.** Egyidejű burst esetén csak a *tranziens backlog* = `max(0, Σ per-cam arrival − host drain) × emit_window`-nek kell megfelelnie, nem az egész burstnak. Gyors gazdagép / lassú kamera felépítésű hálózat esetén (egy **PCIe**10G-s gazdagép + 4× 1 GbE-s cam: érkezés ≈ 320 MB/s, kiürítés ≈ 1063 MB/s) a gazdagép gyorsabban üríti ki az adatokat, mint ahogy a kamerák megtöltik, a backlog ≈ 0, így a teljes felbontású szimulált kibocsátás**megengedett**, annak ellenére, hogy a 25 MB-os burst meghaladja a 13,5 MB-os gyűrűt. Ha ugyanezt a négy kamerát egy**USB** 10GbE-adapter mögé helyezzük Windows rendszeren, az adatkiadás 200 MB/s lesz, nem pedig 1063 — az érkező adatmennyiség meghaladja ezt, és a veszteség nem alacsonyabb képkockasebességként, hanem sérült képkockákként jelenik meg. Egy 1 GbE-s gazdagépen a kamerák 31,25 MB/s-os DLThr alsó határa miatt az érkező adatmennyiség meghaladja a kiadási sebességet → a rendszer helyesen **blokkol**(**ez**a blokkolási osztály esetében csökkentse a ROI-t, vagy használjon ≥ 2-es binninget). Az átvitel engedélyezése a**két** csatlakozási kapu egyike – a másik az alábbi összesített túljegyzés-ellenőrzés.
- **A tervezett fps a leglassabb kamera ciklusa.**Egyidejű kibocsátás esetén minden kamera egyszerre küldi el a képkockáját, így a tömb a leglassabb tagjának érzékelő-leolvasási ideje és kibocsátási ideje közül a hosszabbik szerint ciklikus (képkocka-bájtok ÷ a kamera**hozzáférési kapcsolata**, 1 GbE ≈ 80 MB/s, vagy a gazdagép vezetékes sávszélességéből rá eső rész, ha az kisebb). Minden expozíció átfedi az előző képkocka kiolvasását, így az expozíció nem kerül be a ciklusba: az automatikus expozíció kitölti a sebesség által megadott periódust, majd növeli az erősítést, és egy sötét jelenet már nem csökkenti a sebességet (az 1.2.0-s verzióban még csökkentette). Csak egy, a periódusnál hosszabb kézi expozíció lassítja a kioldást (lásd [Expozíció egy rendszerelemnél](#exposure-on-an-array-member)). A kamerák hozzáadása nem lassítja a tömböt, amíg a vezetéki sávszélességük nem csökken. Soros kibocsátás esetén (`slip-emit-and-capture`) a kamerák kibocsátási idejei összeadódnak, és minden elem expozíciója beleszámít a lépcsőzésbe. Teljes szenzoros, 12 bites mérése egy 9 kamerás tömbön, PCIe 10 GbE gazdagéppel: 10,7 fps, vagy 14,0 fps [tömörített átvitellel](#packed-12-bit-transport---packed).
- **Folyamatos rögzítés után a kivetítés megmutatja, hogy milyen sebességet tartott fenn.** Egy 30 másodperces vagy annál hosszabb, automatikus sebességgel végzett rögzítés rögzíti azt a sebességet, amelyen az adott kamerarendszer-méret mellett ezen a számítógépen stabilizálódott; a sebességszabályozó gyorsabb sebességre irányuló rövid kísérleteit a rendszer kihagyja. Ettől kezdve az `array-connect` sor és a Tömbbeállítások panel ezt a sebességet jelzi, ha az alacsonyabb a modellben megadottnál, feltüntetve, mi korlátozta: `clock-bound` (a tömb a kialakult sebességgel futott, a rögzítő hurok a trigger órajelre várt), `loop-bound` (a rögzítő huroknak nem maradt ideje), `capture-bound` (a rögzítés lemaradt és csoportokat hagyott ki) vagy `measured` (nem derül ki, melyikről van szó). A modell adatai az `wire_fps_max`-ben találhatók. A tömb továbbra is a hálózat által biztosított értékig emelkedik.
- **A tömb bármilyen hálózaton működik — a képkockasebesség határozza meg a teljesítményt.**Minden kamera egyenlő részesedést kap az**ütközésbiztos vezeték-felső határértékből**(`sustained × sim_emit_factor`), a GevSCPD-vel szinkronban, és mindegyikük továbbra is ugyanazon a triggeren exponál. Lassú vezetéken a részesedés kicsi, és a képkockasebesség is ezzel arányosan csökken: 8 teljes felbontású, 12 bites kamera 1 GbE-n, jumbo keretek nélkül, egyenként 52 / 8 = 6,5 MB/s-ot kap, ami körülbelül**1 fps**(csomagolva kb. 1,3 fps), míg egy 10G-s gazdagép 11–14 fps-sel futtatja őket. Ha a kamerákat nem lehet ilyen lassan szinkronizálni – a connect parancs kiolvassa az egyes kamerák GevSCPD-tartományát, hogy ezt kiderítse –, a rendszer átvált**soros**kibocsátásra (`slip-emit-and-capture`) üzemmódra vált: a kamerák felváltva, teljes kapcsolati sebességgel használják a vonalat, egymás után indítva őket, így minden keret megérkezik, de az elemek**nem kerülnek egyszerre ki**. Az `analyze-array` és a connect kimenet nevezi meg az üzemmódot; az `--force-tier sim-capture-sim-emit` beállítással elkerülhető a visszaesés. A képkocka mérete nem játszik szerepet ebben a megosztásban, így**a binning és a kisebb ROI növeli a képkockasebességet, de nem befolyásolja, hogy egy sorozat elfér-e**. Csak az a vezeték, amely még soros módon sem képes továbbítani a tömböt,**túlterhelt**— pl. sokkal több kamera van, mint amennyit egy vezeték képes kezelni, vagy sok más kamera is streamel rajta —, és ebben az esetben a tünet a GVSP-csomagvesztés, nem pedig az alacsonyabb sebesség: Az `analyze-array` nullára állítja az elérhető képkockasebesség-értékeket, és kiírja az `** OVER-SUBSCRIBED**` üzenetet; a csatlakozás hangos figyelmeztetéssel folytatódik, majd egy `pin_resolution: true` (szigorú mód) kóddal rendelkező HTTP-kapcsolat **elutasítja**. A megoldások: kevesebb kamera, végpontok közötti jumbo keretek, vagy gyorsabb hálózati kártya. Az `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` a tesztelés során az elutasítást hangos figyelmeztetésre csökkenti — lásd [Környezeti változók](#environment-variables).

### A tömb állapota — melyik alrendszer veszít kereteket

Egy csatlakoztatott tömb `GET /api/camera/array/<array_id>/capability` értéke egy élő
`health` blokkot hordoz, amelyet egy gördülő **10 másodperces** ablakban újraértékelnek. A keretvesztést
két, egymással ellentétes javítást igénylő okra bontja, ahelyett, hogy egyetlen „hiányos”
arányt jelentené, amelyik egyik okot sem nevezi meg:

| Mező | Mit jelent | Melyik alrendszer |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (soros portonként) | A keret **megérkezett, de szerkezetileg hibás volt**— GVSP-csomagvesztés. |**Hálózat**: vezeték-kapacitás, ütemezés, NIC RX-gyűrű, MTU |
| `never_arrived_rate_pct` (soronként) | A keret **egyáltalán nem érkezett meg**— a kamera nem indult el, vagy semmi sem hagyta el azt. |**Kiváltás / szinkronizálás**: M8 kábel, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Az egyes kamerák legrosszabb aránya. | — |
| `per_cam_rate_pct` | Kameránkénti összesített hiányossági arány (mindkét ok együtt). | — |
| `stable_for_seconds` | Az egyes kamerák 0,01 % alatti tartózkodási ideje. | — |

5 % felett a háttérrendszer egy `[array-health <id>] WARN` sort rögzít a felosztás megnevezésével — az
első túllépéskor, a súlyossági sáv változásakor, percenként, amíg fennáll, és egyszer, amikor
megszűnik. A sérült fele az `[gvsp-corrupt <SN>]` sort írja a háttérnaplófájlba
kameránként és okonként az első találatkor, majd 60 másodpercenként összesítve; az alkalmazás naplópaneljén ehelyett az
`[array-health <id>]` sort. Minden értékelés továbbra is bekerül a háttér naplófájlba;
a számlálók minden puffer esetén továbbhaladnak, függetlenül attól, hogy mi kerül kinyomtatásra.

Ugyanez a rekord jelzi azt a számot, amelyhez az egész allokáció kapcsolódik:

| Mező | Mit jelent |
| --- | --- |
| `wire_ceiling_mbps` | A gazdagép érvényben lévő, fenntartott hálózati sávszélessége, MB/s. |
| `wire_ceiling_source` | Honnan származik ez az érték, szavakkal kifejezve — pl. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` (Linux x86_64 rendszeren: `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`) vagy `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, ha az `--wire-ceiling-mbps` (vagy a grafikus felület **Wire Budget** mezője) így állítja be. |
| `nic_is_usb` | `true` USB Ethernet-adapter esetén — lásd a fenti USB-korlátot. |

**Értelmezés:** ha az `gvsp_corrupt_rate_pct` értéke nem nulla, az `never_arrived_rate_pct` pedig 0
azt jelenti, hogy a triggerelés és a kábelszinkronizálás tökéletes, és a veszteség 100%-a a hálózati
útvonalon keletkezik — csökkentse az `--wire-ceiling-mbps` értéket, majd csatlakozzon újra. Az ellenkező mintázat inkább a
szinkronkábelre vagy a triggervezetékre utal.

> **Az `--target-fps` nem a sérült keretek szabályozója.** A GevSCPD ütemezése
> a csatlakozáskor egyszer kerül beállításra, így a triggerfrekvencia csökkentése a kitöltési arányt változtatja meg, nem pedig a
> egyidejű kibocsátási burst-frekvenciát. Egy 5-szeres igénycsökkentés nem hozott javulást;
> a vezetéki felső határ 240-ről 200 MB/s-ra történő csökkentése ugyanazon a berendezésen a 10,4 %-os
> sérültségi arányt 0,00 %-ra csökkentette.

> **A futó tömb nem tudja magát újratervezni a LATTICE (M3C / M3M) firmware-en.** Ez
> önmagában nem tudja kijavítani a hibát; válassza le, majd csatlakoztassa újra, hogy a csatlakozási idő-választó
> az új felső határ értékével újratervezhesse a rendszert.

### Tünet → megoldás

| Tünet (Tömbbeállítások / csatlakozás / `analyze_array_network`) | Ok | Megoldás |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` visszaáll 32-re (jellemzően illesztőprogram-frissítés után) | Állítsa be az `ReceiveBufferLen`→256, `PendingReceives`→64; nyissa meg újra a panelt (indítsa újra a háttérprogramot, ha az a régi gyűrűméretet tárolta a gyorsítótárban) |
| Az újraindítás/leállítás elakad; később `Invalid class` WMI-hibák, a hálózati kártya nem aktiválódik, az USB-meghajtók hiányoznak | Régi, 2016-os Realtek USB 10GbE illesztőprogram → BSOD `0x9F` → kényszerű kikapcsolások | Frissítse az adapter illesztőprogramját ≥ v10.67 (2026), majd a fenti vételi gyűrű beállítások újbóli alkalmazása |
| A tömb teljes szenzorral csatlakozik, de alacsony a képkockasebesség | Az „Auto” beállítás megtartja a teljes szenzort, és hagyja, hogy a sebesség a vezeték által támogatott szintre csökkenjen | Válasszon az egyik gyorsabb opciót a csatlakozási kimeneti listából (`--binning 2` / `4`, vagy egy kisebb `--width`/`--height`) közül, engedélyezze a jumbo kereteket, vagy frissítse a kapcsolatot |
| Az Array megfelelő célképkockasebességet jelez, de annak csak egy töredékét biztosítja; `health.gvsp_corrupt_rate_pct` nem nulla, `never_arrived_rate_pct` 0 | A gazdagép becsült sávszélessége magasabb, mint amit ténylegesen bír (jellemző USB Ethernet-adapter, keskeny PCIe sáv vagy megosztott hálózati struktúra esetén) | Csatlakozzon újra alacsonyabb `--wire-ceiling-mbps` értékkel, és ellenőrizze újra az állapotblokkot. **Nem** `--target-fps` — A GevSCPD sebességszabályozása a csatlakozáskor rögzül |
| Linux: egy tömb keretcsoportokat veszít, és csökkenti a keretsebességét, miközben a hálózati kártya nem jelenti a kieséseket; az `lattice network` az `socket receive buffers too small` értéket jelzi | A rendszermag socket-fogadó pufferei alapértelmezett értékeken vannak, ezért a kamera csomagjai a rendszermagban kiesnek | Növelje az `net.core.rmem_max` / `rmem_default` értéket 32 MB-ra — lásd [Linux-gépek](#linux-hosts--socket-receive-buffers). A .deb telepítéskor ezt elvégzi |
| A kamerák hiányoznak a közzétett csoportokból; az `health.never_arrived_rate_pct` értéke nem nulla, `gvsp_corrupt_rate_pct` 0 | Indító-/szinkronizáló útvonal — a kamerák nem aktiválódnak, nem hálózati probléma | Ellenőrizze az M8 szinkronkábelet és az `--line` értéket; győződjön meg róla, hogy minden tag be van kapcsolva (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` túllépése az `analyze-array`-ben, vagy csatlakozási elutasítás szigorú módban (`pin_resolution`; `array over-subscribes the wire`) | A kábel nem képes továbbítani a kamerák által elérhető bármely adatátviteli sebesség mellett a tömböt, sem egyszerre, sem felváltva — sokkal több kamera van, mint amennyit a kábel meg tud osztani, vagy más kamerák is streamelnek rajta | Kevesebb kamera, végpontok közötti jumbo keretek, vagy gyorsabb hálózati kártya. **Az ROI/binning NEM segít** (a megosztás keretmérettől független). Az `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` felülírja a tesztpadon (elfogadja a csomagvesztést) |
| A kamerasor az `slip-emit-and-capture`-ben (soros) csatlakozik egy lassú hálózaton | A kamerák nem állíthatók elég lassú ütemre ahhoz, hogy egyszerre osztozzanak a kábelen, ezért felváltva működnek; a képkockák nem kerülnek rögzítésre ugyanabban a pillanatban | A jumbo keretek vagy egy gyorsabb hálózati kártya helyreállítja az egyidejű rögzítést; az `--force-tier sim-capture-sim-emit` ragaszkodik hozzá (és ehelyett csomagvesztést szenved) |

---

## `chloros-cli daq`

Spektrális érzékelő parancsok. Az `pool-*` parancsok vékony HTTP-kliensek, amelyek a háttérrendszer állandó poolján keresztül vezérlik az érzékelőt, ezért futó háttérrendszerre van szükségük (lásd [A háttérrendszer elindítása](#start-the-backend)). A háttérrendszer kezeli az adatátvitelt, így a CLI- és az SDK-parancsfájlok egy aktív kezelőt osztanak meg, ahelyett, hogy a soros portért versengnének. Az `pool-discover` az egyetlen módja a DAQ-M BLE MAC-címének megismerésének.

> **`chloros-cli daq --help`** (vagy az `-h`, illetve az `daq help`) felsorolja az `pool-*` alparancsokat (Linux rendszeren a build által beépített közvetlen hardverparancsokkal együtt), az `chloros-cli daq <subcommand> --help` (vagy `daq help <subcommand>`) pedig az egyes parancsok opcióit mutatja be. Ezek kiterjednek a csatlakozásra, stream, kalibrált `.daq` fájlok rögzítése, kapacitásprofilok cseréje és a DAQ kamerához való csatlakoztatása, valamint DAQ-E esetén a hálózati beállítások, a PTP, az újraindítás és a firmware-frissítések. A DAQ-ot Python ból is vezérelhetjük az `chloros_sdk.connect_daq_sensor()` paranccsal, amely ugyanazt a közös elérési utat használja.
>
> **Használja az `pool-*` parancsokat minden platformon.** Ezek Windows, Linux és Jetson rendszereken egyformán viselkednek. Windows rendszeren az `daq discover` és az `daq list` az `pool-discover` aliasai, és minden más `daq` parancs, amely nem az `pool-*`-től eltérő parancsok az `1` parancsot az `DAQ direct-hardware commands require the full daq package` parancsra váltják. Linux on és Jetsonon az `daq discover`, az `daq list` és az egyéb, nem `pool-*` `daq` parancsok a DAQ eszköz közvetlenhardverparancsai: az érzékelőt a CLI saját folyamatában nyitják meg, a háttérrendszer poolján kívül (tehát nem egy olyan érzékelőn, amelyet a háttérrendszer, a grafikus felület vagy egy szkript tart fenn), és ott fogadják az `daq discover`, `--only` és `--timeout` parancsokat, de nem az `--json`-et. Ez a referencia kizárólag az `pool-*` parancsokat dokumentálja.

### DAQ-érzékelő első csatlakozási munkafolyamat

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### `pool-*` referencia

| Alparancs | Cél |
| --- | --- |
| `daq pool-connect` (smart-detect) | Érzékelő megnyitása a háttérpoolban. |
| `daq pool-connect --port PORT` (`-p`) | DAQ-U egy adott soros porton. |
| `daq pool-connect --ble` | DAQ-M BLE-n keresztül, automatikus MAC-címkereséssel. |
| `daq pool-connect --mac MAC` (`-m`) | DAQ-M BLE-n keresztül ismert MAC-címen (magában foglalja az `--ble` parancsot). |
| `daq pool-connect --eth-host HOST` | DAQ-E Etherneten keresztül ismert gazdagépnévvel vagy IP-címmel (magában foglalja az `--eth`-et). |
| `daq pool-connect --eth` | DAQ-E Ethernet-en keresztül, a gazdagép automatikus felismerésével (mDNS + ARP tartalék; Windows és Linux rendszereken üres ARP-gyorsítótárból is működik). |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | Integrációs ablak / AE állapot beállítása. |
| `daq pool-connect --no-stream` | Csatlakozás, de az adatátvitel még nem indul el (folytatás az `pool-stream --start` paranccsal). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Cap-korrekciós profil. A háttérrendszer alapértelmezett értéke az `sunshine_cosine`; az `none` egy csupasz érzékelő. A [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor) figyelmen kívül hagyja (rögzített diffúzor). **Az érzékelők fedél nélkül kerülnek kiszállításra, ezért nevezze el az Ön által felszerelt fedelet:** az alapértelmezett érték feltételezettként kerül rögzítésre (`cap_id_source=auto_default`), és a feldolgozás figyelmeztet rá, mivel egy csupasz DAQ-U a napfény-korrekció mellett ~30-szor magasabb értéket jelez. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Minden közlekedési eszközön keresse meg azokat az érzékelőket, amelyekhez csatlakozhatna, anélkül, hogy csatlakozna hozzájuk. **Így lehet megtalálni egy DAQ-M BLE MAC-címét.** Az `--timeout` parancs transzportonkénti (alapértelmezett: 5 másodperc, 1–20). A poolban már megnyitott érzékelők nem szerepelnek a listában — egy csatlakoztatott DAQ-M leállítja a hirdetést —, ezért azokhoz használja az `pool-list` parancsot. Az `usb` kizárólag az USB-soros eszközöket sorolja fel: a számítógép saját soros portjai (`COM1`, Linux esetén `/dev/ttyS*`) nem lehetnek érzékelők, ezért kimaradnak. |
| `daq pool-list` | A háttérpoolban található összes érzékelő megjelenítése. |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | Kamera vagy tömb hozzárendelése egy DAQ-hoz, ahogyan azt a grafikus felület fényérzékelő-kiválasztója is teszi: a rögzítések felhasználják, összehangolják és elmentik a DAQ mérési eredményeit. Az `SENSOR` egy érzékelő-azonosító az `pool-list`-ből, vagy a DAQ elérhetőségének címe. Az `--clear` (nincs `SENSOR`) megszünteti a kapcsolást, és a rögzítések visszatérnek a legfrissebb DAQ-ra. A kapcsolódás addig tart, amíg a háttérprogram újraindul, vagy amíg az adott kamera, sorozat vagy DAQ le nem válik: egy leválás (itt az SDK-ból vagy az alkalmazásból) megszünteti a leválasztott hardver minden kapcsolódását, függetlenül attól, ki állította be, a sorozat elemeinek kapcsolódásait pedig a sorozattal együtt. A kamerának csatlakoztatva kell lennie, a megnyitott projektben kell szerepelnie, vagy korábban megnyitottnak kell lennie ezen a számítógépen, és a sorozatnak is csatlakoztatva kell lennie vagy szerepelnie kell a megnyitott projektben; bármely más sorozatszámot vagy azonosítót a rendszer név alapján elutasít, és semmi sem lesz összekapcsolva. |
| `daq pool-disconnect {--sensor-id ID \| --all}` | Felszabadítás — függetlenül attól, hogy ki nyitotta meg az érzékelőt, így felszabadítja azt az érzékelőt is, amelyet egy összeomlott szkript hagyott a poolban (az azon folyó felvételeket előbb befejezi). A poolban nem szereplő érzékelőt nincs mit felszabadítani: az `ID: already released (not in the pool)` üzenetet írja ki, majd kilép (`0`); ha a háttérprogram elutasította vagy nem erősítette meg a felszabadítást, akkor az `1` üzenettel lép ki. SDK: `DAQSensorSession.force_disconnect()`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | A legutóbbi N spektrumképkocka (alapértelmezés szerint 1). |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | A streamelés folytatása / szüneteltetése. |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | .daq felvétel indítása / leállítása. Az `-o` alapértelmezés szerint az `~/Documents/DAQ Live View` a háttérszolgáltató számítógépén (a relatív elérési út a parancs futtatásának helye szerinti mappában található); `--duration` (`-d`, alapértelmezett: `0` = egészen `--stop`-ig). Az `--confirm-after S` figyelmeztet, ha a kezdés után S másodperccel még nem érkezett be olvasási érték (alapértelmezett: 20; 0 kikapcsolja); az `--duration` esetén az `--heartbeat S` S másodpercenként kinyomtatja az értékek számát, és figyelmeztet, ha az nem növekszik tovább (alapértelmezett: 30; 0 kikapcsolja). Egy DAQ-E-n, amely elhallgat, ugyanabban a fájlban folytatódik, miután a háttérrendszer újra csatlakozott hozzá, és az `--stop` felsorolja a hiányzó értékeket ([A DAQ-E, amely leáll az adatátvitellel](#a-daq-e-that-stops-delivering)). A Ctrl+C gombnyomással a `--duration` felvétel idő előtt befejeződik, és erről jelentést kapunk; `--duration` nélkül a felvétel a Ctrl+C után is tovább fut (kilépés: `130`), egészen az `--stop`-ig. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | A kondenzátor-korrekciós profil cseréje futásidőben. Ehhez az `--cap-id` szükséges; az `--cap-id none` eltávolítja a kondenzátor-korrekciót (csupasz érzékelő). A DAQ-E-S megtartja az `as_recorded` beállítást, függetlenül attól, hogy mi van beállítva. |
| `daq pool-enable-ptp --sensor-id ID` | Kapcsolja vissza a PTP-t egy olyan DAQ-E esetében, amelynek PTP-je ki van kapcsolva. A beállítás el van mentve a készüléken, így a kikapcsolt egység kikapcsolt állapotban marad – időbélyegei nem PTP-szinkronizáltak –, amíg ezt a parancsot nem futtatják. Az `pool-list` jelzi az ilyen készüléket (`!! PTP is switched OFF`), és kinyomtatja számára ezt a parancsot. A Chloros soha nem kapcsolja vissza önállóan a készüléket, mert a kikapcsolt állapot szándékos is lehet. |
| `daq pool-info --sensor-id ID [--json]` | A DAQ-E saját beállításai: a firmware-je (és hogy ez a Chloros-kiadás tartalmaz-e újabbat), a címe és annak megszerzésének módja (DHCP / link-local / statikus), az érzékelő által futtatásig mentett hálózati beállítás (amelyet az `--no-reboot` parancs futtatásával mentettek), a bekapcsoláskori adatfolyam, valamint a PTP. Ha az érzékelő adatfolyama nem éri el ezt a számítógépet, a program új kapcsolat létrehozásakor kiolvassa az érzékelő adatait, és erről tájékoztatást ad. |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | DAQ-E beállítása DHCP-re (`--auto`) vagy statikus címre (`--mask` alapértelmezés szerint `255.255.255.0`). Az érzékelő újraindul a beállítások alkalmazásához, és a korábbi beállításokkal csatlakozik újra, erre legfeljebb `--timeout` másodpercig (alapértelmezés szerint 45) vár; `--no-reboot` csak elmenti a beállítást. Lásd [DAQ-E eszközbeállítások](#daq-e-device-settings). |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | Meghatározza, hogy a DAQ-E bekapcsoláskor elindítja-e az adatfolyamot, ha nincs csatlakoztatva szoftver (az érzékelőn mentve). A Chloros mindkét esetben elindítja az adatfolyamot. |
| `daq pool-reboot --sensor-id ID [--timeout S]` | A DAQ-E újraindítása és újracsatlakoztatása a korábbi beállításokkal (legfeljebb `--timeout` ideig vár, alapértelmezés szerint 45 másodperc). |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | A jelen Chloros-kiadáshoz tartozó DAQ-E firmware-t írja fel egy régebbi verziót futtató érzékelőre, majd csatlakoztassa újra (körülbelül egy perc; ne kapcsolja ki; `--timeout` alapértelmezés szerint 45 másodperc, amíg újra bekapcsol). A már naprakész érzékelőt nem érinti a művelet. |

### A spektrumküldést leállító DAQ-E

A háttérrendszer újra csatlakoztatja azt a medencében lévő DAQ-E-t, amely 20 másodpercig nem küld spektrumot. Ezt az érzékelőt tápláló kapcsoló áramellátásának kikapcsolásával és újraindításával érik el: az érzékelő ezzel újraindul, és addig nem továbbít adatokat, amíg erre utasítást nem kap (kivéve, ha az `pool-set-auto-stream --on` beállítás van érvényben).

- **Először ellenőrzi, hogy ugyanarról az érzékelőről van-e szó.** Az érzékelő korábbi címén, semmit sem változtatva, lekérdezi az érzékelő nevét, MAC-címét és sorozatszámát. Az érzékelő `daq-e-<id>.local` nevén is lekérdezi, de csak akkor bízik meg a válaszban, ha az az érzékelő saját hálózati adapterétől (MAC-címétől) érkezik. Semmi más, ami válaszol, nem kerül újra csatlakoztatásra, és nem írnak rá.
- **A munkamenet beállításai szerint állítja vissza a kapcsolatot:** az integrációs időt, a képkockák átlagolását, az automatikus expozíciót és a cap-profilt úgy, ahogy voltak, nem pedig az érzékelő bekapcsoláskori alapértelmezett értékeivel. Azokat az érzékelőket, amelyeknél a PTP be volt kapcsolva, de kikapcsolt állapotban tértek vissza, újra bekapcsolja. Ezután újraindítja az adatfolyamot és a számítógép ehhez tartozó vevőjét.
- **Egyre hosszabb időközönként próbálkozik újra:** azonnal, majd 5, 10, 20 és 40 másodperc múlva, végül 60 másodpercenként, amíg az érzékelő nem elérhető. Minden kísérlet során csak azt kérdezi az érzékelőtől, hogy melyik érzékelőről van szó. Azokat az érzékelőket, amelyek válaszolnak, de három újracsatlakozási kísérlet után sem küldenek semmit, békén hagyják, és jelentik (`auto-reconnect GAVE UP`). Az `pool-stream --start` újra megpróbálja.
- **Az `pool-list` jelzi az aktuális állapotot.** Amíg az érzékelő nem ad választ, az `STREAM FAULT` sorban látható, hogy a kapcsolat-újraépítés milyen fázisban van. A sor az első spektrummal törlődik, majd az érzékelő értéke `reconnected automatically N time(s)` lesz.
- **A felvétel ugyanazon az `.daq` sorban folytatódik.** A fájl minden szünetet két sorral jelöl: `event_type` 5-öt a szünet előtti utolsó spektrumnál, és 6-ot az azt követő elsőnél, mindkettő csak időbélyeggel. `pool-record --duration` az `!! STREAM LOST …` értéket írja ki, amíg az érzékelő nincs jelen, és az `stream resumed after …` értéket, amikor visszatér. Az `pool-record --stop` felsorolja az egyes szüneteket, és figyelmeztet, ahogyan azt egy üres fájl esetében (ha egy felvétel nem tartalmaz mérési értéket, akkor az `1`-re vált).
- **Egy felvétel egy kalibrációt tart meg.** Ha a felvétel futása közben az érzékelő más kalibrációval tér vissza, a felvétel hátralévő része megtartja a kezdeti kalibrációt, és a leállítási figyelmeztetés ezt jelzi. `pool-disconnect` + `pool-connect` az újat veszi át.
- **A parancsai az irányadók.** Az a szenzor, amelyet `pool-disconnect` parancsával leállít, vagy `pool-stream --stop` parancsával szüneteltet, soha nem csatlakozik újra, még akkor sem, ha az újracsatlakozás már folyamatban van.
- **Az `pool-info` parancs egy nem válaszoló érzékelőn** új kapcsolaton olvassa be az érzékelőt, és hozzáadja az `!! stream NOT reaching this computer` parancsot. A nem válaszoló érzékelőt azonnal jelenti. Az `pool-list` parancs már nem jeleníti meg a PTP állapotot olyan érzékelő esetében, amely már nem válaszol.
- A DAQ-U és a DAQ-M nem tartoznak a javítás hatálya alá. A DAQ-M önállóan újracsatlakozik Bluetooth-on keresztül, az USB-ről leválasztott DAQ-U-hoz pedig ismét szükség van az `pool-connect` kódra.

### DAQ-E-S — rögzített diffúzor és dőlésérzékelő

A **DAQ-E-S** egy DAQ-E, amely két, a szoftver szempontjából fontos különbséggel rendelkezik. Minden egyéb – felismerés, csatlakozás, PTP, rögzítés, az alábbi eszközbeállítások – pontosan úgy működik, mint a DAQ-E esetében.**Diffúzorja rögzített, ezért nincs szükség sapkakorrekcióra.** A Sunshine-típusú diffúzor beépített, és már a készüléken volt, amikor a gyári kalibrálását elvégezték, így a kalibrálás már a valódi lefelé irányuló besugárzást jelzi. Az Chloros a sapkáját `as_recorded`-re (profil nélkül) állítja be, függetlenül attól, hogy mit kérnek tőle: a készülék nevére utaló korlátot (`sunshine_cosine`, `fov_*`, `none`) a rendszer figyelmen kívül hagyja és üzenetet jelenít meg, a felvételei pedig az `cap_id_source=model`, így a feltételezett korlát figyelmeztetés soha nem lép működésbe. Ne alkalmazzon korlátot az adataira kézzel sem: a tetején lévő `sunshine_cosine` körülbelül 11-szer magasabb értéket jelezne.

**3-tengelyes gyorsulásmérővel rendelkezik.** A napfény-szóró koszinusz-válasza csak akkor érvényes, ha vízszintes helyzetben van, így egy megdőlt érzékelő hibás besugárzási értéket jelent – és a visszaverődés is átveszi a hibát –, egyéb tünet nélkül. Minden DAQ-E-S-leolvasás ezért tartalmazza a készülék helyzetét:

| Mező | Jelentés |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Szögek fokban. `None`, ha a firmware a szöget meghatározhatatlannak jelöli (szabad esés, holt tengely). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Gyorsulás milli-g-ben. Ha az `mag_mg` értéke messze eltér a ~1000-től, az azt jelenti, hogy a készülék mozgásban volt, és a szögek nem értelmezhetők az adott mintára vonatkozóan. |
| `cal_applied` | Beállítva: a tengelyek és a szögek a diffúzor koordinátarendszerében vannak (az egység rögzítési kalibrációja érvényes). Törölve: nyers kártyakoordinátarendszer, amely akár ~2,3°-os dőlést is jelezhet, ami nem valós. |
| `sample_age_ms` | A gyorsulásmérő-minta kora a spektrum elküldésének időpontjában. |
| `temp_raw` | Kalibrálatlan chip-hőmérséklet **trend** — csak különbségek, soha nem hőmérő-leolvasás. |

Az `daq pool-latest --json` ezeket az egyes keretek `imu` kulcsai alatt adja vissza (`null` egy DAQ-U, DAQ-M vagy egyszerű DAQ-E esetén `null`). A DAQ-E-S-ről rögzített `.daq` ugyanazokat az értékeket tárolja az `imu_*` oszlopokban (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) oszlopokba, így az olvasó meg tudja különböztetni a felhőt a felborult állványtól. Minden más modellnél ezek az oszlopok üresek.

A helyzetadat-sorozatot külön jelentik: az `daq pool-list` kinyomtatja az `!! ATTITUDE STREAM FAULT (spectra unaffected)` értéket, amíg le van kapcsolva, és azt is, hogy utána hány hibából állt helyre. Egy ott fellépő hiba soha nem jelenti a spektrális felvételben lévő hiányt. Az `daq pool-discover` az egységet `DAQ-E-S` néven sorolja fel (`DAQ-E` néven, ha körülbelül 2 másodpercen belül nem válaszol az azonosítási ellenőrzésre).

### DAQ-E eszközbeállítások

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **Az `--static` az `--gateway` nélkül nem menti el az átjárót** (a korábban beállított átjárót elveti), ami megfelelő egy útválasztó nélküli kamerahálózat esetében.
- **A számítógép hálózatán statikus címnek kell lennie.** A Chloros elutasítja azokat, amelyek alhálózata nem tartalmazza a számítógép címét az érzékelő hálózatán: újraindítás után semmi sem érheti el az érzékelőt, hogy visszaállítsa a beállítást. Csak akkor adja meg az `--force` parancsot, ha a számítógépet átcímzik vagy átirányítják az új alhálózatra.
- **Az újraindítás nem engedélyezett, amíg az érzékelő rögzít.** Először állítsa le a rögzítést (`pool-record --stop`). Az újraindítás előtt elindított rögzítést az újraindítás lezárja, és az `pool-set-network`, `pool-reboot` és `pool-update-firmware` üzenetek egy `Warning:` sorral zárulnak, amely megnevezi a felvétel fájlját (akkor is, ha az érzékelő ezután nem tér vissza).
- **Újraindítás után az Chloros csak magát az érzékelőt csatlakoztatja újra.** Ott vár, ahol az érzékelő volt – és az `--auto` után az `daq-e-<id>.local` név alatt is –, az érzékelő nevére, a gazdagép nevét és a modellt, a korábban rendelkezésre álló hálózati adapter (MAC) alapján, és a válaszadó címen újra csatlakoztatja. A név alatt érkező választ csak azzal a MAC-címmel fogadja el, így egy olyan számítógépről, amely útválasztón keresztül éri el az érzékelőt, vagy a WSL2 alapértelmezett NAT-hálózatából, a DHCP által új címre áthelyezett érzékelő nem kerül újra csatlakoztatásra: a parancs ezt jelzi, és kiírja az `pool-connect --eth-host <its address> …` parancsot, amely visszahozza az érzékelőt a korábbi expozíciós és fényérzékenységi beállításokkal.
- **Az `pool-update-firmware` elutasítja azokat az érzékelőket, amelyek firmware-verzióját nem tudja kiolvasni.** Az a szenzor, amelynek visszatérése a vaku villanása után a várakozási időnél tovább tart, nem hibás: amint válaszol, újra csatlakozik az új firmware beállításainak kitöltésével, és a parancs figyelmeztet, hogy lassú volt.
- **A firmware-frissítéshez bejövő TCP-kapcsolatra van szükség.** Az érzékelő visszacsatlakozik ehhez a számítógéphez a kép begyűjtése érdekében. Az Chloros az érzékelőhöz vezető adapteren hallgatózik (minden adapteren, ha nem tudja megkülönböztetni őket), és kizárólag az érzékelő saját címét szolgálja ki; bármely más kapcsolatot kiszolgálás nélkül lezár. Windows rendszeren a telepítő `Chloros Backend (TCP-In)` tűzfalszabálya engedélyezi a bejutást (lásd [Windows (.exe)](#windows-exe)); egy tűzfal, amely ezt blokkolja, az `… device never connected back on TCP <port> …` hibakóddal meghiúsítja a frissítést.
- **WSL2 alatt használjon tükrözött hálózatot** (`networkingMode=mirrored` az `%UserProfile%\.wslconfig`-ben). Az alapértelmezett NAT-módban a számítógép címe az érzékelő hálózatán a WSL saját címe, ezért minden statikus címet elutasít (kivéve az `--force`-et), az élő adatfolyam nem éri el a WSL-t, és a firmware-frissítés nem tud befejeződni, mert az érzékelő visszacsatlakozik a számítógéphez a kép fogadásához.
- **Az érzékelő azonosítója és a gazdagép neve nem változik.** Bármilyen is legyen a címe, egy DAQ-E továbbra is `daq-e-<id>` és `daq-e-<id>.local`, így az `--eth-host daq-e-<id>.local` továbbra is megtalálja.
- **A grafikus felületen ugyanazok a beállítások**: „Fényérzékelők” fül, az érzékelő beállításai (fogaskerék), „Hálózat”. A grafikus felület fenntartja saját kapcsolatát az érzékelővel, ezért ha a grafikus felületen nyitott érzékelőt módosítunk: az CLI-ről történő újraindítás véget vet a grafikus felület munkamenetének az érzékelővel.
- Az SDK-ben ezeknek megfelelő azonosítók: `DAQSensorSession.device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` és `update_firmware()`.

---

## `chloros-cli project`

Chloros-projekt megnyitása, csatlakozás hozzá és vezérlése (egy mappa, amely tartalmazza az `cameras.json` + `sensors.json` + `project.json` fájlokat). Minden a háttérrendszeren keresztül történik, így a grafikus felhasználói felület (GUI) és a parancssori felület (CLI) azonos hardverállapotot eredményez.

### Alparancsok áttekintése

| Alparancs | Cél |
| --- | --- |
| `project open PATH [--relink FOLDER]` | Kinyomtatja a projekt eszközlistáját (kamerák, mátrixok, érzékelők), beolvassa a fájljaiból, és ellenőrzi a forrásfájljait (lásd [Áthelyezett projekt](#a-moved-project---relink)). Nem nyitja meg a projektet a háttérrendszerben: a projekt ott nyitva marad, és az eszközei, valamint felvételei változatlanok maradnak. Kilép az `1` parancsból, ha a projekt forrásfájljai közül egyet sem talál. |
| `project devices PATH [--reconnect]` | Az eszközfelismerés listázása vagy újrafuttatása. |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | Minden mentett kamera / sorozat / érzékelő csatlakoztatása. Minden sorozat úgy csatlakozik, ahogyan az `lattice array-connect` parancs: legfeljebb 600 másodpercig, a lépéseit a stderr-re írja ki (`[array <id>]    <s> s: <step>`; a stdout a JSON-jelentés), és az ezen időn belül nem csatlakoztatott rendszer visszavonásra kerül — kamerái visszaállnak az eredeti állapotukba —, és az `errors` alatt kerülnek jelentésre, soha nem maradnak csatlakoztatva. Az `--packed` beállítja minden rendszer tömörített 12 bites átvitelét (alapértelmezés: `auto`; egy projekt nem menti el a tömbönkénti beállításokat; lásd [Tömörített 12 bites adatátvitel](#packed-12-bit-transport---packed)). Minden fényérzékelőt a háttérrendszer DAQ-pooljában a mentett címén nyit meg, a mentett integrációs idővel, képkocka-átlagolással, automatikus expozícióval és korlátozással: az a szenzor, amelyhez nem választottak korlátot, az `sunshine_cosine` értéket kapja, a mentett „bare” (`none`) szenzor „bare” marad, a DAQ-E-S pedig soha nem kap korlátot. A GUI DAQ-panele nem része a poolnak: az érzékelőt először ott kell bezárni. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Egyetlen felvétel egy megnevezett kamerából vagy sorozatból (`--prefix` alapértelmezett, `capture`). `--orient` (képalkotó rendszerek): a képalkotó rendszer nyers adatokat ment, amelyek natív formátumban maradnak — ez rögzíti, hogy az `process` által ebből létrehozott termékeket orientálni kell, mint az `array-capture --orient` esetében. Alapértelmezés: natív. `project burst` / `stream` és egy recept `capture` / `burst` / `stream` (`orient: true`) ugyanazt az opciót veszik fel. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | N képkockás sorozat egy megnevezett kamerából vagy tömbből (`-n/--count` alapértelmezett érték: 5; `-i/--interval` képkockák közötti idő másodpercben, alapértelmezett érték: 0). A sorozatfelvételek eltávolítják az ismétlődő szinkronizált csoportokat (elavulás-ellenőrző), így egy részlegesen ciklusban működő sorozat nem adhat vissza egy képkockából N másolatot; kinyomtatja az egyes iterációk eredményeit. Ha a rögzítés félúton meghiúsul, a sorozat leáll, és az `1` kilép, jelezve, hogy előtte hány képkocka került mentésre. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | Rögzítés lemezre ciklusban: az `--fps F` másodpercenként (alapértelmezés szerint 3), egy képkockát rögzít egy kamerából, egy szinkronizált képkockacsoportot egy tömbből (ugyanazzal az elavulási őrrel, mint az `project burst`), amíg az `-n N` rögzíti (alapértelmezés szerint `0` = amíg a Ctrl+C-t nem nyomják meg, ami kinyomtatja, hány képet mentettek, és kilép az `0`-ből, vagy az `1`-ből, ha egy sem mentődött, vagy egy fájl létrehozása sikertelen volt; ha a rögzítés megkezdése előtt (`-n N`) a Ctrl+C billentyűkombinációt nyomják meg, az `130` kilép). A részben sikertelen rögzítés leállítja az adatfolyamot, és kilép az `1`-ből, megjelölve, hogy hány képkocka mentésre került előtte. Az `--poll-interval` elavult és figyelmen kívül marad: még mindig elfogadott, így a régebbi szkriptek futnak, de a stderr-en megjelenik egy figyelmeztetés. |
| `project sensor read PATH NAME [--json]` | Legutóbbi spektrumképkocka. |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | .daq fájl rögzítése az `--seconds` (alapértelmezés szerint 10). Ha a felvétel nem tartalmaz mérési értéket, a rendszer erről üzenetet küld a stderr-re, és kilép az `1` parancsból. |
| `project run PATH RECIPE.yaml` | YAML/JSON-rögzítési recept futtatása. Az `--dry-run` futtatás nélkül érvényesíti. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Számítsa ki a tömb igazítását — lásd [az alábbi jelzőtáblázatot](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Az aktuális igazítási profil kinyomtatása. |
| `project align clear PATH NAME` | A gyorsítótárban tárolt profil törlése. |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | Egy szolga transzformációjának finomhangolása (alapértelmezés szerint `0`, `0`, `0`, `1.0`: nincs változás). |
| `project align export PATH NAME --to FILE` | A profil mentése JSON formátumban. |
| `project align import PATH NAME --from FILE [--no-validate]` | Mentett profil betöltése. |

#### `project align calibrate` Beállítások

| Jelző | Alapértelmezett | Leírás |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Igazítási módszer. **Ezek az írásmódok eltérnek az `lattice align-calibrate`** parancstól, amely az `orb` / `akaze` / `phase` rövidítéseket használja; a két parancs ezen a jelzőn nem cserélhető fel egymással. |
| `--model {translation, rigid, affine, homography}` | `affine` | A modell átalakítása illesztés céljából. |
| `--frames N` | `1` | Szinkronizált képkockák átlagolása átlaghoz. |
| `--reference SN` | a fő | Referenciakamera sorozatszáma; minden más tag ehhez van igazítva. |
| `--max-features N` | `5000` | ORB jellemzőszám-korlát. |
| `--ratio-threshold F` | `0.75` | Lowearánytesztje. |
| `--ransac-threshold-px F` | `3.0` | RANSAC belső pontok küszöbértéke. |
| `--min-matches N` | `15` | **Minőségi küszöb** — elutasítja a megoldást, ha az inlier-egyezések száma alacsonyabb ennél. |
| `--max-reproj-err-px F` | `4.0` | **Minőségi küszöb** — elutasítja a megoldást, ha az RMS-újraprojektálási hiba meghaladja ezt az értéket. |
| `--checkerboard RxC` | — | Az `--method checkerboard` táblageometriája, pl. `9x6`. |
| `--name PROFILE` | üres | A mentett JSON-fájlba ágyazott profilnév. **Nem a tömb neve** — az a pozicionális `NAME`. |

A két minőségi ellenőrzés az oka annak, hogy egy kalibrálás sikeresen megoldódhat, de mégis
elutasíthatja a mentést: egy profil, amelyik bármelyiknél is megbukik, minden
későbbi felvételt észrevétlenül rosszul regisztrálna, ezért elutasításra kerül, ahelyett, hogy mentésre kerülne.

### Áthelyezett projekt (`--relink`)

Egy projekt `project.json` minden forrásfájlt – minden képet és fényérzékelő-
felvételt – a teljes elérési útjával nevez meg. Ha a projektmappát vagy az egész projektek mappáját
egy másik meghajtóra helyezzük át, azok az elérési utak továbbra is a régi helyet jelölik.

A projekt megnyitásakor a rendszer önállóan kijavítja, amit tud: a projekt
mappájában korábban található fájlokat a projekt jelenlegi helyén lévő mappában keresi, és az ott talált összes elérési utat
átirányítja és elmenti (az `project.json`-et először
`project.json.before-relink-<date>-<time>` néven mentik). Ez minden megnyitáskor megtörténik – az alkalmazás,
az `chloros-cli project open`, `chloros-cli process` egy meglévő projektbe, az SDK-be
és az `chloros-mcp`-be. Az elérési út átirányítása mindig csak olyan fájlra történik, amely ott található.

A projektmappa kívül lévő fájlok – például egy hub-munkamenet vagy egy „Mappa hozzáadása” importálás során –
ilyen módon nem találhatók meg. Nevezze el azt a mappát, amelyben jelenleg találhatók:

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink` a fájlok célmappáját vagy annak bármely felette lévő mappát veszi alapul: minden hiányzó
fájl mappáját a fájlnév végén szereplő mappanevek alapján keresi
(`Hub Projects/<session>/raw/<serial>`, majd `<session>/raw/<serial>`, ...), és az egyes
mappa fájljait abból a helyről veszi át, ahol a legtöbbet találja. Az `project open` kinyomtatja,
hogy mit irányított át, hány forrásfájlt talált, és mely mappákban voltak a többi
fájlok. Néhány még hiányzik: figyelmeztetés, kilépés `0` – a projekt a többi fájllal megnyílik. Egy sem
található: hibaüzenet a mappák megnevezésével és `--relink`, kilépés `1`; a háttérprogram szintén
nem hajlandó megnyitni egy ilyen projektet az `process` vagy az `project devices --reconnect` esetében (HTTP 409 `PROJECT_SOURCES_MISSING`).

### Példák

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### Recept-DSL

Az `project run RECIPE.yaml` egy YAML- vagy JSON-fájlt fogad el, amely egy műveletsorozatot ír le:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Támogatott műveletek: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. Az `apply` átveszi az `cameras.json`, `settings` kulcsokat — `exposure_us`, `auto_exposure`, `gain_db`, `auto_gain`, `target_brightness`, … (a régebbi nevek: `exposure_time`, `exposure_auto`, `gain`, `gain_auto` aliasként elfogadottak). Az így beállított expozíció, erősítés vagy célérték a felhasználó saját beállításaként marad meg: az Chloros automatikus expozícióvezérlése (beleértve a DAQ-méréssel meghatározott expozíciót is) nem módosítja azokat. A kézi expozíció esetén az erősítés változatlan marad: automatikus üzemmódban a kamera folyamatosan (a teljes tartományon belül) a célfényerő felé állítja az erősítést, amíg be nem állítja az `gain_db` vagy az `auto_gain: "Off"` parancsot. Egy későbbi `auto_exposure: "Continuous"` parancs visszaadja az expozíciót az automatikus vezérlésnek, az `target_brightness_auto: true` pedig a rögzített célértéket visszaadja a Chloros automatikus célérték-vezérlésének. Az `burst` művelet az `name` (kötelező), az `count` (alapértelmezett érték: 5), `interval` (másodperc, alapértelmezett érték: 0), `output`, `format`, és az `settings` értéket (ugyanaz a kameránkénti beállítási forma, mint az `apply` esetében); a tömbös sorozatfelvételek ugyanazt a frissen szinkronizált csoportos őrprogramot használják, mint az `project burst`.

Az `capture`, `burst` vagy `stream` lépés esetén az `output` alapértelmezett értéke az `output`. Egy relatív `output` — amely alapértelmezés szerint szerepel — a `chloros-cli project run` futtatásának helye szerinti mappához van rendelve, nem pedig a receptfájl vagy a projekt mappájához (az 1.2.0-s verzióban a háttérprogram telepítési mappájába került).

Futtassa:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**Minden futtatásnál először a projekthez viszonyítva történik az ellenőrzés.**Minden műveletnek pontosan a fenti igék egyikét kell kulcsszavaként tartalmaznia — `- capture: {name: FrontLeft}`, nem pedig `- action: capture` vagy `- type: capture` — ha például az `name` kulcshoz a projektben az `cameras.json` / `sensors.json` szerepel, akkor az `apply` / `capture` / `burst` / `stream`, valamint egy fényérzékelővel az `sensor` esetében; egy `stream`-hez szükséges, hogy az `count` értéke > 0 legyen, egy `wait`-hez pedig egy időtartam. Egy ilyen hibát tartalmazó recept**nem** hajt végre semmilyen műveletet (`[recipe] error: …`, `[recipe] not run: …`, kilépés `1`), még `stop_on_error: false` esetén sem. Azok a gombok, amelyeket egy lépés nem használ, `[recipe] warning:`-nek minősülnek, akárcsak minden olyan tömb vagy fényérzékelő, amelyet a recept vezérel, de jelenleg nincs csatlakoztatva. A recept soha nem csatlakoztat eszközöket: először futtassa az `chloros-cli project connect PATH` parancsot. Az `--dry-run` kinyomtatja a receptet, majd ugyanazokat a hibákat és figyelmeztetéseket, az `[recipe] valid: …` / `NOT valid: …` parancsot, nem futtat semmit, és kilép az `1` parancsra, ha a receptben hiba van.

---

## Környezeti változók

| Változó | Hatás |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Felülírja a háttérURL-jét (alapértelmezett: `http://127.0.0.1:5000`) — **csak az `lattice`, `project` és `daq pool-*` parancscsaládok veszik figyelembe** (az `lattice network-analysis` parancs helyett az `--backend-url` parancsot fogadja el). Az alapparancsok (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) az `http://127.0.0.1:<port>`-et használják, és figyelmen kívül hagyják ezt a változót (az IPv4-literál elkerüli a Windows `localhost`→`::1` ~2 másodperces kérésenkénti büntetést), így mindig a helyi gépet célozzák meg. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | A háttérrendszer környezetében állítható be. `1` enyhíti a túlterhelés miatti elutasítást — egy `pin_resolution`-szel összekapcsolt tömb, vagy egy [átmeneti többkamerás parancs](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) — hangos figyelmeztetésre, elfogadva a GVSP-csomagvesztést. Kizárólag tesztelési célra — lásd [Tömb fps és burst modell](#array-fps--burst-model). |
| `CHLOROS_ARRAY_PACKED12` | A háttérrendszer környezetében állítsa be: `on` vagy `off` beállítása minden tömb esetében az [összecsomagolt 12 bites átvitel](#packed-12-bit-transport---packed) alapértelmezett értékét határozza meg (alapértelmezett: `auto`). Egy kifejezetten megadott `--packed` érték továbbra is elsőbbséget élvez. |
| `CHLOROS_DISABLE_UPDATE_CHECK` | Az `1` (vagy `true` / `yes`) kikapcsolja a Linux CLI [automatikus frissítés-ellenőrzését](#chloros-cli-update) indításkor. |
| `CHLOROS_CLI_MODE` | Magát a CLI állítja be; utasítja a háttérprogramot a párhuzamos feldolgozás engedélyezésére. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` kihagyja a GVSP tartalékpróbát (csak ICMP-eredmények esetén). **Ez kikapcsolja a jumbo-t, nem csupán elnémítja a naplózást** — a kamera minden útvonalon legfeljebb 1500-ig válaszol a DF pingekre, így ez a próba az egyetlen, amely képes észlelni a jumbo-t. Kameránként és csatlakozásonként ~1 másodpercet takarít meg; ~1,45-szeres vezetékes sávszélesség-korlátot igényel, ha a hálózat *képes* lenne jumbo-csomagok továbbítására. Az SDK figyelmeztet, ha beállítod. |
| `CHLOROS_GVSP_SESSION_REUSE` | A háttérrendszer környezetében állítsa be: az `0` minden csatlakozáskor méri a csomagméretet, ahelyett, hogy kameránként és háttérrendszer-munkamenetenként egyszer mérné. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | A GVSP-csomagméretet N bájtra rögzíti; teljesen kihagyja a felmérést. A végleges beállítás helyett inkább a parancsonkénti beállítást (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) javasoljuk: a rögzített méret nem alkalmazkodik az előtte lévő hálózathoz, és ha 9000-et rögzítünk egy olyan útvonalon, amely nem képes jumbo-csomagokat továbbítani, akkor **minden** rögzítés időtúllépést eredményez az `SC_ERR_TIMEOUT -1011` használatával. |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | A háttérrendszer környezetében állítsa be: a [tranziens többkamerás parancs](#multi-camera-transient-sessions), vagy egy olyan kamera `lattice capture`-je, amelyet semmi sem tart fenn, az egyes kamerák expozíciójának beállítására fordíthat az előző jelenetből, mielőtt megőrizne egy képkockát (alapértelmezett `5`). Az `0` beállítás kikapcsolja ezt: ekkor az egyes kamerák saját automatikus expozíciója az első képkockától kezdve az expozíciós határértékükön indul. |
| `CHLOROS_LOG_DIR` | A háttérprogram környezetében állítható be: az a mappa, ahová a részletes napló kerül. Alapértelmezett érték: `%LOCALAPPDATA%\Chloros\logs` Windows rendszeren, `~/.cache/chloros/logs` Linux rendszeren (`$XDG_CACHE_HOME/chloros/logs`, ha az be van állítva). A .deb gyökérszolgáltatása beállítja az `/var/log/chloros` értéket, amelyet az `adm` csoport olvashat — lásd [A háttérprogram futtatása szolgáltatásként](#running-the-backend-as-a-service-headless). |
| `TMPDIR` (Linux) | Az ideiglenes fájlok helye. A CLI automatikusan az `/mnt/ssd/tmp`-et használja, ha az meg van adva. |

---

## Kilépési kódok

| Kód | Jelentés |
| --- | --- |
| `0` | Siker. |
| `1` | Általános hiba (a legtöbb alparancs-hiba). A rögzítési parancs akkor is kilép (`1`), ha bármelyik fájl írása sikertelen volt, még akkor is, ha a többi fájl mentésre került, valamint (`array-burst`) / `array-record` kilép az `1`-ből, ha a rögzítő hibát jelent – lásd [Írási hiba miatt elmaradt fájlok](#files-that-fail-to-write). Az `process` szintén leállítja az `1` programot, ha egy futtatás exportált adatai nem ellenőrizhetők, vagy ha egy kép elveszett, annak ellenére, hogy a többi termék leírásra került. |
| `2` | Argumentumhiba (ismeretlen vagy hiányzó jelző; ugyanez vonatkozik az `time-sync set-priority`-re is, ha egyik prioritás sem szerepel). |
| `130` | Ctrl+C billentyűkombinációval megszakítva: bármely parancs, amelyet ez megszakít — `lattice`, `project`, `daq` és `process` egyaránt (`process`, miután a háttérprogram leállította a futást, lásd [`process` Megjegyzések](#notes)). Olyan parancs, amelynek normál leállítása a Ctrl+C — `lattice array-capture --continuous` / `--interval`, `lattice viewer`, `lattice continuous`, `multi-stream`, `align-stream`, `array-record`, `array-burst` és `project stream` parancsok esetében az `-n 0` parancs — a végrehajtott művelet függvényében lép ki: `0`, ha minden leírásra került, `1`, ha valami hiba történt. |
| `143` / `129` | SIGTERM / SIGHUP (128 + a jel) által leállítva, miután a parancs utómunkálatai lefutottak: egy futó feladat leállt és kamerái visszaálltak, a háttérben egy feldolgozási folyamat leállt. Egy `lattice continuous` vagy más olyan felvétel, amelynek normál leállítása a Ctrl+C billentyűkombinációval történik, szintén az `143`-et zárja be, nem pedig az `0`-et. |

Az `lattice network-analysis` kilép az `1`-ből az `needs_force_slip` és az `error` ítéletek esetén, az `selftest` pedig az `1`-et zárja be, hacsak mind a hét ellenőrzés sikeres nem.

---

## Hibaelhárítási tippek

- **„Bejelentkezés szükséges”** → Futtassa egyszer az `chloros-cli login EMAIL PASSWORD` parancsot ezen a gépen.
- **„A projektmappa túl mély ezen a számítógépen”** → Ezen a gépen a Windows hosszú elérési útjai ki vannak kapcsolva (ez a Windows alapértelmezett beállítása), így egyetlen elérési út sem haladhatja meg a 260 karaktert, és a termékek legfeljebb körülbelül 130 karakterrel helyezkedhetnek el a projektmappa alatt. Semmi sem került feldolgozásra. Az üzenet megadja a leghosszabb, még elfogadható projektmappát: használjon rövidebbet (`-o` vagy `chloros-cli set-project-folder`), vagy kapcsolja be a Windows hosszú elérési útjait, majd indítsa újra a Chloros programot. A Windows-telepítő ezt „Windows hosszú elérési útvonalak engedélyezése” néven kínálja, amely alapértelmezés szerint be van jelölve; egyéb esetben használja a Csoportházirendet: *Számítógép konfigurációja > Felügyeleti sablonok > Rendszer > Fájlrendszer > Win32 hosszú elérési útvonalak engedélyezése*.
- **„backend unreachable”** → Indítsa el a Chloros asztali alkalmazást (Windows), vagy indítsa el saját maga a háttérszolgáltatást: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` Windows rendszeren (a telepítési mappa `backend` mappájában; nem az `PATH` mappában található), `chloros-backend` Linux rendszeren. Vagy ellenőrizze az `CHLOROS_BACKEND_URL` mappát, ha távoli rendszerről van szó.
- **Az `lattice` parancsok „LATTICE kamera-illesztőprogramok nem találhatók” hibával buknak meg** → A háttérprogram nem tudta betölteni a telepítéskor mellékelt LATTICE kamera futásidejű környezetet. Minden `lattice` parancs a háttérprogramban fut; maga a CLI nem tartalmaz kamerameghajtókat. Telepítse újra a Chloros-t: a Windows-telepítő és a .deb fájl a futásidejű környezetet a háttérprogram mellé telepíti.
- **Az Array connect / Array Settings a „FRAMES WILL DROP” vagy „Reduce ROI to enable” üzenetet jeleníti meg** → A gazdagép hálózati kártyájának vételi gyűrűje túl kicsi (a hálózati kártya-illesztőprogram frissítése után általában 32-re áll vissza). Lásd [Host NIC beállítás és hangolás](#host-nic-setup--tuning-lattice-arrays) — állítsa be az `ReceiveBufferLen=256` és `PendingReceives=64` értékeket.
- **A gép újraindításkor/leállításkor lefagy, majd a WMI `Invalid class` hibát jelez / a hálózati kártya nem aktiválható / az USB-meghajtók hiányoznak** → Elavult USB 10 GbE adapter-illesztőprogram, amely `DRIVER_POWER_STATE_FAILURE` hibát okoz (BSOD `0x9F`). Frissítse az adapter illesztőprogramját — lásd [Host NIC beállítás és hangolás](#host-nic-setup--tuning-lattice-arrays).
- **Linux: a rögzítések a root felhasználóhoz tartoznak, ezért nem törölhetők, és a háttérprogram naplófájlja nem található a `~/.cache` mappában** → A háttérprogram root jogosultsággal fut a `chloros-backend.service`-en keresztül. Váltson át az `chloros-backend@<user>.service`-re, amely a háttérprogramot az Ön felhasználói fiókjával futtatja — lásd [A háttérprogram futtatása szolgáltatásként](#running-the-backend-as-a-service-headless). Addig a root szolgáltatás naplófájlja az `/var/log/chloros`-ben található (amelyet az `adm` csoport olvashat).
- **Linux: egy háttérmodul az `failed`, és a „Már fut egy másik példány” üzenet jelenik meg (kilépési állapot: 98)** → A portot már egy másik háttérmodul foglalja: egyszerre csak egy futhat. Ha két háttérmodul van engedélyezve, tartson meg egyet: az `sudo systemctl disable --now <unit>`-et a nem használt modulhoz — lásd [A háttérmodul szolgáltatásként való futtatása](#running-the-backend-as-a-service-headless). Ha csak ez az egység van engedélyezve, akkor a portot egy a systemd-n kívül indított háttérprogram (az `chloros-backend`, az `chloros-cli` vagy egy SDK-parancsfájl által) foglalja: állítsa le azt a háttérprogramot, majd futtassa az `sudo systemctl start <unit>` parancsot.
- **Jetson swap figyelmeztetés** → Adjon hozzá fájlalapú swap-ot; aCLI kinyomtatja a pontos `fallocate` / `swapon` parancsokat.
- **„backend not reachable” hibaüzenet egy `lattice`, `project` vagy `daq` parancs futtatásakor** → nincs futó backend: ezek a parancsok soha nem indítanak el egyet sem. Futtassa az `chloros-cli backend status` parancsot, és olvassa el a [A háttérprogram elindítása](#start-the-backend) című részt.
- **„Az `daq net ...` nem érhető el a Chloros-ban”** (Linux és Jetson) → A DAQ-E saját beállításainak egy része a MAPIR szolgáltatás számára van fenntartva. A hálózati beállítások, az újraindítások és a firmware-frissítések az `pool-set-network`, az `pool-reboot` és az `pool-update-firmware` parancsokkal végezhetők el; a többi esetben vegye fel a kapcsolatot a MAPIR ügyfélszolgálattal.
- **„A DAQ közvetlen hardverparancsaihoz a teljes daq csomag szükséges”** (Windows) → Használja az `pool-*` parancsot, amely a háttérrendszeren keresztül vezérli ugyanazt az érzékelőt, vagy az `chloros_sdk.connect_daq_sensor()` parancsot Pythonból.

---

## Lásd még

- [Python SDK-referencia](sdk-reference.md) — minden CLI-parancs programozási megfelelője.
- [DAQ fényérzékelők](https://mapir.gitbook.io/chloros/daq-light-sensors) a Chloros-kézikönyvben — az érzékelők, azok határai és kalibrált tartománya, rögzítés, DAQ-E hálózat.
- [LATTICE kamerák](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) a Chloros kézikönyvben — csatlakoztatás, kamera beállítások, rögzítési módok, tömbök.
- Online dokumentáció: `https://mapir.gitbook.io/chloros/cli`
