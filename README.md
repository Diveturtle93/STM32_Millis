# STM32 Millis

Eine in C implementierte Bibliothek, die auf STM32-Mikrocontrollern eine Arduino-ähnliche
Funktion `millis()` bereitstellt. Sie liefert die Zeit seit dem Systemstart in Millisekunden.
Die Zeitbasis ist der Tick-Zähler der STM32-HAL.

## Beschreibung

`millis()` gibt den Wert von `HAL_GetTick()` zurück. Die HAL erhöht diesen Zähler im
Interrupt ihrer Zeitbasis jede Millisekunde, sodass keine eigene Peripherie (Timer, Interrupt)
konfiguriert werden muss. Die Zeitbasis wird von `HAL_Init()` gestartet. In STM32CubeMX ist
das unter „System Core → SYS → Timebase Source“ der SysTick oder ein Timer (z. B. bei
FreeRTOS), `millis()` funktioniert mit beiden. Die Funktion erleichtert das Portieren von
Arduino-Code und Zeitmessungen ohne blockierendes Warten.

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

`millis.h` und `millis.c` in das STM32-Projekt kopieren und den Header einbinden. Außerhalb
der STM32CubeIDE muss `millis.c` zusätzlich im Makefile bzw. in der `CMakeLists.txt` als
Quelldatei eingetragen werden.

```c
#include "millis.h"
```

### 2. Zeit messen

Eine Initialisierung ist nicht nötig, die Zeitbasis wird von `HAL_Init()` gestartet.

```c
uint32_t start = millis();

while ((millis() - start) < 500)
{
	// Andere Aufgaben erledigen, bis 500 ms vergangen sind
}
```

### 3. Periodische Aufgaben und Timeouts

```c
#define INTERVAL_MS							500U
#define TIMEOUT_MS							5000U

uint32_t lastToggle = 0;

// In der Hauptschleife: LED alle 500 ms umschalten
if ((millis() - lastToggle) >= INTERVAL_MS)
{
	lastToggle = millis();
	HAL_GPIO_TogglePin(LED_GPIO_Port, LED_Pin);
}

// Auf einen Sensor warten, nach 5 s abbrechen
uint32_t start = millis();

while (!sensorReady())
{
	if ((millis() - start) >= TIMEOUT_MS)
	{
		handleTimeout();
		break;
	}
}
```

## Hinweise

- Der Rückgabewert ist `uint32_t` und läuft nach ca. 49,7 Tagen (2³² ms) über. Zeitdifferenzen
  in der Form `millis() - start` bleiben auch über den Überlauf hinweg korrekt, solange die
  gemessene Zeitspanne kürzer als 49,7 Tage ist.
- Vergleiche in der Form `millis() >= start + INTERVAL` sind beim Überlauf falsch und dürfen
  nicht verwendet werden.
- Der Wert ist immer in Millisekunden. Die Schrittweite hängt von `HAL_TICK_FREQ_DEFAULT` in
  `stm32xxxx_hal_conf.h` ab: Bei `HAL_TICK_FREQ_1KHZ` (Standard) steigt er um 1 ms, bei
  `HAL_TICK_FREQ_100HZ` in 10-ms-Schritten.
- Für eine feinere Auflösung als 1 ms eignet sich der Zykluszähler aus
  [STM32_DataWatchpointTrigger](https://github.com/Diveturtle93/STM32_DataWatchpointTrigger).
- Bleibt `millis()` auf 0 stehen, wurde `HAL_Init()` nicht aufgerufen oder der Interrupt der
  Zeitbasis ist gesperrt.
- Das Lesen des 32-Bit-Zählers ist auf Cortex-M atomar, `millis()` darf daher auch in
  Interrupts aufgerufen werden. In einer ISR mit höherer Priorität als die Zeitbasis zählt
  der Wert aber nicht weiter, Warteschleifen auf `millis()` hängen dort fest.

## Abhängigkeiten

- `main.h` – STM32 HAL (`HAL_GetTick()`)
- `stdint.h` – `uint32_t`

## Lizenz

Dieses Projekt steht unter der [GPL-3.0 Lizenz](LICENSE).
