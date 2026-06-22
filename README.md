# STM32 Millis
 
Eine leichtgewichtige C-Bibliothek für STM32-Mikrocontroller, die eine `millis()`-Funktion
im Arduino-Stil bereitstellt und die vergangene Zeit seit dem Systemstart in Millisekunden
zurückgibt.
 
## Beschreibung
 
Die Bibliothek kapselt `HAL_GetTick()` in einer einzigen Funktion und bietet damit eine
vertraute Zeitbasis für nicht-blockierende Verzögerungen, Intervallsteuerung und
Timeout-Erkennung. Da der Aufruf direkt an den HAL-Timer weitergeleitet wird, entsteht
kein zusätzlicher Overhead.
 
`millis()` setzt voraus, dass der SysTick-Timer auf eine Periode von 1 ms konfiguriert
ist – das ist die Standardeinstellung in STM32CubeMX-generierten Projekten.
 
## Dateien
 
| Datei       | Beschreibung                                                        |
|-------------|---------------------------------------------------------------------|
| `millis.h`  | Header mit Funktionsdeklaration und Versionsmakros                  |
| `millis.c`  | Implementierung – Wrapper um `HAL_GetTick()`                        |
 
## API
 
```c
uint32_t millis(void);
```
 
Gibt die seit dem Systemstart vergangene Zeit in Millisekunden zurück. Der Rückgabewert
ist ein `uint32_t` und läuft nach ca. 49,7 Tagen (2³² ms) auf 0 zurück.
 
### Versionsmakros
 
| Makro           | Beschreibung                                      |
|-----------------|---------------------------------------------------|
| `MILLIS_MAJOR`  | Major-Version – inkompatible API-Änderungen       |
| `MILLIS_MINOR`  | Minor-Version – neue abwärtskompatible Funktionen |
| `MILLIS_PATCH`  | Patch-Version – Fehlerbehebungen                  |
| `MILLIS_DEV`    | Entwicklungs-Flag (0 = Release, 1 = Entwicklung)  |
 
## Verwendung
 
### 1. Dateien einbinden
 
`millis.h` und `millis.c` in das STM32-Projekt kopieren und den Header einbinden:
 
```c
#include "millis.h"
```
 
### 2. Funktion aufrufen
 
```c
// Nicht-blockierende Verzögerung
uint32_t start = millis();
while (millis() - start < 1000) {
    // warte 1 Sekunde
}

// Periodische Aufgabe
#define INTERVAL_MS  500U

uint32_t lastToggle = 0;

void periodicTask(void) {
    if (millis() - lastToggle >= INTERVAL_MS) {
        lastToggle = millis();
        HAL_GPIO_TogglePin(LED_GPIO_Port, LED_Pin);
    }
}

// Timeout-Erkennung
uint32_t timeout = millis();
while (!sensorReady()) {
    if (millis() - timeout > 5000) {
        handleTimeout();
        break;
    }
}
```
 
## Hinweise
 
- Zeitdifferenzen müssen immer als `millis() - startTime` berechnet werden. Diese Form
  ist auch nach einem Überlauf des Zählers korrekt, solange die gemessene Zeitspanne
  2³² ms (≈ 49,7 Tage) nicht überschreitet.
- Der Ausdruck `millis() >= startTime + INTERVAL` ist bei einem Überlauf fehlerhaft
  und sollte nicht verwendet werden.
- Die Auflösung beträgt 1 ms. Feinere Zeitauflösungen erfordern einen separaten
  Timer (z. B. `DWT->CYCCNT`).
 
## Abhängigkeiten
 
- `main.h` – STM32 HAL (enthält `HAL_GetTick()`)
 
## Lizenz
 
Dieses Projekt steht unter der [GPL-3.0 Lizenz](LICENSE).
