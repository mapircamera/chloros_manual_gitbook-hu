# Chloros+ Bejelentkezés

## Chloros és Chloros (böngésző) Bejelentkezés

A felhasználói <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> oldalsáv menü lehetővé teszi, hogy bejelentkezzen az Chloros+ fiókjába, és további funkciókat nyisson meg.

Bejelentkezés után a fiók adatait jeleníti meg:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI Bejelentkezés

Jelentkezzen be Chloros+ hitelesítő adataival, hogy engedélyezze az CLI feldolgozást.

**Szintaxis:**

```bash
chloros-cli login <email> <password>
```

**Példa:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Különleges karakterek**: Használjon egyszerű idézőjeleket az olyan karaktereket tartalmazó jelszavak körül, mint `$`, `!` vagy szóközök.
{% endhint %}

**Kimenet:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### A terv lejárati ideje

A GUI-ban látható a terv lejárati ideje, amely azt mutatja, hogy mikor válik érvénytelenné a licenc. Havi előfizetések esetén a lejárati idő a hónap végén van. Éves előfizetések esetén a lejárati idő az előfizetés megkezdésétől számított egy év. A licenc ellenőrzéséhez havi internetkapcsolat szükséges, 30 napos türelmi idővel.

### Eszközkorlát

Minden Chloros+ terv különböző számú regisztrált eszközt kínál. Minden eszköz, amelyre Chloros+ fiókkal jelentkezik be, beleszámít a regisztrált eszközök számába. Az MAPIR Cloud fiókoldalon átnevezheti és eltávolíthatja az eszközöket.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ csomag</th><th align="center">COPPER</th><th align="center">BRONZ</th><th align="center">SILVER</th><th align="center">ARANY</th></tr></thead><tbody><tr><td align="right">Támogatott eszközök</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
