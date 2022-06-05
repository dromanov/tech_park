# Скорость 200, вперёд-назад

## Скетч
```Cpp
#include <Dynamixel2Arduino.h>

// Проверяю, что одно колесо не двигает шасси из-за баланса сил

const int led_pin = 14;         // LED PIN on OpenCM 9.04 board

#define DXL_SERIAL Serial3  // Serial3

#define DEBUG_SERIAL Serial

// Информационный пин (DATA) в TTL-триаде GND-DATA-VDD на борде.
const uint8_t DXL_DIR_PIN = 22; // = 22

const float DXL_PROTOCOL_VERSION = 2.0;

// Мой мотор (`id` прошивается с помощью DYNAMIXEL Wizard 2.0).
const uint8_t DXL_ID_FR = 2,
              DXL_ID_BR = 3,
              DXL_ID_BL = 4,
              DXL_ID_FL = 5;

const uint8_t legs[4] = {
  DXL_ID_FL, DXL_ID_FR,
  DXL_ID_BL, DXL_ID_BR
};


Dynamixel2Arduino dxl(DXL_SERIAL, DXL_DIR_PIN);

void set_velocity(int fr, int br, int bl, int fl)
{
  // Порядок как в массиве legs. 
  // Знак переводит направление вращения в направление движения вперёд.
  int speed[] = {fl, -fr, bl, -br}; 

  for (int i = 0 ; i < 4 ; i++) {
    dxl.setGoalVelocity(legs[i], speed[i]);
  }
}

void setup() {
  DEBUG_SERIAL.begin(9600);

  // put your setup code here, to run once:
  pinMode(led_pin, OUTPUT);

  dxl.begin(57600); // В книге 1000000!
  dxl.setPortProtocolVersion(DXL_PROTOCOL_VERSION);

  for (auto id : legs) {
    dxl.torqueOff(id);
    dxl.setOperatingMode(id, OP_VELOCITY);
    dxl.torqueOn(id);
    dxl.ping(id);
  }
}

void loop() {
  int speed = 100;
  int time = 10000;
  
  digitalWrite(led_pin, HIGH);

  DEBUG_SERIAL.println("Forward");
  set_velocity(speed, speed, speed, speed);
  delay(time);

  set_velocity(0, 0, 0, 0);
  delay(2000);

  DEBUG_SERIAL.println("Backward");
  digitalWrite(led_pin, LOW);
  set_velocity(-speed, -speed, -speed, -speed);
  delay(time);

  set_velocity(0, 0, 0, 0);
  delay(2000);
}
```

## Результаты

## Вперёд-назад

> Вперёд: set_velocity(V, V, V, V);


| Скорость, units | Время, мсек | Расстояние, см |
| --------------- | ----------- | -------------- |
| 200             | 5000        | 90             |
| 100             | 10000       | 90             | 


## Влево-вправо

> Влево:   set_velocity(V, -V, V, -V);
> Нужен груз от 300 грамм, иначе вектор движения сильно скачет на ковре. На линолеуме надо больше. Надо будет калибровать поле и нагрузку на колёса.

| Скорость, units | Время, мсек | Расстояние, см |
| --------------- | ----------- | -------------- |
| 200             | 5000        | 70             | 
| 100             | 10000       |                |


## Буква Г
> Влево:   set_velocity(V, -V, V, -V);
> Нужен груз от 300 грамм, иначе вектор движения сильно скачет на ковре. На линолеуме надо больше. Надо будет калибровать поле и нагрузку на колёса. 
> **UPD**: ослабил раму, колёса легли равномерней [[Технопарк - дневник#^jan27]]

| Скорость, units | Время, мсек | Расстояние, см |
| --------------- | ----------- | -------------- |
| 200, лево       | 4000        | 63.5           |
| 200, вперёд     | 4000        | 71.4           | 

[[speed_calibration_200_horse]]