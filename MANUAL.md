# manual

In addition to the core BBC Basic for Z80 core language (details of which [can be found here](bbcbasic.txt)), BBC Basic for Next adds the following functionality:

## Editor

A line editor is provided, that allows the user to enter a line of code up to 255 characters long. A cursor can be moved around this line, and text can be deleted or inserted at the current character position.

A copy mode is provided, by pressing EDIT (Shift+1). When in copy mode, the cursor can be moved around the screen. Pressing Delete (Shift+0) will copy the character under the cursor into the current line. Pressing EDIT or CR will exit copy mode.

## Assembler

The assembler has been extended to handle Z80N instructions.

See [assembler_Z80N.bbc](tests/source%20text/assembler_Z80N.txt) in the folder [tests/source text](tests/source%20text) for a usage example.

## BASIC

The following statements differ from the BBC Basic standard:

### PUT port,value

Write to a port; add &10000 to port to write to a NEXTREG register

Examples:

- `PUT 254,5` Output 5 to Z80 port 254
- `PUT &10007,3` Output 3 to Next register 7

### GET(port)

Read from a port; add &10000 to port to read from a NEXTREG register

Examples:

- `A = GET(254)` Read from Z80 port 254
- `A = GET(&10007)` Read from Next register 7

### GET(x,y)

Read a character code from the screen position X, Y. If the character cannot be read, return -1

Example:

- `A = GET(0,0)` Read character code from screen position(0,0)

### GET$(x,y)

As GET(x,y), but return a string rather than a character code

Example:

- `A$ = GET$(0,0)` Read character code from screen position(0,0)

### PLOT type,x,y

The only plot modes supported currently are:

- `PLOT 0, X, Y` Draw a line from last plot position to X, Y
- `PLOT 64, X, Y` Plot a point
- `PLOT 80, X, Y` Draw a filled triangle. The other two points are the last two plot positions.
- `PLOT 144, 0, R` Draw a circle of radius R. The center position is the last plot position

Examples:

Draw a triangle:

`MOVE 10,15: MOVE 253,40: PLOT 80,55,181`

Draw a circle

`MOVE 128,96: PLOT 144,0,95`

### MODE n

Three modes supported:

- `MODE 0` ULA mode (Normal Spectrum 256x192 Graphics)
- `MODE 1` Layer 2 mode (256 x 192, 256 colours per pixel)
- `MODE 2` Layer 2 mode (320 x 256, 256 colours per pixel)
- `MODE 3` Layer 2 mode (640 x 256, 16 colours per pixel)

### COLOUR n[,type]

An additional optional parameter, type, has been added. If not specified, will default to 0.

- 0: Set colour as BBC Standard; add 128 to set paper colour.
- 1: Set foreground (ink) colour
- 2: Set background (paper) colour
- 3: Set border colour
- 4: Set bright (Mode 0 only)
- 5: Set flash (Mode 0 only)

Mode 1 and 2 are useful in Mode 1, as it allows you to access all 256 Next colours. Mode 0 is maintained as a default, but will only allow you to access the first 128 colours.

Examples:

- `COLOUR 2,3` Set the border colour to Red
- `COLOUR 1,255` Set the foreground colour to 255 (Mode 1)
- `COLOUR 5,1` Set to flash (Mode 0)

### GCOL mode, colour[, type]

Sets the graphic colour, as per COLOUR

Mode values for MODE 0 is currently one of the following:
- 0 and 1 set the pixel
- 2 and 6 clear the pixel
- 3 and 4 invert the pixel
This is an attempt to stick to the BBC BASIC standard with 1-bit graphics

Mode for MODE 1 should work as per BBC Basic specifications but currently only supports 0 and 6 due to a memory mapping issue I need to resolve.

All types are valid for Mode 0. Mode 1 only recognises type 0, 1 and 3.

### VDU

The VDU command is a work-in-progress with a handful of mappings implemented:

- `VDU 8` Backspace
- `VDU 9` Advance one character
- `VDU 10` Line feed
- `VDU 11` Move cursor up one line
- `VDU 12` CLS
- `VDU 13` Carriage return
- `VDU 16` CLG
- `VDU 17,col` COLOUR col
- `VDU 18,mode,col` GCOL mode,col
- `VDU 19,l,r,g,b` COLOUR l,r,g,b
- `VDU 22,n` Mode n
- `VDU 23,c,b1,b2,b3,b4,b5,b6,b7,b8` Define UDG c
- `VDU 25,mode,x;y;` PLOT mode,x,y
- `VDU 29,x;y;` Set graphics origin to x,y
- `VDU 30` Home cursor
- `VDU 31,x,y` TAB(x,y)

Examples:

`VDU 25,64,128;88;` Plot point in middle of screen

`VDU 22,1` Change to Mode 1

### SOUND channel,volume,pitch,duration

Play a sound

- channel: AY channel between 0 and 2
- volume: 0 (off), -15 (loud)
- pitch: Pitch in quarter-semitones, where 53 is middle C, and 89 is A440
- duration: Duration in 25ths of a second

Sounds are played asynchronously; up to 5 notes can be queued ahead. If a sixth note is added to the queue, the SOUND command will block until there is space in the queue.

## STAR commands

The star commands are all prefixed with an asterisk. These commands do not accept variables or expressions as parameters. Parameters are separated by spaces. Numeric parameters can be specified in hexadecimal by prefixing with an '&' character. Paths are unquoted. 

If you need to pass a parameter to a star command, call it using the OSCLI command, for example:

- `LET T% = 3: OSCLI("TURBO " + STR$(T%))`

### BYE

Exits BBC Basic by doing a soft reset (does not work on emulators)

### CAT (or .)

List the contents of the current directory

- `*CAT`

### DIR

Change the current directory; works in much the same way as a PC/Mac command line:

- `*DIR name` Change to the specified directory by name
- `*DIR ..` Go back up a directory
- `*DIR \` Go to the root directory

Aliases: CD

### ERASE

Erase a file

- `*ERASE test.bbc` Erase the file test.bbc
- `*ERASE *.txt` Wildcards are permitted

Aliases: DELETE

### MKDIR

Make a directory

- `*MKDIR tests` Make the folder tests

### RMDIR

Remove a directoy

- `*RMDIR tests` Remove the folder tests

### DRIVE

Select the current working drive

- `*DRIVE D` Change to drive D
- `*DRIVE $` Change to the default drive

### LOAD

Load a block of memory in

- `*LOAD SCREEN.SCR 16384` Load file SCREEN.SCR to address 168384

### SAVE

Save a block of memory out

- `*SAVE SCREEN.SCR 16384 6912` Save the screen out to file SCREEN.SCR

### TIME

Output the current RTC time (if RTC fitted) in BBC Master format.

The time is also available in the system variable TIME$

### TURBO n

Will set the Next CPU turbo mode:

- `*TURBO 0`: 3.5Mhz
- `*TURBO 1`: 7Mhz
- `*TURBO 2`: 14Mhz
- `*TURBO 3`: 28Mhz

### MEMDUMP start len

List contents of memory, hexdump and ASCII.

- `*MEMDUMP 0 200` Dump the first 200 bytes of ROM

### FX n

- `*FX 19`: Wait for the horizontal sync
- `*FX 20 n`: Reserve space for UDGs in RAM

## Other considerations

I use ULA hardware scrolling, so the top-left screen address is not guaranteed to be 0x4000, likewise the attributes address at 0x5800.
