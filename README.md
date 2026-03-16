# Faraday Porteur E-Bike: Torque Sensor Replacement Guide

Reverse engineering the Faraday Porteur's motor controller XML configs to determine if an aftermarket **Grin Technologies NCTE_120** torque sensor is a drop-in replacement.

**TL;DR: Yes, the stock `NCTE 43V 11 Rev3.xml` works as-is. No custom config needed.**

---

## Background

The [Faraday Porteur](https://transistor-man.com/faraday_bike_revival.html) is a defunct SF-designed e-bike with a 250W Bafang front hub motor and torque-sensing pedal assist. The original torque sensor (in the bottom bracket) can fail or wear out, and Faraday no longer exists to supply parts.

The motor controller (a Bafang-derived "BAC") is configured via XML files loaded through a Windows utility called BACDoor. Faraday shipped three different XML configs for three torque sensor variants used across production years:

| Sensor | Years | Type | PAS Poles |
|--------|-------|------|-----------|
| THUN | ≤2016 | Magnetostrictive, square taper | 8 |
| TDCM | 2017 (Porteur S) | Chain tension flex | 12 |
| NCTE | 2017+ | Magnetostrictive, square taper | 8 |

The **Grin Technologies NCTE_120** (from [ebikes.ca](https://ebikes.ca/ncte-120.html)) is a readily available aftermarket NCTE torque-sensing bottom bracket with 8 PAS poles, ~2.5V rest signal, and inverted torque output — electrically identical to the original.

## Key Findings

### 32 of 256 parameters differ across the three configs

The XML files are arrays of 256 unlabeled integer parameters (address 0–255). Of those, **224 are identical** across all three. The 32 that differ fall into clear categories:

| Category | What differs | Example addresses |
|----------|-------------|-------------------|
| **Sensor type** | PAS poles, sensor ID, signal polarity | 234, 235, 183 |
| **Torque calibration** | Zero-offset, full-scale, mapping curve | 180, 182, 168–171 |
| **Battery voltage** | ADC calibration for 43V vs 36V packs | 7, 8, 147, 149 |
| **Assist tuning** | Speed-dependent gain, startup ramp | 162–164, 200, 221, 223 |
| **Motor tuning** | Phase offset, current limits | 14, 16, 27, 29 |

### NCTE and THUN configs are nearly identical

They share the same sensor-critical parameters (8 PAS poles, inverted signal, same torque mapping). Differences are only battery voltage scaling (43V vs 36V) and minor ride-feel tuning. This confirms NCTE and THUN are electrically equivalent — as expected, since NCTE manufactured sensors for THUN.

### TDCM is fundamentally different

12 PAS poles, non-inverted torque signal, different mapping curves, no torque zero-offset. **Do not use the TDCM config with an NCTE sensor.**

### Critical parameter identifications

| Addr | Value (NCTE) | Meaning |
|------|-------------|---------|
| **234** | 8 | PAS pole count (must match sensor hardware) |
| **235** | 2 | Sensor type ID (2 = NCTE/THUN, 1 = TDCM) |
| **183** | -100 | Torque signal polarity (negative = inverted, as NCTE requires) |
| **180** | 4800 | Torque zero-offset (~2.5V rest voltage in ADC counts) |
| **182** | 9959 | Torque full-scale value |
| **67** | 5792 | Firmware version (matches `BAC Application 5792.ehx`) |

## Recommendation

Use **`NCTE 43V 11 Rev3.xml`** unchanged with the Grin NCTE_120.

The only post-install adjustment you *might* need: if assist engages without pedaling (or doesn't engage at all), tweak **Addr 180** (torque zero-offset) up or down slightly to match your specific sensor's rest voltage (Grin specs 2.45–2.55V).

### Wiring

The Grin NCTE_120 ships with a JST SM connector — re-terminate to the Faraday green Molex inside the seat tube:

| Wire | Function |
|------|----------|
| White | +10V supply |
| Black | GND |
| Blue | PAS Digital (cadence) |
| Brown | PAS Pulse |
| Grey | Torque analog signal |

## Project Structure

```
faraday-xml-project/
├── CLAUDE.md                    # Full project brief and task definitions
├── README.md                    # This file
├── xml/
│   ├── NCTE 43V 11 Rev3.xml    # NCTE sensor config (43V battery)
│   ├── TDCM 43V 11 Rev3.xml    # TDCM sensor config (43V battery)
│   └── THUN Rev4 36V.xml       # THUN sensor config (36V battery)
└── output/
    └── analysis-report.md       # Full parameter-by-parameter analysis
```

## References

- [Dane Kouttron's Faraday bike revival](https://transistor-man.com/faraday_bike_revival.html) — the definitive teardown and repair guide
- [Faraday software archive](https://transistor-man.com/files/faraday_repair/Faraday_Files_Q42018.zip) — firmware, BACDoor utility, and XML configs
- [Grin NCTE_120 product page](https://ebikes.ca/ncte-120.html)
- [Grin PAS options guide](https://ebikes.ca/getting-started/pas-options.html)
- [Endless Sphere: Restoring Faraday Porteur](https://endless-sphere.com/sphere/threads/restoring-faraday-porteur.127815/)
- [Endless Sphere: NCTE + CA thread](https://endless-sphere.com/sphere/threads/ncte-torque-sensor-with-ca3-1.114504/)
- [OpenBafangTool](https://github.com/andrey-pr/OpenBafangTool) — Bafang parameter reference
- [Faraday support (partially live)](https://support.faradaybikes.com)
