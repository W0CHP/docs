# Compile VR2VYE / BI7JTA MMDVM Repeater Board Firmware on W0CHP's Fork of MW0MWZ’s Pi-Star

The Steps i did to get M17 with the MMDVM Repeater Board Pi-Star running. If you find mistakes, please let me know!

### These are just my personal notes, do this at your own risk! 
### No support from me! 
### Backup everything before you do anything!

## Links
- https://m17project.org/
- https://w0chp.net/w0chp-pistar-dash/
- https://www.bi7jta.org/shop/mmdvm-repeater-board-v3f4-dmr-ysf-d-star-nxdn-pocsag-fm-3#attr=63,27,117

1.) Install the Pi-Star Fork

```bash
# Enable Read / Write Mode
rpi-rw
```

2.)  Grow your root Partition if needed:
```bash
# Check which partition is your root (/) partition
# In my case it was /dev/sda2
mount

# Install cloud-utils for growpart
sudo apt-get install cloud-utils

# Grow root partiion
sudo growpart /dev/sda 2

# Resize filesystem
sudo resize2fs /dev/sda2
```

3.) Compile Firmware:
```bash
# Install dependencies. Some of them are probably not needed.
sudo apt-get install git gcc-arm-none-eabi gcc-arm-none-eabi gdb-arm-none-eabi libstdc++-arm-none-eabi-newlib autoconf libtool pkg-config libusb-1.0-0 libusb-1.0-0-dev

# Download the precompiled flashing tool
curl -OL https://www.bi7jta.org/files/fm-patch/firmware/stm32flashV5

# Clone the repo MMDVM and it's Subrepos
git clone https://github.com/g4klx/MMDVM.git --recurse-submodules
cd MMDVM/

# Configure it (Example which i use ATM below)
nano Config.h

# Clean (just in case) and build
make clean
make dvm
cd ..

# Kill the MMDVMHost process
sudo killall MMDVMHost >/dev/null 2>&1

# Flash the new Firmware
sudo ./stm32flashV5 -v -w ./MMDVM/bin/mmdvm_f4.hex  -R  -i 20,-21,21:-20,-21,21 /dev/ttyAMA0
```

## Content of my Config.h:
```c
/*
 *   Copyright (C) 2015,2016,2017,2018,2020 by Jonathan Naylor G4KLX
 *
 *   This program is free software; you can redistribute it and/or modify
 *   it under the terms of the GNU General Public License as published by
 *   the Free Software Foundation; either version 2 of the License, or
 *   (at your option) any later version.
 *
 *   This program is distributed in the hope that it will be useful,
 *   but WITHOUT ANY WARRANTY; without even the implied warranty of
 *   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 *   GNU General Public License for more details.
 *
 *   You should have received a copy of the GNU General Public License
 *   along with this program; if not, write to the Free Software
 *   Foundation, Inc., 675 Mass Ave, Cambridge, MA 02139, USA.
 */

#if !defined(CONFIG_H)
#define  CONFIG_H

// Allow for the selection of which modes to compile into the firmware. This is particularly useful for processors
// which have limited code space and processing power like the STM32F103, which is found on older/cheaper boards.

// Enable D-Star support.
#define MODE_DSTAR

// Enable DMR support.
#define MODE_DMR

// Enable System Fusion support.
#define MODE_YSF

// Enable P25 phase 1 support.
#define MODE_P25

// Enable NXDN support, the boxcar filter sometimes improves the performance of NXDN receive on some systems.
#define MODE_NXDN
#define USE_NXDN_BOXCAR

// Enable M17 support.
#define MODE_M17

// Enable POCSAG support.
#define MODE_POCSAG

// Enable FM support.
#define MODE_FM

// Enable AX.25 support, this is only enabled if FM is also enabled.
#define MODE_AX25

// Allow for the use of high quality external clock oscillators
// The number is the frequency of the oscillator in Hertz.
//
// The frequency of the TCXO must be an integer multiple of 48000.
// Frequencies such as 12.0 Mhz (48000 * 250) and 14.4 Mhz (48000 * 300) are suitable.
// Frequencies such as 10.0 Mhz (48000 * 208.333) or 20 Mhz (48000 * 416.666) are not suitable.
//
// For 12 MHz
#define EXTERNAL_OSC 12000000
// For 12.288 MHz
// #define EXTERNAL_OSC 12288000
// For 14.4 MHz
// #define EXTERNAL_OSC 14400000
// For 19.2 MHz
// #define EXTERNAL_OSC 19200000

// Select a baud rate for host communication. The faster speeds are needed for external FM to work.
#define SERIAL_SPEED 115200 // Suitable for most older boards (Arduino Due, STM32F1_POG, etc). External FM will NOT work with this!
// #define SERIAL_SPEED 230400 // Only works on newer boards like fast M4, M7, Teensy 3.x. External FM might work with this
// #define SERIAL_SPEED 460800	// Only works on newer boards like fast M4, M7, Teensy 3.x. External FM should work with this
//#define SERIAL_SPEED 500000  // Used with newer boards and Armbian on AllWinner SOCs (H2, H3) that do not support 460800

// Use pins to output the current mode via LEDs
#define MODE_LEDS

// For the original Arduino Due pin layout
// #define ARDUINO_DUE_PAPA

#if defined(STM32F1)
// For the SQ6POG board
//#define STM32F1_POG
#else
// For the ZUM V1.0 and V1.0.1 boards pin layout
// #define ARDUINO_DUE_ZUM_V10
#endif

// For the SP8NTH board
// #define ARDUINO_DUE_NTH

// For ST Nucleo-64 STM32F446RE board
#define STM32F4_NUCLEO_MORPHO_HEADER
// #define STM32F4_NUCLEO_ARDUINO_HEADER

// Use separate mode pins to switch external channel/filters/bandwidth for example
// #define MODE_PINS

// For the VK6MST Pi3 Shield communicating over i2c. i2c address & speed defined in i2cTeensy.cpp
// #define VK6MST_TEENSY_PI3_SHIELD_I2C

// Pass RSSI information to the host
#define SEND_RSSI_DATA

// Use the modem as a serial repeater for Nextion displays
#define SERIAL_REPEATER

// Use the modem as an I2C repeater for OLED displays
// #define I2C_REPEATER

// To reduce CPU load, you can remove the DC blocker by commenting out the next line
#define USE_DCBLOCKER

// Constant Service LED once repeater is running
// Do not use if employing an external hardware watchdog
// #define CONSTANT_SRV_LED

// Use the YSF and P25 LEDs for NXDN
// #define USE_ALTERNATE_NXDN_LEDS

// Use the D-Star and P25 LEDs for M17
#define USE_ALTERNATE_M17_LEDS

// Use the D-Star and DMR LEDs for POCSAG
#define USE_ALTERNATE_POCSAG_LEDS

// Use the D-Star and YSF LEDs for FM
#define USE_ALTERNATE_FM_LEDS

#if defined(STM32F1_POG)
// Slower boards need to run their serial at 115200 baud
#undef SERIAL_SPEED
#define SERIAL_SPEED 115200
#endif

#endif

```
