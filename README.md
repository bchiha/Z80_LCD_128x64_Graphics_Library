# Z80 LCD 128x64 Graphics Library <!-- omit in toc -->
__Full native Z80 Graphics Library for the 128x64 Pixel LCD Screen__

<img src="img\LCD_example.png" width="700">

The above image demonstrates most of the graphical routines.  This includes line drawing, circles, filled shapes, text and pixel drawing.

---

- [Files in this repository](#files-in-this-repository)
- [LCD Screens](#lcd-screens)
- [Example Code](#example-code)
- [Programming Guide](#programming-guide)
  - [INIT\_LCD](#init_lcd)
  - [CLEAR\_GBUF](#clear_gbuf)
  - [CLEAR\_GR\_LCD](#clear_gr_lcd)
  - [CLEAR\_TXT\_LCD](#clear_txt_lcd)
  - [SET\_GR\_MODE](#set_gr_mode)
  - [SET\_TXT\_MODE](#set_txt_mode)
  - [DRAW\_BOX](#draw_box)
  - [DRAW\_LINE](#draw_line)
  - [DRAW\_CIRCLE](#draw_circle)
  - [DRAW\_PIXEL](#draw_pixel)
  - [FILL\_BOX](#fill_box)
  - [FILL\_CIRCLE](#fill_circle)
  - [PLOT\_TO\_LCD](#plot_to_lcd)
  - [PRINT\_STRING](#print_string)
  - [PRINT\_CHARS](#print_chars)
  - [DELAY\_US](#delay_us)
  - [DELAY\_MS:](#delay_ms)
  - [SET\_BUF\_CLEAR](#set_buf_clear)
  - [SET\_BUF\_NO\_CLEAR](#set_buf_no_clear)
  - [CLEAR\_PIXEL](#clear_pixel)
  - [FLIP\_PIXEL](#flip_pixel)
  - [LCD\_INST](#lcd_inst)
  - [LCD\_DATA](#lcd_data)
  - [SER\_SYNC](#ser_sync)
- [Future Work](#future-work)

## Files in this repository
  * lcd_128x64_glib.z80 - Z80 Graphics library for the ST7920 Controller
  * lcd_3d_demo.z80 - 3D Frame rotation program
  * lcd_mad_program.z80 - MAD Magazine face drawing
  * lcd_maze_gen.z80 - Maze Generation program
  * ST7920.pdf - ST7920 Datasheet
  * QC12864B.pdf - LCD Screen Datasheet
  * 3D_Fundamentals.JPG - Amstrad Basic 3D Frame article
  * [add_on_PCB](./add_on_PCB) directory - Add-on Board for the TEC-1 Computer

## LCD Screens
There are a few variants of these LCD screens, but all typically use the [ST7920](./ST7920.pdf) LCD Controller.  The LCD Screen that I used is the [QC12864B](./QC12864B.pdf).  This screen has two ST7921 Panels (128 x 32) stacked one above the other.  Other LCD boards might not do this.  If so, the [PLOT\_TO\_LCD](#plot_to_lcd) function will need to be modified. (future work)

<img src="img\QC12864B_front.png" style="float: left;" width="400">
<img src="img\QC12864B_back.png" width="400">
These screens have Graphics Display RAM (GDRAM) and Display Data RAM (DDRAM) areas.  GDRAM is for drawing pixels and DDRAM is for displaying text or characters.  Both RAM areas __can be__ displayed at the same time.

The Pinout and connection to the Z80 for the QC12864B board is as follows:

| Pin | Name | Description | Serial<sup>1</sup> | Parallel |
| --- | ---- | ----------- | ------ | -------- |
| 1 | VSS | Ground | GND | GND |
| 2 | VDD | Power | 5v | 5v |
| 3 | V0 | Contrast | N/A | N/A |
| 4 | D/I | IR/DR (CS) | 5v | A7<sup>4</sup> |
| 5 | R/W | R/W (SID) | D0 | RD (inverted)<sup>2</sup> |
| 6 | E | Enable (SCLK) | D1 | Port 7 <sup>4</sup> (inverted)<sup>2</sup> |
| 7 | DB0 | Data | N/A | D0 |
| 8 | DB1 | Data | N/A | D1 |
| 9 | DB2 | Data | N/A | D2 |
| 10 | DB3 | Data | N/A | D3 |
| 11 | DB4 | Data | N/A | D4 |
| 12 | DB5 | Data | N/A | D5 |
| 13 | DB6 | Data | N/A | D6 |
| 14 | DB7 | Data | N/A | D7 |
| 15 | PSB | Serial/Para | GND | 5v |
| 16 | NC |  |  |
| 17 | RST | Reset | RST | RST |
| 18 | VEE | LCD Drive | N/A | N/A |
| 19 | A | Backlight | 5v/NC | 5v/NC |
| 20 | K | Backlight | GND/NC | GND/NC |

__<sup>1</sup>__ Three communication lines are need for Serial SPI transfer.  Pin 4 is Chip Select (CS).  As there is only one peripheral connected, this can just be tied high at 5v.  Pin 5 is Serial Input Data (SID) and Pin 6 is Serial Clock (SCLK).  Pin 5 and 6 are to be latched via a 74HC74 Dual D-Flip Flop.  The inputs to the latch are D0 (SID) and D1 (SCLK).  Have a look at the schematic below.

__<sup>2</sup>__ A CMOS CD4049 Inverter Buffer is needed to invert the input

__<sup>3</sup>__ Some 128x64 LCD Screens have different Pinouts than in the table above.

__<sup>4</sup>__ I use `Port 7` and `A7` to connect the LCD screen to the TEC computer.  You will need to modify these two constants if connecting the LCD in a different way.  See the header part of the [lcd_128x64_glib.z80](lcd_128x64_glib.z80) file to modify these values.

<img src="img\LCD_connection.png" style="float: left;" width="400">
<img src="img\LCD_sconnection.png" width="400">

## Example Code

To see the library working, have a look at the example code provided.  The [LDC\_3D\_demo.z80](lcd_3d_demo.z80) has a `DISPLAY_MENU` label that displays the above picture.  Look at this piece of code for better understanding.

Also, see the library and these files in action on [YouTube](https://youtu.be/xADbvfVs6Mg) and [YouTube](https://youtu.be/tZHt6CwbkVA)!

Thanks to [PCBWay](https://www.pcbway.com) for sponsoring this video.  ** For $5 off your first order at PCBWay, click [here](https://www.pcbway.com/setinvite.aspx?inviteid=599966) **.  PCBWay Printed Circuit Boards the Easy Way.

## Programming Guide

The Graphics Library has routines that interact with the LCD Screen.  Routines are called via a __Jumpblock__.  These routines are at the start of the Graphics Library.  By default the library is placed at location `0x3000`.  If this location is changed, the start locations of the functions are also to be changed.

For example, to initalise the LCD, do something like this:
```
    CALL 3000H
```
Or to make it more readable create a label and do this:
```
    G_INIT_LCD:	EQU 3000H
    CALL G_INIT_LCD
```

In summary, here are the routines and their default address locations:
| Address | Routine | Description |
| ------- | ------- | ----------- |
| 3000H | [INIT\_LCD](#init_lcd) | Initalise the LCD |
| 3003H | [CLEAR\_GBUF](#clear_gbuf) | Clear the Graphics Buffer |
| 3006H | [CLEAR\_GR\_LCD](#clear_gr_lcd) | Clear the Graphics LCD Screen |
| 3009H | [CLEAR\_TXT\_LCD](#clear_txt_lcd) | Clear the Text LCD Screen |
| 300CH | [SET\_GR\_MODE](#set_gr_mode) | Set Graphics Mode |
| 300FH | [SET\_TXT\_MODE](#set_txt_mode) | Set Text Mode |
| 3012H | [DRAW\_BOX](#draw_box) | Draw a rectangle between two points |
| 3015H | [DRAW\_LINE](#draw_line) | Draw a line between two points |
| 3018H | [DRAW\_CIRCLE](#draw_circle) | Draw a circle from Mid X,Y to Radius |
| 301BH | [DRAW\_PIXEL](#draw_pixel) | Draw one pixel at X,Y |
| 301EH | [FILL\_BOX](#fill_box) | Draw a filled rectangle between two points |
| 3021H | [FILL\_CIRCLE](#fill_circle) | Draw a filled circle from Mid X,Y to Radius |
| 3024H | [PLOT\_TO\_LCD](#plot_to_lcd) | Display the Graphics Buffer to the LCD Screen |
| 3027H | [PRINT\_STRING](#print_string) | Print Text on the screen in a given row |
| 302AH | [PRINT\_CHARS](#print_chars) | Print Characters on the screen in a given row and column |
| 302DH | [DELAY\_US](#delay_us) | Microsecond delay for LCD updates |
| 3030H | [DELAY\_MS](#delay_ms) | Millisecond delay for LCD updates |
| 3033H | [SET\_BUF\_CLEAR](#set_buf_clear) | Clear the Graphics buffer on after Plotting to the screen |
| 3036H | [SET\_BUF\_NO\_CLEAR](#set_buf_no_clear) | Retain the Graphics buffer on after Plotting to the screen |
| 3039H | [CLEAR\_PIXEL](#clear_pixel) | Clear one pixel at X,Y |
| 303CH | [FLIP\_PIXEL](#flip_pixel) | Invert one pixel at X,Y |
| 303FH | [LCD\_INST](#lcd_inst) | Send a parallel or serial instruction to LCD |
| 3042H | [LCD\_DATA](#lcd_data) | Send a parallel or serial data to LCD |
| 3045H | [SER\_SYNC](#ser_sync) | Send serial synchronize byte to LCD |

Here is the detailed routine descriptions with examples

### INIT_LCD

Initalise the LCD Screen.  Needed to be called before any other routine

- Entry: No conditions
- Exit: All registers corrupt

```
    CALL INIT_LCD       ;Initalise the LCD Screen
```

### CLEAR_GBUF

Clear the Graphics Buffer.  The Graphics Buffer or GBUF is the internal memory area that contains pixel data for the LCD.  The drawing routines write to the GBUF.  Once all pixels are set, this buffer is then plotted to the LCD with the [PLOT\_TO\_LCD](#plot_to_lcd) Routine.  Clearing the GBUF is a good way to ensure the pixel area is empty.

- Entry: No conditions
- Exit: All registers corrupt

```
    CALL CLEAR_GBUF     ;Clears the Graphics Buffer
```

### CLEAR_GR_LCD

Clear the Graphics LCD Screen.  This routine clears the GDRAM or Graphics screen on the LCD.

- Entry: No conditions
- Exit: All registers corrupt

```
    CALL CLEAR_GR_LCD     ;Clears the Graphics LCD Screen
```

### CLEAR_TXT_LCD

Clear the Text LCD Screen.  This routine clears the DDRAM or Text screen on the LCD.

- Entry: No conditions
- Exit: All registers corrupt

```
    CALL CLEAR_TXT_LCD    ;Clear the Text LCD Screen
```

### SET_GR_MODE

Set the LCD to Graphics Mode.  This routine puts the LCD in Graphics mode (Extended Instructions) and any further instructions to the LCD will be for the graphics screen.  It only needs to be called once if multiple graphics routines are used.

- Entry: No conditions
- Exit: AF, DE corrupt
  
```
    CALL SET_GR_MODE      ;Set Graphics Mode
```

### SET_TXT_MODE

Set the LCD to Text Mode.  This routine puts the LCD in Text mode (Basic Instructions) and any further instructions to the LCD will be for the text screen.  It only needs to be called once if multiple text routines are used.

- Entry: No conditions
- Exit: AF, DE corrupt

```
    CALL SET_TXT_MODE     ;Set Text Mode
```

### DRAW_BOX

Draws a single-line box between two points X1,Y1 and X2,Y2.

- Entry:
  - B = X1-coordinate (0-127)
  - C = Y1-coordinate (0-63)
  - D = X2-coordinate (0-127)
  - E = Y2-coordinate (0-63)
- Exit: AF, HL corrupt

```
    LD BC, 0020H 	;X0, Y0
    LD DE, 7F3FH 	;X1, Y1
    CALL DRAW_BOX ;Draw a outline box from X0,Y0 to X1,Y1
```       

### DRAW_LINE

Draws a straight line between X1,Y1 and X2,Y2.  Uses [Bresenham](http://members.chello.at/~easyfilter/bresenham.html) Line drawing algorithm 

- Entry:
  - B = X1-coordinate (0-127)
  - C = Y1-coordinate (0-63)
  - D = X2-coordinate (0-127)
  - E = Y2-coordinate (0-63)
- Exit: All registers corrupt

```
    LD BC, 0010H 	;X0, Y0
    LD DE, 7F30H 	;X1, Y1
    CALL DRAW_LINE
```

### DRAW_CIRCLE

Draws a circle from a mid point to a radius.  Uses [Bresenham](http://members.chello.at/~easyfilter/bresenham.html) Circle drawing algorithm 

- Entry:
  - B = Mid-X-coordinate (0-127)
  - C = Mid-Y-coordinate (0-63)
  - E = Radius (1-63)
- Exit: All registers corrupt

```
    LD BC, 0818H 	;Mid X, Mid Y
    LD E, 08H 		;Radius
    CALL DRAW_CIRCLE
```

### DRAW_PIXEL

Draws a single Pixel.

- Entry:
  - B = X-coordinate (0-127)
  - C = Y-coordinate (0-63)
- Exit: AF, HL corrupt

```
    LD BC, 4020H 	;X,Y
    CALL DRAW_PIXEL
```

### FILL_BOX

Draws a filled box between X1,Y1 and X2,Y2.

- Entry:
  - B = X1-coordinate (0-127)
  - C = Y1-coordinate (0-63)
  - D = X2-coordinate (0-127)
  - E = Y2-coordinate (0-63)
- Exit: AF, HL corrupt

```
    LD BC, 0020H 	;X0, Y0
    LD DE, 7F3FH 	;X1, Y1
    CALL FILL_BOX ;Draw a filled box from X0,Y0 to X1,Y1
```       

### FILL_CIRCLE

Draws a filled circle from a mid point to a radius.  This routine iteratively calls the [DRAW\_CIRCLE](#draw_circle) routine increasing the radius until is equals the register E.  There might be gaps in the filled circle, but hey it looks just like what you get on a BASIC program.

- Entry:
  - B = Mid-X-coordinate (0-127)
  - C = Mid-Y-coordinate (0-63)
  - E = Radius (0-63)
- Exit: All registers corrupt

```
    LD BC, 1018H 	;Mid X, Mid Y
    LD E, 08H 		;Radius
    CALL FILL_CIRCLE
```

### PLOT_TO_LCD

This routine draws the Graphics Buffer or GBUF to the Graphics LDC screen.  It is usually called after one of the drawing routines is called.

- Entry: No conditions
- Exit: All registers corrupt

```
    CALL PLOT_TO_LCD     ;Display the Graphics Buffer to the LCD Screen
```

### PRINT_STRING

Prints ASCII text on the screen on a given row.   There are 4 text rows on the LCD screen.  The text to be displayed is to be defined directly after the CALL routine and is to be terminated with a zero.  

Here are the 128 characters that are available.  Conveniently, Alphanumeric characters align with the ASCII table.
<img src="img\character_map.png" width="700">

- Entry:
  - A = row number (0-3)
  - Text = "String" on the next line, terminate with 0
- Exit: All registers corrupt

```
    LD A,2
    CALL PRINT_STRING
    DB 02H, " This Text ", 1BH ,00H

    ; This will display a smiley face, "This Text" 
    ; and a left arrow on row 2 of the LCD screen.
```

### PRINT_CHARS

Print Characters on the screen in a given row and column.  This routine is similar to the one above but character row __and__ column placement can be made.  Characters to be printed are to be terminated with a zero. 

__Note:__ Even though there are 16 columns, only every second column can be written to and two characters are to be printed.  IE: if one character is to be printed in column 2, then set B=0 and print " x", putting a space before the character.

- Entry:
  - B = column (0-7)
  - C = row (0-3)
  - HL = start address of text data
- Exit: All registers corrupt.  HL will be at the end of the text data table.

```
    LD HL, MENU_DATA ;Load HL with Menu data table
    LD BC, 0102H 	;Column 1, Row 2
    CALL PRINT_CHARS ;Display Text
    ...
    MENU_DATA:
    DB "Hello!",0
```

### DELAY_US

Delay loop for LCD to complete its instruction.  Every time a command is sent to the LDC, it requires a small amount of time to complete that operation.  IE: setting extended instruction mode.  Time needed for most operations is defined in the LDC specification.  It is usually around 72us.  This routine is used internally, but can also be used here.  The delay time depends on how fast the CPU is running at.  A variable `V_DELAY_US` can be changed to suit your LDC setup.  I found that my LDC can handle a delay of `0004H`.  

This routine will delay the cpu by `V_DELAY_US` and can be used if outputting instructions to the LDC directly.

- Entry: No conditions
- Exit: DE, AF corrupt

```
    LD A, 02H       ;Home command
    OUT (LCD_IR), A ;Move the cursor to the top of LCD
    CALL DELAY_US   ;Delay the CPU by V_DELAY_US
```

### DELAY_MS:

This is the same as the above routine, but the delay can be software controlled.

- Entry: DE = delay value
- Exit: DE, AF corrupt

```
    LD A, 01H       ;Clear command
    OUT (LCD_IR), A ;Clear the LCD screen
    LD DE, 0050H    ;Longer Delay
    CALL DELAY_MS   ;Delay the CPU by DE
```

### SET_BUF_CLEAR

On every [PLOT\_TO\_LCD](#plot_to_lcd), Clear the graphics buffer GBUF.  Calling this routine will clear the graphics buffer on every draw to LCD.  This is useful if doing animation that require a new drawing to be display on every plot or frame.

- Entry: No conditions
- Exit: AF corrupt

```
    CALL SET_BUF_CLEAR    ;Clear GBUF on plot
```

### SET_BUF_NO_CLEAR

Do not clear the graphics buffer on every [PLOT\_TO\_LCD](#plot_to_lcd).  Calling this routine will not clear the graphics buffer on every draw to LCD.  This is useful adding graphics data to an existing drawing.  This is used in the [MAD](lcd_128x64_mad.z80) example, where lines are being continually drawn to the screen until the whole picture is complete.

- Entry: No conditions
- Exit: AF corrupt

```
    CALL SET_BUF_NO_CLEAR    ;Do not clear GBUF on plot
```

### CLEAR_PIXEL

Removes or clears a single Pixel.

- Entry:
  - B = X-coordinate (0-127)
  - C = Y-coordinate (0-63)
- Exit: AF, HL corrupt

```
    LD BC, 4020H 	;X,Y
    CALL CLEAR_PIXEL
```

### FLIP_PIXEL

Inverts a single Pixel.  If the Pixel is on, it will turn off and if the Pixel is off, it will turn on.

- Entry:
  - B = X-coordinate (0-127)
  - C = Y-coordinate (0-63)
- Exit: AF, HL corrupt

```
    LD BC, 4020H 	;X,Y
    CALL FLIP_PIXEL
```

### LCD_INST

Send an instruction to the LCD.  This routine will send the value of Register `A` to the Instruction Register of the LCD Screen.  The routine is universal regardless of Serial or Parallel connection.
For Serial connection, this will also send the Synchronization Byte.

- Entry:
  - A = Value to send
- Exit: AF, DE (Parallel only) corrupt

```
    LD A, 38H   ; Clear Screen 
    CALL LCD_INST
```

### LCD_DATA

Send data to the LCD.  This routine will send the value of Register `A` to the Data Register of the LCD Screen.  The routine is universal regardless of Serial or Parallel connection.  
__For Serial connection__, a call to [SER\_SYNC](#ser_sync) is to be done prior.  This is because only one Synchronization Byte is needed for multiple (up to 256) Data bytes.

- Entry:
  - A = Value to send
- Exit: AF, DE (Parallel only) corrupt

```
    ;Parallel example
    LD A, "B"   ; The letter B
    CALL LCD_DATA
```

### SER_SYNC

Send a Serial Synchronization Byte to the LCD.  This routine is only needed for Serial connection __AND__ prior to an [LCD\_DATA](#lcd_data) call.  The Register `A` must be `02H`

- Entry:
  - A = 02H 
- Exit: AF corrupt

```
    LD A, 02H 
    CALL SER_SYNC   ;Data Block Sync
    LD A,  "T"      ;Letter T
    CALL LCD_DATA
    LD A,  "E"      ;Letter E
    CALL LCD_DATA
    LD A,  "C"      ;Letter C
    CALL LCD_DATA
```

## Future Work

Displaying Text on this screen is clumsy as text can only be placed at defined locations.  I plan to make a graphics font of 128 characters that is smaller in size, possibly 5x7 and can be placed anywhere on the screen.

Some LCD screens have a slightly different way data is written to them in terms of how the screens are laid out.  The [QC12864B](./QC12864B.pdf) as two ST9721 screens placed on top of each other, but are connected in series.  Plotting to the LCD routine will need to be changed for other LCD screen layouts.

