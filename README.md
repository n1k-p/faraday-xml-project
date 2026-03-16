# Faraday Torque Sensor XML Analysis

## Quick Start

1. Download the Faraday software archive:
   https://transistor-man.com/files/faraday_repair/Faraday_Files_Q42018.zip

2. Unzip and find the Motor Parameters folder. Copy the three XML files into `xml/`:
   - `NCTE 43V 11 Rev3.xml`
   - `TDCM 43V 11 Rev3.xml`
   - `THUN Rev4 36V.xml`

3. Run Claude Code from this directory:
   ```
   claude
   ```

4. Prompt:
   ```
   Read the CLAUDE.md, then parse and compare all three XML files in xml/. 
   Generate the full analysis report per the tasks defined.
   ```

## Project Structure

```
faraday-xml-project/
├── CLAUDE.md          # Project brief and context for Claude Code
├── README.md          # This file
├── xml/               # Place the three Faraday XML configs here
│   ├── NCTE 43V 11 Rev3.xml
│   ├── TDCM 43V 11 Rev3.xml
│   └── THUN Rev4 36V.xml
└── output/            # Generated reports and custom XMLs go here
```

## What This Project Does

Compares the three Faraday motor controller XML configuration files to determine
if an aftermarket Grin Technologies NCTE_120 torque sensor will work with the
stock Faraday controller, and if not, what XML parameters need to be modified.
