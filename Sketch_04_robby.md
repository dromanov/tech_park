# 04: Робби

Задаю скорость одного колеса:

```Cpp
#include <Dynamixel2Arduino.h>

const int led_pin = 14;         // LED PIN on OpenCM 9.04 board

#define DXL_SERIAL Serial3  // Serial3
                            
#define DEBUG_SERIAL Serial

// Информационный пин (DATA) в TTL-триаде GND-DATA-VDD на борде.
const uint8_t DXL_DIR_PIN = 22; // = 22

const float DXL_PROTOCOL_VERSION = 2.0;

// Мой мотор (`id` прошивается с помощью DYNAMIXEL Wizard 2.0).
const uint8_t DXL_ID = 2;

Dynamixel2Arduino dxl(DXL_SERIAL, DXL_DIR_PIN);

void setup() {
  // put your setup code here, to run once:
  pinMode(led_pin, OUTPUT);

  DEBUG_SERIAL.begin(57600);
  
  dxl.begin(57600); // В книге 1000000!
  dxl.setPortProtocolVersion(DXL_PROTOCOL_VERSION);

  dxl.torqueOff(DXL_ID);
  dxl.setOperatingMode(DXL_ID, OP_VELOCITY);
  dxl.torqueOn(DXL_ID);
  dxl.ping(DXL_ID);
} 

void loop() {
  digitalWrite(led_pin, HIGH);  // LED is OFF (?!)
  DEBUG_SERIAL.println("led off");
  dxl.setGoalVelocity(DXL_ID, 30);   // Не влияет на скорость поворота - смотреть макс. скорость/ускорение?
  dxl.setGoalPosition(DXL_ID, 818);
  delay(2000);

  // put your main code here, to run repeatedly:
  digitalWrite(led_pin, LOW);   // LED is ON (?!)
  DEBUG_SERIAL.println("led on");
  dxl.setGoalVelocity(DXL_ID, 0);
  dxl.setGoalPosition(DXL_ID, 205);
  delay(2000);
}
```
