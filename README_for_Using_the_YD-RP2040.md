# YD-RP2040 Compatibility Fixes

This note records practical modifications to avoid pin conflicts between student modules and the YD-RP2040 breakout module. The goal is to make the YD-RP2040 safe and predictable for classroom module interchange.

## Summary of the issue

The YD-RP2040 uses pins on the “outer module header” (J4) that are also shared with internal RP2040 GPIO functions. Certain student modules may drive these lines with expectations that do not match the YD-RP2040 internal wiring, creating conflicts and unreliable operation.

Two critical conflicts are:

- Pin 37: 3V3_EN (module connector) is tied to GPIO23 (on the YD-RP2040).
- Pin 35: ADC_VREF (module connector analog reference) is tied to GPIO29.

---

## Fix 1: pin 37 conflict (3V3_EN vs GPIO23)

### Problem

- Some student modules attempt to drive 3V3_EN low to disable power, or treat it as a general I/O signal.
- On YD-RP2040 this line is also connected to GPIO23 and to RGB LED logic.
- Driving 3V3_EN can inadvertently pull GPIO23 low or short outputs, causing erratic RGB LED behavior or bus contention.

### Cause

- The YD-RP2040 regulator is hardwired always-on so external 3V3_EN control is not needed.
- The board design originally includes a solder jumper connecting the outer module 3V3_EN pin (J4-4) to the YD-RP2040 net (J5-4/GPIO23).

### Solution

1. Locate solder jumper between J5-4 and J4-4.
2. Cut the trace cleanly (open the solder jumper) to isolate the student module path.
3. Verify continuity: J4-4 should be disconnected from J5-4/GPIO23.

Result: The outer module pin is isolated from GPIO23 and cannot disturb the YD-RP2040 regulator/LED.

---

## Fix 2: pin 35 conflict (ADC_VREF vs GPIO29)

### Problem

- Student modules expecting an analog reference (ADC_VREF) may source/sink a voltage on pin 35.
- On YD-RP2040 this same pin is tied to digital GPIO29.
- The analog signal path can then drive GPIO29, causing invalid digitals states, or vice versa.

### Cause

- The board has solder jumper between J5-6 (ADC_VREF internal on YD) and J4-6 (outer module pin) tied by default.

### Solution

1. Locate and cut solder jumper between J5-6 and J4-6.
2. Confirm J4-6 is isolated from GPIO29.
3. Add a manual physical jumper from the isolated module pad (J4-6) to the P4 header testpoint (ADC_VREF/GND testpoints) on the YD-RP2040 board.

   - This preserves ADC_VREF connectivity for student modules without tying it through GPIO29.
   - The P4 header is positioned in the top-right region of the schematic (second sheet).

Result: ADC_VREF is safe and does not conflict with the RP2040 digital GPIO29.

---

## Why this is the right approach

- No firmware change required.
- Removes hardware contention at the connector boundary.
- Maintains compatibility with student modules that legitimately use 3V3_EN/ADC_VREF signals.

## Safety and verification checklist

- Power off board before soldering or cutting.
- Use a fine knife or hot-air gun to open the solder jumper.
- Check with a multimeter (continuity/diode mode):
  - J4-4 should not connect to J5-4/GPIO23.
  - J4-6 should not connect to J5-6/GPIO29.
- After wiring the jumper from J4-6 to P4 ADC_VREF, verify expected voltage with no load.

## Notes for educators

- Document the change on the board label or with a sticker to avoid accidental reconnection.
- For mass production, design revision should remove the conflicting internal footer link entirely and provide dedicated ADC_VREF / 3V3_EN headers.

