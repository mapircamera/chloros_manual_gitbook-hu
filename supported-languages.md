# Támogatott nyelvek

Az Chloros **világszerte 38 nyelven** biztosít teljes felületi támogatást, így a világ minden táján elérhető a felhasználók számára. A nyelveket az összes felületen azonnal megváltoztathatja: asztali számítógép, böngésző, CLI és Python SDK.

Az Chloros a következő nyelveket támogatja:

| # | Nyelv | Eredeti név | CLI kód |
|---|----------|-------------|----------|
| 1 | 🇺🇸 Angol | English | `en` |
| 2 | 🇪🇸 Spanyol | Español | `es` |
| 3 | 🇵🇹 Portugál | Português | `pt` |
| 4 | 🇫🇷 Francia | Français | `fr` |
| 5 | 🇩🇪 Német | Deutsch | `de` |
| 6 | 🇮🇹 Olasz | Italiano | `it` |
| 7 | 🇯🇵 Japán | 日本語 | `ja` |
| 8 | 🇰🇷 Koreai | 한국어 | `ko` |
| 9 | 🇨🇳 Kínai (egyszerűsített) | 简体中文 | `zh` |
| 10 | 🇹🇼 Kínai (hagyományos) | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 Orosz | Русский | `ru` |
| 12 | 🇳🇱 Holland | Nederlands | `nl` |
| 13 | 🇸🇦 Arab | العربية | `ar` |
| 14 | 🇵🇱 Lengyel | Polski | `pl` |
| 15 | 🇹🇷 Török | Türkçe | `tr` |
| 16 | 🇮🇳 Hindi | हिंदी | `hi` |
| 17 | 🇮🇩 Indonéz | Bahasa Indonesia | `id` |
| 18 | 🇻🇳 Vietnámi | Tiếng Việt | `vi` |
| 19 | 🇹🇭 Thai | ไทย | `th` |
| 20 | 🇸🇪 Svéd | Svenska | `sv` |
| 21 | 🇩🇰 Dán | Dansk | `da` |
| 22 | 🇳🇴 Norvég | Norsk | `no` |
| 23 | 🇫🇮 Finn | Suomi | `fi` |
| 24 | 🇬🇷 Görög | Ελληνικά | `el` |
| 25 | 🇨🇿 Cseh | Čeština | `cs` |
| 26 | 🇭🇺 Magyar | Magyar | `hu` |
| 27 | 🇷🇴 Román | Română | `ro` |
| 28 | 🇺🇦 Ukrán | Українська | `uk` |
| 29 | 🇧🇷 brazil portugál | Português Brasileiro | `pt-BR` |
| 30 | 🇭🇰 kantoni | 粵語 | `zh-HK` |
| 31 | 🇲🇾 Maláj | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 Szlovák | Slovenčina | `sk` |
| 33 | 🇧🇬 Bolgár | Български | `bg` |
| 34 | 🇭🇷 Horvát | Hrvatski | `hr` |
| 35 | 🇱🇹 Litván | Lietuvių | `lt` |
| 36 | 🇱🇻 Lett | Latviešu | `lv` |
| 37 | 🇪🇪 Észt | Eesti | `et` |
| 38 | 🇸🇮 Szlovén | Slovenščina | `sl` |

## Hogyan lehet nyelvet váltani

### Chloros asztali számítógépen/böngészőben

1. Nyissa meg az alkalmazás beállításait
2. Keresse meg a nyelvválasztó menüt
3. Válassza ki a listából a kívánt nyelvet
4. A felület azonnal frissül

### Chloros CLI

Az `language` parancs segítségével megtekintheti vagy megváltoztathatja az CLI felület nyelvét:

```bash
# View current language
chloros-cli language

# Change to Spanish
chloros-cli language es

# Change to Chinese (Simplified)
chloros-cli language zh

# Change to Brazilian Portuguese
chloros-cli language pt-BR

# List all available languages
chloros-cli language --list
```

További részletekért lásd az [CLI dokumentációt](CLI.md).

### Az Chloros Python SDK

Az SDK inicializálásakor állítsa be a nyelvi paramétert, hogy az üzenetek és kimenetek a kívánt nyelven jelenjenek meg.

## Lefedettség

Mind a 38 nyelv teljes mértékben támogatott a következő területeken:

* **Chloros Desktop** - Teljes GUI-fordítás
* **Chloros Browser** - Webes felület minden nyelven
* **Chloros CLI** - Parancssori felület és kimeneti üzenetek
* **Chloros Python SDK** - API üzenetek és dokumentáció

A nyelvi támogatás biztosítja, hogy a felhasználók világszerte akadálymentesen, anyanyelvükön dolgozhassanak.
