# SYSEX

System Exclusive messages are special MIDI messages of variable size. They are used for device-specific communication.
In particular, lots of physical devices that are known as MIDI controllers can usually be configured by sending specially crafted SYSEX MIDI messages.

## SYSEX messages

SYSEX messages are of variable size. They are formed of a prefix byte `F0` and a suffix byte `F7`. All bytes between these two markers can only be coded on **7 bits**, i.e. bytes values should always be <= 127.

`F0 .. .. .. .. F7`

## SYSEX Device ID

Usually a device aware of SYSEX messages respond to a generic SYSEX ID request.

The Identity Request message is formed of the following bytes:

| Byte | Description                                                                           |
|------|---------------------------------------------------------------------------------------|
| F0   | SysEx                                                                                 |
| 7E   | Non-Realtime                                                                          |
| 7F   | The SysEx channel. Could be from 0x00 to 0x7F. Here we set it to "disregard channel". |
| 06   | Sub-ID -- General Information                                                         |
| 01   | Sub-ID2 -- Identity Request                                                           |
| F7   | End of SysEx                                                                          |

See https://www.midi.org/specifications-old/item/manufacturer-id-numbers for a list of manufacturer IDs

# Arturia MIDI controllers

(`F0` and `F7` prefix/suffix bytes are omitted in the following)

## Arturia Minilab mk II

### SYSEX Device ID

| Byte                    | Description                                              |
|-------------------------|----------------------------------------------------------|
| 7E                      | Non-Realtime                                             |
| 00                      |                                                          |
| 06                      | Sub-ID -- General Information                            |
| 02                      | Sub-ID2 -- Identity Reply                                |
| 00 20 6B                | Arturia manufacturer ID                                  |
| 02 00 04 02 aa bb cc dd | Minilab mk ii Model ID                                   |
|                         | aa bb cc dd = 53 09 00 01 => firmware version 1.0.9.83   |
|                         | aa bb cc dd = 0e 02 01 01 => firmware version 1.1.2.1166 |

### Specific SYSEX commands

Specific SYSEX commands are always prefixed by the following byte sequence:
`00 20 6B 7F 42`

The byte following this sequence is the command to execute.

#### 01 - Read a value

This command asks the device to output part of its internal state.

The general form of a read value command is (prefix ommited):

| Value | Description      |
|-------|------------------|
| 01    | Read value       |
| 00    |                  |
| pp    | Parameter **pp** |
| bb    | Button ID **bb** |

The "button" ID **bb** refers to which knob or pad a parameter is to be retrieved.

#### 02 - write a value

This command updates the internal state by changing a parameter value of a knob or pad.

The general form of a write value command is (prefix ommited):

| Value | Description      |
|-------|------------------|
| 01    | Read value       |
| 00    |                  |
| pp    | Parameter **pp** |
| bb    | Button ID **bb** |
| vv    | Value **vv**     |

### Regular knobs

Knobs 2 to 8 and 10 to 16 are regular knobs, or rotary encoders. Their main purpose is to send a CC midi message when they are rotated.

| **bb** = "Button" ID | Designation    |
|----------------------|----------------|
| 01                   | Knob 2         |
| 02                   | Knob 3         |
| 03                   | Knob 4         |
| 04                   | Knob 5         |
| 05                   | Knob 6         |
| 06                   | Knob 7         |
| 07                   | Knob 8         |
| 08                   | Knob 10        |
| 09                   | Knob 11        |
| 0A                   | Knob 12        |
| 0B                   | Knob 13        |
| 0C                   | Knob 14        |
| 0D                   | Knob 15        |
| 0E                   | Knob 16        |
| 30                   | Knob 1         |
| 32                   | Knob 1 + shift |
| 33                   | Knob 9         |
| 35                   | Knob 9 + shift |

Available parameters for regular knobs:

| **pp** = Parameter | Description                 | Possible values        | RW |
|--------------------|-----------------------------|------------------------|----|
| 00                 | Current CC value            | 0 - 127                | RW |
| 01                 | Mode                        | 00 = OFF               | RW |
|                    |                             | 01 = Control (default) |    |
|                    |                             | 04 = NRPN              |    |
| 02                 | Channel                     | MIDI channel           | RW |
| 03                 | CC number / NRPN data entry | If mode = Control      | RW |
|                    |                             | 0-127: CC number       |    |
|                    |                             | If mode = NRPN         |    |
|                    |                             | 00 = 1:128             |    |
|                    |                             | 01 = 1:64              |    |
|                    |                             | 02 = 1:32              |    |
|                    |                             | 03 = 1:16              |    |
|                    |                             | 04 = 1:8               |    |
|                    |                             | 05 = 1:4               |    |
|                    |                             | 06 = 1:2               |    |
|                    |                             | 07 = 1:1               |    |
| 04                 | If mode = NRPN, LSB         | 0-127                  | RW |
| 05                 | If mode = NRPN, MSB         | 0-127                  | RW |
| 06                 | Option                      | If mode = Control      | RW |
|                    |                             | 00 = Absolute          |    |
|                    |                             | 01 = Relative 1        |    |
|                    |                             | 02 = Relative 2        |    |
|                    |                             | 03 = Relative 3        |    |

### Buttons

| **bb** = "Button" ID | Designation                      |
|----------------------|----------------------------------|
| 10                   | Oct -                            |
| 11                   | Oct +                            |
| 2E                   | Shift                            |
| 2F                   | Button "Pad 1-8/9-16"            |

Buttons send their state when they are pressed / released.

Available parameters for buttons:

| **pp** = Parameter | Description   | Possible values | RW |
|--------------------|---------------|-----------------|----|
| 00                 | Current state | 00 = released   | R  |
|                    |               | 7F = pressed    |    |

The Shift button is a simple button with two states: released or pressed.

"Oct -" and "Oct +" buttons allow us to change the current octave of the piano keyboard.
**Open question**: is there a way to get the current selected octave ? to set it ?

The last button allows to switch between pads 1-8 and pads 9-16.
**Open question**: is there a way to get the current pads selection ? to set it ?

### Pitch bend

| **bb** = "Button" ID | Designation |
|----------------------|-------------|
| 41                   | Pitch bend  |


| **pp** = Parameter | Description   | Possible values | RW |
|--------------------|---------------|-----------------|----|
| 00                 | Current state | 0-127           | RW |
| 01                 | Mode          | 00 = OFF        | RW |
|                    |               | 10 = Picth bend |    |
| 06                 | Option        | 00 - Default    | RW |
|                    |               | 01 - Hold       |    |
|                    |               |                 |    |

### Modulation (slider)

??

### Knob buttons

| **bb** = "Button" ID | Designation   |
|----------------------|---------------|
| 31                   | Knob 1 button |
| 34                   | Knob 9 button |

### Pads

| **bb** = "Button" ID | Designation |
|----------------------|-------------|
| 70                   | Pad 1       |
| 71                   | Pad 2       |
| 72                   | Pad 3       |
| 73                   | Pad 4       |
| 74                   | Pad 5       |
| 75                   | Pad 6       |
| 76                   | Pad 7       |
| 77                   | Pad 8       |
| 78                   | Pad 9       |
| 79                   | Pad 10      |
| 7A                   | Pad 11      |
| 7B                   | Pad 12      |
| 7C                   | Pad 13      |
| 7D                   | Pad 14      |
| 7E                   | Pad 15      |
| 7F                   | Pad 16      |

| **pp** = Parameter | Description      | Possible values               | RW |
|--------------------|------------------|-------------------------------|----|
| 00                 | Current state    | 0-127                         | RW |
| 01                 | Mode             | 00 = OFF                      | RW |
|                    |                  | 07 = MMC                      |    |
|                    |                  | 08 = Switched                 |    |
|                    |                  | 09 = MIDI note                |    |
|                    |                  | 0B = Patch change             |    |
| 02                 | Channel          | 0X = Channel X                | RW |
|                    |                  | 0x41 = Channel "Keyboard" (?) |    |
| 03                 | CC / Note        | If mode in [08,09] => note    | RW |
|                    |                  | Else => CC                    |    |
| 04                 | Off value        | 0-127                         | RW |
| 05                 | On value         | 0-127                         | RW |
| 06                 | Option           | 00 = Toggle                   | RW |
|                    |                  | 01 = Gate                     |    |
|                    |                  | If mode = MMC                 |    |
|                    |                  | 01 = Stop                     |    |
|                    |                  | 02 = Play                     |    |
|                    |                  | 03 = Deferred play            |    |
|                    |                  | 04 = Fast forward             |    |
|                    |                  | 05 = Rewind                   |    |
|                    |                  | 06 = Record Strobe            |    |
|                    |                  | 07 = Record Exit              |    |
|                    |                  | 08 = Record Ready             |    |
|                    |                  | 09 = Pause                    |    |
|                    |                  | 0A = Eject                    |    |
|                    |                  | 0B = Chase                    |    |
|                    |                  | 0C = InList Reset             |    |
| 10                 | Set color        | 00 = Black / None             | W  |
|                    |                  | 01 = Red                      |    |
|                    |                  | 04 = Green                    |    |
|                    |                  | 05 = Yellow                   |    |
|                    |                  | 10 = Blue                     |    |
|                    |                  | 11 = Purple                   |    |
|                    |                  | 14 = Cyan                     |    |
|                    |                  | 7F = White                    |    |
| 10                 | Set toggle color | 00 = Black / None             | W  |
|                    |                  | 01 = Red                      |    |
|                    |                  | 04 = Green                    |    |
|                    |                  | 05 = Yellow                   |    |
|                    |                  | 10 = Blue                     |    |
|                    |                  | 11 = Purple                   |    |
|                    |                  | 14 = Cyan                     |    |
|                    |                  | 7F = White                    |    |



### Global settings

FIXME: also modulation ? (slider)

| **bb** = ID | Designation                    |
|-------------|--------------------------------|
| 40          | Special ID for global settings |

| **pp** = Parameter | Description         | Possible values  | RW |
|--------------------|---------------------|------------------|----|
| 02                 | Modulation channel  | ?                | RW |
| 06                 | Keyboard channel    | ?                | RW |
| 19                 | Key velocity curve  | 00 = Linear      | RW |
|                    |                     | 01 = Logarithmic |    |
|                    |                     | 02 = Exponential |    |
|                    |                     | 03 = Full (?)    |    |
| 1A                 | Pad velocity curve  | 00 = Linear      | RW |
|                    |                     | 01 = Logarithmic |    |
|                    |                     | 02 = Exponential |    |
|                    |                     | 03 = Full (?)    |    |
| 1B                 | Knob acceleration   | 00 = Slow        | RW |
|                    |                     | 01 = Medium      |    |
|                    |                     | 02 = Fast        |    |
| 1D (new in 1.1.2)  | Octave button blink | 00 = OFF         | RW |
|                    |                     | 7F = ON          |    |
| 1E (new in 1.1.2)  | Pad off backlight   | 00 = OFF         | RW |
|                    |                     | 7F = ON          |    |
