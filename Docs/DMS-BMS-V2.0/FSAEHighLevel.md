# FSAE BMS High-Level Software Architecture

## 1. Overview
Battery Management System for FSAE Electric Vehicle compliant with 2026 Draft Rules (EV.7.3-EV.7.5)

---

## 2. Major Software Functions

### 2.1 Voltage Monitoring (EV.7.4)
- **Cell Voltage Measurement**
  - Monitor voltage of each cell in the tractive battery
  - Support for parallel cells (single measurement point)
  - Maintain measurement accuracy for limit checking
  
- **Voltage Limit Enforcement**
  - Check cell voltages against datasheet min/max limits
  - Account for measurement accuracy in limit checking
  - Generate fault on out-of-range detection

### 2.2 Temperature Monitoring (EV.7.5)
- **Cell Temperature Measurement**
  - Monitor minimum 20% of cells (lithium-based)
  - Measure at negative terminal or busbar (<10mm from terminal)
  - Ensure equal distribution of monitored cells
  
- **Temperature Limit Enforcement**
  - Enforce maximum 60°C or datasheet limit (whichever is lower)
  - Account for measurement accuracy
  - Generate fault on over-temperature condition

### 2.3 Fault Detection & Management (EV.7.3.4)
- **Fault Detection**
  - Voltage out of permitted range
  - Blown/tripped voltage sense overcurrent protection
  - Temperature out of permitted range
  - Missing/interrupted voltage measurements
  - Missing/interrupted temperature measurements
  - Internal BMS faults
  
- **Fault Response**
  - Open Shutdown Circuit immediately
  - Activate BMS Indicator Light (red)
  - Activate Tractive System Status Indicator (red flashing)
  - Latch fault until manual reset

### 2.4 Shutdown Circuit Control (EV.7.2)
- **Normal Operation**
  - Maintain Normally Open relay configuration
  - Close circuit when all conditions are safe
  
- **Fault Condition**
  - Open circuit on any detected fault
  - Maintain open state until manual reset
  - Ensure galvanic isolation between modules

### 2.5 Charging Mode Support (EV.8.3)
- **Charging Operation**
  - Continue monitoring during charging
  - Interface with Charging Shutdown Circuit
  - Disable cell balancing when Shutdown Circuit is open
  
- **Cell Balancing**
  - Only permit when Shutdown Circuit is closed
  - Disable during charging (when Shutdown Circuit open)

### 2.6 Communication & Isolation
- **Module-to-Module Communication**
  - Galvanic isolation at each Module boundary
  - Data integrity verification
  - Timeout detection for lost communications
  
- **External Interfaces**
  - Status reporting to vehicle systems
  - Fault code communication
  - Data logging interface

### 2.7 Diagnostics & Self-Test
- **Self-Monitoring**
  - Internal BMS health checks
  - Sensor connectivity verification
  - Power supply monitoring
  
- **Fault Logging**
  - Timestamp and record all faults
  - Maintain fault history
  - Support diagnostic retrieval

---

## 3. Software Block Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                         BMS CONTROL SYSTEM                          │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    MAIN CONTROL LOOP                          │ │
│  │  • System state machine                                       │ │
│  │  • Timing & scheduling                                        │ │
│  │  • Mode management (Active/Charging)                          │ │
│  └───────────┬───────────────────────────────────────────────────┘ │
│              │                                                       │
│  ┌───────────▼──────────┐  ┌─────────────────┐  ┌───────────────┐ │
│  │  VOLTAGE MONITORING  │  │   TEMPERATURE   │  │     FAULT     │ │
│  │      MODULE          │  │    MONITORING   │  │  MANAGEMENT   │ │
│  │                      │  │     MODULE      │  │    MODULE     │ │
│  │ • Cell voltage ADC   │  │ • Temp sensors  │  │ • Detection   │ │
│  │ • Parallel detection │  │ • 20% coverage  │  │ • Logging     │ │
│  │ • Min/max checking   │  │ • Limit check   │  │ • Latching    │ │
│  │ • Fuse monitoring    │  │ • Distribution  │  │ • Indicators  │ │
│  └──────────┬───────────┘  └────────┬────────┘  └───────┬───────┘ │
│             │                       │                    │          │
│             └───────────┬───────────┘                    │          │
│                         │                                │          │
│  ┌──────────────────────▼────────────────────────────────▼───────┐ │
│  │              SHUTDOWN CIRCUIT CONTROL                         │ │
│  │  • Normally Open relay driver                                 │ │
│  │  • Fault response (open on any fault)                         │ │
│  │  • Manual reset handling                                      │ │
│  │  • Precharge coordination                                     │ │
│  └──────────┬────────────────────────────────────────────────────┘ │
│             │                                                        │
│  ┌──────────▼───────────┐  ┌─────────────────┐  ┌──────────────┐  │
│  │   CELL BALANCING     │  │  COMMUNICATION  │  │ DIAGNOSTICS  │  │
│  │      CONTROL         │  │     MODULE      │  │   & SELF     │  │
│  │                      │  │                 │  │    TEST      │  │
│  │ • Only when SD       │  │ • Galvanic      │  │ • BMS health │  │
│  │   Circuit closed     │  │   isolation     │  │ • Sensor     │  │
│  │ • Disabled during    │  │ • Module-to-    │  │   checks     │  │
│  │   charging           │  │   Module comms  │  │ • Power      │  │
│  └──────────────────────┘  │ • External I/F  │  │   monitor    │  │
│                             └─────────────────┘  └──────────────┘  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
           │                        │                        │
           ▼                        ▼                        ▼
    ┌──────────┐           ┌──────────────┐         ┌─────────────┐
    │ SHUTDOWN │           │  INDICATOR   │         │   VEHICLE   │
    │ CIRCUIT  │           │    LIGHTS    │         │   SYSTEMS   │
    │  OUTPUT  │           │              │         │             │
    │          │           │ • BMS Light  │         │ • Status    │
    │ • To IRs │           │ • TS Status  │         │ • Data log  │
    │ • Precharge│          │   Indicator  │         │ • Telemetry │
    └──────────┘           └──────────────┘         └─────────────┘
```

---

## 4. Software State Machine

```
                    ┌─────────────┐
                    │   STARTUP   │
                    │             │
                    │ • Init HW   │
                    │ • Self test │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │    IDLE     │
                    │             │
                    │ • No TS     │
            ┌───────│ • Standby   │◄──────┐
            │       └──────┬──────┘       │
            │              │              │
      TS Active      Charging Start    Manual
         OR              │              Reset
    Charging Start       │                │
            │              │                │
            │       ┌──────▼──────┐         │
            │       │  CHARGING   │         │
            │       │             │         │
            │       │ • Monitor   │         │
            │       │ • No balance│         │
            │       └──────┬──────┘         │
            │              │                │
            └──────►┌──────▼──────┐         │
                    │   ACTIVE    │         │
                    │             │         │
                    │ • Monitor   │         │
                    │ • Balance OK│         │
                    └──────┬──────┘         │
                           │                │
                    Fault Detected          │
                           │                │
                           ▼                │
                    ┌─────────────┐         │
                    │    FAULT    │         │
                    │             │         │
                    │ • SD open   │         │
                    │ • Lights on │─────────┘
                    │ • Latched   │
                    └─────────────┘
```

---

## 5. Key Design Considerations

### 5.1 Safety Requirements
- Shutdown Circuit must open within 100ms of fault detection (EV.7.3.5)
- Fault must latch until manual reset (EV.7.2.3)
- Driver cannot reset from inside vehicle
- Independent circuits for BMS, IMD, and BSPD (EV.7.1.4)

### 5.2 Compliance Requirements
- Galvanic isolation at module boundaries (EV.7.3.2)
- Voltage sense overcurrent protection monitoring (EV.7.4.3)
- Temperature sensor electrical isolation (EV.7.5.7)
- Support both Active and Charging modes (EV.7.3.1)

### 5.3 Monitoring Coverage
- **Voltage**: Every cell (or parallel group)
- **Temperature**: Minimum 20% of cells, equally distributed
- **Location**: Negative terminal or busbar <10mm from terminal

### 5.4 Indicators
- **BMS Indicator Light**: Red, visible to driver, marked "BMS"
- **TS Status Indicator**: Red flashing (2-5 Hz) on BMS or IMD fault

---

## 6. Interface Specifications

### 6.1 Inputs
- Cell voltage measurements (ADC channels)
- Temperature sensor readings (thermistors/RTDs)
- Fuse status signals
- Manual reset signal
- Charging mode signal

### 6.2 Outputs
- Shutdown Circuit control (relay driver)
- BMS indicator light (red LED)
- TS status indicator (red LED, flashing capable)
- Cell balancing control signals
- Status/fault data (CAN/serial)

### 6.3 Power
- Derived from GLV system
- Must operate during both Active and Charging modes
- Isolated power for module-to-module communication

---

## 7. Software Modules Summary

| Module | Function | Key Requirements |
|--------|----------|------------------|
| Voltage Monitor | Cell voltage measurement | All cells, min/max limits |
| Temperature Monitor | Cell temperature measurement | 20% coverage, 60°C max |
| Fault Manager | Detect and log faults | 6 fault types, latching |
| Shutdown Control | Open/close circuit | Normally open, fault response |
| Balancing | Cell voltage balancing | Only when SD closed |
| Communication | Data exchange | Galvanic isolation |
| Diagnostics | Self-test and health | BMS internal monitoring |

---

## 8. Future Enhancements (Optional)
- State of Charge (SOC) estimation
- State of Health (SOH) tracking
- Predictive maintenance algorithms
- Enhanced data logging and analysis
- Thermal management optimization
- Energy optimization strategies

