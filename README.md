s# Clap-Controlled RGB Lighting System with LCD Status Feedback

An interactive hardware and firmware project built using an Arduino Uno R3. This system detects distinct acoustic clap patterns, evaluates them within a timing window, translates them into custom RGB color states via PWM, and provides real-time status feedback through an LCD1602 display.

---

## 🚀 Features

* **Acoustic Pattern Detection:** Utilizes an LM393 sound sensor module to capture sharp sound spikes (claps) via digital thresholding.
* **Smart Clap Counting & Debouncing:** Implements a time-window algorithm (`millis()`) combined with software debouncing to accurately count consecutive claps while filtering out ambient noise and mechanical signal bounce.
* **Dynamic PWM Color Mixing:** Controls an HW-479 common-cathode RGB LED module to smoothly transition between Red, Green, Blue, and White based on input frequency.
* **Real-Time UI Feedback:** Interfaces with an LCD1602 display in 4-bit mode to display active clap counts and current color states instantly.

---

## 🛠️ Hardware Components

* Arduino Uno R3
* HW-479 RGB LED Module (Common Cathode with internal resistors)
* LM393 Sound Sensor Module
* LCD1602 Display Module
* 10kΩ Potentiometer (for LCD contrast adjustment)
* 830-Point Breadboard & Jumper Wires

---

## 📊 Wiring & Pinout Reference

| Component | Pin / Terminal | Arduino Pin / Rail | Notes |
| :--- | :--- | :--- | :--- |
| **Sound Sensor** | `+` (VCC) | 5V Rail | Shared power rail |
| **Sound Sensor** | `G` (GND) | GND Rail | Common ground |
| **Sound Sensor** | `D0` (Digital Out) | **Digital Pin 8** | Clap trigger signal |
| **HW-479 RGB LED** | `-` (Ground) | Direct GND | Independent ground path |
| **HW-479 RGB LED** | `R` (Red) | **Digital Pin 9** | PWM Color Control |
| **HW-479 RGB LED** | `G` (Green) | **Digital Pin 10** | PWM Color Control |
| **HW-479 RGB LED** | `B` (Blue) | **Digital Pin 11** | PWM Color Control |
| **LCD1602** | RS, E, D4–D7 | **Pins 7, 6, 5, 4, 3, 2** | 4-bit data mode |
| **LCD1602** | V0 (Contrast) | Potentiometer Wiper | Contrast tuning |

---

## 💻 Arduino Firmware Source Code

```cpp
#include <LiquidCrystal.h>

// Initialize LCD pins (RS, E, D4, D5, D6, D7) matching our wiring: 7, 6, 5, 4, 3, 2
LiquidCrystal lcd(7, 6, 5, 4, 3, 2);

// Pin Definitions
const int soundPin = 8;     // Sound sensor DO connected to Digital Pin 8
const int redPin = 9;       // HW-479 RGB Red pin
const int greenPin = 10;    // HW-479 RGB Green pin
const int bluePin = 11;     // HW-479 RGB Blue pin

// Clap tracking variables
int clapCount = 0;
unsigned long firstClapTime = 0;
const unsigned long detectionWindow = 1000; // 1 second window to count claps
boolean lastSoundState = HIGH;
unsigned long lastDebounceTime = 0;
const unsigned long debounceDelay = 50;     // Debounce delay to prevent false triggers

void setup() {
  // Set up pins
  pinMode(soundPin, INPUT);
  pinMode(redPin, OUTPUT);
  pinMode(greenPin, OUTPUT);
  pinMode(bluePin, OUTPUT);

  // Set up the LCD's number of columns and rows:
  lcd.begin(16, 2);
  
  // Initial display message
  lcd.clear();
  lcd.print("Clap Project Ready");
  lcd.setCursor(0, 1);
  lcd.print("Clap to change!");
  
  // Start with LED off
  setColor(0, 0, 0);
}

void loop() {
  int currentSoundState = digitalRead(soundPin);
  unsigned long currentTime = millis();

  // Detect a falling edge (sound spike / clap) with basic debouncing
  if (currentSoundState == LOW && lastSoundState == HIGH && (currentTime - lastDebounceTime) > debounceDelay) {
    lastDebounceTime = currentTime;
    
    if (clapCount == 0) {
      firstClapTime = currentTime;
      clapCount = 1;
    } else {
      clapCount++;
    }
  }
  lastSoundState = currentSoundState;

  // Once the time window closes, evaluate the total claps counted
  if (clapCount > 0 && (currentTime - firstClapTime > detectionWindow)) {
    handleClaps(clapCount);
    clapCount = 0; // Reset for the next round
  }
}

// Function to handle color changes based on total claps
void handleClaps(int count) {
  lcd.clear();
  lcd.setCursor(0, 0);
  
  if (count == 1) {
    setColor(255, 0, 0); // Red
    lcd.print("Clap: 1");
    lcd.setCursor(0, 1);
    lcd.print("Color: RED");
  } 
  else if (count == 2) {
    setColor(0, 255, 0); // Green
    lcd.print("Clap: 2");
    lcd.setCursor(0, 1);
    lcd.print("Color: GREEN");
  } 
  else if (count == 3) {
    setColor(0, 0, 255); // Blue
    lcd.print("Clap: 3");
    lcd.setCursor(0, 1);
    lcd.print("Color: BLUE");
  } 
  else {
    setColor(255, 255, 255); // White for 4 or more claps
    lcd.print("Claps: ");
    lcd.print(count);
    lcd.setCursor(0, 1);
    lcd.print("Color: WHITE");
  }
}

// Helper function to set HW-479 RGB LED color
void setColor(int redValue, int greenValue, int blueValue) {
  analogWrite(redPin, redValue);
  analogWrite(greenPin, greenValue);
  analogWrite(bluePin, blueValue);
}


https://github.com/user-attachments/assets/171aef43-78b4-4901-bc8e-7ed5cb937e0a



