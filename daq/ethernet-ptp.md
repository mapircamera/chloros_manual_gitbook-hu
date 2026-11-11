# DAQ-E hálózat és időszinkronizálás

> Az érzékelő fizikai hálózati beállítása – kábelezés, PoE, IP-cím hozzárendelés és az eszköz saját hálózati beállításai – a **[DAQ felhasználói kézikönyvben](https://mapir.gitbook.io/daq/daq-e/network-setup)** található. Ez az oldal az Chloros

-oldalt tárgyalja: a csatlakozást, az időszinkronizálást, valamint azt, hogy mit kell tenni, ha a felderítés nem jár eredménnyel.

A DAQ-E a DAQ-család Ethernet-alapú tagja: PoE-n keresztül kap áramellátást, mDNS-en keresztül (`_daq-e._tcp` szolgáltatás) ismerhető fel, és az érzékelő-azonosítójából származtatott gazdagépnévvel címezhető — `daq-e-<6 hex>.local`, pl. `daq-e-def330.local`. Ez az oldal bemutatja, hogyan továbbítja az adatokat a hálózaton, és hogyan vesz részt a PTP időszinkronizálásban.

## Átviteli módok

| Mód | Végpont | Felhasználók | Megjegyzések |
| --- | --- | --- | --- |
| **Multicast** (alapértelmezett) | UDP `239.10.10.10:5002` | Ugyanazon a LAN-on lévő tetszőleges számú eszköz ugyanazt az adatfolyamot veszi | Minden datagramot CRC-16/CCITT-vel ellenőriznek |
| **Raw** | TCP-port `5000` | Pontosan egy kliens (kizárólagos) | Bájtkompatibilis a DAQ-U-val |

A Chloros

alapértelmezés szerint multicastot használ, ami lehetővé teszi, hogy a grafikus felhasználói felület (GUI), a parancssori felület (CLI

) és az SDK

egyszerre figyeljenek egy érzékelőt.

## Hálózati követelmények

* **Ugyanaz a broadcast-tartomány.** A Chloros

-t futtató gépnek ugyanazon az L2 hálózati szegmensen kell lennie, mint az érzékelőnek — az mDNS-felfedezés nem halad át útválasztókon.
* **Tűzfal.** Windows

rendszeren a telepítő minden hálózati profilhoz hozzáadja a **Chloros

Backend (UDP-In)**tűzfalszabályt a háttérszolgáltatáshoz, amely kiterjed a DAQ-E adatokra (UDP 5002), az mDNS-t (UDP 5353) és a PTP-t (UDP 319/320), valamint egy**„Chloros

Backend (TCP-In)”** szabályt, amely a helyi alhálózatra korlátozódik, és a DAQ-E által a firmware-frissítés során a számítógéppel létesített visszacsatolási kapcsolatra vonatkozik. Linux

rendszeren az `.deb` megnyitja az UDP-portokat az `ufw` vagy az `firewalld` fájlban, ha a telepítéskor ez a tűzfal aktív. Egy harmadik féltől származó tűzfal, vagy egy később engedélyezett tűzfal esetében ugyanezeket a portokat kell engedélyezni.
* **PoE-tápellátás, nincs állapotjelző LED.** A DAQ-E-nek nincs saját LED-je — ellenőrizze a tápellátást a kapcsoló vagy az injektor portján található link/PoE-jelzőfény segítségével, és a bekapcsolás után várjon néhány másodpercet, amíg a készülék elindul és csatlakozik a hálózathoz.

## Csatlakoztatás

**GUI:** Fényérzékelők fül → Érzékelő csatlakoztatása → Eszköztípus „DAQ-E (Ethernet)”. A felderítés csak addig fut, amíg a csatlakozási párbeszédablak a képernyőn látható (mDNS-keresés és ARP-letapogatás Windows

rendszeren), 15 másodpercenként megismétlődve; a Frissítés gomb megnyomásával azonnal újraindul a keresés. Az észlelt érzékelők megjelennek a legördülő menüben; az első észlelt érzékelő automatikusan kijelölésre kerül.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (futó háttérprogram):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

A poolban már szereplő DAQ-E-t a gazdagépneve és az IP-címe alapján egyaránt felismeri a rendszer: ha a másik címen próbáljuk újra csatlakoztatni, a rendszer a meglévő pool-bejegyzést adja vissza, ahelyett, hogy másodszor is megnyitná az egységet.

### Több hálózati kártyával rendelkező gazdagépek és az első csatlakozás a rendszerindítás után

Több aktív hálózati interfésszel rendelkező gazdagépeken a rendszerindítás utáni **első** `pool-connect --eth` hibaüzenet üres lehet, még akkor is, ha az érzékelő rendben működik — az érzékelőkeresés elmulaszthatja azt az interfészt, amelyen az érzékelő található, amíg az ARP-gyorsítótár még nem töltődött be. A megbízható megoldás az, ha kihagyjuk a felderítést, és kifejezetten megadjuk a címet:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

Az `--eth-host` elfogadja az mDNS-gépnevet vagy az IP-címet, mindig a megfelelő érzékelőt célozza meg, és ez az ajánlott forma szkriptekhez és headless telepítésekhez. A grafikus felhasználói felületen használja a csatlakozási párbeszédablak Frissítés gombját, és várja meg az újrakeresési ciklust.

## Eszközbeállítások és firmware

Maga az érzékelő tárolja a hálózati beállításait: DHCP (gyári beállítás, amely 169.254.x.x link-local címre vált, ha nincs válasz a DHCP-kiszolgálótól) vagy statikus cím. Azt is tárolja, hogy bekapcsoláskor elindul-e az adatátvitel. Ezeket a beállításokat a grafikus felhasználói felületen (az érzékelő fogaskerék-ikonjával elérhető beállítások, **Hálózat**:**Módosítás…**,**Adatfolyam bekapcsoláskor**,**Érzékelő újraindítása**; lásd [A DAQ fül](gui.md)), vagy a CLI

-n és az SDK

-ben az `pool-set-network`, `pool-set-auto-stream` és `pool-reboot` (`DAQSensorSession.set_network()`, `set_auto_stream()`, `reboot()`; lásd a [CLI

-referenciát](../reference/cli-reference.md#daq-e-device-settings)). Az érzékelő újraindul a hálózati változás alkalmazása érdekében, és a Chloros

újra csatlakoztatja. A Chloros

ot futtató számítógép hálózatán statikus címnek kell lennie: a Chloros

elutasítja azokat, amelyekhez az újraindítás után nem tudott hozzáférni. Az érzékelő azonosítója és az `daq-e-<id>.local` gazdagépneve soha nem változik. Újdonság az 1.3.0-s verzióban.

**A firmware-frissítések beépültek a grafikus felhasználói felületbe.** Ha egy csatlakoztatott DAQ-E olyan firmware-t futtat, amely régebbi, mint a Chloros

-verzióhoz mellékelt kép, az érzékelő sorában sárga **Frissítés elérhető** ikon jelenik meg, és a fogaskerék-beállítások ablakban megjelenik egy „Frissítés a <version>” gomb. A frissítés körülbelül 30 másodperc alatt lezajlik a hálózaton keresztül; az érzékelő automatikusan újraindul és újra csatlakozik, és egy megszakadt átvitel esetén a jelenlegi firmware sértetlen marad. CLI-ről vagy SDK-ről, valamint Linux rendszeren, ahol nincs grafikus felület, futtassa az `chloros-cli daq pool-update-firmware --sensor-id <id>` (`DAQSensorSession.update_firmware()`) parancsot.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP időszinkronizálás

A DAQ-E v1.2.0+ firmware az IEEE 1588 PTPv2 szabványban rendes (kizárólag szolga) óraként vesz részt. **A Chloros-gazdagép háttérrendszere a PTP grandmaster** — minden DAQ-E és minden LATTICE kamera a LAN-on a 0. tartományban ehhez van rendelve, így az összes eszköz időbélyegét ~1 ms-os tűréshatáron belül tartja. Ez a közös óra teszi lehetővé, hogy a DAQ-leolvasások időbélyegei egybeessenek a kamera expozícióival (lásd [Felvétel és a .daq formátum](recording.md)).

Ellenőrizze a szinkronizálást a parancssorból (CLI):

| Parancs | Megjelenít |
| --- | --- |
| `chloros-cli time-sync status` | A gazdagép grandmaster állapota, a BMCA prioritások, az óra azonosítója, valamint `health`: `OK`, vagy `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT`, valamint egy `health_detail`, amely megmagyarázza, miért nem érnek el a csomagjai a hálózatot |
| `chloros-cli time-sync peers` | Minden észlelt szolga (DAQ-E érzékelők + LATTICE kamerák) |
| `chloros-cli time-sync cameras` | Kameránkénti PTP állapot (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | A grandmaster folyamat újraindítása |

A grafikus felhasználói felületen a DAQ-E beállítások ablakában egy élő **PTP Sync** sor jelenik meg, amely az érzékelő aktuális PTP állapotát mutatja.**A kikapcsolt PTP-vel rendelkező DAQ-E továbbra is kikapcsolt állapotban marad.** A készülék a saját memóriájában tárolja a PTP be-/kikapcsolási beállítását, így egy olyan készülék, amelynek PTP-je ki volt kapcsolva, újracsatlakozás vagy áramellátás-visszaállítás után is kikapcsolt állapotban marad, és időbélyegei nem PTP-szinkronizáltak. Az `chloros-cli daq pool-list` jelzi az ilyen egységeket (`!! PTP is switched OFF on this unit …`), és kinyomtatja a hiba kijavításához szükséges parancsot:

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
``` A

Chloros soha nem kapcsolja vissza önállóan az egységeket, mivel a kikapcsolt állapot szándékos is lehet.

Részletek a szigorú igazításra szoruló felhasználók számára:

* Minden streamelt adatgram tartalmaz egy flags mezőt; **a 2. bit be van állítva azoknál a kereteknél, amelyek időbélyege PTP-szinkronizált**. Azok a feldolgozófolyamatok, amelyek szigorú kamera/DAQ-igazítást igényelnek, ennek a bitnek a jelenlétét kell figyelniük.
* Szinkronizált rögzítés előtt ellenőrizze, hogy az érzékelő megjelenik-e az `chloros-cli time-sync peers`-ben, és hogy az `chloros-cli daq pool-list` nem jelzi-e a PTP-jét kikapcsoltként. (A MAPIR belső, közvetlenül a hardverhez kapcsolódó eszközei szintén szűrhetik a felvételt a PTP-zár alapján egy `--wait-ptp` jelzővel, amely legfeljebb 15 másodpercig várja, amíg az érzékelő eléri a SLAVE állapotot; ez az eszköz nem része a szállított CLI-nek.)
* Amíg a PTP aktívan szolgaállapotban van, az érzékelő elutasítja a kézi órajel-továbbításokat („A PTP biztosítja az órajelet”). Ez a tervezésnek köszönhető – bízzon a PTP-ben.

## Linux-megjegyzések

* **PTP-portok root jogosultság nélkül.** A PTP az UDP 319/320 portokat használja. Az `.deb` az `/etc/sysctl.d/60-chloros-ptp.conf` (`net.ipv4.ip_unprivileged_port_start = 319`) portot írja, így a háttérprogram ezekhez kapcsolódik, és a felhasználó nevében fut. Az 1.3.0-s verziótól kezdve a csomag nem állít be fájlkezelési jogosultságot (`setcap`) a háttérprogramon, és nincs szüksége az `libcap2-bin`-re; ne adjon hozzá saját maga jogosultságot, mert akkor a kicsomagolt háttérprogram nem indul el. Ha a PTP nem tudja lekötni a 319-es portot, lásd a [Linux-telepítés](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319) című részt.

* **Headless Jetson / Linux-gép:** a csomag telepíti a háttérszolgáltatásokat, de egyet sem aktivál. Ha a PTP-t (és a DAQ-t) folyamatosan üzemeltetni szeretné anélkül, hogy valaki parancsot futtatna, aktiválja a szolgáltatást a saját felhasználói fiókjával:

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  Enélkül a PTP csak akkor fut, ha valami elindította a háttérszolgáltatást — egy `chloros-cli` parancs vagy egy SDK-parancsfájl. A root `chloros-backend.service` fiókról és arról, hogyan lehet átváltani róla, lásd a [Linux-telepítés](../linux/linux-installation.md#always-on-ptp-for-headless-hosts) című részt.

## Hibaelhárítás: „Nincs DAQ-E eszköz”

| Ellenőrzés | Részletek |
| --- | --- |
| Áramellátás | Nincs LED-jelzés az érzékelőn — ellenőrizze a kapcsoló/injektor port PoE- és kapcsolatjelzőit; várjon néhány másodpercet a bekapcsolás után |
| Broadcast-tartomány | A gazdagép és az érzékelő ugyanazon az L2-szegmensen van; az mDNS nem továbbítja a forgalmat |
| Tűzfal | Windows: a telepítő **Chloros Backend (UDP-In)** szabálya engedélyezi a DAQ-E adatfolyam (UDP 5002), az mDNS (5353) és a PTP (319/320) bejövő forgalmát; egy harmadik féltől származó tűzfal esetében ugyanez szükséges. Linux: az `.deb` csak akkor nyitja meg ezeket a portokat az `ufw` / `firewalld` fájlokban, ha a tűzfal a telepítéskor aktív volt — később engedélyezze őket saját maga egy bekapcsolt tűzfalban |
| Több hálózati kártyával rendelkező gazdagép | A felderítés az mDNS-t böngészi, ARP-visszaeséssel; az `--eth-host <ip-or-hostname>` kihagyja a felderítést, és közvetlenül csatlakozik |
| GUI-újrakeresés | A felderítés csak a csatlakozási párbeszédablak nyitva tartása alatt fut; használja a Frissítés gombot |</version>
