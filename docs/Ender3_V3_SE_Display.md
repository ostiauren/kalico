# Creality Ender 3 V3 SE display

This adds support for the original display of the Creality Ender 3 V3 SE
(a 3.2 inch TJC3224 display with a rotary encoder) using the printer's
existing display cable, without any hardware modification. The MCU acts as a
serial bridge between Klippy and the display's UART, and a UI running inside
Klippy draws the menus.

The code comes from
[jpcurti/ender3-v3-se-klipper-with-display](https://github.com/jpcurti/ender3-v3-se-klipper-with-display)
(which builds on E4ST2W3ST's serial bridge and the
[DWIN_T5UIC1_LCD](https://github.com/odwdinc/DWIN_T5UIC1_LCD) project). Only the
display support was taken over; the strain gauge and Z drive changes of that
repository are not included. The only change needed for Kalico is that
`serial_bridge.py` uses `mcu.register_response()`.

## Requirements

- An Ender 3 V3 SE with the STM32F401 mainboard (hardware version
  `CR4NS200320C14`). The original STM32F103 board uses different build settings
  (28KiB bootloader offset).
- The display must run Creality's **display firmware 1.0.6**. The UI refers to
  icons and images stored in the display's own flash, and other versions
  change their locations. Details are in the
  [Creality downloads](https://www.creality.com/pages/download-ender-3-v3-se).

## Building and flashing the MCU firmware

The firmware needs the serial bridge enabled on USART2. This is available under
"Low-level options" (`make menuconfig`) as "Enable serial bridge". A complete
configuration is in `test/configs/e3v3se_display.config`. Use these settings
for the Ender 3 V3 SE mainboard:

- Micro-controller architecture: STMicroelectronics STM32
- Processor model: STM32F401
- Bootloader offset: 64KiB bootloader
- Communication interface: Serial (on USART1 PA10/PA9)
- Enable low-level options and the serial bridge: USART2

Build with `make`, then flash as usual for this board: put `klipper.bin` in a
folder named `STM32F4_UPDATE` on a FAT32 SD card and power the printer on. If
the printer does not flash it, give the file a name different from the one used
for the previous flash.

## Configuration

```
[e3v3se_display]
#language: english
#  The display language. The default is english.
#logging: False
#  Log the data sent to the display. The default is False.
#encoder_pins: ^PA12, ^PA11
#encoder_steps_per_detent: 4
#encoder_fast_rate: 0.030
#click_pin: ^!PB1
#  The rotary encoder and its button. The defaults are the pins of the
#  Ender 3 V3 SE mainboard.
#tx_pin: PA2
#rx_pin: PA3
#baud: 115200
#config: 3
#  The UART of the display. The defaults are the Ender 3 V3 SE display
#  connector (USART2). The bridge configurations available in the firmware
#  can be listed with the SERIAL_BRIDGE_LIST_CONFIGS command.
```

The display module needs the sections `[extruder]`, `[heater_bed]`, `[fan]`,
`[virtual_sdcard]` and `[stepper_z]`. When the printer has a `[bltouch]`
section, its `z_offset` is shown as the Z offset.

### Custom macros

The "Misc" menu lists macros defined in the printer configuration:

```
[e3v3se_display MACRO1]
gcode: LOAD_FILAMENT
label: Load filament
icon: 14
```

`gcode` and `label` are required. `icon` is the number of one of the display's
internal icons (default 14). Run `ENDER_SE_DISPLAY_ICON_FINDER` to browse the
icons on the display.

## Supported functions

Print a file, tune, pause, continue and stop a print, move and home the axes,
set the Z offset, preheat, cool down, set the nozzle and bed temperature, disable
the stepper motors, the manual probe pop-up and the custom macros menu. Setting
the maximum speed, acceleration and steps per mm is not supported and shows a
pop-up.

## Related commands

- `SERIAL_BRIDGE_SEND BRIDGE=<name> TEXT=<text>`: send text (with `\xNN` escapes)
  to a serial bridge.
- `SERIAL_BRIDGE_LIST_CONFIGS`: list the serial bridge configurations that the
  MCU firmware supports.
- `SERIAL_BRIDGE_LIST_BRIDGES`: list the configured bridges.
