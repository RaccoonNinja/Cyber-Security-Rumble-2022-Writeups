## Skills required: Research

A simple challenge but I'm bad at hardware.

## Solution:

I received an .sr file for the challenge.
While searching online what file type .sr can be didn't reveal much information, I noticed that the file can be understood as a zip archive.
There are a couple of ways to do it: you can analyze the header or in Ubuntu it renders with an archive shape and opens just like an ordinary zip.

The metadata file inside revealed the software to be used:
```
[global]
sigrok version=0.5.2
```

After basic research, [Sigrok](https://sigrok.org/wiki/Downloads) offers both a GUI software called PulseView as well as a CUI software sigrok-cli.
I used the GUI version.

![image](https://user-images.githubusercontent.com/114584910/195635683-68154ec3-fd71-4ec6-bb99-82581af5a18e.png)

While the GUI offered many decoders, I didn't know which one to use.
Looking up `what serial bus signal uses a clock` gave me [this resource](https://learn.sparkfun.com/tutorials/serial-communication/all).
Upon trying the I2C decoder (24xx EEPROM):

![sr](https://user-images.githubusercontent.com/114584910/195638902-3ada5373-8bbd-4a1f-8a73-e18f402659d8.png)

I then copied the hex and worked in CyberChef.
