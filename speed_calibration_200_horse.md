Links: [[Технопарк - дневник]] [[speed_calibration_200_forward]]

# Калибровка скорости и коррекция анизотропии parr vs perp
## Скетч
Сделал замеры смещения вдоль и поперёк на скорости 200 и времени 4000 msec (это старый замер [[speed_calibration_200_forward#Буква Г]]). Забил его и получил скетч:
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

// Замеренные на столе смещения при одинаковых скоростях вращения и отсутствии калибровки.
const double L_forward = 71.4,
             L_left    = 63.5;
const double V_perp_multiplier = L_forward / L_left;

Dynamixel2Arduino dxl(DXL_SERIAL, DXL_DIR_PIN);

// Forward: set_velocity(V, V, V, V);
// Right: set_velocity(-V, V, -V, V);
// Rotate CCW:   set_velocity(V, V, -V, -V);
void set_velocity(int fr, int br, int bl, int fl)
{
  // Порядок как в массиве legs.
  // Знак переводит направление вращения в направление движения вперёд.
  int speed[] = {fl, -fr, bl, -br};

  for (int i = 0 ; i < 4 ; i++) {
    dxl.setGoalVelocity(legs[i], speed[i]);
  }
}

void forward(int V) {
  set_velocity(V, V, V, V);
}

void left(int V) {
  V *= V_perp_multiplier;
  set_velocity(V, -V, V, -V);
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
  int speed = 200;
  int time = 4000;

  digitalWrite(led_pin, HIGH);

  DEBUG_SERIAL.println("Forward");
  left(speed);      delay(time);
  left(0);          delay(2000);
  forward(speed);   delay(time);
  forward(0);       delay(2000);
  forward(-speed);  delay(time);
  forward(0);       delay(2000);
  left(-speed);     delay(time);
  left(0);          delay(2000);
}
```

## Буква Г - v2
> Рама ослаблена, добавил два самореза в правый передний серв, груз - болты [[Технопарк - дневник#^jan27]]

| Скорость, units | Время, мсек | Расстояние, см |
| --------------- | ----------- | -------------- |
| 200, лево       | 4000        | 73.15          |
| 200, вперёд     | 4000        | 71.5           |

Делаю коррекцию скорости, вторая итерация.

| Скорость, units | Время, мсек | Расстояние, см |
| --------------- | ----------- | -------------- |
| 200, лево       | 4000        | 71.8           |
| 200, вперёд     | 4000        | 70.8           |

Диагональ: 96.0 см $\ne \sqrt{parr^2 + perp^2} = 100.836$ 

