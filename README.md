# Closed-Loop Speed Control System for DC Motor

An embedded closed-loop speed regulation system using a **Proportional-Integral (PI)** controller, an **Arduino Uno**, a **BTS7960 (43A)** H-Bridge driver, a **Hall Effect sensor** for feedback, and an **incremental rotary encoder** for setpoint manipulation.

---

## Key Features

- **PI Controller Logic**: Implements proportional-integral feedback to maintain RPM with dynamic load disturbance rejection ($\pm 2\%$ steady-state error).
- **Smooth Speed Ramping**: Software-level low-pass filter / ramp mechanism to eliminate abrupt inrush currents.
- **Bidirectional Control**: Forward (Quadrant 1) and Reverse (Quadrant 3) motoring controlled via Serial commands (`F`, `R`, `S`).
- **High-Current Drive**: Driven by an Infineon BTS7960 dual half-bridge driver capable of delivering up to 43A peak current.
- **Direct Feedback**: Single-pulse Hall Effect interrupt capturing microsecond periods (`micros()`) for real-time RPM derivation.

---

## System Architecture

[ Rotary Encoder ] ---> [ Arduino Uno (PI Controller) ] ---> [ BTS7960 Driver ] ---> [ RS-555 Motor ]
^                                                            |
|________________ [ Hall Effect Sensor ] <___________________|

### Block Diagram Breakdown
1. **User Input Stage**: Incremental quadrature rotary encoder setting dynamic target RPM.
2. **Control Stage**: Arduino Uno executing the PI algorithm and software smoothing.
3. **Power Driver Stage**: BTS7960 H-Bridge receiving PWM and direction logic.
4. **Actuator Stage**: High-torque 12V RS-555 DC motor.
5. **Sensing Stage**: Magnet disc mounted on motor shaft paired with a digital Hall Effect sensor triggered via external hardware interrupts (`D2`).

---

## Pin Configuration

| Arduino Uno Pin | Module / Component | Function |
| :--- | :--- | :--- |
| **D2 (INT0)** | Hall Effect Sensor OUT | Speed feedback (Hardware Interrupt) |
| **D3** | Rotary Encoder DT | Quadrature Data line |
| **D4** | Rotary Encoder CLK | Quadrature Clock line |
| **D5 (PWM)** | BTS7960 RPWM | Forward drive PWM (Q1) |
| **D6 (PWM)** | BTS7960 LPWM | Reverse drive PWM (Q3) |
| **D7** | BTS7960 R_EN | Right bridge enable (Active HIGH) |
| **D8** | BTS7960 L_EN | Left bridge enable (Active HIGH) |
| **5V / GND** | Logic Supply | Sensor & Driver logic supply |
| **External 12V** | Motor Power Rails | BTS7960 $B+ / B-$ motor supply |

---

## Control Algorithm

The motor's measured RPM is computed using period measurement:
$$\text{RPM} = \frac{60 \times 10^6}{\Delta t \, (\mu\text{s})}$$

The PI control output is evaluated as:
$$\text{PWM} = K_p \cdot e(t) + K_i \int e(t)\,dt + \text{BasePWM}$$

Where:
- $K_p = 0.18$
- $K_i = 0.09$
- $\text{BasePWM} = 50$
- **Anti-Windup**: Integral term clamped to $[0, 1000]$.

---

## Serial Command Interface

Open the Arduino Serial Monitor at **9600 baud**:

- `F` : Forward Motoring (Quadrant 1)
- `R` : Reverse Motoring (Quadrant 3)
- `S` : Coast / Stop
