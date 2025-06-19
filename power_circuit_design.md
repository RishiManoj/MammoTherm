# Li-ion Battery Power Management Circuit Design

## Circuit Overview
This design provides a complete power management solution for a Raspberry Pi Zero 2W and thermal camera powered by a 3.7V Li-ion battery with USB-C charging capability.

## Main Components and Functions

### 1. Battery Charging Circuit
**IC: TP4056 or MCP73831**
- **TP4056**: Popular, inexpensive charging IC
  - Input: 5V via USB-C
  - Charging current: 1A (adjustable with resistor)
  - Built-in thermal regulation
  - Status LEDs (charging/standby)

**Alternative: MCP73831**
- More compact SMD package
- Programmable charge current (up to 500mA recommended for safety)

### 2. Battery Protection Circuit
**IC: DW01A + FS8205A (dual MOSFET)**
- Overvoltage protection (>4.3V)
- Undervoltage protection (<2.4V)
- Overcurrent protection (>3A)
- Short circuit protection
- Reverse polarity protection

**Alternative: Single IC solution - BQ29700**
- Integrated protection with MOSFETs
- More compact but slightly more expensive

### 3. Voltage Boost Circuit (3.7V to 5V)
**IC: MT3608 or TPS61023**
- **MT3608**: Adjustable boost converter
  - Input: 2V-24V (perfect for Li-ion 2.5V-4.2V range)
  - Output: Up to 28V (set to 5V with feedback resistors)
  - Efficiency: ~93%
  - Current capability: 2A+

**Feedback resistor calculation for 5V output:**
- R1 = 18kΩ (top resistor)
- R2 = 6.8kΩ (bottom resistor)
- Vout = 0.6V × (1 + R1/R2) = 5.0V

### 4. Battery Level Indicator (4 LEDs)
**IC: LM3914 or MAX17043**
- **LM3914**: Bar/dot display driver
  - Configure for 4 LEDs representing: 25%, 50%, 75%, 100%
  - Voltage thresholds: 3.2V, 3.5V, 3.8V, 4.1V

**Alternative: Microcontroller-based (ATtiny85)**
- More accurate readings
- Can implement low-power sleep modes
- Programmable thresholds

### 5. Power Control Circuit
**Power Button: Latching circuit with P-channel MOSFET**
- **MOSFET: IRF9540 or similar P-channel**
- **Control IC: TPS2113 (power multiplexer) or simple RC circuit**

### 6. Current and Voltage Protection
**Additional protection ICs:**
- **PTC Fuse**: 1.5A-2A rating for overcurrent
- **TVS Diode**: 5.1V for overvoltage protection on 5V rail
- **Schottky Diodes**: Reverse current protection

## Detailed Circuit Sections

### USB-C Charging Section
```
USB-C Connector → 5.1kΩ pulldown on CC pins → TP4056
- C1, C2: 5.1kΩ resistors on CC1, CC2 for 5V/3A capability
- Input capacitor: 47µF electrolytic
- PROG resistor: 2kΩ for 1A charging current
```

### Battery Protection Section
```
Li-ion Battery → DW01A → FS8205A MOSFETs → Protected Battery Output
- Bypass capacitors: 100nF ceramic + 47µF electrolytic
- Protection thresholds are fixed in DW01A
```

### Boost Converter Section
```
Protected Battery → MT3608 → 5V Output
- Input capacitor: 470µF electrolytic
- Output capacitor: 330µF electrolytic
- Inductor: 22µH (rated for 3A+)
- Schottky diode: Built into MT3608
```

### Battery Level Display
```
Battery Voltage → Voltage Divider → LM3914 → 4 LEDs
- Reference voltage: 2.5V (internal)
- LED current limiting: 10mA per LED
- Dot mode configuration
```

## Component List

### ICs
- TP4056: Battery charging controller
- DW01A: Battery protection IC
- FS8205A: Dual N-channel MOSFET (protection)
- MT3608: Boost converter IC
- LM3914: LED bar display driver
- IRF9540: P-channel MOSFET (power switch)

### Passive Components
- Resistors: 2kΩ, 5.1kΩ (×2), 6.8kΩ, 18kΩ, 1kΩ (×4)
- Capacitors: 47µF (×3), 330µF, 470µF, 100nF (×3)
- Inductor: 22µH (3A rated)
- PTC Fuse: 2A
- TVS Diode: 5.1V
- Schottky Diodes: 1N5819 (×2)

### Mechanical
- USB-C receptacle
- Tactile switch (power button)
- LEDs: 5× 3mm (4 for battery, 1 for power)
- JST connector for battery
- Headers for RPi and camera connections

## Power Calculations

### Current Requirements
- Raspberry Pi Zero 2W: ~400mA @ 5V (2W)
- Thermal Camera: 600mW @ 5V = 120mA
- **Total Load**: ~520mA @ 5V (2.6W)

### Battery Life Estimation
- Typical 18650 Li-ion: 2500mAh @ 3.7V (9.25Wh)
- With 90% boost efficiency: 8.3Wh usable
- **Runtime**: 8.3Wh ÷ 2.6W = ~3.2 hours

### Charging Time
- 2500mAh battery at 1A charging: ~3 hours (including taper phase)

## PCB Layout Considerations

1. **Thermal Management**: Place boost converter and charging IC away from each other
2. **Ground Planes**: Separate analog and digital grounds, connect at single point
3. **Power Traces**: Use wide traces (minimum 20 mil for 2A current)
4. **Component Placement**: Keep switching components away from sensitive analog circuits
5. **EMI Reduction**: Add ferrite beads on USB power input

## Safety Features Summary

- ✅ Overvoltage protection (battery and output)
- ✅ Undervoltage protection (prevents deep discharge)
- ✅ Overcurrent protection (PTC fuse + IC protection)
- ✅ Short circuit protection
- ✅ Thermal protection (in charging IC)
- ✅ Reverse polarity protection
- ✅ USB-C compliant charging (proper CC resistors)

## Alternative Design Options

### For Higher Integration:
- **BQ25895**: Advanced charging IC with I2C control
- **TPS63070**: Buck-boost converter (works from 1.8V-5.5V)
- **MAX17048**: Fuel gauge IC for accurate battery percentage

### For Lower Cost:
- Replace LM3914 with simple voltage comparators (LM339)
- Use fixed 5V boost modules instead of discrete design
- Simplify power switch to basic toggle switch

This design provides a robust, safe, and feature-complete power management solution for your project. The modular approach allows you to implement sections incrementally and test each part independently.