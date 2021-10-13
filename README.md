# Raspberry PI Compute Module 4 Carrier Board

This board is designed for robotic applications at [HEIG-VD](http://heig-vd.ch), targeting the Eurobot/SwissEurobot contest.

![Board](assets/board.png)

## To Do

Things to be fixed for the next version.

## TO DO

- [x] **CHANGE** remove MCP9803 because useless
- [x] **BUG** I2C conflict LTC2990, fix address
- [x] **BUG** Invert SCL et SDA sur TFA9879
- [x] **BUG** RTC not working: 32.768kHz crystal : (1,4) not (1,2)
- [x] **CHANGE** D3 et D4 too bright. Increase resistor to 470
- [x] **CHANGE** Remove R12, R18 as GPIO above 8 are pulled-down
- [x] **CHANGE** LTC2990, change monitored voltages
- [x] **CHANGE** Put TFA9879 on main internal I2C1
- [x] **CHANGE** Use GPIO44,45 for I2C0 (Camera0 + DSI)
- [x] **CHANGE** Use GPIO22,23 for I2C 6 (Second external I2C)
- [x] **CHANGE** Use GPIO2,3 for I2C1 (Internal + Camera1 + External 1)
- [x] **CHANGE** Add fault detection on I2C buses
- [x] **CHANGE** Add HAT EEPROM 24CW1280T-I/OT
- [x] **CHANGE** Separate 2 I2C EN ? -> No, not necessary
- [x] **CHANGE** Add I2C active terminator LTC4311 on both external I2C ports
- [x] **CHANGE** USE I2C6 as second external I2C port
- [x] **ENHANCE** Add debug test point on I2C
- [x] **CHANGE** Reserve GPIO0,1 : ID_SD, ID_SC for HAT EEPROM
- [x] **CHANGE** Isolate I2C0 from the rest?
- [x] **CHANGE** I2CEN pull-up to EN I2C port by default -> Done in device-tree
- [x] **CHANGE** Add tantalum capacitor near 5V CM4 pins
- [x] **CHANGE** Add TP to 1.8 and 3.3
- [x] **CHANGE** replace electrolytic capacitor with Tantalum
- [x] **CHANGE** GPIO Port GND on 9-10
- [x] **CHANGE** Text Bottom side upside down
- [x] **CHANGE** Add semantic version number on PCB
- [x] **CHANGE** WIRE GPIO24 to TPM reset?
- [x] **CHANGE** Add plane near 5V CM4 pins
- [x] **CHANGE** D2, D1 not visible: move to the side of the PCB
- [x] **CHANGE** Console FTDI : Power and transmit LEDs
- [x] **CHANGE** Reduce Test Point. Hole: 0.6
- [x] **CHANGE** Plated holes near electrolytic capacitor
- [x] **CHANGE** Replace EMC2101 with EMC2301 (out of the box with the Kernel)
- [x] **BUG** Activity and power not working : Replace U16 with SN74AUC2G07DBVT
- [x] **CHANGE** Increase size of do not trash
- [x] **CHANGE** GPIO Port : indicate pin 1
- [x] **CHANGE** Symbol CM4 GPIO0, GPIO1 are SDA0 and SCL0 not 1
- [x] **CHANGE** Remove silkscreen CM4 on PCB
- [x] **CHANGE** Change pad size for CM4
- [x] **MINOR** Add CRH logo (Club de Robotique)
- [x] **CHANGE** Add GPIO Expander on GPIO port, keep one PWM
- [x] **CHANGE** Protect power supplies for back-powered on I2C

- [ ] **ENHANCE** Allow RPI to reset itself and to power off?
- [ ] **ENHANCE** Add USB boot connector (`FSUSB42MUX`)
- [ ] **FIX** Package for 24CW16X (SOT23-6?)

### Not changed in this version

- [ ] **CHANGE** SD card hard to remove : move SDcard toward edge -> not really
- [ ] **QUESTION** CHANGE SD_CARD DETECT PIN ? -> Not existing
- [ ] **CHANGE** Use `MCP2562` instead of `MCP2561` -> Nop : 52W leadtime
- [ ] **ENHANCE** Add BNo085 IMU on board -> Better use a Adafruit 9-DOF BNO082

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
