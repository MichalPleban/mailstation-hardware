# Interrupt handling

There are several interrupt sources on he MailStation, including two timers, the modem chip and the real time clock. They are all capable to generating a *maskable* interrupt (IRQ) to the processor; the computer does not use non-maskable interrupts (NMI).

The interrupts are controlled by the I/O port `0x03`. The port has different functions when being read and being written to:

**Writing to the port** enables or disables a specific interupt source. To enable a specific source, set the bit to 1:

| Bit | Interrupt source |
| --- | ---------------- |
| 7   | Caller ID |
| 6   | Modem |
| 5   | Real Time Clock |
| 4   | "Slow"" timer |
| 3   | Unused |
| 2   | Unused |
| 1   | "Fast" timer |
| 0   | Unused |

**Reading from the port** gets the mask of pending interrupts. If a bit reads 1, it means that the coresponding interrupt is pending. You normally read the port value in the interrupt handler, in order to determine the source of the interrupt.

Please note that reading the port *does not* clear the interrupt, so upon returning from the interrupt service routine the IRQ will be immediately fired again. In order to acknowledge a specific interrupt, you need to disable it, and then re-enable it again.

Since reading and writing to the port have different functions, you need to maintain a copy (a so-called "shadow register") of the values written to the port, in order to be be able to set and reset individual bits (for example to acknowledge an interrupt).

## Timer interrupts

The processor in the MailStation has two timers, which can be used for fine- or coarse-grained time events. 

The first timer (corresponding to bit 1 in port `0x03`) is managed by the I/O port `0x0D`; the bits 0-1 of this port allow to change the interrupt frequency from 16 to 128 times per second:

| Value | Frequency |
| ----- | --------- |
| 00    | 128 Hz    |
| 01    | 64 Hz     |
| 10    | 32 Hz     |
| 11    | 16 Hz     |

The second timer (corresponding to bit 4 in port `0x03`) is managed by the I/O port `0x2F`; the bits 4-6 of this port allow to change the interrupt frequency from once every second to once every 8 seconds:

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

