# Gyors útmutató az CLI-hez (pool-*)

A szállított `chloros-cli` a **`daq pool-*`** parancscsaládon keresztül vezérli a DAQ-érzékelőket — ezek olyan egyszerű HTTP-kliensek, amelyek a Chloros háttérrendszer állandó érzékelő-poolján keresztül működtetik az érzékelőket. A háttérrendszer kezeli az adatátvitelt, így a grafikus felhasználói felület (GUI), a parancssori felület (CLI) és az SDK-parancsfájlok mind egy élő kezelőt használnak, ahelyett, hogy a portért versengnének. Minden, amire az ügyfélnek szüksége van, elérhető az `pool-*` parancs segítségével: csatlakozás, adatfolyam, kalibrált `.daq` fájlok rögzítése és cap-profilok cseréje.

Az `pool-*` a kiadott verziókban támogatott DAQ-felület. Az `chloros-cli daq --help` felsorolja az `pool-*` alparancsokat, az `daq discover` / `daq list` ugyanazt a vizsgálatot futtatja, mint az `pool-discover`. Ha egy közvetlen hardveres DAQ-alparancs nem áll rendelkezésre, a program egy kifejezett hibajelentéssel lép ki, amely megnevezi a hiányzó csomagot, és visszavezet az `pool-*` parancshoz — semmi sem bukik meg észrevétlenül. (Az `pip install chloros-sdk` sem biztosít közvetlen hardveres parancsokat.)

***

## Előfeltételek

* **A Chloros háttérprogramnak futnia kell** — az `pool-*` parancsok HTTP-kliensek, nem hardver-illesztőprogramok. Windows rendszeren indítsa el a Chloros asztali alkalmazást (ez elindítja a háttérprogramot). Linux/Jetson headless rendszeren engedélyezze a szolgáltatást a saját felhasználói fiókjával: `sudo systemctl enable --now chloros-backend@$USER.service` (lásd [Linux telepítés](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)).
* **Chloros+ (fizetős szint) bejelentkezés**: először futtassa az `chloros-cli login` parancsot. A végrehajtás szerveroldali — bejelentkezés nélkül a parancsok hibaüzenettel buknak meg az `401 AUTH_REQUIRED` esetén; az ingyenes (Iron) csomagban az `403 PLAN_UPGRADE_REQUIRED` esetén buknak meg.
* A parancsok alapértelmezés szerint az `http://127.0.0.1:5000`-et célozzák meg; az `daq pool-*` család figyelembe veszi az `CHLOROS_BACKEND_URL` környezeti változót, ha a háttérrendszer máshol fut.

***

## Ötperces munkamenet

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

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
| `daq pool-connect --mac MAC` | DAQ-M egy ismert BLE MAC-címen (ez magában foglalja az `--ble`-et is). |
| `daq pool-connect --eth-host HOST` | DAQ-E ismert gazdagépnévvel vagy IP-címmel — **a megbízható útvonal**. |
| `daq pool-connect --eth` | DAQ-E automatikus felderítéssel (mDNS, ARP-visszaeséssel). Lásd az alábbi figyelmeztetést. |

Beállítási jelzők, mind opcionális:

| Jelző | Jelentés |
| --- | --- |
| `--integration-time MS` / `-t MS` | Kézi integrációs idő milliszekundumokban. |
| `--frame-avg N` / `-f N` | A jelentett spektrumonként átlagolt képkockák száma. |
| `--no-ae` | Az automatikus expozíció letiltása (az AE alapértelmezés szerint be van kapcsolva). |
| `--no-stream` | Csatlakozás a stream elindítása nélkül (később folytatható az `pool-stream --start` paranccsal). |
| `--cap-id CAP` | Cap-korrekciós profil; a háttérprogram alapértelmezett értéke `sunshine_cosine`. Lásd [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

Ha egy olyan DAQ-E-t csatlakoztatunk, amely már szerepel a poolban egy másik címén – az `daq-e-<id>.local` gazdagépnevével vagy IP-címével –, akkor a rendszer a meglévő pool-bejegyzést adja vissza, ahelyett, hogy másodszor is megnyitná az egységet.

{% hint style="warning" %}
**`--eth` automatikus felismeréssel kapcsolatos figyelmeztetés.** Több hálózati kapcsolattal rendelkező gazdagépen (több mint egy aktív hálózati interfész) a rendszerindítás utáni *első* `pool-connect --eth` parancs eredménye üres lehet, még akkor is, ha az érzékelő rendben működik — az észlelési keresés elmulaszthatja az érzékelő interfészét, amíg az ARP-gyorsítótár üres. Ha az `--eth` nem talál semmit, próbálja meg újra, vagy hagyja ki teljesen a felderítést az `--eth-host <ip-or-hostname>` paranccsal, ami a több hálózati csatlakozással rendelkező gépeken a megbízhatóbb megoldás. A DAQ-E gazdagépneve `daq-e-<id>.local` (pl. `daq-e-def330.local`); a sima IP-címe is működik.
{% endhint %}

## `pool-discover` — vizsgálat csatlakozás nélkül

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

Megkeresi azokat az érzékelőket, amelyekhez csatlakozni lehet, és megjeleníti a csatlakozáshoz szükséges címet, anélkül, hogy csatlakozna. Az `--only` az `usb,ble,eth` vesszővel elválasztott részhalmazát veszi figyelembe; `--timeout SEC` beállítja az egyes átviteli módokhoz tartozó keresési időt (alapértelmezés szerint 5 másodperc, 1–20 másodperc közötti tartományban). A poolban már megnyitott érzékelők nem szerepelnek a listában (a csatlakoztatott DAQ-M leállítja a hirdetést) — ezekhez használja az `pool-list` parancsot. A kiadott verzióban az `chloros-cli daq discover` és az `daq list` ugyanazt a vizsgálatot futtatja.

## `pool-list` — a csatlakoztatott eszközök megtekintése

Megjeleníti a háttérpoolban található összes érzékelőt, beleértve az `sensor_id`-et is, amelyre minden más parancsnak szüksége van:

| Modell | `sensor_id` alak | Példa |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-oktett, kötőjeles | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Egy olyan DAQ-E, amelynek PTP-je ki van kapcsolva (a beállítás a készüléken van elmentve), az `pool-list`-ben az `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.` jelöléssel van megjelölve, valamint a visszakapcsolásra szolgáló parancs is szerepel – lásd [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on).

## `pool-latest` — spektrumkeretek olvasása

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Visszaadja a legfrissebb keretet, vagy a legfrissebb `--recent N` kereteket; az `--json` géppel olvasható kimenetet ad a szkripteléshez. A keretek radiometrikusan kalibrált spektrális besugárzási intenzitást (W/m²/nm) jelentenek a 135 pontból álló, 340–1010 nm-es rácson, az érzékelő fedélprofiljának már alkalmazásával. A kvantitatív besugárzási értékekhez legalább 15 másodpercnyi képkockát átlagoljon – ez a műszer jellemzője, nem pedig hiba.

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
| `--duration SEC` / `-d SEC` | `0` | A felvétel hossza másodpercben; az `0` azt jelenti, hogy a program addig fut, amíg ki nem adja az `--stop` parancsot. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Kimeneti könyvtár. Ha a háttérprogram ezen a számítógépen fut, a relatív elérési út a parancs futtatásának helye szerinti mappához viszonyítva kerül feloldásra; távoli háttérprogram esetén az elérési út **a háttérprogramot futtató gépen** kerül feloldásra. |
| `--device-name NAME` | — | A felvételhez tárolt címke. |
| `--stop` | — | A folyamatban lévő felvétel leállítása. |
| `--confirm-after SEC` | `20` | Az indítás után ennyi ideig várjon az első sor írására, és hangosan figyelmeztessen (`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`), ha nem érkezik be. Az `0` letiltja az ellenőrzést. |
| `--heartbeat SEC` | `30` | Az `--duration` paranccsal SEC másodpercenként nyomtassa ki a sorok számát, és figyelmeztessen, ha a szám növekedése leáll. Az `0` letiltja ezt. |

{% hint style="info" %}
A rögzítés a háttérben történik, így az `.daq` fájl a **háttérgép** fájlrendszerébe kerül — alapértelmezés szerint az `~/Documents/DAQ Live View/` mappába, nem feltétlenül oda, ahol a CLI-t futtatta. A fájlnevek tartalmazzák az érzékelő azonosítóját és egy időbélyeget.
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

Az érzékelő készletén kívüli fedélazonosítót a csatlakozáskor egyértelmű hibakóddal elutasítja a rendszer. Az `none` (DAQ-E) azt jelenti, hogy a kupakot fizikailag eltávolították — a DAQ-E süllyesztett üvegdiffúzorára továbbra is a gyári geometriai profilt alkalmazza, így nem tekinthető működésképtelennek, és a kupak nélküli DAQ-E egy laboratóriumi konfiguráció, nem pedig támogatott terepi konfiguráció. (A fedél nélküli DAQ-U valóban fedél nélküli, és egyáltalán nem igényel korrekciós profilt; a DAQ-M-et a Sunshine fedéllel együtt használják.)

## `pool-disconnect` — érzékelők felszabadítása

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

Az `pool-disconnect` felszabadítja az érzékelőt, függetlenül attól, hogy ki nyitotta meg, így felszabadítja azt az érzékelőt is, amelyet egy összeomlott szkript hagyott a poolban (először befejeződik a rajta futó felvétel). A poolban nem szereplő érzékelőt nem lehet felszabadítani: a parancs kiírja az `<sensor-id>: already released (not in the pool)` üzenetet, majd kilép (`0`). Ha a háttérrendszer elutasította vagy nem erősítette meg a felszabadítást, a program kilép (`1`).

## `pool-enable-ptp` — a DAQ-E PTP-jének újra bekapcsolása

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

A DAQ-E a PTP be-/kikapcsolási beállítását a saját memóriájában tárolja, így egy olyan egység, amelynek PTP-je ki volt kapcsolva, kikapcsolt állapotban marad — és időbélyegei nem PTP-szinkronizáltak — mindaddig, amíg újra be nem kapcsolják. Az `pool-list` jelzi az ilyen egységeket, és kinyomtatja számukra ezt a parancsot. Az Chloros soha nem kapcsolja vissza önállóan az egységeket, mivel a kikapcsolt állapot szándékos is lehet. Újdonság a 1.3.0-s verzióban.

## DAQ-E eszközbeállítások

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info` megjeleníti az érzékelő firmware-jét (és azt, hogy ez a Chloros-kiadás tartalmaz-e újabbat), a címét és annak megszerzésének módját, a mentett hálózati beállításokat, amennyiben azok eltérnek a futó beállításoktól, valamint a bekapcsoláskori adatfolyamot és a PTP-t. Az `pool-set-network` újraindítja az érzékelőt a változtatások alkalmazásához, és a korábbi beállításokkal csatlakoztatja újra; az `--no-reboot` csak elmenti azokat. A statikus címnek a számítógép hálózatán belül kell lennie, különben a Chloros elutasítja; az `--force` felülírja ezt, de csak olyan számítógép esetében, amelyet átcímkéznek vagy oda irányítanak. Az újraindítások nem engedélyezettek, amíg az érzékelő adatokat rögzít. Az `pool-update-firmware` a parancssorban a grafikus felhasználói felület **Update to \<version\>** parancsának felel meg, és ez a módszer a DAQ-E frissítésére Linux rendszeren, ahol nincs grafikus felhasználói felület. Újdonság a 1.3.0-s verzióban.

***

## Parancsok összefoglalása

| Parancs | Cél |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Érzékelő megnyitása a háttérpoolban. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Érzékelők keresése csatlakozás nélkül (megkeresi a DAQ-M BLE MAC-címét). |
| `daq pool-list` | Az összes érzékelő megjelenítése a poolban az `sensor_id`-szel együtt. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | A legutóbbi N kalibrált spektrumkeret. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Folytatja / szünetelteti az adatfolyamot. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | Elindítja / leállítja egy `.daq` felvételt (háttéroldalon). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | A felső határ-korrekciós profil cseréje futásidő alatt. |
| `daq pool-disconnect --sensor-id ID [--all]` | Egy vagy az összes érzékelő felszabadítása. |
| `daq pool-enable-ptp --sensor-id ID` | A PTP kikapcsolt DAQ-E esetében a PTP újra bekapcsolása. |
| `daq pool-info --sensor-id ID [--json]` | A DAQ-E firmware-je, címe, mentett hálózati beállításai, bekapcsoláskori adatfolyam és PTP. |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP vagy statikus cím; újraindítja és újra csatlakoztatja az érzékelőt. |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | Adatfolyam bekapcsoláskor. |
| `daq pool-reboot --sensor-id ID` | A DAQ-E újraindítása és újracsatlakoztatása. |
| `daq pool-update-firmware --sensor-id ID` | A jelen kiadás DAQ-E firmware-jének feltöltése egy régebbi érzékelőre. |

***

## Hibaelhárítás az első DAQ-E-csatlakozáskor

1. A DAQ-E-nek nincs állapotjelző LED-je — ellenőrizze az áramellátást a kapcsoló vagy az injektor port PoE/link jelzőfényén keresztül, és a bekapcsolás után várjon néhány másodpercet, amíg a készülék elindul és csatlakozik a hálózathoz.
2. A háttérgépnek az érzékelővel **azonos broadcast-tartományban** kell lennie – az mDNS nem halad át a routereken.
3. A tűzfalnak engedélyeznie kell az mDNS-t (UDP 5353), a DAQ-E adatokat (UDP 5002) és a PTP-t (UDP 319/320). A Windows-telepítő ehhez hozzáad egy szabályt, a Linux `.deb` pedig megnyitja őket egy `ufw` / `firewalld` fájlban, amely a telepítéskor aktív volt; egy harmadik féltől származó tűzfal, vagy egy később engedélyezett tűzfal esetében ezeket engedélyezni kell.
4. Még mindig nincs válasz az `--eth`-től? Használja az `--eth-host` parancsot a készülék gazdagépnevével (`daq-e-<id>.local`) vagy IP-címével, ami kihagyja a felderítést.

***
{% hint style="info" %}
**Tipp az AI-asszisztensek számára.** A kézikönyv minden oldala nyers Markdown formátumban érhető el — illessze hozzá az `.md` parancsot az oldal kisbetűs URL-címéhez (ez az oldal: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); a géppel olvasható index pedig `https://mapir.gitbook.io/chloros/llms.txt`. Az `chloros-cli daq` és minden más parancscsalád teljes, parancsjel-szintű dokumentációjához töltse le a [CLI-referenciát](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); a Python-útvonal az [SDK-referenciában](../reference/sdk-reference.md) található: `chloros_sdk.connect_daq_sensor()`.
{% endhint %}
