# BMS Functional Requirements

## 1. Overview

This document defines the detailed functional requirements for the FSAE Battery Management System (BMS) software. These requirements are from FSAE 2026 rules and best practices for lithium-ion battery management systems.

**System Components:**

- BMS IC: ADBMS6030b (Analog Devices)
- Microcontroller: STM32G0B1 (or compatible STM32 variant)
- Communication: isoSPI (BMS IC), CAN bus (vehicle network)

---

## 2. Monitoring Requirements

### 2.1 Cell Voltage Monitoring

**REQ-MON-001:** The BMS shall measure individual cell voltages for all cells in the battery pack.

- **Frequency:** Minimum 10 Hz during active operation
- **Accuracy:** ±5 mV or better
- **Range:** 2.5V to 4.5V per cell

**REQ-MON-002:** The BMS shall detect and flag any cell voltage measurement that is out of valid range.

- **Action:** Set fault flag if voltage < 2.0V or > 5.0V  which indicates a sensor fault.

**REQ-MON-003:** The BMS must calculate and store:

- Minimum cell voltage
- Maximum cell voltage
- Average cell voltage
- Cell voltage delta **Math** (max - min)

**REQ-MON-004:** The BMS shall validate voltage measurements for consistency.

- **Check:** Sum of cell voltages shall match pack voltage within ±2%
- **Action:** Set diagnostic warning if mismatch detected

### 2.2 Temperature Monitoring

**REQ-MON-005:** The BMS shall measure temperature at multiple locations in the battery pack.

- **Minimum sensors:** One per battery segment/module
- **Frequency:** Minimum 1 Hz
- **Accuracy:** ±2°C
- **Range:** -40°C to +85°C

**REQ-MON-006:** The BMS shall detect and flag any temperature sensor failure.

- **Detection:** Open circuit, short circuit, or out-of-range reading
- **Action:** Set sensor fault flag and use redundant sensors if available

**REQ-MON-007:** The BMS shall calculate and store:

- Minimum pack temperature
- Maximum pack temperature
- Average pack temperature

### 2.3 Current Monitoring

**REQ-MON-008:** The BMS shall measure pack current (charge and discharge).

- **Frequency:** Minimum 10 Hz
- **Accuracy:** ±1% of full scale
- **Range:** Bidirectional, sufficient for maximum pack current

**REQ-MON-009:** The BMS shall track cumulative charge/discharge.

- **Function:** Coulomb counting for SOC estimation
- **Precision:** Sufficient for ±5% SOC accuracy

**REQ-MON-010:** The BMS shall detect current sensor failures.

- **Detection:** Stuck reading, out-of-range, or inconsistent values
- **Action:** Set sensor fault and disable SOC estimation if needed

### 2.4 Pack Voltage Monitoring

**REQ-MON-011:** The BMS shall measure total pack voltage.

- **Frequency:** Minimum 10 Hz
- **Accuracy:** ±0.5% of reading
- **Method:** Direct measurement via ADC and/or calculation from cell voltages

### 2.5 State of Charge (SOC) Estimation

**REQ-MON-012:** The BMS shall estimate State of Charge (SOC) continuously.

- **Method:** Coulomb counting with voltage-based correction
- **Accuracy:** ±5% under normal operating conditions
- **Range:** 0% to 100%

**REQ-MON-013:** The BMS shall reset/calibrate SOC under known conditions.

- **Conditions:** Full charge (cell voltage reaches max), full discharge (reaches min)
- **Action:** Synchronize coulomb counter with voltage-based SOC

**REQ-MON-014:** The BMS shall persist SOC estimate to non-volatile memory.

- **Frequency:** Periodically (every 5% change or every 60 seconds)
- **Purpose:** Recover SOC estimate after power cycle

### 2.6 Data Validation and Filtering

**REQ-MON-015:** The BMS shall filter sensor measurements to reduce noise.

- **Method:** Moving average, median filter, or low-pass filter
- **Balance:** Response time vs. noise rejection

**REQ-MON-016:** The BMS shall validate all sensor data before use.

- **Checks:** Range check, rate-of-change check, consistency check
- **Action:** Reject invalid data and flag sensor fault

**REQ-MON-017:** The BMS shall detect stuck sensor readings.

- **Method:** Monitor for unchanging values over extended period
- **Action:** Set diagnostic warning

---

## 3. Fault Detection Requirements

### 3.1 Cell Voltage Faults

**REQ-FLT-001:** The BMS shall detect cell overvoltage conditions.

- **Warning Threshold:** Configurable (e.g., 4.15V)
- **Fault Threshold:** Configurable (e.g., 4.20V)
- **Critical Threshold:** Configurable (e.g., 4.25V)
- **Action:** Warning → log, Fault → reduce current, Critical → open contactors

**REQ-FLT-002:** The BMS shall detect cell undervoltage conditions.

- **Warning Threshold:** Configurable (e.g., 3.0V)
- **Fault Threshold:** Configurable (e.g., 2.8V)
- **Critical Threshold:** Configurable (e.g., 2.5V)
- **Action:** Warning → log, Fault → reduce current, Critical → open contactors

**REQ-FLT-003:** The BMS shall detect excessive cell voltage imbalance.

- **Threshold:** Cell voltage delta > configurable limit (e.g., 100mV)
- **Action:** Log diagnostic warning, trigger balancing if safe

### 3.2 Temperature Faults

**REQ-FLT-004:** The BMS shall detect over-temperature conditions.

- **Warning Threshold:** Configurable (e.g., 50°C)
- **Fault Threshold:** Configurable (e.g., 55°C)
- **Critical Threshold:** Configurable (e.g., 60°C)
- **Action:** Warning → log, Fault → reduce current, Critical → open contactors

**REQ-FLT-005:** The BMS shall detect under-temperature conditions.

- **Warning Threshold:** Configurable (e.g., 0°C)
- **Fault Threshold:** Configurable (e.g., -5°C)
- **Action:** Warning → log, Fault → limit charge current

**REQ-FLT-006:** The BMS shall detect rapid temperature rise.

- **Threshold:** Rate of change > configurable limit (e.g., 5°C/minute)
- **Action:** Set thermal runaway warning, increase monitoring frequency

### 3.3 Current Faults

**REQ-FLT-007:** The BMS shall detect discharge overcurrent conditions.

- **Warning Threshold:** Configurable based on pack specifications
- **Fault Threshold:** Configurable (e.g., 110% of continuous rating)
- **Action:** Warning → log, Fault → open contactors

**REQ-FLT-008:** The BMS shall detect charge overcurrent conditions.

- **Warning Threshold:** Configurable based on pack specifications
- **Fault Threshold:** Configurable (e.g., 110% of continuous rating)
- **Action:** Warning → log, Fault → open contactors

**REQ-FLT-009:** The BMS shall implement short circuit protection.

- **Detection:** Current exceeds emergency threshold (e.g., 200% rating)
- **Response Time:** < 100ms
- **Action:** Immediate contactor opening

### 3.4 System Faults

**REQ-FLT-010:** The BMS shall detect communication failures with ADBMS6030b.

- **Detection:** No response within timeout period (e.g., 100ms)
- **Action:** Set communication fault, attempt recovery, enter safe state if persistent

**REQ-FLT-011:** The BMS shall detect CAN bus communication failures.

- **Detection:** Bus-off condition or no message acknowledgment
- **Action:** Set communication fault, attempt bus recovery

**REQ-FLT-012:** The BMS shall detect power supply faults.

- **Monitored:** 5V, 3.3V, and auxiliary supplies
- **Thresholds:** ±10% of nominal voltage
- **Action:** Set hardware fault, enter safe shutdown if critical

**REQ-FLT-013:** The BMS shall detect contactor welding or malfunction.

- **Detection:** Unexpected voltage across open contactor
- **Action:** Set contactor fault, prevent further operation

**REQ-FLT-014:** The BMS shall detect insulation monitoring device (IMD) faults.

- **Source:** External IMD via discrete input or CAN
- **Action:** Open contactors per FSAE rules

### 3.5 Fault Prioritization and Response

**REQ-FLT-015:** The BMS shall classify faults by severity level.

- **Level 1 - Warning:** Log only, no operational impact
- **Level 2 - Fault:** Limit operation, reduce current/power
- **Level 3 - Critical:** Immediate shutdown, open contactors

**REQ-FLT-016:** The BMS shall implement fault escalation.

- **Logic:** Warning → Fault → Critical based on duration or repetition
- **Example:** Overvoltage warning for >5 seconds becomes fault

**REQ-FLT-017:** The BMS shall log all faults with timestamp.

- **Storage:** Non-volatile memory (EEPROM or Flash)
- **Data:** Fault code, timestamp, relevant sensor values
- **Capacity:** Minimum 50 fault events

---

## 4. Watchdog Requirements

### 4.1 Software Watchdog

**REQ-WDG-001:** The BMS shall implement an independent hardware watchdog timer.

- **Type:** Internal or external watchdog IC
- **Timeout:** Configurable (e.g., 100-500ms)
- **Action on timeout:** System reset

**REQ-WDG-002:** The BMS software shall refresh the watchdog timer periodically.

- **Location:** Main control loop at known execution point
- **Frequency:** Well within timeout period (e.g., every 50ms for 500ms timeout)

**REQ-WDG-003:** The BMS shall detect watchdog resets and log them.

- **Detection:** Check reset source register on startup
- **Action:** Increment watchdog reset counter, log to non-volatile memory

### 4.2 Task Monitoring

**REQ-WDG-004:** The BMS shall monitor execution time of critical tasks.

- **Tasks:** Safety checks, monitoring, communication
- **Method:** Timer-based or RTOS task monitoring
- **Action:** Set fault if task exceeds maximum execution time

**REQ-WDG-005:** The BMS shall detect missed task executions.

- **Method:** Deadline monitoring for periodic tasks
- **Action:** Set scheduling fault if task doesn't execute on time

### 4.3 Software Health Checks

**REQ-WDG-006:** The BMS shall perform periodic self-tests.

- **Tests:** RAM test, stack overflow check, critical variable validation
- **Frequency:** Every main loop iteration or slower (e.g., 1 Hz)
- **Action:** Set internal fault if test fails

**REQ-WDG-007:** The BMS shall implement stack overflow protection.

- **Method:** Stack canary, MPU, or pattern checking
- **Action:** Trigger watchdog reset or controlled shutdown

**REQ-WDG-008:** The BMS shall validate critical configuration parameters.

- **Check:** Safety thresholds, calibration values, state machine variables
- **Action:** Use safe defaults and set fault if corruption detected

---

## 5. Cell Balancing Requirements

### 5.1 Balancing Strategy

**REQ-BAL-001:** The BMS shall support passive cell balancing.

- **Method:** Discharge high cells through resistors via ADBMS6030b
- **Control:** Individual cell selection and enable/disable

**REQ-BAL-002:** The BMS shall balance cells when imbalance exceeds threshold.

- **Threshold:** Configurable (e.g., 20mV difference from average)
- **Target:** Bring all cells within configurable tolerance (e.g., 10mV)

**REQ-BAL-003:** The BMS shall prioritize balancing during charging.

- **Condition:** Only balance when pack voltage is high (>90% SOC)
- **Reason:** Most effective and safest time for balancing

**REQ-BAL-004:** The BMS shall support manual balancing mode.

- **Trigger:** External command or diagnostic interface
- **Purpose:** Maintenance and testing

### 5.2 Balancing Safety

**REQ-BAL-005:** The BMS shall monitor temperature during balancing.

- **Limit:** Disable balancing if cell or module temperature exceeds threshold
- **Threshold:** Configurable (e.g., 45°C)

**REQ-BAL-006:** The BMS shall limit balancing current and duration.

- **Current:** Per ADBMS6030b datasheet specifications
- **Duration:** Maximum continuous balancing time (e.g., 30 minutes)
- **Cool-down:** Minimum rest period between balancing cycles

**REQ-BAL-007:** The BMS shall not balance while vehicle is in active operation.

- **Condition:** Disable balancing during discharge >X amps
- **Reason:** Avoid additional heat generation during high power

**REQ-BAL-008:** The BMS shall disable balancing if any critical fault is present.

- **Faults:** Overvoltage, over-temperature, sensor failures
- **Action:** Immediately stop balancing

### 5.3 Balancing Monitoring

**REQ-BAL-009:** The BMS shall track balancing statistics per cell.

- **Data:** Total balancing time, energy dissipated, last balance time
- **Purpose:** Identify weak cells and verify balancing effectiveness

**REQ-BAL-010:** The BMS shall verify balancing is functioning.

- **Check:** Monitor voltage change during active balancing
- **Action:** Set fault if no voltage change detected (balancing circuit failure)

---

## 6. Communication Protocol Requirements

### 6.1 CAN Bus Communication

**REQ-COM-001:** The BMS shall communicate via CAN bus.

- **Speed:** Configurable (typically 250 kbps, 500 kbps, or 1 Mbps)
- **Standard:** CAN 2.0B (29-bit extended identifiers)
- **Compliance:** Follow team's CAN database definition

**REQ-COM-002:** The BMS shall transmit periodic status messages.

- **Message 1:** Pack voltage, current, SOC, state
    - Frequency: 10 Hz
- **Message 2:** Min/max cell voltages, delta
    - Frequency: 10 Hz
- **Message 3:** Min/max temperatures
    - Frequency: 1 Hz
- **Message 4:** Fault status, warning flags
    - Frequency: 10 Hz (or event-driven)

**REQ-COM-003:** The BMS shall transmit individual cell voltages.

- **Format:** Multiple messages covering all cells
- **Frequency:** 1 Hz (can be lower priority)

**REQ-COM-004:** The BMS shall transmit diagnostic data.

- **Content:** Balancing status, SOH, sensor health, statistics
- **Frequency:** On request or slow periodic (0.1 Hz)

**REQ-COM-005:** The BMS shall receive and process CAN commands.

- **Commands:**
    - Contactor control (open/close request)
    - Balancing enable/disable
    - Calibration/configuration updates
    - Reset commands
- **Security:** Implement basic command validation and source checking

**REQ-COM-006:** The BMS shall implement CAN error handling.

- **Monitoring:** Error counters, bus-off detection
- **Recovery:** Automatic bus recovery after error conditions
- **Reporting:** Transmit communication fault status

### 6.2 isoSPI Communication (ADBMS6030b)

**REQ-COM-007:** The BMS shall communicate with ADBMS6830b via isoSPI.

- **Initialization:** Configure ADBMS6030b on startup
- **Commands:** Read voltages, read temperatures, control balancing, read status

**REQ-COM-008:** The BMS shall implement error detection for isoSPI.

- **Method:** CRC checking per ADBMS6030b protocol
- **Action:** Retry on error, set fault if persistent failures

**REQ-COM-009:** The BMS shall handle ADBMS6030b daisy chain communication.

- **Support:** Multiple slave devices if pack requires
- **Addressing:** Proper device addressing and data routing

**REQ-COM-010:** The BMS shall read all voltages and temperatures in one cycle.

- **Efficiency:** Minimize communication overhead
- **Timing:** Complete read cycle within allocated time budget

### 6.3 Diagnostic Interface

**REQ-COM-011:** The BMS shall support diagnostic communication.

- **Interface:** CAN, UART, or USB (TBD based on hardware)
- **Purpose:** Calibration, testing, firmware updates, debugging

**REQ-COM-012:** The BMS shall provide access to internal data via diagnostic interface.

- **Data:** All sensor readings, calculated values, fault logs, statistics
- **Format:** Human-readable or standardized protocol (e.g., UDS)

---

## 7. Safety Logic Requirements

### 7.1 Contactor Control

**REQ-SFT-001:** The BMS shall control high-voltage contactors safely.

- **Outputs:** Main positive, main negative, precharge contactors
- **Method:** GPIO outputs with appropriate drive circuitry

**REQ-SFT-002:** The BMS shall implement precharge sequence.

- **Sequence:**
    1. Verify all contactors open and no faults
    2. Close negative contactor
    3. Close precharge contactor
    4. Wait for bus voltage to reach target (e.g., 95% of pack voltage)
    5. Close main positive contactor
    6. Open precharge contactor
- **Timeout:** Abort and set fault if precharge doesn't complete within time limit

**REQ-SFT-003:** The BMS shall verify contactor states.

- **Method:** Read back actual contactor position via feedback or voltage sensing
- **Action:** Set fault if commanded state doesn't match actual state

**REQ-SFT-004:** The BMS shall open contactors on critical faults.

- **Response Time:** < 100ms from fault detection
- **Sequence:** Open main positive first, then precharge, then negative
- **Bypass:** Hardware shutdown circuit shall operate independent of software

**REQ-SFT-005:** The BMS shall prevent contactor closure if faults are present.

- **Check:** Verify no critical faults before closing contactors
- **Override:** Allow manual override only via diagnostic interface for testing

### 7.2 Shutdown Circuit Integration

**REQ-SFT-006:** The BMS shall interface with FSAE shutdown circuit.

- **Input:** Monitor shutdown circuit state
- **Output:** BMS OK signal to shutdown circuit
- **Logic:** Assert BMS OK only when all safety conditions met

**REQ-SFT-007:** The BMS shall detect shutdown circuit activation.

- **Detection:** Monitor shutdown circuit state via digital input
- **Action:** Open contactors and enter safe state

**REQ-SFT-008:** The BMS shall implement interlock monitoring.

- **Input:** HV interlock loop status
- **Action:** Open contactors immediately if interlock opens

### 7.3 Safe State Management

**REQ-SFT-009:** The BMS shall enter safe state on any critical fault.

- **Safe State:**
    - Contactors open
    - Balancing disabled
    - Non-essential functions disabled
    - Fault status communicated
    - Wait for manual reset or fault clearance

**REQ-SFT-010:** The BMS shall require manual reset after critical fault.

- **Methods:** Physical reset button, power cycle, or diagnostic command
- **Purpose:** Prevent automatic recovery from serious faults

**REQ-SFT-011:** The BMS shall implement fail-safe defaults.

- **Power-up:** All contactors open, balancing off
- **Undefined state:** Assume fault condition
- **Communication loss:** Enter safe state if critical messages missing

### 7.4 Operating Limits

**REQ-SFT-012:** The BMS shall enforce current limits based on conditions.

- **Factors:** SOC, temperature, cell voltage, fault status
- **Communication:** Broadcast allowable charge/discharge current on CAN
- **Enforcement:** Open contactors if limits grossly exceeded

**REQ-SFT-013:** The BMS shall implement state-of-charge limits.

- **Maximum SOC:** Prevent charging above configurable limit (e.g., 95%)
- **Minimum SOC:** Prevent discharging below configurable limit (e.g., 10%)
- **Action:** Request current reduction, open contactors if violated

**REQ-SFT-014:** The BMS shall protect against reverse current.

- **Detection:** Current flowing in unexpected direction
- **Action:** Open contactors if reverse current exceeds threshold

---

## 8. State Machine Requirements

**REQ-STM-001:** The BMS shall implement a well-defined state machine.

- **States:** INIT, READY, PRECHARGE, ACTIVE, CHARGING, FAULT, SHUTDOWN

**REQ-STM-002:** The BMS shall enforce valid state transitions.

- **Logic:** Only allow transitions based on safety conditions
- **Invalid:** Reject and log any invalid transition attempts

**REQ-STM-003:** The BMS shall broadcast current state on CAN.

- **Frequency:** Every status message
- **Purpose:** Allow vehicle controller to coordinate operations

**REQ-STM-004:** The BMS shall log all state transitions.

- **Data:** Timestamp, previous state, new state, trigger reason
- **Storage:** Non-volatile memory for post-analysis

---

## 9. Data Logging and Storage Requirements

**REQ-LOG-001:** The BMS shall log fault events to non-volatile memory.

- **Capacity:** Minimum 50 events
- **Data:** Fault code, timestamp, sensor values at fault time

**REQ-LOG-002:** The BMS shall store configuration parameters in non-volatile memory.

- **Parameters:** Safety thresholds, calibration values, CAN IDs
- **Protection:** Checksum or CRC to detect corruption

**REQ-LOG-003:** The BMS shall maintain operational statistics.

- **Statistics:** Total energy throughput, cycle count, max/min values seen
- **Purpose:** Battery health tracking and diagnostics

**REQ-LOG-004:** The BMS shall support configuration backup and restore.

- **Method:** Via diagnostic interface
- **Purpose:** Ease of setup and recovery from corruption

---

## 10. Performance Requirements

**REQ-PRF-001:** The BMS shall complete all safety checks within timing budget.

- **Critical Loop:** Execute at minimum 10 Hz
- **Monitoring:** All voltage/temperature readings and safety evaluations
- **Margin:** Maintain at least 20% CPU time margin

**REQ-PRF-002:** The BMS shall respond to critical faults within specified time.

- **Response Time:** < 100ms from detection to contactor opening
- **Includes:** Detection, decision, GPIO actuation

**REQ-PRF-003:** The BMS shall minimize startup time.

- **Target:** < 2 seconds from power-on to READY state
- **Requirement:** Quick enough for vehicle startup sequence

**REQ-PRF-004:** The BMS shall operate reliably in automotive environment.

- **Temperature Range:** -40°C to +85°C (ambient)
- **EMC:** Meet automotive EMC standards
- **Vibration:** Withstand FSAE vibration requirements

---

## 11. Configuration and Calibration Requirements

**REQ-CFG-001:** The BMS shall support field-configurable parameters.

- **Parameters:** Safety thresholds, current limits, CAN IDs, pack specifications
- **Interface:** Diagnostic tool via CAN or UART

**REQ-CFG-002:** The BMS shall validate configuration parameters.

- **Validation:** Range checking, consistency checks
- **Action:** Reject invalid configurations, use safe defaults

**REQ-CFG-003:** The BMS shall support sensor calibration.

- **Sensors:** Voltage offset, current sensor zero and gain, temperature curves
- **Method:** Calibration mode accessible via diagnostic interface

**REQ-CFG-004:** The BMS shall maintain configuration version control.

- **Tracking:** Store configuration version and firmware version
- **Purpose:** Ensure configuration matches firmware capabilities

---

