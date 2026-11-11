# A Chloros használata mesterséges intelligencia-asszisztensekkel

Ez a kézikönyv két célközönségnek szól: az embereknek és azoknak az AI-asszisztenseknek, amelyekkel az emberek egyre gyakrabban dolgoznak együtt. Minden oldalon pontos értékek, alapértelmezett beállítások és másolható-beilleszthető parancsok találhatók, így egy asszisztens (Claude, ChatGPT, Copilot, kódoló ügynök stb.) már az első próbálkozásra működő Chloros-automatizálást tud írni.

Chloros-verzió: **

1.3.0**. CLI/SDK-platformok: Windows 10/11 x64 és Linux (x86_64 / Jetson aarch64).

## Mit adjon át az asszisztensének

| Erőforrás | URL | Mire szolgál |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | A kézikönyv minden oldalának géppel olvasható indexe. |
| **CLI-referencia** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | A teljes `chloros-cli` parancssori felület: minden parancs, jelző, alapértelmezett érték, kilépési kód és kimeneti mappa szabály. LLM-ek számára készült. |
| **SDK-referencia** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | A teljes `chloros_sdk` Python API: osztályok, aláírások, kivételek és gyakorlati példák. LLM-felhasználásra készült. |
| **Bármely oldal nyers Markdown formátumban** | illessze be az `.md` szót az oldal URL-jéhez | pl. az `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` az oldalt nyers Markdown formátumban adja vissza — ideális kontextusablakba való beillesztéshez vagy ügynökből való letöltéshez. |

A kézikönyvben található linkek: [CLI-referencia](reference/cli-reference.md) · [SDK-referencia](reference/sdk-reference.md).

{% hint style="info" %}
A két referenciaoldal önálló: egy asszisztens, aki elolvasta az egyiket, nem szorul a kézikönyv többi részére ahhoz, hogy helyes szkriptet írjon.
{% endhint %}

## Hagyja, hogy az asszisztens közvetlenül futtassa a Chloros-t: `chloros-mcp`

A fentiak mindegyike arra utasítja az asszisztenst, hogy *írjon* egy szkriptet vagy parancsot. Az `chloros-mcp` segítségével egy olyan asszisztens, amely a [Model Context Protocol](https://modelcontextprotocol.io) (MCP) nyelvet használja – például a Claude Desktop, a Claude Code vagy a Cursor –, közvetlenül a Chloros-t vezérli. Egyszerű nyelven adsz utasítást, és az asszisztens meghívja a szerver eszközeit:

* **Képek feldolgozása:** projekt létrehozása, a felvételeket tartalmazó mappa importálása, beállítások kiválasztása, feldolgozás, valamint a futtatás eredményeinek jelentése.
* **Hardver vezérlése:** LATTICE kamerák, szenzortömbök és DAQ fényérzékelők csatlakoztatása, előnézet, expozíció és szín beállítása, felvétel készítése, rögzítés és a firmware frissítése.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

A szerveres eszközök lefedik a [Python SDK](api-python-sdk.md) funkcióit. Az SDK-vel együtt települ, így általában nincs szükség további telepítésre: Windows rendszeren a Chloros telepítő az `chloros-mcp.exe` fájlt a Python `Scripts` mappájába helyezi, Linuxon pedig az `.deb` az `chloros-mcp` fájlt helyezi el az `/usr/bin` mappába.

**Claude kód:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**ezt add hozzá az `claude_desktop_config.json` fájlhoz (a**Beállítások &gt; Fejlesztő &gt; Konfiguráció szerkesztése**menüponttal nyitható meg), majd indítsd újra a Claude Desktopot. A**kurzor** ugyanazt a bejegyzést veszi át az `~/.cursor/mcp.json` fájlba.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Ha a kliens azt jelzi, hogy a szerver indítása sikertelen volt, akkor az `chloros-mcp` valószínűleg nem szerepel az általa látott `PATH`-ben: ehelyett adja meg a teljes elérési utat (`where chloros-mcp` Windows rendszeren, `which chloros-mcp` Linux rendszeren).

A Chloros-t ugyanazon a számítógépen kell telepíteni, és ott be kell jelentkezni, akár az alkalmazásban, akár az `chloros-cli login` segítségével, bármely fizetős Chloros+ csomagban (Copper vagy magasabb). A szerver nem tud helyetted bejelentkezni.

{% hint style="info" %}
Az egyes alkalmazások, beállítások, eszközök és a futtatások viselkedésének kliensbeállításai: [MCP-kiszolgáló (AI-asszisztensek)](reference/sdk-reference.md#mcp-server-ai-assistants) az SDK-referenciában.
{% endhint %}

## Parancssori sablonok

Másold ki, töltsd ki az `<placeholders>` mezőt, majd illeszd be az asszisztensedbe.

### 1. Repülési mappa feldolgozása NDVI-re

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Egy rögzítési könyvtár kötegelt figyelése

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. LATTICE-mátrix csatlakoztatása és rögzítés

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. DAQ fényérzékelő spektrumok rögzítése

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
A parancssorból történő DAQ-parancsfájlok futtatása mindig az `daq pool-*` családon keresztül történik (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`) családon keresztül történik. Az asszisztensed által esetleg kitalált egyéb `daq` alparancsok nem érhetők el a kiadott verziókban, és hibaüzenettel lépnek ki.
{% endhint %}

## Miért működnek jól az AI által írt szkriptek a Chloros-szal?

Mindezek a Chloros 1.3.0 valódi, ellenőrzött viselkedései — kiküszöbölik a géppel írt automatizálás klasszikus hibalehetőségeit:

* **Nincs bonyolult beállítási folyamat.**Az SDK intelligens csatlakozási segédprogramjai (`connect_camera`, `connect_array`, `connect_daq_sensor`) és a feldolgozási belépési pontok (`ChlorosLocal`, `process_folder`)**automatikusan elindítják a helyi háttérprogramot**. A generált szkriptnek nincs szüksége a GUI megnyitására vagy egy manuálisan elindított szerverre — csupán a desktop/CLI-csomag telepítésére van szüksége.
* **Az egész folyamat egyetlen hívásból áll.** Az `chloros_sdk.process_folder("path", indices=["NDVI"])` végigfuttatja az importálást → kalibrálást → reflektancia-mérést → index-exportálást. Kevesebb a felület, kevesebb a hely, ahol a generált szkript hibásodhat.
* **A kimenet nélküli futtatások önellenőrzést végeznek.** Az `process()` után a futtatás összefoglalója csatolva van az eredményhez, és minden feldolgozási utalás (pl. *miért* nem hozott kimenetet a futtatás) szintén újrakibocsátásra kerül Python `UserWarning` formájában — így még egy olyan szkript is megjeleníti a diagnózist, amely soha nem vizsgálja meg az eredmény dict-jét.
* **A parancssori felület (CLI) egyértelműen jelzi a hibát.**Egy olyan futtatás, amely eredményeket kért, de nem írt ki semmit, az `chloros-cli process` kódot írja ki, és**nem nulla kóddal lép ki**, így a shell szkriptek és a CI egy egyszerű kilépési kód-ellenőrzéssel is észlelik a hibát. A sikeres futtatások az `Image products written: N` kódot jelentik, valamint az `(plus M clamp masks)` kódot, ha maszkok kerültek írásra. A rögzítő parancsok hasonlóképpen megnevezik minden olyan fájlt, amelyet nem tudtak írni, és az `1` kóddal lépnek ki.
* **A relatív útvonalak azt jelentik, amit a szkript elvár.** Helyi háttérprogram esetén a relatív kimeneti útvonal (`-o out/` a CLI-ben, `output_dir="out/"` az SDK-ben) a parancs vagy szkript futtatási helyéül szolgáló mappához viszonyítva kerül feloldásra.

Egy aszimmetria, amit az asszisztensnek tudnia kell: az SDK `process()` hibája szándékosan **nem** lép fel nulla kimenetű futtatás esetén — ehelyett az összefoglalóban/tippekben jelzi a problémát. Ha egy Python-folyamatnak üres futás esetén le kell állnia, ellenőrizze az összefoglalót (a 2. recept ezt teszi).

## Figyelmeztetések

* **Chloros+ bejelentkezés szükséges.**A CLI-hez és az SDK-hez**fizetős** Chloros+ szint szükséges, amelyet szerveroldalon érvényesítenek: a bejelentkezés hiányában a kérések `401 AUTH_REQUIRED` hibakóddal, az ingyenes szinten pedig `403 PLAN_UPGRADE_REQUIRED` hibakóddal buknak meg. Futtassa az `chloros-cli login` parancsot egyszer minden gépen, mielőtt a generált szkripteket futtatná. Lásd: [Chloros+ bejelentkezés](chloros+-login.md).
* **A rögzítési parancsok valódi hardvert vezérelnek.** Az `lattice` / `daq` / `project` parancsok, az SDK-munkamenet-objektumok és az `chloros-mcp` eszközök csatlakoznak a fizikai kamerákhoz és érzékelőkhöz, adatfolyamot továbbítanak és azokat működtetik. Az első futtatás előtt ellenőrizze a generált szkriptet, és futtassa azt úgy, hogy a hardver mellett tartózkodik.
* **Végezzen szúrópróba-ellenőrzést a kimeneteken.** Az eredmények közzététele előtt ellenőrizze a termékmappákat és néhány képpontértéket. Különösen a reflexiós TIFF-fájlok mérete forrásonként változik — olvassa el az `Chloros:PixelScale` XMP-címkét (a LATTICE-fájlok tartalmazzák: 32768 = 1,0 reflexió; a Survey3-fájlok nem, és 65535 = 1,0-t használnak) ahelyett, hogy osztót feltételeznének. Mindkét referenciaoldal ezt a „Reflektancia-pixelek olvasása” cím alatt dokumentálja.
* **Apró buktatók, amelyek megakadályozzák a generált kód működését:**Az `pool-record` a**háttérgép** fájlrendszerébe ír (alapértelmezés szerint `~/Documents/DAQ Live View/`); több hálózati interfésszel rendelkező gépeken az automatikus felismerés helyett inkább az `daq pool-connect --eth-host <ip-or-hostname>`-et használja; az `http://127.0.0.1:5000`-et (soha ne az `localhost`-et) használja mindenhol, ahol háttérURL jelenik meg; egy olyan szkript, amely végigjárja a projekt kimeneti fa szerkezetét, hagyja ki az `clamp_masks/` mappákat (azok `*_clampmask.tif` fájljai maszkok, nem képek — lásd [Kimeneti képformátumok](output-image-formats.md#clamp-masks)); valamint Windows rendszeren, ahol a hosszú elérési útvonalak ki vannak kapcsolva, a projektmappákat rövidnek kell tartani – a feldolgozás megkezdése előtt a rendszer elutasítja az 130 karakternél hosszabb projektmappákat.
