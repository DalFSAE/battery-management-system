# 02-Requirements

## Project Goals

- Create a distributed battery management system using the [ADBMS3680B IC](Docs\ADBMS6830B\Datasheet\ADBMS6830B_Rev0.pdf)
- Improve reliability of the tractive system, by removing long voltage tap and temperature harnesses.
- Eliminate the need for a large, commercial BMS, improve design flexibility 

## Design Requirements

### Hardware Requirements

- Must use the ADBMS3680B, and related components.
- Must meet FSAE 2026 latest ruleset.

### Software Requirements

- Must meet FSAE 2026 latest ruleset.
- Must use STM32 microcontroller.
- Software watchdog
- Constant voltage and temperature monitoring. 
- Individual cell balancing algorithm.
- CAN communication and data logging.


## Related Rules

See [06-Related_Rules](06-Related_Rules.md)

## References 

- [BatteryDesign.net: Battery Management System](https://www.batterydesign.net/battery-management-system/)
- [Wikipedia: Electric Vehicle battery](https://en.wikipedia.org/wiki/Electric_vehicle_battery)