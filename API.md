# API-Dokumentation – STM32_Millis

## Überblick

| Element | Datei | Beschreibung |
|---|---|---|
| `millis()` | `millis.c` / `millis.h` | Vergangene Zeit seit Systemstart in ms |
| `MILLIS_MAJOR` | `millis.h` | Major-Versionsnummer |
| `MILLIS_MINOR` | `millis.h` | Minor-Versionsnummer |
| `MILLIS_PATCH` | `millis.h` | Patch-Versionsnummer |
| `MILLIS_DEV` | `millis.h` | Entwicklungs-Flag |

---

## Funktionen

### `millis`

```c
uint32_t millis(void);
```

Gibt die seit dem Systemstart (bzw. letztem Aufruf von `HAL_Init()`) vergangene Zeit in Millisekunden zurück.

**Parameter:** keine

**Rückgabewert:** `uint32_t` – Millisekunden seit Systemstart. Läuft nach 2^32 − 1 ms (≈ 49,7 Tage) auf 0 zurück.

**Implementierung:** Direkter Aufruf von `HAL_GetTick()`.

**Beispiele:**

```c
// Einfache Verzögerung (nicht-blockierend)
uint32_t start = millis();
while (millis() - start < 1000) {
    // warte 1 Sekunde
}

// Periodische Aufgabe
#define INTERVAL_MS  100U

uint32_t lastRun = 0;

void periodicTask(void) {
    if (millis() - lastRun >= INTERVAL_MS) {
        lastRun = millis();
        // Aufgabe ausführen
    }
}

// Timeout-Erkennung
uint32_t timeout = millis();
while (!sensorReady()) {
    if (millis() - timeout > 5000) {
        // Timeout nach 5 Sekunden
        handleTimeout();
        break;
    }
}
```

**Hinweise:**
- Die Auflösung beträgt 1 ms, da `HAL_GetTick()` vom SysTick-Timer mit 1-ms-Periode gespeist wird.
- Die Funktion ist nicht interrupt-sicher in dem Sinne, dass `HAL_GetTick()` selbst einen 32-Bit-Lesezugriff auf einer ARM Cortex-M-Architektur ausführt – dieser ist atomar, sodass kein Mutex erforderlich ist.
- Differenzberechnungen (`millis() - start`) sind korrekt, solange die gemessene Zeitspanne 2^32 ms nicht überschreitet.

---

## Versionsmakros

Definiert in `millis.h`:

```c
#define MILLIS_MAJOR  0   // Inkompatible API-Änderungen
#define MILLIS_MINOR  0   // Neue abwärtskompatible Funktionen
#define MILLIS_PATCH  0   // Fehlerbehebungen
#define MILLIS_DEV    0   // 0 = Release, 1 = Entwicklungsversion
```

**Verwendung zur Compile-Zeit:**

```c
#if MILLIS_MAJOR >= 1
    // Neuere API verwenden
#else
    // Kompatibilitätspfad
#endif
```

---

## Abhängigkeiten

| Abhängigkeit | Zweck |
|---|---|
| `main.h` | Bindet die STM32 HAL-Bibliothek ein (enthält `HAL_GetTick()`) |
| `stdint.h` | Typdefinition für `uint32_t` |

`millis.h` bindet `stdint.h` direkt ein und kann daher unabhängig von `main.h` in anderen Header-Dateien verwendet werden. `millis.c` benötigt `main.h` zur Kompilierung.
