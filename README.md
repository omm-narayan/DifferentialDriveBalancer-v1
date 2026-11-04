# Differential Drive Balancer v1

Two-wheel self-balancing robot driven by stepper motors, running on an
STM32 NUCLEO-F446RE. Balances with a cascaded speed/angle controller,
can stand itself back up after tipping, and takes drive and tuning
commands over Bluetooth (HC-05).

## Hardware

| Part | Notes |
|---|---|
| NUCLEO-F446RE | Arduino framework via PlatformIO, flashed over the on-board ST-Link |
| ICM-20948 IMU | I2C at `0x69`, 400 kHz |
| 2x stepper motors + drivers | 3200 steps/rev (1/16 microstepping) |
| HC-05 | UART on `PC_11` (RX) / `PC_10` (TX), 115200 baud |

Stepper pins (Arduino header numbering):

| Signal | Pin |
|---|---|
| Left step / dir | D2 / D5 |
| Right step / dir | D4 / D7 |
| Enable (shared, LOW = on) | D8 |

## How it balances

- Loop runs at 200 Hz (`LOOP_PERIOD_US = 5000`).
- Tilt comes from a complementary filter (`CF_ALPHA = 0.98`) on the
  gyro and accelerometer, with gyro bias measured at boot.
- Outer loop: speed error (plus a small position-hold term) goes through
  a PI to give a target lean angle, clamped to 10°.
- Inner loop: PD on angle error and gyro rate gives wheel acceleration,
  integrated into a step rate.
- Past 70° the motors cut out. Between 25° and 70° a rise routine
  drives one way then the other to swing the robot back upright.

## Build and flash

```bash
pio run -t upload
pio device monitor
```

On boot, hold the robot still and upright for 3 seconds while the gyro
bias is measured. It then jogs forward and back once as a direction
check and starts balancing.

## Commands

Send over the HC-05 or the USB serial monitor, one per line.

| Command | Action |
|---|---|
| `F` `B` `L` `R` | Drive forward / back / left / right for 1.5 s |
| `S` | Stop |
| `ST` | Print status |
| `?` | Help |
| `p<val>` `d<val>` | Set angle P / D gain |
| `s<val>` `i<val>` | Set speed P / I gain |
| `o<val>` | Balance angle offset (degrees) |
| `f<val>` `b<val>` | Target speed forward / back |
| `t<val>` | Steering |
| `r` | Reset integrals |

## Other sketches in `src/`

`main-test.cpp` is the active firmware. The `.txt` files are earlier or
bench-test versions kept for reference; rename one to `.cpp` (and the
current one away) to build it.

- `motor-test.txt`: spins both steppers two turns to check wiring
- `imu-test.txt`: reads the IMU and steps toward the tilt
- `main-auto.txt`: balancing without Bluetooth
- `main-ble.txt`: earlier Bluetooth version
