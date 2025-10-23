# Atari ST RP2040/RP2350 IKBD Emulator

This project allows you to use a RP2040 or RP2350 microcontroller (Raspberry Pi Pico or Pico 2) to emulate the HD6301 controller that is used as the intelligent keyboard controller for the Atari ST/STe/TT series of computers. This is useful if for example you have a Mega ST that is missing its keyboard. The emulator provides the ability to use a USB keyboard, mouse and joysticks with the ST.

**Latest Version: 7.5.0** - Now with Nintendo Switch Pro Controller support!

This project supports both the Raspberry Pi Pico (RP2040) and Raspberry Pi Pico 2 (RP2350). You can select which board to build for using CMake options. The firmware should work on any RP2040/RP2350 based board that includes a USB host capable connector and enough I/O for the external connections.

The emulator displays a simple user interface on an OLED display. This is entirely optional and you can build a working version without it but
it is certainly useful to show successful connection of your USB devices as well as to allow the mouse speed to be tweaked and to view the data
flowing between the emulator and the Atari ST.

The interface is now available in English, French, German, Spanish and Italian.

![English](mouse_EN.jpg) &emsp; ![French](mouse_FR.jpg) &emsp; ![German](mouse_DE.jpg)

![Spanish](mouse_SP.jpg) &emsp; ![Italian](mouse_IT.jpg)

The emulator supports both USB and Atari ST compatible joysticks, supported a maximum of two joysticks at a time. Using the user interface
you can select whether the USB joystick or Atari joystick are assigned to Joysticks 0 and 1.

## How it works
The Atari ST keyboard contains an HD6301 microcontroller that can be programmed by the Atari TOS or by user applications to read the keyboard, mouse and joysticks. The keyboard is connected to the Atari via a serial interface. Commands can be sent from the Atari to the keyboard and the keyboard sends mouse movements, keystrokes and joystick states to the Atari.

Instead of writing code to handle the serial protocol between the Atari and the keyboard, this project provides a full emulation of the HD6301 microcontroller and the hardware connected to it. This means that it appears to the Atari as a real keyboard, and can be customised and programmed by software like a real keyboard, providing maximum compatibility.

The RP2040 USB host port is used to connect a keyboard, mouse and joysticks using a USB hub. These are translated into an emulation of the relevant device and fed into the emulated HD6301 control registers, allowing the HD6301 to determine how to communicate this with the Atari.

## Building the emulator
The emulator is configured as per the schematic below.

![Schematic](schematic.png)

All of the external components except the level shifter are optional - you do not need to include the display and buttons if you are happy to hardcode mouse acceleration and joystick assignment settings in code. Also, if you only plan to use USB joysticks then you can omit the DB-9 connectors.

The level shifter is required as the Atari uses 5V logic over the serial connection whereas the Pico uses 3.3V logic. You can possible get away with leaving UART_RX disconnected and connect UART_TX to the Atari without a level shifter but many games and applications will not work like this as they send commands to the IKBD/emulator.

## Compiling the Emulator Firmware

To compile the firmware you will need to checkout this repository, sync the included submodules and make a small change to the pico-sdk CMakelist.txt.

### Quick Build (Both Pico and Pico 2)

```bash
./build-all.sh
# Outputs: dist/atari_ikbd_pico.uf2 and dist/atari_ikbd_pico2.uf2
```

### Mac (ARM)  

Compiling on the mac requires xcode, gcc and arm embedded toolchain. A build can be performed with the following commands:

```
# Install GCC components from homebrew
brew install gcc armmbed/formulae/arm-none-eabi-gcc

# Clone the main repo
git clone -b main  https://github.com/trickydee/atari-st-rpikb
cd atari-st-rpikb

# Sync submodules
git submodule sync
git submodule update --init --recursive

# Build for Raspberry Pi Pico (RP2040) - default
cmake -B build -S . -DPICO_BOARD=pico && cd build && make

# OR build for Raspberry Pi Pico 2 (RP2350)
cmake -B build -S . -DPICO_BOARD=pico2 && cd build && make
```

PC (Linux)
```
#Install GCC
sudo apt install gcc-arm-none-eabi

#Install cmake
sudo apt install cmake

#Clone the main repo in your home folder
git clone --recursive https://github.com/klyde2278/atari-st-rpikb.git

#Update submodules
cd atari-st-rpkib
git submodule update --init --recursive

#fix missing hidparser include in cmakelists.txt
Open file: atari-st-rpikb/pico-sdk/src/rp2_common/tinyusb/CMakeLists.txt
If not present, add this line after line 91:  ${PICO_TINYUSB_PATH}/src/class/hid/hidparser/HIDParser.c
You should have:

target_sources(tinyusb_host INTERFACE
            ${PICO_TINYUSB_PATH}/src/portable/raspberrypi/rp2040/hcd_rp2040.c
            ${PICO_TINYUSB_PATH}/src/portable/raspberrypi/rp2040/rp2040_usb.c
            ${PICO_TINYUSB_PATH}/src/host/usbh.c
            ${PICO_TINYUSB_PATH}/src/host/usbh_control.c
            ${PICO_TINYUSB_PATH}/src/host/hub.c
            ${PICO_TINYUSB_PATH}/src/class/cdc/cdc_host.c
            ${PICO_TINYUSB_PATH}/src/class/hid/hid_host.c
            ${PICO_TINYUSB_PATH}/src/class/hid/hidparser/HIDParser.c
            ${PICO_TINYUSB_PATH}/src/class/msc/msc_host.c
            ${PICO_TINYUSB_PATH}/src/class/vendor/vendor_host.c
            )
From your atari-st-rpikb folder:
mkdir build
cd build

# Choose one of the cmake command below according to the language and version number you want to display:
cmake -DLANGUAGE=EN .. # For English interface
or
cmake -DLANGUAGE=FR ..  # For French interface
or
cmake -DLANGUAGE=DE ..  # For German interface
or
cmake -DLANGUAGE=SP ..  # For Spanish interface
or
cmake -DLANGUAGE=IT ..  # For Italian interface

make
```
## Downloading the firmware
If you don't know how or can't build the firmware by yourself, please find the released files here: https://github.com/klyde2278/atari-st-rpikb/releases

## Using the emulator
If you build the emulator as per the schematic, the Pico is powered directly from the Atari 5V supply. The Pico boots immediately but USB enumeration can take a few seconds. Once this is complete, the emulator is fully operational.

The user interface has 4 pages that are rotated between by pressing the middle UI button. The first three pages all show the number of connected USB devices at the top but allow configuration of an option below. The pages in order are:

1. USB Status + Mouse speed. Left and right buttons change allow the mouse speed to be altered.
   
   ![Mouse speed](mouse_EN.jpg)

2. USB Status + Joystick 0 assignment. Left and right buttons toggle between USB joystick and DB-9 joystick.
   
   ![Joystick 0](joy2_usb.jpg) &emsp; ![Joystick 0](joy0_dsub.jpg)

3. USB Status + Joystick 1 assignment. Left and right buttons toggle between USB joystick and DB-9 joystick.
   
   ![Joystick 1](joy1_usb.jpg) &emsp; ![Joystick 0](joy1_dsub.jpg)

4. Serial data Tx/Rx between emulator and Atari. Data received from the Atari is on the left, data sent to the Atari is on the right.
   
   ![Comms](comms.jpg)

The serial data page should only be used for ensuring the connection works. Displaying the page slows down the emulator and you may seem some mouse lag whilst it is active.

The real ST keyboard has a single DB-9 socket which is shared between the mouse and Joystick 0. The emulator allows you to have a mouse and joystick plugged in simultaneously but you need to select whether the mouse or joystick 0 is active. This can be toggled by pressing the Scroll Lock button on the keyboard. The current mode is shown on any of the status pages on the OLED display.

## Keyboard Shortcuts

The emulator supports several keyboard shortcuts for convenient control:

| Shortcut | Function | Description |
|----------|----------|-------------|
| **Ctrl+F12** | Toggle Mouse Mode | Switches between USB mouse and joystick 0 |
| **Ctrl+F11** | XRESET | Triggers HD6301 hardware reset (like power cycling the IKBD) |
| **Ctrl+F10** | Toggle Joystick 1 | Switches Joystick 1 between D-SUB and USB |
| **Ctrl+F9** | Toggle Joystick 0 | Switches Joystick 0 between D-SUB and USB |
| **Alt+/** | INSERT Key | Sends Atari ST INSERT key (useful for modern keyboards) |
| **Alt+[** | Keypad /** | Sends Atari ST keypad divide key - Default HAGA Save key |
| **Alt+]** | Keypad *** | Sends Atari ST keypad multiply key - Default HAGA Quit key |
| **Alt+Plus** | Set 270MHz | Overclocks RP2040 to 270MHz for maximum performance |
| **Alt+Minus** | Set 150MHz | Sets RP2040 to 150MHz for stability |

For detailed information about keyboard shortcuts, see [KEYBOARD_SHORTCUTS.md](KEYBOARD_SHORTCUTS.md).

## USB Controller Support

The emulator supports multiple types of USB game controllers with reliable hot-swapping:

- **Xbox Controllers**: Xbox 360 (wired/wireless), Xbox One, Xbox Series X|S, Original Xbox
- **PS4 DualShock 4**: Full support via USB (wired)
- **Nintendo Switch Pro Controller**: Full support with analog stick precision and no drift (v7.5.0)
- **Generic HID Joysticks**: Standard USB joysticks and gamepads

**v7.5.0:** Added Nintendo Switch Pro Controller support with BetterJoy-style USB initialization, 12-bit analog stick precision, and automatic deadzone compensation.

**v7.3.0:** Xbox and PS4 controllers can now be swapped freely without issues! Previously Xbox wouldn't work after PS4 usage - this is now fixed.

Controllers use D-Pad and left analog stick for movement, with buttons (A/B/X/Y/triggers) mapped to fire button. All controllers support hot-swapping between different types.

## Known limitations
The RP2040 USB host implementation seems to contain a number of bugs. This repository contains a patched branch of the TinyUSB code to workaround many of these issues, however there are still some limitations and occasional issues as summarised below:

* Occasionally on startup USB devices are not enumerated. You need to restart the emulator to try again.
* The emulator supports only a single USB hub. So, if you use a keyboard that has a hub built in then that must be connected directly to the Pico and not plugged into another hub.

## Acknowledgements
This project has been pieced together from code extracted from [Steem SSE](https://sourceforge.net/projects/steemsse/). All of the work of wiring up the keyboard functions to the HD6301 CPU is credited to Steem SSE. This project contains a stripped-down version of this interface, connecting it to the Raspberry Pi's serial port.

Steem itself uses the HD6301 emulator provided by sim68xx developed by Arne Riiber. The original website for this seems to have gone but an archive can be found [here](http://www.oocities.org/thetropics/harbor/8707/simulator/sim68xx/).

The code to handle the OLED display is Copyright (c) 2021 David Schramm and taken from https://github.com/daschr/pico-ssd1306.

## Credits
Special thanks to the following projects for inspiration and reference implementations:

- **[tusb_xinput](https://github.com/Ryzee119/tusb_xinput)** by Ryzee119 - TinyUSB XInput host driver used for Xbox controller support
- **[BetterJoy](https://github.com/Davidobot/BetterJoy)** by Davidobot - Reference implementation for Nintendo Switch Pro Controller USB initialization and protocol handling
