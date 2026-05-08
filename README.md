# DAQ — Industrial Data Acquisition Board

A custom-designed 16-bit 8-channel industrial Data Acquisition (DAQ) board for interfacing with 4–20mA industrial sensors. The system is designed with a strong focus on analog signal integrity, protection circuitry, low-noise power delivery, and industrial communication reliability.

## Key Features

* STM32-based embedded control system
* Dual ADS1115 16-bit ADCs for 8-channel acquisition
* 4–20mA sensor interface
* RS485 communication via RJ45 connector
* Analog front-end protection against:

  * ESD
  * negative transients
  * high-frequency noise
* RC anti-aliasing filters on all input channels
* Dedicated 24V, 5V, and 3.3V power rails
* Low-noise analog power architecture
* I2C-based multi-ADC addressing

---

# System Architecture

```text
4–20mA Sensors → Protection + RC Filtering → ADS1115 ADCs → STM32 Microcontroller → RS485 Transceiver → RJ45 Interface
```

---

# Analog Front-End Design

The analog input stage was designed specifically for industrial environments where noise, voltage spikes, and grounding issues are common.

Features include:

* SMAJ26A TVS diode for ESD and transient suppression
* Schottky clamp for negative spike protection
* RC low-pass filtering for:

  * anti-aliasing
  * 50/60Hz noise reduction
* Precision shunt resistor for converting 4–20mA current signals into measurable voltages

The RC filter cutoff frequency was chosen to suppress industrial noise while preserving slow-changing sensor signals such as temperature measurements.

---

# Power System

The board operates from a 24V input rail commonly available in industrial PLC systems.

Power architecture:

* 24V input rail for sensor excitation
* Switching converter for 5V generation
* LDO regulation for low-noise 3.3V analog/digital supply

The power system design includes:

* ripple analysis
* ESR calculations
* inductor sizing
* converter efficiency evaluation
* transient response considerations

Multiple converter topologies were evaluated to optimize efficiency at low load currents while maintaining low ADC noise.

---

# ADC System

The board uses two ADS1115 16-bit delta-sigma ADCs communicating over I2C.

Features utilized:

* programmable gain amplifier (PGA)
* configurable data rates
* single-shot conversion mode
* multi-device addressing using ADDR pins
* comparator/alert functionality

The ADC subsystem was designed for precise low-frequency industrial sensing applications.

---

# Engineering Focus Areas

This project emphasized:

* analog circuit design
* industrial protection design
* power integrity
* ADC interfacing
* signal conditioning
* embedded system integration
* low-noise PCB design considerations
* industrial communication systems

---

# Current Status

* Schematic design completed
* Analog front-end finalized
* Power system analysis completed
* ADC architecture implemented
* PCB development in progress

---

