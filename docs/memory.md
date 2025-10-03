# Memory map

The 8-bit Z80 CPU in the MailStation can only address 64 kB at a time. Therefore, like with many other Z80 machines, a banking scheme must be employed in order to access the bigger memory chips of the machine.

Additionally, I/O chips (modem and LCD controller) that would normally be mapped in the Z80 I/O space, are memory-mapped on the MailStation. This is because the I/O space is only used internally by the CPU for its various registers.

The 64 kB CPU space is divided into four 16 kB page:

| Address             | Mapping                                |
| ------------------- | -------------------------------------- |
| `0x0000` - `0x3FFF` | Always the beginning of Code Flash ROM |
| `0x4000` - `0x7FFF` | Available to map any device            |
| `0x8000` - `0xBFFF` | Available to map any device            |
| `0xC000` - `0xFFFF` | Always the beginning of RAM            |

The first and last 16 kB page cannot be changed - the beginning of the address space, where the reset and interrupt vectors are located, is always mapped to ROM, whereas the end of the address space, where the stack pointer would normally be placed, always maps to RAM.

The remaining two areas can be remapped to any device:

| Value | Device |
| ----- | ------ |
| 0     | Code Flash ROM |
| 1     | RAM |
| 2     | Left LCD controller |
| 3     | Data Flash ROM |
| 4     | Right LCD controller |
| 5     | Modem |

For the ROM and RAM chips, it is also possible to specify which 16 kB area of the chip will be available in the given page. The mapping is done by writing to I/O ports `0x05` - `0x08`:

| Port | Description |
| ---- | ----------- |
| [0x05](ports.md#port-0x05-0x4000-device-page) | 0x4000 device area |
| [0x06](ports.md#port-0x06-0x4000-device-selection) | 0x4000 device selection |
| [0x07](ports.md#port-0x07-0x8000-device-page) | 0x8000 device area |
| [0x08](ports.md#port-0x08-0x8000-device-selection) | 0x8000 device selection |

For example, to map the area `0x10000` - `0x13FFF` of the RAM chip to CPU addresses `0x4000` - `0x7FFF`, one would write the value `1` to port `0x06`, and `4` to port `0x05`. 

All these I/O ports can be read back; it is thus possible to save and restore the device mapped to a specific page when temporarily using another one (like one of the LCD controllers).

This mapping has the following consequences:

* It is not possible to change the CPU reset vector without reflashing the whole Code Flash ROM.
* To use a custom IRQ handler, the application must use the IM 2 interrupt trick.
* The computer cannot run CP/M, as the boot ROM at the bottom of memory cannot be mapped out.

Commonly, the page at `0x4000` is used to access ROM (for example, when custom apps are ran from the Data Flash, the firmware maps the appropriate chip area there), `0x8000` for accessing I/O chips and additional areas of RAM, and `0xC000` for any common variables and stack.
