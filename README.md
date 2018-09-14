# Introduction

The ARSE (Advanced Rigid Solderless Experiment) is a general purpose homebrew 8-bit computer built from a Z80 CPU. It is aimed to be a minimum design for a working general purpose computer.

The ARSE is capable of being programmed and operated interactively through a USB port by a serial terminal or terminal emulator. It can be programmed to operate autonomously through the USB port. The boot ROM is switchable between the system ROM and the user-programmable EEPROM. Alternative ROMs could also be substituted for the system ROM and second ROM, although this is not recommended as it could lead to loss of control of your ARSE.

Your ARSE has two EEPROMs to allow storage of your own programs and data, but also to allow bootstrapping if you wish to modify the system software. There are some system routines available which are summarised in the relevant section of the documentation.

# Specification

    •	Z80 running at 1.8Mhz
    •	32K RAM
    •	2x16K EEPROM max ('factory' fitted with 2x8K)
    •	UART running at 19,200 BAUD, 8-N-1, no handshaking
    •	USB port
    •	Switchable boot-ROM
    •	5V 1A power supply through USB
    •	Soft reset momentary switch
    •	Two programmable LEDs
    •	Power LED
    •	Proprietary ARSEOS with interactive command mode and basic machine monitor.

# Operation - Basic User Manual (BUM)

This section comprises a Basic User Manual or BUM. The BUM is designed to facilitate use of your ARSE and the operating software - 'ARSEos'.

## ROM switch

The ROM switch will switch between `ROM0` and `ROM1` as bootup ROM, that is it will switch the ROMs around with one another in the memory map. With the switch at position '`ROM0`' `ROM0` will be at address $0000 with `ROM1` at $4000. With the switch at position '`ROM1`' `ROM1` will be at address $0000 and `ROM0` at $4000. Do not adjust the switch while your ARSE is powered up, this will have an undefined effect and could damage your ARSE. `ROM0` does not allow writing for protection of the system software.

## Power up and reset

Plug a USB extension lead female end into the USB port on your ARSE, the other end should be connected to a powered USB port on a computer or device. Then press the `RESET` switch; this will initialise your ARSE and begin program execution at memory location $0000 - this will either be `ROM0` or `ROM1` depending on the position of the ROM switch.
The `RESET` switch will not clear the contents of RAM. For a complete hard reset, unplug the USB lead from your ARSE. The `RESET` switch must always be pressed after power is applied to initialise your ARSE.
On reset, if `ROM0` is selected as boot ROM then the ARSE will output the ARSEOS Monitor Main Menu to the device connected and then enter input mode (see next section).

## Input / Output

All text input and output is via the USB port at 38400 BAUD (bits per second). The ARSE can be operated and programmed using a suitable terminal emulator on a personal computer; for example puTTY or screen:
On a Mac: `screen /dev/cu.SLAB_USBtoUART 38400`

## The ARSEos Monitor

The ARSEos Monitor consists of basic functionality to enable rudimentary monitoring and programming of your ARSE. The Monitor consists of a basic command mode with each command activated by a single character input from the UART and a corresponding display of characters. On initialisation, the Monitor displays the Monitor Main Menu followed by the memory location status showing the currently selected memory location which will be $0000 at initialisation and the contents of that memory location in hexadecimal followed by the ASCII character for that code, if applicable.

The Monitor will allow the user to display, dump, set and execute any address. It will also allow copy of any length from one address to another.

The Monitor has a 'current address' dictating where in memory the current action or command will operate. The current address can be changed by pressing return to increment, backspace to decrement, or entering a four digit hexadecimal number (`0`-`9`, `A`-`F`) and pressing `return`. The address input is a rolling 4-digit input so that if more than four hex digits are entered, the last four are used. Ensure you use upper case to initiate digit-entry mode if the desired address starts with a letter - eg. `C123`, not `c123`.

### Memory Location Status Display

```
[0000] 65 (A)
|----------- Currently selected memory location
       |-------- Current contents of memory location
           |------ ASCII character for contents of memory location
```

### Commands

The commands are:

- `h` Display Monitor Main Menu
- `space` Display memory location status
- `return` Increment current memory address
- `backsp` Decrement current memory address
- `l` (ell) list 16 memory locations and advance current address
- `d` begin dump of memory from current address
- `c` copy contents of memory
- `shift-S` set contents of current address and increment current address
- `shift-X` execute program at current address
- `A`-`F` enter address entry mode
- `0`-`9` enter address entry mode
- Any other input will cause the ARSE to respond with `?`

## List (lower case l) and Dump (lower case d)

The list function will output one line showing the contents in hexadecimal and ASCII (if applicable) of 16 memory locations from the current address and the current address will be advanced by 16 locations. If the ASCII character for the contents of a memory location is not displayable then a dot '`.`' is displayed. The displayed address (and current address in List mode) will pass through $FFFF to $0000.

The displayed line is in the following format:

```
0000 nn nn nn nn nn nn nn nn nn nn nn nn nn nn nn nn ABCDEFGHIJKLMNOP
|------------ Current address
     |-------------- hexadecimal contents of address
        |---------------- hexadecimal contents of address+1
ASCII character for each memory location ----------------------|
```

The dump function will continually output lines of 16 memory locations until a key is pressed (ie. a character is received through the UART). The dump function does not alter the current address.

### Copy (lower case c)

The copy function will copy a range of memory to a specified address using confirmed writes. Confirmed writes are used to write to ROM so are slower than simple `LD` instructions. When the copy function is selected by sending a '`c`' in command mode, ARSEOS will respond with:

```
copy fr:[0000]
```

ARSE is now in four character address entry mode. Enter the address to copy from and press return. To abort, send `escape` (ASCII `$1B`). ARSEOS will now respond with:

```
to:[0000]
```

enter the destination address for the copy and press return. If you make a mistake at this point, the copy operation can be aborted at the next stage by entering 0 at the length: prompt. When the destination address has been submitted, the ARSE will respond with:

```
length:[0000]
```

enter the number of bytes in hexadecimal to be copied from the source address to the destination address and send return. Entering a length of zero will abort the copy operation.

Note that the copy command is indisciminate; it will attempt to write to wherever it is instructed, including ROM1 (although ROM0 is hard-wired write protected). The copy operation will loop over $FFFF to $0000.

### Set (upper case S)

The set function enables setting of the contents of memory locations and increments the current address. On selecting Set mode, ARSE will respond with:

```
SET [aaaa][nn]
```

where aaaa is the current address and nn is the current contents of that address. ARSE is now in two digit input routine; enter the desired new content of the memory location and press return. Pressing escape will abort the set operation and return you to Memory Location Status Display. On pressing return, the memory location contents will be set as entered, the current address will be incremented and you will still be in set mode. This is designed to allow continuous setting of contiguous blocks of memory (for as long as you have the patience). To exit set mode, send escape.

### Execute (upper case X)

The execute function will transfer program execution to the current address after a confirmation message. This is accomplished simply by setting the CPUs program counter to the current address - there is no sanity checking performed so proceed with caution. ARSE can be reset by eXecuting location 0000. When execute mode is selected, ARSE will respond with:

```
exec [aaaa]
```

and wait for a character. Any character recieved other than return will abort and return you to Memory Location Status Display. Sending return will transfer execution to the instruction at the current address.
Be careful! - your ARSE will simply start executing instructions at the current address; you are most likely leaving ARSEos at this point.

# TODO

- add interrupt-driven UART I/O
- add CALL command (call code at location nnnn as procedure)
- don't use slow EEPROM write for everything
