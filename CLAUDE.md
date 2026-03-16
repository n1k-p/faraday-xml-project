# Faraday E-Bike Torque Sensor Configuration Project

## Project Goal
Reverse engineer and compare the Faraday Porteur e-bike motor controller XML configuration files for three torque sensor variants (NCTE, TDCM, THUN) to determine compatibility with an aftermarket Grin Technologies NCTE torque sensor (model NCTE_120 from ebikes.ca), and if needed, generate a custom XML configuration.

## Background

### The Bike
- Faraday Porteur e-bike (defunct SF-based company, originally $3,500)
- 250W Bafang (8Fun) sensorless geared front hub motor
- 12S2P 18650 battery (~44V nominal) stored in downtube
- Torque-sensing bottom bracket activates pedal assist
- Custom motor controller (repackaged Bafang-derived) with intermediate PCB
- Three-mode selector switch: OFF / Standard / Boost

### The Controller
- Referred to as "BAC" (likely Bafang Assist Controller)
- Communicates via serial (COM port) using proprietary protocol
- Configured via "BACDoor" Windows utility
- Motor parameters loaded from XML files specific to the torque sensor installed
- Firmware: `BAC Application 5792.ehx`
- Can be updated via Faraday Utility + BACDoor over USB diagnostic cable

### The Torque Sensors
Three sensor types were used across Faraday production years:

| Sensor | Years | Type | PAS Poles | Senses |
|--------|-------|------|-----------|--------|
| THUN   | ≤2016 | Magnetostrictive, square taper | 8 | Left crank only |
| TDCM   | 2017 Porteur S | Chain tension flex | 12 | Both cranks |
| NCTE   | 2017+ | Magnetostrictive, square taper | 8 | Left crank only |

- NCTE and THUN are electrically identical (NCTE manufactured for THUN)
- TDCM uses a fundamentally different sensing method and has 12 PAS poles vs 8
- All connect via green Molex connector inside seat tube

### The Replacement Sensor
- Grin Technologies NCTE_120 (ebikes.ca)
- NCTE square taper torque sensing bottom bracket
- 120mm spindle, 68mm BSA shell
- 8 PAS poles
- 5-wire: White (10V+), Black (GND), Blue (PAS D), Brown (PAS P), Grey (Torque)
- Torque signal: 2.45-2.55V at rest (no force on cranks)
- Supply voltage: 10-16V, current draw: 12mA
- Terminated with JST SM connector (needs re-termination to Faraday green Molex)

## Input Files

Place these files in the `xml/` directory:

```
xml/
├── NCTE 43V 11 Rev3.xml      # NCTE sensor config
├── TDCM 43V 11 Rev3.xml      # TDCM sensor config
└── THUN Rev4 36V.xml          # THUN sensor config
```

Source: https://transistor-man.com/files/faraday_repair/Faraday_Files_Q42018.zip
(Download, unzip, find XMLs in the Motor Parameters folder)

## Tasks

### Task 1: Parse and Document
- Parse all three XML files
- Create a unified table/report of ALL parameters across all three configs
- Identify and label each parameter with best-guess descriptions
- Flag parameters that differ between configs vs those that are identical

### Task 2: Diff Analysis
- Generate a focused diff showing ONLY the parameters that change between configs
- Group changes by category (sensor-related, voltage-related, motor-related, etc.)
- Highlight which differences are likely sensor-specific vs bike-variant-specific
- Note: filename hints — NCTE/TDCM are "43V 11" while THUN is "Rev4 36V", suggesting voltage/hardware revision differences beyond just sensor type

### Task 3: Cross-Reference with Known Specs
- Compare XML parameter values against known Grin NCTE specs:
  - 8 PAS poles
  - Torque signal ~2.5V at rest
  - 10-16V supply
  - Left crank only sensing
  - Magnetostrictive principle (torque signal falls with rising torque)
- Identify any parameters that might need adjustment for the Grin NCTE

### Task 4: Generate Custom XML (if needed)
- If the stock `NCTE 43V 11 Rev3.xml` should work as-is, document why
- If modifications are needed, generate a new XML with changes annotated
- Include safety notes about any parameters that should NOT be changed

### Task 5: Bafang Parameter Cross-Reference (stretch goal)
- Attempt to cross-reference unlabeled XML parameters against known Bafang controller configuration formats
- Reference: Luna Cycle BBSTool, OpenBafangTool (github.com/andrey-pr/OpenBafangTool)
- The Faraday controller is described as a "repackaged" Bafang with an intermediate middleman board

## Output Format
- Generate a markdown report with all findings
- Include the parameter comparison table
- Include the diff analysis
- Include a recommendation (use stock XML vs custom XML)
- If custom XML needed, output the file to `output/` directory

## Reference Links
- Dane Kouttron's Faraday revival: https://transistor-man.com/faraday_bike_revival.html
- Faraday support (partially live): https://support.faradaybikes.com
- Faraday software archive: https://transistor-man.com/files/faraday_repair/Faraday_Files_Q42018.zip
- Grin NCTE product page: https://ebikes.ca/ncte-120.html
- Grin NCTE install guide (PDF): https://www.ebikes.ca/documents/Pkg_NCTEBottomBracket.pdf
- Grin PAS options page: https://ebikes.ca/getting-started/pas-options.html
- Endless Sphere Faraday thread: https://endless-sphere.com/sphere/threads/restoring-faraday-porteur.127815/
- Endless Sphere NCTE+CA thread: https://endless-sphere.com/sphere/threads/ncte-torque-sensor-with-ca3-1.114504/
- Endless Sphere NCTE custom controller: https://endless-sphere.com/sphere/threads/custom-controller-settings-for-combined-torque-and-cadence-sensor-by-ncte.117978/
- OpenBafangTool: https://github.com/andrey-pr/OpenBafangTool
- Faraday BAC firmware update procedure: https://support.faradaybikes.com/hc/en-us/articles/360000114934

## Notes
- The XML parameter labels are reportedly unlabeled/opaque — expect key-value pairs without human-readable names
- The Faraday controller is NOT a standard off-the-shelf Bafang — it has custom firmware and an intermediate PCB
- The BACDoor utility format may or may not match standard Bafang config tools
- The NCTE torque signal is INVERTED (falls with rising torque) — this is a known quirk that controllers must account for
