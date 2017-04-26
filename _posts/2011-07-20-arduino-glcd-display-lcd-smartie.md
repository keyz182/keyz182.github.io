---
id: 19
title: Arduino GLCD Display + LCD Smartie
date: 2011-07-20T15:26:57+00:00
author: Kieran Evans
layout: post
guid: 19
aktt_notify_twitter:
  - 'no'
categories:
  - Uncategorized
---
A long while ago, I bought myself an Arduino Mega, and it came with a 128x64 Graphic LCD screen. Finally, I've found a use for it. I'm going to make a fron panel display for my desktop. I'd used LCDSmartie several years ago for a small LCD case screen, and liked it. Unfortunately, it doesn't support GLCDs (AFAIK). So, I built an Arduino "wrapper" around it. It uses the Matrix Orbital command set, then translates them to calls to the Arduino GLCD library.

The code is based on code found <a href="http://milesburton.com/LCD_Smartie_Powered_By_Arduino_Liquid_Crystal_library" title="here" target="_blank">here</a>. I modified some of the calls, and got rid of the character LCD code, and put in GLCD calls.

In essence, I create a text area the size of the screen, and throw text at it. The great thing about this is that you can resize the text area, then have the Arduino do something in the rest, while LCD smartie throws it's stuff in the textbox. Doing it this way means no modification at all to LCDSmartie.

I'm going to work on this a bit more as I get time. I'll be replacing the Mega with a barebones Arduino (made from the bag'o atmega chips I have).

Settings for LCD Smartie are:

Display settings > plugin > Display Plugin : select matrix.dll

startup parameters: COM4,9600 (i.e. the com port for the arduino, and the baud rate)

Display settings > screen > display size: 4x20

Oh, whats that? You want code? And Pictures? Very well.

<script type="text/javascript" language="javascript">
  function toggle() {
 	var ele = document.getElementById("codediv");
 	var text = document.getElementById("showhide");
 	if(ele.style.display == "block") {
     		ele.style.display = "none";
 		text.innerHTML = "[show code]";
   	} 	else {
 		ele.style.display = "block";
 		text.innerHTML = "[hide code]";
 	}
 }
</script>

<a id="showhide" href="javascript:toggle();">[show code]</a>
<div id="codediv" style="display: none; overflow: auto; height: 300px;">
<pre name="code" class="cpp">#include "glcd.h"

#include "fonts/allFonts.h"

// these constants won't change.  But you can change the size of
// your LCD using them:
const int numRows = 4;
const int numCols = 18;
gText t1; // will define runtime later

void setup() {
        Serial.begin(9600);
	GLCD.Init(NON_INVERTED);
	t1.DefineArea(textAreaFULL);
	t1.SelectFont(System5x7);
}

byte serial_getch(){
  int incoming;
  while (Serial.available()==0){}
	// read the incoming byte:
  incoming = Serial.read();

  return (byte) (incoming &amp;0xff);
}

void loop()
{
  byte rxbyte;
  byte temp;
  int col = 0;
  int row = 0;

  rxbyte = serial_getch();

  if (rxbyte == 254) //Matrix Orbital uses 254 prefix for commands
  {
    switch (serial_getch())
    {
    case 66: //backlight on (at previously set brightness)
      // not implemented				

      break;
    case 70: //backlight off
      // not implemented
      break;
    case 71:  //set cursor position
      col = (serial_getch() - 1);  //get column byte
      row = (serial_getch() - 1);

      t1.CursorTo(col,row);
      break;
    case 72:  //cursor home (reset display position)
      t1.ClearArea();
      break;
    case 74:  //show underline cursor
      //lcd.command(0b00001110);
      break;
    case 75:  //underline cursor off
    case 84:  //block cursor off
      //lcd.command(0b00001100);
      break;
    case 76:  //move cursor left
      //lcd.command(16);
      break;
    case 77:  //move cursor right
      //lcd.command(20);
      break;
    case 78:  //define custom char
      //lcd.command(64 + (serial_getch() * 8));  //get+set char address
      //for (temp = 7; temp != 0; temp--)
      //{
      //  lcd.print(serial_getch()); //get each pattern byte
      //}
      break;
    case 83:  //show blinking block cursor
      //lcd.command(0b00001111);
      break;
    case 86:  //GPO OFF
      //implement later
      break;
    case 87:  //GPO ON
      /*temp = serial_getch();
       				if (temp == 1)
       				{
       					GPO1 = GPO_ON;
       				}*/
      break;
    case 88:  //clear display, cursor home
      t1.ClearArea();
      //t1.CursorTo(0,0);
      break;
    case 152: //set and remember (doesn't save value, though)
    case 153: //set backlight brightness
      //not implemented
      break;

      //these commands ignored (no parameters)
    case 35: //read serial number
    case 36: //read version number
    case 55: //read module type
    case 59: //exit flow-control mode
    case 65: //auto transmit keypresses
    case 96: //auto-repeat mode off (keypad)
    case 67: //auto line-wrap on
    case 68: //auto line-wrap off
    case 81: //auto scroll on
    case 82: //auto scroll off
    case 104: //init horiz bar graph
    case 109: //init med size digits
    case 115: //init narrow vert bar graph
    case 118: //init wide vert bar graph
      break;
    default:
      //all other commands ignored and parameter byte discarded
      temp = serial_getch();  //dump the command code
      break;
    }
    return;
  } //END OF COMMAND HANDLER

  //change accented char to plain, detect and change descenders
  //NB descenders only work on 5x10 displays. This lookup table works
  //  with my DEM-20845 (Display Elektronik GmbH) LCD using KS0066 chip.
  switch (rxbyte)
  {
    //chars that have direct equivalent in LCD charmap
    /*		case 0x67: //g
     			rxbyte = 0xE7;
     			break;
     		case 0x6A: //j
     			rxbyte = 0xEA;
     			break;
     		case 0x70: //p
     			rxbyte = 0xF0;
     			break;
     		case 0x71: //q
     			rxbyte = 0xF1;
     			break;
     		case 0x79: //y
     			rxbyte = 0xF9;
     			break;
     */  case 0xE4: //ASCII "a" umlaut
    rxbyte = 0xE1;
    break;
  case 0xF1: //ASCII "n" tilde
    rxbyte = 0xEE;
    break;
  case 0xF6: //ASCII "o" umlaut
    rxbyte = 0xEF; //was wrong in v0.86
    break;
  case 0xFC: //ASCII "u" umlaut
    rxbyte = 0xF5;
    break;

    //accented -&gt; plain equivalent
    //and misc symbol translation
  case 0xA3: //sterling (pounds)
    rxbyte = 0xED;
    break;
    /*		case 0xB0: //degrees symbol
     			rxbyte = 0xDF;
     			break;
     */  case 0xB5: //mu
    rxbyte = 0xE4;
    break;
  case 0xC0: //"A" variants
  case 0xC1:
  case 0xC2:
  case 0xC3:
  case 0xC4:
  case 0xC5:
    rxbyte = 0x41;
    break;
  case 0xC8: //"E" variants
  case 0xC9:
  case 0xCA:
  case 0xCB:
    rxbyte = 0x45;
    break;
  case 0xCC: //"I" variants
  case 0xCD:
  case 0xCE:
  case 0xCF:
    rxbyte = 0x49;
    break;
  case 0xD1: //"N" tilde -&gt; plain "N"
    rxbyte = 0x43;
    break;
  case 0xD2: //"O" variants
  case 0xD3:
  case 0xD4:
  case 0xD5:
  case 0xD6:
  case 0xD8:
    rxbyte = 0x4F;
    break;
  case 0xD9: //"U" variants
  case 0xDA:
  case 0xDB:
  case 0xDC:
    rxbyte = 0x55;
    break;
  case 0xDD: //"Y" acute -&gt; "Y"
    rxbyte = 0x59;
    break;
    /*		case 0xDF: //beta  //mucks up LCDSmartie's degree symbol??
     			rxbyte = 0xE2;
     			break;
     */  case 0xE0: //"a" variants except umlaut
  case 0xE1:
  case 0xE2:
  case 0xE3:
  case 0xE5:
    rxbyte = 0x61;
    break;
  case 0xE7: //"c" cedilla -&gt; "c"
    rxbyte = 0x63;
    break;
  case 0xE8: //"e" variants
  case 0xE9:
  case 0xEA:
  case 0xEB:
    rxbyte = 0x65;
    break;
  case 0xEC: //"i" variants
  case 0xED:
  case 0xEE:
  case 0xEF:
    rxbyte = 0x69;
    break;
  case 0xF2: //"o" variants except umlaut
  case 0xF3:
  case 0xF4:
  case 0xF5:
  case 0xF8:
    rxbyte = 0x6F;
    break;
  case 0xF7: //division symbol
    rxbyte = 0xFD;
    break;
  case 0xF9: //"u" variants except umlaut
  case 0xFA:
  case 0xFB:
    rxbyte = 0x75;
    break;
  default:
    break;
  }

  t1.Puts(rxbyte);
  return;
}</pre>
</div>
[flickr-gallery mode="photoset" photoset="72157627240501814"]