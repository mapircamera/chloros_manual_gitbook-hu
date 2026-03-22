# Chloros+ Bejelentkezés

## Chloros és Chloros (böngésző) Bejelentkezés

A felhasználó <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> oldalsáv menüjével bejelentkezhet az Chloros+ fiókjába, és feloldhatja a további funkciókat.

Bejelentkezés után megjelennek a fiók adatai:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI Bejelentkezés

Jelentkezzen be Chloros+ hitelesítő adataival az CLI feldolgozás engedélyezéséhez. Az Linux (GUI nélkül) rendszeren ez az egyetlen módja a licenc aktiválásának.

**Szintaxis:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK felhasználók**: Az Python SDK egy programozási `logout()` módszert is biztosít a gyorsítótárban tárolt hitelesítő adatok törléséhez. A részleteket lásd az [Python SDK dokumentációban](api-python-sdk.md#logout).
{% endhint %}

**Példa:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Különleges karakterek**: Az olyan jelszavakat, amelyek `$`, `!` vagy szóköz karaktereket tartalmaznak, egyszerű idézőjelek közé kell tenni.
{% endhint %}

**Kimenet:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Hitelesítő adatok tárolása

A gyorsítótárba mentett hitelesítő adatok platformspecifikus helyen kerülnek tárolásra:

| Platform | Hitelesítő adatok gyorsítótárának elérési útja |
| --- | --- |
| **Windows** | `%APPDATA%\Chloros\cache\` |
| **Linux** | `~/.cache/chloros/` |

### A terv lejárati ideje

A GUI-ban látható terv lejárati ideje jelzi, mikor válik érvénytelenné a licenc. Ismétlődő havi előfizetések esetén a lejárat a hónap végén következik be. Éves előfizetések esetén ez az előfizetés megkezdésétől számított egy év. A licencellenőrzéshez havi internetkapcsolatra van szükség, 30 napos türelmi idővel.

### Eszközkorlát

Minden Chloros+ tervezet különböző számú regisztrált eszközt kínál. Minden eszköz, amelyre Chloros+ fiókkal jelentkezik be, beleszámít a regisztrált eszközök számába. Az MAPIR Cloud fiókoldalán átnevezheti és eltávolíthatja az eszközöket.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ csomag</th><th align="center">COPPER</th><th align="center">BRONZ</th><th align="center">EZÜST</th><th align="center">ARANY</th></tr></thead><tbody><tr><td align="right">Támogatott eszközök</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
