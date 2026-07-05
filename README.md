# Project 2: Automated Irrigation Controller (Actuator Logic)

**Decode Labs — IoT Department Internship Program**
**Author:** Noor Fatima

---

## Overview

An automated, closed-loop feedback system that activates mechanical hardware based on real-time environmental input. The controller continuously monitors soil moisture with an analog sensor, applies threshold logic to decide whether the soil is too dry, and switches a relay-controlled 5V water pump on/off accordingly — with no manual intervention required.

## Goal

Build a closed-loop irrigation system that:
- Reads analog input from a **Soil Moisture Sensor**
- Applies an explicit **threshold logic gate** to decide if the soil is too dry
- Drives a **5V relay module** to switch a water pump actuator on/off
- Continuously repeats this read → evaluate → act cycle

## Key Skills

| Skill | Description |
|---|---|
| Analog-to-Digital Conversion (ADC) | Converts the sensor's analog voltage into a usable 0–1023 digital value |
| Electronic Relay Mechanics | Uses a low-current digital signal to safely switch a high-current pump circuit |
| Closed-Loop Threshold Logic | Compares live sensor data against a defined threshold to make an autonomous decision |

## Hardware Components

- Arduino Uno
- Soil Moisture Sensor
- 5V Relay Module
- 5V DC Water Pump
- External 5V DC Power Supply (for the pump circuit)
- Breadboard + jumper wires

## Wiring Summary

| Signal | From | To |
|---|---|---|
| Power (5V) | Arduino 5V | Breadboard (+) rail |
| Ground | Arduino GND | Breadboard (–) rail |
| Sensor Power | Breadboard (+) | Sensor VCC |
| Sensor Ground | Breadboard (–) | Sensor GND |
| Analog Signal | Sensor AOUT | Arduino A0 |
| Relay Power | Breadboard (+) / (–) | Relay VCC / GND |
| Relay Control | Arduino D7 | Relay IN |
| Pump Circuit | External 5V Supply → Relay COM/NO → Pump → Supply (–) | — |

> The relay electrically isolates the low-power Arduino control signal from the higher-current pump circuit, which is powered by a separate external 5V supply.

## How It Works

1. **Read (ADC):** `analogRead(A0)` samples the sensor's voltage and returns a 0–1023 value. Dry soil → high value; wet soil → low value.
2. **Convert:** The raw value is mapped to a 0–100% moisture reading using calibration constants for dry and wet extremes.
3. **Threshold Gate:** If `moisturePercent < MOISTURE_THRESHOLD_PERCENT`, the soil is considered dry.
4. **Actuate:** Digital pin D7 goes HIGH, energizing the relay coil, which closes the COM–NO contact and powers the pump.
5. **Close the Loop:** As the pump adds water, moisture rises; once it crosses back above the threshold, the same logic automatically turns the relay off — no manual reset needed.

## Arduino Sketch

```cpp
const int SOIL_SENSOR_PIN = A0;
const int RELAY_PIN       = 7;

const int DRY_ADC_VALUE = 1023;
const int WET_ADC_VALUE = 300;
const int MOISTURE_THRESHOLD_PERCENT = 40;

const unsigned long READ_INTERVAL_MS = 30000;
unsigned long lastReadTime = 0;

void setup() {
  pinMode(RELAY_PIN, OUTPUT);
  digitalWrite(RELAY_PIN, LOW);
  Serial.begin(9600);
}

void loop() {
  unsigned long now = millis();
  if (now - lastReadTime >= READ_INTERVAL_MS) {
    lastReadTime = now;

    int rawValue = analogRead(SOIL_SENSOR_PIN);
    int moisturePercent = map(rawValue, DRY_ADC_VALUE, WET_ADC_VALUE, 0, 100);
    moisturePercent = constrain(moisturePercent, 0, 100);

    if (moisturePercent < MOISTURE_THRESHOLD_PERCENT) {
      digitalWrite(RELAY_PIN, HIGH);   // Dry -> pump ON
    } else {
      digitalWrite(RELAY_PIN, LOW);    // Wet -> pump OFF
    }
  }
}
```

## Calibration Notes

- `DRY_ADC_VALUE` and `WET_ADC_VALUE` should be re-measured for your specific sensor and soil, since resistive moisture sensors vary between units.
- `MOISTURE_THRESHOLD_PERCENT` controls sensitivity: too low risks under-watering, too high risks over-watering.

## Testing (Tinkercad Simulation)

- Drag the simulated Soil Moisture Sensor to different moisture levels to exercise both branches of the threshold logic.
- Use the Serial Monitor to confirm the raw ADC value, computed moisture %, and relay state on each cycle.
- Watch for the relay's click sound/LED indicator to confirm switching before checking the pump circuit.

## Suggested Improvements

- Add hysteresis (a second, higher threshold) so the relay doesn't rapidly toggle near the boundary value.
- Log readings over time to tune the threshold more precisely for the target soil type.

---
*Prepared as part of the Decode Labs IoT Internship Program.*
