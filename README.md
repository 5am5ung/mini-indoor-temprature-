# Mini Indoor Climate Gauge (Darwin, NT Edition)

An Arduino Nano-based indoor environmental monitoring unit tailored specifically for the tropical climate of Darwin, Northern Territory[cite: 2]. The system tracks indoor temperature, relative humidity, and barometric pressure to optimize Panasonic air conditioner settings for efficiency, comfort, and active mold prevention[cite: 2].

---

## 1. System Overview

* **Microcontroller:** Arduino Nano (ATmega328P) powered via 5V USB wall outlet[cite: 2].
* **Dedicated Sensing Roles:**
  * **Temperature Source:** Sensed exclusively via an externally mounted **XC3702 (BMP180)** to avoid internal enclosure heat soak and utilize its higher-precision calibration curve[cite: 2].
  * **Relative Humidity Source:** Sensed via the **XC4520 (DHT)** module[cite: 2].
  * **Barometric Pressure Source:** Sensed via the **XC3702 (BMP180)**[cite: 2].
  * **Ambient Light Sensing:** Sensed via a **Light Dependent Resistor (LDR)** to govern automatic sleep dimming.

---

## 2. Darwin Environmental Operating Logic

### Temperature Targets (Air Conditioning Optimization)
* **Target Setpoint:** Maintain indoor temperatures between **24°C and 27°C** during AC operation to provide comfort without causing temperature shock when moving outdoors[cite: 2].
* **Efficiency Constraint:** In Northern Territory conditions, lowering the AC setpoint each degree below 24°C increases running costs by roughly 10%[cite: 2].
* **Display Format:** Rounded to the nearest whole integer (e.g., `28°C`).

### Humidity Targets (Mold & Comfort Control)
* **Target Range:** **40% to 60% Relative Humidity (RH)**, with a bias below 50% to prevent active mold propagation and dust mite proliferation[cite: 2].
* **Wet Season Alert:** Outdoor humidity frequently surges between 80% and 95% from November to April[cite: 2]. When indoor humidity rises above 60% (and particularly 80%+), AC "Dry" mode or active dehumidification is required[cite: 2].

### Barometric Pressure & Weather Warning Thresholds
* **Fair Weather:** High pressure (**> 1015 hPa**) signals stable, clear dry-season conditions[cite: 2].
* **Rain Alert:** Pressure dropping below **1010 hPa** indicates atmospheric instability; triggers a `[RAIN!]` alert on the display[cite: 2].
* **Tropical Storm / Monsoon:** Below **1005 hPa** signifies a developing monsoon trough or active tropical low[cite: 2].
* **Cyclone Warning:** Below **1000 hPa** flags a severe tropical low or nearby cyclonic weather system[cite: 2].

---

## 3. Hardware Bill of Materials (BOM)

| Item | Model / Part # | Interface | Purpose |
| :--- | :--- | :--- | :--- |
| **Microcontroller** | Arduino Nano v3.0 | USB 5V | Central processing unit[cite: 2] |
| **Character LCD** | Jaycar QP5521 (16x2 Blue/White) | Direct 4-bit Parallel | Displays primary readings and weather flags[cite: 2] |
| **Barometer / Temp** | Jaycar XC3702 (BMP180) | Hardware I2C (A4/A5) | Temperature and atmospheric pressure monitoring[cite: 2] |
| **Humidity Sensor** | Jaycar XC4520 (DHT11/DHT22) | Single-bus Digital (D2) | Relative humidity tracking[cite: 2] |
| **RGB LED Strip** | Jaycar XC4380 (WS2812B, 8 LEDs) | Single-wire Data (D3) | Visual zone status indicators (Temp & Humidity)[cite: 2, 3] |
| **Light Sensor** | 5mm LDR + 10kΩ Resistor | Analog In (A0) | Ambient room light monitoring for auto-dimming |
| **Potentiometer** | 10kΩ Trimpot | Analog to LCD Pin 3 | Manual LCD character contrast adjustment |
| **Resistors** | 220Ω, 330Ω, 10kΩ | Passive | Backlight limit, LED data buffer, and LDR divider |

---

## 4. Hardware Pin Mapping

| Component | Pin / Function | Arduino Nano Pin | Voltage | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **QP5521 LCD**[cite: 2] | Pin 1 (VSS) | GND | 0V | Ground |
| | Pin 2 (VDD) | 5V | 5V | Logic Power |
| | Pin 3 (V0) | Potentiometer Wiper | 0–5V | 10kΩ pot ends connected to 5V and GND |
| | Pin 4 (RS) | **D8** | 5V | Register Select |
| | Pin 5 (R/W) | GND | 0V | Hardwired to GND (Write mode) |
| | Pin 6 (E) | **D9** | 5V | Enable Strobe |
| | Pins 7–10 (D0–D3) | *NC* | — | Unconnected (4-bit mode) |
| | Pin 11 (D4) | **D4** | 5V | Data bit 4 |
| | Pin 12 (D5) | **D5** | 5V | Data bit 5 |
| | Pin 13 (D6) | **D6** | 5V | Data bit 6 |
| | Pin 14 (D7) | **D7** | 5V | Data bit 7 |
| | Pin 15 (Backlight A) | **D10** | 0–5V (PWM) | Connected via 220Ω series resistor |
| | Pin 16 (Backlight K) | GND | 0V | Backlight Cathode to Ground |
| **XC3702 (BMP180)**[cite: 2] | VCC | 3.3V or 5V | 3.3V / 5V | Shared power rail |
| | GND | GND | 0V | Common ground |
| | SDA | **A4** | 3.3V / 5V | Hardware I2C Data bus |
| | SCL | **A5** | 3.3V / 5V | Hardware I2C Clock bus |
| **XC4520 (DHT)**[cite: 2] | VCC | 5V | 5V | Shared power rail |
| | GND | GND | 0V | Common ground |
| | DATA | **D2** | 5V | Add 4.7kΩ–10kΩ pull-up to 5V if unbuffered |
| **XC4380 Strip**[cite: 2, 3] | +5V | 5V | 5V | Shared power rail |
| | GND | GND | 0V | Common ground |
| | DIN | **D3** | 5V | Connected via 330Ω series buffer resistor |
| **LDR Sensor** | Leg 1 | 5V | 5V | Connected to 5V rail |
| | Leg 2 | **A0** | 0–5V | Node between LDR and 10kΩ pull-down resistor |
| | 10kΩ Resistor | GND | 0V | Pull-down resistor leg to ground |

---

## 5. Visual Indicators & Strip Configuration

### LCD Layout (16x2)
* **Line 1:** `T:26°C  H:54%   ` (Whole-number readings from BMP180 and DHT)[cite: 2].
* **Line 2:** `1008hPa [RAIN!]  ` (Pressure reading with dynamic storm/monsoon flags)[cite: 2].

### WS2812B 8-LED Color Mapping
The LED strip divides into two 4-LED status meters[cite: 2, 3]:

| LED Index | Source | Metric / Range | Color | RGB Value |
| :---: | :---: | :---: | :---: | :---: |
| **0** | BMP180 | Temperature < 24°C | Blue | `(0, 0, 255)`[cite: 2, 3] |
| **1** | BMP180 | Temperature 24°C – 27°C (Optimal Baseline) | Green | `(0, 255, 0)`[cite: 2, 3] |
| **2** | BMP180 | Temperature 28°C – 32°C | Yellow | `(255, 255, 0)`[cite: 2, 3] |
| **3** | BMP180 | Temperature > 32°C | Red | `(255, 0, 0)`[cite: 2, 3] |
| **4** | XC4520 | Humidity < 40% | Orange | `(255, 64, 0)`[cite: 2, 3] |
| **5** | XC4520 | Humidity 40% – 60% (Optimal Target) | Green | `(0, 255, 0)`[cite: 2, 3] |
| **6** | XC4520 | Humidity 60% – 80% | Light Blue | `(64, 128, 255)`[cite: 2, 3] |
| **7** | XC4520 | Humidity > 80% | Blue | `(0, 0, 255)`[cite: 2, 3] |

---

## 6. Night-Light & Ambient Dimming Logic

* **Daylight Mode (Normal Lighting):**
  * WS2812B brightness scales up to `120 / 255`.
  * LCD backlight PWM runs up to `255 / 255` for high-contrast sunlight readability.
* **Night-Light Mode (Dark Bedroom):**
  * When the LDR detects dark ambient conditions, the display does **not** shut off completely.
  * **LCD Backlight:** Dims to a low glow (`PWM 6 / 255`) so characters remain legible without casting light across the room.
  * **WS2812B LEDs:** Dims to minimum intensity (`brightness = 1 / 255`) to keep color status identifiable without disturbing sleep.

---

## 7. Required Arduino Libraries

* `LiquidCrystal` (Standard Arduino built-in)
* `Adafruit_BMP085` (Required for BMP180 / XC3702 compatibility)
* `DHT sensor library` by Adafruit
* `Adafruit_NeoPixel` by Adafruit
* `Wire` (Standard I2C built-in)
