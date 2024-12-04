---
title: How to use a Rubberducky
published: 2022-09-18
description: It's cool, but it's mostly for trolling your friends.
tags: [Security, Gadgets]
category: Gadgets
draft: false
---
There are a bunch of possible uses for this little device. From a little bit of trolling or a complete Windows installation, to stole data or infect a computer with a ransomware.

I don't care what specific use case you are in, but you have to learn at least how to write a payload, and how to use that payload with your Rubberducky.

Firstly, you have to download the duck encoder [here](https://github.com/hak5darren/USB-Rubber-Ducky/wiki/Downloads).

Then, to use it, you have to install java on your machine. The instructions for use are as follows (Credits to [https://blog.hartleybrody.com/rubber-ducky-guide/](https://blog.hartleybrody.com/rubber-ducky-guide/)):

```bash
 ~ java -jar ~/Downloads/duckencoder.jar

Hak5 Duck Encoder 2.6.3

Usage: duckencode -i [file ..]      encode specified file
   or: duckencode -i [file ..] -o [file ..] encode to specified file

Arguments:
   -i [file ..]     Input File
   -o [file ..]     Output File
   -l [file ..]     Keyboard Layout (us/fr/pt or a path to a properties file)

Script Commands:
   ALT [key name] (ex: ALT F4, ALT SPACE)
   CTRL | CONTROL [key name] (ex: CTRL ESC)
   CTRL-ALT [key name] (ex: CTRL-ALT DEL)
   CTRL-SHIFT [key name] (ex: CTRL-SHIFT ESC)
   DEFAULT_DELAY | DEFAULTDELAY [Time in millisecond * 10] (change the delay between each command)
   DELAY [Time in millisecond * 10] (used to overide temporary the default delay)
   GUI | WINDOWS [key name] (ex: GUI r, GUI l)
   REM [anything] (used to comment your code, no obligation :) )
   ALT-SHIFT (swap language)
   SHIFT [key name] (ex: SHIFT DEL)
   STRING [any character of your layout]
   REPEAT [Number] (Repeat last instruction N times)
   [key name] (anything in the keyboard.properties)
```

Secondly, you have to learn how to write a payload. [Here](https://github.com/hak5darren/USB-Rubber-Ducky/wiki/Duckyscript) is the syntax.

Ok, that's it. With those two links I learned how to use the RubberDucky, the rest is up your imagination...

Oh, ok, you don't want to do all by yourself... well, [here](https://github.com/hak5darren/USB-Rubber-Ducky/wiki/Payloads) is a fistful of useful payloads.

[Here](https://github.com/hak5darren/USB-Rubber-Ducky/wiki/Dransomware:-A--ransomware-which-will-encrypt-user's-data-without-root-privileges-in-30-sec's.) is a payload that uses a ransomware to encrypt user's data without root privileges.

[Here](https://null-byte.wonderhowto.com/how-to/use-usb-rubber-ducky-disable-antivirus-software-install-ransomware-0180418/) is a payload that disables antivirus and installs a ransomware.

[Here](https://github.com/hak5/usbrubberducky-payloads/tree/master/payloads/library/credentials/-RD-Credz-Plz) is a payload to steal Microsoft credentials.

[Here](https://github.com/hak5/usbrubberducky-payloads/blob/master/payloads/library/prank/Physical_Rick_Roll/payload.txt) is a payload to add a physical copy of "Never gonna give you up" to the victim Amazon Cart.

Those are my favourite ones. Enjoy!