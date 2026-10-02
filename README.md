# ESP32-Remote-Antenna-Switch-R2A3-HF
Remote antenna switch for 2 radios and 3 antennas

# ESP32 2-Radio / 3-Antenna Relay Antenna Switch

A DIY antenna selector for **two radios and three antennas**, controlled by an ESP32.

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

Contains:

- ESP32-WROOM development board
- ULN2803APG Darlington driver
- 4×3 membrane keypad
- 8 × red LEDs
- 8 × 2.2 kΩ LED resistors
- 12 V input
- Main power switch
- 12 V → 5 V buck converter
- 100 nF bypass capacitor
- CAT6 cable to the antenna box
- One separate +12 V wire to the antenna box

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
