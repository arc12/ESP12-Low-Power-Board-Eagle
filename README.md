# ESP12 Low Power Board
Repo for a PCB based around the ESP12 module intended to support low power use with deep sleep and 18650 cell. It is designed to allow multiple options, outlined below.

## Configuration
Solder jumpers are intended for settings which are constant for a given application of the board, e.g. an X sensor attached. Normal jumpers are for settings which might vary between locations or at different times (including setup and testing).

__SJ1__ (Deep Sleep) - Option A routes GPIO16 to the breakout for normal IO use; option B connects it to the reset pin for use of Deep Sleep.  
__SJ2,3__ - These jumpers are NC and may be cut if the serial adapter does not have both DTR and RTS signals on the relevant pins. If these signals are present, then flashing (and terminal reset) will be automatic. Otherwise S1 and S2 must be used. Note that if DTR is connected (and the Serial adapter is powered via either the battery connection or USB) then the “Cmd” (aka Flash) button cannot be used as an input. RTS may be connected and DTR left unconnected; in this case S1 must be used when boot-loading firmware, with only the reset function being automatic (pressing S2 not needed).  
__JP1-3__ (Cfg0/1/2) - May be used to select runtime configurations, for example to put the software into a testing or calibration loop. Omit these if the GPIOs will be used at the Breakout header. GPIO14 can either be programmed as Cfg2 or used for peripheral power control (p-Vcc, see below): pin pair “A” is for Cfg2 and “B” is for enabling the p-Vcc control.  
__JP4,5__ (Signal) - If shorted, LEDs 1 and 2 are connected to GPIO2 or GPIO0, respectively. This is intended for testing. May be omitted, along with the LEDs and resistors. GPIO2 shows activity during flashing and boot.  
__JP6__ (USB Power) - If shorted, allows for the USB-Serial breakout to power the device. NB: must be 3.3V regulated. In this case, the battery should be disconnected (although this is probably not critical). Normally open (or no header).  
__JP7__ (LDO Enable) - allows the LDO enable to be controlled off-board. Cut the PCB track jumper on the underside to use this.
__JP8__ (Diode Bypass) - allows the reverse supply protection schottky diode to be bypassed. Normally leave unpopulated.  
__JP9__ (Permanent Peripheral VCC) - See below.

## Runtime Switches
__S1__ (Flash/Cmd) -  Reset will trigger the boot loader if this switch is held on (i.e. flashing is done by holding S1 and then pressing/releasing S2). Once the new programme has been flashed, S2 (reset) must be done manually. After boot, this may be used as an ad hoc “command” (if DTR is not connected). This is active low.  
__S2__ (Reset) - ESP-12 reset.

## Headers
### Serial
UART pins and power, intended for flashing the device and diagnostic monitoring, although some applications might use them. When using a Bluetooth serial monitor, JP6 should be on, to supply it with power. Note that the pin labels are to match those on the serial device, rather than being those of the ESP8622.  See note on JP6. 3.3V logic must be used.

Note that, if the DTR and RTS signals are connected, then the PlatformIO monitor (or any other serial interaction which is not bootloading) must be configured as follows (add to platformio.ini), otherwise attempts to use the Monitor cause the board to hang (recovers if the monitor is killed):
```
monitor_rts = 0
monitor_dtr = 0
```

### Breakout
ESP-12 I/O and power for attached devices. The higher pin numbers may also be used for on board functions, in which case a shorter set of header pins can be used.

Some notes:
- GPIO 4 and 5 may be used without consideration, and are conventionally used for I2C.
- A0 may be used only if battery monitoring is NOT being used.
- GPIO 12 and 13 may be used at the breakout or one/both may be used for configuration setting.
- GPIO 14 may be used for onboard signalling for testing via JP4, for control of power to "peripherals" (see below), or used for I/O at the breakout header.
- GPIO 16 is used for sleep control in Deep Sleep (active low, drives RST). Avoid using at the breakout if deep sleep is used (this GPIO has other limitations too: see datasheet).
- RST can be used to wake the ESP8266 from a deep sleep.

### I2C
Convenience headers, ordered to match the SHT4X, BH1750, ... breakout boards. Note that one has the power pins reversed.

## Omittable Components
- LEDs and related resistors and jumpers.
- JP1-3 and headers.
- R10,11 should be omitted if I2C communications is not occurring on GPIO4,5. A value of 4k7 is shown but values between 10k and 2.2k are normal. Some breakout boards already include 10k pull-up resistors, which may be satisfactory; if so, omit R10,11, especially if there is more than one I2C breakout with pull-ups present.
- Q1, R12, JP8, JP3 (partial) - see next section.
- JP7, if PCB trace not cut.
- S1 is usually not needed.
- S3, S4 - see next section.
- EP2, EP3 - see next section.
- D1, JP8, JP7, C1, C2, U2 - if powering through E2, see next section.

The ESP-12 modules have a LED on GPIO2, which may be unwanted!

## Optional Extras

### External Power
The main use case is to use the on-board 18650 battery clip, with the clip wires soldered to EP1. Alternatively, an off-board battery may be connected using a screw terminal or solderless connector at EP1.

Alternatives are:  
- use EP2 to connect a 3.3V PSU, in which case the voltage regulation components through to U2 should be omitted.  
- use EP3, a USB-mini socket to supply 5V

### Reed Switches
Provision is made to place two reed switches on the underside so that sealed enclosures can be crudely controlled from the outside using magnets with N-S pole alignments parallel to the reed switch axes. One switch (S3) is parallel with Cfg1 and another (S4) parallel to the Reset switch. A typical use is to use Cfg1 to set the device into OTA update & web server mode, then resetting to wake from sleep.

### Peripheral/sensor Power Rail Control Option
__AKA p-Vcc__
This option allows the ESP8266 to disconnect power to the peripherals on SV2-5. NB: SV1 is not affected since power may be supplied TO the board via that header.

The default case is Vcc to peripherals being “permanent”; Q1 + R12 should be omitted (and JP3 reduced to 2 pins). If p-Vcc is required then cut the PCB track on the under-side of JP9. The normal operating condition will have JP9 open and JP3 set to connect GPIO14 to Q1 (option B on PCB silk). In this case, Cfg2 cannot be used. The MCU will need to restore power by pulling GPIO14 low before attempting communications. JP9 maybe closed to override the control to “always on”.

JP3 may be soldered as a 2 pin or 3 pin configuration according to need, for example to allow for the situation when R12 and Q1 are in place but no longer used; i.e. it allows GPIO14 to be used without R12 acting as a pull-up. In this case, JP9 would be closed.

## Future Variant Ideas
To allow for 5V devices (even if 3.3V logic): a double LiPo battery with both 5V and 3.3V step-down and mosfet power rail control.

## Device Construction Notes
_To match software, elsewhere in a currently-private repo._
### TH
Options:
- Remove LED from ESP-12
- Battery and LDO circuit as normal, omit JP8 and the PSU connector. (track under JP7 intact)
- Omit p-Vcc components (track under JP9 intact)
- LED2 as amber, 180R resistor, with jumper. (omit LED1 and R9)
- Cfg0 and Cfg1 jumpers (neither connected for normal use.
- SJ3 to deep sleep.
- Include serial header and external power jumper, but no others; solder SHT40 direct to one I2C header slot.
- Reset switch but not "flash/cmd" (omit R7).