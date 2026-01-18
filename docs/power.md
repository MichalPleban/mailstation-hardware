# Power management

The MailStation hardware has several functions that allow for power management:

## Power button

The power button is a special button that, when pressed, turns the machine on. The reverse is, however, not true: pressing it again will not turn it off. You need to detect the power key presses and turn the computer off yourself. This means that if your software hangs or does not handle the power key properly, the computer will never power off unless disconnected from power or reset by the hardware reset switch!

In order to check the status of the power key, read the bit 4 of of the I/O port `0x09`. If the bit value is 0, the power button is pressed.

## Power off

To power off the machine, write the value 1 to bit 0 of the I/O port `0x28`. The computer will power off immediately.

## Power status

You can check if the computer is connected to the AC adapter, check the value of bit 7 in the I/O port `0x09`. Similarly, bit 6 shows if there are batteries in the computer, and bit 5 whether any power source (the AC adapter or the batteries) gives at least 6V of power.

