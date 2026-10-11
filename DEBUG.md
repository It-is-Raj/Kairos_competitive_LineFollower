# Debug — LazyDragon Kairos Line Follower Controller

What to expect from the Kairos board and how to isolate common problems across
power, USB, motor drive, and analog inputs. For the hardware overview see
[`README.md`](README.md); for the board details see [`MANUAL.md`](MANUAL.md).

> **Status:** no firmware ships with this board yet, so firmware-specific
> symptoms are marked **TODO** and are written against the expected board
> described in `MANUAL.md`.

---

## Expected behavior

**Boot / startup:** with a charged LiPo on the XT30 connector, the power LED
lights and the 5 V rail comes up. Plugging in mini-USB enumerates a CH340 serial
port for programming and telemetry. Motors stay still until firmware drives
`nSLEEP` (`D10`) high and applies PWM.

**Normal operation:** the analog inputs on `A0..A7` are scanned, the control loop
computes left/right commands, and the two DRV8874 channels drive the motors in
PH/EN mode.

**Edge cases:**
- **Battery below the driver minimum:** motors cut out (4.5 V `VM` floor).
- **`nSLEEP` low:** both drivers are off and the motors coast.
- **Motor stall:** current rises and is reported on `IPROPI`; the driver may
  enter current regulation.

---

## Quick checks

- Measure the **battery voltage** at `CN1` (XT30) — within the 2-3S range.
- Measure **5 V** at the regulator output / logic rail.
- Confirm the board **enumerates over USB** (CH340 serial port appears).
- Confirm **`nSLEEP` (`D10`)** is driven high while running.
- Confirm the **analog-input FPC** is fully seated in `FPC1`.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| No USB device / cannot upload | Missing CH340 driver, bad mini-USB cable, or 12 MHz crystal fault | Install CH340 driver; try another cable; check `X5` |
| No 5 V rail | LM7805 input reversed/overvoltage or in thermal shutdown | Check polarity and battery voltage; let it cool; reduce load |
| Both motors dead | `nSLEEP` low or no PWM | Drive `D10` high; verify PWM on `D5`/`D6` |
| One motor dead | That channel's wiring or driver | Check the motor terminal and `U5`/`U6`; compare with the working side |
| Motors spin wrong way | Direction polarity | Swap the motor leads or invert `PH/IN2` in firmware |
| Board resets when motors start | Battery sag / supply noise | Use a charged, high-C battery; check bulk capacitors |
| Regulator very hot | Linear dropout loss at high `Vin` and load | Use a lower cell count or reduce 5 V load |
| Analog values stuck / all equal | FPC not seated or wrong channel mapping | Reseat `FPC1`; verify `A0..A7` order |

---

## Known limitations / errata

- **Linear 5 V regulation.** The LM7805 burns `(Vbat - 5 V) x I` as heat; on a
  3S pack with a heavy 5 V load it can get hot and thermally throttle.
- **No firmware published.** `code/` is empty; the board is inert until flashed.
- **`H1` pin 2** is a spare, unconnected pin (no net on the board).
- **Snap-off wings.** The `SafeToDrill_Area` wings are intended to be cut/trimmed
  to fit a chassis; trimming them changes the board outline.
- <!-- TODO: add any further errata discovered during bring-up. -->

---

## Support

Open an issue in the project repository, or contact
**contact@lazydragon.club**.
