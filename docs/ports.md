# I/O port list

This document describes the I/O ports available on the system.

## List of ports by category

For port that need it, a sensible initialization value is provided if necessary. The values are mostly taken from the existing firmware code, with some differences where it seems appropriate.

### Screen

| Port | Init | Description |
| ---- | ---- | ----------- |
| [0x00](#port-0x00-screen-memory-location) | 0x00 | Screen buffer position |

### Keyboard scan

| Port | Init | Description |
| ---- | ---- | ----------- |
| [0x01](#port-0x01-keyboard-scan) | 0x00 | Keybard matrix scan |
| [0x02](#port-0x02-gpio-group-1) | | Keyboard rows 8-9 |

### GPIO pins

| Port | Init | Description |
| ---- | ---- | ----------- |
| [0x02](#port-0x02-gpio-group-1) | 0xA4 | GPIO 1: keyboard, LCD, LED, modem power, caller ID input |
| [0x0B](#port-0x0b-gpio-group-1-direction) | 0xFF | GPIO 1 pin direction |
| [0x09](#port-0x09-gpio-group-2) | 0x0F | GPIO 2: printer control, voltage info, power button |
| [0x0A](#port-0x0a-gpio-group-2-direction) | 0x0F | GPIO 2 pin direction |
| [0x21](#port-0x21-gpio-group-3) | | GPIO 3: printer status |
| [0x28](#port-0x28-gpio-group-4) | 0x00 | GPIO 4: power off, modem reset, caller ID control |
| [0x29](#port-0x29-gpio-group-4-direction) | 0x03 | GPIO 4 pin direction |
| [0x2D](#port-0x2d-gpio-group-5) | 0x00 | GPIO 5: printer data lines |
| [0x2C](#port-0x2c-gpio-group-5-direction) | 0xFF | GPIO 5 pin direction |
| [0x26](#port-0x26-modem-reset) | 0x00 | Modem reset pin |

### Interrupts

| Port | Init | Description |
| ---- | ---- | ----------- |
| [0x03](#port-0x03-interrupt-mask) | 0x22 | Interrupt mask / source |

### Memory management

| Port | Init | Description |
| ---- | ---- | ----------- |
| [0x06](#port-0x06-0x4000-device-selection) | | 0x4000 device selection |
| [0x05](#port-0x05-0x4000-device-page) | | 0x4000 device page |
| [0x08](#port-0x08-0x8000-device-selection) | | 0x8000 device selection |
| [0x07](#port-0x07-0x8000-device-page) | | 0x8000 device page |

### Real time clock

| Port | Init | Description |
| ---- | ---- | ----------- |
| [0x10](#port-0x10-rtc-seconds-low) | | Seconds low nibble |
| [0x11](#port-0x11-rtc-seconds-high) | | Seconds high nibble |
| [0x12](#port-0x12-rtc-minutes-low) | | Minutes low nibble |
| [0x13](#port-0x13-rtc-minutes-high) | | Minutes high nibble |
| [0x14](#port-0x14-rtc-hour-low) | | Hours low nibble |
| [0x15](#port-0x15-rtc-hour-high) | | Hours high nibble |
| [0x16](#port-0x16-rtc-day-of-the-week) | | Day of the week |
| [0x17](#port-0x17-rtc-day-low) | | Day low nibble |
| [0x18](#port-0x18-rtc-day-high) | | Day high nibble |
| [0x19](#port-0x19-rtc-month-low) | | Month low nibble |
| [0x1A](#port-0x1a-rtc-month-high) | | Month high nibble |
| [0x1B](#port-0x1b-rtc-year-low) | | Year low nibble |
| [0x1C](#port-0x1c-rtc-year-high) | | Year high nibble |
| [0x1D](#port-0x1d-rtc-page-register) | 0x04 | Page register |
| [0x1E](#port-0x1e-rtc-test-register) | 0x00 | Test register |
| [0x1F](#port-0x1f-rtc-reset-register) | 0x0C | Reset register |

### Other / mysterious ports

| Port | Init | Description |
| ---- | ---- | ----------- |
| [0x0D](#port-0x0d-cpu-clock-speed) | 0x30 | CPU clock speed |
| [0x2F](#port-0x2f-irq-speed) | 0x80 | IRQ frequency |
| [0x20](#port-0x20-gpio-group-3-copy) | | Copy of GPIO group 3 |
| [0x0C](#port-0x0c-unknown) | | Unknown |

## List of ports by number

Only used ports are listed.

### Port 0x00 (screen memory location)

* Direction: write
* Default value: `0x00`
* Requires shadow: *Yes*

**Note**: This port is only usd in "new" Mailstations!

Bits 0-5 set the 256-byte page in the `0xC000` area where the screen memory starts. Therefore `0x00` means `0xC000`, `0x01` means `0xC100` and so on.

The CPU mirrors all writes to the screen area and sends them directly to the LCD controller. This mechanism is not used in "old" Mailstations, which use memory-mapped LCD controller devices instead that require manual copying of data from RAM to the LCD.

### Port 0x01 (keyboard scan)

* Direction: read/write
* Default value: `0x00`
* Requires shadow: No

Value written to this port sets the keyboard row bits 0-7. Value read from this port gets the keyboard column bits 0-7.

Because the keyboard matrix has 10 rows, the remaining two rows are controlled by bits 1-0 of port `0x02`.

By default, the firmware writes `0x00` to this register, to enable reading from all rows at once. This way, any value read other than `0xFF` indicates that some key is pressed, anf the 64 Hz interrupt routine can proceed with testing the whole keyboard matrix.

### Port 0x02 (GPIO group 1)

* Direction: read/write (depending on bit direction)
* Default value: `0xA4`
* Requires shadow: **Yes**

Individual GPIO bits in this port have the following purpose:

| Bit | Direction | Description |
| --- | --------- | ----------- |
| 7   | Output    | LCD enable |
| 6   | Input     | Caller ID data [^1] |
| 5   | Output    | Modem power on |
| 4   | Output    | New email LED |
| 3   | Output    | LCD controller column latch |
| 2   | Input     | Caller ID ready [^1] |
| 1   | Output    | Keyboard matrix row 9 |
| 0   | Output    | Keyboard matrix row 8 |

### Port 0x03 (Interrupt mask)

* Direction: read/write
* Default value: `0x22`
* Requires shadow: **Yes**

Writing the registers enables or disabled specific interrupts. Reading shows which interrupts are pending.

Acknowledging an interrupt is done by disabling it, the enabling again.

| Bit | Interrupt source |
| --- | ---------------- |
| 7   | Caller ID |
| 6   | Modem |
| 5   | Real Time Clock |
| 4   | 1 Hz timer |
| 3   | Unused |
| 2   | Unused |
| 1   | 64 Hz timer |
| 0   | Unused |

### Port 0x05 (0x4000 device page)

* Direction: read/write
* Default value: *none*
* Requires shadow: No
* See also: [Port 0x06](#port-0x06-0x4000-device-selection)

This port select the 16 kB page for the device selected by port `0x06`. Makes only sense for ROM and RAM chips.

### Port 0x06 (0x4000 device selection)

* Direction: read/write
* Default value: *none*
* Requires shadow: No
* See also: [Port 0x05](#port-0x05-0x4000-device-page)

This port select the device that appears at memory location `0x4000-0x7FFF` in the CPU address space:

| Value | Device |
| ----- | ------ |
| 0     | Code Flash ROM |
| 1     | RAM |
| 2     | Left LCD controller |
| 3     | Data Flash ROM |
| 4     | Right LCD controller |
| 5     | Modem |

### Port 0x07 (0x8000 device page)

* Direction: read/write
* Default value: *none*
* Requires shadow: No
* See also: [Port 0x08](#port-0x08-0x8000-device-selection)

This port select the 16 kB page for the device selected by port `0x08`. Makes only sense for ROM and RAM chips.

### Port 0x08 (0x8000 device selection)

* Direction: read/write
* Default value: *none*
* Requires shadow: No
* See also: [Port 0x07](#port-0x07-0x8000-device-page)

This port select the device that appears at memory location `0x8000-0xBFFF` in the CPU address space.

### Port 0x09 (GPIO group 2)

* Direction: read/write
* Default value: `0x0F`
* Requires shadow: *Yes*

Individual GPIO bits in this port have the following purpose:

| Bit | Direction | Description |
| --- | --------- | ----------- |
| 7   | Input     | 1 = Power good from AC adapter |
| 6   | Input     | 1 = Power good from battery |
| 5   | Input     | 1 = Power level sufficient |
| 4   | Input     | 0 = Power button pressed |
| 3   | Output    | Printer port /SELECT (pin 17) |
| 2   | Output    | Printer port /RESET (pin 16) |
| 1   | Output    | Printer port /AUTOLF (pin 14) |
| 0   | Output    | Printer port /STROBE (pin 1) |

### Port 0x0A (GPIO group 2 direction)

* Direction: write
* Default value: `0x0F`
* Requires shadow: No
* See also: [Port 0x09](#port-0x09-gpio-group-2)

Pin direction for port `0x09` (0 = input, 1 = output).

### Port 0x0B (GPIO group 1 direction)

* Direction: write
* Default value: `0xFF`
* Requires shadow: No
* See also: [Port 0x02](#port-0x02-gpio-group-1)

Pin direction for port `0x02` (0 = input, 1 = output).

### Port 0x0C (unknown)

* Direction: unknown
* Default value: *none*
* Requires shadow: Unknown

Unknown purpose, but most bits can be set or cleared. Reads `0xE3` after reset.

### Port 0x0D (CPU clock speed)

* Direction: write
* Default value: `0x32`
* Requires shadow: *Yes*

Bits 4 and 6 set the CPU cpeed:

| Value | Speed  |
| ----- | -----  |
| 00    | 12 MHz |
| 01    | 10 MHz |
| 10    | 12 MHz |
| 11    | 8 MHz  |

Bits 1-2 set the frequency of the level 1 IRQ:

| Value | Frequency |
| ----- | --------- |
| 00    | 128 Hz    |
| 01    | 64 Hz     |
| 10    | 32 Hz     |
| 11    | 16 Hz     |

The meaning of other bits is not known. Thus, setting the default value of `0x32` sets CPU speed to 10 MHz (bit 6 = 0, bit 4 = 1) and interrupt frequency to 64 Hz (bit 1 = 1, bit 0 = 0).

### Port 0x10 (RTC seconds low)

Lower decimal digit of the clock seconds (only available for the timer, unused for the alarm).

### Port 0x11 (RTC seconds high)

Higher decimal digit of the clock seconds (only available for the timer, unused for the alarm).

### Port 0x12 (RTC minutes low)

Lower decimal digit of the clock minutes.

### Port 0x13 (RTC minutes high)

Higher decimal digit of the clock minutes.

### Port 0x14 (RTC hour low)

Lower decimal digit of the clock hours.

### Port 0x15 (RTC hour high)

Higher decimal digit of the clock hours.

### Port 0x16 (RTC day of the week)

Weekday number (0-6).

### Port 0x17 (RTC day low)

Lower decimal digit of the day of the month.

### Port 0x18 (RTC day high)

Higher decimal digit of the day of the month.

### Port 0x19 (RTC month low)

Lower decimal digit of the month number (1-based) (only available for the timer).

### Port 0x1A (RTC month high)

Higher decimal digit of the month number (1-based) (only available for the timer).

### Port 0x1B (RTC year low)

Lower decimal digit of the year number minus 1980 (timer) or 24/12 hour format selection (alarm).

### Port 0x1C (RTC year high)

Higher decimal digit of the year number minus 1980 (timer) or leap year digits (alarm).

### Port 0x1D (RTC page register)

* Direction: read/write
* Default value: `0x04`
* Requires shadow: No

Seems to correspond to TC8521 page register. The bits seem to have the following purpose:

| Bit | Description |
| --- | ----------- |
| 3   | 1 = Timer enable |
| 2   | 1 = Alarm enable |
| 1   | Page number bit 1 |
| 0   | Page number bit 0 |

The page selection determines what data is read/written to registers `0x10-0x1C`:

| Page | Descrption |
| ---- | ---------- |
| 0    | Timer |
| 1    | Alarm, clock format, leap year |
| 2    | Static ram 1 |
| 3    | Static ram 2 |

### Port 0x1E (RTC test register)

* Default value: `0x00`

Seems to correspond to TC8521 test register. If that's the case, must be always set to 0.

### Port 0x1F (RTC reset register)

* Direction: read/write
* Default value: `0x0C`
* Requires shadow: No

Seems to correspond to TC8521 reset register. The bits seem to have the following purpose:

| Bit | Description |
| --- | ----------- |
| 3   | 0 = 1 Hz interrupt enable |
| 2   | 0 = 16 Hz interrupt enable |
| 1   | 1 = timer reset |
| 0   | 1 = alarm reset |

### Port 0x20 (GPIO group 3 copy)

* Direction: read
* Default value: *none*
* Requires shadow: No

This appears to be a copy of [port 0x21](#port-0x21-gpio-group-3)

### Port 0x21 (GPIO group 3)

* Direction: read
* Default value: *none*
* Requires shadow: No

This port is used exclusively for reading printer status lines:

| Bit | Direction | Description |
| --- | --------- | ----------- |
| 7   | Input     | Printer port /BUSY (pin 11) |
| 6   | Input     | Printer port /ACK (pin 10) |
| 5   | Input     | Printer port /PAPER_OUT (pin 12) |
| 4   | Input     | Printer port /SELECT (pin 13) |
| 3   | Input     | Printer port /ERROR (pin 15) |
| 2   | Input     | Unknown, possibly ring indicator? [^2] |

### Port 0x26 (Modem reset)

* Direction: write
* Default value: `0x00`
* Requires shadow: No

Bit 0 of this port is connected to the modem reset pin. To reset the modem properly, write `0x00` to this port, wait a short while, then write `0x01`.

### Port 0x28 (GPIO group 4)

* Direction: write
* Default value: `0x00`
* Requires shadow: **Yes**

Individual GPIO bits in this port have the following purpose:

| Bit | Direction | Description |
| --- | --------- | ----------- |
| 3   | Output    | Unknown, related to caller ID (FSK/DTMF select?) |
| 2   | Output    | Caller ID reset |
| 1   | Output    | Caller ID clock |
| 0   | Output    | System power off |

### Port 0x29 (GPIO group 4 direction)

* Direction: write
* Default value: `0x03`
* Requires shadow: No
* See also: [Port 0x28](#port-0x28-gpio-group-4)

Pin direction for port `0x28` (0 = input, 1 = output)

### Port 0x2C (GPIO group 5 direction)

* Direction: write
* Default value: `0xFF`
* Requires shadow: No
* See also: [Port 0x2D](#port-0x2d-gpio-group-5)

### Port 0x2D (GPIO group 5)

* Direction: write
* Default value: `0x00`
* Requires shadow: No

This port controls the eight printer port data lines.

### Port 0x2F (IRQ speed)

* Direction: read/write
* Default value: `0x80`
* Requires shadow: No

Bit 7 is set by firmware very early in the reset procedure, the purpose of this is not know.

Bits 4-6 set the rate of the level 4 IRQ:

| Value | Rate |
| ----- | ---- |
| 000   | 1s   |
| 001   | Disabled |
| 010   | Disabled |
| 011   | Disabled |
| 100   | 1s   |
| 101   | 2s   |
| 110   | 4s   |
| 111   | 8s   |

Setting bits 2 and 3 seem to affect I/O lines in a strange way, switching off the LCD and LED (possibly disabling the I/O ports or tristating them?).

Bits 0 and 1 seem to have no effect.

# Footnotes

[^1]: Firmware code reads these bits, but by default they are set as outputs.

[^2]: Firmware code reads this bit, but the purpose is not fully understood. It is connected to pin 3 of U603 on the motherboard.

