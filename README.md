# RC-car-version1
using fleshy and arduino uno
<div align="center">

# Robo-Soccer — Version 2

**Four-wheel RC soccer robot · Arduino Uno · FlySky FS-i6 · BTS7960**

[![Arduino](https://img.shields.io/badge/Arduino-Uno-00979D?style=flat-square&logo=arduino&logoColor=white)](https://www.arduino.cc/)
[![Language](https://img.shields.io/badge/Language-C%2B%2B-00599C?style=flat-square&logo=cplusplus&logoColor=white)]()
[![RC](https://img.shields.io/badge/RC-FlySky%20FS--i6-E8491D?style=flat-square)]()
[![Status](https://img.shields.io/badge/Status-Field%20Tested-2ea44f?style=flat-square)]()

</div>

---

### What this is

A four-wheel differential drive robot controlled over FlySky PWM radio.
Each side has **two motors wired in parallel** to a single BTS7960 driver — the driver sees them as one combined load and pushes current through both simultaneously. No code change is required versus a two-motor build.

The main problem with most beginner code is `pulseIn()` — it reads one channel, blocks for 25 ms, then tries to read the second. By then that pulse is long gone, so steering never responds.

This code reads **both channels at the same time** using a tight polling loop. No blocking. No missed pulses.

```
CH2  (throttle) ──┬──► Left  BTS7960 #1 ──► Left  Motor A ┐ (parallel)
                  │                      ──► Left  Motor B ┘
CH4  (steering) ──┘──► Right BTS7960 #2 ──► Right Motor A ┐ (parallel)
                                         ──► Right Motor B ┘
```

---

### Hardware

| Qty | Part |
|:---:|------|
| 1 | Arduino Uno |
| 1 | FlySky FS-i6 transmitter + FS-iA6B receiver |
| 2 | BTS7960 43 A H-bridge motor driver |
| 4 | DC gear motors (6 V – 12 V) |
| 1 | 7.4 V 2S or 11.1 V 3S LiPo battery |
| 1 | Buck converter — output set to 5 V (for Arduino logic) |

> With two motors in parallel per driver, the BTS7960 carries twice the current. Use at least **18 AWG wire** between driver and motors (16 AWG preferred for high-stall motors) and add heatsinks to the BTS7960 boards.

---

### Wiring

**Receiver → Arduino**

| Receiver | Arduino | What |
|:---:|:---:|---|
| CH2 | `A0` | Throttle — forward / back |
| CH4 | `A1` | Steering — left / right |
| GND | `GND` | Common ground (required) |
| +5V | `5V` | Receiver power |

> CH4 is the VrA knob by default. To use a stick instead:
> `Menu → Functions → Aux. channels → Channel 4 → assign axis`

---

**Arduino → Left BTS7960**

| Arduino | BTS7960 |
|:---:|:---:|
| `D5` | `RPWM` |
| `D6` | `LPWM` |
| `D7` | `R_EN` |
| `D8` | `L_EN` |
| `5V` | `VCC` |
| `GND` | `GND` |

---

**Arduino → Right BTS7960**

| Arduino | BTS7960 |
|:---:|:---:|
| `D9` | `RPWM` |
| `D10` | `LPWM` |
| `D11` | `R_EN` |
| `D12` | `L_EN` |
| `5V` | `VCC` |
| `GND` | `GND` |

---

**Motor wiring — parallel connection (Left side)**

Both left motors share the same BTS7960 output terminals:

| BTS7960 #1 | Left Motor A | Left Motor B |
|:---:|:---:|:---:|
| `M+` | `+` terminal | `+` terminal |
| `M−` | `−` terminal | `−` terminal |

**Motor wiring — parallel connection (Right side)**

Both right motors share the same BTS7960 output terminals:

| BTS7960 #2 | Right Motor A | Right Motor B |
|:---:|:---:|:---:|
| `M+` | `+` terminal | `+` terminal |
| `M−` | `−` terminal | `−` terminal |

> Simply connect both motors' `+` wires together to `M+`, and both `−` wires together to `M−`. The driver does not need any configuration change.

---

**Power**

| From | To |
|---|---|
| Battery `+` | `B+` on both BTS7960 boards |
| Battery `−` | `B−` on both BTS7960 boards |
| Battery `+` | Buck converter input |
| Buck converter output (5 V) | Arduino `5V` |
| Battery `−` | Arduino `GND` — tie all grounds together |
| BTS7960 `M+` / `M−` | Both motors on that side (in parallel) |

> Never power motors from the Arduino 5 V pin. Always use the buck converter for logic and connect battery power directly to the BTS7960 B+ / B−.

---

### Channel map

```
CH1   right stick  left/right    — not used
CH2   right stick  up/down       — THROTTLE → A0
CH3   left stick   up/down       — not used
CH4   VrA / axis   left/right    — STEERING → A1
```

---

### Calibration

Open `soccer_bot.ino` and update these six values at the top to match your receiver:

```cpp
const int CH2_MIN    =  997;
const int CH2_CENTER = 1496;
const int CH2_MAX    = 1988;

const int CH4_MIN    =  997;
const int CH4_CENTER = 1496;
const int CH4_MAX    = 1988;
```

To find the real values: enable `Serial.begin(115200)`, print `ch2Raw` and `ch4Raw` in `loop()`, open Serial Monitor, and move each stick to its extremes and centre. Write down what you see.

---

### How the drive mixing works

```
left  speed = throttle + steering
right speed = throttle − steering
```

| Stick | Left wheels (A+B) | Right wheels (A+B) |
|---|:---:|:---:|
| Forward | fwd | fwd |
| Backward | rev | rev |
| Steer right | fwd | rev |
| Steer left | rev | fwd |
| Forward + right | faster | slower |
| Forward + left | slower | faster |

If one side spins backwards, swap its `M+` and `M−` wires on the BTS7960. No code change needed. If only **one motor** of a parallel pair spins backwards, swap that individual motor's wires before they join at the terminal.

---

### Upload

```bash
# 1. Clone
git clone https://github.com/YOUR_USERNAME/robo-soccer-v2.git

# 2. Open in Arduino IDE
File → Open → soccer_bot/soccer_bot.ino

# 3. Board + port
Tools → Board → Arduino Uno
Tools → Port  → your port

# 4. Upload
Ctrl + U
```

Power-on order: **transmitter first**, then robot. Wait for receiver LED to go solid before moving sticks.

---

### Troubleshooting

| Problem | Fix |
|---|---|
| No movement at all | Check CH2 → A0, CH4 → A1, receiver GND |
| Steering does nothing | Make sure you are using `readChannels()`, not `pulseIn()` |
| Motors twitch at rest | Raise `DEADBAND` to 50–70 |
| One side reversed | Swap `M+` / `M−` on that BTS7960 |
| One motor in a pair reversed | Swap that individual motor's wires before the parallel junction |
| Creeps with sticks centred | Re-calibrate `CH2_CENTER` / `CH4_CENTER` |
| Signal drops under load | Add 470 µF cap across Arduino 5 V and GND |
| BTS7960 overheats | Add heatsink; two parallel motors double the current draw — check stall current |
| Arduino resets on motor start | Use separate power paths; shared buck converter for logic only |
| One motor of a pair slower | Motors are not perfectly matched — swap for a closer-spec pair or accept the small imbalance |

---

<div align="center">
Built for the field.<br/>
If this helped, leave a star. (try to copy this if you can >__<)
</div>
