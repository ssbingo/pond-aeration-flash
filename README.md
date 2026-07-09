# Pond Aeration — Firmware-Flash-Seite

Öffentliche Verteilstelle für die **fertige ESP32-Firmware** der automatischen Teichbelüftung.
Anwender flashen hier direkt im Browser (Chrome/Edge, [ESP Web Tools](https://esphome.github.io/esp-web-tools/)) —
**ohne** PlatformIO, ohne Kommandozeile.

> Der **Quellcode** der Firmware liegt in einem separaten, **privaten** Repository. Hier liegt nur der
> fertige, flashbare Binary plus die Installationsseite.

## Inhalt

| Datei | Zweck |
|------|-------|
| `index.html` / `en.html` | Installations- & Freischaltungsanleitung (DE / EN) mit dem „Firmware installieren"-Knopf |
| `style.css` | gemeinsames Design (hell/dunkel) |
| `manifest.json` | ESP Web Tools Manifest (Chip `ESP32-S3`, ein gemergtes Image bei Offset 0) |
| `firmware.factory.bin` | gemergtes Firmware-Image (Bootloader + Partitionen + App) |

## Hosting (GitHub Pages)

Repo öffentlich → **Settings → Pages → Branch `main` / root**. Die Seite ist dann unter
`https://<user>.github.io/<repo>/` erreichbar (HTTPS ist Pflicht, weil der Browser den USB-Port nur
über eine sichere Verbindung freigibt).

## Aktualisieren (neue Firmware-Version)

Der Release-Build im **privaten** Firmware-Repo erzeugt `firmware.factory.bin` (gemergtes Image,
z. B. via `esptool merge_bin` / `pio run`). Dieser Binary wird hierher kopiert und `manifest.json`
(Feld `version`) sowie die Versionsangabe in `index.html`/`en.html` angepasst, dann committen —
GitHub Pages veröffentlicht automatisch. Idealerweise übernimmt das ein CI-Schritt im privaten Repo
(Deploy-Key / PAT auf dieses Repo).

> **Wichtig — Produktions-Build:** Der hier abgelegte Binary muss aus dem **prod**-Profil des
> Schlüssel-Werkzeugs stammen (`POND_PROFILE=prod … fw-header`, dann bauen), damit die ausgestellten
> Freischaltcodes passen. Ein versehentlich veröffentlichter **dev**-Build akzeptiert nur dev-Codes.

## Chip-Offsets (Referenz)

`firmware.factory.bin` ist ein bei **Offset 0** flashbares Komplett-Image für den **ESP32-S3**
(Waveshare ESP32-S3-POE-ETH-8DI-8RO). Alternativ könnten die Einzelteile im Manifest gelistet werden
(`bootloader.bin` @ 0x0, `partitions.bin` @ 0x8000, `boot_app0.bin` @ 0xe000, `firmware.bin` @ 0x10000).
