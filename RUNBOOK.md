# Runbook – STM32_Millis

Dieses Runbook beschreibt die Integration und Fehlersuche für die STM32_Millis-Bibliothek.

## Integration in ein STM32CubeIDE-Projekt

### Schritt 1: Dateien hinzufügen

```
Projektstruktur (Ziel):
├── Core/
│   ├── Inc/
│   │   └── millis.h      ← hier ablegen
│   └── Src/
│       └── millis.c      ← hier ablegen
```

### Schritt 2: Build-System prüfen

STM32CubeIDE erkennt neue `.c`-Dateien im `Core/Src/`-Verzeichnis automatisch. Bei anderen Build-Systemen muss `millis.c` manuell in das Makefile oder CMakeLists.txt aufgenommen werden:

**Makefile:**
```makefile
C_SOURCES += Core/Src/millis.c
```

**CMakeLists.txt:**
```cmake
target_sources(${PROJECT_NAME} PRIVATE Core/Src/millis.c)
```

### Schritt 3: SysTick-Konfiguration sicherstellen

`millis()` basiert auf `HAL_GetTick()`, das den SysTick-Timer verwendet. Die Standardkonfiguration von STM32CubeMX setzt den SysTick auf 1 ms – das ist die erwartete Einstellung.

Prüfen in `stm32xxxx_hal_conf.h`:
```c
#define HAL_TICK_FREQ_DEFAULT  HAL_TICK_FREQ_1KHZ  // muss 1KHZ sein
```

## Häufige Probleme

### `millis()` gibt immer 0 zurück

**Ursache:** `HAL_Init()` wurde nicht aufgerufen oder der SysTick-Interrupt ist deaktiviert.

**Lösung:** Sicherstellen, dass `HAL_Init()` am Anfang von `main()` aufgerufen wird und der SysTick-Interrupt in NVIC nicht deaktiviert ist.

### Zeitbasierte Logik läuft zu langsam oder zu schnell

**Ursache:** SysTick-Periode ist nicht auf 1 ms konfiguriert.

**Lösung:** In STM32CubeMX unter "System Core → SYS" den Timebase Source auf SysTick setzen und sicherstellen, dass der Systemtakt korrekt konfiguriert ist.

### Kompilierungsfehler: `HAL_GetTick` nicht gefunden

**Ursache:** `main.h` ist nicht erreichbar oder die HAL-Bibliothek wurde nicht korrekt eingebunden.

**Lösung:**
1. Prüfen, ob `Core/Inc/` im Include-Pfad liegt.
2. Prüfen, ob `main.h` die passende HAL-Header-Datei einbindet (z. B. `stm32f4xx_hal.h`).

### Überlauf nach ~49 Tagen

`millis()` gibt einen `uint32_t` zurück. Nach 2^32 ms (≈ 49,7 Tage) läuft der Zähler auf 0 zurück. Zeitdifferenzen mit vorzeichenloser Subtraktion sind dennoch korrekt:

```c
// Korrekt – funktioniert auch über den Überlauf hinweg:
if ((uint32_t)(millis() - lastTime) >= INTERVAL) { ... }

// Falsch – kann bei Überlauf negative Werte erzeugen:
if (millis() >= lastTime + INTERVAL) { ... }
```

## Versionsinformation prüfen

Die Versionsnummer ist als Makro in `millis.h` definiert:

```c
#define MILLIS_MAJOR  0
#define MILLIS_MINOR  0
#define MILLIS_PATCH  0
#define MILLIS_DEV    0
```

Diese Werte können zur Laufzeit abgefragt oder per `#if`-Direktive geprüft werden.
