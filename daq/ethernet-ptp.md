# DAQ-E hálózat és időszinkronizálás

> Az érzékelő fizikai hálózati beállítása – kábelezés, PoE, IP-cím hozzárendelése és az eszköz saját hálózati beállításai – a **[DAQ felhasználói kézikönyvben](https://mapir.gitbook.io/daq/daq-e/network-setup)** található. Ez az oldal az Chloros oldalát tárgyalja: a csatlakozást, az időszinkronizálást, valamint azt, hogy mit kell tenni, ha a felderítés nem jár eredménnyel.

A DAQ-E a DAQ-család Ethernet-alapú tagja: PoE-n keresztül kap áramellátást, mDNS-en keresztül (`_daq-e._tcp` szolgáltatás) ismerhető fel, és az érzékelő azonosítójából származtatott gazdagépnévvel címezhető — `daq-e-<6 hex>.local`, pl. `daq-e-def330.local`. Ez az oldal azt tárgyalja, hogyan továbbítja az adatokat a hálózaton, és hogyan vesz részt a PTP időszinkronizálásban.

## Átviteli módok

| Mód | Végpont | Felhasználók | Megjegyzések |
| --- | --- | --- | --- |
| **Multicast** (alapértelmezett) | UDP `239.10.10.10:5002` | Ugyanazon a LAN-on lévő tetszőleges számú eszköz ugyanazt az adatfolyamot veszi | Minden datagramot CRC-16/CCITT-vel ellenőriznek |
| **Raw** | TCP-port `5000` | Pontosan egy kliens (kizárólagos) | Bájtszinten kompatibilis a DAQ-U-val |

Az Chloros alapértelmezés szerint multicastot használ, ami lehetővé teszi, hogy a GUI, az CLI és az SDK egyszerre figyeljenek egy érzékelőt.

## Hálózati követelmények

* **Ugyanaz a broadcast-tartomány.** Az Chloros-et futtató gépnek ugyanazon az L2 hálózati szegmensen kell lennie, mint az érzékelőnek — az mDNS-felfedezés nem halad át útválasztókon.
* **Windows tűzfal-kérés: fogadja el.** Amikor az Chloros először köti a multicast-socketeket, az Windows Defender egyszer kérdez. Az engedélyezés kiterjed a DAQ-E adatokra (UDP 5002), az mDNS-re (UDP 5353) és a PTP-re (UDP 319/320). Az Linux esetében ez csendben történik.
* **PoE-tápellátás, nincs állapotjelző LED.** A DAQ-E-nek nincs saját LED-je — ellenőrizze a tápellátást a kapcsoló vagy az injektor portján található link/PoE-jelzőfény segítségével, és a bekapcsolás után várjon néhány másodpercet, amíg a készülék elindul és csatlakozik a hálózathoz.

## Csatlakoztatás

**GUI:** Light Sensors fül → Connect Sensor → Device Type „DAQ-E (Ethernet)”. A felderítés csak addig fut, amíg a csatlakozási párbeszédablak a képernyőn látható (mDNS-keresés és ARP-letapogatás az Windows-en), 15 másodpercenként megismétlődve; a Frissítés gomb megnyomásával azonnal újraindul a keresés. Az észlelt érzékelők megjelennek a legördülő menüben; az első észlelt érzékelő automatikusan kijelölésre kerül.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (háttérfolyamat fut):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### Több hálózati kártyával rendelkező gazdagépek és az első csatlakozás rendszerindítás után

Több aktív hálózati interfésszel rendelkező gazdagépeken az indítás utáni **első** `pool-connect --eth` üres eredményt adhat, még akkor is, ha az érzékelő rendben van — az észlelési keresés elmulaszthatja azt az interfészt, amelyen az érzékelő található, amíg az ARP-gyorsítótár még nem melegedett fel. A megbízható megoldás az, ha kihagyjuk a felderítést, és kifejezetten megadjuk a címet:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

Az `--eth-host` elfogadja az mDNS-gépnevet vagy az IP-címet, mindig a megfelelő érzékelőt célozza meg, és ez az ajánlott formátum szkriptek és headless telepítések esetén. A grafikus felhasználói felületen használja a csatlakozási párbeszédpanel Frissítés gombját, és engedélyezzen egy újrakeresési ciklust.

## Eszközbeállítások és firmware

Maga az érzékelő tárolja a hálózati beállításokat – statikus IP vagy DHCP + link-local címzés, eszköznév, automatikus adatfolyam indításkor, OTA jelszó. Ezek az eszközoldali beállítások nem jelennek meg parancsokként a szállított CLI verzióban; kezelésük az Chloros grafikus felhasználói felületen történik, ahol megjelennek, vagy az MAPIR támogatás segítségével.

**A firmware-frissítések beépültek a grafikus felhasználói felületbe.**Ha egy csatlakoztatott DAQ-E olyan firmware-t futtat, amely régebbi, mint az Chloros verzióhoz mellékelt kép, az érzékelősorán sárga**Frissítés elérhető** ikon jelenik meg, és a fogaskerék-beállítások ablakban megjelenik egy „Frissítés a<version>

” gomb. A frissítés körülbelül 30 másodperc alatt történik a hálózaton keresztül; az érzékelő automatikusan újraindul és újra csatlakozik, és egy megszakadt átvitel esetén a jelenlegi firmware változatlan marad.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP időszinkronizálás

A DAQ-E v1.2.0+ verziójú firmware az IEEE 1588 PTPv2 szabványban rendes (kizárólag szolga) óraként vesz részt. **Az Chloros gazdagép háttérrendszere a PTP grandmaster** — minden DAQ-E és minden LATTICE kamera a LAN-on a 0. tartományban ehhez csatlakozik, így az összes eszköz időbélyege ~1 ms-os tűréshatáron belül marad. Ez a megosztott óra teszi lehetővé, hogy a DAQ-leolvasások időbélyegei egyezzenek a kamera expozícióival (lásd [Felvétel és a .daq formátum](recording.md)).

Ellenőrizze a szinkronizálást az CLI-ből:

| Parancs | Megjelenít |
| --- | --- |
| `chloros-cli time-sync status` | A gazdagép grandmaster állapota, BMCA prioritások, óraazonosító |
| `chloros-cli time-sync peers` | Minden észlelt alrendszer (DAQ-E érzékelők + LATTICE kamerák) |
| `chloros-cli time-sync cameras` | Kameránkénti PTP-állapot (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | A grandmaster folyamat újraindítása |

A grafikus felhasználói felületen a DAQ-E beállítások modál ablakában egy élő **PTP Sync** sor jelenik meg, amely az érzékelő aktuális PTP állapotát mutatja.

Részletek a szigorú összehangolást igénylő felhasználók számára:

* Minden streamelt adatgram tartalmaz egy flag mezőt; **a 2. bit be van állítva azoknál a kereteknél, amelyek időbélyege PTP-szinkronizált**. Azok a folyamatok, amelyek szigorú kamera/DAQ-összehangolást igényelnek, ennek a bitnek a jelenlétét kell figyelniük.
* A szinkronizált rögzítés előtt ellenőrizze, hogy az érzékelő megjelenik-e az `chloros-cli time-sync peers` listában. (Az MAPIR belső, közvetlen hardveres eszközei szintén szabályozhatják a felvételt PTP-zár esetén egy `--wait-ptp` jelzővel, amely legfeljebb 15 másodpercig várja, amíg az érzékelő eléri a SLAVE állapotot; ez a funkció nem része a szállított CLI-nek.)
* Amíg a PTP aktívan szolgaállapotban van, az érzékelő elutasítja a kézi órajel-továbbításokat („A PTP biztosítja az órajelet”). Ez a tervezésnek köszönhető — bízzon a PTP-ben.

## Linux megjegyzések

* **A PTP-hez telepítéskor az `libcap2-bin` szükséges.** Az `.deb` postinst parancs megadja az `cap_net_bind_service=+ep` jogosultságot az `/usr/lib/chloros/chloros-backend`-en, így az root jogosultság nélkül is hozzárendelheti a 319/320-as PTP-portokat. Ha az `libcap2-bin` hiányzik, akkor ezt a lépést kihagyja a rendszer, és a PTP nem indul el. Megoldás:

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **Headless Jetson / Raspberry Pi:** az első telepítéskor a systemd egység `chloros-backend.service` létrejön, de nincs engedélyezve. A GUI nélküli, folyamatosan aktív PTP (és DAQ elérhetőség) érdekében:

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  Enélkül a PTP csak addig fut, amíg az Chloros GUI nyitva van.

## Hibaelhárítás: „No DAQ-E devices found”

| Ellenőrzés | Részletek |
| --- | --- |
| Tápellátás | Nincs LED az érzékelőn — ellenőrizze a kapcsoló/injektor port PoE- és kapcsolati jelzőfényeit; várjon néhány másodpercet a bekapcsolás után |
| Broadcast-tartomány | A gazdagép és az érzékelő ugyanazon az L2-szegmensen van; az mDNS nem irányítja az adatforgalmat |
| Windows tűzfal | Az első futtatáskor fogadja el a Defender kérését (UDP 5002, 5353, 319/320) |
| Több hálózati kártyával rendelkező gazdagép | A rendszerindítás utáni első felderítés során előfordulhat, hogy az érzékelő kimarad — csatlakozzon az `--eth-host <ip-or-hostname>` segítségével |
| Újrafelderítés a grafikus felületen | A felderítés csak a csatlakozási párbeszédablak nyitott állapotában fut; használja a Frissítés gombot |</version>
