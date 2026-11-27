# MiniKeys 1x4 — Design Review Notes

---

## REV A01 Schematic Review - before PCB layout

### Changes (agreed)

- [x] **LED_DIN → 5 V-tolerant pin.** Move LED_DIN off PA6 (not FT) to a 5 V-tolerant timer pin — PA8 (TIM1_CH1) is unused and FT. Keep open-drain and drop R21 from 10 kΩ to ~1 kΩ so the rising edge is fast enough (10 k ≈ 150 ns τ vs a 200–410 ns "0"-code high time). Fallback if still marginal: 5 V-powered TTL-input buffer (e.g. 74AHCT1G125) + 33–100 Ω series R at D5 DIN.
- [x] **D6 "disable" method.** Fitting D6 *and* R27 doesn't disable D6 (see A01 note 1). Choose: (a) fit D6, R27 DNP, firmware writes 0,0,0 to D6 — recommended, re-enable is firmware-only; or (b) D6 DNP, R27 fitted — re-enable = fit D6 + remove R27.
- [x] **Break out PA9 (USART1_TX) / PA10 (USART1_RX)** for ROM-bootloader (USART) flashing. SWD stays the primary programming method. Also usable as a debug UART.
- [x] **Add GND to J3** (optionally 3V3).
- [x] **R12–R14 → standard value.** 500 Ω isn't a standard value; use 510 Ω (E24) or 499 Ω (E96).
- [x] **Add a schematic comment at R1/R2:** "Never populate with R3/R4 fitted — Rp to VBUS on a sink breaks CC attach detection."
- [x] **Y2 footprint + BOM part.** 3-pin symbol (pin 2 → GND) on a 2-pad `CC7V-T1A` footprint → change symbol to 2 pin crystal. 
- [x] **D1 package/clamp.** ESD441DPLR is the 0201 (0.6 × 0.3 mm) version on a hand-solder footprint — hard to place/rework. 0402 version exists (ESD441DPY). 
    - 0402 version has good stock, use this one instead

### Verify before release

- [x] **J1 footprint vs MPN (designer to double-check).** Footprint is GCT USB4105; MPN is Amphenol 10155435-00011LF. Amphenol drawing could not be retrieved during review. 
    - verified footprint fits USB-C connector
- [x] **Y1 footprint.** `Oscillator_SMD_Fox_FT5H_5.0x3.2mm` is an oscillator land; with the 3-pin crystal symbol, pad 4 has no net. 
    - Pin 4 is used, it's tied to GND but hidden in the symbol.

### For consideration

- [x] **Key switch common to GND** instead of +3V3 (internal pull-ups). Avoids routing the MCU rail to four exposed hotswap sockets; lets R19/R22/R25/R29 become ~100 Ω–1 kΩ series protection instead of 0 Ω.
    - Keeping the switches on active low external pull up for now 
- [x] **Block back-powering from the debugger.** J2-8 (+3V3) driven by a probe can feed backwards through the LDO into VBUS, the LEDs and the USB connector (confirm MIC5504 reverse-current behaviour first). 
    - Likely not a concern since the EN is VBUS side

- **USB-C shield default (R10/C11).** DNP for prototype. Pick a default before boards in daily use — ESD issues don't show up in bench testing. 0 Ω is simplest for a bus-powered, plastic-cased board; or 1 MΩ ∥ 4.7–10 nF. Put intended values in the Value field instead of "DNP" (also C13/C14, R9, R26/R28).
- **Capacitor selection.** VBUS caps ≥ 10 V rating. For more *effective* capacitance under DC bias, go up a package size (0603 → 0805) rather than up in voltage rating — within one package/dielectric the rating often makes little difference. Check the DC-bias curve of the actual MPN (Murata SimSurfing / TDK / Samsung tools). C0G for C9/C10; 1 % resistors.
- **U4–U6 VP** — optional: tie to +3V3 or float instead of VBUS. Low priority.

### Future uprev

- **GPIO-switched D+ pull-up** (PNP or R7 to a spare GPIO) so firmware can force re-enumeration after reflashing.

