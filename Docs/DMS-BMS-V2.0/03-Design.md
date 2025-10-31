# 03-Design

## Preliminary Research

### Thunderstruck BMS

**Key Features**

- Cell Monitoring Modules (CMMs): Measures individual cell voltages and temperatures across multiple modules. Each CMM can monitor up to 12 cells, allowing scalability.

- Master Control Unit (MCU): Collects data from CMMs, manages balancing operations, and communicates with the vehicle’s control systems (CAN bus interface).

- Passive Balancing: Ensures all cells remain within safe voltage limits during charge and discharge cycles to prevent over-voltage, under-voltage, and thermal runaway conditions.

- Configurable Safety Thresholds: Supports user-defined voltage, temperature, and current limits. Exceeding these triggers fault states or pre-charge/interlock shutdowns.

- Data Logging & Communication: Integrates over CAN with the vehicle’s ECU, display, and data acquisition systems for telemetry and fault monitoring.


**FSAE Implementation**

- Rule Compliance: The Thunderstruck BMS meets FSAE EV rules requiring continuous monitoring of cell voltages and temperatures, fault detection, isolation, and communication to the shutdown circuit (EV 7.4.3–7.5.4).

- Time Efficiency: Saves extensive design, testing, and effort required for a custom BMS.

- CAN Integration: Compatible with existing vehicle control designs, simplifying communication between the accumulator, Motor Controller, and AMS light circuit.

**Implementation Requirements**

- High-Voltage Cell Interface Wiring from all cells/modules to the CMMs.

- Low-Voltage Communication Harness connecting the CMMs to the Master Controller via the provided daisy-chain harness (communication cable that links all the Cell Monitoring Modules together and then connects them to the Master Control Unit)

- CAN Bus Integration with the vehicle ECU and shutdown circuit.

- Power Supply (12 V or 24 V) for the MCU.

- Software Configuration via Thunderstruck’s BMS setup tool to calibrate thresholds and scaling factors.

- Physical Mounting & Shielding within the accumulator container per EV rules.

## Conceptual Design



## Final Design



## Validation 
