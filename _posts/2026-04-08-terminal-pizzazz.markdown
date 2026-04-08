---
layout: post
title: "Terminal Pizzazz"
date: 26-04-08 13:40:08 +0100
categories: tui
---

Since my first interaction with a terminal, I have always been interested in the
deeper magic that happens under the hood. Terminals have evolved quite a lot
since the MS-DOS days. Terminals historically were physical devices that used
text based communication. Like everything in the networking all devices
need a set of rules or protocols to be able to understand each other. Today
we are only going to focus on a small subset of these rules called
[ANSI Escape Codes](https://en.wikipedia.org/wiki/ANSI_escape_code#Control_Sequence_Introducer_commands).

ANSI escape codes give us a way to control a terminal using the same
communication lines. Lets look at some examples using the best programming
language C.

## ASCII Control Characters

The ASCII standard defines a set of specific byte codes that define particular
functionality such as the widely known `'\n'` (newline) and `'\r'`
(carriage return). The ASCII standard also defines additional control
Characters like `'\a'` (bell), `'\e'` (escape), `'\t'` (tab), `'\b'` (backspace).
The ASCII `'\e'` control character is really special because it opens the door
to developing TUI'S (Terminal User Interface's). The escape character allows
for text colouring, font-styling, cursor position reading + repositioning and
so much more.

## CSI - Control Sequence Introducer

There are a few different types of control sequences defined by the ANSI
standard although to stay on the topic of pizzazz we will only focus on the
control sequence introducer `\e**[**`. This will serve as the base sequence
of characters for all our following commands so its worth remembering. In
some documentation they refer to this as CSI.

## SGR - Select Graphic Rendition

Select Graphic Rendition (SGR) allows us to control the way the characters and
their background are rendered on the terminal.
***Depending on the terminal
emulator used on your device some of content may not render due to compatibility
issues.***
SGR is typically written in the format `CSI n m`. n is the specific
control command that tells the terminal the format we want to use on the text
that proceeds it. All text colour commands have n with the base number 30.
Lets look at the command to set the text to red, first we write our CSI `'\e['`,
when n is 31 this tells the terminal the text colour should be red so the final
SGR format command comes out to be `'\e[31m'`. Lets put this to use, get out your
terminal and copy the following c code, compile it and run to see the pizzazz:

```
/*
 * Purpose:
 * C program to show ANSI escape sequence codes to print text as red.
 * Programmer: Matas Noreika 26/04/08 14:23:38
*/

//libc input/output API
#include <stdio.h>

int main(int argc, argv** argv){ // start of main
  printf("\e[31mHello World!\n");
  return 0;
} // end of main
```

The output should be your Hello World! in red. But there is one problem.
If we continued writing the text we will see all of it is still red.
To fix this we need to reset the SGR and that can be done using `'\e[m'`.
The fixed code is:

```
/*
 * Purpose:
 * C program to show ANSI escape sequence codes to print text as red.
 * Programmer: Matas Noreika 26/04/08 14:23:38
*/

//libc input/output API
#include <stdio.h>

int main(int argc, argv** argv){ // start of main
  printf("\e[31mHello World!\n\e[m");
  return 0;
} // end of main
```

You can try out changing the n value between 30-37 to see the different
colour options. Similarly if you use 40-47 you can see the background colour
of the text change.

The above examples show 1 of three different colour picking modes, for the
examples above it was simple 3-4 bit colour. Most terminals support 8-bit and
even less support 24-bit colour. I will leave this topic for you to explore
as there is more options I wish to discuss.

## Cursor Control

One important feature of the CSI is to enable cursor repositioning and reading.
While cursor repositioning is heavily documented and quite easy to do, reading
requires a bit more work. When I first started out the cursor reading was very
poor described so hopefully this comes in handy to somebody. Cursor reading is
done by using the following sequence `'\e[6n'`. Now the complicated thing with
this command is that the reply is sent to the input stream and to complicate
matters more the stream in in canonical mode meaning you need to press enter or
send an additional `\n`. You can enter non-canonical mode (raw mode) although
each operating system has its own interface to configuring the terminal.
I will have to provide two examples one for windows and another for POSIX
compliant operating systems.

POSIX compilant example:

```
/*
 * Purpose:
 * Program that sets terminal in non-canonical mode and reads the cursor
 * position. This program is POSIX C compliant.
 * Programmer: Matas Noreika 26/04/08 16:00:58
*/

//libc I/O API - printf()
#include <stdio.h>
//POSIX system API - read()
#include <unistd.h>
//POSIX C terminal/Serial I/O API - termios struct and methods
#include <termios.h>

int main(int argc, char** argv){//start of main
  int row, col; // variables to store the row and column data
  //read current terminal settings (used to reset later)
  struct termios originalSettings, rawSettings;
  if(tcgetattr(0, &originalSettings)){
    //exit with error (we are not going to do too much error handling)
    return -1;
  }
  printf("Terminal attributes loaded\n");
  //set new settings (raw mode same as non-canonical)
  rawSettings = originalSettings; // copy the settings over
  rawSettings.c_lflag &= ~ICANON; // disable cannonical processing
  rawSettings.c_lflag &= ~ECHO; // disable input echo
  rawSettings.c_cc[VMIN] = 1; // set minimum byte count to 1
  rawSettings.c_cc[VTIME] = 0; // set minimum time delay 0 * 0.1s
  //set the raw settings
  if(tcsetattr(0, TCSANOW, &rawSettings)){
    //exit with error (we are not going to do too much error handling)
    return -1;
  }
  printf("Terminal set to raw mode\n");
  printf("\e[6n"); // write position request
  printf("Sent position request\n");
  char readBuff[4096] = {0}; // create a read buffer
  // (4906 - sandard buffer size for stream)
  ssize_t bytes = read(0,readBuff,4095);
  if(bytes < 0){
    printf("error while reading\n");
  }else{
    readBuff[0] = '^'; // replace first character to prevent blank string
    printf("%s\n", readBuff); // print the input string
    sscanf(readBuff,"^[%d;%dR",&row, &col); // parse the row and col out of string
    printf("Row: %d, col: %d\n", row, col); // print our obtained data
  }
  //reset back to originalSettings (will repeat until successful)
  while(tcsetattr(0, TCSANOW, &originalSettings));
  return 0;
}//end of main
```

Windows example:

```
/*
 * Purpose:
 * C Program to read cursor position on windows platform.
 * Developed using MSYS2.
 * Programmer: Matas Noreika 26-04-08 7:20:58
*/

#include <windows.h>
#include <stdio.h>

int main(int argc, char** argv){
  //position variables
  int row, col;
  //handle for stdin (reference like file descriptor)
  HANDLE hStdin;
  // variable to hold old and raw stream configuration
  DWORD fdwMode, fdwRawMode;
  //get the handle for stdin
  hStdin = GetStdHandle(STD_INPUT_HANDLE);
  if(hStdin == INVALID_HANDLE_VALUE){
    return -1;
  }
  //get the stream settings
  if(!GetConsoleMode(hStdin, &fdwMode)){
    return -1;
  }
  //Configure raw mode and set it
  fdwRawMode = fdwMode & ~(ENABLE_LINE_INPUT | ENABLE_ECHO_INPUT);
  if(!SetConsoleMode(hStdin, fdwRawMode)){
    return -1;
  }
  //send position request
  printf("\e[6n");
  char readBuff[4096] = {0};
  DWORD bytes;
  if(!ReadFile(hStdin, readBuff, 4095, &bytes, NULL)){
    printf("error while reading\n");
  }else{
    readBuff[0] = '^'; // change first character to prevent blank string
    printf("%s\n", readBuff);
    sscanf(readBuff, "^[%d;%dR", &row, &col);
    printf("Row: %d, Col: %d\n", row, col);
  }
  //reset originalSettings
  SetConsoleMode(hStdin, fdwMode);
  return 0;
}
```

## The End?

If you made it this far congratulations you are slightly more aware of the
possibilities you have to play around with the terminal. This knowledge
can allow you to create a text editor like vim or a terminal game like
snake. In addition you can use the POSIX example above to create your own
custom Serial Monitor like screen to interface with your microcontrollers.
I hope you enjoyed thank you for reading.

***If you encounter any bugs or misinformation please leave an issue on this
[github repository](https://github.com/matas-noreika/matas-noreika.github.io) or contact me by email.***
