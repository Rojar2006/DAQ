# DAQ
A 16-bit 8-channel industrial data acquisition board, with extensive analog front-end protection, RC filtering, two ADCs, STM32 microcontroller, RS485 transceiver for ethernet communication via RJ45 connector. 
**SCHEMATIC OVERVIEW**

- Used an STM32C031F4x as the microcontroller because it is cheap and processing power of the microcontroller is not a deciding factor. The user can program the microcontroller, to perform tasks based on the data input from the ADC. The microcontroller can be used to control other peripherals using the RS485 transceiver on board via ethernet.
- Used a relatively large 24V input power rail to drive the sensors so there would be enough headroom after the voltage drop associated with a series connection. Also because most Programmable Logic Controllers (PLC) with transistor output also has a 24V output pin, thus eliminating the need for a separate power supply.
- Used switching converters to step down from 24V to 5V and an LDO to step 5V down to 3.3V. Two switching converters are in consideration.
- Used the ADS1115 as the analog-to-digital converters.
- LTC2861 - The RS485 transciever
- RJ45 connector

**ANALOG FRONT END**
  Protective devices were placed to handle various faults.

- **Electrostatic Discharge(ESD)**
- **Negative Spike**
- **High frequency noise**

- For ESD protection an SMAJ26A diode was placed with clamps at 26V, it can handle upto hundreds of volts. A capacitor is also placed before the SMA diode to smooth small voltage spikes, however the capacitor cannot handle hundreds of volts.
- A shunt resistor is used as the sensors used are of the type 4-20mA, which gives current as output, ADC measure voltage so, in order to convert current to voltage, we use a simple resistor. The value chosen is 150 ohms. at 3mA it gives 0.6V at 4mA and 3V at 20mA.
- Another Schottky diode is placed on the rail to ground, in reverse bias condition. So that It turns ON when there is a negative voltage at the input channel. It clamps at -0.3V, acting as a guard rail. Cannot shunt it to Earth because it is directly connected to the ADC pins. If a negative spike hits and the diode clamps it to earth, and there is a potential mismatch between Earth and signal ground which is very common, the ADC still sees negative voltage. ADC only measures with reference to its GND pin. If you clamp to Earth and Earth is lower than GND, ADC sees negative voltage.
- There is also a low-pass RC filter at the end. The values are R = 1k ohms and C = 4.7uF. This gives a cutoff frequency of approximately 34Hz. This much smaller than the power line noise frequency(50-60Hz), and other noise which is typically seen in a factory. This filter also prevents aliasing. Aliasing is when the input signal has a frequency higher than half of the sampling frequency (Nyquist Frequency) and the ADC can’t track the signal properly.
    - If the input signal is exactly 64Hz and the sampling rate is 128 samples per second, and the sampling starts at t = 0, every the time the input gets sampled the ADC would record 0V, because since the frequencies are 64 and 128, it implies that in 1 input cycle, it gets sampled twice, with the same time interval. So if the sampling were to start at t = 0, the ADC would record 0 forever. This is called the Nyquist dead zone. Even though it depends on phase, which means if the sampling started later than the input signal, the ADC could record a value other than 0, but the values recorded would be repetitive in pairs of two.
    - Hence it is best to sample at a frequency much greater than twice the input signal frequency.

 **LM2596 Asynchronous Switching converter**
 **Power Rails**

- There are three power rails used in the schematic design, 5V, 3.3V and the input 24V line.
- The 24V is used to drive the 4-20mA sensors.
- The 5V is achieved through a asynchronous switching converter, the LM2596 - S. The fixed output voltage version.
- The LM2596 can drive upto 3A load.
- It is a non-synchronous buck converter, and it has a switching frequency of 150KHz.
- It is a 5-pin package.
    - Vin - Unregulated input voltage
    - Output - Regulated output voltage.
    - Ground - Reference voltage
    - Feedback
    - ~(ON)/OFF: Pin used to control when the regulator operates, if shutdown function is not needed, this pin can be connected to ground or left open. If pulled below 1.3V, it. is ON, if pulled above 1.3V upto 25V, it’s OFF.
- Absolute maximum supply voltage = 45V
- For a 24V input, with a load of 3A, the efficiency of the converter is approximately 82%, inferred from the data sheet.

**Freewheeling diode**

- LM2596 is a non-synchronous switching converter, which means an external Schottky diode carries current during the low phase of the switching cycle. When the internal MOSFET turns OFF, the inductor, which has stored energy, resists the change in current and generates a voltage across itself to keep current flowing. With no current source, the converter-side node can swing negative until it can draw current.
- Without the Schottky diode, this negative voltage can reach hundreds of volts and damage the internal MOSFET. When the node voltage reaches about -0.3V, the Schottky diode turns on and supplies current to the load. This is why it is also known as the freewheeling diode.
- Current path: Ground — Diode — Inductor — Load — Ground.
- When the internal MOSFET turns ON again, the process resumes, hence there is no break in power supply and voltage conversion is achieved.
- To provide a current path when the internal MOSFET is OFF.
- It must be fast with very low recovery time.
- Schottky diodes provide the best performance because of their low forward voltage drop and fast switching speed.

**Input Capacitor**

- The data sheet suggests a low-ESR aluminium or tantalum SMD capacitor between the input and ground pins, placed very close to the converter. This capacitor provides the instantaneous current when the internal switch turns ON.
- Important parameters while choosing an input capacitor are, Voltage rating and RMS current rating.
- A relatively large amount of current flows through the input capacitor of a buck converter, so the RMS current rating has highest priority.
- The RMS current when flowing through the internal ESR of the capacitor produces power as heat. The capacitors ability to dissipate this heat into the surroundings will determine the current it can safely handle.
- For a given capacitor value, a capacitor rated for a higher voltage will be larger and will be able to dissipate heat better, and therefore will have a higher RMS current rating.
- The ripple current which is supposed to flow through the capacitor during turn ON, must be lower the RMS current rating. Data sheet suggests a ripple current of 50% of DC load current for ambient temperatures upto 40 degrees.
- Estimated current draw = 133.651mA. 50% of estimated current draw =66.826mA. **The RMS current rating of the input capacitor must be larger than 66.826mA**. Data sheet also suggests that voltage rating must be at least 1.25 times the max input voltage. Often a much higher voltage rating is required to match the RMS current rating requirements.
- Standard electrolytic capacitors have high ESR values and therefore low RMS current ratings. They have a short operational lifetime.

**Output Capacitor**

- The output current is decided by the peripherals connected to the power rail. The duty cycle calculated by the switching converter decides that the average of the pulsating signal produced will be 5V.
- The output capacitor is part of the circuit that provides the constant DC voltage. The inductor connected to OUT pin of the switching converter doesn’t let the current be in phase with the voltage, instead it makes the current slowly ramp up while growing the magnetic field in the inductor. Then the internal MOSFET opens and the input voltage drops to 0, the magnetic field dissolves to keep the current flowing. Hence we get a triangular current output.
- The current required by the load(which depends upon the load), can be said to be the baseline on the triangular wave.
- The capacitor’s ESR converts the current to voltage. When the current is above the baseline(load current), the extra current charges the capacitor. when the current goes below the baseline, the capacitor discharges and provides current.
- Voltage noise limit for the ADC: 20mV
- Ripple current: ~40mA
- 20mV/40mA = 0.5 ohms, **ESR of the chosen capacitor must be less than 0.5 ohms.**
- Capacitance should be large enough so that during a spike in current draw, until the inductor can catch up the capacitor can supply current, **100uF** would prove sufficient.
- Ripple current rating is important because if a current greater than this value flows through the capacitor the electrolyte will evaporate and the capacitor will fail. Any capacitor above 100uF can handle current well above 40mA(ripple current, not RMS).
- **Cout = 100uF, ESR ≤ 0.5 ohms.**

**Inductor** 

Design of inductor value:

ADS1115: maximum current = 0.3mA

- The ADC is a device that senses voltage, and also outputs voltage levels.

STM32C031F4: Maximum current= 100mA

- The current through the VDD pin (max) = 100mA
- Current Sourced/Sinked by any GPIO pin(max) = 20mA
- The I2C pins are open-drain meaning they can’t source current, they have an internal MOSFET which connects them to ground when activated. The pull-up resistors are what provides the current. The TX pin doesn’t source much current because the LTC2861 has a very high input impedance.

LTC2861: Maximum current =~130mA 

- The maximum current of the LTC2861 is derived from obtaining the differential driver output voltage from the data sheet and using Ohm’s law with the given resistance with which it was tested.
- For RS485 R = 27ohms, and using a realistic value of 3.5V differential voltage between he output lines, external current = 130mA.
- We use 27 ohms because in a bus there would two termination resistors at the end and RS485 can support upto 32 devices. Each ‘unit’ corresponds to 12k ohms. 32 of them in parallel connection results in an equivalent resistance of 375 ohms. This 375 ohms is in parallel with the two 120 ohm termination resistors at the ends of the bus. This results in a total resistance of approximately 54ohms. Half of that is seen in one output line so R = 27 ohms.

Total current drawn from 5V line: 2*(ADS1115) + STM32C031F4 + LTC2861 = 2*(0.3) + 3.05 + 130 ‎ = 133.651mA
 Load current estimate = 133.651mA
30% load current (di) =40.0953mA = ripple current
V = L*di/dt
L = V*dt/di	dt = (1/frequency)*duty cycle; switching frequency = 150kHz
L = **658.1uH** 

After reviewing the current load of the 5V power rail, I’ve arrived at a revised value of **inductance L = 658.1uH.** The nearest commercially available value is 680uH.

The absolute maximum current rating of the LM2596 is 3A. It is most efficient at load current in the range of 2A. With a load current of 133mA, this switching converter would be very inefficient. Also in the data sheet, in the table suggesting inductance values for corresponding load current and input voltage, the lowest load current given is 0.6A. Since the estimated load current is a lot lower than this, it is best to switch to a converter more suitable for lighter loads.

 An alternate component is the LM5165, which is a high frequency low-current converter. It uses pulse-frequency modulation (PFM) at light loads to improve efficiency. **But the load current estimate is too close to the absolute current rating of the LM5165.**

Another option is the **LM5166** from the same family as LM5165. It’s maximum load current is 500mA and also uses PFM at light loads to increase efficiency.

**LM5166: Synchronous switching converter**
- The LM5165 is a synchronous switching converter.
- It has a high-side and low-side MOSFET at the SW pins, thus eliminating the need for a freewheeling diode.
- It supports upto 500mA load current.
- It has two modes of operation;
    - Constant ON-time(COT)
    - Pulse Frequency Modulation (PFM)

### Constant ON-time

- This is also a form of frequency modulation.
- The ‘ON’ time or the width of the pulse remains constant.
- Depending on the output voltage, the converter varies the switching frequency.
- If the output voltage drops due to an increase in current draw, the converter increases frequency, to charge the output capacitor more frequently.
- When the output capacitor has drained to a threshold voltage, the converter fires a pulse with a constant width(ON-time), to refill the capacitor. At heavier loads, this needs to happen more frequently, so switching frequency increases. At lighter loads, the capacitor discharges slowly, so it requires lesser amount of refills. Hence the switching frequency decreases.

### Pulse Frequency Modulation

- In pulsed frequency modulation, the converter changes the switching frequency to match the load current requirements as earlier.
- When the output voltage drops low due to increased load current, the converter sends a fixed width pulse to recharge the capacitor.
- After this pulse the converter goes into ‘sleep’ mode, shutting down all the internal circuitry lowering the quiescent current to the uA range.
- This is assuming the fact that the output capacitor drains very slowly, meaning the load current is very small. Which is why PFM increases efficiency at lighter loads.
- Drawback of PFM is that since the converter goes into sleep after every pulse, it takes longer for the converter to turn ON again, in the event of a voltage drop. So if the LTC2861 suddenly transmits and load current increases, the converter might react slowly and cause a significant voltage drop.
- Another drawback is that PFM produces high voltage ripple. The objective of PFM is to stay in ‘sleep’ mode as long as possible, so it lets the output capacitor discharge to the absolute minimum voltage and suddenly recharges it to maximum voltage. Causes more noise in the power rail than the COT.
- Also since it lets the capacitor discharge to absolute minimum, the brown out resistor of the microcontroller might trip, causing resets.

Considering the pros and cons, COT is what reduces the output voltage ripple, which is crucial for the ADCs, and reduces the chances of voltage drop in case of sudden spikes in load current.

**LM5166 will operate in Constant ON-time mode.**

### Pin configuration and pin functions

1. SW: The node at high side switching PFET and low side switching NFET, connected to inductor.
2. Vin: Regulator supply input to high-side power MOSFET and internal bias LDO. Connect to input capacitor to ground.
3. ILIM: Load current limit programming pin. Left open for maximum current of 300mA.
4. SS: Soft-start programming pin, used to program the time required to start up. Left open for a start-up time of 900us.
5. RT: Mode selection pin for PFM or COT, also a programming pin for the COT, hence determining switching frequency.
    
    $$
    R = \frac{V_{out}}{F_{SW}}*\frac{10^4}{1.75}
    $$
    
6. PGOOD: Power output flag pin, it is an open-drain pin, pulled low if the output voltage is outside of regulation limits. Used as a flag for downstream electronics(Microcontrollers, other converters etc.).
7. EN: Enable pin, the regulator is enabled if the EN is above 1.22V
8. Vout: Output voltage feedback for the regulator, to be connected to the inductor-capacitor node from the SW pin.
9. HYS: Drain of the NFET that is turned off when EN is active. Resistor from the EN to HYS to ground programs the Under Voltage Lockout (UVLO) threshold.
10. GND: Regulator ground return

**Power supply protection and noise handling**
## 24V POWER RAIL

- For the input power supply line(24V), a fuse was used for overcurrent protection,
- An SMA diode was used for Transient Voltage Suppression(TVS).
- Two capacitors are placed, before and after the fuse. Both the capacitors are tied to signal ground. The first capacitor dumps high frequency incoming from the power line, the fuse and parasitic inductance from it’s traces chokes the noise further, as inductance resists sudden changes in voltage. The second capacitor dumps whatever noise is left to ground.

## 5V POWER RAIL

- 5V is achieved through an asynchronous switching conductor, stepping drown from 24V to 5V with
- Most of the noise in the system will be due to the switching converter, the ripple current from the inductor changes to ripple voltage in the capacitor’s ESR.
    - The output capacitor is chosen so as to have a voltage ripple of 20mV on the 5V rail. The calculated max ESR value is 0.5 ohms.
    - Also the 100uF capacitor also acts as reservoir when the LTC2861 starts transmitting and draws lot of current.
    - The value is chosen such that the capacitor won’t run out before the inductor can catch up to the increased load current.

## 3.3V POWER RAIL

- Used an LDO to step down from 5V to 3.3V. It is more cost efficient than a switching converter. Since the voltage difference is low, heat produced is negligible.
- Also the LDO further helps to clear ripples from the switching converter.
- Placed a 0.1uF capacitor at the output pin of the LDO to dump high frequency left on the power rail to ground.

## Ripple voltage and ripple current calculations

Total current drawn from 5V line: 2*(ADS1115) + STM32C031F4 + LTC2861 = 2*(0.3) + 3.05 + 130 ‎ = 133.651mA
 Load current estimate = 133.651mA
30% load current (di) =40.0953mA = ripple current
V = L*di/dt
L = V*dt/di	dt = (1/frequency)*duty cycle; switching frequency = 150kHz
L = **658.1uH** 

After reviewing the current load of the 5V power rail, I’ve arrived at a value of **inductance L =** 658.1uH**.** The nearest commercially available value is **680uH**.

A ripple voltage of 20mV is acceptable on the ADC input lines. Since ripple current is approximately 40mA, and it is the ESR of the output capacitor which converts the current to voltage, 20mV/40mA = 0.5 ohms.

**ADS1115: Analog-to-Digital Converter**
The ADS1115 can perform conversions at data rates of upto 860 samples per second. It also has an input multiplexer for differential or single ended input. It features a Programmable Gain Amplifier (PGA), enabling accurate small and large signal measurements. It is a 16-bit delta-sigma converter with an I2C interface.

![***Fig 1***](attachment:d4835a5e-9214-432f-8d2a-bc29e4ba9f72:Screenshot_2026-05-07_at_8.57.38_PM.png)

***Fig 1***

The above is the block diagram of the ADS1115. 

## Overview of the ADS1115

The ADS1115 can perform conversions at data rates of upto 860 samples per second. It also has an input multiplexer for differential or single ended input. It features a Programmable Gain Amplifier (PGA), enabling accurate small and large signal measurements. It is a 16-bit delta-sigma converter with an I2C interface. It has 4 channels.

The ADC core inside the ADS1115 measures a differential signal between its two inputs. These two inputs don’t refer to the physical leads on the IC but rather that input of the ADC core inside the IC. Let these two inputs be A and B. 

### Single-ended and Differential mode input

In single-ended mode, the input multiplexer connects the input A to pin AIN0 and input B to ground. The multiplexer rotates through AIN0 to AIN3, measuring all 4 channels.

In differential mode, the multiplexer connects the core input A to AIN0 and input B to AIN1, thus making the input leads differential pairs. The multiplexer changes the connection to AIN2 and AIN3 to measure the next differential pair.

### Conversion modes

The ADS1115 has two available conversion modes;

- Single-shot conversion
- Continuous conversion

Single-Shot conversion: ADC performs a conversion upon request and stores it in an internal register and powers down to an idle state. Helps in saving power and in applications which require only periodic conversions.

Continuous conversion: The ADC automatically begins conversion after the previous conversion has finished. The conversion rate is equal to the programmed data rate. (upto 860 samples per second).

For this application the **single-shot conversion mode** will be used. As 4-20mA sensors are used, which primarily consists of temperature sensors. Temperature doesn’t take huge spikes suddenly. Periodic sampling is sufficient to monitor temperature.

### FSR ans LSB values

FSR stands for full-scale range, the range across which the ADC measures input voltage. By changing the voltage range across which the ADC measures its input(FSR), you can use the same ADC to measure very small signals in the mV range and large signals in the V range.

Since this is a 16-bit ADC the FSR is divided into 2^16 segments. The size of each segment is known as the resolution of the ADC, voltage required to make a change in LSB of output.

By changing the FSR, the resolution of the ADC is being changed, so smaller voltages will have more effect on the output.

The FSR is changed by the programmable gain amplifier. It can be programmed using the PGA[2:0] bits in the config register.

The FSRs and corresponding LSB of the ADS1115

| FSR | LSB |
| --- | --- |
|   1. ±6.144V | 187.5μV |
|  2. ±4.096V | 125μV |
|  3. ±2.048V | 62.5μV |
|  4. ±1.024V | 31.25μV |
|  5. ±0.512V | 15.625μV |
|  6. ±0.256V | 7.8125μV |

Only input signals within the VDD voltage can be measured. Even if the FSR gives a greater voltage range, if the VDD is smaller than that range, it doesn’t matter, any voltage above the VDD supply won’t be recorded. FSR is the mathematical limit, VDD is the physical limit.

### Output data rate

The output data rate is programmable using the DR[2:0] bits in the config register. This is to be programmed by sending a command over I2C bus from the MCU.

### Digital Comparator

The digital comparator inside the ADS1115 compares the input voltage value to user defined thresholds, Hi_thresh and Lo_thresh. It activates ALERT/READY pin based on the mode of operation. It is an active-low pin. It can be operated in 2 modes:

1. Traditional comparator
2. Window Comparator

When operated as a traditional comparator, if the input signal goes higher than the Hi_thresh the ALERT pin is pulled low. It goes high only when the input signal goes below the Lo_thresh.

When operated as a window comparator, the ALERT pin is pulled low if the signal goes below the Lo_thresh or goes higher than Hi_thresh.

**Latching Output**
Whether the digital comparator is operated in mode 1 or mode 2, it can be configured to be a latching or non-latching output. A latching output means, even if the input signal comes back down to the acceptable range depending upon the mode of operation, the ALERT pin doesn't change unless the converted data isn’t fetched from the conversion register or an SMBus alert response is sent to the ADC.

**Queue system**

The ALERT pin can also be configured to activate after a set number of successive readings exceed the Hi_thresh or Lo_thresh, to make sure that there are no false alarms due to noise. 

The ALERT/READY pin has another function as evident from the name. It can also be configured to be a **conversion ready pin**.
