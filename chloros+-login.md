# Chloros+ Bejelentkezés

## Bejelentkezés a grafikus felületen

Az <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> oldalon található oldalsáv menü segítségével bejelentkezhet az Chloros+ fiókjába, és hozzáférhet a további funkciókhoz.

**Gépenként csak egyszer kell bejelentkeznie.** A grafikus felület, az CLI és az Python SDK ugyanazt a gyorsítótárazott munkamenetet használja — az asztali grafikus felületen történő bejelentkezés az adott gépen az CLI és az SDK alkalmazásokat is aktiválja (és fordítva az `chloros-cli login`-en keresztül).

Bejelentkezés után a fiók adatai megjelennek:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.2.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->
## Csomagszintek

| Csomag | `plan_id` | Típus |
| --- | --- | --- |
| Iron | `0` | Ingyenes |
| Copper | `1` | Fizetős (Chloros+) |
| Bronze | `2` | Fizetős (Chloros+) |
| Ezüst | `3` | Fizetős (Chloros+) |
| Arany | `4` | Fizetős (Chloros+) |

Az egyes fizetős csomagok tartalmáról a [csomagok és árak](https://cloud.mapir.camera/pricing) oldalon tájékozódhat.

### Az CLI / SDK hozzáféréshez fizetős csomag szükséges

Az CLI és az Python, valamint az SDK eléréséhez **bármely fizetős Chloros+ csomag (Copper vagy magasabb)**szükséges. Ez**szerveroldalon** kerül érvényesítésre — minden CLI/SDK kérésnek tartalmaznia kell egy aktív munkamenetet és egy fizetős csomagot:

| HTTP állapot | `error_code` | Jelentés | Megoldás |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Nincs bejelentkezve ezen a gépen | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Be van jelentkezve, de a csomagszint túl alacsony (ingyenes Iron szint) | Frissítsen bármely fizetős Chloros+ csomagra |

Az `chloros-cli status` az ingyenes csomagban is elérhető marad, így bármikor megtekintheti az aktuális csomagját és azt, hogy miért utasították el a hozzáférést.

### Csatlakoztatható eszközök száma csomagonként

Minden csomag korlátozza, hogy egyszerre hány LATTICE kamera és DAQ fényérzékelő csatlakoztatható élőben:

| Csomag | LATTICE kamerák | DAQ fényérzékelők |
| --- | --- | --- |
| Iron (ingyenes / be nem jelentkezve) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## CLI bejelentkezés

Jelentkezzen be az Chloros+ hitelesítő adataival az CLI feldolgozás engedélyezéséhez. Az Linux (GUI nélkül) rendszeren ez az egyetlen módja a licenc aktiválásának.

**Szintaxis:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK felhasználók**: Az Python SDK egy programozási módszert is biztosít a gyorsítótárban tárolt hitelesítő adatok törlésére. A részleteket lásd az [SDK Referencia](reference/sdk-reference.md) című dokumentumban.
{% endhint %}

**Példa:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Különleges karakterek**: Az olyan jelszavakat, amelyek `$`, `!` vagy szóköz karaktereket tartalmaznak, egyszerű idézőjelek közé kell tenni.
{% endhint %}

**Kimenet:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.2.0"; capture a successful login with the current output format. -->
### Hitelesítőadatok tárolása

A gyorsítótárba mentett hitelesítőadatok és konfiguráció **minden platformon** a felhasználói otthoni könyvtár `.chloros` mappájában kerülnek tárolásra:

| Platform | Hitelesítőadatok gyorsítótárának elérési útja |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

### A terv lejárati ideje és az offline türelmi időszak

A grafikus felhasználói felületen (GUI) megjelenő terv lejárati ideje jelzi, mikor válik érvénytelenné a licenc. Ismétlődő havi előfizetések esetén a lejárat a hónap végén következik be; éves előfizetések esetén az előfizetés megkezdésétől számított egy év múlva.

Az Chloros online ellenőrzi a licencét, de az offline munkavégzés egy türelmi időszakon belül támogatott:

* A sikeres szerveres érvényesítéseket **5 percig** tárolja a rendszer, így normál használat esetén nagyon kevés licenc-lekérdezésre van szükség.
* Az aláírt, gépre kötött licenc-gyorsítótár hosszabb offline időszakokat fed le: **havi csomagok esetén 30 napig**,**éves csomagok esetén pedig az előfizetés lejárati dátumáig (legfeljebb 365 napig)**.
* A türelmi időszak lejártával a csomag az ingyenes Iron szintre vált át, amíg a gép egyszer el nem éri a licenckiszolgálót; a hozzáférés a következő sikeres ellenőrzés után folytatódik.

### Eszközkorlát

Minden Chloros+ csomag különböző számú regisztrált eszközt kínál. Minden eszköz, amelyen Chloros+ fiókkal jelentkezik be, beleszámít a regisztrált eszközök számába. Az MAPIR Cloud-fiók oldalán átnevezheti és eltávolíthatja az eszközöket.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ csomag</th><th align="center">COPPER</th><th align="center">BRONZ</th><th align="center">EZÜST</th><th align="center">ARANY</th></tr></thead><tbody><tr><td align="right">Támogatott eszközök</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>Fiókjának pontos eszközkvótája az MAPIR Cloud fiókoldalán látható. Az eszközről való kijelentkezés biztosan felszabadítja a hozzá tartozó helyet, és egy már regisztrált eszköz bármikor újra bejelentkezhet, még akkor is, ha a fiók elérte az eszközkvótáját.
