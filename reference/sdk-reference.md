# Chloros Python SDK-referencia

**Verzió:**

1.3.0**Létrehozás:**

2026.07.29. 19:19 ·**Frissítés:**

2026.11.06.**Csomag:** `chloros-sdk` (PyPI)**Célközönség:** LLM-felhasználásra optimalizálva; ember számára olvasható.**Hatály:** Az `import chloros_sdk` által elérhetővé tett minden nyilvános osztály, függvény és segédfüggvény, másolható példákkal a képfeldolgozás, az egykamerás vezérlés, a szinkronizált tömbök, DAQ-érzékelőket és a projekt automatizálását, valamint az `chloros-mcp` szervert az AI-asszisztensekhez.

Ha csak a legfontosabb információkra van szüksége, ugorjon ide:
- [Telepítés és gyorsindítás](#telepítés)
- [Smart-Connect a LATTICE-tömbökhöz](#smart-connect-for-lattice-cameras)
- [DAQ-érzékelő-munkamenetek](#daq-sensor-sessions)
- [Kamera-segédprogramok](#camera-utilities)
- [Transzient-sorozatok, igazítási profilok és indexek](#transient-arrays-alignment-profiles--indices)
- [Projektmappa és receptek](#project-folder--recipes)
- [Projekt automatizálás](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Élő képkockák a saját feldolgozási folyamatban](#live-frames-in-your-own-pipeline)
- [MCP-kiszolgáló (AI-asszisztensek)](#mcp-server-ai-assistants)

> **Változások az 1.3.0-ban** (amit egy meglévő 1.2.0-s szkript észlel).
>
> **Változások, amelyekkel egy 1.2.0-s szkriptnek esetleg meg kell birkóznia:**
>
> - **A háttérrendszer által jelenleg nem megerősített kiadás hibaüzenetet eredményez.** `CameraSession.disconnect()`, `ArraySession.disconnect()` / `force_disconnect()` egy olyan tömbön, amelyet a munkamenet nyitott meg, `DAQSensorSession.disconnect()` / `force_disconnect()` egy általa megnyitott érzékelőn, valamint az `ChlorosProject.disconnect_all()` hibaüzenetet vált ki az `ChlorosConnectError`, ha a háttérrendszer elutasítja a felszabadítást vagy nem erősíti meg időben, mert az eszköz még mindig foglalt lehet. Az 1.2.0-s verzióban ezek csendben tértek vissza. Egy `with` blokk kilépése továbbra sem vált ki hibajelentést.
> - **A kölcsönzött eszközt soha nem szüntetik meg.** Az `with attach_array(...)` és egy kifejezett `disconnect()` hívás a tömböt futó állapotban hagyja (az 1.2.0-s verzióban a blokk kilépése megszüntette azt, még akkor is, ha a GUI birtokolta); hívja meg az `force_disconnect()` parancsot, ha be akarja fejezni. Hasonlóképpen egy `connect_daq_sensor()` parancs, amely már megtalálta az érzékelőt a poolban – beleértve a nulla-jelzővel ellátott intelligens felismerést is –, nyitva hagyja azt az `disconnect()`-en, és az `with` blokk elhagyásakor is nyitva hagyja.
> - **A PNG és a JPEG formátumok csak az `display` és az indexelt termékeket fogadják el.** `capture()` az `format="png"` / `"jpg"` (`ext=".png"` / `".jpg"`) és egy `radiance` vagy `reflectance` szint, vagy `processing="all"`, minden kérés előtt kiadja az `ValueError`-et; egy 12 bites `raw` / `debayered`-et a háttérrendszer elutasít. Lásd [PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only).
> - **Az `connect_daq_sensor(sensor_id=...)` megnyitja az adott érzékelőt, vagy** kiadja az `ChlorosConnectError` hibakódot, felsorolva, mit talált; soha nem vált át egy másik érzékelőre. Lásd: [Rögzített átvitel](#pinned-transport).
> - **`frame_stream()` Az  hibaüzenetet ad ki, ahelyett, hogy csendben maradna:** `FrameFetchError` Elutasítás esetén a következő ciklusban az  megismétli (egy reflektancia-leolvasást, amely egyáltalán nem tartalmaz DAQ-ot), 60 másodperc elteltével keret hiányában, valamint 60 másodperc elteltével új trigger hiányában. Lásd: [Egy trigger leolvasásonként](#one-trigger-per-read--group_id).
> - **Egy leolvasási időtúllépés vagy megszakadt kapcsolat minden smart-connect hívásnál `ChlorosConnectError`** hibát vált ki, valamint `analyze_array_network()` hibát, ahol az 1.2.0 verzió az alapul szolgáló időtúllépést vagy kapcsolati hibát váltotta ki.
> - **Az `connect_array()` alapértelmezés szerint 600 másodpercig vár (`timeout`, 1.2.0-ban 180 másodperc), és a sikertelennek ítélt kapcsolatot visszavonja.** A hívás elküldi az `timeout` hibakódot a háttérrendszernek, amely minden kamerát visszaállít az eredeti állapotába, és nem hoz létre kapcsolatot, ha a csatlakozás nem történik meg időben; a hívás `ChlorosConnectError` hibakódot generál (HTTP 504 `CLIENT_DEADLINE_PASSED`) hibát. Az 1.2.0 verzióban a kamerák bekapcsolása után azonnal kezdeményezett kapcsolat meghaladhatta a hívás időtartamát, és a hiba kiváltása után is összekapcsolhatta a tömböt. Az `connect_all()` minden mentett tömb esetében ugyanúgy várakozik (120 másodpercig az 1.2.0-ban). Lásd [`connect_array()` aláírás](#connect_array-signature).
> - **Az `connect_array()` megőrzi a teljes érzékelőt.** `width` / `height` / `binning` hiányában minden kamera teljes érzékelőre van beállítva, és a képkockasebesség a gazdagép és a hálózat által biztosított értékre csökken; 1.2.0 a képkockákat kicsinyítette vagy összevonta a képsebesség fenntartása érdekében. Lásd [Szinkronizált tömb](#synchronized-array--arraysession-smart-prep).
> - **Az M3M (mono) kamerán a Bayer-pixelformátum a mélységet jelenti.** Az M3M csak monó formátumokat kínál, és az előre beállított értékek nevei Bayer-formátumokra utalnak: az 1.2.0-ban a kamera elutasította a formátumot, és megtartotta a sajátját, míg az `settings.pixel_format` a Bayer-formátumot nevezte meg. `LatticeCamera(preset=...)`, `apply_preset()`, `set_format()`, egy `CameraSettings(pixel_format=...)`, `connect_camera(preset=...)` és egy tömbmegosztott formátum most ugyanolyan mélységű Mono formátumot állít be egy M3M-en: az `"BayerRG8"` és az `high_speed` az `Mono8`-et adja; az `"BayerRG12"`, az `high_quality` és az `triggered` az `Mono12`-et eredményezi; az `cam.settings.pixel_format` nevezi meg. Ugyanez fordítva is igaz: egy M3C-t soha nem helyeznek Mono formátumra, így az `"Mono12"` – bármelyikből, `connect_array(pixel_format=...)`, `set_settings(pixel_format=...)` vagy egy tömb mentett formátumából származik – M3C-n `BayerRG12`-et eredményez, függetlenül attól, hogy melyik kamera a tömb vezetője. Az átadott `CameraSettings` megtartja a nevét, így ugyanaz az objektum továbbra is minden kamerától azt a mélységet kéri a saját családján belül. Az `connect_camera(preset=...)` megtartja az előre beállított érték formátumát (az 1.2.0-s verzió az előre beállított érték alkalmazása után 12 bitesre állítja a kamerát), és egy előre beállított értékre megnyitott kamera már nem írja be az `PRESETS`-be azt, amit rögzített. Bármely más formátumot, amelyet a kamera elutasít (amely hiányzik az `PixelFormat` listájából, például az `RGB8` egy M3M-en, vagy az `BayerRG16` egy olyan firmware-en, amely nem támogatja), azt sem nevezi el sajátként: egy `LatticeCamera` csatlakozás után az `apply_settings()` vagy az `set_format()`, az `cam.settings.pixel_format` és az `packed_transport` megnevezik a kamera által továbbított formátumot, míg egy kinyomtatott sor megnevezi az elutasított formátumot.
> - **Az `analyze_array_network()` meghatározza, hogy melyik csatlakozás érvényes:** az `pixel_format=None` (korábban `"BayerRG8"`) azt jelenti, hogy a csatlakozás az (Mono12 / BayerRG12), az `packed="auto"` pedig a csomagolt átvitelre vonatkozó döntést, így az általa megadott fps eltérhet az 1.2.0-tól-tól eltérhet. Egy átadott `pixel_format` egy M3M-et jelöl az adott mélységben. Lásd [Hálózatelemző segédprogram](#network-analysis-helper).
> - **Az `CameraSession.capture()` egy `CaptureResult`-et ad vissza** (még mindig egy keret-szótárakból álló lista) az `.skipped`, `.failed`, `.daq_files` és `.daq_matches` értékekkel. Az `ArraySession.capture()` eredmény szintén tartalmazza az `.alignment`-et (hogy alkalmazták-e az igazítási torzítást, és ha nem, miért nem) és az `.oriented`-et.
> - **A tömbök rögzítéseit alapértelmezés szerint igazítják, akárcsak az alkalmazáséit.** `ArraySession.capture()`, `continuous()`, `capture_repeated()` és `ArrayHandle.capture()` / `capture_smart()` minden tag `debayered`, `display`, `radiance`, `reflectance` és indexfájljait a fő kamera képkockájába, és kivágja őket a tagok átfedési (amely a kamerák eltolásainak megfelelően kisebb, mint az érzékelő, `XMP-mapir:AlignmentJSON` `"applied": true`), így ezek ugyanazok a fájlok, mint az Chloros alkalmazás által ugyanarról a jelenetről készített felvételek; az `raw` natív formátumban marad. Korábban egy `aligned=True` nélküli rögzítés minden fájlt natív méretben, torzítás nélkül mentett. Az `aligned=False` ezt megtartja. Az `chloros-mcp``capture_array` és `start_capture_run` ugyanazt az alapértelmezett beállítást veszi át. Lásd [`ArraySession` módszerek](#arraysession-methods).
> - **Az `connect_all()` egy olyan háttérrendszert jelöl, amely elutasítja az SDK-t.** A futó háttérrendszer, amelyből kijelentkeztek, vagy amely a Copper alatti csomagon fut, az `errors` alatt kerül jelentésre a saját okával (HTTP 401 / 403), nem pedig „nincs futó Chloros-háttérprogram”ként, és minden `connect_all()`-hívás újra ellenőrzi a háttérprogramot, így a bejelentkezés utáni hívás ugyanazon a projektobjektumon működik.
> - **A relatív `output_dir` hivatkozás a helyi háttérrendszerre vonatkozik.** A relatív útvonalakat a szkript munkakönyvtárához viszonyítva abszolút útvonalakká alakítja a rendszer, mielőtt elküldené őket. Az 1.2.0-s verzióban a háttérrendszer a saját telepítési mappájában értelmezte őket, ami „Hozzáférés megtagadva” hibaüzenettel járt.
> - **Az `numpy` mostantól az SDK követelménye** (a live-frame hívások numpy tömböket adnak vissza).
> - **Az „egy” értéktől eltérő exportszintű nevet a rendszer elutasítja.** Az `processing=` és az `levels=` az `raw`, az `debayered`, az `radiance`, `reflectance` vagy `display` (vagy egy olyan álnevet, mint például az `preview`; az `processing=` és az `"all"` is). Bármely más név elutasításra kerül, mielőtt a kamerát megérintenék (HTTP 400 `UNKNOWN_LEVEL`: `capture()` hibaüzenetet eredményez `ChlorosConnectError`, `grab()` / `frame_stream()` `FrameFetchError`); az 1.2.0 verzió ezt elmentette vagy visszaadta `raw` kóddal.
> - **A Texture Aware funkció nem használható LATTICE-felvételek esetén.** Egy LATTICE-felvételeket tartalmazó projektben az `configure(debayer="Texture Aware (Slow, Highest Quality)")` hiba az `ChlorosConfigurationError` hibát váltja ki, és ugyanez történik az `process()` esetében is, miután az importálás előtt egy `configure()` hiba az importálás előtt. A projekt által korábban mentett Texture Aware érték Standardra van állítva, egy `UserWarning` hibaüzenettel. A LATTICE-felvételek mindig a standard demosaic-ot kapják, ezért az 1.2.0 verzió elfogadta a beállítást és figyelmen kívül hagyta; a Chloros-alkalmazás sem kínálja fel ezt számukra. Lásd [Támogatott értékek](#supported-values).
> - **A kamerák visszatérnek az eredeti állapotukba — kivéve, ha `keep_settings=True`.** `ArraySession.disconnect()` / `force_disconnect()`, minden átmeneti funkció vége (`capture_cameras_transient()`, `start_multi_camera_stream()`, `test_array_sync()`, `calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index()`) és az `start_camera_continuous()` leállítása minden kamerát visszaállít az eredeti állapotába: a kioldó- és szinkronvezeték-kábelezését, a PTP-jét, valamint minden olyan kamerabeállítást, amelyet a munkamenet során megváltoztattak (expozíció és erősítés, az AE-célérték, a határértékek és a csillapítás, a fehéregyensúly, a feketeszint, a képpontformátum, a binning és az ROI, a tükrözés, a képkockasebesség). Az `keep_settings=True` parancs viszont minden kamera beállításait a kamera saját memóriájába menti, így a kamera ezekkel a beállításokkal indul el; az `clear_settings=True` parancs pedig a gyári beállításokra állítja vissza azt a memóriát. Mindkét esetben a kamerák szabadon futnak tovább. A projekt a saját mentett beállításai alapján állítja be a kameráit, amikor csatlakozik hozzájuk. Lásd [Kamera memória](#camera-memory--keep_settings--clear_settings).
> - **Minden üzemmód ugyanúgy menti el a projektet.** A projekthez csatlakoztatott kamerák, sorozatok vagy fényérzékelők bármilyen módosítása – akár az SDK-n, az alkalmazáson, a CLI-n vagy az MCP-kiszolgálón keresztül történik – elmentésre kerül a projekt `cameras.json` / `sensors.json` mappába kerül, és a projekt bármelyik üzemmódban történő megnyitásakor az eszközök beállításai ebből a mappából kerülnek beállításra; az `ChlorosProject` nevű mappa pedig a saját mappáját nevezi meg a háttérrendszer számára. Korábban csak az alkalmazás mentette el a projektet. Lásd [Egy projektbejegyzés, minden módban](#one-project-record-every-mode).
> - **A több kamerából készült fájlok mindegyikét megnevezik.** A többsávos képcsomag, a több kamera sávjaiból készült élő index és a tömbfelvétel `composite/` képe listaként hordozza az egyes forráskamerák sorozatszámát, modelljét és kalibrációs csomagját (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), az EXIF `SerialNumber` pedig a sorozatszámokat vesszővel elválasztva sorolja fel. Korábban vagy egyáltalán nem tartalmaztak kamerát, vagy csak az első kamera sorozatszámát. Lásd [Több kamerából készült fájlok](#files-made-from-several-cameras).
> - **Csak az `raw` fájl tartalmazza a kamera kalibrációs címkéit.** A MicaSense-kompatibilis készlet (`BlackLevel`, `RadiometricCalibration`, az egyescsatornás `Band{R,G,B}RadiometricCalibration`, valamint az `MicaSenseVignetting*` készletek) és az `CalibrationJSON` hasznos adat a szenzor nyers értékeit írja le, ezért kizárólag az `raw` rögzítési fájlban szerepelnek. Minden egyéb fájl — a rögzítési`debayered`, `display`, `radiance`, `reflectance` és indexfájlok, a folyamatos felvételű és többkamerás-folyamból származó képkockák, valamint minden `process()` termék – hordozza a kamera azonosítóját (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...), valamint felvételkor a képkocka saját adatait (`XMP-mapir:FrameJSON`: expozíció, erősítés, hőmérséklet, képpontformátum, felvételi idő). Az 1.2.0-s verzióban a nyers adatsort is tartalmazták, amelyet egy feldolgozó eszköz (Pix4D, Metashape) alkalmazott azokra a képpontokra, amelyek nem nyers értékek voltak. A feldolgozó program az ilyen fájlok kalibrációját a sorozatszám és az `CalBundleSha256` alapján határozza meg.
>
> **Egyéb változások:**
>
> - **A tömb automatikus képkockasebessége a tömb által szolgáltatott értékekhez igazodik.** Az a sebesség, amely a triggerjeinek több mint 1%-át elveszíti, lecsökken – ezt nem egy, hanem több hosszú megfigyelés alapján ítélik meg –, és az a sebesség, amelyről lecsökkent, időnként újra megpróbálásra kerül. A saját magad által beállított sebességet a rendszer nem módosítja.
> - **Az automatikus expozíció stabilabb.** Állandó fényviszonyok mellett állandó marad, változás esetén másodpercek alatt stabilizálódik, figyelmen kívül hagyja a kis fénycsúcsokat, és helyreállítja a fényes napfényben nem reagáló kamerát. Lásd: [Hogyan viselkedik az automatikus expozíció](#how-auto-exposure-behaves).
> - **Az `import_images(recursive=True)` csak a képeidet és felvételeidet számolja.** Egy olyan mappában, amely egy projekt `camera_cal/` fájljait tartalmazza, ezek a kalibrációs fájlok már nem számítanak képeknek és felvételeknek, és egy projekt, amely már felvette őket, eltávolítja őket.
> - **Az `configure(radiometric_output=...)` érvényes.** Mostantól ez állítja be a projekt `Export radiance` / `Export reflectance` termékeit; korábban minden projekt saját termékbeállításai felülírta ezt, így az `radiance`, az `sensor-response` és az `all` mind sugárzási és visszaverődési értékeket exportált.
> - **Az `test_array_sync()` a kamerák bekapcsolt állapotából indul.** A futtatás idejére bekapcsolja a kamerák PTP-jét, megvárja, amíg az óráik szinkronizálódnak és stabilizálódnak az első trigger (`ptp_lock`) előtt — minden kamera 120 másodpercig, a bekapcsolástól számított körülbelül 2 percig szinkronizálódik (`ptp_settle_s=` állítja be), mivel egy éppen szinkronizált óra körülbelül 2 percig akár ~200 us-os eltérést is mutathat — majd minden kamera PTP-jét visszaállítja az eredeti állapotába. Azokra a kamerákra, amelyek egy korábbi tesztből még mindig szinkronizált állapotban vannak, nem vár újra. Korábban kikapcsolt állapotban hagyta a PTP-t egy olyan egykapcsolós hálózaton, ahol egyetlen DAQ-E sem válaszolt, és minden kamera szabadon futó óráját olyan PTP-óraként olvasta be, amely nem szinkronizálódik („A PTP nincs rögzítve”, minden futtatáskor).
> - **Egy `.daq` felvétel jelzi, hogy a sapkáját választották-e ki, vagy feltételezték-e.** A DAQ-U, a DAQ-M és a DAQ-E fedél nélkül kerül forgalomba, és a Chloros nem tudja látni, hogy van-e rajta, ezért egy név nélküli fedél nélkül csatlakoztatott érzékelő alapértelmezés szerint az `sunshine_cosine` nevet kapja, ami egy *feltételezés*. Minden Chloros-felvétel (1.30) mostantól az `cap_id` mellett tárolja az `cap_id_source` értéket: `operator`, ha megnevezett egy sapkát (beleértve az `none`-et is), `auto_default`, ha az volt az alapértelmezett. Ha olyan felvételt dolgozunk fel, amelynél a kondenzátor értéke feltételezett volt, és a projekt DAQ-kondenzátorának beállítása „Auto” maradt, akkor a fájl nevében megjelenik egy `[daq-dls] WARNING`: egy puszta DAQ-U a napfény-korrekció alatt körülbelül 30-szor magasabb értéket jelez. **Nevezze el a beállított korlátot** a csatlakozáskor (`connect_daq_sensor(cap_id=...)` vagy `DAQSensorSession.set_cap()`, `chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap`, vagy a GUI kondenzátor-legördülő menüjéből), vagy állítsa be a projekt DAQ-kondenzátorát, és a figyelmeztetés eltűnik.
> - **Nincs több pip konzolablak Windows rendszeren.** Egy Python-tel felszerelt PC-n a háttérprogram és az egyes képfeldolgozók indításkor egyszer futtatták az `pip` parancsot, és mindegyik megnyitott egy konzolablakot. Ez már nem történik meg.
> - **Az élő `.daq` felvételek túlélik az áramkimaradást.**A felvétel minden commitkor naplóval és teljes lemezszinkronizálással kerül írásra az érzékelő mögött, így egy lassú lemez késlelteti a fájlt, de soha nem a mérési értékeket. Egy commit közben leállított felvétel (áramkimaradás, leállítás) mellett maradhat egy `<name>.daq-journal` fájl:**másolja össze a kettőt.** A Chloros az első fájlbeolvasáskor visszagörgeti a naplófájlt, így a felvétel a leállás előtt elküldött összes adattal nyílik meg (kb. az utolsó 3 másodperc veszik el); az 1.2.0-s verzióban egy rossz pillanatban bekövetkező áramkimaradás az egész fájlt olvashatatlanná tehette („az adatbázis lemezképe hibás”).
> - **A forgalmas lemezen található `DAQSensorSession.record_start()` már nem blokkolja az érzékelőt**: az értékek sorba kerülnek, és saját szálon kerülnek írásra, és az, amit nem sikerült írni, az `status()`-ben az `record_errors`-ben (az `record_last_error`-szel együtt) kerül számításba.
> - **Közvetlen `LatticeCamera`:**egy olyan képkocka, amelyben nincs megadva a chunk expozíciója/erősítése, mostantól a kamera**valódi** expozícióját és erősítését jelenti (a korábbi megoldás egy gyorsítótárban tárolt értéket olvasott be, amely egy teljes automatikus expozíciós futás alatt változatlan maradhatott), az `apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')` pedig ugyanazt az 5 µs-os bemeneti zavarmentesítést alkalmazza, mint amit a Chloros kábeles szolgaegységei kapnak (szűretlenül, a szolgaegység trigger nélkül is elindít képkockákat).
> - **A szoftveresen vezérelt kamerák már nem hagyják ki minden második triggerjelet.** A szoftveres trigger módban lévő kamera most már sorba állítja az olvasás során beérkező triggerjeleket (vagy a Chloros megvárja, amíg a kamera készen áll), ahogyan azt a kábeles szinkronizált kamerák már eddig is tették.
> - **Az `discover_daq_sensors()` egy DAQ-E-S-t nevez meg `DAQ-E-S`-ként.** Minden Ethernet-érzékelő esetében az `model` az `DAQ-E` volt; az, amelyik körülbelül 2 másodpercen belül nem válaszol az azonosítási ellenőrzésre, továbbra is szerepel a listán, mint `DAQ-E`.
> - **Új: `chloros-mcp`, egy MCP-kiszolgáló AI-asszisztensekhez.** A Claude Desktop, a Claude Code, a Cursor és más MCP-kliensek képesek képanyagokat feldolgozni (projekt létrehozása, felvételek importálása, beállítások kiválasztása, feldolgozás, a futtatás eredményeinek jelentése), valamint LATTICE kamerákat, sorozatokat és DAQ fényérzékelőket működtetni: csatlakozás, előnézet, expozíció és szín beállítása, felvétel, rögzítés. Eszközei lefedik az SDK funkcióit. Az SDK-vel együtt települ, a Linux .deb csomag pedig az `/usr/bin` könyvtárba helyezi. Lásd [MCP-kiszolgáló (AI-asszisztensek)](#mcp-server-ai-assistants).
> - **Újdonság: kamera segédprogramok**, az `chloros-cli lattice` parancsok, ahogyan az SDK-t hívja meg: színprofil, telítettség / kontraszt, fényes részek, fehéregyensúly (egyszeri, automatikus, rögzített arányok), előre beállított értékek, szűrőmodellek, firmware-állapot és frissítés, tápellátás (állapotfelmérés, alvó mód, ébresztés), célkalibrálás, hálózati adapterek (ellenőrzés, becslés, javítás), fájlok vignettakorrekciója, DLS-leolvasás és a gazdagép állapotfelmérése. Lásd: [Kamera segédprogramok](#camera-utilities).
> - **Újdonság: a többi CLI-parancs SDK-hívásként**, így a CLI, az SDK és az `chloros-mcp` ugyanazokat a funkciókat kínálják (a CLI biztonsági okokból csak a bejelentkezést, a DAQ-E-k konfigurálását és a közvetlen DAQ-kiszolgálókat tartja meg): egyetlen parancsra megnyitott kamerák (`start_camera_continuous()`, `start_multi_camera_stream()`, `capture_cameras_transient()`, `test_array_sync()`, `detect_array_master()`, `analyze_array_models()`), igazítási profilfájlok (`calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `alignment_profile_info()`, `reorder_alignment_profile()`), indexek (`index_catalog()`, `compute_index()`, `capture_live_index()`), a PTP grandmaster (`time_sync_status()` és társai), a projektmappa (`project_folder()`, `set_project_folder()`, `reset_project_folder()`) és a receptek (`run_project_recipe()`, a CLI saját futtatója). Fényérzékelők esetében: `connect_daq_sensor(cap_id=...)`, `DAQSensorSession.set_cap()`, `daq_bindings()` / `set_daq_bindings()`, `daq_band_presets()` és `light_sensor_reflectance()` (két érzékelő környezeti + objektum-visszaverődése és indexei); kamerához: `CameraSession.status()` és `capture_repeated()`; egy sorozat esetében: `ArraySession.capability()` és `set_member_settings()` (egy elem expozíciója, erősítése, ...). Lásd [Tranziens sorozatok, igazítási profilok és indexek](#transient-arrays-alignment-profiles--indices), [Projektmappa és receptek](#project-folder--recipes) és [DAQ-érzékelő munkamenetek](#daq-sensor-sessions).
> - **Továbbfejlesztett LATTICE indextermékek** (NDVI és egyéb indexek) az `process()`-ből, minden feldolgozási módban.
> - **LATTICE OCN kamerák: javított sugárzási, visszaverődési és indextermékek napfényben.** A sávszétválasztás után flottakorrekciót alkalmaznak — élőben, rögzítésekben és az `process()` verzióban; az RGN kamerák változatlanok maradnak. Lásd [Radiometrikus kimenet](#radiometric-output-lattice-multispectral-pipeline).
> - **Minden rögzítés elmenti a felhasznált DAQ-leolvasást, ahogyan a grafikus felület is.** Az `CameraSession.capture()` és az `ArraySession.capture()` / `continuous()` fájlok az `.daq` fájlt és annak `.match.json` fájlokat, amikor egy DAQ-ot hozzárendelnek vagy csoportosítanak, beleértve a kizárólag nyers adatokat tartalmazó rögzítéseket is; az 1.2-es verzióban0-ban a nyers vagy debayered rögzítés csak az `force_daq=True`-szel mentett egyet. Az `daq=False` kikapcsolja, az `daq_tolerance_ms=` pedig beállítja az egyezési tűréshatárt; mindkét eredmény megmutatja, mi mentődött az `.daq_files` és az `.daq_matches` fájlokba. A rögzítés most várja a DAQ következő leolvasását, hogy össze tudja keverni az expozíció mindkét oldalán lévő leolvasásokat. Lásd [A rögzítés által mentett DAQ-leolvasás](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches) című részt.
> - **Az `ArraySession` élő képkockákat és szerveroldali folyamatos rögzítést tartalmaz.** Az `grab()` és az `frame_stream()` egy triggerből származó numpy-képkockákat ad vissza (ugyanaz az olvasás, mint a projekt API-jének `ArrayHandle.grab()` / `frame_stream()` parancsai), az `continuous()` pedig minden szinkronizált csoportot elment a háttérben, és egy [`ContinuousHandle`](#continuoushandle) értéket. Lásd: [Élő képkockák a saját feldolgozási folyamatodban](#live-frames-in-your-own-pipeline).
> - **Egy `grab()` tömb egy triggerhez van rögzítve** (`ArrayHandle.grab()` és az új `ArraySession.grab()`). Az 1.2.0-s verzióban a 20 négy-kamerás `ArrayHandle.grab()` olvasás közül 12 két triggerre terjedt ki. Most minden képkocka hordozza az `group_id`-et, és egy `frame_stream()` tömb minden triggert egyszer ad vissza. Lásd: [Egy trigger olvasásonként](#one-trigger-per-read--group_id).
> - **Sávonkénti és igazított képkockák:** az `grab()` / `frame_stream()` az `band=`-et veszi fel, `align=`, `bit_depth=` és `refl_max=` kamerakezelőket és tömbkezelőket. Lásd: [Egyetlen sáv exportálása](#exporting-a-single-band--band).
> - **Új `ArraySession` metódusok és argumentumok:** `set_target_fps()`, `force_disconnect()`, `capture(after_seq=..., orient=...)` (az `CaptureResult.group_seq` és `.failed` mellett), `continuous()`, `connect_array(packed=...)`; az `capture_repeated()` egymást követő csoportokat láncol össze, és a meghajtó megteltével leáll.
> - **`ArraySession.record()` felvételek megnyitott grafikus felület nélkül.** A grafikus felület kombinált nézetének bezárásakor a háttérrendszer maga rendereli a nézetet: a tömbNDVI-jét, vagy az `expr=`-ként átadott indexet. Az `RecorderHandle.stop()` hibaüzenetet dob egy sikertelen felvétel esetén, amely nem írt be egyetlen képkockát sem, és a leállási eredményt az `.result`-en (a `with` blokk elhagyása nem vált ki kivételt: ellenőrizze az `rec.result`-et). Lásd [`RecorderHandle`](#recorderhandle).
> - **Új `DAQSensorSession` módszerek:** `enable_ptp()`, `force_disconnect()`, valamint a DAQ-E saját beállításaihoz `device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` és `update_firmware()`. Az `record_stop()` parancs az `warning` és az `gaps` értékeket adja vissza. Az a újraindítás, amely lezárta a felvételt, megnevezi azt a válaszában, vagy – ha az újraindítás sikertelen – az `ChlorosConnectError`-ben (`recording_closed`; lásd [DAQ-E eszközbeállítások](#daq-e-device-settings)). Lásd [`DAQSensorSession` módszerek](#daqsensorsession-methods).
> - **Egy olyan poolba tartozó DAQ-E, amely leáll az adatátvitellel, a háttérrendszer által újra csatlakoztatódik** a korábbi beállításokkal, és az azon futó felvétel ugyanazon az `.daq`-en folytatódik, a hiányzó rész a fájlban megjelölve; az `record_stop()` a hiányzó részeket `gaps` formátumban adja vissza. Az 1.2.0-s verzióban az érzékelő addig nem adott jelzést, amíg le nem választották, majd újra nem csatlakoztatták, ami véget vetett a felvételnek. Lásd a CLI-referencia [A DAQ-E, amely leáll az adatátvitellel](cli-reference.md#a-daq-e-that-stops-delivering) című részét.
> - **Az `open_project()` egy telepített SDK-ről működik.** A wheel (a telepítőés a PyPI-k) most már tartalmazza a szükséges projektbetöltőket; korábban az `open_project()` hibaüzenetet váltott ki, amelyben az `ImportError` hibaüzenetet említette, az `project_cameras` / `project_sensors`-et bármely pip-tel telepített SDK-ben. Az `connect_all()` a futó háttérrendszeren keresztül csatlakoztatja a mentett kamerákat, tömböket és fényérzékelőket (a fényérzékelőket a DAQ-poolján keresztül), és egy olyan mentett eszköz, amelyet nem tud megkeresni, az `errors` alatt szerepel az ok megjelölésével, ahelyett, hogy szó nélkül kihagyná. Egy olyan mentett tömb, amely már fut a háttérrendszerben (a GUI-k, vagy az `chloros-cli project connect`-é) csatlakoztatva marad és tovább fut, míg korábban hibaüzenet jelent meg („A kamera  <sn>már </sn>szerepel <sn>a tömbben<id>”). Lásd [`ChlorosProject` módszerek](#chlorosproject-methods).
> - **Az `ChlorosLocal()`, az `process_folder()` és az `process_lattice_capture()` megtalálja a Chloros-t Windows rendszeren.** Megkeresik azt a helyet, ahová a telepítő elhelyezi a háttérprogramot (alapértelmezés szerint az `<install folder>\backend\` és az `C:\Program Files\Chloros\backend\` mappát, a telepítő saját bejegyzéséből származó egyéni mappába), és egyáltalán nem keresnek új háttérprogramot, hanem csatlakoznak a már futóhoz. Az 1.2.0-s verzióban „Chloros-háttérprogram nem található” hibát jeleztek a telepített gépen, hacsak nem adtad meg az `backend_exe=` paramétert.
> - **Az indításkor leálló háttérprogramról azonnal jelentés készül.** Amikor az `ChlorosLocal()` vagy egy smart-connect hívás elindít egy háttérprogramot, amely még a válaszadás előtt leáll, a hívás másodpercek alatt hibaüzenetet generál: az `ChlorosBackendError` az `ChlorosLocal`-tól, az `ChlorosConnectError` pedig a smart--connect hívásokból, a kilépési kóddal, a futtatott parancsokkal és a háttérprogram által kinyomtatott utolsó sorokkal együtt. Az 1.2.0-s verzió kivárta a 60 másodperces indítási időtúllépést, de nem jelezte az okát. Lásd [Háttérprogram automatikus indítása](#backend-auto-start).
> - **A háttérprogram a szkriptek között is tovább fut.** Ha kilépünk egy `with ChlorosLocal(...)` blokkból, az `process_folder()` / `process_lattice_capture()` visszatér, és az `chloros-mcp` kilépése már nem állítja le az SDK-val elindított háttérprogramot; a következő szkript újra felhasználja azt. Az `logout()`-nél áll le (hacsak a Chloros-alkalmazás nincs megnyitva, vagy nincs folyamatban futtatás), az `shutdown_backend()`-nél, vagy önmagától 30 perc elteltével, ha semmi sem használja (az `CHLOROS_BACKEND_IDLE_MIN` a háttérprogram környezetében ezt megváltoztatja; `0`: soha).
> - **Az SDK-vel elindított háttérprogramot leállításra kérik, nem pedig megszüntetik.** Az `shutdown_backend()` lehetővé teszi a háttérprogram számára, hogy felszabadítsa kameráit és fényérzékelőit, és befejezze az esetleges felvételeket; csak akkor kerül leállításra, ha 10 másodperc múlva is még fut. Az 1.2.0 verzió azonnal leállította (Windows rendszeren ez erőszakos leállítást jelentett), ami miatt a háttérprogramhoz csatlakoztatott DAQ fényérzékelő nem reagált. Az SDK által futó állapotban talált háttérprogramot továbbra is békén hagyja.
> - **Linux rendszeren az automatikus indítás soha nem futtatja az `backend_server.py` parancsot az aktuális mappából.** Ha nincs telepített háttérprogram, egy olyan mappában elindított szkript, amelyben volt ilyen, futtatta azt; most már csak a telepített háttérprogram indul el (vagy, az `ChlorosLocal` esetében, az, amelyet az `backend_exe=` megnevez).
> - **Az `configure(indices=...)`**minden előre beállított értéket hozzáad a projekthez a kamera sávjaihoz, beleértve egy új projektet is, és figyelmeztet (`UserWarning`), ha van olyan név, amelyet nem tud hozzáadni.**`import_images()`** az `.daq` felvételeket `scan_count` néven jelenti, és már nem jelez figyelmeztetést a LATTICE rögzítési mappával kapcsolatban.
> - **Az `read_image_audit_tags()` a pip install parancsból fut.** Magától olvassa be a kép metaadatait; az 1.2.0-s verzióban az `{}`-et adta vissza anélkül, hogy a Python-útvonalon szerepeltek volna a desktop telepítés belső elemei.
> - **Az `capture()` közvetlenül az `connect_array()` után** legfeljebb 20 másodpercig vár a tömb első képkockacsoportjára, ahelyett, hogy HTTP 503 „Array still warming up” hibával szakadna meg.
> - **Az `status()` leállást jelez.** Az új `stalled` mező True értéket vesz fel, amíg a rögzítési ciklus fut, de nem tesz közzé semmit, és az `actual_fps` ekkor 0 lesz. Az 1.2.0-s verzióban az `actual_fps` megtartotta az utolsó értékét leállás alatt is megtartotta az utolsó értékét, így egy leállt tömb az `actual_fps > 0` értéket mutatta, miközben a rögzítések 503-as „Nincs friss szinkronizált képkockacsoport” hibával buktak meg.
> - **Kevesebb hiányos képkockakészlet és 503-as rögzítési hiba:** minden szolga-kamera trigger bemenete most már zavarmentes.
> - **Tömörített 12 bites adatátvitel** (`connect_array(packed="auto")`) körülbelül egyharmadával növeli a teljes érzékelősorozat képkockasebességét. Lásd [Tömörített 12 bites adatátvitel](#packed-12-bit-transport-packed).
> - **Expozíció az érzékelőtömbön egy ROI alatt:** egy kis ROI már nem korlátozza az egyes elemek expozícióját az érzékelő szabad futási periódusára (4,9 ms 768 × 304 felbontásnál). Lásd [Expozíció az érzékelőtömb elemein](#exposure-on-array-members).
> - **Az automatikus expozíció már nem csökkenti a szenzortömb automatikus képkockasebességét.** A kábelen szinkronizált tömb a hálózat és a kamerák kiolvasási sebességének megfelelően működik; az automatikus expozíció kitölti az adott sebesség által biztosított periódust, majd növeli az erősítést. A periódusnál hosszabb kézi expozíció esetén a rendszer nem utasítja el a kérését, hanem a kioldást a kéréshez igazítja. Lásd [Expozíció a tömb elemein](#exposure-on-array-members).
> - **`read_nodes()`**hozzáadja az `frame_values` és az `notes` értékeket az automatikus expozíció futása közben (lásd [`read_nodes()` automatikus expozíció alatt](#read_nodes-under-auto-exposure--frame_values-and-notes)). Amikor a kamera elutasít egy expozíciót vagy erősítést, az**`set_settings()`**megadja az általa elfogadott tartományt, és visszaállítja az automatikus módot, amelyet az adott értékre kikapcsolt; Az**`set_settings(target_brightness_auto=True)`** új kód.
> - **`orient=`** az `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` és `ArrayHandle.capture()` esetében / `capture_smart()` a kamerának megfelelően elforgatja a termékeket. Lásd [`ArraySession` módszerek](#arraysession-methods).
> - **Az `DLS()`-hez szükség van az érzékelő saját `mac_address`-ére.** Már nincs alapértelmezett MAC-cím (ez korábban egy egység címe volt); az `DLS()` ennek hiányában az `DLSError` hibát eredményez. Lásd [DAQ-M sávmagnitúdók](#daq-m-band-magnitudes--dlsband_magnitudes).
> - **A Windows-telepítő** felajánlja a „Windows hosszú elérési útvonalak engedélyezése” és a „Chloros-háttérprogram kizárása a Microsoft Defender vizsgálatából” opciókat, amelyek alapértelmezés szerint be vannak jelölve. Lásd [1. lépés](#step-1--install-the-chloros-platform-package).
> - **A Linux .deb** telepíti a rendszerhez tartozó Python SDK-t (`python3` (korábban csak a wheel csomagot helyezte el a lemezen), növeli a kernel socket-fogadó puffereit, amelyekre a kamerastreameknek szükségük van, és hozzáad egy háttérszolgáltatást, amely a felhasználó nevében fut. A Jetson-csomag az `chloros_1.3.0_arm64_jp6.deb`, a JetPack 6-hoz, a JetPack CUDA PyTorch-jával. Lásd az [1. lépést](#step-1--install-the-chloros-platform-package) és a [2. lépést](#linux-deb).

---

## Az architektúra 60 másodpercben

Az SDK egy vékony Python-réteg a Chloros-háttér felett (ugyanaz a Flask-szerver, amelyet az asztali GUI és a CLI is használ). Az automatizáláshoz importálja az `chloros_sdk`-et, és hívja meg a magas szintű módszereket; a háttérben minden hívás HTTP-kéréssé alakul a 5000-es porton futó helyi háttérrendszer felé — `http://127.0.0.1:5000/api/...` (szándékosan nem az `localhost`, amely Windows-en először az `::1`-re oldódik fel, és kizárólag IPv4-es háttérrendszer esetén kérésenként ~2 másodpercet vesz igénybe). A háttérrendszer birtokolja a hardverpoolt — kamerák, DAQ-érzékelők, igazítási profilok, képkocka-pufferek — így az SDK-szkriptek a GUI-val párhuzamosan működhetnek anélkül, hogy a soros portokért vagy a hálózati kártya sávszélességéért kellene versengeniük.

Három felületet fogsz használni:

1. **`ChlorosLocal` + szabad függvények** (`process_folder`, `process_lattice_capture`) — Képfeldolgozó folyamat. Egyetlen Python-hívással futtathatja végig egy teljes mappát a kalibráláson, a debayerezésen és az indexexporton.
2. **Smart-connect kezelők** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Tartós háttérmunkamenet megnyitása élő hardverhez. Ugyanaz a „smart-prep” folyamat, mint a grafikus felhasználói felületen: hálózati próba, réteg automatikus kiválasztása, PTP, AE inicializálás, GPIO-trigger konfiguráció.
3. **`ChlorosProject` / `open_project`** — Betölti a mentett projektet (a `cameras.json` + `sensors.json` + `project.json` fájlokat tartalmazó mappát), egyszerre csatlakoztatja a kamerákat, a szenzorcsoportokat és a fényérzékelőket, majd névvel ellátott kezelőkkel vezérli a felvételeket.

Az 1. és 2. felület **automatikusan elindít egy helyi háttérprogramot**, ha még nincs ilyen futó (ugyanazt a csomagban lévő bináris fájlt, amelyet a GUI/CLI indít el) — így egy egyszerű szkript is működik egy új parancssorból anélkül, hogy előbb el kellene indítanod a háttérprogramot: `ChlorosLocal` (és `process_folder()` / `process_lattice_capture()`), `connect_camera()`, `connect_array()`, `connect_daq_sensor()`, `discover_lattice_cameras()` és `discover_daq_sensors()`. Az `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, az `attach_array()` és az `analyze_array_network()` soha nem indulnak el: ezekhez futó háttérprogramra van szükség. Az `auto_start_backend=False` megadásával le lehet tiltani (pl. ha távoli háttérprogramra mutatnak, amely soha nem indul el). Lásd [Háttérprogram automatikus indítása](#backend-auto-start). A Surface 3 másképp viselkedik: az `open_project()` nem fogad el `auto_start_backend` paramétert, az `connect_all()` pedig soha nem indít el háttérprogramot — egyszer megpróbálja lekérdezni az `http://127.0.0.1:5000`-et, és ha nem érkezik válasz, akkor visszatér a közvetlen (háttérprogram nélküli) `lattice_sdk` eszközvezérlésre. Ehhez szükség van az Arena SDK futásidejére és az OpenCV (`cv2`) importálhatóságát a Python-ben, amit az SDK wheel nem telepít; ezek nélkül minden elmentett kamera és tömb hibaként kerül jelentésre, az ok megjelölésével. A projekt fényérzékelői nem rendelkeznek háttérprogram nélküli útvonalakkal: ha nincs futó háttérprogram, minden elmentett érzékelő hibaként kerül jelentésre, az ok megjelölésével (lásd [`SensorHandle`](#sensorhandle)). Csak az `proj.process()` és az `stream(..., overlays=True)` hozza létre késleltetetten egy `ChlorosLocal()`-et (amely automatikusan elindul).

Mindhárom auth-gated: futtassa az `chloros-cli login`-et egyszer a gépen, vagy jelentkezzen be az asztali grafikus felületen keresztül. Hogy egy érvényes munkamenet nélküli hívás milyen hibát eredményez, az a felülettől függ (lásd [Kivételek](#exceptions)): az `ChlorosAuthenticationError` az `ChlorosLocal`-ből (és az `process_folder()` / `process_lattice_capture()`), az `ChlorosConnectError` a smart-connect hívásokból és azok munkameneteiből, az `status: "error"` az `error_code`-ből az `analyze_array_network()`-ből, valamint egy bejegyzés az `errors` alatt az `connect_all()` jelentésben.

Követelmények:
- Python 3.7+ (a csomag leírása szerint; 3.10-es verzión fejlesztve/tesztelve)
- Helyileg telepített Chloros Desktop (a háttérprogram a telepítőben található)
- Aktív Chloros+ bejelentkezés **bármely fizetős Chloros+ csomagban (Copper vagy magasabb)**; az ingyenes**Iron**csomag nem biztosít SDK/CLI hozzáférést. Ezt**szerveroldalon** érvényesül: minden SDK/CLI jelöléssel ellátott kérésnek tartalmaznia kell egy aktív munkamenetet és egy fizetős csomagot. A Copper alatti csomagoknál az `403` hibaüzenet jelenik meg az `error_code: PLAN_UPGRADE_REQUIRED` kóddal (az `ChlorosLicenseError` hibaüzenet az `ChlorosLocal`, `ChlorosConnectError` a smart-connect hívásokból). A kijelentkezett hívó helyette az `401` / `AUTH_REQUIRED` kódot kap (`ChlorosAuthenticationError` az `ChlorosLocal`-ből, `ChlorosConnectError` a smart-connect hívásokból) — a kettő különbözik egymástól, mert az `chloros-cli login` újbóli futtatása kijavítja az `401` hibát, de nem tudja kijavítani az `403` hibát, amelyhez a terv frissítésére van szükség.
- Az offline használat a csomag türelmi időszakán belül támogatott: a szintet a szerver-érvényesítési gyorsítótárból (5 perc) vagy az aláírt, gépre kötött licenc-gyorsítótárból olvassa be a rendszer (havi csomagok esetén 30 napig, éves előfizetések esetén az előfizetés lejáratáig). A türelmi időszak lejárta után a csomag ingyenes szintre vált, és az SDK/CLI-hozzáférés leáll, amíg a gép legalább egyszer el nem éri a szervert. Az `chloros-cli status` (`GET /api/license-status`) hibaüzenet az ingyenes szinten is elérhető marad, így az ok látható.
- Windows 10/11 64 bites, **Ubuntu 22.04 LTS vagy újabb**, vagy Jetson (JetPack 6). Az Ubuntu 20.04**nem** támogatott: az `.deb` függőségei abból származnak, amire a háttérprogram hivatkozik, beleértve az `libc6 (>= 2.35)`-et is, a Focal pedig a glibc 2.31-et szállítja.

---

## Telepítés

A Python SDK egy vékony Python réteg a Chloros háttérprogram felett. Hacsak nem egy másik gépen futtatja a háttérprogramot (lásd [Távoli háttérprogram mód](#remote-backend-mode-pip-only-host-via-tunnel)), akkor **a Chloros asztali csomagot helyileg kell telepítenie** (Windows-telepítő vagy Linux `.deb`), amely tartalmazza a DAQ fényérzékelőket is. A csomag tartalmazza a háttérprogram bináris fájlját, az Arena SDK futásidejű környezetet a LATTICE kamerákhoz, valamint a kalibrációs csomagokat.

Legfrissebb letöltések: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### 1. lépés — Az Chloros platformcsomag telepítése

#### Windows (.exe)

1. Töltse le az `Chloros Installer 1.3.0.exe` fájlt a letöltési oldalról.
2. Futtassa a telepítőt, és kövesse a varázsló utasításait. Az alapértelmezett telepítési útvonal az `C:\Program Files\Chloros\`.
3. Indítsa el a Chloros-t legalább egyszer, és jelentkezzen be a Chloros+ fiókjával.

A telepítő minden hálózati profilban a Windows tűzfalon keresztül engedélyezi a háttérprogramot (`chloros-backend.exe`, semmi más) áthaladását a Windows-tűzfalon minden hálózati profil esetében: UDP bejövő forgalom a kamerastreamek, a PTP, az mDNS és a DAQ-E stream számára, valamint TCP bejövő forgalom kizárólag a helyi alhálózatról a DAQ-E számára, amely a firmware-frissítés során visszakapcsolódik a számítógéphez ([`update_firmware()`](#daq-e-device-settings)). Az eltávolító mindkét szabályt eltávolítja. A háttérprogram HTTP-API-je mindkét esetben az `127.0.0.1`-en marad.

A varázsló **Rendszerbeállítások** csoportjában két opció található, amelyek alapértelmezés szerint mindkettő be van jelölve:

| Opció | Mit csinál | Csendes telepítés kikapcsolása |
| --- | --- | --- |
| **Windows hosszú elérési útvonalak engedélyezése** | Lehetővé teszi, hogy a projektmappák a Windows 260 karakteres elérési útvonal-korlátjánál mélyebben helyezkedjenek el. Ha a hosszú elérési útvonalak ki vannak kapcsolva (ez a Windows alapértelmezett beállítása), a termékekhez képest túl mélyen elhelyezkedő projektmappát a rendszer még a feldolgozás megkezdése előtt elutasítja — lásd [Hibaelhárítás](#hibakeresési-tippek). | `/MERGETASKS="!longpaths"` |
| **A Chloros-háttérprogram kizárása a Microsoft Defender vizsgálatából** | Megakadályozza, hogy a Defender valósidőben történő vizsgálatát minden olyan fájl esetében, amelyet a kamera rögzít. Magas rögzítési sebesség mellett a vizsgálat a sebesség körülbelül 30 %-át veszi igénybe: 9 kamerás rendszerben, 512-es ROI-nál mért értékek szerint a folyamatos nyers rögzítés sebessége vizsgálattal 52,7 kép/másodperc, a háttérprogram kizárásával pedig 77,7 kép/másodperc volt. | `/MERGETASKS="!defenderexclusion"` |

A folyamatos rögzítés a Defender terhelését `defender_cores` kóddal jelenti (lásd [`ContinuousHandle`](#continuoushandle)).

#### Linux amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Telepítse az `apt install ./…`-et, ne az `dpkg -i`-et: az apt a Python SDK követelményeit is telepíti, amelyeket a csomag inkább ajánl, mint előír (lásd [2. lépés](#linux-deb)).

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Jetson Orin / Orin NX / Orin Nano eszközökhöz JetPack 6 (L4T R36) rendszeren. A PyTorch a JetPack 6 CUDA-verziója, így a GPU-feldolgozás az Orin GPU-ján fut; a JetPack 6 által telepített CUDA- és cuDNN-könyvtárakat használja (`sudo apt install nvidia-jetpack` olyan képen, amelyeken ezek hiányoznak).

#### Linux: fogadó pufferek és a háttérszolgáltatás

Mindkét .deb fájl 32 MB-ra növeli a kernel socket-fogadó puffereit, amelyekre a LATTICE kamerák adatfolyamainak feldolgozásához szükség van (`/etc/sysctl.d/60-chloros-gige.conf`). A kernel alapértelmezett beállításainál a kamera csomagjai a kernelben elvésznek, és az array képkockákat és képkockasebességet veszít; az `chloros-cli lattice network` jelzi, ha a pufferek alacsony szintre csökkennek. Lásd az CLI-referencia [Linux-gépek — socket fogadó pufferek](cli-reference.md#linux-hosts--socket-receive-buffers) című részét.

Headless gépen futtassa a háttérprogramot szolgáltatásként **a saját felhasználói fiókjával**, hogy a szkript megtalálja a futó példányt (az `connect_all()` soha nem indít el háttérprogramot):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

A háttérprogram ezután az Ön felhasználójaként fut: az általa írt rögzítések az Önéi, és az Ön `chloros-cli login`-jét használja. A root `chloros-backend.service` továbbra is telepítve van azokon a gépeken, ahol engedélyezték, de minden fájl, amit egy root háttérprogram ír, a root tulajdonát képezi. Lásd az CLI-referencia [A háttérprogram szolgáltatásként való futtatása](cli-reference.md#running-the-backend-as-a-service-headless) című részét.

A szolgáltatás által futtatott háttérprogram megosztott, és az ügyfelek soha nem állítják le: az SDK-parancsfájlok és az `chloros-cli` úgy használják, ahogyan megtalálják (az `ChlorosLocal.shutdown_backend()` csak azt a háttérszolgáltatást állítja le, amelyet az `ChlorosLocal` maga indított el), az `GET /api/status` pedig az egységet `service` néven azonosítja. Indítsa újra az `sudo systemctl restart chloros-backend@$USER.service` paranccsal. A csomag frissítése leállítja a háttérszolgáltatásokat a telepítés során, így egy futó szkript elveszíti a kameráit és érzékelőit, majd minden engedélyezett elemet újraindít; a csomag eltávolítása leállítja őket, és letiltja az engedélyezetteket. Egyszerre csak egy háttérszolgáltatás fut, ezért csak egy egységet engedélyezzen: a második nem indul el, és `failed` állapotban marad állapotban marad (kilépési állapot 98, nem indul újra), és az `systemctl status` üzenet megmagyarázza, miért. Az egységek közötti váltáshoz tiltsa le azt, amelyet nem használ.</id></sn>

### 2. lépés — A Python SDK telepítése

**A Chloros telepítője tartalmaz egy megfelelő SDK wheel fájlt.** Minden Windows-telepítő és Linux .deb fájl egy `chloros_sdk-X.Y.Z-py3-none-any.whl` fájlt helyez el a lemezen, amely pontosan megegyezik a GUI / CLI / háttérprogram verziójával. Nem kell a PyPI-t követned a verziók szinkronban tartása érdekében.

#### Windows

A telepítő automatikusan futtatja az `pip install` fájlt a csomagban található wheel fájl segítségével, a rendszer Python-jét használva (előnyben részesíti az `py.exe` indítót, de visszaesik az `python -m pip`-re). Nincs szükség semmilyen beavatkozásra — az `import chloros_sdk` a sikeres telepítés után az Ön Python-környezetében működik. Ha nincs Python a gépen, a telepítő észrevétlenül kihagyja ezt a lépést, és a GUI + CLI továbbra is működik.

#### Linux (.deb)

Mindkét .deb-csomag (x86_64 és Jetson) telepíti az `python3` rendszerhez tartozó SDK-t, így az `import chloros_sdk` a telepítés után pip és hálózat nélkül is működik. A csomagok (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` és a projektbetöltők) az `/usr/lib/python3/dist-packages`-be kerülnek, az SDK `chloros-mcp` parancsa (az [MCP-kiszolgáló](#mcp-server-ai-assistants)) pedig az `/usr/bin` mappába kerül, mindegyik a csomag tulajdonában van: egy frissítés felülírja őket, a csomag eltávolítása pedig törli őket.

Az SDK saját követelményei, az `python3-numpy` és az `python3-requests`, **ajánlottak, nem kötelezőek** (a GUI, a CLI és a háttérrendszer önállóak): Az `sudo apt install ./chloros_<version>_amd64.deb` (Jetson esetén az `…_arm64_jp6.deb`) telepíti őket, az `dpkg -i` viszont nem. A telepítés kimenete jelzi az `python3` által importált SDK-verziót, az esetleges hiányzó követelményeket, valamint azokat a `chloros_sdk` fájlokat, amelyek az `python3` keresési útvonalán máshol találhatók, és amelyek felülírják a csomag másolatát (például egy korábban az `pip install --user` paranccsal telepített példányt).

Egy virtualenv-ben, amely nem látja a rendszer csomagjait, telepítse bele a csomagban található wheel-t:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Air-gapped Jetson-telepítések esetén ez teljesen offline történik — az SDK és a wheel már a lemezen vannak.

#### Nyilvános PyPI

Kizárólag pip-et használó gazdagépek esetében (nincs telepítve Chloros asztali csomag; az SDK ekkor egy másik gépen futó háttérprogramot vezérel, lásd [Távoli háttérprogram mód](#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

A PyPI a legfrissebb stabil kiadást tartalmazza. Az SDK-t tartsa ugyanazon a verzión, mint az általa vezérelt Chloros-háttérprogramot (lásd [Verziókezelés és kompatibilitás](#versioning--compatibility)). Az `DLS` DAQ-M segédprogramhoz az `bleak`: `pip install "chloros-sdk[camera]"` (vagy `pip install bleak`) verzió is szükséges.

#### Ellenőrzés

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Chloros+ előfizetés szükséges.** Minden SDK-híváshoz aktív Chloros+ bejelentkezés szükséges. Futtassa az `chloros-cli login user@example.com 'YourPassword'` parancsot egyszer minden gépen; a hitelesítő adatok az `~/.chloros/` fájlban kerülnek tárolásra.

### Szükségem van a Desktop Package-re?

A legtöbb munkafolyamat esetében a pip csomag önmagában **nem** elegendő. Az egyes SDK-felületek követelményei a következők:

| SDK-felület | Szükséges a Desktop Package? | Miért |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Igen**(egy elindításához)**/ Nem** (ha már elérhető egy háttérfolyamat) | Futó háttérfolyamatot használ (a desktop alkalmazásét, vagy egy másik elindított szkriptét); ellenkező esetben automatikusan elindítja a telepítettet: `/usr/lib/chloros/chloros-backend` (Linux) vagy `<install folder>\backend\chloros-backend.exe` (Windows; alapértelmezés szerint `C:\Program Files\Chloros\backend\`, az egyéni telepítési mappát a telepítő nyilvántartásából olvassa be). Adja meg az `backend_exe=` paramétert, ha saját nevet szeretne adni neki. |
| `chloros-mcp` ([MCP-kiszolgáló](#mcp-server-ai-assistants)) | **Igen**(elindításához)**/ Nem** (a háttérszolgáltatás már elérhető) | Az `ChlorosLocal`-re épül, így megegyezik a fenti sorral. |
| `read_image_audit_tags` | **Nem** | Beolvassa a feldolgozott kép saját metaadatait (csak a Python-szabványkönyvtárban). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Igen**(helyi)**/ Nem**(távoli) | Tiszta HTTP-kliensek a háttérrendszeren keresztül. Helyi háttérrendszer → asztali csomag szükséges. Távoli háttérrendszer → `backend_url=`**alagúton keresztül** (lásd: Távoli háttérrendszer mód — a mellékelt háttérrendszerek csak a loopback-hez kapcsolódnak). |
| `ChlorosProject` / `open_project` | **Igen** | A mentett projekteket a háttérrendszeren keresztül futtatja. |
| Közvetlen LATTICE osztályok (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Igen** | Szükség van az Arena SDK futtatókörnyezetére, amely a desktop csomagban található, valamint az OpenCV-re (`cv2`) a Python környezetben, amelyet sem a desktop csomag, sem az SDK nem telepít (`pip install opencv-python`). Az `CAMERA_AVAILABLE` egyébként importáláskor `False`-ként jelenik meg. Az `DLS`-hez az `bleak` is szükséges. |

### Távoli háttérmód (csak pip-es gazdagép, alagúton keresztül)

> **A telepített háttér nem érhető el a LAN-on keresztül.** Csak a
> loopback-en hallgatózik (`127.0.0.1` / `::1`), így az `backend_url="http://<lan-ip>:5000"`
> **nem éri el a telepített Chloros-t**. Ha egy másik
> gépen lévő háttérprogramot szeretne vezérelni, saját maga továbbítsa annak loopback-portját, és az SDK-t az
> alagútra irányítsa:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

A headless / CI / robotikai gazdagépek esetében egy gépet tarthatnak meg a teljes asztali telepítéssel, mint „Chloros-kiszolgálót”, a többi helyen pedig az `pip install chloros-sdk`-et — de a köztük lévő átvitel a fenti, felhasználó által beállított alagút, nem pedig közvetlen LAN-URL.

> **`ChlorosLocal` egy kizárólag pip-et futtató gazdagépen.** Az `ChlorosLocal(backend_url=BACKEND)` olyan háttérrendszerhez csatlakozik, amelyet asztali telepítés nélkül is elérhet: csak akkor keresi a helyi háttérrendszer bináris fájlját, ha el kell indítania egyet (nincs válasz, és `auto_start_backend=True`). Az 1.2.0-s verzióig először a bináris fájlt kereste, és `ChlorosBackendError` („Chloros-háttérprogram nem található…”) hibát jelezett egy kizárólag pip-et futtató gépen, még akkor is, ha a háttérprogram elérhető volt. Az átadott mappákat (`process_folder`, `import_images`) ezen a gépen kerülnek ellenőrzésre, és a háttérprogram olvassa be őket, ezért távoli háttérprogram esetén mindkét oldalon ugyanazt a mappát kell megadni, például egy megosztott meghajtót.

### Csak DAQ-ra korlátozott munkafolyamat (csak pip-et futtató gazdagép)

Az `pip install chloros-sdk` önmagában nem képes vezérelni egy DAQ-érzékelőt: a DAQ-érzékelőt mindig egy Chloros-háttérrendszeren keresztül vezérlik az `connect_daq_sensor()` segítségével:

- **Érzékelő ezen a gépen:** telepítse ide a Chloros asztali csomagot. Az `connect_daq_sensor()` elindítja a háttérprogramját, ha még nem fut.
- **Érzékelő egy másik gépen:** arra a gépre is szükség van a desktop csomagra; a háttérprogramot a [Távoli háttérprogram mód](#remote-backend-mode-pip-only-host-via-tunnel) alatt leírt alagúton keresztül érheti el. A háttérprogram megnyitja az érzékelőt a saját gépén, tehát egy DAQ-U csatlakozik ahhoz a géphez, egy DAQ-M a Bluetooth-hatótávolságán belül van, és egy DAQ-E pedig egy olyan hálózaton van, amelyet elérhet.

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Bármelyik esetben is, a háttérrendszer érvényesíti a „Chloros+” tervet: jelentkezzen be a háttérrendszert futtató gépen (`chloros-cli login` vagy az asztali alkalmazás) bármely fizetős csomaggal (Copper vagy magasabb). A kijelentkezett háttérrendszer és a Copper alatti csomagok egyaránt `ChlorosConnectError` hibát generálnak, amely tartalmazza a háttérrendszer hibaokát (`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`).

---

## Gyors útmutató

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Legfelső szintű API-index

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## Képfeldolgozás — `ChlorosLocal`

A fő folyamatkezelő osztály. A létrehozásakor elindítja a háttérrendszert (ha egyik sem válaszol, és `auto_start_backend=True`), létrehozza és konfigurálja a projekteket, figyelemmel kíséri a folyamatot, és futás utáni összefoglalókat ad vissza.

### Konstruktor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Módszerek

| Módszer | Leírás |
| --- | --- |
| `create_project(project_name, camera=None)` | Új projekt létrehozása (opcionálisan egy kamerasablonnal, például az `"Survey3N_RGN"`-szel). |
| `open_project(project_path, relink=None)` | Megnyit egy mentett projektet (a projekt mappáját vagy a nevét a projektek mappájában), és azt nyitott projektté teszi. Az áthelyezett projekt elérési útjait átirányítja a projekt jelenlegi helyére; az `relink` megadja annak a mappának a nevét, amelyben a hiányzó fájlok jelenleg találhatók. Visszaadja a háttérprogram válaszát az `source_files` segítségével; `ChlorosProjectFilesMissingError` hibát dob, ha egyik sem található. Lásd az alábbi **Áthelyezett projekt** részt. |
| `import_images(folder_path, recursive=False)` | RAW/TIF/JPG/DNG képek **és `.daq` fényérzékelő-felvételek** importálása. Visszaadja az `count` (képek) és `scan_count` (felvételek) értéket ad vissza. Csak akkor jelez figyelmeztetést, ha a mappában egyik sem található. |
| `export_light_sensor(daq=True, csv=True)` | A projekt minden fényérzékelő-felvételéhez kalibrált `.daq` + `.csv` fájlokat ír az `<project>/Light Sensor/` fájlba. Lásd [Fényérzékelő-felvételek](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Állítsa be a feldolgozási csúszkákat. Az `indices` előre beállított nevek (`NDVI`, `GNDVI`, `OSAVI`, …), amelyek mindegyike a projektbe került a kamerája sávjai alapján; egy olyan név, amely nem előre beállított, vagy amelyet a sávok nem tudnak létrehozni (az `NDRE`-hez RE és NIR szükséges, amit egyetlen kamera sem biztosít), az `UserWarning` nevet kapja, és nem rendelkezik indexképpel. Az `daq_log_path` egy fényérzékelős felvételt jelöl (egy `.daq` vagy `.csv` fájl, vagy ezekből álló mappa), amelyhez minden képkockát összehasonlítanak – mind a LATTICE, mind a Survey3 esetében –, a képekkel együtt importált felvételek helyett; ha egy elérési út hiányzik vagy nem tartalmaz felvételt, akkor az importált felvételeket használja. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Futtatja a feldolgozási folyamatot. Visszaadja az `{"status": "complete", "async": False}` fájlt, valamint egy `summary` kulcsot, ha a háttérrendszer biztosít ilyet — lásd [Futtatás utáni összefoglaló és tippek](#post-run-summary--hints) — `export_check_failure`-et, ha a futtatás exportjait nem lehetett ellenőrizni, és `image_failure`-et, ha egy kép elveszett. Az `wait=False` egyszerre adja vissza az `{"status": "started", "async": True}`-et egyszerre. |
| `get_config()` / `get_status()` / `status()` | Ellenőrizze a háttérrendszer állapotát. |
| `logout()` | Kijelentkezés: törli a gyorsítótárban tárolt hitelesítő adatokat, és felszabadítja a gép eszközhelyét (az eredményben: `device_deregistration`). A folyamatban lévő feldolgozási futtatás nem áll le: abban a módban fejeződik be, amelyben elindult (az eredményben: `run_in_progress: true` és `run_mode`), és a következő futtatás a szokásos feldolgozást használja. |
| `shutdown_backend()` | Leállítja a háttérprogramot, ha ezt az adott példány indította el; a már futó háttérprogramot viszont nem érinti. Egy `with` blokk meghagyása vagy az példány garbage-collected állapota nem állítja le a háttérprogramot: a háttérprogram tovább fut a következő szkript számára, és 30 perc elteltével, ha már senki sem használja, magától leáll. A háttérprogramot leállításra kérik, ezért először felszabadítja a kameráit és fényérzékelőit, és befejezi az esetleges felvételeket; csak akkor kerül leállításra, ha 10 másodperc múlva is még mindig fut. |
| `discover_cameras()` | LATTICE kamerák felderítése **ezen példány háttérprogramján keresztül** (`/api/camera/discover`). Szótárak listáját adja vissza (`serial`, `model`, `ip`, …) — ugyanolyan szerkezetű, mint amit a GUI/a CLI lát. Üres lista, ha nem található, vagy a háttérprogram nem érhető el. |
| `camera_capture(output_dir, format="tiff", **settings)` | Egyetlen képkocka rögzítése**a háttérrendszeren keresztül**(ezt a kezelő automatikusan elindítja), így ugyanazt az előkészítést kapja, mint a GUI/CLI (alapértelmezés szerint 12 bit, pool újrafelhasználás, beágyazott kalibrációs metaadatok). A célpontot az `serial=` vagy az `device_index=` parancsokkal kell feloldani; adja át az `exposure`/`gain`/`pixel_format`/`preset` értékeket az `**settings`-nek. Visszaadja a régi metaadat-szótárat (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Összeállított előnézeti képkockák előállítása egy összevont kamerából — vékony MJPEG-kliens a háttérrendszer `/api/camera/<serial>/stream-annotated` útvonalán keresztül (zebra / rács / célkereszt / hisztogram / csúcsjelzés / pont, amelyeket szerveroldalon rajzolnak). Az `decode=True` BGR-tömböket ad vissza, és OpenCV-re van szüksége (`cv2`), amelyet az SDK nem telepít; az `False` nyers JPEG-bájtokat ad vissza. Projekt-szinten is elérhető az `ChlorosProject.stream(overlays=True)` néven. |

**A futás közbeni folyamat megtartja a projektjét.** Amíg egy feldolgozási futtatás folyamatban van — legyen az a szkripté, az alkalmazásé, az `chloros-cli process`-é vagy az `chloros-mcp`-é —, a háttérrendszer nem engedi létrehozni, megnyitni, importálni vagy módosítani egy projekt feldolgozási beállításait (HTTP 409 `PROCESSING_IN_PROGRESS`): az `create_project()` és az `import_images()` az `ChlorosProcessingError` kivételt dobja, az `configure()` pedig az `ChlorosConfigurationError`-et váltja ki, az `process()` pedig – ha a futó módtól eltérő módban van – az `ChlorosProcessingError`-et váltja ki (egy futtatás megtartja azt az üzemmódot, amelyben elindult), mindegyik a háttérrendszer üzenetével. A projektbe mentett rögzítés ezután csatlakozik hozzá, amikor a futtatás befejeződik (ezt jelzi a válaszában szereplő `project_add`). Várjuk meg a futtatás befejezését (`get_status()`), majd folytassuk.**Áthelyezett projekt.** A projekt `project.json` fájlja minden forrásfájlt (minden képet és fényérzékelő-felvételt) a teljes elérési útjával nevez meg, így a projektmappa – vagy az egész projects mappa – másik meghajtóra történő áthelyezése esetén a fájlok továbbra is a régi helyet jelölik meg. Minden megnyitáskor a rendszer önállóan kijavítja, amit tud: a projektmappában korábban található fájlokat a projekt jelenlegi helyén lévő mappában keresi, és az ott talált elérési utakat átirányítja, majd elmenti (az `project.json`-et először `project.json.before-relink-<date>-<time>` néven mentik). Az elérési út átirányítása mindig csak olyan fájlra történik, amely ott is megtalálható. A projektmappa kívül található fájlokat – például egy hub-munkamenetet vagy egy „Mappa hozzáadása” importot – csak az `relink=<the folder they are in now>` (vagy bármelyik felette lévő mappa) segítségével lehet megtalálni. Az `open_project()` az `source_files`-et adja vissza: `total`, `found`, `missing`, `missing_roots` (azok a mappák, amelyekben a hiányzó fájlok voltak), `missing_outside_project` és `relink` (mit irányítottak át, honnan hová). Egy olyan projekt, amelyből néhány fájl hiányzik, a többi fájllal együtt megnyílik; az, amelyből egyet sem találtak, nem nyílik meg, az `open_project()` pedig az `ChlorosProjectFilesMissingError` hibát váltja ki (`.missing_roots`, `.audit`; HTTP 409 `PROJECT_SOURCES_MISSING`). A CLI `project open --relink` és `chloros-mcp``open_project` (`relink_folder`) kódjai is ugyanezt teszik.

Használja kontextuskezelőként a garantált tisztítás érdekében:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Fényérzékelő-felvételek — kalibrált `.daq` + `.csv`

A DAQ-U / DAQ-M / DAQ-E készülékek **a**kalibrációs csomag**nélkül** is rögzíthetők. Ez az, amit
a nyilvános [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
felvevők (`record_daq.py`) alapértelmezés szerint teszik: a nyers érzékelőértékeket írják ki, és időbélyeggel látják el a
fájlt, így a Chloros **sorozatszám alapján** letölti az adott érzékelő gyári kalibrációját — először a helyi gyorsítótárból, majd a MAPIR Cloudból — és az importáláskor alkalmazza azt.

A Chloros az eredményt felvételenként két termékként írja vissza, az
`<project>/Light Sensor/` alatt:

| Termék | Mi ez? |
| --- | --- |
| `<name>_calibrated.daq` | Az újrafeldolgozható archívum — ugyanaz a sémája, mint egy élő felvételnek, most már feltünteti azt a csomagot, amely létrehozta. Újbóli importálása **nem** eredményezi a második kalibrálást. |
| `<name>_calibrated.csv` | Spektrális besugárzás W/m²/nm-ben az érzékelő saját hullámhossz-rácsán, egy sor minden leolvasáshoz, valamint fotometrikus oszlopok (teljes teljesítmény, fotopikus/skotopikus lux, PPFD és annak kék/zöld/vörös bontása, csúcshullámhossz). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Kizárólag csomag nélküli érzékelők (DAQ-A).** Nyers spektrális érzékelőimpulzusok — *nem* besugárzási erősség. Lásd alább. |

`<name>` a felvétel saját fájlneve. Két, különböző mappákban található felvétel megoszthat
egy — felvételenkénti mérési értéket (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`), amelyet két kamera
ugyanazon milliszekundumban mentett — és mindkettő megkapja a saját kimeneti adatait: az első megtartja
az `<name>`-et, a következő pedig **a név után** hozzáadja azokat a mappákat, amelyek megkülönböztetik
(`<name>__<capture>_daq_calibrated.daq`), így a felvétel dátuma és ideje továbbra is az első helyen szerepel.

Az `process()` ezt az exportálást az egyik lépéseként hajtja végre. Ehhez **nincs** szükség képanyagra:
egy önállóan repülő fényérzékelő is elsőrendű munkafolyamatot jelent, és egy ilyen projektnek
konstrukciója miatt nincs képe.

**A DAQ-A felvételek nyers számadatokként kerülnek exportálásra.** A DAQ-A család a sorozatszámon alapuló
csomagrendszer bevezetése előtt jelent meg, és nincs letöltendő csomagja — helyette a terepen egy
reflektancia-célponttal kalibrálják, ezért soha nem is volt rá szüksége. Ezek a felvételek
`_raw` kiterjesztéssel exportálódnak, nem pedig `_calibrated`-szel: egy eltérő fájlnévvel, nem pedig a fájl
belsejében található jelzővel, mivel a jelölésnek meg kell maradnia, ha puszta névként továbbítják e-mailben. Az
`.csv` fejléc `raw spectral sensor counts (NOT irradiance)`-et jelöl, és figyelmeztet, hogy az
értékek **a** fájlon belül összehasonlíthatók – pontosan erre szolgál a célalapú kalibrálás
– és nem az érzékelők között. A teljesítményfüggő fotometrikus oszlopok (teljes teljesítmény,
fotopikus/skotopikus lux, PPFD) **NULL** értéket adnak vissza, a számlálásokból történő integrálás helyett.

Azokat a DAQ-U / DAQ-M / DAQ-E eszközöket, amelyek csomagja egyszerűen nem tölthető le, a rendszer továbbra is **kihagyja**,
nem írja ki nyers formában: ilyen esetben a csomag létezik, és a „újracsatlakozás és újrafeldolgozás” valóban jó tanács.

A régi **v1.01 / v1.02** felvételek (ezeket a DAQ-A-SD írja) nem tartalmaznak olvasásonkénti korszakot,
csak a fájl írási idejét. A kép↔lefelé irányuló sugárzás-egyeztető továbbra is elutasítja őket — egy
képkocka és az írási idő összehasonlítása láthatatlanul hibás lenne — de az exportáló program beolvassa őket, és a
CSV-fájl kinyomtatja az `clock=daq_created_on` kódot, így a termék jelzi, hogy melyik órára vonatkozik.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Azokat a felvételeket, amelyek kalibrációs csomagja nem tölthető le (offline állapotban, vagy olyan érzékelő esetén, amelynek
nincs kalibrációja a fájlban), az `skipped` kód alatt jelenti **az ok megjelölésével**. Soha nem
íródik ki „kalibrált” fájlként, amely nyers számadatokat tartalmaz – csatlakozzon az internethez, és
futtassa újra a műveletet, és az export befejeződik.

### Haladásról szóló visszahívások

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Futás utáni összefoglaló és tippek

A befejezés után az `process()` lekérdezi az `GET /api/processing-summary` fájlt, és a testét `result["summary"]` néven csatolja. A lekérdezés a legjobb erőfeszítés alapján történik, és soha nem blokkolja a sikeres visszatérést — ha az összefoglaló nem érhető el, az `process()` visszatér az egyszerű `{"status": "complete", "async": False}` formához. Az `summary["hints"]` minden bejegyzése — teljes mondatok a javasolt javítással, pl. miért nem eredményezett kimenetet egy futtatás — szintén Python-formátumú `UserWarning`-ként kerül újrakibocsátásra, így a nulla kimenetű futtatások önellenőrzőek, még akkor is, ha soha nem vizsgálod meg a szótárat:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` a géppel olvasható rész:

| Kulcs | Mit számol |
| --- | --- |
| `models` | A futtatásban szereplő kameracsoportok. |
| `images_in_groups` | Az említett csoportokhoz tartozó forrásképek. |
| `targets_found` | Észlelt reflektancia-célpontok. |
| `images_calibrated` | Azok a képek, amelyekhez a futtatás reflektancia-terméket írt. |
| `images_failed` | Azok a képek, amelyek sikertelenek voltak és nem kerültek exportálásra. |
| `exported_files` | **A futtatás által írt képtermék-fájlok**, amelyeket a futtatás létrehozott**: minden termék (debayered, előnézet, sugárzás, fényvisszaverődés, index, célrenderelések), minden formátumban. |
| `reflectance_files` | A köztük található fényvisszaverődési termékek (`Reflectance_Calibrated_Images`), függetlenül attól, hogy célkép vagy a fényérzékelő (DAQ) sugárzási intenzitása kalibrálta-e őket. Ha egy célkép nélküli futtatás mégis reflektancia-adatokat írt, ezt egy tipp jelzi; csak az olyan futtatásnál jelenik meg hibaüzenet, amelyik egyáltalán nem írt ki reflektancia-adatokat. |
| `clamp_masks_written` | Az [unmix clamp maszkok](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) – kiegészítők, nem termékek, ezért nem szerepelnek az `exported_files`-ben (a CLI is ugyanúgy külön számolja őket). |
| `captures_left_out` | A projekt `captures` mappájában található nyers képkockák, amelyeket ez a futtatás nem rögzített. Egy futtatás indításkor megvárja az előtte rögzített adatokat, így ez az érték 0, kivéve, ha a futtatás során rögzítést mentettek, vagy lejárt a várakozási idő. Az `summary["captures_left_out_note"]` tartalmazza, hogy mit kell velük tenni. |
| `daq_recordings_exported` / `daq_recordings_skipped` | Fényérzékelő-felvételek, amelyeket szándékosan külön számolnak — ezek egy másik szakaszból származnak, és olyan futásoknál is léteznek, ahol egyáltalán nincs kép, így ha összevonjuk őket, akkor egy kizárólag adatgyűjtésre szolgáló futás úgy tűnne, mintha képeket is exportált volna. |

Ezek mellett: `summary["output_dirs"]` (minden írt könyvtár),
`summary["light_sensor_export"]`, `summary["stopped"]` (igaz, ha a felhasználó megszakította a
futást, így a részleges számlálások nem jelzik, hogy a futtatás befejeződött, de kevesebb eredményt hozott), és
`summary["groups"]` (a csoportonkénti bontás).

Az `exported_files`-et a folyamat **az írás közben** rögzíti, nem pedig utólag a
projekt képobjektumaiból olvassa ki. A párhuzamos és a GPU-stratégiák saját képobjektumokat
hoznak létre (a GPU-útvonalak esetében a munkavégző alfolyamatokban), így a régi beolvasás minden ilyen futtatásnál
`0 file(s) written`-et jelentett minden ilyen futtatásról, majd kiadta a nulla exportra utaló jelzést — olyan futtatásoknál is,
ahol minden rendben működött. Ha ezt a számot használja a szkriptjében, egy hibátlan párhuzamos futtatás most már
nem nulla értéket jelent.

A kihagyott fájlokról szóló jelentések a leolvasó által az egyes fájlok esetében ténylegesen megállapított okot –
olvashatatlan sémát, hiányzó csomagot, írási hibát – **duplikációmentesen** jelzik, így ha húsz fájlt
egy ok miatt hagytak ki, az egy okként jelenik meg, nem pedig húsz ismétlésként.

> **Az `process()` hibaüzenet nem jelenik meg, ha egy futtatás nem eredményez képeket.** Ez az egyik olyan pont, ahol az SDK és
> a CLI szándékosan eltér egymástól: az `chloros-cli process` a „termékeket kérték, de egyet sem
> írtak” helyzetet hibaként kezeli, és nem nulla kóddal lép ki, míg az SDK normálisan visszatér, és az
> állapotot az `summary` /hints segítségével jelenti. Ha a feldolgozási folyamatnak üres futás esetén le kell állnia, ellenőrizd
> magad – vizsgáld meg az `summary` (vagy számold meg a projektmappa alatti fájlokat), ahelyett, hogy
> a kivétel hiányára támaszkodnál. A szokásos okok a következők: egy bemeneti mappa, amelyet nem ismertek fel
> rögzítésként, valamint a jelen lévő kamerákra nem alkalmazhatónak minősített, kihagyott termékek (pl. kizárólag RGB-ből származó
> kamerák sugárzása).
>
> Az `process()` sem lép fel, ha egy futtatás exportált adatai nem ellenőrizhetők: figyelmeztetést ad és beállítja az
> `result["export_check_failure"]` értéket. A termékek leírásra kerülnek. Ha le szeretnéd állítani a folyamatot, ellenőrizd ezt a kulcsot. Ugyanez vonatkozik egy olyan futtatásra is, amelynél
> egy kép elveszett: figyelmeztetést ad ki, és beállítja az `result["image_failure"]` értéket, amely esetén az `chloros-cli process`
> nem nulla értékkel lép ki.

### Kényelmi függvények

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

Mindegyik a saját `with ChlorosLocal(...)`-én belül fut; az általa elindított háttérfolyamat tovább fut, amikor visszatér, a következő hívásig, majd 30 perc elteltével, ha senki sem használja, magától leáll. Egyik sem fogadja az `auto_start_backend`, `radiometric_output` vagy `daq_log_path` paramétereket; ezekhez használja az `ChlorosLocal`-et és az `configure()`-et használja ezekhez. Az `process_lattice_capture()` nem fogadja az `camera`, `debayer` vagy `ppk` értékeket: kihagyja a panelfelismerést, és a standard debayert használja.

### Támogatott értékek

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Radiometrikus kimenet (LATTICE multispektrális feldolgozási folyamat)

Az `process` feldolgozási folyamat LATTICE multispektrális (M3C/M3M) exportszintje — `reflectance` (alapértelmezett), `radiance`, `sensor-response` vagy `all` (minden alkalmazható mód képenként) — beállítja a projekt **„Export radiance”**és**„Export reflectance”**** termékeit: az `radiance` csak a sugárzást exportálja, az `reflectance` csak a visszaverődést, az `sensor-response` egyikét sem (csak a debayered és az előnézeti termékek), az `all` mindkettőt. A debayered és az előnézeti termékek nem a szint feladata megváltoztatni. Az `configure()`-nek erre van egy külön kulcsszava:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

A speciális menekülési út – a termékek `custom_settings`-en keresztül történő írása – továbbra is működik, de ne feledje, hogy ez a teljes beállítási blokkot felülírja (lásd az alábbi figyelmeztetést). Írja be az `"Export <product>"` kulcsokat: egy projekt saját `"Export radiance"` / `"Export reflectance"` beállításai felülírják a régebbi `"Radiometric output"` kulcsot, így az a kulcs önmagában semmit sem változtat.

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (az alapértelmezett) a kamera sugárzási intenzitását osztja az **időbélyeggel egyező DAQ lefelé irányuló sugárzással**, amelyet automatikusan határoz meg egy rögzített `.daq` (DAQ-U/M/E)**vagy a képek mellett található DAQ-M natív `.csv`**értékből; a helyileg hiányzó kameránkénti vagy DAQ-kalibrációs csomagokat az első használatkor**automatikusan letölti az AWS-ből**. A CLI ezt a-típusú termékkapcsolókként az `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.**Kinek a leolvasási értéke ez?**Egy `.daq` rögzítés által mentett érték az általa leírt kamera(k) leolvasási értéke — amelyet a mellette található `<file>.daq.cameras.json` mellékfájl nevez meg (egyszeri, szinkronizált és tömbös rögzítések; a nyers sorozatfelvételek a manifesztben megnevezik az egyes elemek felvételét) — és**kizárólag azokhoz a kamerákhoz tartozó képekhez** illeszkedik. Egy önálló kamera, amelynek nincs saját felvétele, ezért már nem veszi át egy másik kamera DAQ-ját; rendeljen hozzá egy DAQ-t, vagy rögzítsen egyet. Egy DAQ saját felvétele (`chloros-cli daq pool-record`, vagy a grafikus felhasználói felületen indított felvétel), illetve bármely, az említett sidecar-fájl nélküli `.daq` (beleértve a régebbi rögzítéseket is) továbbra is minden kamerát kiszolgál.**Napfényben lévő OCN-kamerák.** Minden OCN-kalibrációs csomagot keskeny sávú fényben állítanak össze, sávonként egy LED-del, míg a napfény keveri a sávokat. A Chloros 1.3.0 a sávok szétválasztása után egy gyors napfény-alapú korrekciót alkalmaz, ami javítja az OCN sugárzási, visszaverődési és indextermékeit napfényben. A korrekció ott fut, ahol a sávok szétválasztása is – az `process()`-ben, a felvételeknél, az `grab()` / `frame_stream()`-ben és az élőképen – az ezt támogató OCN-csomagok esetében; az RGN-kameráknál nincs változás. A korábbi verzióval rögzített OCN-képek is megkapják ezt a korrekciót, ha olyan számítógépen dolgozzák fel őket, amelyen megtalálható a felvételkészítéskor használt kalibráció (a Chloros megőrzi a kamera kalibrációját, miután a kamerát csatlakoztatták); ellenkező esetben a képeket a korábbi módon dolgozzák fel. Az OCN sugárzási, visszaverődési és indexértékei ezért eltérnek az 1.2.0-s verzióétól.

> Az `custom_settings` **felváltja** a teljes számított beállítási blokkot (tervezés szerint megkerüli az `configure()` többi kulcsszavát és érvényesítését). Használatakor vegyen fel minden olyan `Project Settings` kulcsot, amely fontos Önnek, a fenti példához hasonlóan.

#### Indexpixelek olvasása (az NDVI és az egyéb vegetációs indexek)

**Egy egész számú indexraszter kép-specifikus kontrasztnyújtás, nem pedig −1…1-es leképezés.** Az `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` és `JPG (8-bit)` indextermékek — beleértve a 8 bites TIFF-fájlt is, amelyben a 8 bites felvétel tárolódik — az

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

ahol az `IndexMin` / `IndexMax` **az adott képkocka saját legalacsonyabb és legmagasabb véges indexértékei**. A DN 0 nem −1, a teljes skála pedig nem +1.

A Chloros mindkét végpontot megjelöli, így a leképezés inverz marad:

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

> **Ha nincs `Chloros:Index*` címke, az azt jelenti, hogy nincs megadott leképezés.** A LUT-export egy színezett renderelés, amelynek képpontjai a LUT küszöbérték-ablakához igazított színátmenetek, és egy olyan képkockának, amelyben nincs véges indexértékű képpont, nincs megadható tartománya. Semelyik esetben se feltételezz leképezést — ugyanaz a szabály, mint a hiányzó `PixelScale` esetében.**Mivel a nyújtás képenkénti, egy repülés két képkockája nem összehasonlítható DN-ként**— mindegyik a saját végpontjait hordozza, így ugyanaz az NDVI különböző képeken különböző DN-értékeket eredményez. Repülésen átívelő kvantitatív munkákhoz (zónás statisztikák, rögzített küszöbértékek, idősorok) exportálja a**`TIFF (32-bit, Percent)`**fájlt: az indexrasterek ott float32 típusú**indexértékekként** kerülnek rögzítésre, átméretezés és címkék nélkül. A formátum neve ellenére az indextermékek nem ×100-asok — ez a méretezés csak az egész szám típusú reflektanciára vonatkozik.

#### Unmix clamp maszk — mely pixeleken korlátozták egy sávot

A 3×3-as spektrális szétválasztás negatívvá teheti egy sávot — ott, ahol a fény alig tartalmazza azt a sávot, vagy ahol a jelenet kívül esik a kamera szétválasztási mátrixa által leírt tartományon. Mivel ez nem valós érték, az export 0-ra rögzíti — és az élő nézet is ugyanazt a 0-t jeleníti meg. Egy sáv néhány képponton vagy szinte az összes képponton korlátozódhat; a maszk mindkét esetet rögzíti. Az `Chloros:Band{R,G,B}UnmixClampFraction` megadja, hogy a képkocka mekkora részén fordult ez elő; a korlátozó maszk pedig azt, **melyik képpontokon**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— egy `clamp_masks/` mappában**a termék mappája mellett**, soha nem benne (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`) — minden sugárzási, visszaverődési és indextermék esetében, amelyben legalább egy pixel korlátozásra került. Egycsatornás `uint8`, a termék saját pixelrácsán (tartalmazza a tájolást, ROI, binning és a tömb igazítási torzítás is benne van). Minden maszk megnevezi a termék kameráját (`Make` / `Model` / sorozatszám), a felvétel idejét és az objektív címkéit, valamint felsorolja azokat a sávokat, amelyeket a bitjei jelölnek (`XMP-Camera:BandName`, 0. bit elsőként); az `ImageDescription` és `Software` jelölésekkel rögzített maszknak minősül, és nem tartalmaz kalibrációt.
- **Bitek**: 0. bit = a termék R-csatornájának sávja (`Chloros:BandRName`), 1. bit = G (`BandGName`), 2. bit = B (`BandBName`). A beállított bit azt jelenti, hogy az adott sávban a pixel értéke 0-ra lett korlátozva — ezt, valamint az abból épített bármely indexet alacsony megbízhatóságúnak kell tekinteni.
- **`Chloros:UnmixClampMask`** jelölés szerepel minden olyan terméken, amelyen elvégezték a színszétválasztást: a maszk elérési útja a termék saját mappájához viszonyítva (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, mindig `/`-szal elválasztva — csatlakoztassa a kép mappájához a maszk megnyitásához), vagy `none`, ha egyetlen pixel sem lett korlátozva. A címke hiánya azt jelenti, hogy a termék soha nem futtatta le az unmix műveletet (RGB- vagy monokróm kamerák, nyers/debayered szintek).
- A rögzítési eredmények a képkocka **`clamp_mask`**-ként hordozzák (a maszk elérési útja, vagy `None`, ha semmi sem lett korlátozva) — az `CameraSession.capture()` eredményekben az `ArraySession.capture()``CaptureResult` képkockákban és az HTTP-rögzítési válaszokban. Az offline összeállítás a maszkokat `clamp_masks` néven sorolja fel.
- Egy élő index többsávos veremének (`/api/camera/cli-array-index` és `save_multiband`) forráskameránként egy maszkja van, a kimeneti mappa saját `clamp_masks/` mappájában — lásd az CLI-referenciát`lattice index`. A videó nem tartalmaz maszkot: az offline összeállítás eredménye helyette az egyes képkockák korlátozási arányát jelenti.

Mivel egyetlen termékmappa sem tartalmaz maszkot – semmilyen szinten –, a termékmappa – az almappákkal együtt – változatlan formában átadható a fotogrammetriai szoftvernek, amely kizárólag a képeket veszi figyelembe. Adja át neki a termékmappát, ne a fölötte lévő mappát. Az Chloros saját importálója kihagyja a maszkokat.

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## Smart-Connect a LATTICE kamerákhoz

Tartós háttérmunkamenetek élő hardverhez. Ugyanazok a végpontok, amelyeket a grafikus felület is használ, így a viselkedés azonos az SDK-ben, a CLI-ben és a grafikus felületen.

### Egyetlen kamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

Az itt beállított expozíció, erősítés és célértékek a felhasználó beállításainak megfelelően maradnak: a Chloros automatikus expozícióvezérlése (beleértve a DAQ-mérő által mért expozíciót is) nem módosítja őket. Az `cam.set_settings(exposure_auto="Continuous")` (vagy `gain_auto=`) visszaadja az expozíciót (erősítést), az `cam.set_settings(target_brightness_auto=True)` pedig a célt visszaadja a Chloros automatikus célbeállításának.

#### `connect_camera()` Aláírás

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

- **A stream elindulásával tér vissza.** A háttér várakozik, korlátozott ideig (kb. 15 másodpercig, amíg a stream elindul, majd 10 másodpercig, amíg az első teljes puffer elkészül – nem az első előnézeti képkockáig, amely kb. 1,5 másodperccel később következik), így közvetlenül azután, hogy `status()`-et mond, `streaming`-et mond. A munkamenet `connect_stream`-e a válasz `stream` blokkja: `state` `streaming`, `no_frames` (az adatfolyam elindult, de semmit sem továbbított — ezt jelzi az `note`, és egy jumbo-csomagokat használó kamera esetében 1500 bájtos csomagmérettel kell újra csatlakozni), `starting` vagy `start_failed`, ahol `waited_s`. `None` egy már csatlakoztatott kamera esetében. Egy előnézeti (`get_camera_preview`, `/api/camera/jpeg/<serial>`) kérésére a rendszer várakozás után (4 másodpercig) a kamera első előnézeti képkockáját küldi, ahelyett, hogy a „Még nincs képkocka”; egy olyan kamera, amelynek adatfolyama még nem szállított semmit (`no_frames`, vagy hardveres kiváltásra beállítva), továbbra is azonnal válaszol.
- **A csomagméretet Ön állíthatja be.** Az `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}` 1500 bájtos csomagokkal nyitja meg a kamerát, anélkül, hogy előzetesen felmérné (az `chloros-cli lattice cam-connect --packet-size standard` ugyanezt küldi).
- **Egyszerre csak egy kapcsolat tarthatja fenn a kamerakészletet**: a megnyitás, a csomagméret-tesztelés, az első kalibrációs letöltés (kb. 70 másodperc egy e számítógépen új kamera esetén), az adatfolyam indítása. Egy olyan kérés, amelynek közben kamerára van szüksége (`read_nodes`, `set_settings`, egy rögzítés, egy második kapcsolat) legfeljebb 5 másodpercig vár rá, majd HTTP 503 hibával bukik meg (`CAMERA_POOL_BUSY`), megnevezve a csatlakozó kamerát (`ChlorosConnectError`); egy pillanat múlva próbálja meg újra. A listák soha nem várnak: az `list_cameras()` (`GET /api/camera/pool`), az `list_arrays()` és az `CameraSession.status()` azonnal válaszolnak. A csatlakozás közben lévő kamera a pool-válasz alatt szerepel: `connecting` (`serial`, `phase` — `opening the camera`, `rebalancing the pool bandwidth`, `loading its calibration`, `starting its stream`, `waiting for its stream` — és `elapsed_s`), és az `status()` az `connecting: True`-et jelzi az `phase`-szel; a többi kamerát élőben olvassa ki a rendszer, ha a pool szabad, egyébként az utolsó leolvasásuk alapján válaszol (`live: False`).

#### `CameraSession` módszerek

| Módszer | Leírás |
| --- | --- |
| `status(timeout=15.0)` | A kamera élő állapota, ahogyan az `chloros-cli lattice status` kiolvassa: `streaming`, `stream_state` (`starting`, `no_frames`, `streaming`, `start_failed` vagy `stopped`), `device` (modell, sorozatszám, firmware, hőmérséklet, `is_mono`) és annak aktuális beállításait. |
| `read_nodes(names, enum_names=(), timeout=30.0)` | GenICam-csomópontok olvasása; visszaadja az `{nodes, errors, enums, device, was_in_pool, in_array_id}`, valamint az `frame_values` és az `notes` értékeket, miközben az automatikus expozíció fut (lásd alább). |
| `set_settings(**kwargs)` | Csomópontok írása barátságos név alapján (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `target_brightness_auto`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). Az `exposure_time` (`gain`) egy kézi expozíció (erősítés): az `exposure_auto` (`gain_auto`) nélkül írják, azaz először kikapcsolják az automatikus módot, így az értéket a futó automatikus üzemmód nem utasítja el. Az eredmény, az `applied` tartalmazza azt, amit a kamera rögzített; a fényképezőgép által elutasított írási érték az `errors`-ben található, és nem kerül tárolásra. A hiba megnevezi a fényképezőgép által jelenleg elfogadott tartományt (az `ExposureTime` maximális értéke a fényképezőgép képkockasebességétől függ), és az a rendszer, amelyet kizárólag az elutasított érték miatt kapcsoltak ki, visszaáll; az `restored` felsorolja ezeket (pl. `{"ExposureAuto": "Continuous"}`). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | **Egyetlen** képkockát. Visszaad egy `CaptureResult`-et: a képkocka-metadaták szótárainak listáját (minden mentett szinten egyet), az `.skipped`, `.failed`, `.daq_files` és `.daq_matches`. (A sorozatfelvétel/többképkockás rögzítés eltávolításra került — ha sorozatra van szükséged, hívd meg az `capture()` függvényt egy ciklusban.) |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Egymást követő (`interval_s=0`) vagy időközönkénti felvételek ebből a kamerából, az `count` rögzítések és/vagy az `duration_s` (legalább egy, hogy befejeződjön) esetén, ahogyan az `ArraySession.capture_repeated()` is teszi egy tömb esetében. Visszaadja az `CaptureResult`-ek listáját; egy olyan rögzítés után leáll, amely a meghajtó teljességét jelenti. Az `on_capture(i, result)` mindegyik után fut; az egyéb kulcsszavak az `capture()`-hez kerülnek. |
| `disconnect(keep_settings=False, clear_settings=False)` | Felszabadítás a poolból, a kamera visszaállítása a csatlakozáskor talált állapotra (kioldó vezetékek, PTP, beállítások); az `keep_settings=True` ehelyett a saját memóriájába menti a beállításait, az `clear_settings=True` pedig visszaállítja azt a memóriát (lásd [Kamera memóriája](#camera-memory--keep_settings--clear_settings)); `camera_memory` értéket ad vissza, ha bármelyiket kérik. A tömb tagjaként szereplő kamera a teljes tömböt felszabadítja. Nincs hatása, ha már megnyitott munkamenethez csatlakoztunk. Visszatér, miután a háttérrendszer megerősítette a felszabadítást, vagy ha nincs mit felszabadítani (a kamera már nincs meg, vagy nincs futó háttérrendszer); az időtúllépés miatt megszakadt vagy a háttérrendszer által elutasított felszabadítás `ChlorosConnectError` hibakódot eredményez, mivel a kamera még mindig foglalt lehet, és a következő csatlakozás ütközne vele. Az `with` blokk kilépése soha nem eredményez hibakódot. |

Attribútumok: `serial`, `device` (a kamera felderítési rekordja) és `backend_url`.

`capture()` exportvezérlők (ugyanaz a modell, mint a tömb + GUI):

- `processing` / `levels` — Az `processing="all"` minden alkalmazható exporttípust elment; az `levels=["raw","radiance"]` csak azokat menti el (felülírja az `processing`-et). A háttérprogram alapértelmezett beállításához hagyja ki mindkettőt.
- `daq` — a rögzítéshez használt DAQ-leolvasást alapértelmezés szerint `.daq` sidecar fájlként menti el, amely tartalmaz egy kizárólag nyers adatokat tartalmazó rögzítést is, így a képkocka később újra feldolgozható reflektancia/törésmutató értékekké. `daq=False` kihagyja, ha egyetlen szint sem használja. Az `force_daq=True` az alapértelmezett beállítás régebbi írásmódja. DAQ nélkül nincs `.daq`.
- `daq_tolerance_ms` — milyen messze lehet az érték a felvételtől, és még mindig a tűréshatáron belül maradjon (alapértelmezés: 500). Csak az értékelés változik.

#### `read_nodes()` automatikus expozíció esetén — `frame_values` és `notes`

Amíg az `ExposureAuto` (`GainAuto`) fut, az `ExposureTime` (`Gain`) csomópont a **legutóbb írt értéket** olvassa vissza, nem pedig azt, amit a kamera éppen csinál. Az `nodes` továbbra is azt az értéket tárolja. Ha az `ExposureTime` vagy az `Gain` értéket kéri egy olyan csatlakoztatott kamerától, amelynek automatikus üzemmódja nem az `Off`, a válasz a következőket is tartalmazza:

- `frame_values` — a kamera legújabb képkockájában szereplő érték, a csomópont neve alatt (`ExposureTime` µs-ben, `Gain` dB-ben);
- `notes` — minden ilyen csomópontra vonatkozóan egy mondat, amely mindkét értéket tartalmazza.

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

Mindkét kulcs hiányzik, ha az automatikus üzemmód `Off` (ebben az esetben a csomópont a kameravalódi értéke), valamint olyan kamera esetében, amely nincs csatlakoztatva a háttérrendszerhez. Tömb esetén az `ArraySession.status()["member_ae"]` egyszerre adja meg az összes elem képkockaértékeit.

#### DAQ-mért expozíciós felső határ (HTTP)

Csatlakoztatott DAQ fényérzékelő esetén a Chloros a mért nappali fény alapján korlátozza a kamera expozícióját és erősítését, hogy a fehér referenciapanel ne legyen túlexponált. A felső határ csak addig érvényes, amíg a kalibrációs célpont a kamera látómezejében van (az elmúlt 10 másodpercben a képkockáján belül látható); ha nincs célpont a látómezőben, helyette a kamera szokásos automatikus expozíciója működik, saját kiemelt területek védelmével, így a jelenet többi része nem marad szükségtelenül sötétebb. SDK-wrapper nem áll rendelkezésre; kameránként az `GET` / `POST /api/camera/<serial>/daq-exposure-cap` parancsokkal lehet beállítani vagy módosítani (az `{"enabled": false}` POST-test lekapcsolja az adott kameránál). A válasz tartalmazza az `enabled`, `cap_us` értékeket (az élő felső határ, az `None` értéket inaktív állapotban, beleértve azt is, amikor nincs kalibrációs célpont a látómezőben), `k_exp_g` (az a konstans, amellyel a felső határt kiszámítják) és:

| Mező | Jelentés |
| --- | --- |
| `k_exp_g_source` | Melyik állandó van használatban: `camera` — a kamera sajátja, amelyet a kalibrációs csomagjában található laborpontból számítottak ki (RGB-kamerák); `fleet` — a flotta-állandó, amelyet akkor használnak, ha a csomagban nincs laborpont; `override` — a háttérrendszerben beállított mezőfelülírás. |
| `k_exp_g_fleet` | A flotta-állandó, összehasonlítás céljából. |

### Szinkronizált tömb — `ArraySession` (Smart-Prep)

Az `connect_array` **az ajánlott kiindulási pont** a többkamerás konfigurációkhoz. A háttérben a teljes GUI-alapú Smart-Prep munkafolyamatot futtatja:

1. **Hálózati elemzés**(`/api/camera/array/recommend`) — beállítja a**teljes érzékelőt** (nincs binning, nincs ROI-kivágás), és hagyja, hogy a képkockasebesség a gazdagép és a hálózat által biztosított értékre csökkenjen, figyelembe véve a Connect által használt pixelformátumokat és csomagolást. A képkockákat soha nem kicsinyítik a sebesség fenntartása érdekében: adjon meg egy kisebb `width` / `height` vagy `binning=2` / `4` értéket, hogy a felbontást a képkockasebességre cserélje (az endpoint`alternatives` felsorolja ezeket az opciókat a várható fps-értékekkel együtt).
2. **Szint szerinti automatikus kiválasztás** — `sim-capture-sim-emit` (minden kamera ugyanazon a kiváltó jelre exponál, a jelek ütemezése úgy van beállítva, hogy megosszák a vezetéket, a képkockasebesség csökken, ahogy a megosztás csökken) bármely olyan hálózaton, ahol a kamerák ütemezése úgy beállítható, hogy megosszák a vezetéket; `slip-emit-and-capture` (a kamerák felváltva használják a vezetéket, nem egyszerre exponálnak) csak akkor, ha ez nem lehetséges.
3. **PTP alapértelmezés szerint engedélyezve**— a kamerák közötti időbélyegek**~1 ms**-os pontossággal egy közös órára igazodnak, a háttérrendszer [PTP grandmasterje](#ptp-grandmaster--time_sync_status). Az egyidejű expozíciót az M8 hardveres trigger biztosítja (**< 100 µs** modulok között), nem a PTP: a PTP az *időbélyegeket* igazítja, nem az expozíciókat.
4. **Kameránkénti pixelformátum-automatikus kiválasztás** — monokróm → `Mono12`, színes → `BayerRG12`.
5. **AE-beállítás** — pillanatképeket készít az egyes kamerák aktuális AE-állapotáról, így a csatlakozás nem állítja vissza az expozíciót a felvétel közben.
6. **GPIO-trigger konfiguráció** — az `connect_array` minden kamerát aktivál (`TriggerMode=On`, `TriggerSource=Line2`), így a masterimpulzusai az M8-as kábelen keresztül vezérlik a szolga kamerákat. Ez a lépés csak tömb esetén érvényes: az `LatticeCamera` paranccsal megnyitott egyetlen kamera ehelyett szabadon fut. A tömb felszabadításakor (`disconnect()` / `force_disconnect()`, `chloros-cli lattice array-disconnect`) minden kamerát visszaállít szabad futásra (`TriggerMode=Off`), minden egyéb beállítás pedig a csatlakozáskor talált állapotban marad: a kioldó és a szinkronvonal beállításai (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`; a vezérlő szinkronvonalát visszaállítva bemeneti állapotra), a PTP-t, valamint a munkamenet során megváltozott kamera-beállításokat – kivéve, ha `keep_settings=True` / `clear_settings=True` (lásd [Kamera memória](#camera-memory--keep_settings--clear_settings)).

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

Attribútumok: `array_id`, `master_serial`, `slave_serials`, `ptp_enabled`, `sync_mode`, `packed_transport` és `packed_reason` (az utolsó négy az `None` az `attach_array`-ből származó kezelőn), `connect_stream` és `backend_url`. Az `connect_stream` a csatlakozási válasz `stream` blokkja, az egyedi csatlakozás alakja plusz az `members` (`{serial: {state, why?}}`): `streaming`, miután a tömb első szinkronizált csoportja megérkezett (a háttérrendszer korlátozott ideig, körülbelül 10 másodpercig vár rá), vagy az `no_frames`, amelynek `why`-ként minden tag utolsó letöltési hibája szerepel, és jumbo-csomagok esetén az `note`-ben szereplő 1500 bájtos tartalékérték. `None` egy `attach_array` kezelőn.

#### `connect_array()` aláírás

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

Az `timeout` azt jelzi, hogy a hívás mennyi ideig vár a csatlakozásra, és ezt a
határidőként küldi el a háttérrendszernek. Ha a kapcsolat addigra nem jön létre, a művelet visszavonásra kerül: minden
megnyitott kamera visszaáll a megtaláláskori állapotába és bezáródik, egyetlen tömb sem marad csatlakoztatva, és a
hívás `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`, megnevezve azt a
lépést, amelyig a csatlakozás eljutott). Tehát egy kivételt kiváltó hívás soha nem hagy maga után
csatlakozott tömböt maga után. Egy 9 kamerás tömb csatlakozása körülbelül egy percet vesz igénybe; közvetlenül
a kamerák bekapcsolása után ez akár több percet is igénybe vehet. Egy olyan tömb, amely már működik, de még nincs
szinkronizált képkockacsoportja, visszaadódik az `connect_stream["state"]`
`no_frames` kóddal. A háttérrendszer naplójában minden csatlakozási lépéshez egy időzítési sor tartozik
(`[array-connect <id>] <step>: <s> s …`), valamint egy utolsó sor, amely megnevezi a leglassabbat.

`force_tier` értékek:
- `"sim-capture-sim-emit"` — valódi egyidejűség (minden kamera ugyanazon az órajel-élén lő).
- `"sim-capture-ftd-stagger"` — rugalmas időtartománybeli eltolás (a kamerák kissé eltolt időpontokban adnak jelet, így a csomagok sorba rendeződnek a vezetéken).
- `"slip-emit-and-capture"` — kameránkénti szekvenciális rögzítés (nincs időbeli szinkronizálás). Csak akkor választódik ki automatikusan, ha a kamerák nem tudnak elég lassan működni ahhoz, hogy egyidejűleg osztozzanak a vezetéken.

Az `wire_ceiling_mbps` felülírja a **gazdagép folyamatos vezeték-kapacitását** MB/s-ban — ez az az egyetlen
szám, amelytől az egész tömb allokációja függ. Hagyja `None` értéken az automatikusan felismert
érték használatához. Csökkentse, ha a tömb GVSP-sérült kereteket jelent: az automatikus érték a
hálózati kártya által jelzett kapcsolatsebességből származik, amely túlbecsüli az USB-adaptereket, a keskeny PCIe sávokat és
a forgalmas megosztott hálózatokat — és ez a túlbecslés sérült keretek formájában jelentkezik, nem pedig
szemmel láthatóan lassú kapcsolatként. Az érték a projekt tömb-rögzítési blokkjában marad meg, így egy
újbóli megnyitás vagy egy későbbi `connect_array` parancs visszaállítja azt, mint bármely más tömbbeállítást.
Lásd [Tömb állapota](#array-health--which-subsystem-is-losing-frames).

#### Tömörített 12 bites átvitel (`packed=`)

Teljes érzékelőkapacitás mellett a tömb képkockasebességét az határozza meg, hogy az egyes kamerák milyen gyorsan tudják elküldeni
a képkockáikat. A 12 bites formátumok (`BayerRG12`, `Mono12`) általában 16 bites
szavakként haladnak; a tömörített átvitel ugyanazt a 12 bitet 1,5 bájtban küldi el pixelenként, és
a gazdagépen bontja ki őket, így minden képkocka elküldése 25%-kal kevesebb időt vesz igénybe — körülbelül
**33%-kal több képkocka másodpercenként teljes érzékelőteljesítmény mellett** (mért érték: 10,7 → 14,0 fps egy
9 kamerás rendszerben). **A mentett adatok azonosak**: ugyanazok az értékek, ugyanazok a 16 bites
fájlok, ugyanazok a metaadatok.

- Az `packed="auto"` (alapértelmezett) csak akkor tömörít, ha ez előnyös **és** a gazdagép
  képes a kicsomagolásra: a csatlakozáskor a háttérrendszer megméri saját kicsomagolási sebességét,
  a CPU-k számát, az aktuális terhelést és a hálózati sávszélességet. Egy szabad magokkal rendelkező asztali számítógép
  teljes szenzoros adatmennyiség mellett tömörít; egy leterhelt vagy alacsony teljesítményű gazdagép (egylapos számítógépek,
  beágyazott vezérlők) kicsomagolt adatokat továbbít. A kis ROI-k és a binningelt képkockák esetében alig van nyereség.
- Az `packed="off"` (vagy `False`) soha nem tömörít — a diagnosztika során a tömörítés kizárására,
  vagy olyan gazdagépen, amelynek a CPU-ját más feladatokra kell fenntartania.
- Az `packed="on"` (vagy `True`) mindig csomagolja a 12 bites formátumokat.
- Bármely más érték `ValueError` hibát eredményez. A 8-bites és 16-bites formátumokat soha nem
  csomagolják.

Az `analyze_array_network(..., packed="auto")` ugyanilyen döntést hoz,
így az fps-értéke megegyezik a tömb futási sebességével (a válasz tartalmazza
az `packed_transport` és az `packed_reason` értékeket); az `pixel_format` alapértelmezés szerint azt veszi át, amit
a connect alkalmaz (Mono12 / BayerRG12), nem pedig a 8 bites értéket.

A döntés és annak indoklása a munkamenetben található:

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

az `ChlorosProject.connect_all(packed=...)` ugyanazt a felülírást alkalmazza a projekt minden
tömbjére (alapértelmezés: `None`, ami azt jelenti, hogy `"auto"`: a projekt nem ment
tömörített beállítást tömbönként).
Gép-szintű alapértelmezéshez állítsa be az `CHLOROS_ARRAY_PACKED12=off` (vagy `on`) értéket a
háttérprogram környezetében; egy kifejezetten megadott `packed="on"` / `"off"` beállítás továbbra is elsőbbséget élvez.
Az `/api/camera/array/status` az `packed_transport` és az `packed_decision`-et jelenti.

#### Várható képkockasebességek

Mért képkockasebességek (hardveresen szinkronizált, 12 bites; teljes érzékelő = 2048 × 1536). Használja őket az `status()["actual_fps"]` ellenőrzésére; az `analyze_array_network()` kiszámítja a saját gazdagépére és hálózatára vonatkozó képkockasebességet.

| Beállítás | Tömb képkockasebessége |
| --- | --- |
| Teljes érzékelő, PCIe 10 GbE gazdagép-hálózati kártya, tömörített adatátvitel (`packed="auto"` asztali számítógépen) | ~14 fps |
| Teljes érzékelő, PCIe 10 GbE gazdagépes hálózati kártya, csomagolatlan (`packed="off"`) | ~10,5 fps |
| Teljes érzékelő, 8 kamera USB 10 GbE adapteren | ~3,8 fps |
| Teljes szenzor, 1 GbE gazdagép-kapcsolat | 1,2–1,7 fps |
| 2 × 2 binning (`binning=2`), 10 GbE gazdagép-hálózati kártya | ~30–33 fps |

- PCIe 10 GbE kapcsolaton teljes szenzoros üzemmódban a **kamerák** jelentik a korlátot, nem a hálózat: 4 és 8 kamera egyaránt ugyanazzal a sebességgel működik.
- **Az `target_fps` jelzi a kioldási sebességet, és a rendszer soha nem működik ennél gyorsabban.** Az `target_fps=1` 1 fps sebességgel működteti a rendszert; az `None` (az alapértelmezett) a gazdagép és a hálózat által támogatott legmagasabb sebességgel működteti. Az ezt meghaladó értékeket lefelé igazítja erre a szintre (lásd [`set_target_fps`](#interval-capture-for-drone-surveys)).
- **Az `link_speed_mbps` az `4294`-ben az `status()`-ben nem kapcsolati sebesség.** Ez azt jelenti, hogy a Windows nem tudta kiolvasni az adapter sebességét (az illesztőprogram a legnagyobb 32 bites értéket jelentette), ami jellemző az USB-Ethernet-adapterekre. Egy USB-adapter Windows alatt használható sebessége minden esetben 200 MB/s-ra van korlátozva (lásd [Array Health](#array-health--which-subsystem-is-losing-frames)); a beállításhoz lásd a CLI-referencia „USB 10GbE adapterek” című szakaszát a [Host NIC Setup & Tuning](cli-reference.md#host-nic-setup--tuning-lattice-arrays) alatt.
- **Az USB-korlátozás Linux alatt is érvényes: 400 MB/s x86_64 rendszeren, 200 MB/s NVIDIA Jetson rendszeren.** Az adaptert a sysfs alapján azonosítja a rendszer (visszaesve az `ethtool -i`-re), így egy USB 5 GbE vagy 10 GbE adapter teljesítményét a korlát alapján számítja a rendszer, nem pedig a névleges 500 / ~1060 MB/s érték alapján. Az x86_64-es értéket Linuxon mérték: ugyanaz az adapter, amely Windowson 200 MB/s felett megrongálja a képkockákat, 4 kamerás rendszer esetén minden korlátnál 480 MB/s-ig hibátlanul működött. NVIDIA Jetson rendszeren ugyanaz az adapter csak 200 MB/s sebességig működött hibamentesen, e felett pedig időnként kereteket veszített. Ha a gazdagép `gvsp_corrupt` nélkül több keretet képes feldolgozni, növelje azt az `wire_ceiling_mbps` paranccsal. Linuxon egy USB 2.0-porton csatlakoztatott adapter sávszélessége 30 MB/s; csatlakoztassa USB 3-as porthoz.

#### Túlterhelés (a kameránkénti alsó határ)

A tömb bármilyen hálózaton működik; a képkockasebesség határozza meg a teljesítményt. A Sim-emit szabályozás minden kamerának egyenlő részesedést oszt ki az ütközésmentes sávszélességből, és mindegyik továbbra is ugyanazon a triggeren adja ki a képet, így lassú sávszélesség esetén a részesedés – és a képkockasebesség – kicsi: 8 teljesfelbontású, 12 bites kamera 1 GbE-n, jumbo keretek nélkül, egyenként 6,5 MB/s-ot kap, ami körülbelül **1 fps**(csomagolva körülbelül 1,3 fps). A részesedés alsó határa a kamerák leglassabb lehetséges ütemezési sebessége (`per_cam_floor_bps`: csatlakozáskor kiolvasásra kerül az egyes kamerák GevSCPD-tartományából, és soha nem lehet 1 MB/s alatt). Ha a hálózat ennél lassabb sebességet igényel, a csatlakozás**soros**adásra vált vissza (`slip-emit-and-capture`): a kamerák felváltva teljes kapcsolati sebességgel működnek, így minden keret megérkezik, de a tagok**nem jelennek meg egyszerre**— a válasz szintje és blokkolói (`SERIAL: ...`) ezt jelzik; az `force_tier="sim-capture-sim-emit"` átadása elutasítja a visszaesést. A képkocka mérete nem számít bele a megosztásba, így**a binning és az ROI növeli a képkockasebességet, de nem változtatja meg, hogy egy tömb befér-e**. Csak az a vezeték**túlterhelt**, amely semmilyen ütemezés mellett, sem egyidejűleg, sem felváltva nem képes továbbítani a tömböt — a hiba módja a GVSP-csomagvesztés, nem pedig az alacsonyabb képkockasebesség. Megoldások: kevesebb kamera, végpontok közötti jumbo-képkockák, vagy gyorsabb hálózati kártya.

- Az `analyze_array_network()` és `/api/camera/array/connect` válaszok az `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` és `per_cam_floor_bps` értékeket. Ha az `oversubscribed` érték igaz, a vetítés **nullázza az fps mezőket** (`achievable_fps_max` / `fps_bright` / `fps_dark`), ahelyett, hogy félrevezető, lassú, de működő sebességet jelentené.
- Az `POST /api/camera/array/connect` elfogadja az `pin_resolution` testparamétert (**csak HTTP — nem SDK-kwarg**; az `connect_array` nem teszi elérhetővé). Szigorú üzemmódban működik: egy túlterhelt kapcsolat, amelynél az `pin_resolution` be van állítva,**kategorikusan elutasításra kerül**, és a hibaüzenet felsorolja az összes lehetséges megoldást. Ennek hiányában a csatlakozás figyelmeztetéssel folytatódik, miszerint a tömb csomagveszteséget fog szenvedni. Mindkét esetben a csatlakozás megtartja a kért keretméretet — soha nem osztja fel vagy vágja le önállóan.
- Kizárólag tesztelési célra: állítsa be az `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` értéket a háttérrendszer környezetében, hogy az elutasítást hangos figyelmeztetésre csökkentse – így a kapcsolat létrejön, és elfogadja a csomagvesztést.

#### A tömb állapota – melyik alrendszer veszít kereteket

Az `ArraySession.capability()` (`GET /api/camera/array/<array_id>/capability`) egy élő `health` blokkot hordoz egy
csatlakoztatott tömbön, amelyet **10 másodperces** időablakban újraértékelve. A keretveszteséget
két, egymással ellentétes javítást igénylő okra bontja, ahelyett, hogy egyetlen „hiányos” arányt adna meg, amely
egyiket sem nevezi meg:

| Mező | Mit jelent | Melyik alrendszer |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (soronként) | A képkocka **megérkezett, de szerkezetileg hibás volt**— GVSP-csomagvesztés. |**Hálózat**: sávszélesség-korlát, ütemezés, NIC RX-gyűrű, MTU |
| `never_arrived_rate_pct` (soronként) | A keret **egyáltalán nem érkezett meg**— a kamera nem indult el, vagy semmi sem hagyta el azt. |**Trigger / szinkronizálás**: M8 kábel, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | A legrosszabb kameraátviteli arány . | — |
| `per_cam_rate_pct` | Kameraenkénti összesített hiányossági arány (mindkét ok együtt). | — |
| `stable_for_seconds` | Mennyi ideig maradt minden kamera 0,01 % alatt. | — |

Az `health` mellett ugyanaz a rekord jelzi azt a számot is, amelyen az egész allokáció lóg:

| Mező | Mit jelent |
| --- | --- |
| `wire_ceiling_mbps` | A gazdagép érvényben lévő, fenntartott vezetékes sávszélessége, MB/s. |
| `wire_ceiling_source` | Honnan származik ez az érték, szavakkal kifejezve — pl. `USB-capped 200 MB/s (was theoretical 1062; …)` vagy `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, amikor `wire_ceiling_mbps=` beállította. |
| `nic_is_usb` | `true` egy USB-Ethernet-adapter esetében. |

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Értelmezés:** a nullától eltérő `gvsp_corrupt_rate_pct` érték, ha az `never_arrived_rate_pct` értéke 0, azt jelenti, hogy
a triggerelés és a kábelszinkronizálás tökéletes, és a veszteség 100%-a a hálózati útvonalon keletkezik — csökkentse
`wire_ceiling_mbps` értéket, és csatlakoztassa újra. Az ellenkező mintázat inkább a szinkronkábelre vagy a
triggervezetékre utal.

> **Az `target_fps` nem a sérült keretek szabályozója.** A GevSCPD ütemzését egyszer írja be a
> csatlakozáskor, így a triggerfrekvencia csökkentése a kitöltési arányt változtatja meg, nem pedig a
> egyidejű kibocsátási robbanási sebességet. Egy mért 5-szeres igénycsökkentés nem hozott javulást, míg
> a vezeték felső határértékének 240-ről 200 MB/s-ra történő csökkentése ugyanazon rendszer esetében a sérült keretek arányát 10,4 %-ról
> 0,00 %-ra csökkentette.

> **A futó tömb nem tudja magát újratervezni a LATTICE (M3C / M3M) firmware-en.** Nem tudja
> ezt önállóan kijavítani; válassza le, majd csatlakoztassa újra, hogy a csatlakozási idő-választó az
> új felső határ alapján újratervezzen.

Egy **USB-Ethernet-adapter sebességét a szonda 200 MB/s-ra korlátozza**, függetlenül a
névjelölésétől függetlenül: az a hatékonysági táblázat, amely a kapcsolati sebességet tartós értékre konvertálja,
PCIe-alapú, és egy USB hálózati kártya az Ethernet-kapcsolati sebességét hirdeti, miközben az
USB-busz és az illesztőprogram korlátai között működik. A korlát abszolút érték, nem pedig hányad – egy USB 1 GbE adapter
kb. 80 MB/s sebességet ér el, és ez nem érinti. Ez érvényes a Windowsra és a Linuxra (beleértve a Jetsont is);
lásd a [Várható képkockasebességek](#expected-frame-rates) részt, hogy hol mérték.

#### `ArraySession` módszerek

| Módszer | Leírás |
| --- | --- |
| `status(timeout=10.0)` | A futó tömb aktuális állapota: `target_fps` / `target_fps_auto` (a triggerfrekvencia és az, hogy a frekvenciaszabályozó kezeli-e), `target_fps_clamped_reason` (miért tartják alacsonyabban a rögzített frekvenciát, egyébként `None`), `actual_fps` (mért érték; 0, ha a megfogási hurok feladta vagy leállt), `recommended_fps`, `streaming` (False, ha a rögzítő hurok feladta – ezt ellenőrizd, ne csak az `last_error`-et; a beállítások újraindítása után is True marad, amit az `restarting` jelzi), `stalled` (True, amíg a rögzítő hurok fut, de körülbelül tíz képkocka-periódus, 2–30 másodperc óta nem tett közzé képkockacsoportot; az `last_error` csak 60 másodperc elteltével jelzi ezt), `last_frame_age_s`, `last_error`, `serials`, `ptp_enabled` (`True` csak akkor, ha a tömb PTP-vel volt összekötve **és** minden tag legutóbb az `PtpEnable` értéket olvasta be; ez nem azonos a zárolt állapotgal, amelyet az `ptp_epoch` mér), `ptp_configured` (amit a csatlakozás kért), `ptp_off_members` / `ptp_unread_members` (olyan tagok, amelyek utoljára az `PtpEnable` ki állapotot olvasták ki / nem válaszoltak), `member_recovery` (`None`, vagy olyan tag, amely áramellátását vagy kapcsolatát elvesztette: `state`, `waiting`, miközben nem válaszol, `reopening`, majd az `recovered`, az `members` és az `why` — a tömb újra felveszi a tagot, amikor az újra válaszol, és visszaadja neki a csatlakozási idő szerinti PTP-jét, a trigger és a szinkronvezeték bekötését, a tempó- és a kamera-beállításokat, majd újra streamel anélkül, hogy új `connect_array`) és `ptp_epoch` (mért), `sync_mode` / `sync_tier`, `packed_transport` / `packed_decision`, `per_cam_emit_mbps`, `link_speed_mbps` (`4294` azt jelenti, hogy a Windows nem tudta kiolvasni az adapter sebességét — lásd [Várható képkockasebességek](#expected-frame-rates)), `trigger_pipeline` / `pipeline_tune`, `radiometric_chain` (hogy a sávonkénti sugárzási lánc használható-e, és melyik elem szakította meg), valamint `band_clamp` — miközben egy élő nézet megjeleníti a mátrixot: a megvilágított képpontok aránya, amelyet az unmix 0-ra korlátozott, elemként (`members`: `{serial: [band0, band1, band2]}` a kamera R, G, B sávsorrendjében), valamint sávonként a kompozit által mutatott érték (`composite`: `{band name: share}`), az `warn` értékkel, amely az a részarány, amelyet a grafikus felhasználói felület „Alacsony megbízhatóságú sávadatok”ként jelöl (másodpercenként körülbelül egyszer mérve; `None`, ha nincs élőképernyő fut), valamint az `member_ae` — tagonként az expozíció (`exposure_us`) és az erősítés (`gain_db`) értékét, amelyet a legutóbb közzétett képkocka hordozott, magából a képkockából (`read_nodes` vagy `ExposureTime` / `Gain` az automatikus expozíció futása közben utoljára írt értéket adja vissza, nem azt, amit éppen csinál). |
| `set_member_settings(serial, **kwargs)` | Írja be egy tagkamera beállításait, ahogyan az `CameraSession.set_settings()` azokat (expozíció, erősítés, célfényerő, ...); a tag a tömbben marad. `ChlorosConnectError` hibát dob, ha a sorozat nem tag. Lásd [Expozíció a tömb tagjain](#exposure-on-array-members). |
| `capability(timeout=10.0)` | Amit a smart-prep meghatározott (a rögzítési szint, a képkocka mérete, `pixel_format`), az élő `health` blokk és a gazdagép vezeték-kapacitása. Lásd: [Tömbállapot](#array-health--which-subsystem-is-losing-frames). |
| `set_target_fps(fps, timeout=10.0)` | A tömb futása közben módosítja a trigger-frekvenciát, újracsatlakozás nélkül. Egy szám rögzíti (a frekvenciaszabályozó nem avatkozik bele); `None` visszaállítja automatikus üzemmódra. Visszaadja az `{requested, target_fps, target_fps_auto, clamped}` értéket — az `clamped` értéke True, ha a sebességet a gazdagép és a hálózat által kezelhető szintre csökkentették. Lásd [Intervallum-felvétel drónos felmérésekhez](#interval-capture-for-drone-surveys). |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | Minden kamerából egy képkocka, **mindegyik egyetlen triggerből**, numpy-tömbök formájában (fájlok nem kerülnek írásra). Visszaad egy `GrabResult` (egy `dict`): `{serial: array}`, vagy `{serial: (array, metadata)}` az `with_metadata=True`-szel. Az `.group_id` az a kioldás, amelyet a képkockák megosztanak; az `.errors` az okhoz rendeli az egyes kihagyott kamerákat (azt a kamerát, amelyet az adott kioldásnál nem lehet kiszolgálni, kihagyják, és soha nem pótolják egy másikból). Az `processing` az `raw` / `debayered` / `radiance` / `reflectance` / `display`. Ugyanaz, mint a projekt API-jének [`ArrayHandle.grab()`](#one-trigger-per-read--group_id) — egy közös megvalósítás — így minden ott szereplő kulcsszó (`band`, `align`, `bit_depth`, `refl_max`) itt is működik. Szükség van az `numpy`-re. |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | Iterátor, amely tickenként egy `grab()` eredményt ad vissza (csak kulcsszavas argumentumok), `fps` szerint szabályozva, amelyet az `duration` (másodperc) és/vagy az `count` (`None` = korlátlan) határol. **Minden visszaadott halmaz egy új trigger:** az a tick, amelynek legújabb triggerje a már kiadott halmaz, ahelyett, hogy megismételné, a következő triggerre vár, így a stream az `fps` és a tömb saját képkockasebessége közül a lassabbik szerint fut. `FrameFetchError` kivételt dob, ahelyett, hogy elhallgatna: azonnal, ha minden kamera valamilyen okból elutasítja a következő tick ismétlését, 60 másodpercnyi képkockátlan tick után, valamint 60 másodpercnyi új trigger hiánya után („60 másodpercig nincs új trigger”). Ugyanaz a megvalósítás, mint az `ArrayHandle.frame_stream()` esetében. Lásd: [Élő képkockák a saját feldolgozási folyamatodban](#live-frames-in-your-own-pipeline). |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | Elindítja a háttérprogram saját folyamatos rögzítését (a grafikus felület Folyamatos módja) → [`ContinuousHandle`](#continuoushandle). Elmenti **minden** szinkronizált csoportot, amelyet a tömb közzétesz, ahogyan az `capture()` is mentené (ugyanazok az exportvezérlők), amíg le nem állítják, vagy amíg az `max_captures` / `duration_s` (`None` = a leállításig). `ChlorosConnectError` hibát dob, ha már fut egy ilyen a tömbhöz. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | Egy szinkronizált rögzítési csoport. Visszaad egy `CaptureResult` (keret-szótárak listája + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`). Adja át az előző eredményt`group_seq`-et `after_seq`-ként, hogy az azt követő következő szinkronizált csoportot kapja meg, ahelyett, hogy egy újra várna. Az exportvezérlők az alábbiak. |
| `capture(..., smart=True)` | **Intelligens rögzítés** — megvárja, amíg az AE minden kamerán stabilizálódik, majd elindítja a rögzítést. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | Leggyorsabb rögzítés: csak nyers adatok + a hozzárendelt DAQ-érték (+ a szabad kombinált index). A GUI „Leggyorsabb rögzítés” gombját tükrözi. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Egyszeri / Folyamatos / Intervallum egy korlátozott ciklusban. Visszaadja az `list[CaptureResult]` értéket.**Az `count` és/vagy az `duration_s` szükséges** a leállításhoz (az SDK-ben nincs Ctrl+C). Emellett a futtatás után is idő előtt befejeződik, ha egy futtatás teli meghajtót jelent (egy `.failed` bejegyzés `disk_full`-szel); ez a futtatás az utolsó eredmény. |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | Elindítja az élő kombinált indexnézet rögzítését videóba/GIF-be → `RecorderHandle`. A videó egy Motion-JPEG `.avi` (nincs szükség ffmpeg-re). Ha nincs megnyitva GUI-nézet, a háttérrendszer rendereli: `expr` (egy indexkifejezés a tömb sávneveiben, pl. `"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`), vagy alapértelmezés szerint a tömb NDVI-je. Tömbönként egy összetett felvevő. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | Indítson el egy nagy képkockasebességű nyers Bayer-sorozatfelvételt → `RecorderHandle`. Dolgozza fel újra offline módban az `build_video()` segítségével. `orient=True`: az `build_video()` által létrehozott termékek a kamera tájolása szerint vannak elforgatva (a kameránkénti termékek a saját kamerájuk, a kombinált index pedig a vezető kamera tájolása szerint); a nyers képkockák natív formátumban maradnak, és a sorozat manifesztje rögzíti minden elem tájolását. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Mentett nyers sorozatot dolgozzon fel offline módon kalibrált videó(k)vá. A művelet befejezéséig blokkol (`wait=True`), majd visszaadja a feldolgozás eredményét: `{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` — lásd [a feldolgozás eredményét](#the-build_video-result) lent. |
| `build_video_status(job_id, timeout=15.0)` | Ellenőrzi egy offline összeállítási feladat állapotát: `{running, result, error, burst_dir}`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Felszabadítja a teljes tömböt — **ha ezt a munkamenet nyitotta meg** (`connect_array`): akkor ez az `force_disconnect()`, és `ChlorosConnectError` hibát dob, ha a felszabadítást nem erősítik meg (az `with` blokk kilépése soha nem dob kivételt). Az `attach_array`-ből származó azonosítón a tömb továbbra is fut, és erről egy sort nyomtat ki a stderr-re: a tömb a GUI-hoz vagy egy másik szkripthez tartozhat. Minden kamera szabadon fut tovább, és egyébként olyan állapotban marad, amilyenben a csatlakozás megtalálta; az `keep_settings=True` a beállításait a kamera saját memóriájába menti, az `clear_settings=True` pedig azt a gyári beállításokra állítja vissza (lásd [Kamera memória](#camera-memory--keep_settings--clear_settings)). |
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | Felszabadítja a teljes tömböt, függetlenül attól, hogy ki nyitotta meg. Használja egy csatolt kezelőn, ha a tömb lezárását szándékozik végrehajtani (pl. hogy más beállításokkal újra csatlakoztassa). Visszatér, miután a háttérrendszer megerősítette a felszabadítást, vagy ha nincs mit felszabadítani (a tömb már nincs meg, vagy nincs futó háttérrendszer); az időtúllépés miatt megszakadt vagy a háttérrendszer által elutasított felszabadítás `ChlorosConnectError` hibát eredményez. Bármelyik jelzővel is, azt adja vissza, amit az egyes kamerák memóriája kapott (`camera_memory`), egyébként `None` értéket ad vissza. |

`capture()` exportvezérlők (ugyanaz a végpont, mint amit a GUI/CLI használ):

- `processing` / `levels` — az `processing="all"` (vagy `levels=["raw","radiance",…]`) minden alkalmazható exporttípust kameránként ment; egy egyetlen `processing` érték csak azt a szintet menti el.
- `aligned` — **alapértelmezés szerint be van kapcsolva (`None`), mivel a Chloros alkalmazás ugyanazon tömb rögzítése**(a tömbönkénti**Aligned** rögzítési beállítása). Minden tag nem nyers exportját a tömb [igazítási profilja](#array-alignment) segítségével a tagok közös átfedésére vágódik, így a fájlok pixel-regisztráltak, és méretük, kivágásuk, valamint az `Alignment*` címkék megegyeznek az alkalmazáséval (egy ROI alatt a transzformációt az ablakhoz igazítják); az `XMP-mapir:AlignmentJSON` az `"applied": true`-et jelzi az `crop_roi`-szel. A nyers fájlok torzításmentesek maradnak a natív méretükben, de a transzformációt a metaadatokban hordozzák. Az `aligned=False` minden fájlt natív méretben, torzításmentesen ment; az `aligned=True` az alapértelmezett beállítás. `continuous()`, az `capture(smart=True)`, az `capture_repeated()` és a projekt szintű `ArrayHandle.capture()` / `capture_smart()` ugyanazt az alapértelmezést veszik át. Függetlenül attól, hogy a torzítást alkalmazzák-e vagy sem, minden tagfájl rögzíti a kamerája `Alignment*` transzformációját, amint a tömb rendelkezik igazítási profillal — kivéve azokat a kamerákat, amelyeket a profil nem igazít (ki van hagyva, vagy a sikertelen megoldás miatt identitásra van kényszerítve), valamint a referenciakamera, ha nincs más igazított kamera: ezek nem hordoznak `Alignment*` címkéket. Ha a tömbhöz nincs profil, a felvétel igazítás nélkül kerül mentésre, és az eredmény ezt jelzi: az `result.alignment` értéke `{"requested": True, "applied": False, "warnings": [...]}`, egy figyelmeztetés minden olyan okra, ami miatt a torzítás nem alkalmazható (nincs profil, eltérő felbontású elemek, a tömb nemszinkronizálatlan, olyan képkocka, amelyre a profil nem térképezhető fel). Az `applied` csak akkor az `True`, ha minden olyan export, amely a torzítást kérte, meg is kapta azt. A mátrix automatikus igazítása létrehozza a profilt, miután az expozíciója stabilizálódik; az `ArrayHandle.calibrate_alignment()` / a CLI `align-calibrate` parancsa pedig igény szerint hoz létre egyet. Az `result.oriented` jelzi, hogy az `orient=True` elforgatta-e a termékeket.
- `render_index=False` — kihagyja a kameránkénti vegetációs index átfedést; alapértelmezés szerint ott jeleníti meg, ahol be van állítva.
- `daq` / `daq_tolerance_ms` — a DAQ-leolvasást alapértelmezés szerint `.daq` sidecar-ként menti, még akkor is, ha egyetlen kiválasztott szint sem igényli; az `daq=False` ekkor kihagyja, az `daq_tolerance_ms` pedig beállítja az egyezési tűréshatárt (alapértelmezés szerint 500). Az `force_daq=True` az alapértelmezett érték régebbi írásmódja. Lásd [A rögzítés által mentett DAQ-értékek](#a-daq-olvasása-egy-rögzítés-mentése--daq_fájlok-és-daq_egyezések).
- `orient=True` — minden tag **termékeit**(`display`, `radiance`, `reflectance`, a kameránkénti index és a kompozit) a kamera tájolásával az igazítási torzítás után: a**főkamera**tájolásával, amikor azok a főkamera képkockájába lettek torzítva (az `aligned` alapértelmezés — az elemek továbbra is egymáshoz igazítva maradnak), egyébként a saját kamerájuk tájolásával.**Alapértelmezés: minden elemfájl natív**, ahogyan a tömbfelvételeknél mindig is; az `raw` és az `debayered` mindkét esetben natív marad. Minden fájl rögzíti annak a képkockának a tájolását, amelyben a képpontjai találhatók (`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` — egy nyers fájl, a saját kamerájáé), az `Alignment*` XMP pedig a natív képkocka-transzformációt tárolja, amelyet a tájolás előtt alkalmaztak; azt a transzformációt hordozó nyersfájl rögzíti az `ChlorosAligned*` értéket is (a mesteré), amely a mester képkockájába torzított termék tájolását jelenti. A feldolgozás minden terméket a pixeljeihez tartozó értékkel forgat el: az igazítottat, ha a nyersfájlt torzítja (Array alignment be), a kamera sajátját, ha nem igazítja. Az `capture_repeated()` és az `capture_fastest()` átadja; a projekt szintű `ArrayHandle.capture()` / `capture_smart()` szintén átveszi.

**TIFF-tömörítés (csak HTTP-hez tartozó beállító):** Az `ArraySession.capture()` nem küld `compression` kulcsot, így a háttérprogram alapértelmezett beállítása érvényesül — az `POST /api/camera/array/capture` beolvassa az `compression` testparamétert, az `"none"` alapértelmezés szerint: tömörítetlenül, az érzékelő képpontjait pontosan úgy, ahogyan azokat leolvasta (~6,3 MB teljes felbontású képkockánként). Az `"deflate"` (veszteségmentes zlib L1 + vízszintes prediktor, ~4,1 MB/képkocka) kisebb fájlokat ír, de lassabban — mindkettő veszteségmentes, és importáláskor azonos módon olvasódik be. Az SDK nem biztosít ehhez kwarg-ot; a kiút az `chloros-cli lattice array-capture --compression deflate` vagy a nyers HTTP. A DEFLATE lefoglalja a Python GIL-t, így a tömörített írások nem párhuzamosíthatók a kameránkénti író szálak között — a szenzor sebességén történő folyamatos, teljes felbontású rögzítéshez az alapértelmezett tömörítetlen formátumra van szükség. Részletek: [CLI-referencia → array-capture](cli-reference.md).**Egymást követő csoportok (HTTP):** a végpont `group_seq` értéket ad vissza; egy olyan hurok, amely ezt a következő kéréskor `after_seq` formában küldi vissza, megkapja a következő szinkronizált csoportot — beleértve azt is, amely az előző rögzítés mentése közben került közzétételre — ahelyett, hogy a kérés után közzétett csoportra várna, és az `skipped_groups` jelenti azokat, amelyeket a hurok kihagyott. Az `capture_repeated`, a CLI `--continuous` parancsa és a GUI folyamatos üzemmódja is így működik; az intervallumon belüli lekérdezések minden ütemezett pillanatban frissek maradnak.**Tagonkénti export-felülírások (csak HTTP):**ugyanaz a végpont elfogadja az `exclude_serials` parancsot is (lista – tagok eltávolítása a mentett halmazból; a tömb továbbra is egy szinkronizált csoportként működik, és a kizárt tagokat az `excluded` parancs adja vissza), az `serial_levels` (`{serial: [level tokens]}` kameránkénti szintű felülírások), valamint az `serial_index` (`{serial: bool}` kameránkénti index-overlay felülírások) kódokat is elfogadja. Ezek a GUI-paritású testparaméterek, és**még nem SDK-kwargok**; a térképekből hiányzó elemek a tömb egészére vonatkozó `levels` / `render_index` értékekre esnek vissza.

##### Az átugrott kamerák ellenőrzése — `CaptureResult.skipped`

Az `ArraySession.capture()` egy `CaptureResult` értéket ad vissza, amely egy `list` alosztály: iteráljuk át, indexeljük, `len()` — minden meglévő minta továbbra is működik. Az új kód ellenőrizheti az `.skipped` attribútumot, hogy megnézze, mely kamerákat zárták ki és miért. A leggyakoribb eset az, amikor vegyes szűrőtömbben lévő RGB-kamerákról van szó, és `processing="radiance"` vagy `"reflectance"` értéket kérünk — a szélessávú érzékelő esetében a Bayer-mátrixonkénti sugárzás értelmetlen, ezért a háttérprogram inkább kihagyja ezeket a kamerákat, mintsem értelmetlen eredményeket állítson elő.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Az ok-tokenek az `<level>-not-applicable-to-rgb-cam` mintát követik (egy bejegyzés minden kihagyott szintre, mindegyik az `level` értéket tartalmazza). Az a kamera, amelynek kalibrációs csomagja nincs betöltve, kihagyja az `radiance` / `reflectance` értékeket az `<level>-needs-calibration (connect the camera so its cal pack loads)` értékkel, és ha nincs használható DAQ-reflektancia-érték, akkor az `reflectance-skipped-no-dls`-szel ugrik át. A többi reflektanciára vonatkozó kihagyás: `reflectance-skipped-no-fresh-dls` (nincs friss lefelé irányuló mérési érték), `reflectance-skipped-bound-daq-unavailable (…)` (a kapcsolt DAQ nem volt elérhető), valamint az `dls-uncalibrated-band-<nm>` — a sáv nagy része a DAQ fényérzékelőjének radiometrikusan kalibrált tartományán kívül esik (~374–974 nm), ezért a DAQ-alapú abszolút reflexiós értékek szerinti felosztás elutasításra kerül, és a képkocka hangosan visszavált az érzékelő-válasz alapú módra. A forgalomban lévő SKU-k közül csak az F988 váltja ki ezt; az adott kamera támogatott útvonala a reflexiós panel munkafolyamat.

##### Fájlok, amelyeket nem sikerült írni — `CaptureResult.failed`

Egy olyan rögzítés, amelynek csak egy része íródott le, még mindig a következő eredményt adja: a lista minden olyan fájlt tartalmaz, amely **el** lett mentve, az `.failed` pedig a többit sorolja fel, fájlonként egy dict-tel, az `serial` (`None` a kombinált kompozit és egy tömb `.daq` sidecar fájljához), `level` (`raw`, `radiance`, …, `index`, `composite`, `daq`, vagy `match` — az `.daq` `.match.json` származási adatai, amikor az `.daq`-et írták, de a sidecar-ját nem) és `error`. Teljes meghajtón a bejegyzés az `disk_full: True`-et is tartalmazza; minden későbbi rögzítés ezen a meghajtón ugyanúgy meghiúsulna, ezért az `capture_repeated()` ott véget ér. Részleges rögzítés esetén nem kerül kiváltásra kivétel, és az `.failed` fájl nem maradt félig írt állapotban a lemezen. Tiszta rögzítés esetén az `.failed` üres.

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### A rögzítést beolvasó DAQ mentése — `.daq_files` és `.daq_matches`

Amikor egy DAQ a kamerához vagy a sorozathoz van kötve, vagy poolba van rendezve, minden rögzítés elmenti az általa használt lefelé irányuló leolvasást: egy `.daq` fájlt a felvétel mappa `daq/` almappájába, az `.match.json` és az `.cameras.json` fájlokkal együtt (amelyek az adott mérési értékhez tartoznak). Ez vonatkozik a nyers és a debayered felvételekre is, és ugyanaz a fájl jön létre, függetlenül attól, hogy a felvétel a GUI-ból, a CLI-ből vagy az SDK-ből származik-e. Az `daq=False` kihagyja ezt azokat a felvételeket, amelyek szintjei nem használják a DAQ-ot (egy `radiance`, `reflectance` vagy `display` szint még mindig beírja); az `daq_tolerance_ms` áthelyezi az adott felvételre vonatkozó tűréshatárt, soha nem a felhasznált mérési értéket. Melyik DAQ: a kamerához vagy a sorozathoz kötött (a GUI fényérzékelő-kiválasztása, egy projekt kötése vagy az `chloros-cli daq pool-bind`), egyébként a háttérrendszer legfrissebbje. A CLI a vezérlőket `--no-daq` és `--daq-tolerance-ms` néven írja le ([CLI-hivatkozás](cli-reference.md#the-daq-reading-a-capture-saves)).

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

Egy rögzítés elmenti a felhasznált fényérzékelő-értéket `.daq` néven, mellette egy `<file>.daq.match.json` értékkel, amely megadja, hogyan kapcsolódott az érték az expozícióhoz: `method`, `gap_ms` (az expozíciótól a legközelebbi DAQ-leolvasásig) és `in_tolerance` (500 ms-on belül, vagy a felvétel `daq_tolerance_ms`-je). A reflexiós fájl XMP-adatlapja ugyanazt az értéket tartalmazza, mint az `Chloros:DlsMatchMethod`, az `Chloros:DlsMatchGapS` és az `Chloros:DlsInTolerance`.

Egy felvétel az expozíció utáni első DAQ-leolvasásra vár, hogy össze tudja keverni a felvétel mindkét oldalán lévő leolvasásokat (`method` `interpolated`). A DAQ-E nappal 0,4 másodpercenként, beltérben pedig 1–2 másodpercenként küld leolvasási értéket, így az `capture()` körülbelül másfél ilyen intervallummal később tér vissza; maga az expozíció nem késik. Azon DAQ-ra, amely leállt az adatküldéssel, nem várnak. A DAQ-E mérési értéke az előző mérés óta eltelt időtartam alatt átlagolt fényerő, így az adott időtartam közepére kerül: az `gap_ms` értéket ehhez a közepéhez viszonyítva mérik, az `.match.json``bracket_stamps_ns` adja meg az értékek saját időbélyegeit. Az `ArraySession.continuous()` várakozik anélkül, hogy lassítaná a rögzítést. Az `capture_repeated()` visszafelé-vissza átfutásai nem várnak, így megtartják a sebességüket, és a legközelebbi, az expozíció előtti mérési értékhez vannak rendelve.

Egy mérési érték soha nem jár le, így egyetlen képkocka sem veszik el egy késői érték miatt — ehelyett megjelölésre kerül:

- **A DAQ adatfolyama leállt** — a legfrissebb mérési értéket használja, és az `gap_ms` jelzi, hogy az mennyi ideje készült (`in_tolerance: false`: 500 ms-nál régebbi).
- **Nincs leolvasható DAQ**— kapcsolat megszakadt (`disconnect()`, `chloros-cli daq pool-disconnect`), vagy a társított DAQ nem érhető el — a kamera utolsó értéke kerül újra felhasználásra, és**ehhez** az expozícióhoz rendeli: `method`, `retained`, `gap_ms` ebből az expozícióból, az `in_tolerance` újraszámításra kerül, az `retained_from_epoch_ns` pedig azt az expozíciót nevezi meg, amelyhez először illesztették. Az a kamera, amelyről soha nem volt mérési érték, továbbra sem kap visszaverődési értéket, és az eredmény megmagyarázza, miért (`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, vagy a felvétel elutasítása).

A rögzítés által a Chloros-ban megnyitott projekt mappájába mentett fájlok csatlakoznak a projekt fájllistájához, ahogyan az alkalmazásban végzett rögzítés esetén is (a máshová mentett fájlokat nem érinti): `CameraSession.capture()`, `ArraySession.capture()` (`smart=True` is) és `capture_cameras_transient()`. Az eredményt az `.project_add` fájl tartalmazza (`capture_cameras_transient()`: a szótár `project_add` fájlja) -- az `held`, az `files` és a háttérrendszer `message` fájljában; az `held` az `True`, amíg egy feldolgozási futás folyamatban van, és a fájlok a futás befejeztével csatlakoznak hozzá. `None`, ha egyetlen fájl sem került be a projektbe.

Az `CameraSession.capture()` és az `ArraySession.capture()` az eredményként az `.daq_files` és az `.daq_matches` fájlokat adja vissza, HTTP-rögzítési válaszokként (`/api/camera/<serial>/cli-capture`, `/api/camera/capture`, `/api/camera/capture-all-synced`, `/api/camera/array/capture`) az `daq_files` és az `daq_matches` értékeket tartalmazzák, egyet-egyet az `.daq`-hez: `daq_file`, `cameras`, `method`, `gap_ms`, `in_tolerance`, `tolerance_ms` és `stream_fault` — a DAQ saját adatfolyam-hibája, amelyet kizárólag a tűréshatáron kívüli mérési értékek esetén kell ellenőrizni. Mielőtt egy rögzítésből származó visszaverődési értékre támaszkodna, ellenőrizze az `in_tolerance` értéket annak `.match.json` vagy XMP fájljában, illetve a DAQ [`status()`](#daqsensorsession-methods) értékét.

`processing` szintek:

| Szint | Kimenet |
| --- | --- |
| `"raw"` | Egycsatornás Bayer (monokróm kamerák: az egyetlen sáv) közvetlenül az érzékelőből. |
| `"debayered"` *(az SDK alapértelmezett beállítása)* | 3-csatornás színes: a kalibrációs csomag sötét eltolása (DSNU) levonva a nyers adatból, majd az OpenCV élekkel számoló demosaic-ja — nincs linearitási LUT, síkmező vagy színszétválasztás (monokróm kamerák: 1-csatornás szürkeárnyalatos). A mentett TIFF fájl **RGB**csatornasorrendben van (fotometrikus RGB, így bármely képnézegető a valódi színeket jeleníti meg); a memóriában lévő `grab()` / `frame_stream()` tömbök OpenCV**BGR** sorrendben vannak — fordítsa meg a csatornákat (`frame[..., ::-1]`), hogy összehasonlíthassa őket egy mentett fájllal. Az `chloros-cli process` ugyanazt az eredményt írja ki, mint az `Debayered_Images`. |
| `"radiance"` | float32 W/m²/sr/nm a teljes radiometrikus láncon keresztül. Csak multispektrális – az RGB-kamerákat kihagyja. |
| `"reflectance"` | uint16 0..32768 (Pix4D-kompatibilis); abszolút referenciaértékhez élő DAQ-párosítás szükséges. Csak multispektrális. |
| `"display"` | A teljes lánc a GUI-előnézethez igazítva (CCM + WB + gamma a kamera profilja szerint). |
| `"all"` | **Minden kamera esetében egy fájl az adott szintre** (megfelel a GUI „Capture All” / CLI alapértelmezésnek). A visszaküldött `CaptureResult` fájl ezután minden `(cam, level)`-hez egy képkocka-diktumot tartalmaz, az egyes diktumokban a szinttel; az alkalmazhatatlan szintek az `.skipped`-ben jelennek meg. A rögzítés olvasásához használt DAQ-adatokat `.daq` kiegészítő fájlként menti el. |

> **Megjegyzés — az alapértelmezett érték eltér a CLI-től.** Az `ArraySession.capture()` alapértelmezés szerint az `processing="debayered"`-re van beállítva; az `chloros-cli lattice array-capture` parancs alapértelmezés szerint az `processing="all"`-re van beállítva. Az `processing="all"` parancsot kifejezetten az SDK-ből kell átadni, hogy tükrözze a CLI/GUI többszintű mentését.

##### Tömb rögzítési fájlnevek

Minden szintet a saját almappájába ment az `output_dir` (`raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` az `display`, `index/`, `composite/` esetében). Minden fájl a kamerát `SN<serial>-<filter>` formában: először a sorozatszám, majd a kamera modelljének szűrő része — például `SN214000533-FOCN` az `M3C-L87-FOCN`-hez, `SN261902142-F450` az `M3M-L87-F450` esetében — az `-`-szel összekapcsolva, így a név ugyanazokat az `_`-szel elválasztott mezőket tartalmazza, mint korábban. Az a kamera, amelynek modellje nem olvasható ki, önmagában `SN<serial>`, a korábbi kiadásokból származó felvételek nevében pedig nincs szűrő. A fájlnév attól függ, hogy hány szintet kért a tag:

| Kért tagok száma | Fájlnév | Példa (`reflectance`, alapértelmezett: `format="tiff"`) |
| --- | --- | --- |
| **Egy szint** (`processing="reflectance"`, vagy egyelemű `levels`) | `sync_<timestamp>_SN<serial>-<filter>.<format>` — nincs szint-utótag, mint a korábbi kiadásokban, így a meglévő szkriptek továbbra is működnek | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **Több szint** (`processing="all"`, vagy több tételt tartalmazó `levels`) | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

A [clamp maszk](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) a termék nevére következik, `clamp_masks/<product folder>/<product name>_clampmask.tif`: egyszintű reflektancia-felvétel esetén az `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif` nevet írja, míg az `processing="all"` esetén az `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif` nevet. Az utótag szabályát tagonként határozzák meg a kért szintek alapján, így egy kihagyott szint (pl. fényvisszaverődés egy RGB-kamerán) nem változtatja meg azt. A visszaküldött képkockákból olvassa be az elérési utakat, ahelyett, hogy azokat létrehozná. Egykamerás rögzítés (`CameraSession.capture()`) esetén mindig hozzáadódik a következő szint: `capture_<ts>_SN<serial>-<filter>_<level>.tiff`.

##### A PNG- és JPEG-formátumok esetében csak a megjelenítési és indextermékek kerülnek figyelembevételre

A PNG vagy JPEG formátum csatornánként 8 bitet tárol. Ha olyan szintet kérünk, amelyet a formátum nem tud tárolni, a rendszer még a rögzítés megkezdése előtt elutasítja a kérést, és egy üzenetben megnevezi a szintet, valamint a TIFF formátumot, amely képes azt tárolni:

- **`radiance`**(float32) és**`reflectance`**(uint16, `32768` = 1,0) — így az `processing="all"` is, amely ezeket magában foglalja. Az SDK ellenőrzi ezt az argumentumokon: az `capture()`**`ValueError`** hibát dob, ha nincs kérés (`CameraSession`, `ArraySession`, `CameraHandle`, `ArrayHandle`, `capture_smart`).
- **`raw` / `debayered` olyan kamerától, amely 8 bitnél többet továbbít** (12 bites M3C, Mono12 M3M) — a háttérrendszer elutasítja ezeket, mivel ismeri az egyes kamerák bitmélységét: HTTP 400 hiba `error_code` és `FORMAT_CANNOT_HOLD_LEVEL` kódokkal, amelyeket `ChlorosConnectError` kódként jeleznek (`CameraSession` / `ArraySession`) vagy `RuntimeError` (`CameraHandle` / `ArrayHandle`). Egy kamera egyszintű `debayered` rögzítése 8 bites képkockát rögzít PNG/JPEG formátumban, és ezt a rendszer nem utasítja el.

A radiometrikus adatokat `format="tiff"` (`ext=".tiff"`) formátumban mentse; az `display` és az indexelt renderelésekhez használjon PNG/JPEG formátumot.

#### Az automatikus expozíció működése

- **Először az expozíció, utoljára az erősítés.** Ha az erősítés automatikus üzemmódban van, az erősítés 0 dB-en marad, amíg az expozíció el nem éri a megengedett leghosszabb értéket.
- **A kézi expozíció esetén az erősítés automatikus beállításon marad**, amíg meg nem változtatja az erősítést, vagy kézi üzemmódra nem állítja. Visszatérve az automatikus üzemmódba, az erősítés határértéke a korábbi marad.
- **A kis fényes területek nem sötétítik el a jelenetet.** A képkocka legfeljebb 1%-át kitevő tükröződést a rendszer figyelmen kívül hagyja, ismert kalibrációs célpanelek esetében pedig a panel legfeljebb 0,5%-át.
- **A DAQ expozíciós felső határa csak akkor érvényes, amíg a kalibrációs cél a látómezőben van.** Ez megakadályozza a fehér panel túlexponálódását; ha nincs látható panel, a kamera a jelenetnek megfelelően állítja be az expozíciót.
- **Az állandó fényviszonyok mellett az expozíció és annak határa változatlan marad, míg a fényviszonyok jelentős változása után a kép néhány másodpercen belül stabilizálódik.** Állandó fényviszonyok mellett az expozíció és annak határa változatlan marad, míg a fényviszonyok jelentős változása után a kép néhány másodpercen belül stabilizálódik. A túlexponálás már nem korlátozza véglegesen az expozíciót, és az érdeklődési terület (ROI) megváltozása esetén a kamera újra megtalálja a megfelelő expozíciót.
- **A fényerő-célérték elérhető tartományban marad.** Nem emelkedik, amíg a kamera az expozíciós határértékén van, és az erősítése sem növelhető, így a visszatérő fény nem okoz túlexponálást a képen.
- **Az a kamera, amelynek automatikus expozíciója nem reagál, a jelenlegi fényviszonyokhoz igazodó szinten marad.** Erős napfényben a kamera a legrövidebb expozíciós időre állhat át, és sötét képpel ott is maradhat, miközben a beállításai mind normálisnak tűnnek. A Chloros figyeli a képet, a kamerát a jelenlegi fényviszonyokhoz igazodó szinten tartja, és csak akkor engedi el, ha a fényviszonyok megváltoznak. Ezt nem zavarja, ha a fényérzékelő sötét vagy eltakarva van.

#### Expozíció a rendszer tagjain

Egy LATTICE kamera az `ExposureTime` értéket a képkockasebességének egy periódusára korlátozza. Amíg a rendszer adatot továbbít, a Chloros minden tagot az `AcquisitionFrameRateEnable` beállítással futtat, a kioldási frekvenciánál éppen valamivel magasabb sebességgel, így az elem expozíciós felső határa **bármely ROI-n** kissé a kioldási periódus alatt marad:

- Egy elem képkocka-periódusa a trigger-periódusból levonva a periódus 15%-át kitevő tűrés (legalább 3 ms, legfeljebb a periódus fele), expozíciós felső határa pedig az a képkocka-periódus, amelyből körülbelül 0,1 ms-ot levontak. Példa: egy 10 fps-es trigger (100 ms) esetén a tag sebessége körülbelül 11,8 fps, az expozíciós felső határ pedig körülbelül 85 ms.
- **Ha egy manuális `ExposureTime` beállítás meghaladja ezt a felső határt, a rendszer lelassítja a trigger frekvenciáját, hogy az illeszkedjen hozzá.** A Chloros először megnyitja az adott tag felső határát az értékre, és hagyja, hogy a lassabb frekvencia néhány trigger perióduson át futjon (az írás néhány másodperccel tovább tarthat), majd írja be az értéket, és a kioldás nem gyorsabb, mint amennyit az expozíció megenged. Automatikus sebességű tömb esetén a korábban futott sebesség visszatér, miután minden olyan elem, amelynek kézi expozíciója lelassította, visszatért az automatikus expozícióra. A kamerával magával nem rögzíthető értéket továbbra is elutasítja, a jelenleg elfogadott tartományt a hibaüzenetben jelzi, és a tag automatikus expozícióját visszaállítja, ha azt kizárólag az adott írás miatt kapcsolták ki. Az 1.3.0-ás verzió előtt egy kis ROI minden tagot az érzékelő saját szabadfutási periódusára korlátozott, függetlenül a trigger frekvenciától: 4,9 ms 768 × 304-es felbontásnál, így egyetlen tag sem lehetett világosabbá tenni.
- Az Chloros soha nem rövidíti le a kézi expozíciót egy gyorsabb triggerhez való igazodás érdekében. Az automatikus frekvencia a kézi expozíciók által megengedett határon belül marad; ha a rögzített frekvencia gyorsabb, mint amit az expozíció megenged, akkor az adott tag számára elegendő ideig fennmarad a felső határ, és a rendszer nem fogad el olyan gyors trigger-eket, mint amit az expozíció megenged. Csökkentse az expozíciót vagy a rögzített értéket, ha minden tagnak minden trigger-t feldolgoznia kell.
- **Visszaolvasás:** egy elem csomópontjainak kiolvasásakor az `AcquisitionFrameRateEnable` True és az `AcquisitionFrameRate` értékek jelennek meg az elem sebességén; az engedélyezés kikapcsol, amikor a tömb leáll.
- **A tömb birtokolja ezt a két csomópontot. Ne írjon `AcquisitionFrameRateEnable` vagy `AcquisitionFrameRate` értéket a tömb tagjára** — a tömbsebességét az `target_fps` / `set_target_fps()` parancsokkal.

Írja be az elem expozícióját az `arr.set_member_settings(serial, exposure_time=20000)` paranccsal (az `CameraSession.set_settings()` kulcsszavai; az `chloros-cli lattice exposure --serial` ugyanazt végzi). A tag a tömbben marad. Ne nyissa meg a tömb tagját az `connect_camera()` paranccsal: a connect elutasítja a már tömbben lévő kamerát, és az `CameraSession`, amelyet visszaadna, eltávolítja a kamerát a tömbéből, amikor leválasztja. Az `ArraySession.status()["member_ae"]` minden tagexpozícióját és erősítését a legfrissebb képkockából.

### Egy projektrekord, minden üzemmódban

Egy projektmappa tartalmazza minden eszköz beállításait: `cameras.json` — minden kamera beállításait (`cameras[].settings`, egyetlen kamera esetén és a tömb minden tagjára vonatkozóan), valamint minden tömb tömbszintű beállításait (`arrays[].capture`: formátum, ROI, binning, triggerfrekvencia, vezeték-kapacitás) — valamint az `sensors.json`, az egyes fényérzékelők beállításait (integrációs idő, képkockaátlag, korlát). Az alkalmazás, a CLI, az SDK és az MCP-szerver mind ugyanazokat a fájlokat olvassa és írja, ugyanúgy:

- **A projekthez csatlakoztatott eszköz abba kerül mentésre, és minden változást abba rögzít.** A „projekthez tartozó” kifejezés azt jelenti, hogy a háttérben (az alkalmazásban) nyitva van egy projekt, vagy — egy `ChlorosProject` (`open_project()`) fájlon keresztül, amely megadja a saját mappájának nevét a háttérnek (`project_path`). A projektben még nem szereplő kamera, érzékelőcsoport vagy fényérzékelő csatlakozáskor hozzáadódik a fájlokhoz (`connect_camera()`, `connect_array()`, a fényérzékelő csatlakozása, a CLI, az alkalmazás, `chloros-mcp`), a beállításokkal együtt. Bármelyik üzemmódból történő váltás (`set_settings()`, `ArraySession.set_member_settings()`, `set_target_fps()`, a tömbbeállítások, `set_cap()`, az CLI, az alkalmazás vezérlőelemei) egy pillanattal később elmentésre kerül az adott projekt fájljaiba. A kamerán beállított expozíció, erősítés vagy AE-célérték „tartva” állapotban kerül mentésre, így a projekt újbóli megnyitásakor az értékek ismét megmaradnak.
- **A projekt megnyitásakor az eszközök beállításai a projektből kerülnek beállításra**, minden üzemmódban: egy olyan kamera, mátrix vagy fényérzékelő, amelyhez nincs saját beállítás, a projektmentett beállításait veszi át (`lattice cam-connect` / `array-connect` nyitott projekt esetén, `chloros-cli project connect`, `open_project().connect_all()`, az alkalmazás újbóli megnyitásakor). Egy új azonosítóval csatlakoztatott sorozat frissíti a projektben ugyanazokkal a kamerákkal elmentett sorozatot; egy más kamerákkal rendelkező sorozat pedig mellé kerül hozzáadásra.
- **Az alkalmazás saját mentése soha nem írja felül az eszköz aktuális beállításait** azok régebbi másolatával, és megőrzi az egyes elemekre vonatkozó felülírásokat, amelyeket egy CLI- vagy SDK-hívó mentett el.
- **A beállított képpontformátum az, amelyet a projekt megőriz.** A Tömbbeállításokban, az `--pixel-format` vagy az `connect_array(pixel_format=)` parancsokkal beállított formátum – beleértve a 8 biteset is – pontosan mentésre kerül és újracsatlakozáskor is pontosan alkalmazásra kerül. Ha nincs beállítva, minden kamera az alapértelmezett 12 bites formátummal csatlakozik (`BayerRG12`, `Mono12` monokróm érzékelő esetén). A formátum neve a bitmélységet jelenti; a család az érzékelőé, így egy színes kamerát soha nem állítanak be monokróm formátumra, és egy monokróm kamerát sem Bayer-formátumra, és az `[PixelFormat <serial>] ... written` sor jelzi, ha egy formátumot leképeztek. Egy 8 bites felvétel exportált adatai 8 bitesek, a reflexiát is beleértve: átméretezve és az `PixelScale` jelöléssel ellátva.
- Egy projekt nélkül csatlakoztatott eszköz semmit.
- **Egy felvétel ahhoz a projekthez tartozik, amely azt követően jött létre; a megnyitáskor a projektjét megtartja.**Amikor egy projektet hoznak létre — `create_project()`, `chloros-mcp` vagy az alkalmazás**Új projekt**— vagy duplikálásra kerül (az alkalmazás**Duplikálás**funkciója, amely megnyitja a másolatot), miközben a felvétel még mindig a korábban megnyitott projektbe ír, a felvétel folytatódik, és amit ír, az az új projektbe kerül, ugyanazon a helyen (lent). Meglévő projekt megnyitása — `ChlorosProject.process()` (amely a háttérben nyitja meg a projektet), az `chloros-cli process`, az alkalmazás**Projekt megnyitása** funkciója — a felvételt továbbra is futni hagyja abban a projektben, amelybe rögzíti: ott fejeződik be, és a fájljai bekerülnek az adott projekt fájllistájába (az alkalmazás minden felvételt leállít, mielőtt megnyitna egy projektet).
  - `ArraySession.record()` és `ArraySession.burst()`: a videó, illetve az összes képkockát tartalmazó sorozatmappa teljes egészében oda kerül, amikor a felvétel leáll. A leállás eredményeként az új elérési út (`out_dir`, `video_path`), és a sorozatból készült összeállítás ebből indul ki. Ha a meghajtó nem engedi az áthelyezést, a felvétel ott marad, ahová íródott, és az eredmény ezt jelzi (`relocation_error`). A leállított sorozat nyers képkockái és az `.daq` felkerülnek annak a projektnek a fájllistájára, amelybe végül bekerül; a videó nem kerül hozzáadásra a projekt fájllistájához.
  - `ArraySession.continuous()`: a változtatás utáni minden felvétel az új projektbe kerül; a korábban mentettek ott maradnak, ahol vannak.
  - `DAQSensorSession.record_start()` (és az alkalmazás saját fényérzékelős felvétele): az `.daq` az új projekt `light_sensor` mappájába kerül, amikor a felvétel leáll; az `record_stop()` visszaadja az új elérési utat, és a fájl bekerül az adott projekt fájllistájába.

  A nyitott projekten kívüli mappába történő felvétel ott marad, ahol van, az `start_camera_continuous()` / többfolyamos feladatok pedig megtartják a nekik megadott mappát.
- **Az alkalmazásban az új projekt létrehozása után készült rögzítések abba kerülnek.** A projekt létrehozása vagy másolása során minden futó rögzítési folyamat és felvétel továbbra is folytatódik, és az azt követően készített minden rögzítés (Minden rögzítése, Rögzítés indítása) az új projektbe kerül, és felkerül annak fájllistájára. Egy meglévő projekt megnyitása az alkalmazásban először mindet leállítja.
- **A projekt felvétel közben nem kerül áthelyezésre.**A megnyitott projekt áthelyezése egy másik munkakönyvtárba (az alkalmazás**Beállítások → Munkakönyvtár** menüpontja; egyetlen SDK-hívás sem helyezi át a projektet, HTTP-kliens, amely az `/api/move-project`-re küld bejegyzést, viszont igen) HTTP 409 `PROJECT_RECORDING` hibakóddal kerül elutasításra, amíg egy tömbfelvétel vagy sorozatfelvétel, egy tömbfolyamatos rögzítés, egy kamera folyamatos/többcsatornás feladata vagy egy fényérzékelő-felvétel ír bele: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

Ez független attól, hogy a kamerák milyen beállításokkal maradnak a kapcsolat bontásakor (lásd [Kamera memória](#camera-memory--keep_settings--clear_settings)): egy projekt megnyitásakor maga állítja be a kameráit.

### Kamera memória — `keep_settings` / `clear_settings`

Amikor egy tömb csatlakozik, a kamerái átveszik a tömb beállításait, amelyeket ezután módosíthat. Amikor a Chloros elengedi a kamerákat — `ArraySession.disconnect()` / `force_disconnect()`, egy átmeneti funkció vége, az `start_camera_continuous()` leállítása — a rendszer mindegyiket visszaállítja a csatlakozás előtti állapotába: a trigger és a szinkronvonal bekötését, a PTP-t és a kamera beállításait — `ExposureAuto` / `GainAuto` (valamint a kézi `ExposureTime` / `Gain`), az `TargetBrightness`, valamint az AE-határok és csillapítás, a fehéregyensúly, a feketeszint, az `PixelFormat`, binning és az ROI, a színfeldolgozási folyamat (`GammaEnable` / `Gamma`, `LUTEnable`, `ColorTransformationEnable`), az `ReverseX` / `ReverseY`, `UserSetSelector`, a PTP-k `PtpSlaveOnly` / `PtpDomainNumber`, a csomagméret (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`), az ütem (`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`, `GevSCPD`, `GevSCFTD`), a blokkadat (`ChunkModeActive`, az egyes `ChunkEnable`-ek az `ChunkSelector` alatt), a trigger-bemeneti zavarmentesítés – egy tömb adja a szolga szinkronvonalát (`LineFilterSelector` / `LineFilterWidth`) és az adatgyűjtési képkockasebességet (`AcquisitionFrameRate` csak akkor, ha az engedélyezése be volt kapcsolva: kikapcsolt állapotban a sebesség a vezérlés által megengedett legnagyobb értéket jelzi, nem pedig egy beállítást). Csak a különbségek kerülnek rögzítésre: a kamera leállása, a vezérlés és a képkockasebesség az expozíció előtt, amelyhez a képkocka periódusára van szükség.

A kamerák projektön kívüli használatához két argumentum írja a kamera **saját memóriájába** — amivel bekapcsol, bármely szoftverben (az `disconnect()` / `force_disconnect()`, minden átmeneti függvény, `start_camera_continuous()` és `ChlorosProject.disconnect_all()`; MCP: az `disconnect_device` és az átmeneti eszközök azonos argumentumai; CLI-ben: az `--keep-settings` / `--clear-settings`):

- **`keep_settings=True`** elmenti az egyes kamerák beállításait úgy, ahogyan azok a munkamenet végén a memóriában voltak (UserSet2, amely a rendszerindítási beállításokat képezi). A kamerák továbbra is szabadon futnak (`TriggerMode` `Off`), a szinkronvezeték-kábelezés a talált állapotban marad. A UserSet2 nélküli kamera a UserSet1-be menti az adatokat; ezt a számítógép megjegyzi, így a Chloros-rendszer önjavító funkciója — amely visszaállítja a Chloros alapértékétől eltérő UserSet1-et — ezt a kamerát érintetlenül hagyja.
- **`clear_settings=True`** visszaállítja az egyes kamerák memóriáját a gyári beállításokra: Chloros alapértékére, amelyet a rendszer önjavítása rögzít — gyári alapbeállítások az `TriggerMode`, `Off`, `TriggerSource` és `Line2` parancsokkal, automatikus expozíció, valamint az RGB-kamera kalibrált fehéregyensúlya megőrzése — a UserSet1-ben, amely a rendszerindítási készletet alkotja.
- Nem mindkettő (`ValueError`; a háttérrendszer 400-as kóddal válaszol). A válasz tartalmazza az `camera_memory` kódot, kameránként az `{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}` vagy az `{"cleared": bool, "errors": [...]}` — az `force_disconnect()` adja vissza; egy feladat esetén az `stop()` kóddal érkezik.

Az alkalmazás ezeket nem kínálja fel.

**Minden kilépési útvonal visszaállítja a kamerákat.**A talált állapot szerinti visszaállítás minden kilépési útvonalon fut: `CameraSession.disconnect()` (az alkalmazás**X**kamerája), az alkalmazás**X** egy tömbön vagy bármely tömb elemen (a teljes tömb felszabadul), az alkalmazás bezárása (a háttérrendszernek legfeljebb 20 másodperce van arra, hogy minden kamerát, tömböt és futó folyamatos felvételt visszaállítson; egy átmeneti funkció vagy egy egyszeri hívás, amely még fut, a következő biztonsági pontján leáll, először visszaállítja a kameráit, legfeljebb 8 másodpercig, majd HTTP 503 `SHUTTING_DOWN` választ ad; egy még futó `connect_array()` a következő kamera előtt leáll, és visszaállítja a megnyitott kamerákat, legfeljebb 10 másodpercig, majd `ChlorosConnectError` hibakódot (HTTP 503 `SHUTTING_DOWN`); az pedig, amely a leállás közben érkezik, semmit sem nyit meg: HTTP 503 `SHUTTING_DOWN`), valamint egy `chloros-cli` parancs, amelyet leállítanak, vagy amelynek terminálja bezárul (feladatát leállítják, és kameráit először visszaállítja). Egy összeomló vagy erőszakosan leállított háttérprogram nem tud semmit visszaállítani; a készülék újraindítása minden kamerát visszaállít a memóriájába.**Egy projekt maga állítja be a kameráit.** Amikor egy tömb egy projekten keresztül csatlakozik — az alkalmazás nyitott projektjéhez vagy egy `ChlorosProject`-hez, amely megadja a nevét a háttérprogramnak (`project_path`) — minden tag megkapja azokat a beállításokat, amelyeket a projekt számára mentett: egy rá rögzített expozíciót, erősítést vagy AE-célt (a sajátodként rögzítve, így az automatikus expozíció nem változtatja meg), valamint a fehéregyensúlyi módját (a Chloros-korrekció; a kamera saját fehéregyensúlya kikapcsolt állapotban marad), függetlenül attól, hogy mi van a kamera memóriájában.

### Felvételi módok és rögzítők

A tömb felülete a GUI felvételi paneljét tükrözi: Egyedi / Folyamatos / Intervallum / Leggyorsabb záridő módok, valamint két rögzítő (élő kompozit videó és nyers sorozatfelvétel → offline újrafeldolgozás).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**az SDK folyamatos/intervallum hurokja. Mivel nincs `Ctrl+C`, amivel egy szkriptből leállíthatnánk,**kötelező** átadni az `count`-et és/vagy az `duration_s`-et (akkor áll le, amikor bármelyik elérhetővé válik). Az `interval_s`-et az egyes futások kezdetétől mérik (a grafikus felülethez igazodva). A folyamatos futások (`interval_s=0`) láncolják az `group_seq` → `after_seq` értékeket, így ezek a tömb sebességéhez igazodó, egymást követő szinkronizált csoportoksebességén. A fennmaradó kwargok közvetlenül átkerülnek az `capture()`-be.
- **`continuous`**a háttérrendszer saját folyamatos rögzítése — a grafikus felület Folyamatos módja.**Használja ezt az `capture_repeated()` helyett, ha minden triggert nagy képkockasebességgel szeretne lemezre menteni:** Az `capture_repeated()` a szkriptből csoportonként egy HTTP-kérést indít, míg az `continuous()` két rögzítést tart folyamatban a háttérrendszeren belül, csoportok közötti oda-vissza út nélkül. 9 kamerás rendszerben, 512-es ROI-n mérve: 73–77 rögzítés/másodperc, szemben egy körülbelül 78-at közzétevő rendszerrel. Az `capture_repeated()`-et tartsa meg az intervallumalapú rögzítéshez (`interval_s`) és az egyenként ellenőrizendő átvitelekhez. Windows rendszeren a magas sebesség elérése érdekében zárja ki a háttérrendszert a Microsoft Defenderből (lásd [1. lépés](#step-1--install-the-chloros-platform-package)).
- **`record`** *megfigyelési minőségű*: ez rögzíti a renderelt, élő kombinált indexű kompozíciót. A GUI kombinált nézetének megnyitásakor azt a nézetet rögzíti, ahogyan megjelenik. Ha nincs megnyitva, a háttérprogram maga rendereli a nézetet: `expr`, ha átad egyet, egyébként a tömb NDVI-jét (a 660 nm-hez és 850 nm-hez legközelebb eső vörös és NIR sávokat, ahogyan az élő nézet kiemeli őket), miután a sorozatot igazították. Nevezze el a sávokat az `expr`-ben úgy, ahogyan a sorozat nevezi őket: `<band>_<filter>` egy M3C kamera esetében (`NIR_850_RGN`, `Red_660_RGN`, `Orange_615_OCN`), `Mono_<filter>` egy M3M esetén (`Mono_850`); egy rövid színnév (`NIR`, `Red`) csak akkor működik, ha a tömb egyik sávja rendelkezik vele. A felvétel közben megnyitott GUI-nézetet a felvétel rögzíti, amíg nyitva van. Az a felvétel, amelyik nem ír keretet, `error`, `"no-frames: …"` hibával zárul (lehet, hogy a tömb még igazodik vagy állítja be az expozícióját), és az `stop()` hibaüzenet jelenik meg (lásd [`RecorderHandle`](#recorderhandle)). Tömbönként egy összetett felvevő (hibaüzenetet ad, ha már fut egy).
- **`burst` → `build_video`** *elemzési minőségű*: az `burst` nyers képkockákat ír + képkockánkénti manifesztet + egy `.daq`-et minden egyes DLS-leolvasáshoz az `<output>/bursts/<base>/` alatt a rögzítési ciklus teljes sebességén (nincs lánc, nincs exiftool, nincs élő nézet). Az `build_video` minden képkockát időben illeszt a legközelebbi `.daq`-hez, és újra futtatja az importáló folyamat sugárzási/reflektancia/index láncot. Az `products` az `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}`-ek listája (alapértelmezés: a kombinált index). Az `burst().stop()` emellett automatikusan elindít egy „best-effort” kombináltindex-összeállítást, amelyet a leállási eredményben `build_job` néven ad vissza.

##### Az `build_video` eredmény

| Kulcs | Jelentés |
| --- | --- |
| `success` | `True` csak akkor, ha minden kért termék a kérésnek megfelelően elkészült: legalább egy kimenet, semmi az `skipped`-ben, és nincs az `degraded`-ben. |
| `outputs` | Az írt videók (és GIF-ek) elérési útjai. |
| `errors` | Problémák, szöveges formában. |
| `skipped` | Egy `{"serial", "level", "reason"}` minden olyan kért termékhez, amely **nem** hozott létre videót — a kamera (nincs kalibrációs csomag az adott kamerához, nincs DLS-leolvasás a reflexióhoz, a feldolgozási lánc meghibásodott) vagy a kombinált indexeké, amely az `"kind": "combined"`, `serial` és `None` (nincs vörös/NIR sávpár a kamerák között, nincs olyan csoport, amely mindkét sávra sugárzási értéket tartalmazna). Bármely bejegyzés létrehozza az `success` és az `False` kódokat; a többi videó továbbra is írásra kerül és felsorolásra kerül. |
| `substitutions` | A kamerák a kértől eltérő szinten kerültek renderelésre, és ennek oka — egy sugárzási/reflektancia értékeket kérő RGB-kamera megkapja a megjelenítési videóját, fehér egyensúlyozva, ha a kamera kalibrációs csomagja elérhető. Ez nem hiba. |
| `notes` | Tájékoztató szöveg, nem hiba — pl. egy olyan kamera, amelynek nincs kameránkénti indexkifejezése, nem rendelkezik indexvideóval, vagy egy RGB-kamera kijelzővideója nincs fehérkiegyenlítve (nincs kalibrációs csomag, vagy olyan van, amelyben nincs gyári fehérkiegyenlítés), a kamera megnevezésével. Nem hiba. |
| `degraded` | `True`, ha egy termék leírása megtörtént, de nem az a kért termék: az offline kombinált index a beépített vörös/NIR normalizált különbség, így a konfigurált indexkifejezés nem kerül kiértékelésre. Ez eredményezi az `success` és `False` hibákat. |
| `combined` | A kombinált index összefoglalása: a felhasznált sávok, a LUT-tartomány, a létrehozott képkockák. |
| `clamp_masks` | Az egyesképkockánként kalibrált TIFF-fájlok [unmix clamp maszkjainak](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) útvonalai (`save_tiffs=True`), az `clamp_masks/<level>/` mappában a TIFF-fájlok `<level>/` mappája mellett, egy-egy minden olyan TIFF-hez, amelyben egy pixel korlátozásra került. A videók nem tartalmaznak ilyet. |
| `clamp_fractions` | Amit egy videó a maszk helyett tartalmaz: egy `{"kind", "serial", "level", "frames"}` videótermékenként (`kind` `"combined"` az összevont indexhez tartozó kamerák esetében), `frames` egy lista az `[frame, R, G, B]`-ekről — az adott képkocka pixeljeinek azon része, amelyet az unmix a R / G / B csatornája által tartott sávban korlátozott (`Chloros:Band{R,G,B}UnmixClampFraction`). |

#### `RecorderHandle`

Az `ArraySession.record()` és az `ArraySession.burst()` által visszaadott érték. Használja kontextuskezelőként a scope kilépésekor történő automatikus leállításhoz, vagy vezérelje manuálisan.

| Tag | Leírás |
| --- | --- |
| `job_id` | Háttérfeladat-azonosító (str). |
| `kind` | `"composite"` (az `record`-ből) vagy `"raw"` (az `burst`-ből). |
| `start_stats` | Az `start` hívás által visszaadott szótár. |
| `result` | `None` futás közben; a végső leállási eredményt tartalmazó szótár a leállás után. |
| `stats(timeout=10.0)` | Élő feladatstatisztikák (leírt képkockák, tényleges fps, eltelt). |
| `stop(timeout=60.0)` | A felvevő leállítása; visszaadja és gyorsítótárba helyezi a végső eredményt (az `.result` érték megmarad, ha maga a leállítási kérés sikertelen volt). Egy sikertelen `record()` felvétel esetén — amely nem írt ki egyetlen képkockát sem (`error` `"no-frames: …"`) — az eredményt az `.result`-ben tárolja, majd kiadja az `ChlorosConnectError`-et. Idempotens (egy második hívás a tárolt eredményt adja vissza, hibaüzenet kiadása nélkül). A `with` blokk elhagyása hibaüzenet nélkül leállítja a felvevőt, ezért utána ellenőrizze az `rec.result.get("error")`-et. |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

Az `ArraySession.continuous()` adja vissza. Használja kontextuskezelőként — az `with` blokkból való kilépés leállítja a rögzítést, ha az még fut — vagy vezérelje az `status()` / `wait()` / `stop()` parancsokkalparancsokkal vezérelje. Egyszerre egy tömbönként futhat egy folyamatos rögzítés.

| Tag | Leírás |
| --- | --- |
| `start_status` | A start hívás által visszaadott állapot-szótár. |
| `result` | `None` futás közben; a végleges állapot-szótár a rögzítés befejezése után (amelyet az `stop()` állít be, vagy egy olyan `status()` / `wait()` által, amely észlelte a befejezést). |
| `status(timeout=10.0)` | Aktív állapot-szótár (mezők alább). |
| `wait(timeout=None, poll_s=0.5)` | Blokkolja a műveletet, amíg a rögzítés magától be nem fejeződik — `max_captures` vagy `duration_s` elérése, a tömb eltűnése, a meghajtó teljessége, hiba —, majd adja vissza a végső állapotot. Az `timeout` parancs esetén adja vissza a legfrissebb állapotot az adott másodpercek elteltével, még akkor is, ha a folyamat még fut: ellenőrizze az `running` értéket. |
| `stop(timeout=30.0)` | Állítsa le a rögzítést, és adja vissza a végső állapotot. Idempotens (egy második hívás a gyorsítótárban tárolt eredményt adja vissza). |

Állapotmezők:

| Mező | Jelentés |
| --- | --- |
| `running` | Igaz, amíg a rögzítés fut. |
| `captures` | A szinkronizált csoportok mentve. |
| `files` | Fájlok írása megtörtént. |
| `failed_files` | Azok a fájlok, amelyeket nem sikerült írni (lásd [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)). |
| `failed_captures` | Teljesen sikertelen rögzítési kísérletek, beleértve azokat is, amelyeket a rendszer újra megpróbált (HTTP 503 hiba, miközben a tömbnek időben nem volt új csoportja). |
| `skipped_groups` | Az array által közzétett, de nem mentett csoportok (a rögzítés lemaradt). |
| `captures_per_s` | Átlagos mentési arány a kezdet óta. |
| `elapsed_s` / `started_at` | Eddig eltelt másodpercek; kezdési idő (Unix-korszak másodpercek). |
| `last_group_seq` | A legutóbb mentett szinkronizált csoport. |
| `last_error` | A legutóbbi hiba, vagy `None`. |
| `stop_reason` | Befejeződött: `count` (elérte az `max_captures` értéket), `duration` (elérte az `duration_s` értéket), `stopped` (meghívták az `stop()`-et), `array_gone` (a tömb kapcsolata megszakadt), `error` (20 egymást követő hiba, vagy a háttérrendszer által elutasított kérés), `disk_full` (a meghajtón nem volt hely a fájl számára). `None` futás közben. |
| `max_captures` / `duration_s` | Az indításkori határok (`0` = nincs). |
| `defender_cores` | A Microsoft Defender által az elmúlt 10 másodperces időtartam alatt használt CPU-magok száma; `None` amíg ilyet nem mérnek, és mindig `None` olyan gazdagépen, amelyen nincs Defender. |
| `defender_warned` | Igaz, ha a rögzítés naplózza, hogy a Defender lassítja a rendszert (1 vagy több magon). A háttérprogramot zárja ki a Defender vizsgálatából — lásd [1. lépés](#step-1--install-the-chloros-platform-package). |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### Élő képkockák a saját feldolgozási folyamatában

Ha szinkronizált képkockákat szeretne továbbítani a saját kódjába — gépi tanulás, egyéni nézőprogram, saját rögzítő —, használja az `ArraySession.frame_stream()`-et. Minden tick egy `GrabResult`-et eredményez: `{serial: numpy array}` minden kamerához, **mindegyik egyetlen triggerből**, a trigger azonosítójával az `.group_id`-ben. Ez ugyanaz a megvalósítás, mint a projekt API-jének `ArrayHandle.frame_stream()`-je, így a [Egy trigger olvasásonként](#one-trigger-per-read--group_id), [`band`](#exporting-a-single-band--band), [`align`](#aligning-the-streams--align) és [`bit_depth`](#bit_depth--trading-precision-for-bandwidth) alatt találhatóak, mindkettőre érvényesek.

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **Minden kiadott halmaz egy új trigger.** Az `frames.group_id` soha nem ismétlődik egy adatfolyamon belül; ha több `fps`-et kér, mint amennyit a tömb futtat, az adatfolyam megvárja a következő triggert, így a kettő közül a lassabb sebességgel fut.
- **A hiányzó kamerák meg vannak nevezve, nem pótolva.** Az a kamera, amelyet az adott triggerkor nem lehetett kiszolgálni, hiányzik a szótárból, és fel van sorolva az `frames.errors`-ben.
- **Hangos befejezéssel zárul.** Elutasítás esetén az `FrameFetchError` a következő tickben megismétlődik, 60 másodperc elteltével, ha nincs képkocka, vagy 60 másodperc elteltével, ha nincs új trigger.**Mit nyújt.**Az `frame_stream()` teljes numpy-képkockákat továbbít a helyi HTTP-kapcsolaton keresztül, és a maximális sebessége**170 MB/s**közelében van. Ez elegendő a nyers képkockák, illetve a binningelt vagy ROI-tömbök élő feldolgozásához, de nem minden trigger teljes felbontású feldolgozásához —**minden** trigger megőrzéséhez mentse lemezre a [`continuous()`](#continuoushandle) segítségével. A mérés egy négykamerás rendszeren (FOCN, FRGN, FRGB, F450) történt, tömörített adatátvitellel; minden tick egy teljes, egy triggerből álló készlet volt:

| | Teljes szenzor (tömb 14,2–14,4 fps) | 2 × 2 binning (tömb ~33 fps) |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6,7 sorozat/s | 22,2 sorozat/s |
| `frame_stream(processing="debayered")` | 2,3 sorozat/s | 9,3 sorozat/s |
| `frame_stream(processing="display")` | 0,8 sorozat/s | 3,0 sorozat/s |
| Kameránkénti MJPEG, 4 stream 720p felbontásban | egyenként 13,5 fps; minden képkocka egy-egy triggerhez rendelve | egyenként ~25 fps; a triggerek ~45%-a mind a 4 streamen jelen van |

Használja az `processing="raw"`-et az átviteli sebesség és a demosaic feldolgozásához a saját oldalán; az `debayered` szint a kalibrációs csomag sötét eltolását is eltávolítja, amit az `raw` demosaic-ja nem tesz meg.

#### Kameránkénti MJPEG-adatfolyamok (nincs RTSP)

**Az RTSP nem támogatott.** A streamelési interfész MJPEG HTTP-en keresztül (`multipart/x-mixed-replace`, részanként egy JPEG, minden részhez egy `Content-Length`), kameránként egy URL:

| URL | Mit szolgáltat |
| --- | --- |
| `GET /api/camera/stream/<serial>` | 8 bites előnézet, legfeljebb ~30 fps. Egy tömb elem esetében az `max_h` (alapértelmezett: 720, 64–2160) és az `jpeg_q` (alapértelmezett: 80) határozza meg a méretet és a minőséget. |
| `GET /api/camera/<serial>/stream-annotated` | Ugyanaz az előnézet, de a GUI-rétegekkel kiegészítve (zebra, rács, célkereszt, hisztogram, csúcsjelzés, pontmérő). Az `ChlorosLocal.camera_stream()` kiolvassa ezt az adatot. |
| `GET /api/camera/<serial>/stream-reflectance` | Sávonkénti 8 bites reflexió — lásd [Élő reflexió-adatfolyam](#live-reflectance-stream-mjpeg). |

- **Trigger fejlécek.** Egy tömb elem esetében mindhárom adatfolyam minden része tartalmazza az `X-Chloros-Group-Id`, `X-Chloros-Frame-Id` és `X-Chloros-Timestamp-Ns` jelölést hordoz. A különböző kamerákból származó, azonos `X-Chloros-Group-Id` jelöléssel rendelkező részek ugyanahhoz a triggerhez tartoznak: gyűjtsük össze a részeket, és csoportosítsuk őket az adott fejléc alapján a készlet újbóli összeállításához. Egy önálló kamera nem tartozik csoporthoz: előnézeti részei nem hordozzák ezeket a fejléceket (a reflexiós alkatrészei üres `X-Chloros-Group-Id` fejlécet hordoznak).
- **Az előnézeti adatfolyam terhelés alatt kihagy képkockákat.** A legfrissebb képkockát továbbítja, és kihagyja azokat, amelyeket nem tudott időben elküldeni, így magas tömbsebesség esetén nem minden trigger éri el minden kamera adatfolyamát (lásd a fenti táblázatot). Minden triggerhez használja az `frame_stream()` vagy az `continuous()` fejléceket.
- **Az `/api/camera/stream/<serial>` szünetel, amíg a rendszer egyesített élő streamje nyitva van** (`/api/camera/array/stream/<array_id>` — a GUI rendszer nézete nyitja meg). Ezután minden tag streamje elküld egy „Paused (array stream active)” helyőrző képkockát, `X-Chloros-*` fejlécek nélkül, és semmi mást, amíg az összevont adatfolyam bezárul; ezután magától folytatódik. Az `stream-annotated` és az `stream-reflectance` nem állnak le. Zárja be a grafikus felület tömbnézetét, amikor egy szkript olvassa be a kameránkénti előnézeti adatfolyamokat.
- **Számokhoz, nem képekhez használja az `frame_stream()` parancsot.**A JPEG veszteséges és 8 bites; az `frame_stream()` ugyanazokat az eredményeket adja vissza, ha nincs kodek az útvonalban.

### Intervallumonkénti rögzítés drónos felmérésekhez

Egy felmérő repüléshez néhány méterenként egy szinkronizált képcsoportra van szükség, nem pedig a rendszer által előállítható összes képkockára. Ezt két beállítás szabályozza:**Az**`target_fps`**beállítás határozza meg, hogy a kamerák milyen gyakran készítenek képkockát, az**`capture_repeated(interval_s=…)`** pedig azt, hogy milyen gyakran mentik el a képkockákat.**A kamerák a mentések között folyamatosan rögzítenek.** Minden intervallumban a legújabb szinkronizált képcsoport kerül mentésre; a köztes képkockák nem kerülnek mentésre. Az automatikus expozíció minden képkockánál beállítódik, így a mentéskor az expozíció már stabilizálódott. Nincs olyan üzemmód, amelyben a kamerák a felvételek között leállnának, és minden egyes felvételhez újra bekapcsolnának: ilyenkor ugyanis minden felvételnél meg kellene várni, amíg az adatfolyam elindul és az expozíció stabilizálódik.**Állítsa be a képkockasebességet az intervallum igényeinek megfelelően.**Automatikus beállítás esetén a rendszer a gazdagép és a hálózat által támogatott legmagasabb sebességgel működik. Alacsonyabb értékre állítva csökkenthető a kamerák hőtermelése, a hálózati terhelés és a gazdagép CPU-terhelése anélkül, hogy a mentett adatok mennyisége megváltozna. Válasszon olyan sebességet, amely**intervallumonként 2–4 képkockát** biztosít:

- Minden mentés a kérés után megvárja a következő képkockát, így egy mentett csoport legfeljebb egy képkocka-időszakkal késik a tervezett időponttól. A magasabb képkockasebesség csökkenti ezt a késedelmet.
- Az automatikus expozíció képkockánként egyszer áll be, így egy nagyon alacsony képkockasebesség lassítja, hogy az expozíció milyen gyorsan követi az elhaladó felhőt.

**Számítsa ki az intervallumot a repülési terv alapján:**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

Például egy 60 m-es lefedettség 80%-os előremeneti átfedés mellett 12 m-es távolságot jelent; 10 m/s sebességnél ez az `interval_s=1.2`, és egy rögzített `target_fps=3` érték esetén ez intervallumonként 3–4 képkockát jelent. A felvétel időalapú: az SDK-nek nincs távolság- vagy GPS-vezérelt üzemmódja, ezért a felmérési vonalakon tartsa állandó szárazföldi sebességet.

**Mentse a nyers adatokat a repülőgépen, és később kalibrálja őket.** Az `interval_s` értéket minden mentés kezdetétől mérik, így ha egy mentés hosszabb ideig tart, mint az intervallum, a következő mentés késve kezdődik, és a távolság eltolódik. A nyers adatok és a fényérzékelő értékeinek mentése (`processing="raw"`; az értékek alapértelmezés szerint mentésre kerülnek) rövidre szorítja az egyes mentéseket, és biztosítja, hogy minden képkocka utólag reflektanciára konvertálható legyen.

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**a kezdetektől rögzíti a sebességet;**`arr.set_target_fps(fps)`**repülés közben módosítja anélkül, hogy újra csatlakozna, és**`arr.set_target_fps(None)`** visszaállítja automatikus üzemmódra. A gazdagép és a hálózat által kezelhetőnél magasabb sebességet a felső határra csökkentik — ellenőrizze az `clamped` értéket az eredményben —, és a legalacsonyabb sebesség 0,1 fps.
- **Ne használja az `burst()` parancsot a képkockák ritkítására.** Ez a parancs a tömb által létrehozott minden képkockát kiír; követi a rögzített `target_fps` parancsot, így csak akkor a megfelelő eszköz, ha minden képkockára szükség van.
- **Az `target_fps` nem javítja ki a sérült képkockákat.** Az érték csökkentése a generált képkockák számát csökkenti, nem pedig az egyes képkockák küldési sebességét; a GVSP által sérült képkockák esetén inkább az `wire_ceiling_mbps` értéket csökkentse (lásd [Tömb állapota](#array-health--which-subsystem-is-losing-frames)).

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### Csatlakozás egy már csatlakoztatott tömbhöz — `attach_array`

Ha a tömb már aktív (a GUI megnyitotta, vagy egy korábbi SDK-munkamenet hívta meg az `connect_array` parancsot), akkor az újracsatlakozás helyett az `attach_array` parancsot használja a tömb hivatkozójának megszerzéséhez. Az `connect_array` ebben az esetben mindig „Camera  <sn>már szerepel a tömbben<id>” hibaüzenetet ad ki, mivel a `/array/connect` parancs POST-küldése egy poolban lévő elemre nem idempotens; az `attach_array` beolvassa az `/api/camera/array/list`-et, és az array_id vagy a sorozatszám alapján végez összehasonlítást.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Minta: Az asztali grafikus felhasználói felülettel együtt működő SDK-parancsfájloknak először az `attach_array`-et kell megpróbálniuk, és csak akkor kell az `connect_array`-re váltaniuk, ha még nincs tömb a poolban.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **A csatolt tömb kölcsönzött, nem saját tulajdonú.** Az `attach_array`-ből származó kezelő soha nem szünteti meg a tömböt: sem az `with` blokk kilépése, sem egy kifejezett `disconnect()` nem szabadítja fel (a kifejezett hívás egy sort ír ki a stderr-re, amelyben azt jelzi, hogy a tömb továbbra is fut). Ugyanez a szabály vonatkozik az `CameraSession` / `DAQSensorSession`-re is. Tehát a GUI-val való együttes használat biztonságos az `with` használatával vagy anélkül is:
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> Ha tényleg be akarod fejezni — például azért, hogy egy másik `wire_ceiling_mbps`-hez csatlakozz újra — hívd meg az `arr.force_disconnect()` parancsot. Ha egy csatlakoztatott kezelőn egy egyszerű `disconnect()` után próbálsz újra csatlakozni, a művelet hibaüzenettel végződik: „A kamera <sn>már </sn>szerepel <sn>a tömbben<id>”, mivel semmit sem szabadítottak fel.

### Hálózati elemző segédprogram

Hasznos a tömb megnyitása előtt – előre jelzi, hogy a javasolt beállítások megfelelőek-e:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

Soha nem indít el háttérprogramot: ha egy háttérprogram nem válaszol, az `ChlorosConnectError` hibát eredményez, míg egy elutasított kérés eredményként visszajön. Egy hibás `packed` hiba `ValueError` hibát eredményez.

Az `status` az `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` közé tartozik (egyéb esetben az `error` — beleértve az elutasított kéréseket is: nincs bejelentkezve, `error_code: AUTH_REQUIRED`, vagy az SDK-küszöb alatti tervezet, `PLAN_UPGRADE_REQUIRED`). Az `auto_capped_fps`A  azt jelenti, hogy a kért felbontás csak egy korlátozott trigger-frekvencián illeszkedik az RX gyűrűhöz — tartsa meg a felbontást, és lépjen át az `target_fps=result["recommended"]["recommended_target_fps"]`-ről az `connect_array`-re (lásd a [6. példát](#6-capability-probe-before-connecting-a-4-cam-array)). Az `auto_shrunk` csak akkor jelenik meg, ha a kért képkocka semmilyen képkockasebességgel nem szállítható; a connect soha nem zsugorítja önállóan a képkockákat, ezért az `recommended`-et saját maga kell alkalmaznia.

**A vetület értelmezése** (ugyanaz a modell, mint a GUI Array Settings panelen):

- **A Burst (`frame_bytes_total`) kameránként összeadódik az egyes kamerák valós képpontformátumában.**Mono**M3M**kamerák Mono12-t (2 B/px) streamelnek, vagy egy `pixel_format` beállítással a mélységi adatuk Mono formátumát (az `"BayerRG8"` az `Mono8`, 1 B/px), így ha nincs ilyen, egy 4 kamerás, teljes felbontású képkocka mérete**~25 MB** három mono kamerával, nem pedig a ~12,6 MB, amit a teljes 8 bites feltételezés adna. A háttérrendszer a modell alapján határozza meg az egyes kamerák formátumát.
- **Az Admittance (`burst_fits_nic_ring`) figyelembe veszi a leürítési sebességet**, nem pedig a teljes burst és a gyűrű közötti különbséget veszi figyelembe: a sim-emit akkor működik, ha a gazdagép gyorsabban üríti ki az RX-gyűrűt, mint ahogy a kamerák megtöltik. Egy 10G-s gazdagép + 1 GbE-s kamerák**átadják** a teljes felbontású képet akkor is, ha a burst meghaladja a gyűrű kapacitását; egy 1 GbE-s gazdagép viszont blokkolja (`needs_force_slip` / `auto_shrunk`).
- **Az `achievable_fps_max` egy konzervatív soros adatletöltési felső határ** — az `max(readout+emit, N×emit)` esetében a kameránkénti kibocsátás az 1 GbE kamerakapcsolatra van korlátozva, az expozíciótól függetlenül. Például ~2,8 fps egy 4 kamerás, teljes felbontású, 12 bites tömb esetén (megegyezik a futásidő alatt mért ~2,7–3,0 értékkel). Teljes modell: [CLI-referencia → Képkockasorozat fps és sorozatfelvétel modell](cli-reference.md#array-fps--burst-model).
- **Folyamatos rögzítés után a kivetítés megmutatja, milyen értéket tartott.** Egy 30 másodperces vagy annál hosszabb, automatikus sebességgel végzett rögzítés azt a sebességet rögzíti, amelyre az adott sorozatméret mellett ezen a számítógépen állt be; a sebességszabályozó gyorsabb sebességre irányuló rövid kísérleteit a rendszer kihagyja. Ettől kezdve az `array-connect`sorától kezdve az `connect_array()` elemzése és a Tömbbeállítások panel is ezt a sebességet jelzi, ha az alacsonyabb a modellben megadottnál, feltüntetve, mi korlátozta azt az `cycle_limiter`-ben: `"clock"` (a tömb a végleges sebességgel futott, a rögzítő hurok pedig a trigger órajelre várt), az `"loop"` (a rögzítő huroknak nem maradt ideje), `"capture"` (a rögzítés lemaradt és csoportokat hagyott ki) vagy `"measured"` (nem derül ki, melyik). A modell értéke az `wire_fps_max`-ben található. A tömb továbbra is a vezeték által továbbított értékre emelkedik.
- **A túlterhelés (`oversubscribed: true`) azt jelenti, hogy a vezeték nem képes továbbítani a tömböt a kamerák által elért bármely képfrissítési sebesség mellett** — az fps mezők (`achievable_fps_max` / `fps_bright` / `fps_dark`) értéke 0, és a binning sem tudja ezt orvosolni (ez ugyanis a képkockánkénti bájtok számát csökkenti, nem pedig a másodpercenkénti szabályozott bájtok számát). Egy lassú kábel önmagában nem okoz ilyet: ehelyett a képkockasebesség csökken, és ha a kamerák nem szabályozhatók elég lassúra, a szint `slip-emit-and-capture` (soros) szintre. A megoldások: kevesebb kamera, jumbo keretek vagy gyorsabb hálózati kártya; az `max_cams_collision_safe` jelzi a vezeték és a kamerák maximális teljesítményét. A válasz az `aggregate_demand_bps`, `collision_safe_ceiling_bps` és `per_cam_floor_bps` kódokat is tartalmazza. Lásd [Túlterhelés](#over-subscription-the-per-cam-floor).

### Felfedezés és felsorolás

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

Mindkettő csak kulcsszavakat fogad el (`backend_url=` és `timeout=30.0`), és szótárak listáját adja vissza. `discover_lattice_cameras()` (`serial`, `ip`, `mac`, `model`, `lattice_model`, `vendor`, `connected`, `array_id`, …) az `auto_start_backend=True`-et is elfogadja; az `list_*` hívások soha nem indítanak el háttérfolyamatot.

---

## Smart-AE / Smart-Capture

A LATTICE-tömbök a csatlakozás után azonnal folyamatos AE-t futtatnak a háttérben, de egy frissen beállított jelenet konvergenciája egy pillanatig tart. **A Smart-Capture** egy kényelmi funkció: lekérdezi az egyes kamerák expozícióját, megvárja, amíg a rendszer egy ablakon belül stabilizálódik, majd elindítja a rögzítést. Ez megegyezik a grafikus felületen található funkcióval: az asztali alkalmazás „smart” rögzítés gombja ugyanazt a háttérprogram végpontot hívja meg.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Az `ChlorosProject` (következő szakasz) használatával további beállítási lehetőségek állnak rendelkezésre:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Az intelligens AE-beállítás alapértelmezés szerint konzervatív. Szigorítsa az `exposure_tolerance_pct` értéket igényes radiometrikus munkákhoz; lazítsa meg gyorsan változó jelenetek esetén, ahol csak „elég közel” kell lennie.

---

## DAQ-érzékelő munkamenetek

Állandó háttérpool spektrális érzékelőkhöz (DAQ-U USB-n, DAQ-M BLE-n, DAQ-E Etherneten). Tükrözi a kamera felületét: intelligens felismerés, pool újrafelhasználás, idempotens csatlakozás.

### Intelligens felismerés (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Elsőbbségi sorrend: Ethernet → BLE → USB. Bármelyik explicit utalás megadása rögzíti az átviteli módot.

### Rögzített átviteli mód

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

Az `sensor_id=` egy érzékelőt nevez meg, és kizárólag az az érzékelő nyílik meg. A rendszer összehasonlítja a már megnyitott érzékelőkkel, majd egy DAQ-E gazdagépnevét vagy IP-címét (a hálózati szkennelés során kizárólag IP-címen látható DAQ-E-től olvasási joggal lekérdezi a nevét, MAC-címét és sorozatszámát, így bármelyikük is megtalálja), a DAQ-M BLE MAC-címét, valamint a DAQ-U vagy DAQ-M által megnyitáskor jelentett azonosítót (bármely más, ellenőrzés céljából megnyitott eszközt újra bezárnak). Ha nem találják meg, az `connect_daq_sensor` kiadja az `ChlorosConnectError` parancsot, felsorolva a talált elemeket; címmel együtt (`port=` / `mac=` / `eth_host=`), amely egy másik érzékelőt tartalmaz, az is hibaüzenetet generál. Soha nem tér vissza arra az érzékelőre, amelyik látható – erre szolgál a zero-flag hívás.</id></sn></id></sn>

### `connect_daq_sensor()` aláírás

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

A már a készletben lévő érzékelőt változatlanul (kölcsönzöttként) adjuk vissza: az `integration_time`, az `frame_avg` és az `enable_ae` nem kerül rá alkalmazásra. Először válassza le, hogy a saját beállításaival nyissa meg. Az `cap_id` kivételt képez: egy kölcsönzött érzékelőn az `set_cap()`-szel van beállítva. Ha kihagyja, a háttérrendszer alapértelmezett értéke érvényes (lásd a módszertáblázat alatti megjegyzést).

### `DAQSensorSession` módszerek

| Módszer | Leírás |
| --- | --- |
| `status(timeout=10.0)` | Pool-bejegyzés összefoglalása (adatfolyam/felvétel állapota, hullámhossz-tartomány, kalibrációs sha, integrációs idő, frame_avg, AE állapot). Olyan DAQ-E esetén, amelynek adatfolyama elhallgatott: `reconnect` (ahol a háttérrendszer automatikus újracsatlakozása működik: `state`, `attempts`, `next_attempt_in_s`, `last_error`; `None`, amíg adatfolyamot továbbít), `stream_lost_s`, `reconnects` és `last_reconnect`, valamint `record_gaps` a a folyamatban lévő felvételhez; az `rx_stream_fault` jelzi az újracsatlakozási állapotot, amíg meg nem érkezik az első spektrum. |
| `latest(*, n=1, timeout=10.0)` | Az N legutóbbi spektrumkeret visszaadása (mindegyik `ts_ns`, `spectrum`, `is_saturated`, `integration_time`, `x`, `y`, `z`). |
| `stream_start()` / `stream_stop()` | A streamelés folytatása / szüneteltetése (a kezelő nyitott marad). |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | .daq fájl felvételének indítása. Visszaadja a fájl elérési útját. Elutasítja a DAQ-U/M esetében, ha nincs AWS kalibrációs csomag (a DAQ-E kivétel). |
| `record_stop()` | A felvétel leállítása. Visszaadja az `{path, rows, warning, gaps}` értéket; az `warning` értéke `None`, ha a felvétel hibátlan, ellenkező esetben a háttérrendszer diagnózisát (például üres fájl vagy rés a adatfolyamban), amelyet a stderr-re is kinyomtat. Az `gaps` felsorolja azokat a szakaszokat, amikor az adatfolyam nem szállított semmit: `start_utc`, `end_utc` (`None`: a leálláskor még mindig elveszett), `duration_s`, `reconnected`, `marked_in_file`. Folyamatos felvétel esetén üres. |
| `set_cap(cap_id, timeout=15.0)` | Állítsa be a következőtől kezdve az olvasmányokra alkalmazandó felső határ-korrekciós profilt: `"sunshine_cosine"`, `"fov_15"` ... `"fov_90"`, vagy `"none"` (vagy `None`) csupasz érzékelő esetén. Visszaadja az érzékelőösszefoglalóját, amelyet az `cap_id` olvas vissza. A DAQ-E-S az `as_recorded` értéket tartja fenn, bármit is kérdeznek tőle (beépített diffúzorral rendelkezik). Az `ChlorosConnectError` értéket jelzi egy olyan profil esetén, amely nem töltődik be. CLI parancs: `chloros-cli daq pool-set-cap`. |
| `enable_ptp(timeout=15.0)` | Csak DAQ-E esetén: kapcsolja vissza a PTP-t olyan egységnél, amelynek PTP-je ki van kapcsolva (az `status()["rx_ptp_enabled"]` az `False`; a beállítás az egységre van mentve, így kikapcsolt állapotban marad, amíg újra be nem kapcsolják). Visszaadja a frissített állapotot. `ChlorosConnectError` hibakódot generál DAQ-U / DAQ-M esetén, vagy ha a készülék nem nyújt visszaigazolást. Az Chloros soha nem kapcsolja vissza önállóan a készüléket, mivel a kikapcsolt állapot szándékos is lehet. CLI parancs: `chloros-cli daq pool-enable-ptp`. |
| `device_info(timeout=15.0)` | Csak DAQ-E esetén: a készülék saját beállításai. `firmware` (`running`, `bundled`, `update_available`), `ip` és `ip_mode` (`dhcp` / `linklocal` / `static`: amit jelenleg futtat), `network` (a mentett beállítás: `mode`, valamint `ip` / `mask` / `gateway` statikus üzemmódban), `network_change_pending` (addig igaz, amíg a készülék a mentett beállítást futtatja, amely az `set_network(..., reboot=False)`-et is tartalmazza), `auto_stream`, `ptp`. Az `firmware["update_available"]` értéke `None`, ha a készülékverziója nem olvasható. Amíg a készülék adatfolyama nem éri el ezt a számítógépet, addig egy új kapcsolaton keresztül kiolvasásra kerül, és a szótár hozzáadja az `stream_silent` (`True`), `stream_fault` és `reconnect` bejegyzéseket; ha egy egység nem válaszol, hibaüzenet jelenik meg. CLI: `pool-info`. |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | DAQ-E csak: DHCP (`"auto"`) vagy statikus cím. Az egységet újraindítja a beállítások alkalmazásához, majd újra csatlakoztatja; ez a munkamenet továbbra is működik, és az `address` követi az egységet. Az `reboot=False` csak elmenti a beállításokat. Lásd: [DAQ-E eszközbeállítások](#daq-e-device-settings). CLI: `pool-set-network`. |
| `set_auto_stream(enabled, timeout=15.0)` | Csak DAQ-E: adatfolyam indítása bekapcsoláskor szoftver nélkül (az eszközön mentve). Visszaadja a megerősített beállítást. CLI: `pool-set-auto-stream`. |
| `reboot(*, comeback_timeout=45.0)` | Csak DAQ-E: indítsa újra a készüléket, és csatlakoztassa újra a korábbi beállításokkal. CLI: `pool-reboot`. |
| `update_firmware(*, comeback_timeout=45.0)` | Csak DAQ-E: a Chloros kiadáshoz tartozó firmware-t írja be egy régebbi verziót futtató készülékre, majd csatlakoztassa újra (kb. egy perc; ne kapcsolja ki). Visszaadja az `updated`, `fw_before`, `fw_after`, `bundled` és egy `warning` kódot, ha az egység lassan állt vissza, vagy egy másik verzióval állt vissza; az `updated` kód az `False` kódot jelenti egy már naprakész egység esetében. `ChlorosConnectError` hibát dob egy olyan egység esetén, amelynek verziója nem olvasható. CLI parancs: `pool-update-firmware`. |
| `disconnect()` | Felszabadítás a poolból. Nincs-op a csatolt, de nem saját tulajdonú kezelőkhöz. Egy olyan érzékelőn, amelyet ez a munkamenet nyitott meg, ez az `force_disconnect()`, és `ChlorosConnectError` hibát dob, ha a felszabadítás nem kerül megerősítésre (az `with` blokk kilépése soha nem vált ki hibát). |
| `force_disconnect(timeout=30.0)` | Az érzékelő felszabadítása függetlenül attól, hogy ki nyitotta meg — például egy olyan érzékelőt, amelyet egy összeomlott szkript hagyott a poolban, és amelyhez egy új szkript csak csatolt kezelőként fér hozzá, amelynek `disconnect()`-je nem hajt végre semmilyen műveletet. Egy másik kliens érzékelőjének lezárása befejezi az általa végzett felvételt. Visszatér, miután a háttérrendszer megerősíti a felszabadítást, vagy ha nincs mit felszabadítani (az érzékelő nincs a poolban, vagy nincs futó háttérprogram); az időtúllépés miatt megszakadt vagy a háttérprogram által elutasított felszabadítás `ChlorosConnectError` hibát dob. CLI: `chloros-cli daq pool-disconnect --sensor-id ID`. |

Attribútumok: `sensor_id`, `model`, `transport`, `address` (egy olyan DAQ-E-t követ, amelyet az `set_network()` újracímzett), `info` és `backend_url`.

> **Nevezze meg a kupakot!** A DAQ-U, DAQ-M és DAQ-E kondenzátor nélkül kerül forgalomba, és az Chloros nem tudja megállapítani, hogy van-e beépítve: egy `cap_id` nélkül csatlakoztatott érzékelő az `sunshine_cosine` értéket kapja, ami a rendszer felvételei szerint azt jelenti, hogy (`cap_id_source=auto_default`). Egy fedél nélküli DAQ-U e korrekció mellett körülbelül 30-szor magasabb értéket jelez, ezért adja meg az `cap_id="none"` értéket egy fedél nélküli érzékelőhöz (vagy `set_cap("none")`), az egyébként felszerelt kupak profilja pedig másképp.

### DAQ-E-S — rögzített diffúzor és dőlésérzékelő

A **DAQ-E-S** egy DAQ-E, amelynek két, a szoftver szempontjából fontos különbsége van. Minden egyéb – felismerés, csatlakozás, PTP, rögzítés, az alábbi eszközbeállítások – pontosan úgy működik, mint a DAQ-E esetében.**Diffúzorja rögzített, ezért nincs szükség fedélkorrekcióra.** A Sunshine-típusú diffúzor beépített, és már a készüléken volt, amikor a gyári kalibrálását elvégezték, így a kalibrálás már a valódi lefelé irányuló besugárzást méri. A Chloros bármilyen kérésre is az `as_recorded` (profil nélkül) sapkát állítja be: a számára elnevezett sapkát (`sunshine_cosine`, `fov_*`, `none`) egy üzenettel figyelmen kívül marad, a felvételei pedig az `cap_id_source=model` értéket tárolják, így a feltételezett korlátra vonatkozó figyelmeztetés soha nem lép működésbe. Ne alkalmazzon kézzel sem felső határt az adataira: a legfelső `sunshine_cosine` értéke körülbelül 11-szerese lenne a valósnak.

**3-tengelyes gyorsulásmérővel rendelkezik.** A napfény-szóró koszinusz-válasza csak akkor érvényes, ha vízszintes helyzetben van, így egy megdőlt érzékelő hibás besugárzási értéket jelez – és a visszaverődés is átveszi a hibát –, egyéb tünet nélkül. Ezért minden DAQ-E-S-leolvasás tartalmazza a készülék helyzetét:

| Mező | Jelentés |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Szögek fokban. `None`, ha a firmware a szöget meghatározhatatlannak jelöli (szabad esés, meghibásodott tengely). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Gyorsulás milli-g-ben. `mag_mg`, ha az érték messze eltér a ~1000-től, az azt jelenti, hogy a készülék mozgásban volt, és a szögek nem értelmezhetők az adott mintára vonatkozóan. |
| `cal_applied` | Beállítva: a tengelyek és szögek a diffúzor koordinátarendszerében vannak (az egység rögzítési kalibrációja érvényes). Törölve: nyers kártyakoordinátarendszer, amely akár ~2,3°-os, valójában nem létező dőlést is jelezhet. |
| `sample_age_ms` | A gyorsulásmérő-minta kora a spektrum elküldésének időpontjában. |
| `temp_raw` | Kalibrálatlan chip-hőmérséklet **trend** — csak különbségek, soha nem hőmérő-leolvasás. |

Az `DAQSensorSession.latest()` ezeket az egyes keretek `imu` kulcsai alatt adja vissza (`None` egy DAQ-U, DAQ-M vagy egyszerű DAQ-E esetén). A DAQ-E-S-ről rögzített `.daq` ugyanazokat az értékeket tárolja az `imu_*` oszlopokban (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) oszlopokban ugyanazokat az értékeket tárolja minden spektrum mellett, így az olvasó meg tudja különböztetni a felhőt a felborult állványtól. Minden más modellnél ezek az oszlopok üresek.

A helyzetadat-sorozat a saját mezőiről számol be az `status()` (`rx_imu_stream_fault`, `rx_imu_stream_faults`, `rx_last_imu_stream_fault`) mezőkben, elkülönítve a spektrális adatoktól, így a gyorsulásmérő kiesése soha nem értelmezhető a felvételben lévő résként. Az `discover_daq_sensors()` és a munkamenet `model` fájlja az `DAQ-E-S`-et jelzi.

### DAQ-E eszközbeállítások

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **A számítógép hálózatán statikus címnek kell lennie.** Az `set_network` az `ChlorosConnectError` hibát jelzi azoknál, akiknek az alhálózata nem tartalmazza a számítógépaz érzékelő hálózatán: az újraindítás után semmi sem érheti el az érzékelőt, hogy visszaállítsa az eredeti állapotot. A `force=True` hibakódot csak akkor hagyja át, ha ezt a számítógépet átcímkézik, vagy átirányítják az új alhálózatra.
- **Az `set_network`, `reboot` és `update_firmware` újraindítja az érzékelőt**, és az `ChlorosConnectError` hibajelentést generál, miközben az érzékelő rögzíti az adatokat, és ha az `comeback_timeout` másodpercen belül nem áll vissza. Az érzékelő azonosítója és az `daq-e-<id>.local` gazdagépnév soha nem változik.
- **Az újraindítás előtt éppen megkezdett felvételt az újraindítás lezárja és elnevezi.** Amikor az érzékelő visszatér, a visszaküldött dict tartalmazza az `recording_closed` értéket (a fájl elérési útját) és egy `warning` értéket, amely ezt jelzi. Ha nem tér vissza, vagy a firmware-átvitel sikertelen, akkor a `ChlorosConnectError` érték jelzi ezt: az üzenete a figyelmeztetéssel végződik, és a `e.recording_closed` / `e.warning` beállításra kerül (mindkettő `None`, ha nem zárult le felvétel). Indítsa el újra a felvételt, miután az érzékelő visszatért.
- **Csak maga az érzékelő csatlakozik újra.** Az újraindítás után a készüléknek jelentenie kell, hol volt -- és az `set_network("auto")` után az `.local` név alatt is -- a nevét, a gazdagép nevét és a modellt, valamint a korábban használt hálózati adapter (MAC) címét; az `address` lesz az a cím, amelyről a válasz érkezett. A név alatt adott válasz csak az adott MAC-címmel érvényes, így egy olyan számítógépről, amely útválasztón keresztül éri el az érzékelőt, vagy a WSL2 alapértelmezett NAT-hálózatából, a DHCP által áthelyezett érzékelő nem csatlakozik újra (`ChlorosConnectError`; csatlakoztassa újra az új címén).
- **A firmware-frissítéshez az érzékelőnek újra csatlakoznia kell.** `update_firmware()` megkeresi az egységet, és az egység visszanyit egy TCP-kapcsolatot erre a számítógépre a kép letöltéséhez, az operációs rendszer által kiválasztott porton. A háttérrendszer az egységet elérő adapteren figyel (minden adapteren, ha nem tudja megkülönböztetni őket), és kizárólag azt az egységet szolgálja ki: minden egyéb kapcsolatot elutasít (`[DAQ-FW] … refused a firmware connection from <ip>` a háttérprogram kimenetében), miközben továbbra is várakozik. Ha a tűzfal blokkolja a kapcsolatot, a frissítés `device never connected back on TCP <port>` hibakóddal meghiúsul. A Windows-telepítő hozzáadja a szükséges szabályt (lásd [1. lépés](#windows-exe)); egy másik tűzfalon engedélyezze a bejövő TCP-forgalmat a háttérprogram felé az érzékelő alhálózatából.
- **WSL2 alatt használjon tükrözött hálózatot** (`networkingMode=mirrored` az `.wslconfig`-ben): az alapértelmezett NAT módban minden statikus címet elutasít (a WSL saját címe nem szerepel az érzékelő hálózatán), az adatfolyam nem éri el a WSL-t, és a firmware-frissítés nem tud befejeződni.
- A grafikus felület (GUI) saját kapcsolatot tart fenn az érzékelővel: egy szkriptből indított újraindítás véget vet a GUI és az érzékelő közötti munkamenetnek. A grafikus felületen ugyanazok a beállítások érvényesek (Fényérzékelők fül, az érzékelő beállításai, Hálózat).

### Felfedezés — a csatlakozáshoz szükséges cím megkeresése

Az `discover_daq_sensors()` átvizsgálja az USB / BLE / ETH kapcsolatokat olyan érzékelők után, amelyeket *megnyithatna*. Ez az `discover_lattice_cameras()` DAQ-megfelelője, és az egyetlen módja a **DAQ-M BLE MAC-címének** megszerzésére — a DAQ-E-nek van gazdagépneve, a DAQ-U-nak pedig COM-portja, de a MAC-címet sem az eszközön nem nyomtatják ki, sem az operációs rendszer nem sorolja fel.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Mező | Leírás |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM-port / BLE MAC / gazdagépnév — továbbítja az `connect_daq_sensor`-nek, mint `port=` / `mac=` / `eth_host=`. |
| `display` | Ember számára olvasható címke. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`, vagy `None`. Az Ethernet-érzékelő az `DAQ-E-S` vagy az `DAQ-E`, mivel lekérdezésre így azonosítja magát, olvasási joggal, a szkennelés során (`DAQ-E`, ha nem válaszol időben). Egy USB soros port értéke `DAQ-U`, ha a hídchipje olyan, amelyet a DAQ-U is használ (FTDI, CH340, CP210x, Prolific) — erre utal a port leírása, nem pedig egy próba —, míg bármely más USB-eszköz esetében az `None` értéket, amely továbbra is szerepel a listában, így azt `port=` néven adhatod meg. Az a soros port, amelynek nincs USB-gyártói azonosítója és nincs ilyen hídchipje (például egy alaplapi UART, mint a Linux `/dev/ttyS*`-je, egy Bluetooth soros kapcsolat vagy az Intel AMT), nem lehet DAQ-U, és nem szerepel a listán. |
| `extra` | Transzportonkénti részletek (BLE hirdetett név; USB `vid`, `pid`, `manufacturer`, `serial_number`; DAQ-E ip/fw/…). Az üres értékeket kihagyjuk. |

| Paraméter | Alapértelmezett | Leírás |
| --- | --- | --- |
| `transports` | mindhárom | A szkennelést korlátozó sorozat (vagy csv karakterlánc). Érdemes megadni, ha tudod, mit akarsz — a BLE a lassú láncszem. |
| `scan_timeout` | 5 | Szállítási módonkénti szkennelési ablak másodpercben; a háttérrendszer 1–20 közé korlátozza. |
| `timeout` | 60,0 | Az egész hívás HTTP-korlátja (az SDK más részeinél is). |
| `auto_start_backend` | `True` | Helyi háttérprogramot indít, ha még nincs futó. Távoli `backend_url` esetén soha nem indít. |

> **A poolban már megnyitott érzékelők nem jelennek meg.** A csatlakoztatott BLE-periféria leállítja az adását, és egy nyitott COM-portot nem lehet lekérdezni, ezért a felfedezés csak azokat sorolja fel, amelyek *csatlakozásra rendelkezésre állnak*. Várható, hogy közvetlenül a csatlakozás után üres eredményt kap — a már meglévő eszközökhöz használja az `list_daq_sensors()` parancsot. Azokat a transzportokat, amelyek szkennelése nem futtatható (nincs telepítve a bleak / zeroconf), a rendszer kihagyja, ahelyett, hogy hibajelentést adna, így egy Bluetooth nélküli gép is megkapja az USB- és ETH-válaszokat.

### Listázás

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Melyik DAQ táplálja melyik kamerát — `daq_bindings()`

Egy felvétel reflektanciája, valamint az `.daq` fájl, amelybe elmenti, az adott kamerához vagy sorozathoz rendelt DAQ-t használja, egyébként a háttérrendszer legfrissebb DAQ-ját. A grafikus felület fényérzékelő-kiválasztása és az `chloros-cli daq pool-bind` állítja be; ahogyan a következő is:

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

Az `daq_bindings()` az `{"cameras": {serial: b}, "arrays": {array_id: b}}` értéket adja vissza: minden `b` az `binding`-et (a beállított értéket), az `source` (`project`, a projekt `sensors.json`-éből, vagy az `runtime`-ből), az `known` / `known_as` (hogy a kamera vagy a tömb kulcsneve egy-e, amit Chloros ismer, és hogyan: `connected`, `project` — a megnyitott projekt `cameras.json` — vagy, kamera esetén, az `seen`: korábban már megnyitották ezen a számítógépen; az `known: False` egy olyan kulcsra vonatkozik, amely semmit sem nevez meg) és az `resolved` (a pool-érzékelő, amelyhez jelenleg rendelődik: `in_pool`, `sensor_id`, `address`, `model`, `streaming`). A futásidejű kapcsolás felülírja a projekta munkamenet idejére; a projektfájl nem változik. Egy kamera, tömb vagy DAQ leválasztása megszünteti az adott hardverhez tartozó összes kötést, függetlenül attól, ki állította be azokat (a tömb elemei a tömbhöz tartoznak).

Az `set_daq_bindings()` csak olyan kamerát köt össze, amely csatlakoztatva van, a megnyitott projektben szerepel, vagy korábban már megnyitották ezen a számítógépen, valamint olyan tömböt, amely csatlakoztatva van vagy a megnyitott projektben szerepel (vagy egy már összekapcsolt gombot). Bármely más gomb az `ChlorosConnectError` hibát váltja ki, megnevezve azt (`400`, `code`, `UNKNOWN_BINDING_TARGET`, az `unknown` billentyűkkel), és **semmi** nem lesz hozzárendelve abban a hívásban. Az `None` törli az összes kulcsot.

### Környezeti + objektum-reflektancia — `light_sensor_reflectance()`

Két összekapcsolt DAQ, amelyek közül az egyik az égre (környezeti fény), a másik pedig a célpontra (objektum) néz, a legfrissebb mérési eredmények alapján megadja a célpont visszaverődési és növényzetindexét: a GUI „Környezeti fény + Objektum” párját, valamint az `chloros-cli daq reflectance`-et anélkül, hogy az érzékelőket másodszor is megnyitná. Mindkettőnek csatlakoztatva kell lennie, és adatátvitelben kell lennie.

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **Sávok.** Minden sáv visszaverődése az objektum sávjába-integrált besugárzása a környezeti értékhez viszonyítva. Az `daq_band_presets(filter_code=None)` felsorolja a sávkészleteket: egyet minden LATTICE M3C szűrőhöz (`RGN`, `OCN`, `NGB`, a kamera saját sávmodellje, így a DAQ NDVI és a kamera NDVI ugyanazt a mennyiséget jelenti), valamint az `GENERIC` szélessávú. Az `suggested` az előre beállított, az `filter_code`-hez illeszkedő sáv, egyébként az első csatlakoztatott kamera szűrője (az `suggested_from` jelzi, melyik).
- **Visszaadja** az `bands` értéket (sávonként: `rho`, `valid`, `reason`, `e_ambient`, `e_object`), `indices` (a beépítettbeépített), `custom` (mindegyikhez tartozik egy `value`, vagy egy `valid: False` és egy `error`), `invalid_bands`, `reflectance_type`, `preset` és a két mérési érték: `timestamps` (`ts_ns`). Az a tartomány, amelyet a környezeti érzékelő nem tudott mérni (a mérési tartományon kívül, a zajszintjén), érvénytelennek minősül, az ok feltüntetésével, és minden olyan index, amelyhez ez szükséges, kihagyásra kerül, soha nem számítják ki nullához viszonyítva.
- **Látszólagos vagy relatív.** A skálától függő indexek (SAVI, OSAVI, EVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) abszolút arányt igényelnek, ezért csak akkor jelennek meg, ha mindkét érzékelő teljesítménykalibrálva van (mindkettő rendelkezik `calibration_bundle_sha`-szel): `"Apparent Reflectance"`. Ellenkező esetben az eredmény `"Relative Reflectance"`, kizárólag a skála-invariáns indexekkel.
- **Két különböző hullámhossz-rács** átfedését hasonlítjuk össze: a környezeti érzékelő pontjai az objektum hatótávolságán belül vannak, az objektum spektrumát pedig ezekre interpoláljuk.
- **Két érzékelő, soha nem egy.** Ha egy érzékelőt mindkettőként megnevezünk (ugyanaz az azonosító, bármilyen írásmódban, amelyet a pool elfogad), az `ChlorosConnectError` hibát eredményez: egy érzékelő saját magáról mért értéke minden sávban 1,0, ami semmit sem mér. A háttérrendszer is elutasítja (`POST /api/daq/indices` válaszol az `400`-re, `code`-re az `SAME_SENSOR`-et, miközben az `ambient_sensor_id` és az `object_sensor_id`, amelyeket az SDK küld, egy poolba tartozó érzékelőt jelentenek, pl. annak azonosítóját és portját jelenti).

### Közös használat a CLI-vel és más szkriptekkel

A háttérrendszer DAQ-érzékelő-poolját a gépen minden CLI- és SDK-kliens megosztja. Ha egy érzékelő már a poolban van — az `chloros-cli daq pool-connect` vagy egy másik szkript által megnyitva —, akkor az `connect_daq_sensor(port="COM3")` Pythonból történő meghívása egy kölcsönzött kezelőt ad vissza: az `disconnect()` hívás nem hajt végre műveletet, így a szkripted nem vonja el az érzékelőt attól az ügyféltől, amelyik megnyitotta azt, amikor kilépsz a scope-ból. Ugyanez vonatkozik az `sensor_id=` parancsra is (amelyet az `:` vagy az `-` parancsokkal lehet megadni, mindkettő megfelel), egy olyan DAQ-E esetében, amelyet megnyitáskor a gazdagépnév alapján az IP-címével neveztek meg (vagy fordítva), valamint a nulla jelzővel ellátott intelligens felismerés esetén, amikor a háttérrendszer egy már birtokában lévő érzékelővel válaszol — egy `with` blokk kizárólag azt az érzékelőt szabadítja fel, amelyet saját maga nyitott meg.

A Chloros GUI **nem** része ennek a poolnak: a DAQ-panele saját kapcsolatot létesít az érzékelővel, így az `connect_daq_sensor` soha nem veszi át azt az érzékelőt, amelyet a GUI már megnyitott. Ne használja egyszerre ugyanazt az érzékelőt a GUI-ból és egy szkriptből. A GUI által általában tartott DAQ-U vagy DAQ-M egyszerűen nem nyílik meg, de egy DAQ-E elfogad egy második munkamenetet: a szkript connect parancsa megváltoztatja az egység adatgyűjtési beállításait a GUI alatt, az `disconnect()` parancs pedig leállítja az egység adatfolyamát, ami véget vet a GUI élőképernyőjének és az esetleges felvételeknek. Először zárja be az érzékelőt a GUI-ban.

### PTP Grandmaster — `time_sync_status()`

A háttérprogramban fut a PTP grandmaster, amelyből a LATTICE kamerák és a DAQ-E egységek veszik az időbélyegeiket. Igény szerint indul: az `connect_camera()`, `connect_array()`, `connect_daq_sensor()` és `connect_all()` indítja el, ha még nem fut – beleértve azokat a csatlakozási kísérleteket is, amelyek később sikertelenek, például amikor az `connect_all()` egy nem létező DAQ-E-t próbál meg elérni –, és ugyanezt teszi az `test_array_sync()` is, valamint minden olyan hívás, amely egy parancs végrehajtásához kamerákat nyit meg (`capture_cameras_transient()`, `start_multi_camera_stream()`, az igazítási hívások, `capture_live_index()`).

- **Hol jelenti be.** A kamerák felé irányuló vezetékes hálózati adaptereken (beleértve egy `169.254.x.x` link-local címet is), egyébként az alapértelmezett útvonal adapterén, ha az vezetékes. Soha nem Wi-Fi-n vagy virtuális adapteren (Tailscale, WSL): azok mögött nincs kamera vagy DAQ-E. Ha nincs ilyen adapter aktív, akkor semmire sem hirdet (`health` `NO_SOCKETS`), és a következő kamera- vagy DAQ-csatlakozáskor az ahhoz tartozó adapterhez kötődik.
- **Kapcsolatmegszakadás után.** Amikor a kameraadapter elveszíti a kapcsolatot, majd visszanyeri (kábelkihúzás, kapcsolócsere, USB-adapter újraszámlálása esetén), a grandmaster újraindítás nélkül önállóan újjáépíti a socketjeit, és újraellenőrzi az összes csatlakoztatott DAQ-E-t, újraaktiválva azokat, amelyeknél a kapcsolat megszakadt. Akkor is újjáépíti a kapcsolatokat, ha az általa kiszolgált összes kamera és DAQ-E 15 másodpercig nem ad jelet.
- **Eléri a kábelt?** Olvassa el az `time_sync_status()` (`GET /api/time-sync/status`) értéket. Az `state` csak azt jelzi, hogy a grandmaster szál fut; az `health` pedig azt mutatja, hogy a csomagjai elhagyják-e a rendszert: `OK` (nincs hiba), `STOPPED`, `NO_SOCKETS`, `SENDS_FAILING`, `DEGRADED` (néhány adapter meghibásodott) vagy `PEERS_SILENT` (a kiszolgált kamerák és DAQ-ek nem válaszolnak), egy mondattal az `health_detail`-ben. `announce_sent` / `sync_sent` / `delay_resps_sent` csak a kimenő üzeneteket számolja; `announce_send_errors`, `sync_send_errors`, `follow_up_send_errors`, `delay_resp_send_errors`, `last_send_error`, az `failing_interfaces` és az `last_delay_req_age_s` azokat írja le, amelyek nem távoztak, az `rebinds` / `last_rebind` pedig az újjáépítéseket. Az `sync_sends_slow` azokat a szinkronizálásokat számolja, amelyek küldése meghaladta a 200 us-t, az `sync_send_max_us` pedig a leghosszabbat; minden Follow_Up eredeti ideje legfeljebb 200 us-t számol a küldéséből. A grandmaster saját folyamatban fut (`own_process`, `process_pid`), Windows rendszeren magas prioritással (`process_priority`), így egy leterhelt háttérrendszer nem késleltetheti a bélyegzési időket, és nem állíthatja át egyszerre az összes kamera óráját; ha ez a folyamat nem indítható el, akkor a háttérrendszeren belül fut, mint korábban (`own_process` false).

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| Funkció | Mit csinál |
| --- | --- |
| `time_sync_status()` | A grandmaster állapota, épsége, óraazonosítója, prioritásai és üzenetszámlálói (fent). `chloros-cli time-sync status`. |
| `time_sync_peers()` | Az általa érzékelt PTP-szolgák (kamerák, DAQ-E-k): `peers`, mindegyik `port_identity`, `last_seen_ip`, `last_delay_req_age_ms`, `delay_req_count`. `time-sync peers`. |
| `time_sync_cameras()` | Minden csatlakoztatott tömbkamera: `PtpStatus`, `PtpOffsetFromMaster` és `PtpMeanPathDelay` (azoknál a kameráknál, amelyek firmware-jében nincsenek PTP-csomópontok, csak a sorozatszám szerepel). `time-sync cameras`. |
| `restart_time_sync()` | A grandmaster leállítása és újraindítása; ezután minden csoportosított DAQ-E PTP-je újraellenőrzésre kerül (`daq_ptp_rechecked`). `ChlorosConnectError` hibajelentést generál, ha nem tér vissza. `time-sync restart`. |
| `set_time_sync_priority(priority1=None, priority2=None)` | PTP-prioritásai (0–255, az alacsonyabb nyeri a legjobb mester választást), egy másik PTP-mesterrel rendelkező hálózat esetén. A működéséhez szükséges. `time-sync set-priority`. |

Mindegyik az `backend_url=` és az `timeout=` parancsokat veszi át, és elindít egy olyan helyi háttérprogramot, amely még nem fut.

---

## Kamera segédprogramok

Mit csinál az `chloros-cli lattice` a csatlakozáson és a rögzítésen túl, mint egyszerű függvények az `chloros_sdk`-ben (`chloros_sdk.camera_utils` modul). Mindkettő az CLI-parancs által használt háttérprogram-útvonalat hívja meg, az CLI szabályaival, így a kettő hasonlóan viselkedik. A programok a kamera **soros** számát veszik át, nem pedig egy munkamenetet, és ugyanazokat az `backend_url=` / `timeout=` kulcsszavakat használják, mint a smart-connect hívások; a nem futó háttérrendszert elindítják, ahogyan az `connect_camera()` is teszi. Elutasítás esetén az `ChlorosConnectError` hibaüzenet jelenik meg a háttérrendszer indoklásával.

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| Funkció | Mit csinál |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | A kijelző színprofilja: `raw`, `linear`, `natural`, `enhanced` vagy `custom_temp`, valamint annak színhőmérséklete (2000–10000 K tartományba korlátozva; akkor olvasható ki, ha a profil `custom_temp`). |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | A kijelző telítettsége és kontrasztja, mindkettő 0–200 % (100 = változatlan); az `reset=True` mindkettőt visszaállítja 100-ra. |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | A kijelző legragyogóbb, legfehérebb képpontjainak értékét csökkenti: -100..0 (nullára csonkítva, majd korlátozva; 0 = ki). |
| `camera_white_balance_once(serial, settle_s=None)` | Egyszeri fehér egyensúly: a kamera rövid ideig streamel, konvergál, majd visszaadja az `red_ratio` / `blue_ratio` értéket (`ratios_read` False, ha nem tudja azokat jelenteni). Megtartja a kiszámított fehéregyensúlyt; az open által írt minden egyéb beállítás a talált értékre áll vissza. |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` (folyamatos) vagy `"off"`; vagy rögzített `red` / `blue` egyensúlyi arányok, amelyeknél az automatikus beállítás kapcsolódik ki először. Az a arány, amelyet a kamera elutasít, az `errors` név alatt szerepel. |
| `camera_presets()` / `apply_camera_preset(serial, preset)` | A kamera előre beállított értékei, és az egyik alkalmazása esetén: a kamera megtartja a beállításokat, amelyek az előre beállított nevek (a válasz `settings`, mivel a kamera ezt olvassa ki utána, pl. a futási képsebességet; `None` egy olyan mezőre, amelyet az előre beállított profil nem nevez meg), és minden egyéb beállítás, amelyet az open parancs írt, visszaáll a talált értékre. Az `"default"` visszaállítja a kamerát a gyári alapértékekre (gyári beállítások szabadon futó automatikus expozícióval, a kapcsolati átviteli sebesség korlátozásával és a hibajavítás kikapcsolásával, az RGB-kamera kalibrált fehéregyensúlyának megőrzésével), anélkül, hogy azokat a memóriájába mentené: a válasz tartalmazza az `factory_defaults: True`-et és az `settings` minden mezőjét. Az `camera_presets()`-ben az `default` az `{factory_defaults: True, description}`. |
| `camera_filter_models()` / `camera_filter_model_info(name)` | A szűrőmodellek, valamint egy modell részletei (sávok, spektrális válasz kondíciós száma). A kamera szűrőmodelljét a modell karakterláncából olvassa be a rendszer; ezt nem lehet kézzel beállítani. |
| `camera_firmware_status(serial=None)` | Az egyes kamerákfirmware-je: `current_version`, `available_version`, `update_offered`, `needs_minimum_update` és `minimum_firmware`. |
| `update_camera_firmware(serial, force=False)` | Az Chloros-verzió firmware-jének feltelepítése egy kamerára: gyári beállítások visszaállítása, feltelepítés, újraindítás, a Chloros alapbeállításainak újraírása; a kamera kalibrálása megmarad. Néhány percig tart; a kamera áramellátását fenn kell tartani. Az `force=True` újraírja az aktuális kamera firmware-jét. |
| `camera_power_probe(serial)` | A kamera áramellátási és hőmérsékleti csomópontjai, valamint hőmérséklete. |
| `sleep_camera(serial)` / `wake_camera(serial)` | A kamera alacsony fogyasztású készenléti állapota (alvó állapotban marad, amíg fel nem ébresztik: a kapcsolat átviteli sebesség-korlátja be van kapcsolva (`DeviceLinkThroughputLimitMode` `On`) a legalacsonyabb értéken, az energiatakarékos üzemmód be van kapcsolva, ha a kamerának van ilyen), a hőmérséklet az ébresztés előtt és után; az ébresztés a kapcsolatot olyan állapotba hozza vissza, amilyenben az alvó állapotban volt (a korlát, az üzemmód és a csomagkésleltetés `GevSCPD`, amelyet a kamera 0-ra tart, amíg a korlát be van kapcsolva; a feljegyzést ezen a számítógépen tárolják, így egy későbbi háttérrendszerből történő ébresztés is megtalálja), vagy egy olyan kamera esetében, amelyet ez a számítógép nem állított alvó üzemmódba, de alvónak tűnik (a korlátja a legalacsonyabb szinten van), a korlátot kikapcsolva, az `GevSCPD` értéket pedig 0-ra állítva 80-nál (gyári beállítások); a küszöbérték, amelyet a minimális érték felett állítottak be, változatlan marad. Csak az energiaállapotot tárolják; minden egyéb beállítás, amelyre az open parancs írt, visszaáll a talált értékre. |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | Képkockák rögzítése mindaddig, amíg egy MAPIR kalibrációs célpont a látómezőbe nem kerül, majd a csatornánkénti együtthatók illesztése a paneljeiből: `target_id`, `panel_dns`, `coefficients`. |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | A számítógép GigE Vision-hez használt Ethernet-adapterjeinek beállításai (MTU, kapcsolat sebessége, jumbo keretek) és az esetleges `issues`; az `estimate=True` kiegészíti a képkockasebességet és a sávszélességet, amelyet az `cameras` kamerák kapnának. |
| `fix_network_settings()` | Alkalmazza a GigE Vision adapter beállításait a számítógépkameraadapterére. Ez megváltoztatja az adapter beállításait, és csak akkor működik, ha a Chloros rendszergazdaként fut; ellenkező esetben a talált adatok naplójával együtt hibaüzenetet jelenít meg. |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | Vignettakorrekciót hajt végre egy mappában található képeken, mindegyikhez a saját objektívjének síkkép-térképét használva: `processed`, `lens_models` (az egyes fájlok korrekciójához használt objektív), `skipped` (az okokkal együtt). Az `lens_model` hiányában az egyes fájlok objektívje az EXIF Model mezőből származik; az `lens_model` azokra a fájlokra vonatkozik, amelyek modellje nem nevez meg fényképezőgépet, és az olyan fájlokat, amelyek modellje ezzel ellentmond, a rendszer kihagyja. Csak fájlok. |
| `read_dls(mac, filter=None, camera_serial=None)` | Egy DLS-ből (Bluetooth lefelé irányuló fényérzékelő) származó érték: spektrum, sávonkénti besugárzás és sávmagnitúdók a kamera saját válaszán keresztül, ha az `camera_serial` meg van adva. Az `discover_daq_sensors(transports=["ble"])` megkeresi a MAC-címet. |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | A gazdagép kamerákhoz szükséges hardvere és az általa ajánlott konfiguráció (`report_text`), valamint a rendszer CPU-ja, GPU-ja és PyTorch futtatókörnyezete. |

- **Egyeseknek kizárólagos hozzáférésre van szükségük a kamerához.** `camera_white_balance_once()`, `apply_camera_preset()`, `sleep_camera()` / `wake_camera()`, `calibrate_camera_with_target()` és `update_camera_firmware()` megnyitják a kamerát. Amíg a Chloros birtokolja (az alkalmazásban csatlakoztatva, az `connect_camera()`-szel, mint tömb tagja, vagy éppen egy másik parancs használja, például egy tömbteszt vagy a kamerák megnyitását végző rögzítés során), a háttérrendszer HTTP 409 `CAMERA_HELD` hibakóddal utasítja vissza a kérést (mind az `code`, mind az `error_code` esetében)kóddal, megjelölve, hogyan lehet felszabadítani. A színbeállítások, a kiemelések, az `camera_power_probe()` és az olvasási műveletek egy csatlakoztatott kamerán működnek.
- **A szín, a kiemelések és a fehéregyensúly az RGB-kamerákra vonatkoznak.** Egycsatornás monokróm (M3M) kamerán a beállító függvények nem változtatnak semmit, és `{"skipped": True, "reason": ...}` értéket adnak vissza, ahogyan a CLI is. Ezek kijelzőbeállítások: az élőképre és a kijelzőszintű felvételekre vonatkoznak, soha nem a sugárzási vagy visszaverődési értékekre.

---

## Tranziens tömbök, igazítási profilok és indexek

Mit csinál az `chloros-cli lattice` állandó kapcsolat nélkül, mint az `chloros_sdk` (`chloros_sdk.array_utils` modul) függvényei: `continuous`, `multi-capture`, `multi-stream`, `multi-test`, `multi-detect`, `analyze-array`, `align-calibrate`, `align-apply`, `align-stream`, `align-info`, `align-reorder` és `index`. Mindegyik elküldi a CLI-parancs által küldött kérést, a CLI alapértelmezett beállításaival.

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

- **Ezek maguk nyitják meg a kamerákat.** Minden parancs megnyitja a saját kameráit, elvégzi a feladatát, majd bezárja őket, ezért olyan kamerákra van szüksége, amelyeket a Chloros nem tart fenn (az alkalmazásban csatlakoztatva, az `connect_camera()`-szel, vagy tömb tagjaként); a háttérrendszer HTTP 409-es hibakóddal (`CAMERA_HELD`) utasítja vissza a kérést, és megadja, hogyan kell felszabadítani egy kamerát. Az átmeneti megnyitás egy ideig tart (néhány kamera esetén körülbelül 40 másodperc). Ismételt rögzítések esetén az `connect_array()` egyszeri végrehajtása a leggyorsabb megoldás, és minden exporttípusnál megőrzi a képet, míg az átmeneti rögzítés csak a debayered képet menti el. Az `calibrate_alignment_profile()` kivételt képez: az ilyen kamerákból álló, csatlakoztatott tömb esetén a tömb saját képkockáiból oldja meg a feladatot (`used_live_array`).
- **`serials`** azok a kamerák, amelyeket megnyitni kell, az első indítja el a rögzítést; az `None` a hálózaton lévő összes kamerát megnyitja.
- **`settings`** egy szótár a kamera-beállítások felülírásairól, azokat a mezőket, amelyeket a CLI jelzői állítanak be: `exposure_time` (µs), `gain` (dB), `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `frame_rate`, `gev_packet_size` és `preset` (egy név szerinti kamera-előbeállítás). Egy megadott expozíció vagy erősítés kikapcsolja az automatikus vezérlést, ahogyan a jelzők is.
- **`gpu`**: az `True` a GPU-torzítást kényszeríti, az `False` pedig a CPU-t (`--gpu` / `--no-gpu`); az `None` a háttérrendszerre bízza a választást.
- **`keep_settings`**/**`clear_settings`** (minden olyan függvény, amely kamerákat nyit meg, alapértelmezés szerint `False`): a végén minden kamera visszaáll a megtalált állapotába; az `keep_settings=True` helyette a futtatás beállításait a kamera saját memóriájába menti (szabad futás), az `clear_settings=True` visszaállítja a memóriát a gyári beállításokra — lásd [Kamera memória](#camera-memory--keep_settings--clear_settings). A válasz (egy feladat esetében az `stop()`-é) az `camera_memory`-et tartalmazza.
- **Az elérési utak a háttérprograméi**: a relatív útvonalakat abszolútvá alakítja a munkakönyvtárhoz viszonyítva, ha a háttérprogram ezen a gépen fut. Az igazítási profilfájl-funkciók (`alignment_profile_info()`, `reorder_alignment_profile()`) a fájlt ezen a számítógépen olvassák be, ahogyan a CLI is.

| Funkció | Mit csinál |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | Egy kamerát folyamatosan rögzít, amíg le nem állítják. Visszaad egy `CliJob` értéket. Egy `tiff` futtatás során rögzített képkockák az `raw` fájlban találhatók, amelyet egy tömb-rögzítés ír ki az adott kameráról (egy mozaiksík az érzékelősaját értékeiből, tömörítetlenül); az `png` és az `jpg` 8 bitet tartalmaz, így ezek a futtatások megtekintésre szánt, demosaicelt képkockák. Egy kötött vagy összevont DAQ a futtatás`daq/` mappájába (a statisztikákban `daq_files` néven). Minden képkockát egyszer ír le a teljes metaadattal együtt (modell, rögzítési idő, objektív, sávok, nyers képkockán végzett kalibrálás); az `stop()` akkor ad vissza értéket, amikor az összes rögzített képkocka fel lett írva (legfeljebb 25 másodperc alatt), és csak ezután zárul le a kamera. A hardveres triggerre beállított kamera (`TriggerMode` `On`) a futtatás idejére szabad üzemmódra van állítva (a kezdő válaszban szereplő `trigger` jelzi, hogy mit talált a rendszer). A leálláskor a kamera visszaáll a talált állapotába — a kioldási beállításokkal és a futás során megváltozott összes beállítással együtt —, kivéve, ha az `keep_settings=True` parancsot használják, amely a futás beállításait a kamera saját memóriájába menti (szabad futás), vagy az `clear_settings=True`, amely visszaállítja azt a memóriát. Az a futtatás, amely nem rögzít képkockát, 10 másodperc után magától befejeződik az `stop_reason` és az `no_frames` parancsokkal, az okot pedig az `last_error` kódban szerepel, az `stop()` pedig ezt az okot adja vissza `error` kódban. Azok a képkockák, amelyek azért estek ki, mert az írók nem tudták tartani a lépést, számba vételre kerülnek, az `drop_reason` kódban megadva az okot. A Chloros által tartott kamerákhoz használja az `CameraSession.capture_repeated()` parancsot. `lattice continuous`. |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | Szinkronizált képkockacsoportok továbbítása az `fps`, az `count` csoportokig vagy a leállításig. Visszaad egy `CliJob` értéket ad vissza (amelynek `start_reply` értéke tartalmazza az `n_cams` értéket és a vezetéki adatot `pacing`); a végső statisztikái tartalmazzák a megragadott, írt és eldobott csoportokat, azokat a fájlokat, amelyeket az író nem tudott írni, valamint azokat a csoportokat, amelyekhez a kamera nem szállított semmit. `lattice multi-stream`. |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | Egy szinkronizált rögzítés, egy debayerezett kép kameránként: `results` (`filepath`, `serial`, `frame_id`) és `pacing`. Egy kamera hiányát jelző hibaüzenet jelenik meg, amelyben először megadja, hány fájl került mentésre. `lattice multi-capture` csatlakoztatott sorozat nélkül. |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | `count` szinkronizált felvételek rögzített expozícióval: `sync_groups`, az `master`, egy `group_timestamps` sor triggerenként (az a kamera, amelyik kihagyott egyet, hiányzik a sorából) és `sync`: minden második kamera eltérése a mesterhez képest, kioldásonként (`offsets_us`: kameránként `mean_us`, `min_us`, `max_us`, `jitter_us`, `groups`), EGY órajel alatt (`clock_domain`: `ptp`, vagy `device` — PTP ki, minden kamera saját üzemidő-órája, így az eltolódás a rendszerindításkor keletkezik, és csak a jitter méri a szinkronizálást). Azokat a triggereket, amelyek időbélyegei keverik az eszköz óráját és a PTP-t (PTP ki vagy még záródik), kihagyjuk és számba vesszük (`groups_mixed_domain`), ahogyan azokat is, ahol a PTP-óra még mindig több mint `agree_within_us`-re van a mestertől (`groups_clocks_disagree`); ezt az `notes` jelzi. Az `clock_offset_mean_us` / `clock_offset_jitter_us` a legnagyobb kameránkénti átlagot és jittert jelenti. Az `success` értéke hamis, az `error` értéke pedig akkor, ha egy trigger kimaradt, ha kevesebb mint két trigger van, amelynél minden kamera ugyanazon az órajelen van, vagy ha az órajelek soha nem egyeznek (`sync.problems`); a sorok mindkét esetben visszaadódnak. Elindítja a PTP grandmastert, ha az nem fut (az `time_sync` az `grandmaster_running` és az `grandmaster_started_by_this_test` értéket adja meg), bekapcsolja minden kamera PTP-jét (ahogyan az `connect_array()` is teszi), és az első trigger előtt megvárja, amíg minden kamera PTP-órája rögzül a grandmasterhez és stabilizálódik: `PtpStatus` Slave egy grandmasteren, attól 100 us-on belül és 3 másodpercig stabilan, és minden kamera így rögzül, szünet nélkül, 120 másodpercig — körülbelül 2 perccel a bekapcsolás után. Egy éppen rögzített óra legfeljebb ~200 us-os eltérést mutat körülbelül 2 percig azután, hogy „Slave” értéket olvasott, és a kamera saját `PtpOffsetFromMaster` értéke ezt nem jelzi; 120 másodperc alatt a kamerák ~10 us-ra kerülnek a stabilizált órához képest. Egy kamera, amely egy korábbi, a PTP-t bekapcsolva hagyó teszt óta még mindig rögzítve van – ahogyan ezt a háttérrendszer látja –, attól a rögzítéstől számol, így egymást közvetlenül követő tesztek esetén nem kell újra várni; az a kamera, amelyet a teszt bekapcsol, vagy amely PTP-vel be van kapcsolva, de amelynek szinkronizálását ez a háttérrendszer még nem látta (máshol kapcsolták be, vagy a háttérrendszer újraindítása előtt), kivárja a 120 másodpercet, és mindkét, éppen szinkronizálódott kamera a saját idejét várja ki. Az a kamera, amely kilép a Slave állapotból, túllépi a 100 us-t, vagy 3 egymást követő leolvasás során egy másik grandmastert követ (1,5 másodperc) a várakozás alatt, újraindítja a számlálását; egy vagy két ilyen leolvasást naplóznak (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`), és a számlálás folytatódik, míg az a grandmaster-azonosító, amely nem olvasható le, ismeretlenként kerül rögzítésre, nem pedig változásként. Ha minden kamera számlálása egyszerre (egy másodpercen belül) újraindul, az a grandmaster időjének eltolódását jelenti, nem a kamerákéét: a háttérrendszernaplója az `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …` értéket jelzi, az `ptp_lock` az `grandmaster_steps`-ben sorolja fel, és az ezt követően lejáró várakozás hibaüzenete az `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`-szel kezdődik. A határérték nem kerül meghosszabbításra. Az `ptp_settle_s` (0–600; `0`, ha nincs leállási várakozás) állítja be a futás leállási idejét; más érték elutasításra kerül (HTTP 400). A várakozás a lezárási idő plusz 90 másodpercre van korlátozva (alapértelmezés szerint 210 másodperc, legalább 90 másodperc), és a hívás `timeout` másodpercig vár a háttérrendszerre: alapértelmezés szerint 240 plusz a lezárási idő (360 másodperc), ahogyan az `chloros-mcp` `test_array_sync`-je is. Az `ptp_lock` rendelkezik az `converged`-szel, `waited_s`, `timeout_s`, `settle_s`, `locked_s` (az egyes kamerák együttes rögzítési és stabilizálási ideje), az `found_settled` (a megtaláláskor már a stabilizálási időre rögzített kamerák), valamint kameránként az `status`, az `offsets_ns`, `slave_s` (mennyi ideje van rögzítve; Nincs: nincs rögzítve) és `remaining_s` (Nincs, ha egyetlen kamera PTP-je sem kapcsol be). A háttérrendszer naplófájljában 5 másodpercenként szerepel egy sor, amely tartalmazza az egyes kamerák állapotát, azt, hogy mennyi ideje vannak rögzítve, és mennyi idejük van még hátra. Minden kamera kikapcsolt PTP-vel indul el; onnan az órák körülbelül 10 másodperc alatt szinkronizálódnak. Ezt követően minden kamerát visszaállítanak arra az állapotra, ahogyan a teszt megtalálta – a PTP-jét, az automatikus expozíciót és az erősítést a teszt a futás ideje alatt „Ki” állásban tartja (`ExposureAuto` / `GainAuto`, valamint egy manuális `ExposureTime` / `Gain`), a többi pedig, kivéve az `keep_settings=True` esetet — vagyis ha egy kamera PTP-vel marad bekapcsolva (`PtpEnable=True`, vagy egy futtatás `keep_settings=True` beállítással) zárva marad, és ha ez a háttérrendszer látta, hogy a stabilizálási idő alatt zárva volt, egy későbbi teszt nem várja meg. Ha az órák nem stabilizálódnak, az `success` bármely trigger előtt hamis, és az `error` megmondja, mit kell tenni: egy olyan kamera, amely soha nem záródott be (`not_slave`), azt jelenti, hogy a grandmaster nem éri el (`time_sync_status()`); az, amelyik folyamatosan kiesett, a záródási ideje alapján kap nevet (`remaining_s`); az órák még mindig konvergálnak (`far`): ellenőrizze az `time_sync_status()` értéket, és ahhoz, hogy a teszt előtt rögzüljenek, kapcsolja be a PTP-t, és hagyja bekapcsolva (`PtpEnable=True` minden kamerán, vagy `keep_settings=True`; a teszt visszaállítja az eredeti állapotot, így bekapcsolva marad), majd futtassa le. `lattice multi-test`. |
| `detect_array_master(line=None)` | A szinkronizáló vonalon lévő kamerák és egy alapértelmezett vezérlő: `master_serial`, `slave_serials`, az `line` által felderítettek. `lattice multi-detect`. |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | Micsoda sorozata ezeknek a kameramodelleknek (terméknevek, `"LATT-M3C-L87-FOCN"`) kerülhetne fel ennek a számítógépnek a hálózatára: `tier`, `achievable_fps_max`, a vezeték-kapacitás (`max_cams_collision_safe`, `oversubscribed`), `cam_caps` és `fix_suggestions`. Alapértelmezés szerint a felismert kamerák modelljei; nincs megnyitva egyetlen kamera sem. A rendelkezésre álló kamerákról, valamint a csatlakozáshoz szükséges beállításokról lásd: [`analyze_array_network()`](#network-analysis-helper). `lattice analyze-array`. |
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | Igazítási profil létrehozása élő képkockákból és elmentése. `method`: `orb`, `akaze`, `phase`, `checkerboard` (az `checkerboard="9x6"`-szel, a belső sarkok) vagy az `manual`-szel (az `control_points={serial: [[x, y], ...]}`-szel, a referenciakamera sarkai is). `model`: `translation`, `rigid`, `affine` vagy `homography`. A finomabb beállítások a CLI parancssori opciói: `max_features`, `ratio_threshold`, `matcher` (`bf` / `flann`), `knn_k`, `prefilter` (`none` / `gradient` / `clahe` / `blur` / `hist_match`), `ransac_threshold_px`, `ransac_max_iters`, `ransac_confidence`, `averaging`, `lock_rotation`, `lock_scale`, `lock_axis`; az `per_cam_overrides` egyenlő az `{serial: {option: value}}`-szel. Az `rms_threshold_px` nem hajlandó elmenteni egy rosszabb megoldást. Visszaadja az `save_path`, `output_width` / `output_height`, `crop_roi`, `reference_serial`, `transforms` (kameránként `reprojection_error`, `num_matches`) és `preview_path`. `lattice align-calibrate`. |
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | Egy igazított többsávos kép a profilon keresztül: `files` (a többsávos TIFF-fájl az `bands`-szel és méretével, plusz egy sávonként az `per_band` fájllal, és egy kameránként az `per_camera` fájllal) és az `vignette_corrected`. Az `bands` kiválasztja, az `order` pedig név szerint (listában vagy vesszővel elválasztott karakterláncban) rendezi a sávokat. `lattice align-apply`. |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | Áramláshoz igazított többsávos képek (ugyanazok a torzítási beállítások, mint az `capture_aligned_multiband()` esetében), az `count`-ig vagy a leállításig. Visszaad egy `CliJob`-et; a végső statisztikái a `frames_written` és bármely `frame_groups_incomplete`. `lattice align-stream`. |
| `alignment_profile_info(path)` | Az `align-info` néven összefoglalt profilfájl kinyomtatja: `name`, `created`, `method`, `reference_serial`, `sensor_resolution`, `output_width` / `output_height`, `crop_roi`, `camera_serials`, `transforms` (kameránként: `model`, `reprojection_error`, `num_matches`, `reference`), `camera_models`, `layer_order` (kameránként: `band_names`, `wavelengths_nm`, `enabled`); az `profile` az egész fájl. |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | A profilfájl helyben történő módosítása: az `order` a sávnevek (mindegyik a saját kamerájának rétegét feljebb mozgatja, ebben a sorrendben), `enable` / `disable` a sorozatszámok. Visszaadja az új `layer_order`-et. `lattice align-reorder`. |
| `index_catalog()` | Az index-előbeállítások (`presets`: mindegyik `name`, `channels`, `formula`) és a színes `gradients`. `lattice index --list-presets` / `--list-gradients`. |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | Egy igazított többsávos TIFF-fájl indexe: egy `preset` (`NDVI`, `NDRE`, ...) vagy egy `formula` a szimbólumok felett, amelyeket az `channel_map` sávokhoz rendel (`{"NIR": "NIR_850", "Red": 0}`, név vagy index alapján). Egy előre beállítottszimbóluma, az `channel_map` kihagyja a fájl azon sávját, amelynek neve ezzel kezdődik (`red` -> `Red_660`, `nir` → `NIR_850_RGN`), amelyet maga a fájl nevez meg (a sávlistája, vagy egy Chloros-felvétel / termék `Chloros:Band{R,G,B}Name`), így egy Chloros-reflektancia-felvétel NDVI-jének nincs szüksége `channel_map`-re; két, egy szimbólumhoz tartozó sávot elutasítanak, nem becsülnek meg. Az `capture_level` meghatározza, mit tartalmaznak a sávok: az `reflectance` vagy az `radiance` kvantitatív értékeket ad; egy nem kalibrált sorozat (az `align-apply`-ek) kvalitatív értékeket ad; az `"unknown"` beolvassa a fájlból, beleértve egy Chloros-fájl `Chloros:ProcessingLevel` elemét is. Visszaadja az `stats` (alak, az 1–99. percentilis tartomány, a LUT-tartomány, a használt `capture_level`) értéket, valamint a bemenet mellé írt `files` értéket (`<input stem>_<preset>.tif`, `..._LUT.png`) mellé vagy az `output` helyre írt értéket, egy elérési utat: képkiterjesztéssel a fájlok a törzsükből kapják a nevüket (az `output="D:/out/ndvi_FRGN.png"` az `ndvi_FRGN.tif`-et és az `ndvi_FRGN_LUT.png`-et írja), kiterjesztés nélkül pedig előtagként szolgál (`D:/out/ndvi` → `ndvi.tif`, `ndvi_LUT.png`); olvassa be az elérési utakat az `files` fájlból. Ezek tartalmazzák a bemeneti fájlhoz tartozó fényképezőgép-adatokat (gyártó / modell / sorozatszám, GPS, objektív, felvétel ideje UTC-ben, a kalibrációs azonosító, de soha nem a nyers-DN kalibrációját), `Chloros:ProcessingLevel=index`, valamint az indexet egyetlen sávként (`XMP-Camera:BandName`, a TIFF és a LUT PNG fájlokhoz hasonlóan). Csak fájlok; kamera nélkül. `lattice index --input`. |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | Felvétel, kalibrálás (`reflectance` vagy `radiance`; az `raw` kihagyja), igazítás és index kiszámítása egy lépésben; az `save_multiband=True` megőrzi a kalibrált sorozatot, amelyet az `compute_index()` később felhasználhat. Visszaadja az `stats`-et (az `serials`-szel együtt, amelynek sávjait használta) és az `files`-et. Megnyitja az `serials`-et (az első indítja el a kioldót), egyébként a profil azon kameráit, amelyek `layer_order` bejegyzése engedélyezve van, a referenciát először (még akkor is megnyílik, ha sávjai letiltva vannak); egy olyan kamerát, amelynek sávjai letiltva vannak, soha nem kalibrálják. Az `channel_map`-re csak olyan szimbólum esetén van szükség, amelyre a verem nem tud válaszolni, mint az `compute_index()` esetében: egy előre beállított szimbólum azt az egy engedélyezett sávot veszi, amelynek neve azzal kezdődik, amit a kamera megnyitása előtt ellenőriz (ha egyik sem vagy több is válaszol, azokat elutasítja, megnevezve őket); egy egyéni `formula`-hez minden szimbólumra szükség van. Az `index.tif` és a LUT hordozza a verem kameráját, a felvétel idejét (a referencia-kamera expozícióját, UTC) és az objektívet, és mindkettő az indexet nevezi meg egyetlen sávjaként (`XMP-Camera:BandName`, pl. `NDVI`: a színes PNG-fájl az indexet nevezi meg, nem a színcsatornáit). `lattice index --live`. |

### `CliJob`

A három adatfolyam által visszaadott azonosító. A feladat a háttérben fut, és a leállításáig a kamerák az ő tulajdonában maradnak.

| Tag | Leírás |
| --- | --- |
| `stats(timeout=10.0)` | `running`, `state`, `stats` (mentett / írva, eldobva, `fps`, `elapsed`, írási hibák, `stop_reason`; egy folyamatos feladat esetén még `frames_grabbed` és `frames_queued` – minden rögzített képkocka írásra kerül, sorba áll, elvetésre kerül vagy meghiúsul – valamint `drop_reason`, `writers` és `write_ms`, az egy képkocka írásához szükséges átlagos idő) és `output_dir`. Az `state` az `running`, az `ended` (önmagát állította le, és még nem állították le), az `stopping` (egy leállás éppen kiírja a sorát: élő statisztikák, valamint az `stopped_by`) vagy az `finished` (a végső statisztikái, a leállítás `error` és `stopped_by`). A leállított feladat adatait 10 percig tároljuk (legfeljebb az utolsó 32-t), minden kliens számára: egy másik szkript vagy az `chloros-mcp` olvassa ki ugyanazt az állapotot és a végső statisztikákat. Az `chloros-cli` feladat elindult (`lattice continuous`, `multi-stream`, `align-stream`) a háttérrendszer leállítja, amikor a CLI-nek 30 másodpercig nem jelentkezik be — leállítva, a gépe alvó állapotban van — az `stop_reason`, `client_gone` és `stopped_by` a háttérrendszerben; a statisztikáinak innen történő beolvasása nem tartja életben. Az SDK-ből indított feladat addig fut, amíg le nem állítják. Minden indításkor elküldi az `timeout`-et a háttérrendszer határidejeként (`client_deadline_s`): egy utána megnyíló kamera (amely még indítás alatt áll) visszaáll és bezárul, és nem indul el feladat, a hívás pedig `ChlorosConnectError`-et (`CLIENT_DEADLINE_PASSED`) generál. Egy feladat által foglalt kamerát minden más nyitott kamera elutasít (`connect_camera()`, egy újabb indítás, egy átmeneti rögzítés) esetén HTTP 409-es hibakóddal (`CAMERA_HELD`), megnevezve a feladatot és azt, aki elindította. |
| `stop(timeout=30.0)` | Leállítja és bezárja a kameráit: `final_stats`, `output_dir` és `error`, ha a leállítás nem sikerült zökkenőmentesen. Egy második hívás ugyanazt a választ adja vissza, ahogyan egy másik kliensről indított leállítás is: az első még a várólistáját írja ki, amikor az első még írja a várólistáját, és megvárja azt (legfeljebb 25 másodpercig; ennél hosszabb várakozás esetén hibaüzenetet ad, és a végső statisztikák az `stats()`-en következnek). |
| `wait(timeout=None, poll_s=0.5)` | Várjunk, amíg magától befejeződik (elérte az `count` értéket, megtelt a meghajtó, hiba történt), majd az `stop()` parancsot adjuk ki a végső számadatok összegyűjtéséhez. Az `timeout` kiadja az `TimeoutError` hibát, miközben a folyamat még fut; egy `count` nélküli feladat csak leállításkor ér véget. |
| `job_id`, `kind`, `output_dir`, `start_reply`, `stopped` | Az azonosítója és típusa (`continuous`, `multi-stream`, `align-stream`), az írási hely, a kiinduló útvonal válasza, valamint az, hogy az `stop()` már futott-e. |

Használd kontextuskezelőként, hogy a hatókör kilépésekor leállítsd.

---

## Projektmappa és receptek

**A projektmappa** az a mappa, amelyben az új projektek létrehozásra kerülnek, és amelyet a Chloros alkalmazás, a CLI és az SDK (`chloros_sdk.system_utils`; `chloros-cli project-folder` modul) közösen használ:

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

Ha egy beállított mappa már nem létezik, az alkalmazáshoz hasonlóan az alapértelmezett (`Documents/MAPIR/Chloros Projects`) mappára vált vissza. Az `set_project_folder()`-nek megadott relatív elérési út a munkakönyvtárból kerül kiolvasásra, ha a háttérrendszer ezen a gépen fut.

**A recept** az a YAML/JSON műveletlista, amelyet az `chloros-cli project run PATH RECIPE` futtat egy mentett projekten. Az `run_project_recipe()` a CLIsaját futtatójával (az SDK-vel együtt telepítve), így egy recept mindkét esetben ugyanúgy viselkedik:

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`** egy `.yaml` / `.yml` / `.json` fájl, vagy maga a recept: egy dict, amely tartalmazza az `actions` (`apply`, `capture`, `burst`, `stream`, `sensor`, `wait`) elemeiből álló szótár, valamint opcionálisan az `overrides` és az `stop_on_error` (alapértelmezés szerint true: a futtatás az első sikertelen műveletnél leáll). A műveletek a CLI-referencia [Recipe DSL](cli-reference.md#recipe-dsl) szerint vannak meghatározva; a relatív `output` a szkript munkakönyvtárához viszonyítva kerül feloldásra.
- **Visszatérési értékek** `ok` (minden művelet sikeres volt), `actions` (hány), `log` (a futtató kimenete; az `echo=True` futás közben is kinyomtatja) és `validation`. Ha egy receptfájl hiányzik vagy nem értelmezhető, az `ChlorosConnectError` hibát eredményez.
- **A futtatás előtt ellenőrizve.**Az `validation` az `{"ok", "errors", "warnings", "steps"}`: minden műveletnek pontosan egy ismert igét kell kulcsszavaként tartalmaznia (`{"capture": {"name": ...}}`, nem pedig `{"action": "capture"}` vagy `{"type": "capture"}`), amely a projektben található, megfelelő típusú kamerát, tömböt vagy érzékelőt nevez meg. A hibás recept**nem** hajt végre semmilyen műveletet (`ok` false, a hibák az `log`-ben), függetlenül attól, hogy mit mond az `stop_on_error`. A fel nem használt kulcsok, valamint minden olyan tömb vagy fényérzékelő, amelyet a recept vezérel, de jelenleg nincs csatlakoztatva, `warnings` hibát eredményez. A recept nem csatlakoztatja az eszközöket: csatlakoztassa a projekt első eszközét (`ChlorosProject.connect_all()`, `chloros-cli project connect`, vagy az MCP eszköz `connect_project_devices`). Az `dry_run=True` semmit sem futtat, és `{"recipe": ..., "validation": ...}` értéket ad vissza.
- Egy folyamatban egyszerre csak egy recept futhat. A YAML-receptekhez PyYAML szükséges.

---

## Projekt automatizálás — `ChlorosProject`

Egy mentett Chloros-projekt egy olyan mappa, amely tartalmazza az `cameras.json` + `sensors.json` + `project.json` fájlokat. Az `open_project(path, relink=None)` betölti a manifesztet (`relink`: az a mappa, amelyben az áthelyezett projekt hiányzó forrásfájljai jelenleg találhatók, és amelyet a háttérrendszer betöltésekor használ — `process()` — lásd **Áthelyezett projekt** az `ChlorosLocal` alatt), az `connect_all` pedig minden mentett kamerát, szenzortömböt és fényérzékelőt a mentett beállításokkal online állapotba hoz — ugyanazt a hardverállapotot, amit a grafikus felület is létrehozna. A fényérzékelők a háttérprogram [DAQ-érzékelő-készletén](#daq-sensor-sessions) keresztül csatlakoznak, ezért futó háttérprogramra van szükségük (lásd [`SensorHandle`](#sensorhandle)).

### Minimális példa

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
proj.capture_all("./out")

proj.disconnect_all()
```

Vagy kontextuskezelőként:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### `ChlorosProject` metódusok

| Metódus | Leírás |
| --- | --- |
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | Minden mentett kamera, tömb és fényérzékelő felderítése és csatlakoztatása. Osztályonkénti csatlakozási jelentést ad vissza (`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`, mindegyikhez tartozik egy-egy `connected`, `missing` és `errors`; az `arrays` szintén tartalmazza az `alignment`-et). Ha van egy, az `127.0.0.1:5000`-en figyelő háttérprogram, akkor a mentett kamerákat és mátrixokat az azon keresztül keresi meg és csatlakoztatja. Ha nincs ilyen, akkor ez a folyamat maga keresi meg és vezérli őket az `lattice_sdk`-en keresztül, amelyhez az Arena SDK futtatókörnyezete és az OpenCV (`cv2`) szükséges – az SDK-kerék egyikét sem telepíti; ha ez nem sikerül, minden mentett kamera és sorozat felkerül az `errors` alá az ok feltüntetésével, az `missing` pedig üres marad. Egy olyan háttérprogram, amely fut, de elutasítja az SDK-t (kijelentkezett: HTTP 401; a Copper alatti terv: 403), nem minősül „nincs háttérprogram”-nak: minden mentett kamera, tömb és fényérzékelő felkerül az `errors` alá a háttérprogram okával együtt, és ebben a folyamatban semmi sem nyílik meg. Minden hívás újra ellenőrzi a háttérrendszert, így az `connect_all()` bejelentkezés után, vagy a Chloros elindítása után ugyanahhoz a projekt-objektumhoz csatlakozik. Egy már a háttérrendszerben futó, ugyanazokkal a kamerákkal rendelkező mentett tömb (a grafikus felületé vagy az `chloros-cli project connect`-é) változatlanul csatlakozik, anélkül, hogy újraalkalmazná a mentett beállításait, és az `disconnect_all()` futtatásban hagyja. A fényérzékelők mindig a háttérrendszeren keresztül működnek (lásd [`SensorHandle`](#sensorhandle)). Soha nem indít el háttérrendszert. |
| `disconnect_all()` | Mindent leállít. Először minden eszközt felszabadít; majd, ha a háttérprogram nem erősítette meg egy kamera, tömb vagy fényérzékelő felszabadítását (a leállítás időtúllépésbe került, vagy elutasításra került), mindegyiket egy-egy `ChlorosConnectError` névvel jelöli meg. A már megszűnt eszköz, vagy ha nincs futó háttérprogram, nem ad visszajelzést. Egy olyan sorozat vagy fényérzékelő, amelyet egy másik kliens nyitott meg, továbbra is fut. A projekt `with` kilépése soha nem lép fel. |
| `capture_all(output_dir=".")` | Egy képkocka minden csatlakoztatott kamerából + sorozatból, és egy spektrum (`SensorHandle.read()`) minden csatlakoztatott fényérzékelőtől, sorban (először kamerák, majd tömbök, végül érzékelők), nem egy pillanat alatt. |
| `stream(camera, overlays=False, fps=10.0)` | Generátor, amely BGR `numpy` képkockákat állít elő egy megnevezett kamerából (vagy sorozatból). Az `overlays=False` a`grab()` az `debayered` szinten: a háttérrendszeren keresztül, ha a projekt ahhoz csatlakozik, egyébként közvetlen `lattice_sdk` rögzítési ciklusban (a tömbök `{serial: frame}` szótárakat eredményeznek). `overlays=True` az `ChlorosLocal.camera_stream()`-en keresztül → a háttérrendszer `/api/camera/<serial>/stream-annotated` MJPEG-adatfolyamához, a kamera mentett `ui.overlay` blokkját lekérdezési paraméterként továbbítva. Ehhez háttérmód, OpenCV (`cv2`) és egy **önálló kamera** szükséges: egy közvetlen módú kamera az `RuntimeError` hibát dobja (a háttérprogram nem tudja megragadni a folyamat tulajdonában lévő kamerát), egy tömb pedig az `NotImplementedError` hibát dobja (kameránként összetett átfedéseket hoz létre — egy elemet név szerint továbbít). Egyszeri műveletnek megfelelő: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Igazítást futtat minden jelenleg csatlakoztatott tömbön. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Futtassa a kalibrációs/indexelési folyamatot a projekt képein (beágyazza az `ChlorosLocal.process`-et; ez a négy az **egyetlen** elfogadott kulcsszó — `indices=` stb. kiváltja az `TypeError` hibát; az indexeket az `ChlorosLocal.configure()` segítségével állítja be). Lusta módon létrehoz egy `ChlorosLocal()`-et, amely automatikusan elindít egy háttérprogramot. |

Attribútumok:
- `proj.cameras` — `Dict[str, CameraHandle]` név és sorozatszám alapján indexelve.
- `proj.arrays` — `Dict[str, ArrayHandle]`, amelynek kulcsa a név ÉS az array_id.
- `proj.sensors` — `Dict[str, SensorHandle]`, amelynek kulcsa a név ÉS a slot_id.
- `proj.config` — `project.json["config"]` szótár.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Feldolgozási szintek.** Az `capture()`, az `grab()` és az `frame_stream()` mind ugyanazt az `processing`
tokent használnak, és a lánc kumulatív – minden szint futtatja a fölötte lévő összes szintet:

| Szint | Kimenet | Megjegyzések |
| --- | --- | --- |
| `raw` | 1-csatornás Bayer, érzékelő-natív | Nincs demosaic. Ezen a szinten nincsenek átfedések. |
| `debayered` | 3-csatornás színes (**alapértelmezett** az `CameraHandle` és minden `grab()` / `frame_stream()`; az `ArrayHandle.capture()`, `capture_smart()` és `stream()` alapértelmezés szerint `raw`): BGR a memóriában (`grab()` / `frame_stream()`), `capture()` által RGB-ként mentve | Backend módban, rögzítésként: a kalibrációs csomag sötételtolása (DSNU) a nyers adatról, majd az OpenCV élekkel számoló demosaic-ja — nincs linearitási LUT, síkmező vagy színszétválasztás. Az egyetlen szint, amely háttérmód nélkül is működik, ott a kamera-könyvtár egyszerű bilineáris demosaic-ja (nincs sötét eltolás). A mentett TIFF fotometrikus RGB, ezért fordítsuk meg a memóriában lévő képcsatornáit (`frame[..., ::-1]`), hogy összehasonlíthassuk egy fájllal. |
| `radiance` | float32, W/m²/sr/nm, **sávonként egy sík** | Teljes radiometrikus lánc: demosaic + 3×3 szétválasztás (multispec) + DSNU + flat-field + NIST-skála, az expozíció és az erősítés kiszűrésével, így az értékek abszolútak. |
| `reflectance` | uint16, 32768 = 1,0 (vagy uint8, 255 = 1,0 — lásd `bit_depth`), **sávonként egy sík** | Sugárzási erősség osztva a lefelé irányuló besugárzási erősséggel (ρ = π·L/E). DLS/DAQ-leolvasás szükséges — lásd az alábbi megjegyzést. |
| `display` | 8 bites, sRGB-hez hasonló | GUI-val egyenértékű megjelenítés: CCM + fehéregyensúly + gamma a kamera aktív színprofilján keresztül. |

Közvetlen (backend nélküli) módban csak az `debayered` áll rendelkezésre: az `grab()` egy másik szinten felülírja
`NotImplementedError`-et, az `capture()` figyelmen kívül hagyja az `processing`-et, az `frame_stream()` pedig minden szinten
`RuntimeError`-et minden szinten. `reflectance`-nek használható lefelé irányuló leolvasásra van szüksége — a keret végpontja automatikusan behúzza
az összevont DAQ-t a kamera DLS-nyílásába, de ha nincs DAQ hozzárendelve, a lánc elutasítja a
reflektancia-kimenetet, és őszintén jelzi a visszaminősítést a visszaküldött metaadatokban, ahelyett, hogy csendben
visszaadna egy alacsonyabb minőségű terméket.

> **Reflektancia DN skála — ne kódold be mereven.** A LATTICE reflektancia az `32768` = ρ 1,0 értéket használja, és
> XMP `Chloros:PixelScale=32768` jelölést ad; az Survey3 reflektancia az `65535` = ρ 1,0 értéket, és nem tartalmaz
> `Chloros:*` címkét. Olvasd be a címkét, és oszd el vele. Az érték az uint16 tartományban van definiálva, így
> `32768` marad minden olyan formátumban, amely átméretezi (16 bites TIFF, 8 bites TIFF, 8-bites PNG/JPG, 32-bites százalék)
> — először normalizálja vissza a tárolt adattípust uint16-ra (8-bitesből ×257, floatból ×65535). Az egyetlen
> kivétel egy 1.3.0 előtti verzióból származó fájl: akkor egy 8-bites TIFF-et *levágtak*, nem méretezték át, így
> nincs méretarány, amely leírná, és nem tartalmaz `PixelScale` vagy MicaSense tupelt. A hiányzó címkét egy
> LATTICE reflektanciafájlban kezelje „érvényes méretarány hiánya”ként, ne pedig alapértelmezettként.

**Kalibrációs címkék: csak az `raw` fájlban.** Egy felvétel `raw` fájlja tartalmazza a kamera kalibrációs csomagját: az `CalibrationJSON` hasznos adatot és a MicaSense-kompatibilis címkéket (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), amelyek az érzékelő nyers értékeit írják le. Minden más szint fájlja (`debayered`, `display`, `radiance`, `reflectance`, az `index/` render) és minden folyamatos felvételű / többkamerás adatfolyam képkockája hordozza a kamera azonosítóját (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...), valamint a képkocka saját adatait, az `XMP-mapir:FrameJSON`-et (expozíció, erősítés, hőmérséklet, képpontformátum, rögzítési idő, DAQ-egyezés), mivel minden `process()` termék hordozza az azonosítót. A feldolgozás a sorozatszám és az `CalBundleSha256` alapján keresi meg az ilyen fájl kalibrációját a Chloros kalibrációs gyorsítótárában, és ha a számítógépen nincs meg, egyszer letölti a csomagot.

#### Több kamerából készült fájlok

Több kamera sávjaiból készült fájl – többsávos rétegek (`capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index(save_multiband=True)`, `Registration.save_multiband()`), több kamerából készült élő index (`index.tif`, `index_LUT.png`), egy tömbfelvétel `composite/` képe — felsorolja az összes benne szereplő kamerát:

- **Kameránkénti-kameránkénti listák, sávsorrendben** (az összetett kép: háttérréteg, majd előtér; egy indexréteg, azok a kamerák, amelyek sávjaiból származnak): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (az a kalibrációs csomag, amelyen az egyes kamerák sávjai átmentek, egyébként pedig — egy `capture_aligned_multiband()` / `start_aligned_stream()` halmaz, amelynek sávjai egyiken sem mentek át — a kamera azon csomagja, amelyet a Chloros tárolt a gyorsítótárban, amelynek neve megegyezik a felvételi fájlok nevével), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — párhuzamos listák, amelyek *n*-edik eleme mindegyikben ugyanazt a kamerát írja le; `-`, ahol egy kamerának nincs értéke (nincs kalibrációs csomagja, ismeretlen objektívje van). Minden kamera **egy** elem: amennyiben a csomagjának egy mezőjéhez több értéke is van — a csomag halogénlámpa-tanúsítványai — azokat `;` (`F-1479-1;F-1480-1;F-1481-1`) jelöléssel kötik össze, soha nem vesszővel, így a lista kameránként egy elemet tartalmaz, függetlenül attól, hogyan nyomtatják ki.
- **EXIF `SerialNumber`** ugyanazokat a sorozatszámokat tartalmazza, vesszővel elválasztva (`214000535,214000551`).
- **Nem** tartalmaz egyetlen kamera azonosítóját sem (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): ezek egy-egy kamerát és annak csomagját nevezik meg, és a feldolgozás ezekből határozná meg a fájl kalibrációját. A több kamera sávjából kiszámított index (`index.tif`, `index_LUT.png`) EXIF-adatokkal rendelkezik: `Model` `LATTICE` (a rendszer), nem pedig egyetlen kamera `LATT-…` modelljét.

Egy **kamera** sávjaiból álló fájl – sávonkénti TIFF, egy `capture_aligned_multiband(per_camera=True)` képkocka, egy kamera stackje vagy élő indexe – az adott kamera saját azonosító címkéit hordozza, ahogyan minden más rögzítési fájl is. Az `capture_aligned_multiband()` halmaza, sávonkénti és kameránkénti fájljai, valamint minden `start_aligned_stream()` halmaz és annak sávonkénti fájljai a csoportjuk rögzítési idejét (a referenciakamera expozíciója, UTC), és egy monokamera `per_camera` képkockája egy sík, az `bit_depth`-hez hasonlóan, mint a képcsomag.**A kamera tájolása (tükörképezés / forgatás).** Az `grab()` / `frame_stream()` fájlokból származó képkockák minden szinten mindig a
kamera natív tájolásában vannak. A mentett felvételek követik a kamera tájolási
beállításait (a grafikus felhasználói felület Mirror H / Mirror V / forgatás beállításait):

- **Az `raw` és `debayered` fájlok pontosan azt tartalmazzák, amit az érzékelő leolvasott**, függetlenül a beállítástól — egy elfordított vagy tükrözött Bayer-
  mozaik eltérő színszűrő-mintázatot eredményezne, és a kalibrációs térképek
  az érzékelő saját rácsán alapulnak.
- **Az `display`, `radiance`, `reflectance` és index fájlok orientáltak** — egyszer elforgatva, az összes
  kalibrációs számítások után. A clamp maszkjaik ugyanazon a rácson vannak.
- **Minden rögzített fájl rögzíti a beállítást** az XMP-ben: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, az óramutató járásával megegyező irányban, a
  a tükrözések után), valamint `ChlorosOrientationApplied` — `false` a nyers és a debayered képeken, `true` az orientált
  termékeken. Az `ChlorosProcessing` felsorolja az adott fájl képpontjaira alkalmazott műveleteket (`none`, ha
  natív). Ez magában foglalja egy folyamatos felvétel minden képkockáját (a kamera saját beállítása,
  alkalmazás nélkül) és egy többkamerás adatfolyamét (az identitás: tagképkockái natívak, mint egy
  tömbfelvételnél).
- **A projekt feldolgozása a rögzített orientációt** alkalmazza azokra a termékekre, amelyeket nyers
  (először az érzékelő-koordináták szerinti kalibrálás, majd a mátrix-igazítási torzítás, végül a tájolás),
  így azok megegyeznek az élő rögzítésből származó termékekkel — beleértve a nem kalibrált érzékelő-válaszú / vignettakorrekcióval ellátott
  tartaléktermékeket is; a debayerezett termék natív marad.
- **A Chloros képnézegető a felvétel alapján elforgatott nyers vagy debayered fájlt** jelenít meg — ez csak megjelenítési célú,
  a fájl nem változik; a felvétel nélküli fájl a tárolt állapotában jelenik meg.
- **A felvétel nélküli rögzítéseket natívként dolgozza fel a rendszer.** Az 1.3.0-ás verzió előtt az egykamerás felvételek
  maguk fordították meg a nyers és a debayered fájljaikat; a Chloros nem tudja megkülönböztetni az ilyen nyers fájlt a natívtól, és
  nem is próbálja kitalálni.
- **A tömbös felvételek alapértelmezés szerint minden tagfájlt natív formátumban tartanak meg**, és rögzítik az azonosítót;
  az `ArraySession.capture(orient=True)` feldolgozza a termékeiket (lásd a tömbfelvétel vezérlőelemeit).

#### Sávonkénti kimenet és vegyes tömbök

Az `radiance` és `reflectance` szinteken egy kamera **sávonként egy síkot ad vissza** — hármat egy M3C
esetén (három keveretlen sávja, BGR csatornasorrend), egyet pedig az M3M esetében (az
egyetlen keskenysávú szűrője). A monokamerák replikált síkjait összevonják, így egy **kevert
M3C/M3M-mátrix sorozatonként eltérő mátrixformákat eredményez**. Olvassa ki az `bands` kulcsot a képkocka
metadatából, hogy megtudja, mi az egyes síkok; ne a csatornák számából következtessen rá:

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

Az értékek használata előtt mindig ellenőrizze az `hdr["processing_level"] == "reflectance"` értéket. A visszaminősítés
(nincs DAQ-leolvasás, nincs kalibrációs csomag, ismeretlen sávelrendezés) az `processing_level` és
az `radiometric_refused` kódokban kerül jelentésre, nem pedig a „raised” (felminősítés) kódban — a visszakapott antennarendszer továbbra is érvényes keret, csak
nem az a termék, amit kért. Egy visszaminősített, elmentett rögzítés ugyanazt a kódot hordozza az XMP-jében,
mint az `Chloros:RadiometricRefused` (pl. `no-band-layout`, `no-dls-snapshot`).

#### Egy trigger olvasásonként — `group_id`

**A tömb tagjait a tömb szinkronizált képkockacsoportjából szolgáltatják, nem pedig az egyes kamerák saját
rögzítőpufferéből.** A hardveresen szinkronizált tömb kamerái együttesen adnak ki képet, de a kameránkénti N olvasás
nem *mintafelvétel* együttesen — az egyes kamerák legfrissebb pufferelt képkockájának kiolvasása átnyúlik a trigger
határait átíveli, és egyetlen elhagyott képkocka is szinkronhiányt okoz a készletben anélkül, hogy bárki is észrevenné.

A csoportból történő kiszolgálás szükséges, de önmagában nem elegendő: minden kérés továbbra is
a saját pillanatában *a legfrissebb* csoport pillanatképét készíti el, és a rögzítési ciklus közben új képkockákat is közzétehet.
Egy élő, 4 kamerás tömbön végzett mérés során **a 20 párhuzamos olvasás közül 12 két csoportra terjedt ki** — mindig
pontosan egymás mellett. Így az olvasás **rögzített**:

1. Minden képkocka hordozza az `group_id`-et, a rendszer monoton közzétételi számlálóját.
2. Az `grab()` beolvassa a mestert (vagy, ha a mester nem érhető el, a következő elérhető kamerát),
   majd a többi kamerát ahhoz a csoportazonosítóhoz rögzíti, amelyiken landolt (`?group_id=N` az HTTP
   végponton).
3. A háttérrendszer **legalább fél másodpercig** megőrzi a legutóbbi csoportokat, így egy olyan testvér, amelyet már
   későbbi közzétételek megelőztek, továbbra is kiszolgálja azt a triggert — ez elég hosszú idő ahhoz, hogy a kameránkénti
   reflektancia-export oda-vissza útjának lefedésére elegendő időt biztosítson, még egy 9 kamerás, teljes szenzoros rendszer esetén is. Egy olyan hozzárendelést, amelyet a háttérrendszer már nem
   tart fenn, az `409` és `group_aged_out: true` kóddal utasít el, ahelyett, hogy csendben válaszolna
   egy másik triggerrel – az `grab()` újraolvassa a készletet.

Újramérés rögzítéssel: **20 olvasatból 20 egy trigger-t adott vissza.** Az `grab()` ezt elvégzi helyetted;
csak akkor van szükséged az `group_id=`-re, ha közvetlenül az HTTP-végpontot vezéreled.

- Az a kamera, amelyet a rögzített csoportban nem lehet kiszolgálni — hiányzik belőle, vagy elutasítja a kérést
  (nincs meg benne egy `band`, vagy nem tud létrehozni egy szintet) — **kihagyásra kerül, nem pedig pótlásra**
  egy másik kiváltóból, a többi kamera pedig továbbra is visszaadódik.
- Az `grab()` által visszaadott szótár egy `GrabResult` (egy egyszerű `dict` alosztály;
  `from chloros_sdk import GrabResult`): az `.errors` minden
  kihagyott sorozatszámot annak a kivételnek rendel hozzá, amelyet az olvasás váltott ki, az `.group_id` pedig a megosztott csoport. Egy üres
  `.errors` azt jelenti, hogy minden kamera ebbe a csoportba tartozik.
- Egy önálló kamera nem tartozik csoporthoz, `group_id: None` értéket jelent, és **elutasítja** a `group_id=`
  pin-t, ahelyett, hogy figyelmen kívül hagyná.

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

Ugyanazt a számlálót teszik közzé a tömb tagjainak **minden kameránkénti MJPEG-adatfolyamában** — az
előnézetben (`/api/camera/stream/<serial>`), a megjegyzésekkel ellátott előnézetben (`/stream-annotated`) és a
reflektancia-adatfolyam (`/stream-reflectance`) — `X-Chloros-Group-Id` néven minden részben, az
`X-Chloros-Frame-Id` és az `X-Chloros-Timestamp-Ns` mellett. Az adatfolyamok folyamatosak, nem pedig
időponthoz kötöttek, ezért az összeillesztésük a részek összegyűjtésével és az adott fejléc alapján történő csoportosítással történik — nincs
beállítandó pin. Egy önálló kamerának nincs csoportja. Lásd
[Kameránkénti MJPEG-adatfolyamok](#per-camera-mjpeg-streams-no-rtsp).

**Az `frame_stream()` soha nem hallgat el.** Azokat a jelzéseket, amelyek olyan okból buknak meg, amely magától megoldódhat,
a rendszer kihagyja — egy kamera-adatfolyam nem ad vissza semmit rájuk, egy tömb-adatfolyam pedig azokat a kamerákat adja vissza, amelyek
válaszoltak (esetleg egyet sem). Ezek az okok a következők: még nincs képkocka; a kamera letöltő szála egy-két
másodpercre szünetelt (`404 not streaming`, amire a háttérrendszer akkor válaszol, amikor egy másik kamera csatlakozik
vagy lekapcsolódik, vagy egy beállítás-módosítás újraegyensúlyozza a poolt); a háttérprogram újraindulása vagy `5xx`; az igazítás
még folyamatban van (`409 … retry shortly`); valamint visszaverődési elutasítás, mert a DAQ-nak még nincs leolvasott értéke
(`no-dls-snapshot`, elavult vagy üres leolvasás). Egy elutasítás a következő ciklusban csak megismétlődne — egy
ismeretlen `band`, `bit_depth=8` sugárzási értékre, egy sugárzási skála nélküli kalibrációs csomag, egy reflektancia
elutasítás **egyáltalán nincs DAQ** (nincs a kamerához vagy annak sorozatához rendelve, nincs csatlakoztatva a
háttérrendszerhez: az elutasítás `dls_source`-je `"none"`), bármely más `4xx` —
`chloros_sdk.FrameFetchError`-et eredményez (egy `RuntimeError`, amely magában hordozza az `.status`-et és az `.body`-et)
azonnal. Ugyanezt eredményezi **60 másodperc** kihagyott tick is, amely között nincs képkocka, bármi is legyen az `fps` — elég
hosszú ahhoz, hogy átvészelje az igazítás lezáródását és a háttérrendszer újraindulását, de elég rövid ahhoz, hogy egy
ténylegesen kiesett, leállítsa az adatfolyamot. Ez a feladási hiba szintén egy `FrameFetchError`, akár egy kameráról, akár
egy tömbről van szó, az utolsó olvasás saját kivételével az `.errors` (és kamerák esetében `__cause__`).
Tömb esetében a „refused” azt jelenti, hogy minden kamera elutasította; az `.errors` kivétel tartalmazza az egyes kamerák
okát.

**Egy tömb `frame_stream()` minden egyes trigger-t egyszer ad ki** (`ArrayHandle` és `ArraySession` egyaránt). Egy
tick, amelynek legújabb halmaza az a trigger, amelyet már kiadott, a következő triggerre vár, ahelyett, hogy
ismét kiadná ugyanazt a halmazt, így a stream az `fps` és a tömb saját képkockasebessége közül a lassabbik szerint fut, és az egymást követő halmazok soha nem osztoznak egy `group_id`-en. A közzétételt leállító tömb
`FrameFetchError` („60 másodpercig nincs új trigger”) értéket, ahelyett, hogy az utolsó halmazát ismételné. Enélkül egy
olyan adatfolyam, amely többet kért, mint amennyit a tömb közzétett – vagy közvetlenül a csatlakozás után indult –,
ismétléseket adott ki: egy mért futtatás során 20 sorozat 11 különböző csoportazonosítót tartalmazott.

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### Egyetlen sáv exportálása — `band`

`grab()` / `frame_stream()` használatával az `band="<name>"` parancs a kamera sávjai közül csak egyet exportál
egycsatornás `(H, W, 1)` tömbként — ugyanolyan alakú, mint amit egy monokróm kamera állít elő, így az egycsatornás kimenetet
azonos módon kezeli a rendszer, függetlenül attól, hogy M3M-ből származik-e, vagy egy M3C felosztásából. Egy FRGN vörös sávja:

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

Az elfogadott írásmódok a teljes elrendezésnév (`Red_660`, `NIR_850`, `Green_550`) vagy annak rövid
előtagja (`red`, `nir`, `green`), a nagy- és kisbetűk megkülönböztetése nélkül. Az ismeretlen nevet elutasítja, és felsorolja a rendelkezésre álló
neveket.

> **A kiválasztás név szerint történik, nem csatornaszám szerint — szándékosan.** Az „1. csatorna” az `Green_550` a
> mátrix BGR csatornasorrendjében, de *piros* a szűrőkódbetűsorrendjében (`RGN`), és a kettő
> nem egyezik. A rossz csatorna kiválasztása a rossz sávból származó, teljesen hihető képkockát eredményez, amelyet
> a rendszer további részei nem tudnak felismerni. Emiatt nincs numerikus index.

#### A streamek összehangolása — `align`

Az `align=True` a képkockát a tömb közös referenciaképkockájához igazítja. Minden kamerát
**ugyanazon** profilfelbontás és kivágási ROI szerint igazítanak, így a kameránként függetlenül megnyitott adatfolyamok**egymáshoz igazítva** jelennek meg — nem kell őket együtt kérni:

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

Követelmények és viselkedés:

- A kamerának egy **hardveresen szinkronizált** tömbben (M8 szinkronkábel) kell lennie, amely rendelkezik igazítási
  profiljal — a közös regisztráció feltételezi, hogy az elemek egyidejűleg vannak kitéve, amit egy szoftveres vagy
  szabadon futó rendszer nem biztosítja. Ellenkező esetben az `409` kóddal elutasításra kerül.
- **Elutasítva, soha nem kerül csendben igazítás nélkül mentésre.** A rögzítési útvonal figyelmeztetéssel mentheti az igazítás nélküli felvételt,
  mivel a fájl megőrzi a származási címkéit, és később újra regisztrálható; egy élő streamnek nincs
  ilyen második esélye, ezért a hiányzó profil hiba, nem pedig visszaesési lehetőség.
- **Az igazítás kérése elindítja azt.** Ha profil nélküli tömbön kérjük az `align`-et, az elindítja a
  kalibrálást, és visszaadja az `409`-et azzal, amire vár (pl. *„várja, hogy az expozíció
  214000551-re (1/3) álljon be”*) — próbálja meg újra, és általában néhány tíz másodpercen belül sikerrel jár.
  Ez headless módban is fontos: a csatlakozáskori automatikus igazítás 30 másodperc után feladja, és minden kísérletet
  ezen időtartamon belül kihagyásra kerül, amíg a smart-AE még stabilizálódik, így egy olyan tömb, amelynek expozíciója
  lassan konvergál, korábban az egész munkamenet során profil nélkül maradt, és nem volt mód arra, hogy
  azt a desktop kombinált nézet megnyitása nélkül megszerezzük. Az `alignment_status` szintén jelentést küld
  `ae_align_holdouts`-et, megnevezve a még stabilizálódó kamerákat és azok előrehaladását.
- Adj minden sorozat minden egyes streamjének **ugyanazt az `max_h`-et**, különben a kivágásaik különböző pixelrácsokra
  kerülnek.
- Az igazítás *a* bitmélység-kódolás *előtt* történik, így a torzítás teljes pontosságú
  értékeket interpolál.

#### `bit_depth` — a pontosság feláldozása a sávszélesség érdekében

Az `grab()` és az `frame_stream()` elfogadja az `bit_depth=16` (alapértelmezett) vagy az `bit_depth=8` értéket:

| `bit_depth` | Reflektancia-kódolás | Költség |
| --- | --- | --- |
| `16` | uint16, 32768 = ρ 1,0, tartalék ρ 2,0-ig | Teljes pontosság. |
| `8` | uint8, amely átfedi az `0..refl_max` (alapértelmezett ρ 1,0) | Fele annyi bájt; ~0,004 ρ kvantálás az alapértelmezett felső határnál. A felső határ feletti értékek levágódnak — lásd alább. |

##### A 8 bites felső határ — `refl_max`

`refl_max` határozza meg a ρ 8 bites tartományát, így az `pixel_scale = 255 / refl_max` és a ρ értéke továbbra is
`pixel / pixel_scale`. Az `grab()` és az `frame_stream()` mind a kamera, mind a
tömbkezelők esetében is (`?refl_max=` az HTTP-végponton). Növelje meg, hogy több mozgásteret biztosítson a tükröződő
fénypontok számára, arányosan nagyobb lépésekben:

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | lépés |
| --- | --- | --- |
| `1.0` (alapértelmezett) | 255 | 0,0039 ρ |
| `1.5` | 170 | 0,0059 ρ |
| `2.0` | 127,5 | 0,0078 ρ |

> **Az automatikus expozíció nem tudja kijavítani a 8 bites túlexponálást, és ha megpróbáljuk használni, az csak rontja az adatokat.**
> ρ = π·L/E, ahol az expozíciót és az erősítést kiosztottuk, így a ρ a jelenet tulajdonsága, nem pedig a
> kamera beállításáé — élő szenzoron mért értékek alapján az expozíció felére csökkentése a ρ átlagértékét 1,3%-kal változtatta meg. Ha viszont
> expozíció növelése a tartományhoz való „illesztés” érdekében viszont a pixeleket a **érzékelő** telítettségébe hajtja, ami
> alulbecsüli az L értéket, és **alacsony** értékre torzítja a ρ-t hibajelzés nélkül (mérés: max. ρ 1,203 → 0,985
> kétszeres expozíció mellett, átlag −23%). Válassza ki a tartományt az `refl_max` segítségével; az érzékelő
> telítettségének elkerülése az AE feladata, és ez inkább a ρ *érvényességét* védi, mint a tartományát.

**Olvassa el az `reflectance_clipped_fraction`-et, ne a jelzőt.** Az `reflectance_clipped_at_1_0` egyetlen
pixel miatt is kiold – egy valós jelenetnél az `True` értéket jelentette, miközben az arány `0.000001` volt, azaz egy pixel
egymillióból. A törtszám az, ami megmondja, hogy emelje-e a felső határt.

**Az `bit_depth=8` kizárólag a visszaverődésre vonatkozik.** Ha az `radiance` értékre kérik, a kérés elutasításra kerül, nem pedig
teljesítésre: a reflexió tartománya rögzített (0–1), ami 8 bites kódolás mellett is megmarad, míg a sugárzásnak nincs
rögzített tartománya, így 8 bitbe való illesztéséhez képkockánkénti percentilis kiterjesztésre van szükség, amelynek leképezése
minden képkockánál változik — ez már kép, nem pedig mérés.

#### Élő reflektancia-adatfolyam (MJPEG)

Az `GET /api/camera/<serial>/stream-reflectance` sávonként 8 bites reflektanciát szolgáltat
`multipart/x-mixed-replace` formátumban — M3C-ként 3-csatornás JPEG-ként, M3M-ként szürkeárnyalatosként. Lekérdezési paraméterek:
`fps` (alapértelmezett: 10), `jpeg_q` (alapértelmezett: 95), `max_h` (0 = natív), `band` (egy megnevezett sáv, a
fentihez hasonlóan), `align` (`1` a társregiszterhez — ugyanazok a szabályok, mint az SDK `align` esetében), `refl_max` (a 8 bites
kódolás felső határa, mint fent). Minden rész tartalmazza az
`X-Chloros-Group-Id`, `X-Chloros-Frame-Id`, `X-Chloros-Timestamp-Ns`, `X-Chloros-Bands`,
`X-Chloros-Pixel-Scale`, `X-Chloros-Aligned` és `X-Chloros-Reflectance-Clipped` azonosítót.
**A különböző kamerákból származó, azonos `X-Chloros-Group-Id` azonosítót hordozó részek ugyanazt a kiváltó eseményt jelentik** —
így állítják össze újra egy tömb több, kameránkénti adatfolyamát, és ez szerveroldalon garantált,
mivel a tömb elemeit a szinkronizált képkockacsoportból szolgáltatják (lásd
[Egy trigger olvasásonként](#one-trigger-per-read--group_id)). `409` értéket ad vissza, ha a csatlakozáskor a reflexió
nem áll rendelkezésre, és leállítja az adatfolyamot, ha az futás közben válik elérhetetlenné, így egy leállt
adatfolyam azt jelenti, hogy „nincs többé reflexió”, nem pedig azt, hogy „még rendben van, csak statikus”.

> **A JPEG veszteséges – ezek a pixelek mérési értékek.** A tömörítési hiba közvetlenül a
> reflektanciaértékekben jelentkezik, és a széleken a legrosszabb, ahol az anyaghatárok vannak. A minőség
> ezért alapértelmezés szerint 95. Ha a számok sértetlenek maradjanak, használja a
> az `grab(processing="reflectance")` / `frame_stream(...)` parancsokat, amelyek ugyanazt az eredményt adják vissza, anélkül, hogy
> a kódoló lenne az útvonalban (és az `bit_depth=8`-et, ha a vezeték költsége, nem pedig a kódoló jelenti a korlátot).

> **Az EXIF-adatok átkerülnek az exportba.** Az `process()` a forrásfelvétel GPS-blokkját
> **és az ExifIFD-jét** másolja át minden termékre, így az exportált fájlok az `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` és `CameraSerialNumber` fájlokat, valamint a
> földrajzi hivatkozást is tartalmaz. Az `FocalLength` az a fájl, amelyből a Pix4D kiszámítja a talajminta-távolságot – ennek hiányában
> a rekonstrukció rendkívül hibás méretarányra áll át (egy mért esetben egy 411 m-es terület
> 47,8 km-es lett). A másolat szándékosan nem az `-all:all` fájl: az IFD0 szerkezeti címkéi megszakítják
> a LATTICE kimenetét, az `ExifImageWidth`/`Height`-et pedig azért zárjuk ki, mert azok a forrás
> felvételét írják le, nem pedig az exportált rasztert.

Rögzítési szakasz aljelzői (a radiometrikus szintekre vonatkoznak — `radiance`, `reflectance`, `display`):

| Jelző | Alapértelmezett | Jelentés |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + síkkép + 3x3 keverésmentesítés + NIST radiometrikus skála. |
| `apply_white_balance` | `True` | WB LUT. DLS-támogatás, ha egy DAQ van a kamerához kapcsolva. |
| `apply_index` | `False` | Vegetációs index értékelése. |
| `index_expression` | `None` | Képlet felülírása. Ha-üres → automatikusan engedélyezi az indexet. |
| `annotated` | `False` | GUI-díszítések (zebra/rács/csúcsjelzés) átfedése. Nem elérhető az `raw` esetében. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **A visszatérési érték egy `chloros_sdk.CapturePaths`, típusú `CapturePathMap`, nem pedig `Dict[str, str]`.**
> Az `chloros_sdk.CapturePathMap` egy `Dict[str, Union[str, List[str]]]`: egy szintű
> az `processing` minden sorozatszámhoz egy útvonalat ad, míg az `processing="all"` az adott kamerához mentett összes termék
> **sorrendbe rendezett listáját** adja meg. Az `ArrayHandle.capture()`
> nem fogadja az `levels=`-et, és figyelmen kívül hagyja az ismeretlen kulcsszavas argumentumokat (`levels`, `daq`,
> `render_index`, `force_daq`), ezért csak az itt feltüntetetteket add meg (`aligned=False`
> natív fájlokat ment, mint az `ArraySession.capture()` esetében); ezekhez a vezérlőkhöz használja az
> `ArraySession.capture()` kulcsot. Egy élő kombinált kompozit – ha éppen streamelt – a sorozatszám helyett az extra
> `"combined"` kulcs alatt érkezik, nem pedig egy sorozatszám alatt. Az a kód, amely az `str`-et feltételezi, a
> listaformátumban megszakad anélkül, hogy bármely típusellenőrző kifogást emelne — a megjegyzés szerint az `Dict[str, str]`
> a listaforma megjelenése után egy ideig, ezért létezik az alias. Normalizálj
> , ha a lapos formátumot szeretnéd:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **Amit nem lehetett leírni: `.failed`.** Az `capture()` által visszaadott leképezés egy
> `chloros_sdk.CapturePaths` — egy egyszerű `dict` alosztály, így a fentiak mind
> változatlanul működnek — amely tartalmazza az `.failed`-et is: azokat a fájlokat, amelyeket a háttérprogram nem tudott írni,
> ugyanazokat a bejegyzéseket, mint [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)
> (`serial`, `level`, `error` és `disk_full: True` egy teli meghajtón). Egy részleges
> rögzítés még mindig minden olyan elérési utat visszaad, amelyre írás történt; egy teli meghajtón a térkép
> üres, és az `.failed` megmagyarázza, miért. Az `.skipped` felsorolja a kameránkénti kihagyásokat,
> az `.group_seq` pedig a szinkronizált csoportot, az `.alignment` / `.oriented` pedig a
> háttérrendszeréi, mint az `CaptureResult` esetében. Az `capture_smart()` ezeket az alábbiakként adja vissza:
> `result["frames"].failed` és `result["failed"]` néven adja vissza őket.
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
> ```

### Tömb-igazítás

Az `ArrayHandle` a teljes igazítási felületet teszi elérhetővé. A profilok alapértelmezés szerint csak a munkamenetre vonatkoznak — a tartós mentéshez kifejezetten hívja meg az `export_alignment()` parancsot.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Csatlakozáskori igazítás

Az `connect_all(align=...)` minden tömböt automatikusan igazíthat a csatlakozáskor:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Ha nincs megadva, akkor az `project.json["config"]["auto_align_on_connect"]`-re vált vissza.

### `SensorHandle`

Egy mentett fényérzékelő (DAQ-U, DAQ-M vagy DAQ-E), amelyet az `connect_all()` nyit meg a háttérrendszer [DAQ-érzékelő-készletében](#daq-sensor-sessions) megnyitott, elmentett fényérzékelő (DAQ-U, DAQ-M vagy DAQ-E) — ugyanaz a készlet, amelyet az `connect_daq_sensor()` és az `chloros-cli daq pool-connect` is használ.

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- Az `read()` az érzékelő **következő** spektrumára vár — az a spektrum, amely a híváskor már pufferben van, nem számít — és visszaadja az `(spectrum, is_saturated, integration_time, x, y, z)` értéket, vagy az `None` értéket, ha 5 másodpercen belül nem érkezik be semmi. A szüneteltetett adatfolyam folytatódik. Az `spectrum` értéke W/m²/nm-ben van megadva, ha az érzékelő kalibrálva van.
- Az `.sensor` az érzékelő [`DAQSensorSession`](#daq-sensor-sessions) értéke, minden egyéb esetben: `latest(n=...)`, `status()`, `record_start()` / `record_stop()`.
- `.slot_id`, `.model`, `.serial`, `.name` és `.settings` az `sensors.json`-ből mentett bejegyzések.

**Mit állít vissza az `connect_all`.** Minden érzékelőt a mentett címén nyit meg — COM-portján, Bluetooth-címén vagy DAQ-E-gazdagépén —, és ha a projekt rögzítette az érzékelő azonosítóját, akkor annak pontosan annak az érzékelőnek kell lennie: egy másik egységet ezen a címen a rendszer elutasít, nem csatlakozik. Alkalmazásra kerül a mentett integrációs idő, a képkockák átlagolása és az automatikus expozíció, valamint a kap-korrekció:

| Mentett sapka | Csatlakozás |
| --- | --- |
| Egy sapka-profil (pl. `fov_30`) | Az a profil. |
| Nincs kiválasztva | `sunshine_cosine`, a csatlakozás alapértelmezett értéke, amelyet a feltételezettként rögzítettek (`cap_id_source=auto_default`), ezért a feldolgozás figyelmeztetést ad. Az érzékelők fedél nélkül kerülnek forgalomba: válassza ki a felszerelt fedelet. |
| `none` — a grafikus felhasználói felületen **Nincs (csupasz érzékelő)**ként választva | Nincs kupak-korrekció. |

A DAQ-E-S nem fogad el fedélprofilt, függetlenül attól, hogy mi lett mentve: diffúzora a kalibrációjának része (lásd [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)). Egy korábbi Chloros által mentett projekt a fedél nélküli érzékelőt „nincs választás”ként rögzítette, ezért az `sunshine_cosine`-szel csatlakozik újra; válassza ki a GUI-ban ismét a **Nincs (fedél nélküli érzékelő)** lehetőséget, majd mentse el a projektet.

Egy olyan érzékelő, amelyet egy másik kliens már megnyitott a poolban (CLI, egy másik szkript), a beállítások újbóli alkalmazása nélkül, változatlanul csatlakozik, és az `disconnect_all()` futtatásban hagyja; az `connect_all()` által megnyitott érzékelő felszabadul. Az `connect_daq_sensor()`-hez hasonlóan a Chloros-felület **nem** része a poolnak: zárjon be egy érzékelőt a GUI-ban, mielőtt egy szkriptből csatlakoztatná a projektet.**Követelmények és a jelentés.** A fényérzékelőkhöz egy futó Chloros háttérprogramra van szükség, amely be van jelentkezve egy fizetős Chloros+ szintre (Copper vagy magasabb), és az `connect_all()` soha nem indít ilyet. Ha nincs futó háttérprogram, minden mentett érzékelő fel van sorolva az `report["sensors"]["errors"]` alatt, jelezve ezt. Azok az érzékelők is ott szerepelnek, amelyeket a háttérprogram nem tud megnyitni, a háttérprogram által megadott ok feltüntetésével; az `report["sensors"]["missing"]` fényérzékelők esetében üres marad.

---

## Közvetlen hardver (háttérprogram nélkül)

Ha a LATTICE kamerák (CI, fej nélküli robotok, beágyazott rendszerek) esetében egyáltalán nem szeretnél függni a háttérprogramtól, importáld közvetlenül az `lattice_sdk` fájlt — ezt az `chloros_sdk` újraexportálja. Figyelem az `CAMERA_AVAILABLE`-re vonatkozóan: Az `lattice_sdk` szerepel az SDK wheelben, de az Arena SDK futtatókörnyezet és az OpenCV (`cv2`) jelenlétét igényli, és a wheel egyikét sem telepíti. A fényérzékelők a háttérrendszeren keresztül működnek — [`connect_daq_sensor()`](#daq-sensor-sessions) vagy egy projekt `proj.sensors`-én keresztül haladnak — egy kivétellel, a DAQ-M sávmagnitúdó-segédprogrammal [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes), amelyhez szintén szükség van az `bleak`-re.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Előre beállított értékek és a trigger

A négy előre beállított érték közül három **szabad futású**: a kamera folyamatosan exponál, és egy
`capture()` visszaadja a következő képkockát. Az `triggered` a kivétel — ez a
kamerát a 2. vonalon megjelenő hardveres élre érzékenyíti, így addig nem rögzít semmit, amíg ilyen nem érkezik.

| Előre beállított mód | Kioldó | Használata, ha |
| --- | --- | --- |
| `default` | szabad futás | általános használat |
| `high_speed` | szabad futás | 8 bites, 60 fps korlát, rövid expozíció |
| `high_quality` | szabad futás | 12 bites, nincs fps-korlát — a szokásos választás állóképekhez |
| `triggered` | **készenlét, 2. vonal** | a kamera egy M8-as szinkronkábelhez van csatlakoztatva, és valami más indítja el |

Ha az `triggered`-et választod (vagy magad állítod be az `trigger_mode="On"`-et) úgy, hogy semmi
nem vezérli a 2. vonalat, akkor minden `capture()` időtúllépésbe kerül — helyesen, mivel azt kérted
a kamerától, hogy várjon. Az SDK elmagyarázza ezt, amikor ez megtörténik; lásd
[SC_ERR_TIMEOUT a rögzítés során](#direct-hardware-backend-free).

> **Megjegyzés — A csatlakozáskor megjelenő „GVSP probe” / `SC_ERR_TIMEOUT -1011` üzenetek nem hibák.**
> Csatlakozáskor az SDK** jumbo keretek** (9000 bájtos GVSP-csomagok) használatát próbálja kialkudni a nagyobb átviteli sebesség érdekében. Közvetlen pont-pont hálózati kártya-kapcsolaton (pl. link-local `169.254.x.x` cím) a hálózat általában nem képes jumbo keretek továbbítására, ezért ez a próba időtúllépésbe kerül, és olyan sorokat rögzít, mint például:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Ez a **tervezett tartalék megoldás**: az SDK automatikusan visszatér a szabványos 1500 bájtos csomagokhoz, és a kamera továbbra is normál módon csatlakozik (a következő `[chunk-enable …]` sorok a normál csatlakozási sorozat részét képezik). A rögzítés továbbra is működik.
>
> Ezt a próbát kihagyhatja, de **ez nem csupán a naplóbejegyzések elhallgattatására szolgál — hanem kikapcsolja a jumbo kereteket.** A kamera a Don't-Fragment pingekre legfeljebb 1500 bájtig válaszol, függetlenül attól, hogy milyen jó a hálózatod, így a ping-teszt önmagában soha nem tudja felismerni a jumbo kereteket; ezt csak ez a próba képes megtenni. Ha letiltod, a kamera örökre a szabványos 1500 bájtos csomagokat fogja használni, bármilyen hálózaton:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Csak olyan hálózaton érdemes ezt használni, amelyről *biztosan tudod*, hogy nem képes jumbo-csomagokat továbbítani, ahol ez kameránként körülbelül egy másodpercet spórol a csatlakozási időből. Mivel ez valódi kompromisszum, nem pedig pusztán kozmetikai változtatás, az SDK most már jelzi ezt, ha használod:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Hagyd békén, hacsak nincs rá okod.**Ha engedélyezve marad, minden kamerát egyszer mérnek meg háttérszolgáltatási munkamenetenként, majd újra, amikor a kapcsolata megváltozik (másik adapter, másik kapcsolatsebesség, vagy a kapcsolat megszakadása, például egy kapcsoló cseréje miatt); az újracsatlakozáskor az eredményt újrahasznosítják.**Ha a kapcsoló mögött változás történt, de a kapcsoló továbbra is működik, indítsd újra a háttérszolgáltatást az újraméréshez.** Az első csatlakozás alkalmával ez az ellenőrzés néha néhány percet is igénybe vehet, és a napló ezt jelzi, ha így történik.
>
> Ha *szeretnéd* a jumbo átviteli sebességet, engedélyezd a végpontok közötti jumbo átvitelt (NIC MTU 9000 + egy olyan kapcsoló, amely átadja őket), vagy rögzítse az `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` paranccsal, ha tudja, hogy a kapcsolat támogatja — bár a végleges beállítás helyett inkább a parancsonkénti `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` használatát javasoljuk, mivel a rögzített méret kihagyja a próbát, és nem alkalmazkodik az előtte lévő hálózathoz. Az útvonalon lévő **minden** eszköznek át kell engednie a jumbo-csomagokat — beleértve minden PoE-elosztót vagy -injektort is, ami általában az oka annak, hogy egy egyébként jumbo-kompatibilis rendszer nem tudja továbbítani őket.

> **Az `SC_ERR_TIMEOUT -1011` az `capture()` / `grab*()` során egy másik probléma — ez egy valódi hiba.**
> A fenti megjegyzés kizárólag a**connect-time probe**által naplózott `-1011`-re vonatkozik. Ugyanez a hiba egy**rögzítés** során jelentkezve azt jelenti, hogy a kamera rendben csatlakozott, de nem küld képeket:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> A döntő jel az, ha a kamera *vezérlő* csatornája rendben van — a felismerés működik, a beállítások és az `[chunk-enable …]` írások mind sikeresek —, miközben *minden* képkocka időtúllépést okoz.
>
> **A leggyakoribb ok az, hogy a kamera hardveres kiváltásra van beállítva.** Az `trigger_mode="On"` és az `trigger_source="Line2"` parancsok esetén a kamera egyáltalán nem küld semmit, amíg az M8 szinkronkábelre nem érkezik elektromos jel. Ha nincs kábel, amely ezt a vonalat vezérli, minden adatlekérés örökké vár. A kamera nem romlott el, és a hálózat is rendben van — pontosan azt teszi, amit megparancsoltak neki.
>
> Az `CameraSettings()`, valamint az `default` / `high_speed` / `high_quality` előre beállított értékek szabad futást eredményeznek, és egy késleltetés miatt leálló rögzítés magyarázatot ad ahelyett, hogy pusztán `-1011` kódot írna ki. Az `PRESETS["triggered"]` a tervezésnek megfelelően aktiválja a Line2-t.
>
> Bármely kamera szabad futásának kikényszerítéséhez:
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Ha az `trigger_mode="Off"` parancs mellett is időtúllépés történik, akkor a kamera valóban nem továbbít adatokat — küldje el nekünk a naplófájlt és az `ip link show` parancsot.

#### Színprofilok (RGB élő előnézet) — `set_color_profile`

Az `LatticeCamera.set_color_profile(profile, custom_cct_k=None)` beállítás meghatározza az RGB kamerák **élő előnézetéhez** használt kijelző-színprofilt (a multispec kamerák figyelmen kívül hagyják ezt a beállítást):

| Profil | Jelentés |
| --- | --- |
| `raw` | A radiometrikus lánc teljes mellőzése. |
| `linear` | DSNU + flat + WB, nincs CCM, nincs gamma. |
| `natural` | Lineáris + mért CCM + sRGB gamma, kizárólag az egyszerű utómunkával (színárnyalat-simítás + fényes részek telítettségének csökkentése) — a valósághű alapbeállítás. |
| `enhanced` | `natural` plusz a teljes hub-parity utómunka (színszegélyek eltávolítása, élénkség, CLAHE helyi kontraszt). Gazdagabb megjelenés, nagyjából **kétszeres képkockánkénti utómunkaköltséggel**, így alacsonyabb LIVE képkockasebességgel. |
| `custom_temp` | `natural`, de a fehér egyensúly (WB) rögzítve van az `custom_cct_k` Kelvin-értékre értékre van rögzítve (a DLS figyelmen kívül marad; a háttérrendszer oldalán 2000–10000 K-ra van korlátozva). |

A profil egy **kizárólag élő előnézetre szolgáló** sebesség/megjelenés-szabályozó: a mentett felvételek mindig a teljes, gazdag megjelenést kapják, függetlenül a kiválasztott profiltól, így az `natural` kiválasztása a képkocka-idő visszanyerése érdekében nem rontja a lemezre kerülő anyag minőségét. Egy ismeretlen profil az `ValueError` értéket eredményezi; ha elérhető egy chloros háttérrendszer, a változás oda is elküldésre kerül (POST), így a következő előnézeti képkocka már tükrözi azt (a háttérrendszer nélküli közvetlen SDK-felhasználók is megkapják a beállítások változását).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Fénypontok (RGB-kijelző) — `set_highlights`

Az `LatticeCamera.set_highlights(value)` a **kijelző** legfényesebb, legfehérebb képpontjait RGB-kamerákon — ez a javítás megoldja azt a problémát, hogy egy fényes fehér felület lapos fehér foltként jelenik meg. Az `value` az `-100`..`0` (korlátozott); az `0` ki van kapcsolva, és a megjelenítés ilyenkor pontosan olyan, mintha a beállítás nem lenne megadva. Az erősebb beállítások csökkentik a kijelző fehérjét (kb. 60%-ra az `-100` értéknél) és kiszélesítik a tónusgörbét, így egy olyan felület, amelyet az előnézet automatikus fényereje tiszta fehérre emelt, visszakapja a részleteit.

Ez egy kijelzővezérlő, de a telítettségtől és a kontraszttól eltérően (amelyek csak az élő előnézetet módosítják) a mentett fájlokat is megváltoztatja: az élő előnézet és a **kijelzőszintű** felvételek követik ezt a beállítást, így azok a fájlok is sötétebbek lesznek (az `Chloros:DisplayHighlights` értéket hordozzák); a sugárzási és visszaverődési exportok soha nem, és nem tudja helyreállítani a levágott érzékelőadatokat — ehhez csökkentse az expozíciót. A multispektrális és monokróm kamerák figyelmen kívül hagyják. Az érték az `settings.highlights`-en kerül tárolásra, és POST-olásra kerül egy elérhető chloros háttérrendszerbe, pontosan úgy, mint az `set_color_profile`; a CLI-megfelelője az `lattice highlights --set N`.

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### Mono (M3M) kamerák és az `Calibration`

Egy mono **M3M** kamera (`M3M-<lens>-F<wavelength>`) egysávos: egy szürkeárnyalatos sík, nincs Bayer-mozaik, nincs 3×3 spektrális átvitel-mátrix. Az `Calibration` felismeri ezt, és kiad egy `is_mono` jelzőt. A reflexió továbbra is sávonkénti radiometrikus térképként érvényes (a keverés feloldása az identitásmátrix), de az egyetlen kamerán végzett többsávos számítások értelmes eredményt adnak, nem pedig értelmetlen értékeket:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Ha monokróm hardverből szeretnénk növényzetindexet készíteni, egyesítsünk több, különböző hullámhosszúságú M3M kamerát egy igazított többsávos képcsomagba (lásd [Képcsomag-igazítás](#array-alignment)), és számítsuk ki az indexet a képcsomag egészén, nem pedig egyetlen kamerán.

#### DAQ-M sávmagnitúdók — `DLS.band_magnitudes`

Az `DLS(mac_address, filter_model="RGN-IMX265")` a saját BLE MAC-címével csatlakozik egy DAQ-M-hez Bluetooth-on keresztül. Ez az egyetlen fényérzékelő-osztály, amely nem a háttérrendszeren keresztül, hanem közvetlenül kommunikál az érzékelővel, ezért a közvetlen kamerás osztályokhoz szükséges elemekre (`CAMERA_AVAILABLE`: az Arena SDK futásideje és az OpenCV) mellett szüksége van az `bleak`-re a Bluetooth-kapcsolathoz (`pip install "chloros-sdk[camera]"` vagy `pip install bleak`; ennek hiányában az `DLS` kiváltja az `DLSError` hibát). Ha egy máshol csatlakoztatott DAQ-M (a háttérprogram pooljában, egy projektben, a GUI-ban) leáll a hirdetéssel, ezért ott először azt kell felszabadítani. **Az `mac_address` szükséges**: nincs alapértelmezett érték, mivel minden egységnek megvan a sajátja, és az `DLS()` ennek hiányában az `DLSError`-et indítja el, mielőtt bármilyen csatlakozási kísérletet tenne. Keresse meg a MAC-címet az `discover_daq_sensors(transports=["ble"])` segítségével (lásd [Felfedezés](#discovery--finding-an-address-to-connect-with)) vagy az `chloros-cli daq pool-discover --only ble` segítségével.

Az `DLS.band_magnitudes(spectrum=None, *, spectral_response=None)` integrálja a DAQ-M spektrumot (a legfrissebb értéket, ha az `spectrum` értéke `None`; `DLSError`, ha még nincs ilyen) a kamera három csatornájának válaszaiba, és visszaadja az `{"red": …, "green": …, "nir": …}` értéket – a kulcsszavak a csatornákat ebben a sorrendben nevezik meg; egy RGB-kamera görbéi esetén a harmadik (`nir`) a kék csatorna.

- Az `spectral_response` hiányában a szűrőmodell flotta-görbéjét használja (amelyet az `filter_model` és az `DLS` készítéséhez használtak), amelyet egy egységen mértek.
- Az `spectral_response` helyette a kamera **saját** mért görbéit veszi alapul: egy dict `{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}` formájában, amely a kamera kalibrációs csomagjában (`color_correction.spectral_response_smoothed` — RGB-kamerák; a multispektrális csomagok nem tartalmaznak ilyen görbét) van tárolva. Ha nem tudja használni a dict-et, akkor a flotta-görbére tér vissza.

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

A CLI `chloros-cli lattice dls --camera SERIAL` parancsa ugyanezt végzi el a háttérrendszeren keresztül: megkeresi a kamera görbéjét a betöltött vagy gyorsítótárban tárolt kalibrációs csomagjában, és megadja, hogy melyik görbét használta. HTTP-en keresztül ez az `POST /api/dls/cli-read` az `mac`-szel (szükséges: ennek hiányában a válasz HTTP 400 hiba, `code` `MAC_REQUIRED`), a testben pedig az `camera_serial` szerepel; a válasz `band_magnitudes_source` értéke `camera <serial>`, ha a kamera saját görbéjét használták, vagy `model <filter>` a flotta görbéje esetén (görbe nélküli kamera: nem RGB, vagy nincs betöltött csomag vagy gyorsítótárban).

---

## Kivételek

Fogja meg az alaposztályt, hogy kezelje a „Chloros-ban bármi rosszul sült el” helyzeteket:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> Az `ChlorosAuthenticationError` és az `ChlorosConfigurationError` az `chloros_sdk` (és az `chloros_sdk.exceptions`) attribútumai, de nem szerepelnek az `__all__`-ben, ezért az `from chloros_sdk import *` nem hozza be őket: importálja őket név szerint. Az `ChlorosError` családot az `ChlorosLocal` és az arra épülő függvények (`process_folder()`, `process_lattice_capture()`) hozza létre; az intelligens csatlakozási felület az `ChlorosConnectError`-et hozza létre.

Hierarchia:

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Teljes folyamatot bemutató példák

### 1. Mappa feldolgozása egyéni haladási sávval

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Élő LATTICE-tömb → Reflektancia + DAQ-referencia

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Projektvezérelt rögzítési kampány

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Többkamerás képkocka-adatfolyam → NumPy-feldolgozási folyamat

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Headless közvetlen hardveres (háttér nélküli) rögzítési szkript

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Képességvizsgálat 4 kamerás rendszer csatlakoztatása előtt

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Felvételi recept megfelelője (tiszta Python)

Az `run_project_recipe()` egy CLI-receptet futtat változatlan formában (lásd [Projektmappa és receptek](#project-folder--recipes)). Python-ben megírva ugyanazok a lépések:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## MCP-kiszolgáló (AI-asszisztensek)

Az `chloros-mcp` lehetővé teszi, hogy egy olyan AI-asszisztens, amely a [Model Context Protocol](https://modelcontextprotocol.io) (MCP) nyelvet használja – például a Claude Desktop, a Claude Code vagy a Cursor –, vezérelje a Chloros-t: feldolgozza a képanyagot (projekt létrehozása, a felvételeket tartalmazó mappa importálása, beállítások kiválasztása, feldolgozás, a futtatás eredményeinek jelentése), valamint a LATTICE kamerák, szenzortömbök és DAQ fényérzékelők vezérlése (csatlakoztatás, előnézet, expozíció és szín beállítása, felvétel készítése, rögzítés, firmware frissítése). Eszközei lefedik az SDK funkcióit. Ön egyszerű nyelven adja meg a parancsot, az asszisztens pedig meghívja a szerver eszközeit:

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

A szerver ezen az SDK-n alapul: a feldolgozó eszközök a [`ChlorosLocal`](#image-processing--chloroslocal) parancsot, a hardveres eszközök pedig a [smart-connect-hívásokat](#smart-connect-for-lattice-cameras), a [DAQ-munkameneteket](#daq-sensor-sessions), az `ChlorosProject`-et és a [kamerasegédprogramokat](#camera-utilities), amelyek mind ugyanazzal a háttérrendszerrel kommunikálnak, amelyet a Chloros-alkalmazás is használ. Kérései az `X-Chloros-Client: sdk` azonosítót hordozzák, így a háttérrendszer végrehajtja az SDK bejelentkezési és csomagellenőrzéseit, és az SDK szabályai is érvényesülnek (például a Texture Aware funkciót elutasítja a LATTICE-felvételek esetében).

### Telepítés

A szerver az `chloros-sdk` csomagban érkezik; nincs más telepítendő. Az SDK telepíti az `chloros-mcp` parancsot, amely megegyezik az `python -m chloros_sdk.mcp_server` parancsával:

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows:** a Chloros telepítője pip-paranccsal telepíti az SDK-t a Python-ba, ami az `chloros-mcp.exe` parancsot az adott Python `Scripts` mappájába helyezi.
- **Linux (.deb):** a csomag a rendszer `python3` mappájába telepíti az SDK-t, és az `chloros-mcp` parancsot az `/usr/bin` mappába helyezi, így az minden felhasználó `PATH`-ben elérhető. Egy virtualenv-ben az `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` hozzáadja az `chloros-mcp`-et a virtualenv `bin`-éhez.

A szerver kizárólag az SDK-n (Python 3.7+) felül a standard könyvtárat használja. Egy MCP-kliens indítja el, és a stdin/stdout-on keresztül kommunikál vele; Ön nem futtatja saját maga, kivéve az `--help` esetét.

### Kliens beállítása

**Claude Desktop:**adja hozzá a szervert az `claude_desktop_config.json`-hez (Windows esetén az `%APPDATA%\Claude\claude_desktop_config.json`-hez; a**Beállítások > Fejlesztő > Konfiguráció szerkesztése** menüponttal nyitható meg), majd indítsa újra a Claude Desktopot:

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Claude kód:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Cursor:** ugyanaz az `mcpServers` bejegyzés, mint a Claude Desktop esetében, az `~/.cursor/mcp.json` fájlban (minden projektben; Windows rendszeren: `%USERPROFILE%\.cursor\mcp.json`) vagy az `.cursor/mcp.json`-ben egy projektben.

Az opciókat az `args`-be, vagy a Claude Code parancs után (`claude mcp add chloros -- chloros-mcp --no-auto-start`) kell megadni. Ha az `chloros-mcp` helyett egy adott Python SDK-jét szeretné használni az `PATH`-en, futtassa a modult azzal a Python-verzióval (teljes elérési útját `command` formában, ha nem az első `python3` az `PATH`-en):

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

Az ügyfél a szervert a shellje nélkül indítja el, így ha azt jelzi, hogy a szerver indítása sikertelen volt, akkor az `chloros-mcp` valószínűleg nem szerepel azon az `PATH`-en, amelyet az ügyfél lát. Ehelyett adja meg a teljes elérési utat (Windows esetén az `where chloros-mcp`, Linux esetén az `which chloros-mcp`, ahol a .deb fájl neve `/usr/bin/chloros-mcp`), vagy futtassa a modult az SDK-t tartalmazó Python-val.

### Beállítások

| Beállítás | Alapértelmezett | Mit csinál |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | A használni kívánt Chloros-háttérprogram. |
| `--no-auto-start` | off | Soha ne indítson háttérprogramot: használjon egy már futó háttérprogramot, például a Chloros-alkalmazásét, a Linux-háttérszolgáltatást, vagy egy alagutat egy másik gép (lásd [Távoli háttérmód](#remote-backend-mode-pip-only-host-via-tunnel); ebben az esetben a mappaútvonalaknak mindkét gépen ugyanazt a mappát kell megjelölniük). |
| `--backend-exe PATH` | ahová a Chloros telepíti | Az a háttérprogram, amelyet el kell indítani, ha egyik sem fut, olyan Chloros esetén, amelyet az SDK nem keres. |
| `--log-level LEVEL` | `WARNING` | `DEBUG`, `INFO`, `WARNING` vagy `ERROR`. A naplófájlok a stderr-be kerülnek, amelyet a legtöbb kliens az MCP-naplójában jelenít meg; a stdout csak a protokollüzeneteket továbbítja. |

### Eszközök

Minden eszköz argumentumai megegyeznek az általa burkolt SDK-hívás argumentumaival; az ügyfél mindegyiket a teljes sémájával együtt sorolja fel. Az elérési utak abszolútak.

**Feldolgozás**

| Eszköz | SDK-hívás | Mit csinál |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | Létrehoz egy projektet, és azt megnyitott projektté teszi, amelyen a feldolgozó eszközök működnek. Egy egyszerű név esetén a mappa az alapértelmezett projektmappában jön létre; abszolút elérési út megadása esetén ott jön létre. |
| `open_project` | `ChlorosLocal.open_project` | Megnyit egy mentett projektet, és azt nyitott projektté teszi. Egy áthelyezett projekt elérési útjait átirányítja a projekt jelenlegi helyére; az `relink_folder` (abszolút) megnevezi azt a mappát, amelyben a projekt többi hiányzó fájlja jelenleg található. Visszaadja az `source_files` és az `image_count` értékeket. Nem található: hibaüzenet, amely felsorolja, hol voltak, és megnevezi az `relink_folder`-et; semmi sincs megnyitva. |
| `import_images` | `import_images` | Beolvassa a Survey3 `.RAW`+`.JPG` párokat, a „LATTICE” `.tif` rögzítéseket és DAQ `.daq` felvételeket; visszaadja az `count` (képek) és az `scan_count` (felvételek) elemeket. |
| `configure_processing` | `configure` | Csak az átadott beállításokat módosítja, másokat nem (értékek: [Támogatott értékek](#supported-values)); `custom_settings` a megadott teljes konfigurációt elküldi. |
| `get_project_settings` | `get_config` | Beolvassa a megnyitott projekt mentett feldolgozási, exportálási és indexelési beállításait. |
| `start_processing` | `process(wait=False)` | Elindítja a megnyitott projekt feldolgozását, és azonnal visszatér. |
| `get_processing_status` | `get_status` | Jelentést küld az `state` (`idle`, `starting`, `processing` vagy `finished`) és az előrehaladást; a futtatás befejezése után az `summary`, az `export_check_failure` (ha az exportált adatok nem voltak ellenőrizhetők), és az `image_failure` (ha képek vesztek el): a többi termék fel van írva; jelentsd a futtatást sikertelennek. |
| `process_folder` | `process_folder` / `process_lattice_capture` | Egy hívásban létrehoz egy projektet egy mappához, importálja azt, konfigurálja és elindítja a futtatást (`lattice: true` a LATTICE-felvételek esetében). Amíg egy futtatás folyamatban van, a rendszer elutasítja a műveletet, mielőtt bármit is létrehozna, és a háttérrendszer ekkor szintén elutasítja a projektmódosítást bármely eszközről. |
| `export_light_sensor` | `export_light_sensor` | A projekt fényérzékelőjéhez kalibrált `.daq` és `.csv` fájlokat ír felvételeihez kalibrált `.daq` és `.csv` fájlokat ír, a képek feldolgozása nélkül, és a gyári kalibrációt a MAPIR-felhőből tölti le, ha az nincs a gyorsítótárban (lásd [Fényérzékelő-felvételek](#light-sensor-recordings--calibrated-daq--csv)). |
| `read_image_audit_tags` | `read_image_audit_tags` | Beolvassa egy feldolgozott kép Chloros-ellenőrzési metaadatait (forrás, kalibráció, alkalmazott lépések). |

**Kamerák és sorozatok**

| Eszköz | SDK-hívás | Mit csinál |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`, `list_daq_sensors` | Mi van jelenleg csatlakoztatva bármely kliensen, mindegyik az `connected_here`-szel; valamint a szerveren futó feladatok. |
| `discover_devices` | `discover_lattice_cameras`, `discover_daq_sensors` | Csatlakoztatható kamerák és fényérzékelők. |
| `get_device_status` | `CameraSession.status`, `ArraySession.status` / `capability`, `DAQSensorSession.status` | Egy kamera, egy tömb (kapacitásával és állapotával együtt) vagy egy fényérzékelő aktuális állapota. |
| `analyze_array_network` | `analyze_array_network` | Mit tud ez a gazdagép és a hálózat egy javasolt rendszer számára biztosítani. |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | Kamera vagy szinkronizált rendszer (először a vezérlő) csatlakoztatása. |
| `connect_project_devices` | `open_project(...).connect_all` | Csatlakoztassa az összes olyan eszközt, amelyre egy mentett projekt emlékszik, a beállításaikkal együtt. |
| `disconnect_device` | `disconnect` / `force_disconnect` | Kamerát, rendszert vagy fényérzékelőt szabadítson fel (lásd: Viselkedés). Az `keep_settings: true` egy sorozaton minden kamera beállításait a saját memóriájába menti, az `clear_settings: true` pedig visszaállítja azt a memóriát (eredménye az `camera_memory` parancsot hajtja végre). |
| `disconnect_project_devices`, `align_project_arrays`, `capture_project_devices` | `disconnect_all`, `align_arrays`, `capture_all` | A projekt összes eszköze együtt. |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | A kamera beállításainak olvasása vagy írása, barátságos név vagy GenICcsomópontnév alapján. |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | A tömb sebességének rögzítése, vagy visszaállítása az automatikus sebességre. |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | Egy felvétel (az `fastest` és az `wait_for_exposure` az `capture_fastest` és a beállított expozíciójú felvétel). `output_dir` hiányában a nyitott projekt`captures` mappájába menti, és a fájlok bekerülnek a projektbe (ahol a feldolgozási futás befejezéséig visszatartásra kerülnek); ezt jelzi a válasz `project_add` is. |
| `start_capture_run` | `ArraySession.continuous`, `capture_repeated` | Folyamatos vagy időközönkénti rögzítés, feladatként. |
| `start_array_recording` | `ArraySession.record` / `burst` | Kombinált indexű videó vagy nyers sorozatfelvétel, feladatként. |
| `build_burst_video` | `ArraySession.build_video` | Sorozatfelvétel újrafeldolgozása kalibrált videóvá, feladatként. |
| `get_job_status` / `stop_job` | | Feladat követése vagy leállítása. |
| `get_camera_preview` | | Az egyes kamerák élő előnézete, kép formájában visszaadva. |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | Egy képkocka (vagy egy szinkronizált sorozat) egy feldolgozási szinten, mentés nélkül: síkonkénti statisztikák és mindegyikhez egy-egy PNG-fájl. |
| `calibrate_array_alignment`, `get_array_alignment`, `clear_array_alignment`, `nudge_array_alignment`, `export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`, `tweak_alignment`, `export_alignment`, `import_alignment` | A tömb igazítása. |

**Fényérzékelők**

| Eszköz | SDK-hívás | Működése |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | DAQ-U, DAQ-M vagy DAQ-E csatlakoztatása (az `cap_id` beállítja a kondenzátor-korrekciót). |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | A legfrissebb mérési értékek: időbélyeg, telítettség, spektrumtartomány és csúcs, valamint a hullámhossz-tartományokra átlagolt spektrum (`full_spectrum` minden pontra vonatkozóan). |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | Az adatfolyam elindítása vagy leállítása. |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | `.daq` fájl rögzítése feladatként; `duration_s` automatikusan leállítja. |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | A fedélkorrekció (`none` csupasz érzékelő esetén). |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | Melyik érzékelő adja az egyes kamerák és sorozatok fényvisszaverődési értékeit. |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | A sávkészletek, valamint két adatfolyam-érzékelő környezeti és objektum-reflektanciája, illetve indexei. |
| `get_light_sensor_device_info`, `set_light_sensor_network`, `set_light_sensor_auto_stream`, `enable_light_sensor_ptp`, `reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`, `set_auto_stream`, `enable_ptp`, `reboot`, `update_firmware` | A DAQ-E saját beállításai, a készülékben mentve. |

**Kamera segédprogramok** ([Kamera segédprogramok](#camera-utilities))

| Eszköz | SDK-hívás | Működése |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`, `camera_color_adjust`, `camera_highlights` és azok beállító funkciói | Egy RGB-kamera kijelzőjének színei: profilja és színhőmérséklete, telítettsége / kontrasztja, a fényes részek. |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | Egyszeri fehéregyensúly, vagy automatikus / kikapcsolt / rögzített arányok. |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | Kamera-előbeállítások. |
| `list_filter_models` | `camera_filter_models`, `camera_filter_model_info` | A szűrőmodellek, vagy egy név szerint. |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | Firmware-verziók és egy villanás (alapértelmezés szerint egy feladat). |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`, `wake_camera` | Áramellátási és hőállapot; alvó vagy ébren állapot. |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | A nézetben lévő kalibrációs célpontból származó együtthatók. |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | A kamera hálózati adapterei, becslés és a javítás (Csak rendszergazda). |
| `apply_vignette_correction` | `apply_vignette_correction` | Képek mappájának vignettakorrekciója. |
| `read_dls` | `read_dls` | Egy DLS-leolvasás. |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | A gazdagép hardvere és futási környezete. |

**Tranziens kamerák, igazítási profilok és indexek** ([Tranziens tömbök, igazítási profilok és indexek](#transient-arrays-alignment-profiles--indices)). Minden olyan eszköz, amely kamerákat nyit meg, az `keep_settings` / `clear_settings` értéket (alapértelmezés szerint false): lásd [Kamera memória](#camera-memory--keep_settings--clear_settings).

| Eszköz | SDK-hívás | Mit csinál |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | Egy kamera felvétele, vagy több kamera szinkronizált streamelése, feladatként (az `duration_s` magától leállítja). |
| `capture_cameras_transient` | `capture_cameras_transient` | Egy szinkronizált felvétel tömb csatlakoztatása nélkül. |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | A trigger szinkronizálásának ellenőrzése, és hogy mely kamerák vannak a szinkronizáló vonalon. |
| `analyze_array_models` | `analyze_array_models` | Mit érne el egy ilyen modellekből álló rendszer ezen a hálózaton. |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | Igazítási profilfájl kiszámítása és mentése (az `options` tartalmazza a finomabb megoldó beállításokat). |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | Igazított többsávos kép, vagy azok sorozata feladatként. |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | Egy profilfájl tartalma; sávsorrendje és az engedélyezett kamerák. |
| `list_index_presets`, `compute_index`, `capture_live_index` | `index_catalog`, `compute_index`, `capture_live_index` | Előre beállított értékek és színátmenetek indexe; egy többsávos TIFF-fájl indexe; egy lépésben történő rögzítés az indexbe. |

**Időszinkronizálás, projektmappa és receptek** ([PTP Grandmaster](#ptp-grandmaster--time_sync_status), [Projektmappa és receptek](#project-folder--recipes))

| Eszköz | SDK-hívás | Mit csinál |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`, `time_sync_peers`, `time_sync_cameras` | A PTP-főmester állapota, társai és az egyes tömbkamerák PTP-állapota. |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | Indítsa újra; állítsa be a prioritásait. |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | Az a mappa, amelyben az új projektek készülnek. |
| `run_project_recipe` | `run_project_recipe` | Futtassa a mentett projekt receptjét, feladatként, a naplóval együtt. Elsőként ellenőrzés: a hibás receptet elutasítja, és nem futtat semmit; a csatlakozatlan tömbökről / érzékelőkről figyelmeztetést ad (nem csatlakoztatja őket: először az `connect_project_devices` parancsot kell futtatni). Az `dry_run` visszatér az ellenőrzésből, és nem futtat semmit. |

Nincs olyan eszköz, amellyel a mentett projektet feldolgozás céljából megnyithatnánk: minden feldolgozási feladat az `create_project` (vagy az `process_folder`) parancsával indul.

### Követelmények

- **A Chlorosnak ugyanazon a számítógépen kell lennie.** A feldolgozást a Chloros háttérprogram végzi. A szerver helyette használhat egy alagúton keresztül elérhető háttérprogramot is (`--backend-url` az `--no-auto-start`-szel, lásd [Beállítások](#options)).
- **Bejelentkezve kell lennie azon a számítógépen**, a Chloros alkalmazásban vagy az `chloros-cli login` segítségével. A szerver nem tud bejelentkezni helyetted: ha kijelentkezett állapotban hívod meg az eszközt, az a háttérrendszer indokával együtt a „Nincs bejelentkezve a Chlorosba” üzenetet adja vissza, és az asszisztens tájékoztat téged erről.
- **Bármely fizetős Chloros+ csomag (Copper vagy magasabb).** A szerver SDK-hozzáféréssel rendelkezik, az ingyenes Iron csomagban pedig nincs ilyen: az ingyenes csomagban meghívott eszköz a „Nem elérhető ebben a Chloros-csomagban” üzenetet adja vissza a háttérrendszer indoklásával.
- Python 3.7 vagy újabb verzió az SDK-vel, valamint egy MCP-kliens, amely helyi (stdio) szervereket indít.

### Működés

- **A háttérszolgáltatás.** A szerver elindítása vagy az eszközök felsorolása nem indít el semmit; az első eszközhívás hozza létre a kapcsolatot. Ha a Chloros-alkalmazás (vagy a Linux-háttérszolgáltatás) fut, a szerver annak háttérszolgáltatását használja, és futtatásban hagyja. Ha nincs futó háttérszolgáltatás, a szerver elindítja a telepítettet, ablak nélkül. Amikor a szerver leáll (bezárja az ügyfelet, vagy az újraindítja a szervert), leállítja az általa elindított rögzítési feladatokat és fényérzékelő-felvételeket, és felszabadítja a csatlakoztatott eszközöket; a háttérszolgáltatás továbbra is fut, így a befejezetlen feldolgozási folyamatok végigfutnak, majd 30 perc elteltével, ha semmi sem használja, a háttérszolgáltatás magától leáll. Az `--no-auto-start` verzióval a szerver soha nem indít el háttérszolgáltatást: ha nincs futó háttérszolgáltatás, az eszközhívás arra kéri, hogy nyissa meg a Chloros-alkalmazást, és miután megnyitotta, a következő hívás már létrehozza a kapcsolatot.
- **Egy nyitott projekt.** A háttérprogramnak egy nyitott projektje van, amelyet az Chloros-alkalmazással oszt meg, amikor az alkalmazás nyitva van, ezért ne dolgozzon egy másik projekten az alkalmazásban, amíg az asszisztens működik.
- **Csak a megadott beállítások változnak.** Az `configure_processing` minden olyan beállítást a mentett értékén hagy, amelyet nem adtak meg (az SDK `configure()` parancsa a debayert, a vignettakorrekciót, a reflexiós kalibrációt, a PPK-t és az exportformátumot alapértelmezett értékekre állítja vissza, ha azokat nem adják meg; a szerver a projektből tölti be őket). Ha nem tudja beolvasni a projekt mentett beállításait, akkor nem változtat semmit, és erről értesítést ad, kivéve, ha mind az öt beállítást megadják. Az `indices` felülírja a projekt indexlistáját. Az eredmény felsorolja, hogy mit küldtek `settings_sent` néven, az `warnings` alatt pedig azokat az indexeket sorolja fel, amelyeket a kamera sávjai nem tudnak létrehozni.
- **A feldolgozás „start-then-poll” típusú.** Az `start_processing` a futtatás megkezdésekor azonnal visszatér, amiután a feldolgozás percektől órákig tarthat, és az asszisztens ezt követi az `get_processing_status` parancsával. Az `state` lehet `idle` (nincs feldolgozás), az `starting` (a háttérrendszer még nem jelentette a futtatást, legfeljebb 20 másodpercig), az `processing` vagy az `finished`, a futtatás [összefoglalója](#post-run-summary--hints): mit exportált, és figyelmeztetéseket, ha a vártnál kevesebbet exportált (nincsenek képek, nem talált kalibrációs célt). A termékek az alkalmazáshoz hasonlóan a projektmappa alá kerülnek. Az `start_processing` futtatás elutasításra kerül, ha egy futtatás folyamatban van, és a válaszadást leállító háttérrendszer hibaüzenetként kerül jelentésre, nem pedig befejezett futtatásként.
- **Amit az asszisztens lát.** Az SDK figyelmeztetései az `warnings` alatt jelennek meg az eszköz eredményében, a hibák (kijelentkezés, terv, hibás elérési út) pedig olyan hibaüzenetként, amelyet az asszisztens elolvashat és továbbíthat Önnek. A 20 elemnél hosszabb lista az első 20 elemre rövidül, teljes hossza pedig az `<key>_total` kód alatt jelenik meg. Az elérési utaknak abszolútnak kell lenniük.
- **Az eszközök megosztottak.** A kamerák, képérzékelők és fényérzékelők a háttérben találhatók, így az asszisztens vezérelheti az Chloros-alkalmazáshoz csatlakoztatott eszközöket, és a változások az alkalmazásban is megjelennek. Az `disconnect_device` kiadja, hogy mihez csatlakozott a szerver; egy másik kliens (az alkalmazás, az `chloros-cli`, egy szkript, egy másik `chloros-mcp`) csak az `force: true` segítségével szabadul fel, amely az SDK `force_disconnect`-je, és a háttérrendszer, ha ismeri, megnevezi azt, aki csatlakoztatta; azt az eszközt, amelyet a háttérrendszer egyáltalán nem tart nyilván, nem csatlakozottként jelenti (`released: false`, `connected: false`, nincs mit felszabadítani). A háttérrendszer által futtatott felvétel vagy stream az `get_job_status` nézetét az `counts` — `frames_written`, `frames_dropped`, `frames_failed`, `drop_reason` (képkockacsoportok többkamerás stream esetén) — valamint egy egy soros `summary` („220 képkocka írt, 55 eldobva, 0 hiba: ...”) futás közben, majd befejezés után a végső számadatok; egy másik kliens listája erről (`other_clients_jobs`) ugyanazt tartalmazza (az `list_cameras()`, az `list_arrays()` és az `list_daq_sensors()` pedig az egyes eszközök`connected_by`: `client`, `label`, `name`, `at`). Minden `chloros-mcp` a háttérrendszer számára `chloros-mcp (pid N)` néven azonosítja magát (`chloros_sdk.connect.CLIENT_LABEL`, `X-Chloros-Client-Label` néven küldve; bármely SDK-parancsfájl beállíthatja). Egy másik kliens fényérzékelő-felvételének leállítása szintén az `force`-et igényli.
- **A hosszú feladat egy munkafolyamat.** `start_capture_run`, `start_array_recording`, `build_burst_video`, `start_light_sensor_recording`, `start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`, `run_project_recipe`, és bármely rögzítés, megoldás vagy indexelés esetén, ha az `background`-et kérik, azonnal adjon vissza egy `job_id`-et; ezt követi az `get_job_status`, majd az `stop_job` zárja le . A futás addig folytatódik, amíg le nem állítják, kivéve, ha megadja az `max_captures`, `duration_s` (vagy roham esetén az `max_frames`) parancsot; az `min_free_gb` nem indul el, ha a meghajtón kevés a szabad hely. Nincs korlátozás, hacsak nem állít be egyet. A háttérprogram által futtatott felvétel vagy stream (`start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`) megőrzi a háttérprogram feladatazonosítóját, így egy másik `chloros-mcp` — amely később indult — ezen az azonosítón keresztül megtalálja azt: az `get_job_status` követi azt (és felsorolja az ilyen feladatokat, mint az `other_clients_jobs`, az `GET /api/camera/cli-jobs`-től kezdve, mindegyikhez tartozik egy `started_by`), és az `stop_job` csak az `force: true` segítségével állítja le. Bármely más feladat (rögzítési futtatás, tömbfelvétel vagy burst, háttérrögzítés) az azt elindító `chloros-mcp`-ben fut, és csak az a szerver ismeri: egy másik, amelyik lekérdezte, ezt jelzi. A leállított feladat az `stopping`-et olvassa, miközben kiírja a sorát, majd az `finished`-et a végső számadatokkal, 10 percig, és bármelyik `chloros-mcp`-ből: az, amelyik elindította, ugyanúgy érzékeli egy másik kényszerleállását. Az `get_job_status` soha nem várja meg a leállást: egy leállás alatt (a saját `stop_job`-e, egy másik kliensévagy a magától befejeződött feladat kameráit felszabadító leállás) azonnal válaszol az `stopping`-szel és az élő számlálásokkal, és az `stop_job` egy már leálló feladat esetében ezt jelzi. A befejeződött feladat az `stop_reason` kódban jelzi a leállás okát: saját okát (`no_frames`, `disk_full`, `member_missing`, vagy tömbös folyamatos futás esetén `count`, `duration`, `array_gone`), az `limit reached` csak akkor, ha az értékei azt mutatják, hogy az `count` (vagy egy sorozat `max_frames`, egy felvétel`duration_s`) elérése esetén, `error`, `stopped` (az `stopped_by`-szel), ha egy kliens leállította, egyébként az `finished`.
- **Hová kerülnek a rögzítések.** A megnyitott projekt `captures` mappájába, ahová az alkalmazás menti őket, hacsak nem adjuk meg az `output_dir` értéket.
- **Képek.** Az `get_camera_preview` és az `grab_frames` az JSON mellett olyan képeket is visszaad, amelyeket az asszisztens megtekinthet. Az `grab_frames` képek kontrasztja a megtekintés érdekében ki van emelve; a statisztikák a valós értékeket tükrözik.
- **Amikor a szerver leáll**, az általa elindított feladatok leállnak, és az általa elindított fényérzékelő-felvétel lezárul. Ha a szerver elindította a háttérprogramot, akkor a háttérprogram leállítása előtt felszabadítja a csatlakoztatott eszközöket is; az alkalmazás háttérprogramjában azok továbbra is csatlakoztatva maradnak.
- **Protokoll.** Csak stdio (nincs HTTP-végpont), csak eszközök (nincsenek erőforrások vagy felugró ablakok), szöveges és képi eredmények, az MCP-protokoll 2024.11.05–2025.11.25 közötti verziói.**Ami kimaradt.** Be- és kijelentkezés (a fiók a tiéd; a szerver nem tud bejelentkezni), valamint olyan háttérprogram leállítása, amelyet nem a szerver indított el. Minden egyéb, amit ez az SDK a feldolgozással és a hardverrel kapcsolatban végez, rendelkezik egy eszközzel. (A DAQ-E beállítása – neve, hitelesítési tokenje, OTA-jelszava, aláírási követelménye, gyári visszaállítása – valamint a közvetlen DAQ-kiszolgálók kizárólag az `chloros-cli` feladata, az SDK-ben is: egy helytelen név vagy token minden Chloros-telepítést kizár az egységből.)

---

## A háttérprogramok automatikus indítása

A smart-connect belépési pontok — `connect_camera`, `connect_array`, `connect_daq_sensor`, `discover_lattice_cameras` és `discover_daq_sensors` — vékony HTTP-kliensek, amelyek feltételezik, hogy egy háttérrendszer figyel az `127.0.0.1:5000` (a smart-connect felület alapértelmezett URL-je) címet figyeli. Ha a grafikus felület vagy a parancssori felület (CLI) már fut, akkor az egyik biztosan fut. Egy egyszerű szkriptből indítva lehet, hogy nincs ilyen — ezért ezek a függvények **automatikusan elindítják a csomagban lévő háttérprogramot** (ablak nélkül, ugyanúgy, ahogyan az `ChlorosLocal` is teszi) az első hívásuk előtt, majd legfeljebb 60 másodpercig várnak, amíg az elindul. Az `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` és `analyze_array_network()` soha nem indít el ilyet.

Szabályok:

- **Kizárólag helyi URL-t indítunk el.** Az `backend_url`, amely az `localhost` / `127.0.0.1` / `[::1]`-re mutató `backend_url` megfelelő; bármely más gazdagépet másvalaki gépének tekintünk, és azt soha nem indítjuk el.
- **Ha egy háttérprogram a válaszadás előtt kilép, a várakozás véget ér.** A hívás azonnal visszatér a kilépési kóddal, a futtatott paranccsal és a háttérprogram által kinyomtatott utolsó sorokkal, ahelyett, hogy kivárná a 60 másodpercet. A 98-as kilépési kód kivételt képez: egy másik háttérprogram már birtokolja a portot vagy a zárolási fájlt (a Chloros-alkalmazásé, a háttérszolgáltatásé vagy egy másik, még indítás alatt álló szkripté), ezért a hívás továbbra is arra vár, hogy az válaszoljon. Az `ChlorosLocal` ugyanígy viselkedik.
- **A háttérprogram újrafelhasználás céljából futásban marad** (ugyanúgy, mint a CLI-ben) — a szkript kilépésekor nem történik implicit leállítás. A szkript újbóli futtatásakor a rendszer újra felhasználja az aktív háttérszolgáltatást.
- **Bármelyik ilyen hívásnál le lehet tiltani az `auto_start_backend=False` paranccsal** (pl. ha távoli háttérre mutat, vagy maga kezeli a háttér életciklusát).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Ha a csomagban szereplő bináris fájl nem található meg vagy nem indítható el, a következő HTTP-hívás egy kezelhető, **platformfüggő** `ChlorosConnectError` hibakódot dob, ahelyett, hogy egyszerűen csak a kapcsolat elutasítását jelentené — Windows-en az asztali alkalmazásra vagy egy `chloros-cli` parancsra irányít; Linuxon (GUI nélkül) egy `chloros-cli` parancsra vagy az `.deb`-re irányít.

---

## Környezet és fejlécek

Az SDK minden háttér HTTP-hívást `X-Chloros-Client: sdk`-szel jelöl. A háttér az SDK/CLI licencelési szabályait alkalmazza (bejelentkezés **és** fizetős Chloros+ csomag szükséges), ahelyett, hogy a GUI ingyenes szintjét használná. Ez az importáláskor automatikusan beállítódik — Önnek semmit sem kell tennie.

A fejléc minden `requests` / `urllib` híváshoz, amelynek URL-je tartalmazza az `localhost` vagy `127.0.0.1` karakterláncot, bármely porton – beleértve a saját helyi szolgáltatásait is — és felülírja az Ön által beállított értéket. Más gazdagépekhez intézett hívások változatlanok maradnak.

A háttér URL-jét felülírhatja az `backend_url=` átadásával (az `ChlorosLocal` az `backend_url=`-et, az `api_url=`; az `process_folder()` és az `process_lattice_capture()` csak az `api_url=` értéket fogadja el):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(A telepített háttérrendszer kizárólag a loopback-en hallgatózik, ezért egy LAN-on lévő `backend_url` nem érheti el; lásd [Távoli háttérrendszer mód](#remote-backend-mode-pip-only-host-via-tunnel) a tunnel mintáról.)

---

## Verziókezelés és kompatibilitás

- Az SDK-verzió `chloros_sdk.__version__` néven érhető el.
- Az SDK a viselkedést a csomagban szereplő háttérprogram-verzióhoz köti. Egy régebbi SDK és egy újabb háttérprogram együttes használata általában működik (előre kompatibilis végpontok), de egy újabb SDK és egy régebbi háttérprogram együttes használata `404` hibákat okozhat az új végpontokon — frissítse az asztali alkalmazást, hogy összhangba kerüljön velük.
- A smart-connect felület (`connect_camera` / `connect_array` / `connect_daq_sensor`) és a hálózatelemző végpont stabil JSON-sémákat ad vissza; az új mezők kiegészítő jellegűek.

---

## Hibaelhárítási tippek

- **`ChlorosAuthenticationError: Login required`** (az `ChlorosLocal`-ből; a Smart-Connect-hívások ugyanebből az okból `ChlorosConnectError: Login required. …` hibát jelentenek) → Futtassa egyszer az `chloros-cli login EMAIL PASSWORD` parancsot ezen a gépen, vagy jelentkezzen be a Chloros asztali alkalmazáson keresztül.
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`** (vagy `Failed to create project: …` az `create_project()`-ből) → A Windows hosszú elérési útvonalai ki vannak kapcsolva ezen a gépen (ez a Windows alapértelmezett beállítása), így egyetlen elérési út sem haladhatja meg a 260 karaktert, és a termékek legfeljebb körülbelül 130 karakterrel helyezkedhetnek el a projektmappa alatt. Semmi sem került feldolgozásra. Az üzenet megadja a leghosszabb, még elfogadható projektmappát: használjon rövidebbet, vagy kapcsolja be a Windows hosszú elérési útjait, majd indítsa újra a Chloros alkalmazást. A Windows-telepítő ezt „Windows hosszú elérési útvonalak engedélyezése” néven kínálja, amely alapértelmezés szerint be van jelölve; egyébként használja a Csoportházirendet: *Számítógép konfigurációja > Felügyeleti sablonok > Rendszer > Fájlrendszer > Win32 hosszú elérési útvonalak engedélyezése*.
- **`ChlorosConnectError: No Chloros backend is running …`** → A Smart-Connect automatikusan elindít egy helyi háttérprogramot, így ez a hibaüzenet csak akkor jelenik meg, ha a csomagban szereplő bináris fájl nem található meg vagy nem indítható el (pl. egy pip-only gazdagépen, asztali csomag nélkül). Az üzenet platformfüggő: Windows-en nyissa meg az asztali alkalmazást, vagy futtasson bármely `chloros-cli` parancsot; Linuxon futtasson egy `chloros-cli` parancsot (nincs grafikus felület), vagy telepítse az `.deb`-et. Távoli háttérprogram esetén adja meg az `backend_url=` (és az `auto_start_backend=False`) paramétert.
- **`CAMERA_AVAILABLE == False`** az importáláskor → Az `lattice_sdk` betöltése sikertelen: az Arena SDK futásideje vagy az OpenCV (`cv2`) nem importálható ebben a Python-verzióban. A háttérrendszer-vezérelt felület (`connect_*`, háttérrendszeren keresztül vetít, `ChlorosLocal`) továbbra is működik.
- **A tömb teljes szenzorral csatlakozik, de alacsony a képkockasebesség**→ Tervezésből adódóan: az automatikus üzemmód megtartja a teljes szenzort, és hagyja, hogy a sebesség a vezeték által bírható szintre csökkenjen. Adja át az `binning=2` / `4` vagy egy kisebb `width` / `height` értéket a másodpercenkénti képkockák számának növeléséhez, vagy engedélyezze a jumbo kereteket / frissítse a kapcsolatot. A binning és az ROI**nem** oldja meg az összesített túljelentkezés problémáját (`oversubscribed: true`, fps mezők 0): a vezetékre vonatkozó túl sok kamera problémáját nem lehet binninggel/ROI-val megoldani — csökkentse a kamerák számát, engedélyezze a jumbo kereteket, vagy váltson gyorsabb hálózati kártyára (lásd [Túlterhelés](#over-subscription-the-per-cam-floor)).
- **Az `analyze_array_network()` jelzi, hogy a hálózati kártya RX gyűrűje apró (~0,26 MB) / a csatlakozási kapukon „FRAMES WILL DROP”** felirat látható → A gazdagép hálózati kártyájának vételi gyűrűje az alapértelmezett értéken van (gyakran 32-re áll vissza a hálózati kártya illesztőprogram frissítése után). Realtek USB 10GbE adapter esetén állítsa be az `ReceiveBufferLen=256` és az `PendingReceives=64` (emelt jogosultságú) beállításokat, majd indítsa újra a háttérszolgáltatást, hogy újraolvassa a gyűrűt. A teljes eljárás: [CLI-referencia → Gazdagép hálózati kártyájának beállítása és hangolása](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **A gazdagép lefagy újraindításkor/leállításkor, később WMI `Invalid class` hibák jelentkeznek / a hálózati kártya nem aktiválódik** → Elavult USB 10GbE illesztőprogram okozza az `DRIVER_POWER_STATE_FAILURE` hibát (BSOD `0x9F`). Frissítse az adapter illesztőprogramját a legújabb verzióra (≥ 2026), majd állítsa be újra a fogadógyűrű beállításait. Lásd [CLI-referencia → Gazdagép NIC beállítása és hangolása](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **A tömb rögzítése sikertelen, HTTP-503 „Nincs frissen szinkronizált keretcsoport” hibaüzenettel** → A tömb a várakozási idő alatt nem tett közzé új keretcsoportot. Ellenőrizze az `arr.status()`: `stalled: True` (vagy az `streaming: False`) dokumentumot; az `actual_fps` 0 értéke azt jelenti, hogy a rögzítési ciklus leállította a közzétételt — válassza le, majd csatlakoztassa újra a tömböt. Ha az `stalled` értéke False, és az `actual_fps` értéke 0 felett van, próbálja meg újra. Közvetlenül az `connect_array()` után egy rögzítés már legfeljebb 20 másodpercet vár az első csoportra.
- **Az `status()` az `link_speed_mbps: 4294` értéket jelzi** → A Windows nem tudta kiolvasni az adapter sebességét; ez jellemző az USB-Ethernet-adapterekre, és nem hiba. Lásd [Várható keretfrekvenciák](#expected-frame-rates).
- **Az `update_firmware()` hibaüzenetet ad az `device never connected back on TCP <port>` miatt** → Egy tűzfal blokkolta a DAQ-E visszacsatolását erre a számítógépre. A Windows-telepítő hozzáadja a szabályt (lásd [DAQ-E eszközbeállítások](#daq-e-device-settings)); más tűzfal esetén engedélyezze a bejövő TCP-forgalmat a háttérrendszer felé az érzékelő alhálózatából.
- **A reflexancia elutasítva** → Az abszolút skálájú reflektancia méréséhez egy aktív DAQ-t kell a kamerához (vagy sorozathoz) társítani. Vagy a grafikus felhasználói felületen keresztül társítsa, vagy használja az `processing="radiance"` (W/m²/sr/nm) parancsot, amelyhez nincs szükség párosított érzékelőre.
- **Az `smart=True` rögzítése a vártnál hosszabb ideig tart** → Az AE konvergencia a jelenet dinamikájától függ. Az `ArrayHandle.capture_smart()` használatával szélesítse az `exposure_tolerance_pct`-et, rövidítse az `stability_window_s` értéket, vagy csökkentse az `settle_timeout_s` értéket a gyorsabb, kevésbé stabil kioldás érdekében.

---

## Lásd még

- [CLI-referencia](cli-reference.md) — minden CLI-alparancs egy SDK-hívást tükröz.
- [DAQ fényérzékelők](https://mapir.gitbook.io/chloros/daq-light-sensors) a Chloros kézikönyvben — az érzékelők, azok határai és kalibrált tartománya, rögzítés, DAQ-E hálózat.
- [LATTICE kamerák](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) a Chloros kézikönyvben — csatlakoztatás, kamera beállítások, rögzítési módok, tömbök.
- Online dokumentáció: `https://mapir.gitbook.io/chloros/api-python-sdk`
