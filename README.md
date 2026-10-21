# STM32 Millis

Eine in C implementierte Bibliothek, die auf STM32-Mikrocontrollern eine Arduino-ähnliche
Funktion `millis()` bereitstellt. Sie liefert die Zeit seit dem Systemstart in Millisekunden.
Die Zeitbasis ist der SysTick der STM32-HAL.

## Beschreibung

`millis()` gibt den Wert von `HAL_GetTick()` zurück. Die HAL erhöht diesen Zähler im
SysTick-Interrupt jede Millisekunde, sodass keine eigene Peripherie (Timer, Interrupt)
konfiguriert werden muss. Die Funktion erleichtert das Portieren von Arduino-Code und
Zeitmessungen ohne blockierendes Warten.

## Dateien

| Datei      | Beschreibung                                 |
|------------|----------------------------------------------|
| `millis.h` | Prototyp, Versions-Defines, Debug-Symbol     |
| `millis.c` | Implementierung                              |

## API

```c
uint32_t millis (void);    // Zeit seit Systemstart in ms
```

## Verwendung

### 1. Dateien einbinden

`millis.h` und `millis.c` in das STM32-Projekt kopieren und den Header einbinden:

```c
#include "millis.h"
```

### 2. Zeit messen

Eine Initialisierung ist nicht nötig, der SysTick wird von `HAL_Init()` gestartet.

```c
uint32_t start = millis();

while ((millis() - start) < 500)
{
	// Andere Aufgaben erledigen, bis 500 ms vergangen sind
}
```

## Hinweise

- Der Rückgabewert ist `uint32_t` und läuft nach ca. 49,7 Tagen über. Zeitdifferenzen in
  der Form `millis() - start` bleiben auch über den Überlauf hinweg korrekt.
- Die Auflösung hängt von der SysTick-Frequenz der HAL ab (Standard 1 kHz).

## Abhängigkeiten

- `main.h` – STM32 HAL (`HAL_GetTick()`)

## Lizenz

Dieses Projekt steht unter der [GPL-3.0 Lizenz](LICENSE).
