# 🌾 Automatic Field Watering System

**IoT-enabled automated irrigation using NodeMCU (ESP8266)**

An automatic watering system that reads soil moisture and switches a DC water pump ON only when the soil is dry. It saves water by watering based on real soil condition, not on a fixed schedule.

**Author:** Pilla Joshi

---

## 📌 Table of Contents

- [Features](#-features)
- [How It Works](#-how-it-works)
- [Hardware Required](#-hardware-required)
- [Software Required](#-software-required)
- [Pin Connections](#-pin-connections)
- [Threshold and Calibration](#-threshold-and-calibration)
- [Sample Code](#-sample-code)
- [Test Results](#-test-results)
- [Limitations](#-limitations)
- [Future Enhancements](#-future-enhancements)
- [Conclusion](#-conclusion)

---

## ✨ Features

- 🤖 Fully automatic pump control, with no manual monitoring
- 💧 Waters only when the soil is genuinely dry
- 📡 Reads soil moisture every **1 second**
- ⚙️ Pump speed set with **PWM** (speed 200)
- 🛡️ L298 motor driver protects the NodeMCU from pump current
- 💻 Live status on the Arduino IDE serial monitor (115200 baud)
- 📶 Wi-Fi ready: the ESP8266 can be upgraded through firmware only

---

## ⚙️ How It Works

```
Soil Probe → Sensor Module → NodeMCU (A0) → Motor Driver (L298) → DC Water Pump
                                  │
                                  └──→ Serial Monitor
```

1. 📡 **Sense:** The soil probe and sensor module give an analog voltage. Higher reading means drier soil.
2. 🧠 **Decide:** The NodeMCU compares the reading with the threshold (**550**).
3. 💦 **Act:** The motor driver turns the pump ON or OFF.

| Soil Reading | Soil Condition | Pump | Serial Output |
| --- | --- | --- | --- |
| `> 550` | Dry | ON (PWM 200) | `SOIL DRY – MOTOR ON` |
| `≤ 550` | Wet | OFF | `SOIL WET – MOTOR OFF` |

---

## 🧰 Hardware Required

| Component | Purpose |
| --- | --- |
| NodeMCU ESP8266 (ESP-12E) | Main controller |
| Soil moisture sensor (resistive probe) | Measures soil moisture |
| L298-type motor driver module | Drives the pump (speed and direction) |
| DC water pump / motor (6–12 V) | Delivers water |
| 9 V battery | Separate power for the pump |
| Water reservoir (small metal container) | Water supply for bench testing |
| Jumper wires | Connections |
| Micro-USB cable (data-capable) | Power, programming and serial link |
| PC | Compile, upload and monitor |

---

## 💻 Software Required

| Item | Setting |
| --- | --- |
| IDE | Arduino IDE with ESP8266 core (Boards Manager) |
| Board | NodeMCU 1.0 (ESP-12E Module) |
| CPU frequency | 80 MHz |
| Flash size | 4 MB (default) |
| Upload speed | 115200 |
| Serial monitor baud | 115200 |
| Extra libraries | None |

> ⚠️ If the serial monitor shows unreadable characters, check that its baud rate matches the firmware (115200).

---

## 🔌 Pin Connections

| Signal | NodeMCU Pin | Direction | Notes |
| --- | --- | --- | --- |
| Soil sensor analog output | `A0` | Input | Only analog pin on the board |
| Motor driver ENB (speed) | `D1` | Output (PWM) | `analogWrite` 0–255 |
| Motor driver IN3 | `D2` | Output | Direction control |
| Motor driver IN4 | `D3` | Output | Direction control |
| Motor driver power | 9 V battery +/− | Power | Separate from logic supply |
| Ground | `GND` | Reference | Common ground for all modules |

> 🔗 **Important:** Connect the grounds of the NodeMCU, sensor module, motor driver and battery together.

> 🛡️ When the pump is stopped, both `IN3` and `IN4` are held LOW. This avoids an undefined H-bridge state.

---

## 🎚️ Threshold and Calibration

The threshold of **550** was set by testing with the probe and soil used in development. For a different probe or soil, recalibrate:

1. Put the probe in **dry soil** and note the serial reading (dry reference).
2. Water the soil to the ideal level, wait a few minutes, and note the reading (wet reference).
3. Set the threshold between the two values, closer to the dry reference so the pump turns on slightly early.
4. Update the constant in the code and upload again.
5. Move the probe between dry and wet several times to confirm clean switching.

**Things that change readings:** soil type (clay vs sand), fertiliser, probe depth and contact, and supply voltage.

---

## 🧪 Sample Code

A simple sketch that follows the documented logic. Adjust the pump direction (`IN3`/`IN4`) to match your wiring.

```cpp
// Automatic Field Watering System - NodeMCU ESP8266

#define SOIL_PIN   A0
#define ENB        D1   // PWM speed
#define IN3        D2   // direction
#define IN4        D3   // direction

const int THRESHOLD = 550;   // dry if reading > 550
const int PUMP_SPEED = 200;  // PWM value (0-255)

void setup() {
  Serial.begin(115200);
  pinMode(ENB, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);

  // Start with pump OFF
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, LOW);
  analogWrite(ENB, 0);
}

void loop() {
  int soilValue = analogRead(SOIL_PIN);
  Serial.print("Soil Value: ");
  Serial.print(soilValue);
  Serial.print("  ");

  if (soilValue > THRESHOLD) {
    // Soil is dry -> pump ON
    digitalWrite(IN3, HIGH);
    digitalWrite(IN4, LOW);
    analogWrite(ENB, PUMP_SPEED);
    Serial.println("SOIL DRY - MOTOR ON");
  } else {
    // Soil is wet -> pump OFF
    digitalWrite(IN3, LOW);
    digitalWrite(IN4, LOW);
    analogWrite(ENB, 0);
    Serial.println("SOIL WET - MOTOR OFF");
  }

  delay(1000);
}
```

---

## 📊 Test Results

Tested on the bench with the serial monitor open at 115200 baud.

| Test Condition | Reading | Pump | Serial Status |
| --- | --- | --- | --- |
| Dry soil / open air | `> 550` | ON (continuous PWM) | `SOIL DRY – MOTOR ON` |
| Probe in water reservoir | `≤ 550` | OFF | `SOIL WET – MOTOR OFF` |

**Observations**

- ⚡ The pump changed state on the very next sampling cycle after the probe was moved
- 🤝 Pump action and serial status matched in every trial
- ✅ No unintended intermediate states were seen
- 🌡️ No abnormal current draw or heating in the driver during the short test

---

## ⚠️ Limitations

- 🧪 Resistive probes corrode over time (capacitive probes are better for long-term use)
- 🔁 A single fixed threshold has no hysteresis, so the pump can toggle near the boundary
- 📉 The ESP8266 has only one ADC channel, so multiple zones need an external multiplexer or ADC
- 🔧 The pump runs at one fixed speed
- 💾 No data storage: readings are lost when the PC is disconnected
- 🌦️ No weather or rain forecast is used

---

## 🚀 Future Enhancements

- [ ] Add hysteresis with two thresholds (ON and OFF) to stop flickering
- [ ] Average several ADC samples to reduce noise
- [ ] Send data over Wi-Fi (MQTT or HTTP) to ThingSpeak or Blynk
- [ ] Use multiple soil probes for different field zones
- [ ] Connect a weather forecast API to skip watering before rain
- [ ] Power the probe from a GPIO pin only during measurement to reduce corrosion
- [ ] Switch to a capacitive soil probe
- [ ] Add solar charging for off-grid field use

---

## ✅ Conclusion

The system was designed, built, programmed and tested successfully. It reliably tells dry soil from wet soil and controls the DC pump through the motor driver. It shows a complete embedded control loop: **analog sensing → threshold decision → PWM motor actuation**. Because the ESP8266 has built-in Wi-Fi, this project can grow into a remote, cloud-connected irrigation controller with firmware changes only.

---

## 📬 Contact

**Pilla Joshi**
Feedback and ideas are welcome! 🙏
