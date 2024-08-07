<!--yml

category: 未分类

date: 2024-05-27 14:56:07

-->

# 构建 USB SNES 控制器

> 来源：[`blog.chybby.com/posts/building-a-usb-snes-controller`](https://blog.chybby.com/posts/building-a-usb-snes-controller)

`#include  <HID-Project.h>

#define  CLOCK_PIN 14

#define  LATCH_PIN 15

#define  DATA_PIN 16

#define  SNES_BUTTON_B 0

#define  SNES_BUTTON_Y 1

#define  SNES_BUTTON_SELECT 2

#define  SNES_BUTTON_START 3

#define  SNES_BUTTON_UP 4

#define  SNES_BUTTON_DOWN 5

#define  SNES_BUTTON_LEFT 6

#define  SNES_BUTTON_RIGHT 7

#define  SNES_BUTTON_A 8

#define  SNES_BUTTON_X 9

#define  SNES_BUTTON_L 10

#define  SNES_BUTTON_R 11

#define  SNES_BUTTON_UNDEF_1 12

#define  SNES_BUTTON_UNDEF_2 13

#define  SNES_BUTTON_UNDEF_3 14

#define  SNES_BUTTON_UNDEF_4 15

const  uint8_t num_buttons =  16;

// 将 SNES 按钮映射到 HID 游戏手柄按钮。

const  uint8_t snes_id_to_hid_id[] = { 2, 4, 7, 8, 0, 0, 0, 0, 1, 3, 5, 6, 10, 11, 12, 13 };

void  setup() {

Gamepad.begin();

pinMode(CLOCK_PIN, OUTPUT);

pinMode(LATCH_PIN, OUTPUT);

pinMode(DATA_PIN, INPUT);

digitalWrite(CLOCK_PIN, HIGH);

}

// 通过 HID 报告按钮状态。

void  reportButtons(bool  button_states[num_buttons]) {

// 十字键。

int8_t dpad_status = GAMEPAD_DPAD_CENTERED;

if (button_states[SNES_BUTTON_UP]) {

dpad_status = GAMEPAD_DPAD_UP;

if (button_states[SNES_BUTTON_LEFT]) {

dpad_status = GAMEPAD_DPAD_UP_LEFT;

} else  if (button_states[SNES_BUTTON_RIGHT]) {

dpad_status = GAMEPAD_DPAD_UP_RIGHT;

}

} else  if (button_states[SNES_BUTTON_DOWN]) {

dpad_status = GAMEPAD_DPAD_DOWN;

if (button_states[SNES_BUTTON_LEFT]) {

dpad_status = GAMEPAD_DPAD_DOWN_LEFT;

} else  if (button_states[SNES_BUTTON_RIGHT]) {

dpad_status = GAMEPAD_DPAD_DOWN_RIGHT;

}

} else  if (button_states[SNES_BUTTON_LEFT]) {

dpad_status = GAMEPAD_DPAD_LEFT;

} else  if (button_states[SNES_BUTTON_RIGHT]) {

dpad_status = GAMEPAD_DPAD_RIGHT;

}

Gamepad.dPad1(dpad_status);

Gamepad.dPad2(dpad_status);

for (uint8_t snes_id =  0; snes_id < num_buttons; snes_id++) {

if (snes_id >=  4  && snes_id <=  7) {

// 十字键。

continue;

}

if (button_states[snes_id]) {

Gamepad.press(snes_id_to_hid_id[snes_id]);

} else {

Gamepad.release(snes_id_to_hid_id[snes_id]);

}

}

}

void  loop() {

// 从控制器收集按钮状态信息。

// 发送数据锁存。

digitalWrite(LATCH_PIN, HIGH);

delayMicroseconds(12);

digitalWrite(LATCH_PIN, LOW);

delayMicroseconds(6);

bool button_states[num_buttons];

for (uint8_t id =  0; id < num_buttons; id++) {

// 采样按钮状态。

int button_pressed =  digitalRead(DATA_PIN) == LOW;

button_states[id] = button_pressed;

digitalWrite(CLOCK_PIN, LOW);

delayMicroseconds(6);

digitalWrite(CLOCK_PIN, HIGH);

delayMicroseconds(6);

}

// 更新 HID 按钮状态。

reportButtons(button_states);

Gamepad.write();

delay(16);

}`
