# Chloros+ bejelentkezés

## Bejelentkezés a grafikus felületen

A felhasználó <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> oldalsáv menüjéből bejelentkezhet a Chloros+ fiókjába, és így további funkciókat érhet el.

**Gépenként csak egyszer kell bejelentkeznie.** A grafikus felület (GUI), a parancssori felület (CLI) és a Python SDK ugyanazt a gyorsítótárazott munkamenetet használja — az asztali grafikus felületen történő bejelentkezés aktiválja a parancssori felületet és az SDK-t is az adott gépen (és fordítva, az `chloros-cli login` használatával).

Bejelentkezés után a fiók adatai megjelennek:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.3.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->

## Csomagszintek

| Csomag | `plan_id` | Típus |
| --- | --- | --- |
| Iron | `0` | Ingyenes |
| Copper | `1` | Fizetős (Chloros

+) |
| Bronze | `2` | Fizetős (Chloros

+) |
| Silver | `3` | Fizetős (Chloros

+) |
| Arany | `4` | Fizetős (Chloros

+) |

Az egyes fizetős csomagok tartalmáról a [csomagok és árak](https://cloud.mapir.camera/pricing) oldalon tájékozódhat.

### A CLI

-hez és az SDK

-hez való hozzáféréshez fizetős csomag szükséges

A CLI

és a Python

SDK

eléréséhez **bármely fizetős Chloros

+ csomag (Copper vagy magasabb)**szükséges. Ez**szerveroldalon** kerül érvényesítésre — minden CLI

/SDK

kérésnek tartalmaznia kell egy aktív munkamenetet és egy fizetős csomagot:

| HTTP

-állapot | `error_code` | Jelentés | Megoldás |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Nincs bejelentkezve ezen a gépen | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Be van jelentkezve, de a csomag szintje túl alacsony (ingyenes Iron szint) | Frissítsen bármely fizetős Chloros

+ csomagra |

Az `chloros-cli status` az ingyenes csomagban is elérhető marad, így bármikor megtekintheti az aktuális csomagját és azt, hogy miért utasították el a hozzáférést.

### Csatlakoztatható eszközök száma csomagonként

Minden csomag korlátozza, hogy egyszerre hány LATTICE kamera és DAQ fényérzékelő csatlakoztatható élőben:

| Csomag | LATTICE kamerák | DAQ fényérzékelők |
| --- | --- | --- |
| Iron (ingyenes / be nem jelentkezve) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## CLI

-bejelentkezés

Jelentkezzen be a Chloros

+ hitelesítő adataival a CLI

-feldolgozás engedélyezéséhez. Linux

rendszeren (GUI nélkül) ez az egyetlen módja a licenc aktiválásának.

**Szintaxis:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK

-felhasználók**: A Python

SDK

egy programozási `logout()` metódust is biztosít a gyorsítótárban tárolt hitelesítő adatok törléséhez. A részleteket lásd az [SDK

-referenciában](reference/sdk-reference.md).
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
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.3.0"; capture a successful login with the current output format. -->

### Hitelesítőadatok tárolása

A gyorsítótárba mentett hitelesítőadatok és konfiguráció **minden platformon** a felhasználói otthoni könyvtár `.chloros` mappájában kerülnek tárolásra:

| Platform | Hitelesítőadatok gyorsítótárának elérési útja |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

Ha egy újabb verziót telepít a meglévő felett, a bejelentkezési adatai megmaradnak Windows

on (a telepítő) és Linux

on (az `.deb`, beleértve az `chloros-cli update`-et is). A Chloros

eltávolítása — vagy a Linux

-csomag eltávolítása — törli a bejelentkezési adatokat a gyorsítótárból, így újratelepítés után újra be kell jelentkeznie.

### A csomag lejárati ideje és az offline türelmi időszak

A grafikus felhasználói felületen látható csomag lejárati ideje jelzi, mikor válik érvénytelenné a licenc. Havi előfizetések esetén a lejárat a hónap végén következik be; éves előfizetések esetén az előfizetés megkezdésétől számított egy év múlva.

A Chloros

online ellenőrzi a licencét, de egy türelmi időszakon belül támogatja az offline munkavégzést is:

* A sikeres szerveres érvényesítéseket **5 percig** tárolja a gyorsítótár, így normál használat esetén nagyon kevés licenc-lekérdezésre van szükség.
* Az aláírt, gépre kötött licenc-gyorsítótár hosszabb offline időszakokat fed le: **30 napig a havi csomagok esetében**, és**az előfizetés lejárati dátumáig (legfeljebb 365 napig) az éves csomagok esetében**.
* A türelmi időszak lejárta után a csomag az ingyenes Iron szintre vált át, amíg a számítógép egyszer el nem éri a licenckiszolgálót; a hozzáférés a következő sikeres ellenőrzéskor folytatódik.

### Eszközkorlát

Minden Chloros

+ csomag különböző számú regisztrált eszközt kínál. Minden olyan eszköz, amelyre Chloros

+ fiókkal jelentkezik be, beleszámít a regisztrált eszközök számába. Az MAPIR

Cloud fiókoldalán átnevezheti és eltávolíthatja az eszközöket.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros-csomag</th><th align="center">COPPER</th><th align="center">BRONZ</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Támogatott eszközök</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

Fiókjának pontos eszközkvótája a MAPIR Cloud fiókoldalán látható. Ha kijelentkezik egy eszközről, az megbízhatóan felszabadítja a hozzá tartozó helyet, és egy már regisztrált eszköz bármikor újra bejelentkezhet, még akkor is, ha a fiók elérte az eszközkvótát.
