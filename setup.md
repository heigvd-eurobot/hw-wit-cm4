# Setup PI4 board

## Ubuntu

/boot/firmware/usercfg.txt
```sh
dtparam=i2c_arm=on
dtparam=i2s=on
dtparam=spi=on


dtoverlay=mcp2515-can0,oscillator=16000000,interrupt=25
dtoverlay=spi0-hw-cs

dtoverlay=spi-bcm2835

# Disable Bluetooth for energy save
dtoverlay=disable-bt

dtoverlay=i2c0,pins_44_45
dtoverlay=i2c1,pins_2_3
dtoverlay=i2c6,pins_22_23

dtoverlay=i2c-rtc,ds1307,addr=0x68
dtoverlay=dwc2,dr_mode=host

```


## Informations

- Quad-core ARM Cortex-A72 (ARM v8) 1.5 GHz
- BCM2711 

```
pi@raspberrypi:~ $ sudo vi /boot/config.txt
pi@raspberrypi:~ $ sudo systemctl disable hciuart
Removed /etc/systemd/system/multi-user.target.wants/hciuart.service.
sudo systemctl disable snapd
```

- `/boot/`
  - `bootcode.bin` 	2nd Stage bootloader
  - `start.elf`     3rd stage bootloader
  - `.dtb`          Compiled device trree
  - `kernel.img`    Kernel

## Device tree

- [bcm2711-rpi-4-b.dts](https://github.com/torvalds/linux/blob/master/arch/arm/boot/dts/bcm2711-rpi-4-b.dts)
- [Fast boot](https://www.furkantokac.com/rpi3-fast-boot-less-than-2-seconds/)
- [dt-blob.dts](https://github.com/raspberrypi/firmware/blob/master/extra/dt-blob.dts)
- 
For debug: add `dtdebug=1` to `/boot/config.txt`, 

```bash
sudo vcdbg log msg
```

```
sudo apt-get install device-tree-compiler
sudo dtc -I dts -O dtb -o /boot/dt-blob.bin dt-blob.dts
```

## Tools

```
sudo rpi-update
```

- boot/config.txt
- vcgencmd

```
systemd-analyze blame
systemd-analyze critical-chain

pi@raspberrypi:~ $ vcgencmd measure_clock arm
frequency(48)=600169920
pi@raspberrypi:~ $ vcgencmd mesure_temp
error=1 error_msg="Command not registered"
Use 'vcgencmd commands' to get a list of commands
pi@raspberrypi:~ $ vcgencmd measure_temp
temp=47.7'C
pi@raspberrypi:~ $ vcgencmd get_throttled
throttled=0x0
```

```
$ vcgencmd measure_volts core
volt=0.8400V
$ vcgencmd measure_temp
temp=44.8'C
vcgencmd get_mem arm
arm=896M
pi@raspberrypi:~ $ vcgencmd get_mem gpu
gpu=128M
```

## Config

- `[all]` Resets all previous filters for all devices
- `[pi4]` Only for pi4, or `[cm4]` only for cm4

```ini
# I2C
dtoverlay=i2c3,pins_2_3
```

Kernel8 64 bits: mettre dans `config` `kernel=kernel8.img`.


Commandline enable console. We remove the `quiet` word to allow dmesg on output

```
cat /boot/cmdline.txt
console=serial0,115200 console=tty1 root=PARTUUID=e974d811-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait splash plymouth.ignore-serial-consoles
```

## Investigation

### Stress test 

- 39.4 °C Fan full power stress -c 4 -t 900s

### Manuel

#### Enable Serial Console

Raspi-config/3 Interface Options/P6 Serial Port

Alternativement dans `boot/config.txt` dans `[all]` ajouter 

```
enable_uart=1
```

Pour se connecter utilser `115200` bauds.

### I2C

Si quelque chose est connecté sur I2C3 externe et que le bus n'est
pas alimenté. Il bloque L'i2c. Il faut activer le bus avec:

```
echo "27" > /sys/class/gpio/export                        
sudo echo "out" > /sys/class/gpio/gpio27/direction        
sudo echo "1" > /sys/class/gpio/gpio27/value              
```

Je peux lire sur `i2cdetect -y 3` les périphériques suivants:

- I2C3 0x29 VL6180X - Time Of Flight distance sensor [OK]
- I2C3 0x2e PAC1720 - Current sensor [OK]
- I2C3 0x41 TMF8801 1D Time-of-Flight Sensor [OK]
- I2C3 0x49 MCP9803 Temperature Sensor [OK]
- I2C3 0x4c LTC2990 Voltage Monitoring
- I2C3 0x68 DS1307 RTC
- I2C3 0x77 ? fan controller ?

- I2C0 0x38 DFRobot 7'' 800x400 TFT
- I2C0 0x45 DFRobot 7'' 800x400 TFT (FT6x06 ?)

J'utilise 3 ports I2C:

I2C0 (GPIO0,1): camera 0
I2C (GPIO2,3): camera 1 + devices


#### PAC1720

Ca marche j'arrive à lire quelque chose mais il faut calibrer. Je lit:

1.18A sur le 3.3V. Ce qui me semble beaucoup trop.

DEVICE_ADDRESS = 0x2e

DEVICE_REG_VSENSE1_RESULT_LOW = 0x0e
DEVICE_REG_VSENSE1_RESULT_HIGH = 0x0d

DEVICE_REG_VSENSE2_RESULT_LOW = 0x10
DEVICE_REG_VSENSE2_RESULT_HIGH = 0x0f


DEVICE_REG_VSOURCE1_RESULT_LOW = 0x12
DEVICE_REG_VSOURCE1_RESULT_HIGH = 0x11

DEVICE_REG_VSOURCE2_RESULT_LOW = 0x14
DEVICE_REG_VSOURCE2_RESULT_HIGH = 0x13


DEVICE_REG_PRODUCT_ID = 0xfd
DEVICE_REG_MANUFACTURER_ID = 0xfe
DEVICE_REG_REVISION = 0xff

if hex(bus.read_byte_data(DEVICE_ADDRESS, DEVICE_REG_PRODUCT_ID)) != 0x81:
    raise ValueError('Bad revision')
if hex(bus.read_byte_data(DEVICE_ADDRESS, DEVICE_REG_MANUFACTURER_ID)) != 0x5d:
    raise ValueError('Bad manufacturer')

if (hex(bus.read_byte_data(0x2e, 0x04)) & 0x80) 
    print("Conversion cycle complete")

low = bus.read_byte_data(DEVICE_ADDRESS, DEVICE_REG_VSENSE1_RESULT_LOW)
high = bus.read_byte_data(DEVICE_ADDRESS, DEVICE_REG_VSENSE1_RESULT_HIGH)
v1 = low + (high << 8)

VSRC_SAMP_TIME =
den = 
fsv = 40 - 40 / den
v1 = fsv * vsource / den 


low = bus.read_byte_data(DEVICE_ADDRESS, DEVICE_REG_VSOURCE1_RESULT_LOW)
high = bus.read_byte_data(DEVICE_ADDRESS, DEVICE_REG_VSOURCE1_RESULT_HIGH)
vsource1 = (low + (high << 8) >> 5)

low = bus.read_byte_data(DEVICE_ADDRESS, DEVICE_REG_VSOURCE2_RESULT_LOW)
high = bus.read_byte_data(DEVICE_ADDRESS, DEVICE_REG_VSOURCE2_RESULT_HIGH)
vsource2 = (low + (high << 8) >> 5)
return vsource2 / 0x400 * 20


C1CSS 0101b Sampling time (4-5)       -> 80 ms
C1SA  00b Consecutive average (4-4)
C1SR  11b Full scale rage (4-6)       -> 80mV

résolution = 39.06uV

"""Current 3.3V"""
RSENSE = 0.01
low = bus.read_byte_data(DEVICE_ADDRESS, DEVICE_REG_VSENSE2_RESULT_LOW)
high = bus.read_byte_data(DEVICE_ADDRESS, DEVICE_REG_VSENSE2_RESULT_HIGH)
i2 = (low + (high << 8)) * 39.06e-6 / RSENSE


#### MCP9803

On arrive a lire quelque chose mais ce circuit n'est pas très utile...
Il prends de la place pour pas grand chose.

```python
import smbus
import time

DEVICE_ADDRESS = 0x49
# Get I2C bus
bus = smbus.SMBus(1)


# Select configuration register, 0x01(1)
#		0x60(96)	Continuous conversion mode, Power-up, Comparator mode
bus.write_byte_data(DEVICE_ADDRESS, 0x01, 0x60)

time.sleep(0.5)


# Read data back from 0x00(00), 2 bytes
# cTemp MSB, cTemp LSB
data = bus.read_i2c_block_data(DEVICE_ADDRESS, 0x00, 2)

# Convert the data to 12-bits
ctemp = (data[0] * 256 + data[1]) / 16.0
if ctemp > 2047 :
	ctemp -= 4096
ctemp = ctemp * 0.0625
ftemp = ctemp * 1.8 + 32

# Output data to screen
print("%.2f °C" % ctemp)

```

#### LTC2990

Arrive a lire des choses mais toujours 0...

In [166]: [bus.read_byte_data(0x4c, reg) for reg in range(10)]
Out[166]: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]

Du coup je comprend (voir plus bas) Il faut modifier pin 9 (ADR1 ) pour la mettre à +3.3V -> 0x4c -> 0x4e

```python
#!/usr/bin/python
# 
# LTC 2990 Quad I2C Voltage, Current and Temperature Monitor
# Retrieves LTC2990 register and performs some basic operations.
# Specs: http://www.linear.com/product/LTC2990
# Source: https://github.com/rfrht/ltc2990
#
# Copyright (C) 2015 Rodrigo A B Freire
# 
# This program is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License
# as published by the Free Software Foundation; either version 2
# of the License, or (at your option) any later version.
# 
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
# 
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301,
# USA.

import smbus
import time
bus = smbus.SMBus(1)   # 512-MB RPi the bus is 1. Otherwise, bus is 0.

# Pro tip: Ensure that ADR0 and ADR1 are grounded. Do not let them
# open. Otherwise, the i2c address will randomly change.a
address = 0x4e         # I2C chip address
mode = 0x5f            # Register 0x01 mode select. V1, V2, V3, V4

try:
  if bus.read_byte_data(address, 0x01) != mode: # If current IC mode != program mode
    bus.write_byte_data(address, 0x01, mode)    # Initializes the IC and set mode
    bus.write_byte_data(address, 0x02, 0x00)    # Trigger a initial data collection
    time.sleep(1)				# Wait a sec, just for init
except err:
  print(err)

bus.write_byte_data(address, 0x02, 0x00) # Trigger a data collection

r0 = bus.read_byte_data(address, 0x00) # Status
r1 = bus.read_byte_data(address, 0x01) # Control - mode select
r4 = bus.read_byte_data(address, 0x04) # Temp. Int. MSB
r5 = bus.read_byte_data(address, 0x05) # Temp. Int. LSB
r6 = bus.read_byte_data(address, 0x06) # V1, V1 - V2 or TR1 MSB
r7 = bus.read_byte_data(address, 0x07) # V1, V1 - V2 or TR1 LSB
r8 = bus.read_byte_data(address, 0x08) # V2, V1 - V2 or TR1 MSB
r9 = bus.read_byte_data(address, 0x09) # V2, V1 - V2 or TR1 LSB
ra = bus.read_byte_data(address, 0x0a) # V3, V3 - V4 or TR2 MSB
rb = bus.read_byte_data(address, 0x0b) # V3, V3 - V4 or TR2 LSB
rc = bus.read_byte_data(address, 0x0c) # V4, V3 - V4 or TR2 MSB
rd = bus.read_byte_data(address, 0x0d) # V4, V3 - V4 or TR2 LSB
re = bus.read_byte_data(address, 0x0e) # Vcc MSB
rf = bus.read_byte_data(address, 0x0f) # Vcc LSB

# Check for a specific bit value
def get_bit(number, bit):
  return (number >> bit) & 1 

def temperature(msb,lsb):
  msb = format(msb, '08b')
  msb = msb[3:]
  lsb = format(lsb, '08b')
  temp = msb + lsb
  temp = int(temp, 2)/16
  return temp

def voltage(msb,lsb):
  msb = format(msb, '08b')
  msb = msb[1:]
  lsb = format(lsb, '08b')
  signal = get_bit(int(msb, 2),6)
  #print "positive:0 negative:1 %s" %signal
  volt = msb[1:] + lsb
  volt = int(volt, 2) * 0.00030518
  return volt


print("Int. Temp. : %s Celsius" %temperature(r4,r5))
print("Voltage V1 : %s V" %voltage(r6,r7))
print("Voltage V2 : %s V" %voltage(r8,r9))
print("Voltage V3 : %s V" %voltage(ra,rb))
print("Voltage V4 : %s V" %voltage(rc,rd))

# If you want to use TR, use the temperature(msb,lsb) function to get the
# value. I.e., if you have set the mode TR1 & TR2 (mode 0x5d),
# Comment the print "Voltage" lines and uncomment these ones:

# TR1
# print "Temperature TR1: %s Celsius" %temperature(r6,r7)
# TR2
# print "Temperature TR2: %s Celsius" %temperature(ra,rb)

# And print the supply voltage:
vin = voltage(re,rf) + 2.5
print("Vin        : %s V" %vin)
```

On lit ceci : 

```
Int. Temp. : 29.5625 Celsius
Voltage V1 : 3.29930098 V        3.3VPI
Voltage V2 : 3.96337266 V        5V (étrange !)
Voltage V3 : 3.3356174 V         3.3V
Voltage V4 : 0.02716102 V        VBAT
Vin        : 3.3361932 V         Alimentation
```

- Du coup inutile de mesurer 3.3V puisque l'on a VIN. 
- Du coup rajouter 1.8V de la PI
- Devrait être alimenté en 5V pour lire le 5V (voir si possible)
- Voir si compatible I2C si 5V
- Déjà température donc bien en remplacement du MCP9803

#### EMC2101

100 1100
 4    c    Ohhh ! Ceci explique cela...

Oh yeah... J'arrive à faire tourner le ventilateur !

```python
In [15]: bus.write_byte_data(0x4c, 0x4c, 0x1f)

In [16]: bus.write_byte_data(0x4c, 0x4c, 0x10)

In [17]:
```

Et la température interne et externe fonctionne

```python
In [12]: bus.read_byte_data(0x4c, 0x00)
Out[12]: 32

In [13]: bus.read_byte_data(0x4c, 0x01)
Out[13]: 33
```

Donc c'est bien !


https://github.com/neg2led/cm4io-fan/blob/master/emc2301/USAGE.md

Voir pour écrire un module noyau

#### RTC

Changer le port ?

dtoverlay=i2c-rtc,ds1307,addr=0x68

Le quartz est mal soudé, pas le bon numéro de pin.

En activant le module overlay ça marche pas par contre avec ceci, ca marche:

```console
# modprobe i2c-dev
# modprobe rtc-ds1307
# echo ds1307 0x68 >> /sys/bus/i2c/devices/i2c-3/new_device
# i2cdetect -y 3
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- 2e --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- -- 49 -- -- 4c -- 4e --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- UU -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- 77
# hwclock -r
2021-10-03 15:55:56.747064+02:00
```

Le probkème est que l'overlay va chercher sur I2C1. 

> Rather like the other current question on using an ads7846 on spi1, this shows up a hole in the DT overlay mechanism - it is almost impossible to parameterise an overlay for different I2C or SPI busses. For now you will have to copy the standard i2c-rtc overlay and modify it to reference i2c-3 instead. Actually it would be better if you make it an i2c-rtc-gpio overlay instead - a combination of i2c-gpio and i2c-rtc.


### RPLidar

Il est possible de configurer le Hardware PWM correctement

git clone --depth=1 https://github.com/Slamtec/rplidar_sdk.git

```
dtoverlay pwm pin=12 func=4
```

```
echo 0 > /sys/class/pwm/pwmchip0/export
echo 40000 > /sys/class/pwm/pwmchip0/pwm0/period
echo 10000 > /sys/class/pwm/pwmchip0/pwm0/duty_cycle 
echo 1 > /sys/class/pwm/pwmchip0/pwm0/enable
```


### Problème GPIO

Il semblerait qu'avec la CM4 il ne soit pas possible de lire et d'écrire les GPIO directements. 

```
$ gpio readall
Oops - unable to determine board type... model: 20
$ gpio mode 13 out
$ gpio write 13 1
```

Je n'ai rien ni sur la PI4 ni sur la CM4 IO Board. 
Je vais installer une autre CM4 ave eMMC. Il faut Linux, je prend une RPI4 et installe Raspberry Pi Desktop pour pouvoir installer raspberrypi/usbboot...
Bon ça marche pas. Mais il semblerait qu'il existe quelque chose pour Windows. 
Les drivers mettent des plombes à s'installer mais ça marche
Je peux flasher la eMMC avec Raspberry Pi OS

J'arrive a démarrer. Je peux ping avec 
C:\Users\ycr>ping -4 raspberrypi.local


Ca marche pas non plus avec les GPIO. Par contre j'arrive avec ceci : 

```
echo "13" > /sys/class/gpio/export                    
echo "out" > /sys/class/gpio/gpio13/direction         
echo "1" > /sys/class/gpio/gpio13/value               
```

J'arrive à faire de même sur la PI4. Donc les IO marchent mais pas avec "gpio"

Aparamment c'est normal car gpio est obsolète. Il faut passer par Python 

```python
>>> import RPi.GPIO as GPIO
>>> GPIO.setmode(GPIO.BCM)
>>> GPIO.setup(27, GPIO.OUT)
>>> GPIO.output(27, GPIO.HIGH)
```

### Problème USB

Je ne trouve pas d'USB avec :

```
$ lsusb
$ cat /dev/*USB*
```

Ni sur la PI4 ni sur la CM4 IO Board. Est-ce que les GPIO sont mortes?

Il semblerait que l'USB est désactivé par défaut. Il faut ajouter dans `/boot/config.txt`

```
dtoverlay=dwc2,dr_mode=host
```

Après cela j'ai ceci sur la IO board:

```
lsusb
Bus 001 Device 003: ID 046d:c31c Logitech, Inc. Keyboard K120
Bus 001 Device 002: ID 0424:2514 Standard Microsystems Corp. USB 2.0 Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

J'ai essayé sur la PI4 et j'ai aussi l'USB. Donc ça marche !
OUF !

### Audio / Sound

Rien avec ceci : 

```
speaker-test -t wav -c 2
```

Rien non plus sur l'i2c, je n'ai pas 3.3V sur SDA et SCL je suppose qu'il n'y a pas de pullup internes. 

Bon y'avait une inversion sur SDA et SCL. C'est maintenant résolu. 

Le chip à l'air capricieux, je vais m'assurer de mettre des 0R pour la suite

Cela fonctionne avec ceci :

```
wget https://raw.githubusercontent.com/Darmur/bassfly-uhat/master/scripts/install_bassfly_raspios.sh
chmod a+x install_bassfly_raspios.sh
sudo ./install_bassfly_raspios.sh
```

Pour lire un fichier mp3 j'ai utilisé ceci:

```
sudo apt-get install moc
mocp
```

## NVME M2

```
pi@raspberrypi:~ $ sudo dd if=/dev/zero of=/mnt/test1.img bs=1G count=1 oflag=dsync
1+0 records in
1+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 7.96952 s, 135 MB/s

pi@raspberrypi:~ $ sudo dd if=/dev/zero of=/mnt/test1.img bs=100M count=1 oflag=dsync
1+0 records in
1+0 records out
104857600 bytes (105 MB, 100 MiB) copied, 0.943747 s, 111 MB/s
pi@raspberrypi:~ $ sudo dd if=/dev/zero of=/tmp/test1.img bs=100M count=1 oflag=dsync
1+0 records in
1+0 records out
104857600 bytes (105 MB, 100 MiB) copied, 1.99617 s, 52.5 MB/s

```

## Display

vcgencmd display_power 0 # Disable display
vcgencmd display_power 1 # Enable display

## Camera/DSI Display

Rien ne marche mais il semblerait que ce soit désactivé par défaut. Une approche est de faire ceci mais je ne sais pas ce que ca fait...

```
sudo apt-get install p7zip-full
wget https://www.waveshare.com/w/upload/4/41/CM4_dt_blob.7z
7z x CM4_dt_blob.7z -O./CM4_dt_blob
sudo chmod 777 -R CM4_dt_blob
cd CM4_dt_blob/
#If you want to use both cameras and DSI0
sudo  dtc -I dts -O dtb -o /boot/dt-blob.bin dt-blob-disp0-double_cam.dts
#If you want to ue both cameras and DSI1
sudo  dtc -I dts -O dtb -o /boot/dt-blob.bin dt-blob-disp1-double_cam.dts
```

## TPM

https://github.com/raspberrypi/firmware/blob/abc347435437f6e2e85b5f367e75a5114882eaa3/boot/overlays/README
```
Name:   tpm-slb9670
Info:   Enables support for Infineon SLB9670 Trusted Platform Module add-on
        boards, which can be used as a secure key storage and hwrng,
        available as "Iridium SLB9670" by Infineon and "LetsTrust TPM" by pi3g.
Load:   dtoverlay=tpm-slb9670
Params: <None>
```

Il faut donc activer l'overlay avec 

```
dtparam=spi=on
dtoverlay=tpm-slb9670
```

Mais... D'où vient cette description? Bref, il semblerait que cela boot sur le bon chip select...
https://github.com/alexmurray/rpi-tpm2-slb9670-hwe/blob/master/tpm-slb9670-overlay.dts

```
[    6.955636] tpm_tis_spi spi0.1: 2.0 TPM (device-id 0x1B, rev-id 22)
[    6.959239] CAN device driver interface
[    6.961102] tpm tpm0: A TPM error (256) occurred attempting the self test
[    6.961127] tpm tpm0: starting up the TPM manually
```

Peut-être que le self test ne marche pas parce que GPIO24 (RESET) n'est pas cablé.

Chez moi : chez eux : fonction

- GPIO6 : GPIO25 : nPIRQ
-       : GPIO24 : RESET

```
	fragment@2 {
		target = <&spi0>;
		__overlay__ {
			/* needed to avoid dtc warning */
			#address-cells = <1>;
			#size-cells = <0>;
			slb9670: slb9670@1 {
				compatible = "infineon,slb9670";
				reg = <1>;	/* CE1 */
				#address-cells = <1>;
				#size-cells = <0>;
				spi-max-frequency = <32000000>;
				status = "okay";
			};

		};
	};
```

```
/*
 * Device Tree overlay for the Infineon SLB9670 Trusted Platform Module add-on
 * boards, which can be used as a secure key storage and hwrng.
 * available as "Iridium SLB9670" by Infineon and "LetsTrust TPM" by pi3g.
 */

/dts-v1/;
/plugin/;

/ {
	compatible = "brcm,bcm2835", "brcm,bcm2708", "brcm,bcm2709";

	fragment@0 {
		target = <&spi0>;
		__overlay__ {
			compatible = "spi-gpio";
			pinctrl-names = "default";
			pinctrl-0 = <&spi0_gpio7>;
			gpio-sck = <&gpio 11 0>;
			gpio-mosi = <&gpio 10 0>;
			gpio-miso = <&gpio 9 0>;
			cs-gpios = <&gpio 7 1>;
			spi-delay-us = <0>;
			#address-cells = <1>;
			#size-cells = <0>;
			status = "okay";

			/* for kernel driver */
			sck-gpios = <&gpio 11 0>;
			mosi-gpios = <&gpio 10 0>;
			miso-gpios = <&gpio 9 0>;
			num-chipselects = <1>;

			slb9670: slb9670@0 {
				compatible = "infineon,slb9670", "tis,tpm2-spi", "tcg,tpm_tis-spi";
				reg = <0>;
				gpio-reset = <&gpio 24 1>;
				#address-cells = <1>;
				#size-cells = <0>;
				status = "okay";

				/* for kernel driver */
				spi-max-frequency = <1000000>;
			};
		};
	};

	fragment@1 {
		target = <&spi0_gpio7>;
		__overlay__ {
			brcm,pins = <7 8 9 10 11 24>;
			brcm,function = <0>;
		};
	};

	fragment@2 {
		target = <&spidev0>;
		__overlay__ {
			status = "disabled";
		};
	};

	fragment@3 {
		target = <&spidev1>;
		__overlay__ {
			status = "disabled";
		};
	};
};
```

https://raw.githubusercontent.com/PaulKissinger/LetsTrust/master/Scripts/tpm2_install.sh

pi@raspberrypi:~/tpm2-tools/tools $ echo "Les chiens sont des animaux a quatre pattes" | sudo tpm2_hash
hash(sha1):f37d2151fc0b0de5cf66b1db96becd1762ed2352

Donc ca semble marcher

## CAN Bus

```
dtoverlay=mcp2515-can0,oscillator=16000000,interrupt=25
```

https://www.beyondlogic.org/adding-can-controller-area-network-to-the-raspberry-pi/

Je vois le CS qui descend mais rien derrière...

Tient... Il semblerait que j'ai un MCP2510 et non un 2515

Je viens de remplacer le MCP2510 par un MCP2515 et c'est bon, il est initialisé !

```
pi@raspberrypi:~ $ sudo dmesg | grep mcp
[    7.850691] mcp251x spi0.0 can0: MCP2515 successfully initialized.
```

La prochaine étape est de générer du traffic pour voir... 

## To Buy

- [ ] Blue LED for D5
- [ ] FSUSB42 on AliExpress
- [ ] SN74AUC2G07DBVT
- [ ] Find connecteur RP Lidar pas adapté. Je ne trouve pas le bon
- [ ] 24CW1280-I/OT 
- [ ] LTC4311


## Hat 

The biggest change with HAT add-on boards versus older boards designed for models A and B is that the 40W header has 2 special pins (ID_SC and ID_SD) that are reserved exclusively for attaching an 'ID EEPROM'. The ID EEPROM contains data that identifies the board, tells the Pi how the GPIOs need to be set up and what hardware is on the board. This allows the add-on board to be automatically identified and set up by the Pi software at boot time including loading all the necessary drivers.

The ID_SC and ID_SD pins must only be used for attaching a compatible ID EEPROM. Do not use ID_SC and ID_SD pins for anything except connecting an ID EEPROM, if unused these pins must be left unconnected

https://github.com/raspberrypi/hats/blob/master/eeprom-format.md

On va essayer d'ajouter une 24LC256 EEPROm

MSOP

Address  0101 0000  0x50 (required in the specs)

1 A0 (not on MSOP)
2 A1 (not on MSOP)
3 A2 (vss?)
4 VSS
5 SDA
6 SCL
7 WP to VSS
8 VCC

## Activity Power Led

LED_nPWR : This pin needs to be buffered to drive an led. 

0: ON
1: OFF

Est-ce que R30 devrait être à +3.3V ?

Non le problème c'est que j'ai un SN74LVC2GU04 (inverseur) Et il me faut un non inverseur. Genre un SN74AUC2G07DBVT

## IMU ?

- ICM20948 9-axis
- LPS22HB  Barometric

Voir pour souder une https://www.digikey.ch/product-detail/en/xsens-technologies-bv/MTI-2-T/MTI-2-T-ND/5773874 ?

Ou alors un BN0085 https://fr.aliexpress.com/item/1005002618950945.html?spm=a2g0o.productlist.0.0.76334a44y46E3N&algo_pvid=5fa7109c-8c58-4e4a-92c0-7d3fc5f9bfce&algo_exp_id=5fa7109c-8c58-4e4a-92c0-7d3fc5f9bfce-3&pdp_ext_f=%7B%22sku_id%22%3A%2212000021423587829%22%7D

## Test Protocol

- [x] TPS566231
  - [x] +3.3V 
- [x] D5 3.3V
  - /!\ Not blue 
- [x] RPI
  - [x] Boot on SDCard
  - [ ] Activity led
  - [ ] Power led
- [x] GPIO I/Os 
  - [x] GPIO 13 (Utiliser /dev/, marche pas avec gpio)
- [x] FT232 console
- [x] Ethernet
  - [x] Connexion
- [x] USB2514
- [x] MCP9803
  - [x] On Bus
  - [x] Read
  - [x] Write
- [x] PAC1720 (0x56)
  - [x] Read voltage
  - [x] Read current 3.3V
  - [x] Read current 3.3VI2C
- [x] D3 GPIO24 -> Trop puissance augmenter résistance
- [x] D4 GPIO26 -> Trop puissante augmenter résistance
- [x] D5 LED 3.3V -> Pas bleue
- [x] LTC2990
  - [x] See I2C
  - [x] I2C connexion
  - [x] Read Status
  - [x] Write Status
  - [x] Read VBAT
  - [x] Read +3.3V
  - [x] Read +5V
  - [x] Read +3.3VPI
- [x] EMC2101 
  - [x] See I2C
  - [x] Read value
  - [x] Write value
  - [x] Read temperature
  - [x] Run FAN
  - [x] FAN connector
- [x] SI8602AB (Extern I2C3)
- [x] SI8602AB (Extern I2C1)
- [x] Audio TFA9879HN
  - [x] See on I2C
  - [x] PCM
- [x] DS1307 (0x68)
  - [x] On Bus
  - [x] Read
  - [x] Write
  - [x] Read time
- [x] DSI Display
- [x] NVMe / M2
  - [x] See PCI express
- [x] SLB9670 (TPM2.0)
  - [ ] Pourquoi initialisation ?
  - [ ] Pourquoi Kernel Panic
- [x] RPLidar
- [x] Tester avec une EEPROM
- [ ] USB 10pins connector
- [x] Camera1
- [ ] Camera0
- [ ] MCP2515 CAN
  - [ ] SPI
  - [ ] Read Value
  - [ ] Write Value
  - [ ] Communicate on CAN

## UARTs / Bluetooth

[Primary UART](https://www.raspberrypi.com/documentation/computers/configuration.html#primary-uart) says Primary UART is used as console

[Secondary UART](https://www.raspberrypi.com/documentation/computers/configuration.html#secondary-uart), not on the GPIOs is used for Bluetooth

The console is therefore on `/dev/ttyAMA0` (`/dev/serial0 -> ttyAMA0`)
The bluetooth is on `/dev/ttyS0` (`/dev/serial1 -> ttyS0`)

## Device Tree

i2c0_gpio0: i2c0_gpio0 {
	brcm,pins = <0 1>;
	brcm,function = <BCM2835_FSEL_ALT0>;
};
i2c1_gpio44: i2c1_gpio44 {
	brcm,pins = <44 45>;
	brcm,function = <BCM2835_FSEL_ALT2>;
};