# Changelog

Alle nennenswerten Änderungen an diesem Projekt werden hier dokumentiert.
Format nach [Keep a Changelog](https://keepachangelog.com/de/1.1.0/),
Versionierung nach [Semantic Versioning](https://semver.org/lang/de/).

## [Unveröffentlicht]

## [1.0.0] - 2026-10-31

### Hinzugefügt
- `millis()` liefert die Zeit seit dem Systemstart in Millisekunden auf Basis von `HAL_GetTick()`.
- Versions-Defines `MILLIS_MAJOR`, `MILLIS_MINOR`, `MILLIS_PATCH` und `MILLIS_DEV` in `millis.h`.
- Debug-Symbol `DEBUG_MILLIS`, das bei gesetztem `DEBUG` definiert wird.
- README mit Beschreibung, API, Verwendung und Hinweisen zu Überlauf, Auflösung und Interrupts.
- Lizenz GPL-3.0.

[Unveröffentlicht]: https://github.com/Diveturtle93/STM32_Millis/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/Diveturtle93/STM32_Millis/releases/tag/v1.0.0
