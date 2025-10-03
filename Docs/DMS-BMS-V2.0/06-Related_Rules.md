Battery Management System Related Rules

Provided Definition: [Battery management system - Wikipedia](https://en.wikipedia.org/wiki/Battery_management_system)

## **EV.7 SHUTDOWN SYSTEM**

**EV.7.1 Shutdown Circuit**

EV.7.1.1 The Shutdown Circuit consists of these components, connected in series:

1. Battery Management System (BMS) EV.7.3
2. Insulation Monitoring Device (IMD) EV.7.6
3. Brake System Plausibility Device (BSPD) EV.7.7
4. Interlocks (as required) EV.7.8
5. Master Switches (GLVMS, TSMS) EV.7.9
6. Shutdown Buttons EV.7.10
7. Brake Over Travel Switch (BOTS) T.3.3
8. Inertia Switch T.9.4

EV.7.1.3 The BMS, IMD, and BSPD parts of the Shutdown Circuit must be Normally Open

EV.7.1.4 The BMS, IMD, and BSPD must have completely independent circuits to Open the Shutdown Circuit. The design of the respective circuits must make sure that a failure cannot result in electrical power being fed back into the Shutdown Circuit.

EV.7.1.6 The team must be able to demonstrate all features and functions of the Shutdown Circuit and components at Electrical Technical Inspection.

**EV.7.2 Shutdown Circuit Operation**

EV.7.2.3 When the BMS, IMD or BSPD Open the Shutdown Circuit:

1. The Tractive System must stay disabled until manually reset
2. The Tractive System must be reset only by manual action of a person directly at the vehicle
3. The driver must not be able to reactivate the Tractive System from inside the vehicle
4. Operation of the Shutdown Buttons or TSMS must not let the Shutdown Circuit Close

**EV.7.3 Battery Management System – BMS**

EV.7.3.1 A Battery Management System must monitor the Tractive Battery Voltage EV.7.4 and Temperature EV.7.5 when the:

1. Tractive System is Active EV.11.5
2. Tractive Battery is connected to a Charger EV.8.3

EV.7.3.2 The BMS must have galvanic isolation at each Module to Module boundary, as approved in the Electrical Systems Form

EV.7.3.3 Cell balancing is not permitted when the Shutdown Circuit is Open (EV.7.2, EV.8.4)

EV.7.3.4 The BMS must monitor for:

1. Voltage values outside the permitted range EV.7.4.2
2. Voltage sense Overcurrent Protection device(s) blown or tripped
3. Temperature values outside the permitted range EV.7.5.2
4. Missing or interrupted voltage or temperature measurements
5. A fault in the BMS

EV.7.3.5 If the BMS detects one or more of the conditions of EV.7.3.4 above, the BMS must:

1. Open the Shutdown Circuit EV.7.2.2
2. Turn on the BMS Indicator Light and the Tractive System Status Indicator EV.5.11.5 The two lights must stay on until the BMS is manually reset EV.7.2.3

EV.7.3.6 The BMS Indicator Light must be:

1. Color: Red
2. Clearly visible to the seated driver in bright sunlight
3. Clearly marked with the lettering “BMS”

**EV.7.4 Tractive Battery Voltage**

EV.7.4.1 The BMS must measure the voltage of each Cell

When single Cells are directly connected in parallel, only one voltage measurement is needed

EV.7.4.2 Cell Voltage levels must stay inside the permitted minimum and maximum cell voltage levels stated in the cell data sheet. Measurement accuracy must be considered.

EV.7.4.3 All voltage sense wires to the BMS must meet one of:

1. Have Overcurrent Protection EV.7.4.4 below

EV.7.4.4 When used, Overcurrent Protection for the BMS voltage sense wires must meet the two:

1. The Overcurrent Protection must occur in the conductor, wire, or PCB trace which is directly connected to the cell tab.
2. The voltage rating of the Overcurrent Protection must be equal to or higher than the maximum Module voltage

**EV.7.5 Tractive Battery Temperature**

EV.7.5.1 The BMS must measure the temperatures of critical points of the Tractive Battery

EV.7.5.5 For lithium based cells,

1. The temperature of a minimum of 20% of the cells must be monitored by the BMS
2. The monitored cells must be equally distributed inside the Tractive Battery Container(s)

The temperature of each cell should be monitored

**EV.8.3 Charging Shutdown Circuit**

EV.8.3.1 The Charging Shutdown Circuit consists of:

1. Charger Shutdown Button EV.8.2.7
2. Battery Management System EV.7.3
3. Insulation Monitoring Device (IMD) EV.7.6

EV.8.3.2 The BMS and IMD parts of the Charging Shutdown Circuit must:

1. Be designed as Normally Open contacts
2. Have completely independent circuits to Open the Charging Shutdown Circuit. Design of the respective circuits must make sure that a failure cannot result in electrical power being fed back into the Charging Shutdown Circuit.

**EV.8.4 Charging Shutdown Circuit**

EV.8.4.1 When Charging, the BMS and IMD must:

1. Monitor the Tractive Battery
2. Open the Charging Shutdown Circuit if a fault is detected

**EV.5.2 Electrical Configuration**

EV.5.2.4 Soldering electrical connections in the high current path is prohibited

Soldering wires to the cells for the voltage monitoring input of the BMS is permitted, these wires are not part of the high current path

**EV.5.11 Tractive System Status Indicator**

EV.5.11.5 The Tractive System Status Indicator must show when the GLV System is energized:

| Condition | Green Light | Red Light |
| ---       | ---         | ---       |
| No Faults | Always ON   | OFF       |
| Fault in one of the two: BMS EV.7.3.5 or IMD EV.7.6.5 | OFF | Flash2Hz – 5Hz, 50% duty cycle |

**EV.6.6 Overcurrent Protection**

EV.6.6.5 Battery packs with Low Voltage or non voltage rated fusible links for cell connections may be used when all three conditions are met:

1. An Overcurrent Protection device rated at less than or equal to one third the sum of the parallel fusible links and complying with EV.6.6.2.b is connected in series.
2. The BMS can detect an open fusible link and will Open the Shutdown Circuit EV.7.2.2 if a fault is detected.
3. Fusible link current rating is specified in manufacturer’s data or suitable test data is provided.

**Critical Requirements**

1. Ensure cells stay between 3.0 and 4.2 Volts.
2. Ensure cells stay below 60 degrees Celsius.
3. Monitor for loss of communication between BMS slaves and host.
4. Monitor for open wire faults.
5. Include a watchdog timer: Will be reset periodically while the BMS system is monitoring reliably, else if it expires it initiates shut down protocol.

**Communication Protocol: CAN bus**

**Major Software Functions:**

1. The BMS should be receiving inputs from voltage and temp sensors and watchdog timer, and triggering the shutdown protocol if anything is off.
2. Periodically kick the watchdog
3. Communicate via CAN bus to the shutdown circuit and dashboard (red light)
4. Balancing the voltage of different cells in the pack so that it is as close to equal as possible.

Figure: 

![Image](\assets\BMS-ContextDiagram.png)
