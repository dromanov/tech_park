# Делаю движение под любым углом
Нашёл интересное поведение - когда делаю скорость 200 под углом $45^o$, получаю очень медленное движение. Отладочная печать говорит, что я пытаюсь поставить скорость (296, -14, -14, 296).

На серве зашит предел скорости 265 => мотор оставлял старое значение (которое у меня было 0 в покое или 200 при движении вперёд) и из состояния покоя я получал $(0, 0, 0, 0) \oplus (296, -14, -14, 296) \to (0, -14, -14, 0)$.

Подобрал максимальную скорость в 179 = 200 * 265 / 296 и круг нарисовался.

| V             | Скорость                                                  | Величина |
| ------------- | --------------------------------------------------------- | -------- |
| $V_{max}$     | Максимальная по прошивке                                  | 265      |
| $V_{test}$    | На которой калибровал всё                                 | 200      |
| $V_{wheel}^{max}(V = 200)$ | Максимальная на мотор при $\alpha = 45^o$ и $V=200$       | 296      |
| $=200 * 265 / 296$              | Скорость для $\alpha = 45^o$, дающая макс. на мотор в 265 | 179      |


```Cpp
#include <Dynamixel2Arduino.h>

// Проверяю, что одно колесо не двигает шасси из-за баланса сил

const int led_pin = 14;         // LED PIN on OpenCM 9.04 board

#define DXL_SERIAL Serial3  // Serial3

#define DEBUG_SERIAL Serial

#define PRINT(MSG)   DEBUG_SERIAL.print((MSG))
#define PRINTLN(MSG) DEBUG_SERIAL.println((MSG))

// Информационный пин (DATA) в TTL-триаде GND-DATA-VDD на борде.
const uint8_t DXL_DIR_PIN = 22; // = 22

const float DXL_PROTOCOL_VERSION = 2.0;

// Мой мотор (`id` прошивается с помощью DYNAMIXEL Wizard 2.0).
const int max_speed = 265;  // Cell #44 @ EEPROM, see datasheet.
const uint8_t DXL_ID_FR = 2,
              DXL_ID_BR = 3,
              DXL_ID_BL = 4,
              DXL_ID_FL = 5;

const uint8_t legs[4] = {
  DXL_ID_FL, DXL_ID_FR,
  DXL_ID_BL, DXL_ID_BR
};

// Замеренные на столе смещения при одинаковых скоростях вращения и отсутствии калибровки.
const double L_forward = 71.4 * 71.5,
             L_left    = 63.5 * 73.15;
const double V_perp_multiplier = L_forward / L_left;

Dynamixel2Arduino dxl(DXL_SERIAL, DXL_DIR_PIN);

// Forward:    set_velocity(V, V, V, V);
// Right:      set_velocity(-V, V, -V, V);
// Rotate CCW: set_velocity(V, V, -V, -V);
void set_velocity(int fr, int br, int bl, int fl)
{
  // Порядок как в массиве legs.
  // Знак переводит направление вращения в направление движения вперёд.
  int speed[] = {fl, -fr, bl, -br};

  int mx = max(max(abs(fl), abs(fr)), max(abs(bl), abs(br)));
  if (mx > max_speed) {
    PRINTLN("Speed exceeds the max possible, the servo will reject it! Doing corrections...");
    PRINT("Max possible: "); PRINTLN(max_speed);
    PRINTLN(fr); PRINTLN(fl); PRINTLN(br); PRINTLN(bl); 
    PRINTLN("----");
  }
  
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

void any_angle(int V, int angle_deg) {
  double V_parr = V * cos(angle_deg / 180.0 * PI),
         V_perp = V * sin(angle_deg / 180.0 * PI) * V_perp_multiplier;

  DEBUG_SERIAL.print("V_parr: "); DEBUG_SERIAL.println(V_parr);
  DEBUG_SERIAL.print("V_perp: "); DEBUG_SERIAL.println(V_perp);

  set_velocity(V_parr + V_perp, V_parr - V_perp, V_parr + V_perp, V_parr - V_perp);
}

void rotate_CCW(int V) {
  set_velocity(-V, -V, V, V);
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

double CCW[]    = {1};
double angles[] = {180 * 3 + 90};
double times[]  = {4000 * 3};

double calibrate_CCW_speed(void) {
  double ref_speed = 200;
  return angles[0] / times[0] * CCW[0] / ref_speed;
}

double angle = 0;

void loop() {
  int speed = 179; // <<< подогнал, чтобы максимальная скорость на мотор не выходила за 200
  int t_move = 4000 / 360,
      t_pause = 2000;

  digitalWrite(led_pin, HIGH);

  DEBUG_SERIAL.println("Forward");
  any_angle(speed, angle);     
  delay(t_move);
  angle += 1;
  
  /*
    rotate_CCW(speed);      delay(t_move);
    rotate_CCW(0);          delay(t_pause);
    rotate_CCW(-speed);     delay(t_move);
    rotate_CCW(0);          delay(t_pause);
  */
}
```

