# Raspberry PI Compute Module 4 Carrier Board

This board is designed for robotic applications at [HEIG-VD](http://heig-vd.ch), targeting the Eurobot/SwissEurobot contest.

![Board](assets/board.png)

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
