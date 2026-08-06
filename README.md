# Mikey – ESP32-C3 Interactive Desk Robot

Mikey is a custom‑built interactive desk robot based on an ESP32‑C3.  
He can roam around a desk, avoid walls and edges, react to touch, play sounds, show expressive faces on an OLED display, and be controlled from a web dashboard over Wi‑Fi.

This repository contains the full source code and project details for Mikey.

---

## Features

- **ESP32‑C3 based brain** with Wi‑Fi AP mode and built‑in web server  
- **Expressive OLED face** with multiple emotions, blinking, saccades, and “breathing” animations  
- **Custom IR cliff & wall sensors** built from discrete IR transmitter & receiver LEDs + resistors  
- **Desk‑safe navigation**: detects walls and table edges, performs evasive maneuvers  
- **Touch interaction**: petting vs tapping triggers different reactions, moods, and sounds  
- **DFPlayer Mini audio**: voice lines, sound effects, dances, and a mini “story mode”  
- **RGB LED stick (HW160B)** for mood‑based lighting and stage effects in dances  
- **Multiple modes**:
  - Manual drive (via web dashboard)
  - Auto roam with idle personality
  - Action‑style behaviors (dances, stories, dialogues)
- **Web dashboard** (mobile‑friendly):
  - Mode selection (Manual / Auto / Action)
  - Manual drive controls
  - Trigger actions: chirp, sneeze, sleep, dialogues, mini story, dances
  - Live IR sensor values
  - Runtime adjustable IR thresholds + auto calibration
  - LED color picker

---

## Hardware

### Core components

- ESP32‑C3 development board  
- DRV8833 dual motor driver  
- 2 × N20 gear motors with wheels  
- Omni wheel (support / balance)  
- DFPlayer Mini MP3 module  
- Small speaker  

### Power & charging

- 3.7 V 500 mAh 14500 Li‑ion cell  
- TP4056 charging module  
- Slide switch for main power on/off  

### Sensors & interaction

- Custom IR cliff & wall sensors:
  - IR transmitter LEDs  
  - IR receiver (photodiode) LEDs  
  - 150 Ω and 10 kΩ resistors  
- Touch sensor (TTP223 or similar)  

### Display & lights

- 1.3" I2C OLED (SH1106 / SH110X compatible)  
- HW160B RGB LED stick (WS2812B‑type)  

### Mechanical

- Mounting brackets for N20 motors  
- 3D‑printed / custom body (painted sky blue)  
- Screws, glue, wiring, etc.  

> The ESP32‑C3 uses (almost) all GPIOs in this build, so there is effectively no room for extra sensors without redesigning the hardware.

---

## Pinout

### ESP32‑C3 GPIO mapping

**IR sensors**

- `GPIO 0` – Left IR receiver (cliff)  
- `GPIO 1` – Front IR receiver (wall)  
- `GPIO 2` – Right IR receiver (cliff)  

**Motors (DRV8833)**

- `GPIO 3` – MOTOR_A_IN1  
- `GPIO 4` – MOTOR_A_IN2  
- `GPIO 6` – MOTOR_B_IN1  
- `GPIO 5` – MOTOR_B_IN2  

**RGB LED (HW160B / WS2812B stick)**

- `GPIO 7` – DIN  

**OLED (I2C)**

- `GPIO 8` – SDA  
- `GPIO 9` – SCL  

**Touch sensor**

- `GPIO 10` – Touch input  

**DFPlayer Mini**

- `GPIO 20` – TX to DFPlayer RX (ideally through ~1 kΩ resistor)  
- `GPIO 21` – RX from DFPlayer TX  

**Power**

- All modules share a **common GND**.  
- Main 3.7 V battery node goes to:
  - ESP32 5V/VIN  
  - DRV8833 VM  
  - DFPlayer VCC  
  - OLED VCC  
  - Touch sensor VCC  
  - HW160B LED 5V  
- ESP32 **3.3 V** pin powers only the custom IR receivers/emitters (for cleaner analog readings).

---

## Audio files (DFPlayer)

All audio is stored on the DFPlayer in the `/mp3` folder using the `00XX.mp3` naming scheme.

Currently used tracks:

- `0001.mp3` – Boot: “Sys… System online. Hello world, I'm awake!”  
- `0002.mp3` – Short intro: “Hey, I am Mikey!”  
- `0003.mp3` – Full intro  
- `0004.mp3` – Wall alert  
- `0005.mp3` – Cliff alert  
- `0006.mp3` – Ouch / tapped  
- `0007.mp3` – Petting / long press  
- `0008.mp3` – Switched to manual mode  
- `0009.mp3` – Switched to auto mode  
- `0010.mp3` – Switched to action mode  
- `0011.mp3` – Dance 1 music  
- `0012.mp3` – Dance 2 music  
- `0013.mp3` – Dance 3 music  
- `0014.mp3` – Curious sound  
- `0015.mp3` – Happy sound  
- `0016.mp3` – Sneeze  
- `0017.mp3` – Sleep / snore  
- `0018.mp3` – Chirp / whistle  
- `0019.mp3` – Boing / squeak  
- `0020.mp3` – Dialogue 1  
- `0021.mp3` – Dialogue 2  
- `0022.mp3` – Dialogue 3  
- `0023.mp3` – “Wall lesson” line  
- `0024.mp3` – Story part 1 (curious)  
- `0025.mp3` – Story part 2 (realization)  
- `0026.mp3` – Story part 3 (happy)  
- `0027.mp3` – Short SFX “blip” (optional)  

All voice lines were generated with 11Labs using a consistent “Mikey” voice.

---

## Software overview

The firmware is written in Arduino‑style C++ for the ESP32‑C3.

### Modes

- `MANUAL` – Drive Mikey with the web dashboard.  
- `AUTO` – Autonomous roaming with random arcs, safety overrides, and idle personality.  
- `ACTION` – Conceptual “show” mode for bigger behaviors (dances, stories, etc.).  
  - Most action buttons are implemented as **global actions** and work in any mode.

### Face engine

- `FaceState` enum:  
  `NORMAL, DOWN, SURPRISED, PAIN, ANGRY, HAPPY, CURIOUS, SLEEP, SNEEZE, THINKING, SHOCKED`  
- Procedural animations:
  - Blinking  
  - Small eye saccades  
  - Speaking mouth shape  
  - Breathing LED effect

### Mood system

- `RobotMood` enum:  
  `MOOD_NEUTRAL, MOOD_HAPPY, MOOD_SHY, MOOD_PLAYFUL`  
- Mood influences:
  - Which faces/sounds are picked for idle behavior  
  - LED color theme  
- Mood decays back to neutral after a configured timeout.

### Movement & motor ramping

- `setMotors(targetL, targetR)` sets target speeds.  
- `updateMotorRamp()` runs regularly to ease speeds toward targets (no harsh start/stop).  
- AUTO mode favors gentle arcs and small turns instead of constant on‑spot spinning.

### Safety & sensors

- Custom IR sensors powered from 3.3 V for stable analog readings.  
- Left/right cliff detection and front wall detection.  
- In AUTO:
  - Detected danger triggers an evasion maneuver (reverse + turn).  
  - Different audio and face states for cliff vs wall.  
- In MANUAL:
  - Detected danger stops motors and plays alerts.

### Script / story system

- Small “script player” that runs sequences of:
  - `face + soundTrack + soundDuration + motorL + motorR + stepDuration`  
- Used for the mini story:
  - Step 1: curious face  
  - Step 2: “Is that a new pen?” with a small forward move  
  - Step 3: “This is awesome.” with a small backward move

---

## Web dashboard

When Mikey is powered on, the ESP32‑C3 starts a Wi‑Fi access point:

- **SSID:** `Mikey_Robot`  
- **Password:** `12345678` (change in code if needed)

Open a browser on your phone or laptop and go to:

- `http://192.168.4.1/`

### Dashboard sections

**Status & sensors**

- Alert text (e.g., “All Clear”, “ALERT: CLIFF EDGE!”)  
- Current mode and mood labels  
- Live IR sensor values:
  - Left cliff  
  - Front wall  
  - Right cliff  

**Modes**

- Buttons to switch between:
  - Manual  
  - Auto roam  
  - Action  

**Manual drive**

- Directional controls:
  - Forward / backward  
  - Left / right  
  - Stop  

**Global actions**

- Full intro / short intro  
- Chirp, sneeze, sleep  
- Dance 1 / Dance 2 / Dance 3  
- Dialogue 1 / Dialogue 2 / Dialogue 3  
- Mini story  
- LED color picker

**IR thresholds**

- Inputs for:
  - Left cliff threshold  
  - Right cliff threshold  
  - Wall threshold  
- “Auto Calibrate Left/Right” button:
  - Sets new thresholds as `currentReading + margin` for a new table color/surface.

---

## Building and flashing

### 1. Environment

- Arduino IDE or PlatformIO  
- ESP32 core for Arduino installed  
- Board selected as an ESP32‑C3 dev module

### 2. Required libraries (Arduino)

- `Adafruit_GFX`  
- `Adafruit_SH110X`  
- `Adafruit_NeoPixel`  
- `DFRobotDFPlayerMini`  
- `WiFi.h`, `WebServer.h` (from ESP32 core)

### 3. Steps

1. Clone this repository.  
2. Open the main `.ino` / `.cpp` file in Arduino IDE.  
3. Select the correct ESP32‑C3 board and COM port.  
4. Compile and upload the sketch to the ESP32‑C3.

### 4. DFPlayer setup

1. Format a microSD card as FAT32.  
2. Create an `mp3` folder.  
3. Copy your audio files as `0001.mp3`, `0002.mp3`, … matching the mapping above.  
4. Insert the SD card into the DFPlayer module.

---

## Tuning & configuration

Several parameters can be adjusted at the top of the code:

- Motor trim and `MOTOR_RAMP_STEP`  
- IR thresholds and `CLIFF_MARGIN`  
- Mood decay time (`MOOD_DECAY_MS`)  
- Idle/auto roam timing constants  
- LED breathing brightness range  

You can also edit the mini story in the `scriptStory[]` array:

- Change faces, sounds, motor speeds, or duration per step to create new behaviors without touching the main logic.

---

## Possible improvements / ideas

- Store simple stats in NVS/EEPROM:
  - Number of dances performed  
  - Times a cliff was avoided  
- Add more story scripts and dialogue sets  
- Web‑editable configuration for more parameters  
- Additional face “themes” or seasonal modes

---

## Credits

- Hardware design, integration, and project concept by **Altamash**  
- Voice lines generated with **11Labs**  
- OLED & NeoPixel control with **Adafruit** libraries  
- Audio playback using **DFRobot DFPlayer Mini** library  

If you build your own version of Mikey or remix this project, feel free to share it and tag me — I’d love to see what you create.
