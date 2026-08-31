# Az Chloros használata mesterséges intelligencia-asszisztensekkel

Ez a kézikönyv két célközönségnek szól: az embereknek, valamint azoknak a mesterséges intelligencia-asszisztenseknek, amelyekkel az emberek egyre gyakrabban dolgoznak együtt. Minden oldalon pontos értékek, alapértelmezett beállítások és másolható-beilleszthető parancsok találhatók, így egy asszisztens (Claude, ChatGPT, Copilot, programozási ügynök stb.) már az első próbálkozásnál működőképes Chloros-automatizálást tud írni.

Chloros verzió: **

1.2.0**. CLI/SDK platformok: Windows 10/11 x64 és Linux (x86_64 / Jetson aarch64).

## Mit adjon át az asszisztensének

| Erőforrás | URL | Mire szolgál |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | A kézikönyv minden oldalának géppel olvasható indexe. |
| **CLI Referencia** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | A teljes `chloros-cli` parancsszerkezet: minden parancs, jelző, alapértelmezett érték, kilépési kód és kimeneti mappa szabály. LLM-ek számára írt. |
| **SDK Referencia** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | A teljes `chloros_sdk` Python API: osztályok, aláírások, kivételek és gyakorlati példák. LLM-felhasználók számára írva. |
| **Bármely oldal nyers Markdown formátumban** | illessze be az `.md` kódot az URL oldalhoz | pl. az `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` az oldalt nyers Markdown formátumban adja vissza — ideális kontextusablakba való beillesztéshez vagy ügynökből való lekéréshez. |

Kézikönyvben szereplő linkek: [CLI Hivatkozás](reference/cli-reference.md) · [SDK Hivatkozás](reference/sdk-reference.md).

{% hint style="info" %}
A két referenciaoldal önálló: egy asszisztens, aki az egyiket elolvasta, nem szorul a kézikönyv többi részére ahhoz, hogy helyes szkriptet írjon.
{% endhint %}

## Parancssori receptek

Másolja ki az `<placeholders>` tartalmát, töltse ki, majd illessze be az asszisztensébe.

### 1. Repülési mappa feldolgozása az NDVI fájlba

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
A parancssorból indított DAQ-parancsfájlok mindig az `daq pool-*` családon keresztül futnak (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`) családon keresztül történik. Az asszisztensed által esetleg kitalált egyéb `daq` alparancsok nem érhetők el a kiadott verziókban, és hibaüzenettel lépnek ki.
{% endhint %}

## Miért működnek jól az AI által írt szkriptek az Chloros-szel?

Mindezek az Chloros 1.2.0 verziójának valós, ellenőrzött viselkedései – kiküszöbölik a géppel írt automatizálás klasszikus hibaforrásait:

* **Nincs bonyolult beállítási procedúra.**Az SDK intelligens csatlakozási segédprogramjai (`connect_camera`, `connect_array`, `connect_daq_sensor`) és a feldolgozási belépési pontok (`ChlorosLocal`, `process_folder`)**automatikusan elindítják a helyi háttérprogramot**. A generált szkriptnek nincs szüksége a GUI megnyitására vagy egy manuálisan elindított szerverre — csupán a desktop/CLI csomag telepítésére van szüksége.
* **Az egész folyamat egyetlen hívásból áll.** Az `chloros_sdk.process_folder("path", indices=["NDVI"])` végigfut az importáláson → kalibráláson → reflektancián → index exporton. Kevesebb a felület, kevesebb a hely, ahol a generált szkript hibásodhat.
* **A kimenet nélküli futtatások önellenőrzést végeznek.** Az `process()` után a futtatás összefoglalója csatolva van az eredményhez, és minden feldolgozási utalás (pl. *miért* nem hozott kimenetet a futtatás) szintén újraküldésre kerül egy Python `UserWarning` formájában — így még egy olyan szkript is megjeleníti a diagnózist, amely soha nem vizsgálja az eredmény-diktumot.
* **Az CLI futtatás hangosan meghiúsul.**Egy olyan `chloros-cli process` futtatás, amely kimenetet kért, de nem írt ki semmit, az `Processing finished but wrote no image products.` kódot írja ki, és**nem nulla kóddal lép ki**, így a shell szkriptek és a CI egyszerű kilépési kód-ellenőrzéssel észlelik. A sikeres futtatások az `Image products written: N` kódot jelentik.

Egy aszimmetria, amit egy asszisztensnek tudnia kell: az SDK `process()` kódja szándékosan **nem** dob kivételt nulla termékes futás esetén — ehelyett az összefoglalóban/tippekben jelenti. Ha egy Python folyamatnak le kell állnia egy üres futás esetén, ellenőrizze az összefoglalót (a 2. recept ezt teszi).

## Figyelmeztetések

* **Chloros+ bejelentkezés szükséges.**Az CLI és az SDK**fizetős** Chloros+ szintet igényelnek, amelyet szerveroldalon érvényesítenek: a kérések `401 AUTH_REQUIRED` hibával buknak meg, ha nincs bejelentkezés, és `403 PLAN_UPGRADE_REQUIRED` hibával, ha ingyenes szinten futnak. Futtassa az `chloros-cli login` parancsot gépenként egyszer, mielőtt a generált szkripteket futtatná. Lásd: [Chloros+ Bejelentkezés](chloros+-login.md).
* **A rögzítési parancsok valódi hardvert vezérelnek.** Az `lattice` / `daq` / `project` parancsok és az SDK munkamenetobjektumok csatlakoznak a fizikai kamerákhoz és érzékelőkhöz, adatfolyamot továbbítanak és azokat működtetik. Az első futtatás előtt ellenőrizze a generált szkriptet, és futtassa azt úgy, hogy a hardvert személyesen felügyeli.
* **Végezzen szúrópróba-ellenőrzést a kimeneteken.** Az eredmények közzététele előtt ellenőrizze a termékmappákat és néhány képpontértéket. Különösen a reflexiós TIFF-fájlok mérete forrásonként változik – olvassa el az `Chloros:PixelScale` XMP-címkét (LATTICE: 32768 = 1,0 reflektancia; Survey3: 65535) helyett, ahelyett, hogy osztót feltételezne. Mindkét referenciaoldal ezt a „Reflektancia-pixelek olvasása” cím alatt dokumentálja.
* **Apró buktatók, amelyek megakadályozzák a generált kód működését:**az `pool-record` a**háttérgép** fájlrendszerébe ír (alapértelmezés szerint `~/Documents/DAQ Live View/`); több hálózati interfésszel rendelkező gépeken az automatikus felismerés helyett inkább az `daq pool-connect --eth-host <ip-or-hostname>`-et válasszuk; és használja az `http://127.0.0.1:5000`-et (soha ne az `localhost`-et) mindenhol, ahol egy háttér URL jelenik meg.
