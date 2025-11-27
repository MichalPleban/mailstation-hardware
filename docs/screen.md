# Screen organization

The Mailstation has a 320x128 pixel LCD screen, where evey pixel is directly adressable. The screen interface is significantly different between the "old screen" models (DET1 and DET2) and "new screen" models (DET1D, DET1E).

## Old screen

The original Mailstation, as well as the Mivo 200/250 variant, drives the screen through dedicated LCD controllers which are mapped to the CPU address space; the screen data needs to be copied there in order to be displayed.

The screen is divided into two halves: left and right. Each is accessed as a device mapped into the CPU RAM, with device IDs `0x02` and `0x04` respectively. To map a device, set the port `0x06` or `0x08` to one of these values, and the device can then be accessed at the address `0x4000` or `0x8000`.

Each LCD controller has 256 directly addressable bytes. That is, of course, not enough to access all the 160x128 pixels it controls. Therefore writing LCD data is done in two phases:

1. First you need to select the 8-bit screen column that you want to access.
2. Then, you write actual pixel data for this column.

The correct procedure is therefore as follows:

1. Set the bit 3 of port `0x02` to 0.
2. Write the column number to the LCD controller (the actual address does not matter).
3. Set the bit 3 of port `0x02` to 1.
4. Write the bytes for the column to the controller, starting at adsress `0x38`.

The columns are numbered from right to left, i.e. the leftmost column has the number 0, and the rightmost one the number 19. In the bytes describing the pixel data, bit 0 is the leftmost pixel and bit 7 is the rightmost pixel. This is an uncommon convention - most screens use the opposite mapping of bits.

Note that accessing the LCD controllers this way requires manipulating a bit in port `0x02`, which also contains two bits of the keyboard matrix. It is therefore necessary to access this port with interrupts disabled, otherwise the keyboard scanning routine in the interrupt handler can overwrite this bit, leading to strange errors.

The screen organization is described in this picture:

![LCD Old](lcd_old.svg)

## New screen

The "blue LCD" Mailstations, i.e. DET1D and DET1E, use a completely different screen controller, which is built in into the CPU. There is no more need to write the data to the controller; the CPU uses DMA to fetch the data from memory and drive the LCD. The downside is that the screen data needs to be laid out in the first 16 kB of RAM, on a 256-byte boundary and in a specific format. 

The screen format is fortunately straightforward: the bytes are simply laid out sequentially, row by row, with bit 7 of every byte corresponding to the leftmost pixel (thus in the opposite to the old screen). The location of the screen data is controlled by port `0x00`, which selects the start address in RAM in 256-byte increments. Only 6 bits are used, so the screen needs to start somewhere in the first 16 kB of RAM; of the start address is set near the end of this region, the data will wrap around.

The screen organization is described in this picture:

![LCD New](lcd_new.svg)