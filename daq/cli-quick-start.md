# CLI Gyors útmutató (pool-*)

A szállított `chloros-cli` vezérli a DAQ-érzékelőket a **`daq pool-*`** parancscsaládon keresztül vezérli a DAQ-érzékelőket — ezek vékony HTTP kliensek, amelyek az Chloros háttérprogram állandó érzékelő-poolján keresztül működtetik az érzékelőket. A háttérrendszer kezeli az adatátvitelt, így a grafikus felület, az CLI és az SDK szkriptek mind egy élő azonosítót használnak, ahelyett, hogy a portért versengnének. Minden, amire az ügyfélnek szüksége van, elérhető az `pool-*`-en keresztül: csatlakozás, adatfolyam, kalibrált `.daq` fájlok rögzítése és kap-profilok cseréje.

Az `pool-*` egyben a **kizárólagos** DAQ-felület a kiadott verziókban. Az `chloros-cli daq --help` felsorolja az `pool-*` alparancsokat, és ha egy kiadott verzióban közvetlen hardveres DAQ-alparancsot hív meg, a program egy kifejezett hibajelentéssel lép ki, amely megnevezi a hiányzó csomagot, és visszavezet az `pool-*`-hez — semmi sem bukik meg csendben. (A közvetlen hardveres parancsok csak egy MAPIR forráskód-kivonatból futtathatók; az `pip install chloros-sdk` sem biztosítja őket.)

***

## Előfeltételek

* **Az Chloros háttérprogramnak futnia kell** — az `pool-*` parancsok az HTTP kliensek, nem pedig hardver-illesztőprogramok. Az Windows rendszeren indítsa el az Chloros asztali alkalmazást (ez elindítja a háttérszolgáltatást). Headless Linux/Jetson rendszeren engedélyezze az `sudo systemctl enable --now chloros-backend.service` szolgáltatást.
* **Chloros+ (fizetős szint) bejelentkezés**: először futtassa az `chloros-cli login`-et. A végrehajtás szerveroldali — bejelentkezés nélkül a parancsok hibaüzenettel buknak meg az `401 AUTH_REQUIRED` esetén; az ingyenes (Iron) csomagban az `403 PLAN_UPGRADE_REQUIRED` esetén buknak meg.
* A parancsok alapértelmezés szerint az `http://127.0.0.1:5000`-et célozzák meg; az `daq pool-*` család figyelembe veszi az `CHLOROS_BACKEND_URL` környezeti változót, ha a háttérrendszer máshol fut.

***

## Ötperces munkamenet

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — érzékelő megnyitása a poolban

| Változat | Jelentés |
| --- | --- |
| `daq pool-connect` | Intelligens felismerés: bármely DAQ-ot megtalál ezen a gépen. |
| `daq pool-connect --port PORT` | DAQ-U egy adott soros porton (pl. `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M BLE-n keresztül, automatikusan beolvasott MAC-címmel. |
| `daq pool-connect --mac MAC` | DAQ-M egy ismert BLE MAC-címen (ez magában foglalja az `--ble`-et). |
| `daq pool-connect --eth-host HOST` | DAQ-E ismert gazdagépnévvel vagy IP-címmel — **a megbízható útvonal**. |
| `daq pool-connect --eth` | DAQ-E automatikus felderítéssel (mDNS, ARP-visszaeséssel). Lásd az alábbi figyelmeztetést. |

Beállítási zászlók, mindegyik opcionális:

| Zászló | Jelentés |
| --- | --- |
| `--integration-time MS` / `-t MS` | Kézi integrációs idő milliszekundumokban. |
| `--frame-avg N` / `-f N` | A jelentett spektrumonként átlagolt képkockák száma. |
| `--no-ae` | Az automatikus expozíció letiltása (az AE alapértelmezés szerint be van kapcsolva). |
| `--no-stream` | Csatlakozás a stream elindítása nélkül (később folytatható az `pool-stream --start` paranccsal). |
| `--cap-id CAP` | Cap-korrekciós profil; a háttérprogram alapértelmezett értéke `sunshine_cosine`. Lásd [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

{% hint style="warning" %}
**`--eth` automatikus felderítéssel kapcsolatos figyelmeztetés.** Több hálózati csatlakozással rendelkező gazdagépen (több aktív hálózati interfész esetén) a rendszerindítás után az *első* `pool-connect --eth` parancs eredménye üres lehet, még akkor is, ha az érzékelő megfelelően működik — az észlelési keresés elmulaszthatja az érzékelő interfészét, amíg az ARP-gyorsítótár üres. Ha az `--eth` nem talál semmit, próbálja meg újra, vagy hagyja ki teljesen a felderítést az `--eth-host <ip-or-hostname>` paranccsal, ami a több hálózati csatlakozással rendelkező gépeken a megbízhatóbb megoldás. A DAQ-E gazdagépneve `daq-e-<id>.local` (pl. `daq-e-def330.local`); a sima IP-címe is működik.
{% endhint %}

## `pool-list` — a csatlakoztatott eszközök megtekintése

Megjeleníti a háttérpoolban található összes érzékelőt, beleértve az `sensor_id` parancsot is, amelyre minden más parancsnak szüksége van:

| Modell | `sensor_id` formátum | Példa |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-oktett, kötőjeles | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` — spektrumkeretek olvasása

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Visszaadja a legfrissebb képkockát, vagy a legfrissebb `--recent N` képkockákat; az `--json` gépolvasható kimenetet ad a szkripteléshez. A keretek radiometrikusan kalibrált spektrális besugárzási értékek (W/m²/nm) a 135 pontos, 340–1010 nm-es rácson, az érzékelő fedélprofiljának már alkalmazásával. A kvantitatív sugárzási értékekhez legalább 15 másodpercnyi képkockából számítsa ki az átlagot — ez a műszer jellemzője, nem pedig hiba.

## `pool-stream` — az adatfolyam szüneteltetése vagy folytatása

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — `.daq` fájl rögzítése

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Jelző | Alapértelmezett | Jelentés |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | A felvétel hossza másodpercben; az `0` azt jelenti, hogy a folyamat addig fut, amíg ki nem adja az `--stop` parancsot. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Kimeneti könyvtár, amely **a háttérprogramot futtató gépen** kerül feloldásra. |
| `--device-name NAME` | — | A felvételhez tárolt címke. |
| `--stop` | — | A folyamatban lévő felvétel leállítása. |

{% hint style="info" %}
A felvétel a háttérrendszerben történik, így az `.daq` fájl a **háttérprogramot futtató gép** fájlrendszerébe kerül — alapértelmezés szerint az `~/Documents/DAQ Live View/` mappába, nem feltétlenül oda, ahol az CLI programot futtatta. A fájlnevek tartalmazzák az érzékelő azonosítóját és egy időbélyeget.
{% endhint %}

## `pool-set-cap` — a felszerelt kupak megadása

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

A kupak azonosítója kiválasztja a gyárilag mért korrekciós profilt, amelyet minden spektrumra alkalmaznak, és **meg kell egyeznie az érzékelőre fizikailag felszerelt kupakkal** — sem az érzékelő, sem a szoftver nem képes önállóan felismerni a kupakot, és a kiválasztás minden `.daq` fájlba be van jegyezve. Az alapértelmezett érték mindenhol az `sunshine_cosine` (minden DAQ-hoz a Sunshine koszinusz-korrektor sapka van felszerelve, tervezés szerint ~12-szeres csillapítással — a be nem jelentett sapkacsere nagyjából ezzel a tényezővel torzítja a spektrumokat).

| `--cap-id` | Elérhető a |
| --- | --- |
| `sunshine_cosine` (alapértelmezett) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | csak DAQ-U |
| `none` | csak DAQ-E — lásd a megjegyzést |

Az érzékelő beállításain kívüli fedélazonosítót a csatlakozáskor egyértelmű hibaüzenettel elutasítja a rendszer. Az `none` (DAQ-E) azt jelenti, hogy a kupakot fizikailag eltávolították — a rendszer továbbra is a DAQ-E süllyesztett üvegdiffúzorára vonatkozó gyári geometriai profilt alkalmazza, így ez nem egy „no-op” művelet, és a kupak nélküli DAQ-E egy laboratóriumi konfiguráció, nem pedig támogatott terepi konfiguráció. (A fedél nélküli DAQ-U valóban fedél nélküli, és egyáltalán nem igényel korrekciós profilt; a DAQ-M-et a Sunshine fedéllel együtt használják.)

## `pool-disconnect` — érzékelők felszabadítása

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## Parancsok összefoglalása

| Parancs | Cél |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Érzékelő megnyitása a háttérpoolban. |
| `daq pool-list` | Az összes érzékelő megjelenítése a poolban az `sensor_id`-szel együtt. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | A legutóbbi N kalibrált spektrumképkocka. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Folytatja / szünetelteti az adatfolyamot. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | Elindítja / leállítja a `.daq` felvételt (háttér oldalon). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | A felső határ-korrekciós profil cseréje futásidő alatt. |
| `daq pool-disconnect --sensor-id ID [--all]` | Egy vagy az összes érzékelő felszabadítása. |

***

## Hibaelhárítás az első DAQ-E csatlakozáskor

1. A DAQ-E-nek nincs állapotjelző LED-je — ellenőrizze az áramellátást a kapcsoló vagy az injektor port PoE/link jelzőfényén keresztül, és a bekapcsolás után várjon néhány másodpercet, amíg a készülék elindul és csatlakozik a hálózathoz.
2. A háttérgépnek a **szenzorral azonos broadcast-tartományban** kell lennie — az mDNS nem halad át útválasztókon.
3. Az Windows esetében az első indításkor fogadja el a Defender tűzfal figyelmeztetését (mDNS UDP 5353, DAQ-E adatok UDP 5002, PTP UDP 319/320).
4. Még mindig nincs jel az `--eth`-től? Használja az `--eth-host`-et a készülék gazdagépnevével (`daq-e-<id>.local`) vagy IP-címével — ez a megbízhatóbb út, különösen több hálózati csatlakozással rendelkező gazdagépek esetén.

***{% hint style="info" %}**Tipp az AI-asszisztenseknek.** A kézikönyv minden oldala nyers Markdown formátumban érhető el – fűzze hozzá az `.md`-et az oldal kisbetűs URL slugjához (ez az oldal: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); a géppel olvasható index pedig `https://mapir.gitbook.io/chloros/llms.txt`. Az `chloros-cli daq` és minden más parancscsalád teljes, parancsszintű dokumentációjához töltse le az [CLI Referenciát](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); az Python elérési útja az [SDK Referencia](../reference/sdk-reference.md) dokumentumban az `chloros_sdk.connect_daq_sensor()`.
{% endhint %}
