# Raspberry PI Compute Module 4 Carrier Board

This board is designed for robotic applications at [HEIG-VD](http://heig-vd.ch), targeting the Eurobot/SwissEurobot contest.

![Board](assets/board.png)

## To Do

Things to be fixed for the next version. 

## TO DO

- [ ] **BUG** I2C conflict LTC2990, fix address
- [ ] **BUG** Inverser SCL et SDA sur TFA9879 
- [ ] **BUG** RTC not working: 32.768kHz crystal should be connected to (1,4) not (1,2)
- [ ] **BUG** Activity and power not working : Replace U16 with SN74AUC2G07DBVT
- [ ] **CHANGE** Use GPIO44,45 for I2C0 (Camera0 + DSI)
- [ ] **CHANGE** Use GPIO2,3 for I2C1 (Internal bus + Camera1 + First external I2C)
- [ ] **CHANGE** Use GPIO22,23 for I2C 6 (Second external I2C)
- [ ] **CHANGE** Reserve GPIO0,1 : ID_SD, ID_SC for HAT EEPROM 
- [ ] **CHANGE** Remove silkscreen RPI on PCB
- [ ] **CHANGE** Text Bottom side upside down
- [ ] **CHANGE** Add semantic version number on PCB
- [ ] **CHANGE** D3 et D4 too bright. Increase resistor to 470
- [ ] **CHANGE** Increase size of do not trash
- [ ] **CHANGE** Add TP to 1.8 and 3.3
- [ ] **CHANGE** Reduce Test Point. Hole: 0.6
- [ ] **CHANGE** Plated holes near electrolytic capacitor
- [ ] **CHANGE** replace electrolytic capacitor with tantalium
- [ ] **CHANGE** remove MCP9803 because useless
- [ ] **CHANGE** LTC2990, change monitored voltages
- [ ] **CHANGE** Put TFA9879 on main internal I2C
- [ ] **CHANGE** Add 0 Ohm on I2C for `RFA9879`
- [ ] **CHANGE** Replace EMC2101 with a EMC2301 ? See if we can control fan based on internal temperature
- [ ] **CHANGE**/BUG WIRE GPIO24 to TPM reset
- [ ] **CHANGE** D2, D1 not visible. Move to the side of the PCB (beside power connector)
- [ ] **CHANGE** SD card hard to remove : move SDcard toward edge
- [ ] **CHANGE** Add HAT EEPROM https://forums.raspberrypi.com/viewtopic.php?t=108134 ou alors 24CW1280T-I/OT
- [ ] **CHANGE** I2CEN should be pull-up to enable the I2C port by default
- [ ] **CHANGE** Separate 2 I2C EN ?
- [ ] **CHANGE** Symbol CM4 GPIO0, GPIO1 are SDA0 and SCL0 not 1
- [ ] **CHANGE** Add plane near 5V CM4 pins
- [ ] **CHANGE** Add tantalum capacitor near 5V CM4 pins
- [ ] **CHANGE** Add I2C active terminator LTC4311 on both external I2C ports
- [ ] **CHANGE** Isolate I2C0 from the rest?
- [ ] **CHANGE** USE I2C6 as second external I2C port
- [ ] **ENHANCE** Add debug test point on I2C
- [ ] **ENHANCE** Allow RPI to reset itself and to power off?
- [ ] **ENHANCE** Add USB boot connector (`FSUSB42MUX`)
- [ ] **QUESTION** CHANGE SD_CARD DETECT PIN ?
- [ ] **ENHANCE** Add BNo085 IMU on board
- [ ] **CHANGE** Simplify schematic to only use `MCP2562` on CAN
- [ ] **CHANGE** Protect power supplies for back-powered on I2C

## Specifications

```yaml
dimensions: [90, 90] # mm
peripherals:
    ethernet: 
        desc: Gigabit
        ports: RJ45 (HanRun HR911130A)
    usb: 
        desc: 4x ports USB 2.0 based on USB2514 IC
        ports:
            - 2x USB A
            - Header 10 positions (2x auxiliary USB)
    display:
        desc: DSI/MIPI Interface
        ports:
            - 15-pin FPC 1 mm connector
    camera:
        desc: CSI/MIPI Interface for 2 cameras
        ports:
            - 15-pin FPC 1 mm connector (Camera 0)
            - 15-pin FPC 1 mm connector (Camera 1)
    sd-card: SD Card port for micro SD
    audio: 3W Audio amplifier for mini speaker
    uart: 5-pin connector for RPLidar A3 connection
    can: CAN bus based on the MCP2515
    i2c: 2x external ports compatible with the QWIIC standard
    debug: USB debug console with integrated FTDI
    ssd: M2 SSD NVMe port accepting size 2230 and 2242
    tpm: TPM 2.0 SLB9670
    eeprom: HAT EEPROM
    rtc: Real Time Clock based on the DS1307
    fan: Active fan controller 
    power-supplies:
        3.3V: 6A
    monitoring:
        - voltages
        - currents
```
