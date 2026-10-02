# ESP32 2-Radio / 3-Antenna Relay Antenna Switch R2A3

A DIY antenna selector for **two radios and three antennas**, controlled by an ESP32 designed by AI and KD3CSR.

The system provides:

- 2 radio inputs
- 3 antenna outputs
- Independent antenna selection for each radio
- 8 × 12 V DPDT relays
- ESP32 Wi-Fi control
- Physical 4×3 keypad control
- 8 status LEDs
- SAFE / ALL-OFF function
- Break-before-make switching
- Automatic grounding of unselected antennas
- Remote antenna relay box connected by CAT6
- No computer required for physical operation

The design is intended primarily for HF and lower-frequency applications. RF construction quality, relay suitability, power handling, spacing, enclosure construction and grounding should be appropriate for the frequency and power being used.

---

# 1. System Overview

The system is divided into two boxes.

## Remote control box
The remote control box contains:

- ESP32-WROOM development board
- ULN2803APG Darlington driver
- 4×3 membrane keypad
- 8 × red LEDs
- 8 × 2.2 kΩ LED resistors
- 8 × 10 kΩ ULN input pull-down resistors
- 12 V DC input
- Main power switch
- 12 V → 5 V buck converter
- 100 nF bypass capacitor
- CAT6 cable to the antenna box
- One separate +12 V wire to the antenna box

### Power Wiring

The 12 V input enters the remote control box and passes through the main power switch. The switched +12 V is then distributed to the relay driver, LEDs, buck converter, and the separate +12 V feed going to the antenna box.

```text
12 V INPUT
    │
    ▼
MAIN POWER SWITCH
    │
    ├──────────────► +12 V Remote Box
    │
    ├──────────────► ULN2803 pin 10 (COM)
    │
    ├──────────────► LED resistors
    │
    ├──────────────► 12 V → 5 V buck converter
    │                    │
    │                    └──► ESP32 5V/VIN
    │
    └──────────────► +12 V wire to Antenna Box


12 V NEGATIVE
    │
    ├──────────────► ULN2803 pin 9 (GND)
    ├──────────────► ESP32 GND
    ├──────────────► buck converter GND
    └──────────────► 10 kΩ ULN input pull-downs


ESP32 GPIO ─────────► ULN2803 input
                         │
                         │
                       10 kΩ
                         │
                         ▼
                       GND

```                       
## Antenna selector box

Contains:

- 8 × 12 V DPDT relays
- Radio 1 SO-239
- Radio 2 SO-239
- Antenna 1 SO-239
- Antenna 2 SO-239
- Antenna 3 SO-239
- RF ground/chassis bus
- CAT6 termination
- +12 V relay supply bus

The relay box contains no electronics other than the relay coils. Therefore it does **not** require a separate DC ground wire from the remote box.

---

# 2. Relay Type

The relays used in this build are JQX-18F(T)-2 / LY2-type 12 V DPDT relays.

The relay terminal numbering confirmed by physical testing is:

```text
Pole 1:
5 = COM
1 = NC
3 = NO

Pole 2:
6 = COM
2 = NC
4 = NO

Coil:
7 = +12 V
8 = coil return
```

Therefore:

```text
COIL:       7 ───── coil ───── 8

POLE 1:    1 NC
             \
              5 COM
             /
            3 NO

POLE 2:    2 NC
             \
              6 COM
             /
            4 NO
```

When the relay is OFF:

```text
5 ↔ 1
6 ↔ 2
```

When the relay is ON:

```text
5 ↔ 3
6 ↔ 4
```

Always verify relay numbering against the actual relay before soldering a large batch.

---

# 3. Relay Assignment

There are eight relays.

| Relay | Function |
|---|---|
| K1 | Radio 1 → Antenna 1 |
| K2 | Radio 2 → Antenna 1 |
| K3 | Radio 1 → Antenna 2 |
| K4 | Radio 2 → Antenna 2 |
| K5 | Radio 1 → Antenna 3 |
| K6 | Radio 2 → Antenna 3 |
| K7 | Radio 1 master RF connection |
| K8 | Radio 2 master RF connection |

The relay arrangement is:

```text
                    ┌── K1 ── Antenna 1
Radio 1 ── K7 ─ R1 BUS
                    ├── K3 ── Antenna 2
                    └── K5 ── Antenna 3


                    ┌── K2 ── Antenna 1
Radio 2 ── K8 ─ R2 BUS
                    ├── K4 ── Antenna 2
                    └── K6 ── Antenna 3
```

---

# 4. RF Wiring — K7

K7 is the Radio 1 master relay.

Use:

```text
K7 pin 5 = Radio 1 center
K7 pin 3 = R1 BUS
K7 pin 1 = unused
K7 pins 2,4,6 = unused
```

This is important.

Because pin 5 is COM and pin 3 is NO:

### K7 OFF

```text
pin 5 ↔ pin 1
```

Radio 1 is isolated from the R1 BUS.

### K7 ON

```text
pin 5 ↔ pin 3
```

Therefore:

```text
Radio 1 center
     ↓
K7 pin 5
     ↓
K7 pin 3
     ↓
R1 BUS
```

Do not use K7 pin 1 as the R1 BUS connection.

---

# 5. RF Wiring — K8

K8 is the Radio 2 master relay.

Use:

```text
K8 pin 5 = Radio 2 center
K8 pin 3 = R2 BUS
K8 pin 1 = unused
K8 pins 2,4,6 = unused
```

When K8 is ON:

```text
Radio 2 center
     ↓
K8 pin 5
     ↓
K8 pin 3
     ↓
R2 BUS
```

---

# 6. K1 — Radio 1 / Antenna 1

K1 connections:

```text
K1 pin 5 → R1 BUS
K1 pin 3 → Antenna 1 center
K1 pin 6 → Antenna 1 ground
K1 pin 2 → K2 pin 6

K1 pins 1 and 4 → unused
```

The signal path when K1 is ON is:

```text
R1 BUS
  ↓
K1 pin 5
  ↓
K1 pin 3
  ↓
A1 center
```

The grounding contact is:

```text
A1 shell
  ↓
K1 pin 6
```

When K1 is OFF, pin 6 connects to pin 2, which continues through K2 to the RF ground bus.

---

# 7. K2 — Radio 2 / Antenna 1

K2:

```text
K2 pin 5 → R2 BUS
K2 pin 3 → Antenna 1 center
K2 pin 6 → K1 pin 2
K2 pin 2 → RF ground bus

K2 pins 1 and 4 → unused
```

Signal path when K2 is ON:

```text
R2 BUS
  ↓
K2 pin 5
  ↓
K2 pin 3
  ↓
A1 center
```

Grounding path when K2 is OFF:

```text
A1
 ↓
K1 pin 6
 ↓
K1 pin 2
 ↓
K2 pin 6
 ↓
K2 pin 2
 ↓
RF ground
```

---

# 8. K3 — Radio 1 / Antenna 2

```text
K3 pin 5 → R1 BUS
K3 pin 3 → Antenna 2 center
K3 pin 6 → Antenna 2 ground
K3 pin 2 → K4 pin 6

K3 pins 1 and 4 → unused
```

---

# 9. K4 — Radio 2 / Antenna 2

```text
K4 pin 5 → R2 BUS
K4 pin 3 → Antenna 2 center
K4 pin 6 → K3 pin 2
K4 pin 2 → RF ground bus

K4 pins 1 and 4 → unused
```

---

# 10. K5 — Radio 1 / Antenna 3

```text
K5 pin 5 → R1 BUS
K5 pin 3 → Antenna 3 center
K5 pin 6 → Antenna 3 ground
K5 pin 2 → K6 pin 6

K5 pins 1 and 4 → unused
```

---

# 11. K6 — Radio 2 / Antenna 3

```text
K6 pin 5 → R2 BUS
K6 pin 3 → Antenna 3 center
K6 pin 6 → K5 pin 2
K6 pin 2 → RF ground bus

K6 pins 1 and 4 → unused
```

---

# 12. RF Grounding System

All SO-239 shells should ultimately belong to the common RF ground/chassis system.

Connect:

```text
R1 SO-239 shell ─┐
R2 SO-239 shell ─┤
A1 SO-239 shell ─┤
A2 SO-239 shell ─┤── RF ground / chassis
A3 SO-239 shell ─┘
```

The switched antenna-ground system is:

```text
A1 shell → K1 pin 6
K1 pin 2 → K2 pin 6
K2 pin 2 → RF ground

A2 shell → K3 pin 6
K3 pin 2 → K4 pin 6
K4 pin 2 → RF ground

A3 shell → K5 pin 6
K5 pin 2 → K6 pin 6
K6 pin 2 → RF ground
```

Do **not** connect K1/K3/K5 pin 6 directly to the DC supply negative.

These are RF switching contacts.

---

# 13. What Happens to an Unselected Antenna?

An important feature of this design is that an unselected antenna is grounded.

For example, with everything OFF:

```text
A1 center
   ↓
K1 pin 3 is open from pin 5

A1 grounding network
   ↓
K1 pin 6
   ↓
K1 pin 2
   ↓
K2 pin 6
   ↓
K2 pin 2
   ↓
RF ground
```

Therefore, with A1 unselected:

```text
A1 center ↔ A1 shell = continuity
```

This is **intentional**, not a fault.

Likewise:

```text
A2 center ↔ A2 shell = continuity
A3 center ↔ A3 shell = continuity
```

when those antennas are unselected.

---

# 14. Selected Antenna

When R1 selects A1:

```text
K7 ON
K1 ON
```

The RF path becomes:

```text
Radio 1
   ↓
K7
   ↓
R1 BUS
   ↓
K1
   ↓
A1 center
```

At the same time K1's grounding contact changes state, so:

```text
A1 center ↔ A1 shell
```

should become OPEN.

Thus:

```text
R1 center ↔ A1 center = continuity

A1 center ↔ A1 shell = open
```

---

# 15. RF Wire Construction

For short internal RF connections, ordinary copper wire can be used.

Recommended:

- 0.75 mm² copper for short internal RF bus connections
- 1.5 mm² is electrically usable but is unnecessarily bulky
- Short pieces of good coax can also be used
- Keep RF connections short and direct
- Avoid long loops
- Keep RF wiring away from ESP32 and digital wiring where practical

Examples:

```text
K7 pin 3 → R1 BUS
K8 pin 3 → R2 BUS

K1/K3/K5 pin 5 → R1 BUS
K2/K4/K6 pin 5 → R2 BUS
```

0.75 mm² insulated copper is suitable for these short internal runs.

For antenna connector center pins, direct soldered connections are preferred.

For SO-239 shells and the RF ground system, short wide braid or copper strap is preferable to long thin wires.

---

# 16. Remote Control Box

The remote box contains:

```text
12 V input
   ↓
Main power switch
   ↓
+12 V BUS
```

The +12 V bus supplies:

```text
ULN2803 pin 10
LED resistors
Buck converter
Antenna-box +12 V feed
```

The PSU negative goes to the remote GND bus.

The remote GND bus connects to:

```text
ULN2803 pin 9
ESP32 GND
Buck converter output -
Keypad GND
```

---

# 17. ESP32 Power

Use the buck converter to reduce 12 V to approximately 5 V.

```text
12 V BUS
   ↓
Buck IN+

GND BUS
   ↓
Buck IN-

Buck OUT+
   ↓
ESP32 5V/VIN

Buck OUT-
   ↓
ESP32 GND
```

The ESP32 GND and ULN pin 9 must share the same electrical ground.

---

# 18. ULN2803APG

ULN2803APG pin arrangement:

```text
Inputs:
1  2  3  4  5  6  7  8

Outputs:
18 17 16 15 14 13 12 11

Pin 9  = GND
Pin 10 = COM
```

Connect:

```text
ULN pin 9  → remote GND
ULN pin 10 → +12 V
```

The COM pin is connected to +12 V for the internal flyback diode arrangement.

---

# 19. ESP32 → ULN2803 Relay Control

The relay GPIO assignment is:

| Relay | ESP32 | ULN input | ULN output |
|---|---:|---:|---:|
| K1 | GPIO13 | 1 | 18 |
| K2 | GPIO14 | 2 | 17 |
| K3 | GPIO18 | 3 | 16 |
| K4 | GPIO19 | 4 | 15 |
| K5 | GPIO21 | 5 | 14 |
| K6 | GPIO22 | 6 | 13 |
| K7 | GPIO23 | 7 | 12 |
| K8 | GPIO25 | 8 | 11 |

---

# 20. Relay Coil Wiring

Each relay gets +12 V on pin 7.

The ULN2803 controls pin 8.

```text
K1 pin 7 → +12 V
K1 pin 8 → ULN pin 18

K2 pin 7 → +12 V
K2 pin 8 → ULN pin 17

K3 pin 7 → +12 V
K3 pin 8 → ULN pin 16

K4 pin 7 → +12 V
K4 pin 8 → ULN pin 15

K5 pin 7 → +12 V
K5 pin 8 → ULN pin 14

K6 pin 7 → +12 V
K6 pin 8 → ULN pin 13

K7 pin 7 → +12 V
K7 pin 8 → ULN pin 12

K8 pin 7 → +12 V
K8 pin 8 → ULN pin 11
```

The ULN turns a relay ON by pulling pin 8 toward ground.

---

# 21. CAT6 Connection Between Boxes

The remote box sends the eight switched relay returns through CAT6.

Only **one +12 V wire** is additionally sent to the antenna box.

There is no separate DC ground wire.

CAT6 assignment used in this build:

| Relay | CAT6 conductor |
|---|---|
| K1 | Orange |
| K2 | Orange/White |
| K3 | Green |
| K4 | Green/White |
| K5 | Blue |
| K6 | Blue/White |
| K7 | Brown |
| K8 | Brown/White |

The same colour must terminate at the corresponding relay coil pin 8.

For example:

```text
Remote:

ULN pin 18
   ↓
Orange CAT6
   ↓
Antenna box
   ↓
K1 pin 8
```

And:

```text
ULN pin 12
   ↓
Brown CAT6
   ↓
Antenna box
   ↓
K7 pin 8
```

The antenna box gets a separate +12 V feed:

```text
Remote +12 V BUS
      ↓
1.5 mm² wire
      ↓
Antenna box +12 V BUS
      ↓
K1–K8 pin 7
```

Relay coil current returns through the CAT6 wires to the ULN outputs in the remote box.

---

# 22. Important CAT6 Troubleshooting Lesson

If the ULN output is correct but the relay does not operate, do not immediately suspect the ESP32.

For example, for K1:

```text
ESP32 GPIO13
     ↓
ULN input 1
     ↓
ULN output 18
     ↓
Orange CAT6
     ↓
K1 pin 8
     ↓
K1 coil
     ↓
K1 pin 7
     ↓
+12 V
```

A break anywhere between ULN pin 18 and K1 pin 8 prevents the relay from operating.

During testing:

```text
ULN input 1 ON ≈ 3.3 V
ULN output 18 ON ≈ 0–1 V
```

At the relay:

```text
K1 pins 7–8 ≈ 12 V when ON
```

A measured ~11.6 V across the relay coil is perfectly consistent with a working 12 V supply and wiring.

---

# 23. LED Indicators

Use eight red 5 mm LEDs.

Each LED gets its own 2.2 kΩ resistor.

The wiring is:

```text
+12 V
  ↓
2.2 kΩ
  ↓
LED anode
LED cathode
  ↓
ULN output
```

LED assignments:

| LED | Relay | Function |
|---|---|---|
| LED1 | K1 | R1 → A1 |
| LED2 | K2 | R2 → A1 |
| LED3 | K3 | R1 → A2 |
| LED4 | K4 | R2 → A2 |
| LED5 | K5 | R1 → A3 |
| LED6 | K6 | R2 → A3 |
| LED7 | K7 | Radio 1 master |
| LED8 | K8 | Radio 2 master |

The LED cathodes share the corresponding ULN outputs with the relay coils.

---

# 24. 4×3 Keypad

The seven individual pushbuttons can be replaced with a 4×3 membrane matrix keypad.

A 4×3 keypad has:

```text
4 rows
3 columns
```

Therefore it requires seven ESP32 GPIOs.

Use:

| Keypad connection | ESP32 GPIO |
|---|---:|
| Row 1 | GPIO26 |
| Row 2 | GPIO27 |
| Row 3 | GPIO32 |
| Row 4 | GPIO33 |
| Column 1 | GPIO16 |
| Column 2 | GPIO17 |
| Column 3 | GPIO4 |

This replaces the old individual-button arrangement.

GPIO5 is no longer needed for the physical SAFE button.

The ESP32-WROOM board exposes GPIO4, GPIO16 and GPIO17 as usable I/O; GPIO5 is a strapping pin, so GPIO4 is preferable for the seventh keypad line. 

---

# 25. Keypad Functions

Use the keypad as follows:

```text
1 = Radio 1 → Antenna 1
2 = Radio 1 → Antenna 2
3 = Radio 1 → Antenna 3

4 = Radio 2 → Antenna 1
5 = Radio 2 → Antenna 2
6 = Radio 2 → Antenna 3

7 = SAFE / ALL OFF

8 = unused
9 = unused
* = unused
0 = unused
# = unused
```

The unused keys can be reserved for future functions.

---

# 26. Front Panel Layout

A useful layout is:

```text
        ANTENNA SWITCH

        RADIO 1
     ┌────┬────┬────┐
     │  1 │  2 │  3 │
     │ A1 │ A2 │ A3 │
     └────┴────┴────┘

        RADIO 2
     ┌────┬────┬────┐
     │  4 │  5 │  6 │
     │ A1 │ A2 │ A3 │
     └────┴────┴────┘

     ┌────┬────┬────┐
     │  7 │  8 │  9 │
     │SAFE│    │    │
     └────┴────┴────┘

     ┌────┬────┬────┐
     │  * │  0 │  # │
     └────┴────┴────┘

       ● ● ● ● ● ● ● ●
       K1 K2 K3 K4 K5 K6 K7 K8
```

The LEDs show the actual relay state.

---

# 27. Power Filtering

At minimum:

```text
100 nF capacitor
between +12 V and GND
near the ULN2803
```

An additional large electrolytic capacitor such as 470–1000 µF near the incoming 12 V supply is useful if available.

If only 10 nF and 100 nF capacitors are available, use the 100 nF capacitor. The 10 nF capacitor is not required for the basic relay system.

---

# 28. Firmware Logic

The firmware maintains two independent antenna states:

```text
Radio 1:
OFF / A1 / A2 / A3

Radio 2:
OFF / A1 / A2 / A3
```

The same antenna cannot be selected by both radios simultaneously.

For example:

```text
R1 → A1
R2 → A2
```

is allowed.

But:

```text
R1 → A1
R2 → A1
```

is rejected.

---

# 29. Break-Before-Make

The switching sequence is deliberately:

```text
1. Master relay OFF
2. Wait 100 ms
3. Old antenna relay OFF
4. Wait 100 ms
5. New antenna relay ON
6. Wait 100 ms
7. Master relay ON
```

This prevents the system from momentarily connecting two antennas or making an instantaneous hot switch between paths.

The software should therefore never simply turn the new relay on before releasing the old relay.

---

# 30. SAFE / ALL OFF

SAFE performs:

```text
K7 OFF
K8 OFF

wait 100 ms

K1 OFF
K2 OFF
K3 OFF
K4 OFF
K5 OFF
K6 OFF

wait 100 ms

Radio 1 = OFF
Radio 2 = OFF
```

This leaves the antenna selector in the safe idle condition.

With the selector relays OFF, the antenna grounding network leaves unused antenna centers connected to RF ground.

---

# 31. Wi-Fi Control

The ESP32 creates its own Wi-Fi access point.

SSID:

```text
Antenna-Switch
```

Password:

```text
12345678
```

Web interface:

```text
192.168.4.1
```

Existing routes:

```text
/r1a1
/r1a2
/r1a3
/r1off

/r2a1
/r2a2
/r2a3
/r2off

/safe
```

The web interface and physical keypad operate the same relay-control logic.

---

# 32. Important Safety Interlock

The system intentionally does not include a PTT/TX interlock in this version.

This was a deliberate design decision because there were no spare CAT/PTT connections available on the radios.

Therefore:

**Do not change antenna selection while transmitting.**

The selector should be operated only when the radio is not transmitting.

The break-before-make delay is useful protection against accidental overlap, but it is **not a substitute for a TX interlock**.

---

# 33. Initial Power-Up

Do not connect the radios initially.

Power up only the control system.

Check:

```text
12 V supply
↓
ESP32 buck converter
↓
ESP32
```

Verify that the ESP32 starts normally.

Then test the web interface.

---

# 34. Test Each Relay Individually

Test K1 first.

Select:

```text
R1-A1
```

Expected:

```text
K7 clicks
K1 clicks
```

Then select:

```text
R1-A2
```

Expected:

```text
K1 releases
K3 clicks
K7 remains active
```

Then:

```text
R1-A3
```

Expected:

```text
K3 releases
K5 clicks
K7 remains active
```

Repeat for Radio 2:

```text
R2-A1 → K8 + K2
R2-A2 → K8 + K4
R2-A3 → K8 + K6
```

---

# 35. Electrical Relay Test

For a relay that does not click, troubleshoot in this order.

Example: K1.

### ESP32 side

R1-A1 selected:

```text
GPIO13 → GND ≈ 3.3 V
```

### ULN input

```text
ULN pin 1 → GND ≈ 3.3 V
```

### ULN output

```text
ULN pin 18 → GND ≈ 0–1 V
```

### Relay coil

Measure across:

```text
K1 pin 7 ↔ K1 pin 8
```

Expected:

```text
≈ 12 V
```

If the ULN output is correct but the relay coil reads 0 V, investigate the CAT6 wire and its termination.

---

# 36. RF Continuity Testing

Use a multimeter with the entire system powered OFF.

Do not perform continuity tests while powered.

## SAFE / all relays OFF

Expected:

```text
A1 center ↔ A1 shell = continuity
A2 center ↔ A2 shell = continuity
A3 center ↔ A3 shell = continuity
```

Radio inputs should be isolated:

```text
R1 center ↔ A1 center = open
R1 center ↔ A2 center = open
R1 center ↔ A3 center = open

R2 center ↔ A1 center = open
R2 center ↔ A2 center = open
R2 center ↔ A3 center = open
```

---

# 37. R1-A1 Selected

Select R1-A1.

Expected:

```text
R1 center ↔ A1 center = continuity
A1 center ↔ A1 shell = open
```

Meanwhile:

```text
A2 center ↔ A2 shell = continuity
A3 center ↔ A3 shell = continuity
```

The signal path should be:

```text
R1 center
 ↓
K7 pin 5
 ↓
K7 pin 3
 ↓
R1 BUS
 ↓
K1 pin 5
 ↓
K1 pin 3
 ↓
A1 center
```

---

# 38. R1-A2 Selected

Expected:

```text
R1 center ↔ A2 center = continuity
A2 center ↔ A2 shell = open

A1 center ↔ A1 shell = continuity
A3 center ↔ A3 shell = continuity
```

Signal path:

```text
R1 center
 ↓
K7
 ↓
R1 BUS
 ↓
K3
 ↓
A2 center
```

---

# 39. R1-A3 Selected

Expected:

```text
R1 center ↔ A3 center = continuity
A3 center ↔ A3 shell = open
```

A1 and A2 should remain grounded.

---

# 40. Radio 2 Testing

The same logic applies:

```text
R2-A1:
R2 center ↔ A1 center = continuity

R2-A2:
R2 center ↔ A2 center = continuity

R2-A3:
R2 center ↔ A3 center = continuity
```

The appropriate antenna center should be isolated from its shell while selected.

The other antenna centers should remain grounded.

---

# 41. Common Mistakes

## K7 pin 1 vs pin 3

Do not connect the R1 BUS to K7 pin 1.

Correct:

```text
Radio 1 → K7 pin 5
R1 BUS  → K7 pin 3
```

## K8

Correct:

```text
Radio 2 → K8 pin 5
R2 BUS  → K8 pin 3
```

## K1–K6 signal contacts

Correct:

```text
COM = pin 5
NO  = pin 3
```

The RF signal always uses:

```text
pin 5 → pin 3
```

when the relay is energized.

## Antenna grounding

Do not assume:

```text
antenna center → ground = fault
```

For an **unselected antenna**, that connection is intentional in this design.

---

# 42. Final Wiring Summary

## Remote box

```text
12 V +
  │
  ├── Main switch
  │
  └── +12 V BUS
       ├── ULN pin 10
       ├── LED resistors
       ├── Buck converter IN+
       └── +12 V wire → antenna box

12 V -
  │
  └── GND BUS
       ├── ULN pin 9
       ├── ESP32 GND
       └── Buck IN-
```

ESP32:

```text
GPIO13 → ULN1 → K1
GPIO14 → ULN2 → K2
GPIO18 → ULN3 → K3
GPIO19 → ULN4 → K4
GPIO21 → ULN5 → K5
GPIO22 → ULN6 → K6
GPIO23 → ULN7 → K7
GPIO25 → ULN8 → K8
```

Keypad:

```text
R1 → GPIO26
R2 → GPIO27
R3 → GPIO32
R4 → GPIO33

C1 → GPIO16
C2 → GPIO17
C3 → GPIO4
```

CAT6:

```text
Orange      → K1 pin 8
Orange/White→ K2 pin 8
Green       → K3 pin 8
Green/White → K4 pin 8
Blue        → K5 pin 8
Blue/White  → K6 pin 8
Brown       → K7 pin 8
Brown/White → K8 pin 8
```

Antenna-box +12 V:

```text
+12 V → K1 pin 7
      → K2 pin 7
      → K3 pin 7
      → K4 pin 7
      → K5 pin 7
      → K6 pin 7
      → K7 pin 7
      → K8 pin 7
```

---

# 43. Complete Relay/RF Table

| Relay | Pin 5 COM | Pin 3 NO | Pin 6 | Pin 2 |
|---|---|---|---|---|
| K1 | R1 BUS | A1 center | A1 ground | K2 pin 6 |
| K2 | R2 BUS | A1 center | K1 pin 2 | RF ground |
| K3 | R1 BUS | A2 center | A2 ground | K4 pin 6 |
| K4 | R2 BUS | A2 center | K3 pin 2 | RF ground |
| K5 | R1 BUS | A3 center | A3 ground | K6 pin 6 |
| K6 | R2 BUS | A3 center | K5 pin 2 | RF ground |
| K7 | Radio 1 center | R1 BUS | unused | unused |
| K8 | Radio 2 center | R2 BUS | unused | unused |

For all eight relays:

```text
Pin 7 = +12 V
Pin 8 = ULN-controlled return
```

---

# 44. Final Conceptual Diagram

```text
                         REMOTE BOX
                  ┌─────────────────────┐
                  │                     │
                  │       ESP32         │
                  │                     │
                  │  GPIO13 ── K1       │
                  │  GPIO14 ── K2       │
                  │  GPIO18 ── K3       │
                  │  GPIO19 ── K4       │
                  │  GPIO21 ── K5       │
                  │  GPIO22 ── K6       │
                  │  GPIO23 ── K7       │
                  │  GPIO25 ── K8       │
                  │         │           │
                  │     ULN2803         │
                  │         │           │
                  │      CAT6           │
                  └─────────┼───────────┘
                            │
                     15 m maximum
                            │
                  ┌─────────┼───────────┐
                  │    ANTENNA BOX      │
                  │                     │
Radio 1 ─────────┤ K7 ── R1 BUS         │
                  │          ├── K1 ─ A1 │
                  │          ├── K3 ─ A2 │
                  │          └── K5 ─ A3 │
                  │                     │
Radio 2 ─────────┤ K8 ── R2 BUS         │
                  │          ├── K2 ─ A1 │
                  │          ├── K4 ─ A2 │
                  │          └── K6 ─ A3 │
                  │                     │
                  │       RF GROUND     │
                  └─────────────────────┘
```

---

# 45. Final Operating Rules

1. Never transmit while changing antennas.
2. Do not connect a radio until RF continuity tests pass.
3. Verify every relay's pin numbering before soldering.
4. Keep RF center conductors short.
5. Keep RF ground connections short and wide where possible.
6. Keep the RF signal wiring separated from digital/control wiring.
7. Make sure every unselected antenna is grounded.
8. Make sure a selected antenna center is NOT grounded.
9. Make sure the selected radio center reaches only the selected antenna.
10. Test every relay individually before connecting expensive radio equipment.

The final system therefore provides:

```text
        RADIO 1
          │
         K7
          │
       R1 BUS
       /  |  \
     K1  K3  K5
      │   │   │
     A1  A2  A3


        RADIO 2
          │
         K8
          │
       R2 BUS
       /  |  \
     K2  K4  K6
      │   │   │
     A1  A2  A3
```

with **unused antennas automatically grounded**, physical keypad control, Wi-Fi control, relay-state LEDs, and break-before-make switching.


# ESP32 Antenna Switch Firmware

This firmware controls:

- 8 × relay outputs through ULN2803APG
- 2 radios
- 3 antennas
- 4×3 physical keypad
- SAFE / ALL OFF
- Wi-Fi access point
- Browser control
- Break-before-make switching
- Same-antenna protection
- Browser indication of the currently selected antenna

---

## 1. Hardware

### Relay GPIOs

```text
K1 = GPIO13
K2 = GPIO14
K3 = GPIO18
K4 = GPIO19
K5 = GPIO21
K6 = GPIO22
K7 = GPIO23
K8 = GPIO25
```

### 4×3 Keypad

```text
Rows:

R1 = GPIO26
R2 = GPIO27
R3 = GPIO32
R4 = GPIO33

Columns:

C1 = GPIO16
C2 = GPIO17
C3 = GPIO4
```

The keypad functions are:

```text
1 = Radio 1 → Antenna 1
2 = Radio 1 → Antenna 2
3 = Radio 1 → Antenna 3

4 = Radio 2 → Antenna 1
5 = Radio 2 → Antenna 2
6 = Radio 2 → Antenna 3

7 = SAFE / ALL OFF
```

Keys 8, 9, *, 0 and # are currently unused.

---

## 2. Wi-Fi

The ESP32 creates its own access point:

```text
SSID:     Antenna-Switch
Password: 12345678 (please change)
IP:       192.168.4.1
```

Connect a phone, tablet or computer to the Wi-Fi network and open:

```text
http://192.168.4.1
```

---

## 3. Complete Code

```cpp
#include <Arduino.h>
#include <WiFi.h>
#include <WebServer.h>

// ============================================================
// ESP32 2-RADIO / 3-ANTENNA RELAY ANTENNA SWITCH
// ============================================================
//
// Radio 1:
//   K1 = Antenna 1
//   K3 = Antenna 2
//   K5 = Antenna 3
//   K7 = Radio 1 master
//
// Radio 2:
//   K2 = Antenna 1
//   K4 = Antenna 2
//   K6 = Antenna 3
//   K8 = Radio 2 master
//
// Relay control:
//   ESP32 HIGH -> ULN2803 input HIGH
//   ULN2803 output LOW -> relay coil energized
//
// IMPORTANT:
//   Do NOT change antennas while transmitting.
//   This controller does not provide PTT/CAT/TX interlock.
// ============================================================


// ============================================================
// WIFI ACCESS POINT
// ============================================================

const char* AP_SSID     = "Antenna-Switch";
const char* AP_PASSWORD = "12345678";

// Change the password above ^^ for actual deployment.

WebServer server(80);


// ============================================================
// RELAY GPIO ASSIGNMENTS
// ============================================================

const int K1 = 13;
const int K2 = 14;
const int K3 = 18;
const int K4 = 19;
const int K5 = 21;
const int K6 = 22;
const int K7 = 23;
const int K8 = 25;


// ============================================================
// 4x3 KEYPAD
// ============================================================

const int rowPins[4] = {
    26, 27, 32, 33
};

const int colPins[3] = {
    16, 17, 4
};

const char keyMap[4][3] = {
    { '1', '2', '3' },
    { '4', '5', '6' },
    { '7', '8', '9' },
    { '*', '0', '#' }
};


// ============================================================
// ANTENNA STATE
// ============================================================
//
// -1 = OFF
//  0 = Antenna 1
//  1 = Antenna 2
//  2 = Antenna 3
// ============================================================

int radio1Ant = -1;
int radio2Ant = -1;


// ============================================================
// RELAY CONTROL
// ============================================================
//
// ESP32 GPIO HIGH:
//     ULN2803 input HIGH
//     corresponding ULN output LOW
//     relay coil energized
//
// ESP32 GPIO LOW:
//     ULN2803 output OFF
//     relay coil de-energized
// ============================================================

void relayOn(int pin)
{
    digitalWrite(pin, HIGH);
}

void relayOff(int pin)
{
    digitalWrite(pin, LOW);
}


// ============================================================
// ALL RELAYS OFF
// ============================================================

void allRelaysOff()
{
    relayOff(K1);
    relayOff(K2);
    relayOff(K3);
    relayOff(K4);
    relayOff(K5);
    relayOff(K6);
    relayOff(K7);
    relayOff(K8);
}


// ============================================================
// MASTER RELAYS
// ============================================================

void radio1MasterOff()
{
    relayOff(K7);
}

void radio2MasterOff()
{
    relayOff(K8);
}


// ============================================================
// RADIO SELECTOR RELAYS OFF
// ============================================================

void radio1SelectorsOff()
{
    relayOff(K1);
    relayOff(K3);
    relayOff(K5);
}

void radio2SelectorsOff()
{
    relayOff(K2);
    relayOff(K4);
    relayOff(K6);
}


// ============================================================
// GET RADIO 1 SELECTOR RELAY
// ============================================================

int radio1SelectorRelay(int antenna)
{
    switch (antenna)
    {
        case 0:
            return K1;

        case 1:
            return K3;

        case 2:
            return K5;
    }

    return -1;
}


// ============================================================
// GET RADIO 2 SELECTOR RELAY
// ============================================================

int radio2SelectorRelay(int antenna)
{
    switch (antenna)
    {
        case 0:
            return K2;

        case 1:
            return K4;

        case 2:
            return K6;
    }

    return -1;
}


// ============================================================
// RADIO 1 ANTENNA SWITCHING
// ============================================================
//
// Break-before-make:
//
// 1. Master OFF
// 2. Wait 100 ms
// 3. Old selector OFF
// 4. Wait 100 ms
// 5. New selector ON
// 6. Wait 100 ms
// 7. Master ON
//
// antenna = -1 means OFF.
// ============================================================

bool setRadio1(int antenna)
{
    // Prevent both radios from using the same antenna.
    if (antenna >= 0 && antenna <= 2)
    {
        if (radio2Ant == antenna)
        {
            Serial.println("R1 request rejected: antenna already used by R2.");
            return false;
        }
    }

    // Turn master OFF first.
    radio1MasterOff();

    delay(100);

    // Release all Radio 1 selector relays.
    radio1SelectorsOff();

    delay(100);

    // OFF request.
    if (antenna == -1)
    {
        radio1Ant = -1;

        Serial.println("Radio 1 = OFF");

        return true;
    }

    // Turn the requested selector relay ON.
    int relay = radio1SelectorRelay(antenna);

    if (relay == -1)
    {
        return false;
    }

    relayOn(relay);

    delay(100);

    // Turn Radio 1 master ON.
    relayOn(K7);

    radio1Ant = antenna;

    Serial.print("Radio 1 = ");
    Serial.println(antennaName(antenna));

    return true;
}


// ============================================================
// RADIO 2 ANTENNA SWITCHING
// ============================================================
//
// Same break-before-make sequence as Radio 1.
// ============================================================

bool setRadio2(int antenna)
{
    // Prevent both radios from using the same antenna.
    if (antenna >= 0 && antenna <= 2)
    {
        if (radio1Ant == antenna)
        {
            Serial.println("R2 request rejected: antenna already used by R1.");
            return false;
        }
    }

    // Turn master OFF first.
    radio2MasterOff();

    delay(100);

    // Release all Radio 2 selector relays.
    radio2SelectorsOff();

    delay(100);

    // OFF request.
    if (antenna == -1)
    {
        radio2Ant = -1;

        Serial.println("Radio 2 = OFF");

        return true;
    }

    // Turn the requested selector relay ON.
    int relay = radio2SelectorRelay(antenna);

    if (relay == -1)
    {
        return false;
    }

    relayOn(relay);

    delay(100);

    // Turn Radio 2 master ON.
    relayOn(K8);

    radio2Ant = antenna;

    Serial.print("Radio 2 = ");
    Serial.println(antennaName(antenna));

    return true;
}


// ============================================================
// SAFE / ALL OFF
// ============================================================
//
// 1. K7 and K8 OFF
// 2. Wait 100 ms
// 3. K1-K6 OFF
// 4. Wait 100 ms
// 5. Software states OFF
//
// SAFE intentionally disconnects both radios from the antenna
// network and leaves the antenna grounding network in its
// normal unselected condition.
// ============================================================

void safeMode()
{
    Serial.println("SAFE / ALL OFF");

    // Master relays OFF first.
    radio1MasterOff();
    radio2MasterOff();

    delay(100);

    // All selector relays OFF.
    radio1SelectorsOff();
    radio2SelectorsOff();

    delay(100);

    radio1Ant = -1;
    radio2Ant = -1;
}


// ============================================================
// ANTENNA NAME
// ============================================================

String antennaName(int antenna)
{
    if (antenna == 0)
        return "Antenna 1";

    if (antenna == 1)
        return "Antenna 2";

    if (antenna == 2)
        return "Antenna 3";

    return "OFF";
}


// ============================================================
// KEYPAD RAW SCANNER
// ============================================================
//
// Returns the currently detected key.
// Returns 0 when no key is pressed.
//
// This function does NOT perform debounce.
// ============================================================

char readRawKey()
{
    char detectedKey = 0;

    // Scan each row.
    for (int r = 0; r < 4; r++)
    {
        // Set all rows HIGH.
        for (int i = 0; i < 4; i++)
        {
            digitalWrite(rowPins[i], HIGH);
        }

        // Drive current row LOW.
        digitalWrite(rowPins[r], LOW);

        delayMicroseconds(50);

        // Read columns.
        for (int c = 0; c < 3; c++)
        {
            if (digitalRead(colPins[c]) == LOW)
            {
                detectedKey = keyMap[r][c];
            }
        }
    }

    // Restore all rows HIGH.
    for (int i = 0; i < 4; i++)
    {
        digitalWrite(rowPins[i], HIGH);
    }

    return detectedKey;
}


// ============================================================
// KEYPAD DEBOUNCE
// ============================================================
//
// A key action occurs ONLY once when a key becomes stably
// pressed.
//
// Holding a key does NOT repeatedly trigger the action.
//
// The next action is possible only after the key is released.
// ============================================================

const unsigned long KEY_DEBOUNCE_MS = 50;

char lastRawKey = 0;
char stableKey = 0;
unsigned long lastRawChangeTime = 0;

void processKeypad()
{
    char rawKey = readRawKey();

    // Raw key changed.
    if (rawKey != lastRawKey)
    {
        lastRawKey = rawKey;
        lastRawChangeTime = millis();

        return;
    }

    // Raw key has not been stable long enough.
    if (millis() - lastRawChangeTime < KEY_DEBOUNCE_MS)
    {
        return;
    }

    // No key pressed.
    if (rawKey == 0)
    {
        // This is the important part:
        // release resets stableKey, allowing the next
        // physical press to generate exactly one action.
        stableKey = 0;

        return;
    }

    // Key is stable and pressed.
    // Only act if it was previously released.
    if (stableKey != rawKey)
    {
        stableKey = rawKey;

        handleKeyPress(rawKey);
    }
}


// ============================================================
// KEYPAD ACTION
// ============================================================

void handleKeyPress(char key)
{
    Serial.print("Keypad: ");
    Serial.println(key);

    switch (key)
    {
        // ----------------------------------------------------
        // RADIO 1
        // ----------------------------------------------------

        case '1':
            setRadio1(0);
            break;

        case '2':
            setRadio1(1);
            break;

        case '3':
            setRadio1(2);
            break;


        // ----------------------------------------------------
        // RADIO 2
        // ----------------------------------------------------

        case '4':
            setRadio2(0);
            break;

        case '5':
            setRadio2(1);
            break;

        case '6':
            setRadio2(2);
            break;


        // ----------------------------------------------------
        // SAFE
        // ----------------------------------------------------

        case '7':
            safeMode();
            break;


        // ----------------------------------------------------
        // UNUSED
        // ----------------------------------------------------

        case '8':
        case '9':
        case '0':
        case '*':
        case '#':
            break;
    }
}


// ============================================================
// HTML BUTTON GENERATOR
// ============================================================

String antennaButton(
    const String& label,
    const String& url,
    bool selected)
{
    String html;

    html += "<a href=\"" + url + "\">";

    if (selected)
    {
        html += "<button class=\"ant selected\">";
    }
    else
    {
        html += "<button class=\"ant\">";
    }

    html += label;
    html += "</button></a>";

    return html;
}


// ============================================================
// WEB PAGE
// ============================================================

void handleRoot()
{
    String html;

    html += "<!DOCTYPE html>";
    html += "<html>";
    html += "<head>";

    html += "<meta name=\"viewport\" "
            "content=\"width=device-width,initial-scale=1\">";

    html += "<title>Antenna Switch</title>";

    html += "<style>";

    html += "body{";
    html += "font-family:Arial,sans-serif;";
    html += "background:#111;";
    html += "color:white;";
    html += "text-align:center;";
    html += "margin:0;";
    html += "padding:20px;";
    html += "}";

    html += "h1{";
    html += "margin-bottom:10px;";
    html += "}";

    html += ".status{";
    html += "font-size:20px;";
    html += "margin:10px auto 25px;";
    html += "padding:15px;";
    html += "background:#222;";
    html += "border-radius:10px;";
    html += "max-width:500px;";
    html += "}";

    html += ".radio{";
    html += "margin:20px auto;";
    html += "max-width:500px;";
    html += "}";

    html += ".radio h2{";
    html += "margin-bottom:10px;";
    html += "}";

    html += ".ant{";
    html += "font-size:20px;";
    html += "font-weight:bold;";
    html += "padding:20px 25px;";
    html += "margin:5px;";
    html += "border:2px solid #555;";
    html += "border-radius:10px;";
    html += "background:#333;";
    html += "color:white;";
    html += "cursor:pointer;";
    html += "min-width:120px;";
    html += "}";

    html += ".ant.selected{";
    html += "background:#00b050;";
    html += "border-color:#00ff66;";
    html += "color:white;";
    html += "box-shadow:0 0 15px #00ff66;";
    html += "}";

    html += ".safe{";
    html += "font-size:20px;";
    html += "font-weight:bold;";
    html += "padding:18px 45px;";
    html += "margin-top:25px;";
    html += "border:none;";
    html += "border-radius:10px;";
    html += "background:#b00000;";
    html += "color:white;";
    html += "cursor:pointer;";
    html += "}";

    html += ".note{";
    html += "font-size:14px;";
    html += "color:#aaa;";
    html += "margin-top:25px;";
    html += "}";

    html += "</style>";

    html += "</head>";
    html += "<body>";

    html += "<h1>ESP32 Antenna Switch</h1>";


    // --------------------------------------------------------
    // CURRENT STATUS
    // --------------------------------------------------------

    html += "<div class=\"status\">";

    html += "<div><b>Radio 1:</b> ";
    html += antennaName(radio1Ant);
    html += "</div>";

    html += "<div><b>Radio 2:</b> ";
    html += antennaName(radio2Ant);
    html += "</div>";

    html += "</div>";


    // --------------------------------------------------------
    // RADIO 1
    // --------------------------------------------------------

    html += "<div class=\"radio\">";

    html += "<h2>Radio 1</h2>";

    html += antennaButton(
        "Antenna 1",
        "/r1a1",
        radio1Ant == 0
    );

    html += antennaButton(
        "Antenna 2",
        "/r1a2",
        radio1Ant == 1
    );

    html += antennaButton(
        "Antenna 3",
        "/r1a3",
        radio1Ant == 2
    );

    html += "<br>";

    html += "<a href=\"/r1off\">";
    html += "<button class=\"ant\">OFF</button>";
    html += "</a>";

    html += "</div>";


    // --------------------------------------------------------
    // RADIO 2
    // --------------------------------------------------------

    html += "<div class=\"radio\">";

    html += "<h2>Radio 2</h2>";

    html += antennaButton(
        "Antenna 1",
        "/r2a1",
        radio2Ant == 0
    );

    html += antennaButton(
        "Antenna 2",
        "/r2a2",
        radio2Ant == 1
    );

    html += antennaButton(
        "Antenna 3",
        "/r2a3",
        radio2Ant == 2
    );

    html += "<br>";

    html += "<a href=\"/r2off\">";
    html += "<button class=\"ant\">OFF</button>";
    html += "</a>";

    html += "</div>";


    // --------------------------------------------------------
    // SAFE
    // --------------------------------------------------------

    html += "<a href=\"/safe\">";
    html += "<button class=\"safe\">SAFE / ALL OFF</button>";
    html += "</a>";

    html += "<div class=\"note\">";
    html += "Selected antennas are highlighted in green.";
    html += "<br>";
    html += "The displayed state is the commanded software state, "
            "not electrical verification of relay contacts.";
    html += "</div>";

    html += "</body>";
    html += "</html>";

    server.send(200, "text/html", html);
}


// ============================================================
// WEB ROUTES
// ============================================================

void handleR1A1()
{
    if (!setRadio1(0))
    {
        server.send(
            409,
            "text/html",
            "<h2>Radio 1 - Antenna 1 unavailable</h2>"
            "<p>That antenna is currently assigned to Radio 2.</p>"
            "<p><a href=\"/\">Back</a></p>"
        );

        return;
    }

    server.sendHeader("Location", "/");
    server.send(303);
}


void handleR1A2()
{
    if (!setRadio1(1))
    {
        server.send(
            409,
            "text/html",
            "<h2>Radio 1 - Antenna 2 unavailable</h2>"
            "<p>That antenna is currently assigned to Radio 2.</p>"
            "<p><a href=\"/\">Back</a></p>"
        );

        return;
    }

    server.sendHeader("Location", "/");
    server.send(303);
}


void handleR1A3()
{
    if (!setRadio1(2))
    {
        server.send(
            409,
            "text/html",
            "<h2>Radio 1 - Antenna 3 unavailable</h2>"
            "<p>That antenna is currently assigned to Radio 2.</p>"
            "<p><a href=\"/\">Back</a></p>"
        );

        return;
    }

    server.sendHeader("Location", "/");
    server.send(303);
}


void handleR1Off()
{
    setRadio1(-1);

    server.sendHeader("Location", "/");
    server.send(303);
}


void handleR2A1()
{
    if (!setRadio2(0))
    {
        server.send(
            409,
            "text/html",
            "<h2>Radio 2 - Antenna 1 unavailable</h2>"
            "<p>That antenna is currently assigned to Radio 1.</p>"
            "<p><a href=\"/\">Back</a></p>"
        );

        return;
    }

    server.sendHeader("Location", "/");
    server.send(303);
}


void handleR2A2()
{
    if (!setRadio2(1))
    {
        server.send(
            409,
            "text/html",
            "<h2>Radio 2 - Antenna 2 unavailable</h2>"
            "<p>That antenna is currently assigned to Radio 1.</p>"
            "<p><a href=\"/\">Back</a></p>"
        );

        return;
    }

    server.sendHeader("Location", "/");
    server.send(303);
}


void handleR2A3()
{
    if (!setRadio2(2))
    {
        server.send(
            409,
            "text/html",
            "<h2>Radio 2 - Antenna 3 unavailable</h2>"
            "<p>That antenna is currently assigned to Radio 1.</p>"
            "<p><a href=\"/\">Back</a></p>"
        );

        return;
    }

    server.sendHeader("Location", "/");
    server.send(303);
}


void handleR2Off()
{
    setRadio2(-1);

    server.sendHeader("Location", "/");
    server.send(303);
}


void handleSafe()
{
    safeMode();

    server.sendHeader("Location", "/");
    server.send(303);
}


// ============================================================
// SETUP
// ============================================================

void setup()
{
    Serial.begin(115200);

    // --------------------------------------------------------
    // Relay GPIOs
    // --------------------------------------------------------

    pinMode(K1, OUTPUT);
    pinMode(K2, OUTPUT);
    pinMode(K3, OUTPUT);
    pinMode(K4, OUTPUT);
    pinMode(K5, OUTPUT);
    pinMode(K6, OUTPUT);
    pinMode(K7, OUTPUT);
    pinMode(K8, OUTPUT);

    // Make sure every relay is OFF.
    allRelaysOff();


    // --------------------------------------------------------
    // Keypad rows
    // --------------------------------------------------------

    for (int i = 0; i < 4; i++)
    {
        pinMode(rowPins[i], OUTPUT);
        digitalWrite(rowPins[i], HIGH);
    }


    // --------------------------------------------------------
    // Keypad columns
    // --------------------------------------------------------

    for (int i = 0; i < 3; i++)
    {
        pinMode(colPins[i], INPUT_PULLUP);
    }


    // --------------------------------------------------------
    // Wi-Fi Access Point
    // --------------------------------------------------------

    WiFi.mode(WIFI_AP);

    WiFi.softAP(
        AP_SSID,
        AP_PASSWORD
    );


    // --------------------------------------------------------
    // Web routes
    // --------------------------------------------------------

    server.on("/", handleRoot);

    server.on("/r1a1", handleR1A1);
    server.on("/r1a2", handleR1A2);
    server.on("/r1a3", handleR1A3);
    server.on("/r1off", handleR1Off);

    server.on("/r2a1", handleR2A1);
    server.on("/r2a2", handleR2A2);
    server.on("/r2a3", handleR2A3);
    server.on("/r2off", handleR2Off);

    server.on("/safe", handleSafe);

    server.begin();


    // --------------------------------------------------------
    // Startup information
    // --------------------------------------------------------

    Serial.println();
    Serial.println("================================");
    Serial.println("ESP32 Antenna Switch");
    Serial.println("================================");

    Serial.print("SSID: ");
    Serial.println(AP_SSID);

    Serial.print("IP: ");
    Serial.println(WiFi.softAPIP());

    Serial.println("System started.");
    Serial.println("All relays OFF.");
}


// ============================================================
// MAIN LOOP
// ============================================================

void loop()
{
    server.handleClient();

    processKeypad();
}
```

---

# 4. Browser Interface

The selected antenna now gets a **green highlighted button**.

For example, if Radio 1 is using Antenna 2:

```text
Radio 1: Antenna 2


[ Antenna 1 ]   [ ANTENNA 2 ]   [ Antenna 3 ]
                    ↑
                  GREEN
```

The status at the top also says:

```text
Radio 1: Antenna 2
Radio 2: OFF
```

If Radio 2 is subsequently switched to Antenna 3:

```text
Radio 1: Antenna 2
Radio 2: Antenna 3
```

and both selected buttons are highlighted.

---

# 5. Same-Antenna Protection

The firmware prevents this:

```text
Radio 1 → A1
Radio 2 → A1
```

If Radio 1 is already using A1 and you attempt to select A1 for Radio 2, the request is ignored.

The existing Radio 1 selection remains unchanged.

---

# 6. Physical Keypad

The physical keypad operates the same functions:

```text
1 → R1 A1
2 → R1 A2
3 → R1 A3

4 → R2 A1
5 → R2 A2
6 → R2 A3

7 → SAFE
```

Keys 8, 9, 0, * and # currently do nothing.

---

# 7. SAFE

The SAFE key:

```text
7
```

or the browser:

```text
SAFE / ALL OFF
```

turns off:

```text
K7
K8
K1
K2
K3
K4
K5
K6
```

and sets both software antenna states to OFF.

---

# 8. Important Note About the Browser Highlight

The green highlight represents the **ESP32's commanded state**.

It does not independently measure the physical relay contacts.

Therefore, during initial construction/testing, continue using your multimeter and the relay click as the final verification that the physical hardware agrees with the software state.

Once the hardware is confirmed, the browser display gives you a convenient operating-state indication.


## License

| Part | License |
|---|---|
| Firmware (`firmware/`) | [GPL-3.0-or-later](LICENSE) |
| This tutorial, wiring and design documentation | [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) |
| Any schematic / PCB / CAD files added later | [CERN-OHL-S-2.0](https://ohwr.org/cern_ohl_s_v2.txt) |

Copyright (c) 2026 Adam (KD3CSR).

You are free to build, use, modify and share this design. If you distribute a modified version, credit the original, share your changes under the same license, and keep them open for the ham community.

Improvements are welcome via pull request. Sign off your commits (`git commit -s`) to certify you wrote them and agree to the license of the part you changed.
