# Manual — LazyDragon Kairos Line Follower Controller

How the Kairos board is organized, how its motor drivers and analog input are
wired, and how to integrate it into a line-following robot. For the hardware
overview see [`README.md`](README.md); for troubleshooting see
[`DEBUG.md`](DEBUG.md).

> **Status:** this board does not ship with firmware yet. `code/` is empty, so
> every firmware-specific section below is a **specification / TODO**, not a
> description of existing code.

---

## Overview

Kairos is a single-board line-follower controller. An ATmega328P runs the
control loop, a CH340G bridges USB to the MCU's serial port, two DRV8874
H-bridges drive the left and right motors, and a 10-pin FPC brings in eight
analog inputs (`A0..A7`). A 2-3S LiPo powers the board through an XT30
connector; the motor drivers run directly from the battery while an LM7805
produces the 5 V logic rail.

---

## How it works

1. **Power.** The LiPo feeds the DRV8874 `VM` pins directly and the LM7805,
   which regulates 5 V for the MCU, CH340G, and logic.
2. **Analog input.** The eight analog inputs on `FPC1` land on `A0..A7`. The
   firmware scans them and computes a line position.
3. **Control.** The firmware computes left/right motor commands and drives each
   DRV8874 in **PH/EN** mode: PWM on `EN/IN1` for speed, `PH/IN2` for direction,
   and `nSLEEP` (`D10`) to enable the drivers.
4. **Telemetry / programming.** The CH340G exposes the MCU UART over mini-USB
   for flashing and for serial telemetry at the usual Arduino baud rates.

The DRV8874 also reports motor current proportionally on `IPROPI`; the board
provides `VREF`/`IPROPI` networks per channel so firmware can detect stalls and
load changes without a shunt resistor.

---

## Configuration

Hardware parameters (fixed by the board):

| Parameter | Value | Meaning |
|-----------|-------|---------|
| MCU | ATmega328P-AU @ 16 MHz | Arduino-compatible controller. |
| Analog inputs | `A0`..`A7` | Eight analog inputs on `FPC1`. |
| Motor A | `D5` (PWM), `D3` (DIR) | DRV8874 `U6`, PH/EN mode. |
| Motor B | `D6` (PWM), `D4` (DIR) | DRV8874 `U5`, PH/EN mode. |
| Driver enable | `D10` (`nSLEEP`) | Shared enable for both drivers. |
| Battery | 2-3S LiPo (XT30) | Motor supply; 5 V regulated. |
| User buttons | `D3`, `D4` | `SW3`, `SW2`. |

Firmware constants (e.g. `KP`, `KD`, base speed, control period) are **TODO**
until `code/` is published.

---

## Function / API reference

> **TODO:** no firmware is published yet. The functions below describe the
> expected board-support shape so the interface can be designed before code
> lands.

### `boardBegin()`
Expected: configure motor pins (`D5`/`D6`, `D3`/`D4`, `D10`), the analog-input ADC
scan, buttons, and serial at the chosen baud rate.

### `setMotor(uint8_t channel, int16_t command)`
Expected: map a signed command to `PH/IN2` direction and `EN/IN1` PWM duty for
the selected driver; `0` coasts.

### `enableDrivers(bool on)`
Expected: drive `nSLEEP` (`D10`) high to enable / low to sleep the DRV8874s.

### `readCurrent(uint8_t channel)`
Expected: sample the `IPROPI` output for load/stall detection.

---

## Output / data format

> **TODO:** provisional, to be finalized with the firmware.

The expected serial telemetry reports the calibrated input values, a line
position, and status counters at 115200 baud, for example:

```
<c0> <c1> ... <c7> pos=<p> status=<s> lost=<l> jct=<j> sat=<x> dt_us=<d> loops/s=<r>
```

| Field | Meaning |
|-------|---------|
| `c0..c7` | Calibrated input values, 0..1000. |
| `pos` | Line position, integer scaled x1000. |
| `status` | Pattern classification (on-line / edge / junction / lost). |
| `lost` / `jct` / `sat` | Line-loss, junction, and saturation counters. |
| `dt_us` / `loops/s` | Loop timing diagnostics. |

---

## Integration guide

- **Enable before driving.** Bring `nSLEEP` (`D10`) high and allow the DRV8874
  to power up before applying PWM.
- **Keep PWM in a sane range.** The DRV8874 supports high PWM frequencies; pick
  one above the audible band but below the driver's switching limits.
- **Use PH/EN, not independent mode.** The board wires `EN/IN1` and `PH/IN2`;
  configure the driver `PMODE` for PH/EN.
- **Watch battery sag.** Motors draw from the LiPo directly; ensure the supply
  stays within the 4.5–37 V driver range and above the LM7805 dropout under
  load.
- **Current sense scaling.** `IPROPI` output is scaled by `VREF` and the sense
  resistor network on the board; read the DRV8874 datasheet for the ratio.

---

## Examples

> **TODO:** untested reference sketch — will move into `code/` once firmware is
> written and verified.

```cpp
// Reference outline only — not yet tested.
const uint8_t PWM_A = 5,  DIR_A = 3;   // Motor A
const uint8_t PWM_B = 6,  DIR_B = 4;   // Motor B
const uint8_t NSLEEP = 10;

void setMotor(uint8_t pwmPin, uint8_t dirPin, int16_t cmd) {
  digitalWrite(dirPin, cmd >= 0 ? HIGH : LOW);
  analogWrite(pwmPin, min(abs(cmd), 255));
}

void setup() {
  pinMode(PWM_A, OUTPUT); pinMode(DIR_A, OUTPUT);
  pinMode(PWM_B, OUTPUT); pinMode(DIR_B, OUTPUT);
  pinMode(NSLEEP, OUTPUT);
  digitalWrite(NSLEEP, HIGH);          // enable drivers
}

void loop() {
  setMotor(PWM_A, DIR_A, 150);         // forward
  setMotor(PWM_B, DIR_B, 150);
}
```
