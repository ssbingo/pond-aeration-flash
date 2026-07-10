# Pond Aeration — Firmware-Flash-Seite

Öffentliche Verteilstelle für die **fertige ESP32-Firmware** der automatischen Teichbelüftung
([ioBroker-Adapter `automatic-pond-aeration`](https://github.com/ssbingo/ioBroker.automatic-pond-aeration)).
Anwender flashen hier **direkt im Browser** (Chrome/Edge, [ESP Web Tools](https://esphome.github.io/esp-web-tools/)) —
**ohne** PlatformIO, ohne Kommandozeile, ohne Treiber-Gefummel.

**Live-Seite:** https://ssbingo.github.io/pond-aeration-flash/
**Aktuelle Firmware:** siehe `version` in [`manifest.json`](manifest.json) (derzeit **1.2.2**).

> Der **Quellcode** der Firmware liegt in einem separaten, **privaten** Repository. Hier liegen nur
> die fertigen, flashbaren Binärdateien plus die Installations- und Freischaltungsseite.

---

## Für Anwender: So wird geflasht (Kurzfassung)

1. ESP32 per **USB-C-Datenkabel** (kein reines Ladekabel) an einen **Computer** anschließen.
2. Die Seite in **Chrome** oder **Edge** öffnen (am PC/Notebook — nicht am Handy, nicht Safari/Firefox).
3. Auf **„Firmware installieren"** klicken, den seriellen Anschluss des ESP wählen, **Install** bestätigen.
4. Nach ~1–2 Minuten startet der ESP mit der Firmware neu. Danach per Netzwerkkabel (LAN/PoE) ins Netz,
   IP im Router suchen (Hostname `pond-aeration`) und im Browser öffnen.

Die ausführliche, anfängertaugliche Anleitung inkl. **Testphase & Freischaltung** steht auf der Seite
selbst (`index.html` DE / `en.html` EN).

---

## Inhalt des Repos

| Datei | Zweck |
|------|-------|
| `index.html` / `en.html` | Installations- & Freischaltungsanleitung (DE / EN) mit dem „Firmware installieren"-Knopf |
| `style.css` | gemeinsames Design (hell/dunkel, responsiv) |
| `manifest.json` | ESP Web Tools Manifest (Chip `ESP32-S3`, ein gemergtes Image bei Offset 0, `new_install_prompt_erase: true`) |
| `firmware.factory.bin` | **Installer-Image** — gemergt (Bootloader + Partitionstabelle + `boot_app0` + App), wird bei Offset 0 geflasht |
| `pond-aeration-ota.bin` | **OTA-Image** (nur App) — wird vom Gerät für das **Ein-Klick-Online-Update** und für den Datei-Upload auf der Update-Seite verwendet |

---

## Zwei Wege, die Firmware aufzuspielen (wichtig für die Freischaltung)

| Weg | Datei | Was passiert | Freischaltcode |
|-----|-------|--------------|----------------|
| **Browser-Installer** (diese Seite) | `firmware.factory.bin` @ 0 | schreibt das **komplette** Flash ab Offset 0 — überschreibt dabei auch den **NVS-Bereich** | wird **gelöscht** → auf der Geräteseite *Lizenz* erneut eintragen |
| **Update-Seite am Gerät** (Ein-Klick oder Datei-Upload) | `pond-aeration-ota.bin` | schreibt nur die **inaktive App-Partition** (OTA) | bleibt **erhalten** (ebenso alle Einstellungen) |

> **Kein Neu-Anfragen nötig:** Der **Gerätecode** wird fest aus der Hardware (Werks-MAC) abgeleitet und
> ändert sich nie. Nach einem Installer-Flash gilt **derselbe Freischaltcode weiter** — einfach erneut
> einfügen. Für spätere Updates deshalb die **Update-Seite** bevorzugen, nicht den Installer.

---

## Hosting (GitHub Pages)

Repo öffentlich → **Settings → Pages → Branch `main` / root**. Die Seite ist dann unter
`https://<user>.github.io/<repo>/` erreichbar. **HTTPS ist Pflicht**, weil der Browser den USB-Port
(Web Serial) nur über eine sichere Verbindung freigibt. Nach einem Push dauert die Veröffentlichung
über den Pages-CDN typischerweise 1–2 Minuten.

---

## Aktualisieren (neue Firmware-Version veröffentlichen)

Der Release-Build im **privaten** Firmware-Repo erzeugt zwei Dateien:

- `.pio/build/<env>/firmware.factory.bin` → hierher als **`firmware.factory.bin`** kopieren
- `.pio/build/<env>/firmware.bin` → hierher als **`pond-aeration-ota.bin`** kopieren

Danach in **`manifest.json`** das Feld `version` und die Versions-Pille in **`index.html`** und
**`en.html`** (`Firmware x.y.z`) anpassen, committen und pushen — GitHub Pages veröffentlicht
automatisch. Das Ein-Klick-Online-Update der Geräte liest die neue Version aus `manifest.json` und
lädt `pond-aeration-ota.bin`.

> **Wichtig — Produktions-Build:** Die hier abgelegten Binärdateien **müssen** aus dem **prod**-Profil
> des [Schlüssel-Werkzeugs](https://github.com/ssbingo/ioBroker.automatic-pond-aeration) stammen
> (`POND_PROFILE=prod … fw-header`, dann bauen → Schlüssel-ID **kid 2**), damit die ausgestellten
> Freischaltcodes akzeptiert werden. Ein versehentlich veröffentlichter **dev**-Build (kid 1)
> akzeptiert nur dev-Codes.

---

## Chip-Offsets (Referenz)

`firmware.factory.bin` ist ein bei **Offset 0** flashbares Komplett-Image für den **ESP32-S3**
(Waveshare ESP32-S3-POE-ETH-8DI-8RO). Es enthält:

| Offset | Teil |
|--------|------|
| `0x0000` | `bootloader.bin` |
| `0x8000` | `partitions.bin` (Partitionstabelle `default_16MB.csv`) |
| `0xe000` | `boot_app0.bin` |
| `0x10000` | `firmware.bin` (die App) |

Der NVS-Bereich (Freischaltcode/Einstellungen) liegt bei `0x9000` und wird vom Installer-Image
mit überschrieben — daher die Freischaltung-erneut-eintragen-Regel oben. Das OTA-Image
`pond-aeration-ota.bin` ist reine App und lässt NVS unangetastet.
