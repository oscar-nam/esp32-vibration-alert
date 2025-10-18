# ESP32 Vibration Alert System

This project uses an ESP32 microcontroller with an MPU6050 accelerometer and an OLED display to detect vibrations and trigger alerts via LED and buzzer.

## Features
- Real-time vibration monitoring
- OLED display visualization
- LED and buzzer alert when vibration exceeds threshold

## Hardware
- ESP32
- MPU6050 accelerometer
- OLED display (SSD1306)
- LED (connected to pin 25)
- Buzzer (connected to pin 26)

## How It Works
- The MPU6050 measures acceleration in 3 axes.
- The magnitude of vibration is calculated.
- If the vibration exceeds a threshold (2g), the LED lights up and the buzzer sounds.

## Wiring
- MPU6050: SDA to GPIO21, SCL to GPIO22
- OLED: I2C address 0x3C
- LED: GPIO25
- Buzzer: GPIO26

## Setup
1. Install required libraries:
   - `Adafruit_GFX`
   - `Adafruit_SSD1306`
   - `MPU6050`
2. Upload the code to your ESP32
3. Open Serial Monitor at 115200 baud

## License
MIT
