# Xilinx Virtual Cable Server for Raspberry Pi 2/3

[Xilinx Virtual Cable](https://github.com/Xilinx/XilinxVirtualCable/) (XVC) is a TCP/IP-based protocol that  acts like a JTAG cable and provides a means to access and debug your  FPGA or SoC design without using a physical cable.
A full description of Xilinx Virtual Cable in action is provided in the [XAPP1252 application note](https://www.xilinx.com/support/documentation/application_notes/xapp1251-xvc-zynq-petalinux.pdf).

**Xvcpi** implements an XVC server to allow a Xilinx FPGA or SOC to be controlled remotely by Xilinx Vivado using the Xilinx Virtual Cable protocol. Xvcpi uses TCP port 2542.

The **xvcpi** server runs on a Raspberry Pi which is connected, using JTAG, to the target device. **Xvcpi** bitbangs the JTAG control signals on the Pi pins. The bitbanging code was originally extracted from [OpenOCD](http://openocd.org)[bcm2835gpio.c](https://github.com/arduino/OpenOCD/blob/master/src/jtag/drivers/bcm2835gpio.c).

# Wiring
Note: The Raspberry Pi is a 3.3V device. Ensure that the target device and the Pi are electrically compatible before connecting. 100 Ohm resistors may be placed inline on all of the JTAG signals to provide a degree of electrical isolation.


JTAG uses 4 signals, TMS, TDI, TDO and, TCK.
From the Raspberry Pi perspective, TMS, TDI and TCK are outputs, and TDO is an input.

¡¡¡OLD PIN MAPPING!!!!
The pin mappings for the Raspberry Pi header are:
```
#BMC: TMS=25, TDI=10, TCK=11, TDO=9
#PIN: TMS=22, TDI=19, TCK=23, TDO=21
```
In addition a ground connection is required. Pin 23 is a conveniently placed GND.



¡¡¡NEW PIN MAPPING!!!! (like https://github.com/avx/xvcd-rpi)
```
JTAG      BCM      PIN no
====================================
TDI  <-> GPIO17 ==   11
TCK  <-> GPIO27 ==   13
TMS  <-> GPIO22 ==   15
TDO  <-> GPIO23 ==   16

(see rpi-gpio-pinout.png for pinout)

# gpio readall

 +-----+-----+---------+------+---+---Pi 2---+---+------+---------+-----+-----+
 | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
 +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
 |     |     |    3.3v |      |   |  1 || 2  |   |      | 5v      |     |     |
 |   2 |   8 |   SDA.1 | ALT0 | 1 |  3 || 4  |   |      | 5V      |     |     |
 |   3 |   9 |   SCL.1 | ALT0 | 1 |  5 || 6  |   |      | 0v      |     |     |
 |   4 |   7 | GPIO. 7 |   IN | 1 |  7 || 8  | 1 | ALT0 | TxD     | 15  | 14  |
 |     |     |      0v |      |   |  9 || 10 | 1 | ALT0 | RxD     | 16  | 15  |
 |  17 |   0 | GPIO. 0 |   IN | 0 | 11 || 12 | 0 | IN   | GPIO. 1 | 1   | 18  |
 |  27 |   2 | GPIO. 2 |   IN | 0 | 13 || 14 |   |      | 0v      |     |     |
 |  22 |   3 | GPIO. 3 |   IN | 0 | 15 || 16 | 0 | IN   | GPIO. 4 | 4   | 23  |
 |     |     |    3.3v |      |   | 17 || 18 | 0 | IN   | GPIO. 5 | 5   | 24  |
 |  10 |  12 |    MOSI | ALT0 | 0 | 19 || 20 |   |      | 0v      |     |     |
 |   9 |  13 |    MISO | ALT0 | 0 | 21 || 22 | 0 | IN   | GPIO. 6 | 6   | 25  |
 |  11 |  14 |    SCLK | ALT0 | 0 | 23 || 24 | 1 | OUT  | CE0     | 10  | 8   |
 |     |     |      0v |      |   | 25 || 26 | 1 | OUT  | CE1     | 11  | 7   |
 |   0 |  30 |   SDA.0 |   IN | 1 | 27 || 28 | 1 | IN   | SCL.0   | 31  | 1   |
 |   5 |  21 | GPIO.21 |   IN | 1 | 29 || 30 |   |      | 0v      |     |     |
 |   6 |  22 | GPIO.22 |   IN | 1 | 31 || 32 | 0 | IN   | GPIO.26 | 26  | 12  |
 |  13 |  23 | GPIO.23 |   IN | 0 | 33 || 34 |   |      | 0v      |     |     |
 |  19 |  24 | GPIO.24 |   IN | 0 | 35 || 36 | 0 | IN   | GPIO.27 | 27  | 16  |
 |  26 |  25 | GPIO.25 |   IN | 0 | 37 || 38 | 0 | IN   | GPIO.28 | 28  | 20  |
 |     |     |      0v |      |   | 39 || 40 | 0 | IN   | GPIO.29 | 29  | 21  |
 +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
 | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
 +-----+-----+---------+------+---+---Pi 2---+---+------+---------+-----+-----+
```

#Raspberry pi 3
[BCM2837](https://www.raspberrypi.org/documentation/hardware/raspberrypi/bcm2837/README.md): This is the Broadcom chip used in the Raspberry Pi 3, and in later models of the Raspberry Pi 2. The underlying architecture of the BCM2837 is identical to the BCM2836. The only significant difference is the replacement of the ARMv7 quad core cluster with a quad-core ARM Cortex A53 (ARMv8) cluster.


Note that XVC does not provide control of either SRST or TRST and **xvcpi** does not support a RST signal.

# Usage
Start **xvcpi** on the Raspberry Pi. An optional -v flag can be used for verbose output.

Vivado connects to **xvcpi** via an intermediate software server called hw_server. To allow Vivado "autodiscovery" of **xvcpi** via hw_server run:

```
hw_server -e 'set auto-open-servers xilinx-xvc:<xvcpi-server>:2542'
```

Alternatively, the following tcl commands can be used in the Vivado Tcl console to initiate a connection.

```
connect_hw_server
open_hw_target -xvc_url <xvcpi-server>:2542
```

Full instructions can be found in [ProdDoc_XVC_2014 3](ProdDoc_XVC_2014_3.pdf).

# Snickerdoodle
The initial purpose of **xvcpi** was to provide a simple means of programming the [Snickerdoodle](http://snickerdoodle.io).
# Licensing
This work, "xvcpi.c", is a derivative of "xvcServer.c" (https://github.com/Xilinx/XilinxVirtualCable)

"xvcServer.c" is licensed under CC0 1.0 Universal (http://creativecommons.org/publicdomain/zero/1.0/)
by Avnet and is used by Xilinx for XAPP1251.

"xvcServer.c", is a derivative of "xvcd.c" (https://github.com/tmbinc/xvcd)
by tmbinc, used under CC0 1.0 Universal (http://creativecommons.org/publicdomain/zero/1.0/).

Portions of "xvcpi.c" are derived from OpenOCD (http://openocd.org), (https://github.com/arduino/OpenOCD/blob/master/src/jtag/drivers/bcm2835gpio.c)

Documentation GPIO:

[Low Level Programming of the Raspberry Pi in C](http://www.pieter-jan.com/node/15) From there: "Physical addresses range from 0x20000000 to 0x20FFFFFF for peripherals. The bus addresses for peripherals are set up to map onto the peripheral bus address range starting at 0x7E000000. Thus a peripheral advertised here at bus address 0x7Ennnnnn is available at physical address 0x20nnnnnn." And explication for macros. Complete.

[BCM2835 ARM Peripherals](https://www.raspberrypi.org/app/uploads/2012/02/BCM2835-ARM-Peripherals.pdf)

[GPIO pads control](https://es.scribd.com/doc/101830961/GPIO-Pads-Control2)

[The comprehensive GPIO Pinout guide for the Raspberry Pi.](https://pinout.xyz/)

[BCM2835](https://www.raspberrypi.org/documentation/hardware/raspberrypi/bcm2835/README.md)

[BCM2837](https://www.raspberrypi.org/documentation/hardware/raspberrypi/bcm2837/README.md)

https://github.com/raspberrypi/linux/blob/204050d0eafb565b68abf512710036c10ef1bd23/Documentation/devicetree/bindings/arm/bcm/brcm%2Cbcm2835.txt 
```
Raspberry Pi 3 Model B: 
compatible = "raspberrypi,3-model-b", "brcm,bcm2837";
```

https://github.com/raspberrypi/linux/blob/204050d0eafb565b68abf512710036c10ef1bd23/arch/arm/boot/dts/bcm2710.dtsi
```
compatible = "brcm,bcm2837", "brcm,bcm2836";
	model = "BCM2837";

	soc {
		ranges = <0x7e000000 0x3f000000 0x01000000>,
		         <0x40000000 0x40000000 0x00040000>;
		dma-ranges = <0xc0000000 0x00000000 0x3f000000>;
```
vs https://github.com/raspberrypi/linux/blob/204050d0eafb565b68abf512710036c10ef1bd23/arch/arm/boot/dts/bcm2835.dtsi

https://sourceforge.net/p/raspberry-gpio-python/code/ci/default/tree/source/c_gpio.c#l105
```
#define BCM2708_PERI_BASE_DEFAULT   0x20000000
#define BCM2709_PERI_BASE_DEFAULT   0x3f000000
...
...
else if (strcmp(hardware, "BCM2709") == 0 || strcmp(hardware, "BCM2836") == 0) {
                // pi 2 hardware
                peri_base = BCM2709_PERI_BASE_DEFAULT;
                found = 1;

```

"xvcpi.c" is licensed under CC0 1.0 Universal (http://creativecommons.org/publicdomain/zero/1.0/)
by Derek Mulcahy.