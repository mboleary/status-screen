# Interfacing with the Arduino Status Screen

The Arduino should act as a Generic Driver for the Screen so that any embedded device can be programmed to interact with the PC driver.

## Device Specification (Short version)

the Screen device can either be a Character-based LCD, or a Graphical SPI LCD screen. As such, there are serial commands to deal with either type of device.

## Opcodes and Commands:

Keep in mind that a byte has a possible value range from 0 to 128

bytes:
01234567890123456789
ioa...             \n

where: 
- `i` - Device ID
- `o` - Opcode
- `a` - Arguments to the command
- `\n` - Specifies the end of a command, an endline

### Return Data

Data will be returned to tell the PC client whether or not the command was successfully run, among other things

btyes:
0123456789
cd...    \n

where:
- `c` - Return Code (See Return Code Section)
- `d` - Return Data (if applicable)
- `\n` - End of the return data, and endline

### Opcodes

List of Byte Values to determine the command

- Functions (Generic to type of display): `0-7`
    - `0` - Get Device Info
        - [0|1] Off / On
    - `1` - Debug Mode
        - [0|1] Off / On
    - `2` - Show Debug Message
        - Note: Will not do anything if Debug Mode is off
        - `n` bytes: ASCII Message to display + `\n`
    - `3` - Emulate Text LCD

- Platform-specific Command (Graphics SPI Display): `8+`
    - `8` - Fill Display
        - Color Value (2 bytes)
    - `9` - Draw Screen
        - n bytes: Array of Color Values (2 bytes each) like a scan line for a CRT display
    - `10` - Draw Chunk
        - Resolution of chunk to draw
        - Offset of Chunk from Origin
        - Array of Pixels to write
    - `11` - Draw Pixel
        - Coordinates of Pixel
        - Pixel Color
    - Extra Functions that are provided by Adafruit GFX Library
    - `12` - Draw Text
        - Coordinates
        - Text Color
        - 
    - `13` - Draw Shape

- Text Display Platform: `8+`
    - `8` - Clear Display
    - `9` - Clear custom character
    - `10` - Write Text
    - `11` - Write Character
    - `12` - Set Custom Character


### Return Code

List of Byte Values to determine the result of the command

- `0` - Successful
- `1` - Unsuccessful
    - Specify Bad Command Parameters
- `2` - Not Supported

### End of Command

HEX Value: `0x0A` -> `LF`

Since there are situations where `\n` would be used when specifying the colors, we'll have to be smart about how we interpret the end of a command line

Modes:
- Normal:
    Allow for ending with a `\n`
- Color
    Don't check when interpreting colors, as we have a calculated amount for the bytes