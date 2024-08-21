# GAOMON M106K Pro in LINUX

## Overview

Configuring the GAOMON M106K Pro graphics tablet in Linux MINT / DEBIAN / UBUNTU / MX / openSUSE

Out of the box the device works immediately, it recognises the stylus position, but is detected by the system as some variant of the touchpad in the SETUP/Mouse and Touchpad menu.
The stylus pressing force is displayed correctly in the Krita graphical editor, but the function keys don't work.


## Pre-requisites

Check the device is connected using the lsusb command, and you should see something like this, observe the vendor ID of 0256c:
```
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 028: ID 256c:006d M106k Pro digitising tablet
```

A different variant of the tablet was seen with the 256c:006e by @speculatrix


See the section at the end if you need to modify the usb.ids database if the device


## Fully configure the system to use the tablet

1. create a file called **/usr/share/X11/xorg.conf.d/71-gaomon.conf** (in openSUSE it would be **/etc/X11/xorg.conf.d/7-gaomon.conf**). For older versions of LM - 51-gaomon.conf. This software does not work with buttons. You can update the wacom driver from the Wacom page on github: https://github.com/linuxwacom/input-wacom/wiki/Installing-input-wacom-from-source ), added to the end of file:
```
#Gaomon M106K_Pro
Section "InputClass"
	Identifier "GAOMON Gaomon Tablet"
	MatchUSBID "256c:006d"
	# comment above and uncomment below as required
	#MatchUSBID "256c:006e"
	MatchDevicePath "/dev/input/event*"
	Driver "wacom"
EndSection
```

Then reboot to pick up the new Xorg setting.


2. After rebooting, the **xsetwacom --list** command should output the following:
```
GAOMON Gaomon Tablet Pen stylus id: XX type: STYLUS
GAOMON Gaomon Tablet Pad pad id: XX type: PAD
```
Observe that the SETUP/MUSIC and TACHPAD menus now have two GAOMON items instead of one.

The M106Pro shows this:
```
HID 256c:006e Touch Strip pad   	id: 24	type: PAD
HID 256c:006e Pad pad           	id: 25	type: PAD
HID 256c:006e Dial pad          	id: 26	type: PAD
HID 256c:006e Pen stylus        	id: 27	type: STYLUS
```


3. the buttons can now be reassigned using [xsetwacom](https://linux.die.net/man/1/xsetwacom):

**For stylus:**
```
xsetwacom --set 'GAOMON Gaomon Tablet Pen stylus' Button 1 "***"  - is the stylus pen response, best not to change, otherwise the stylus will stop working
xsetwacom --set 'GAOMON Gaomon Tablet Pen stylus' Button 2 "***" - bottom stylus button
xsetwacom --set 'GAOMON Gaomon Tablet Pen stylus' Button 3 "***" - top stylus button
```
where `***` is combination of the `"key"` word and pressed keys. In the case of function keys (`CTRL`, `ALT`, `SHIFT`, etc.), you will first write the `+` symbol, then the name of the function key (this symbolises the key being pressed), then the desired key in combination with the function key, then the `-` symbol, then the function key (this symbolises the key being released). For example, `[CTRL] V`, will be written as `"key +ctrl v -ctrl"`. To press and release `Enter` use: `key 0xff0d`.

**For four tablet keys:**
```
xsetwacom --set 'GAOMON Gaomon Tablet Pad pad' Button 1 "***"
xsetwacom --set 'GAOMON Gaomon Tablet Pad pad' Button 2 "***"
xsetwacom --set 'GAOMON Gaomon Tablet Pad pad' Button 3 "***"
xsetwacom --set 'GAOMON Gaomon Tablet Pad pad' Button 8 "***"
xsetwacom --set 'GAOMON Gaomon Tablet Pad pad' Button 9 "***"
xsetwacom --set 'GAOMON Gaomon Tablet Pad pad' Button 10 "***"
xsetwacom --set 'GAOMON Gaomon Tablet Pad pad' Button 11 "***"
xsetwacom --set 'GAOMON Gaomon Tablet Pad pad' Button 12 "***"
xsetwacom --set 'GAOMON Gaomon Tablet Pad pad' Button 13 "***"
xsetwacom --set 'GAOMON Gaomon Tablet Pad pad' Button 14 "***"
xsetwacom --set 'GAOMON Gaomon Tablet Pad pad' Button 15 "***"
xsetwacom --set 'GAOMON Gaomon Tablet Pad pad' Button 16 "***"
```
where `***` is the combination described above.

 ![CH341A programmer device](https://github.com/bigbigmdm/GAOMON-M106K-PRO-in-LINUX/raw/main/gaomon_m106k_pro.png)

From these commands you can create a BASH file and run it before using the tablet:
```
#!/bin/bash
# TABLET KEY SETTINGS GAOMON M106K
xsetwacom --set 'GAOMON Gaomon Tablet Pad pad' Button 1 "key +ctrl - -ctrl"
xsetwacom --set 'GAOMON Gaomon Tablet Pad pad' Button 2 "key +ctrl + -ctrl"
xsetwacom --set 'GAOMON Gaomon Tablet Pad pad' Button 3 "key +ctrl c -ctrl"
xsetwacom --set 'GAOMON Gaomon Tablet Pad pad' Button 8 "key +ctrl v -ctrl"
xsetwacom --set 'GAOMON Gaomon Tablet Pad pad' Button 9 "key +ctrl z -ctrl"
xsetwacom --set 'GAOMON Gaomon Tablet Pad pad' Button 10 "key +ctrl +shift z -shift -ctrl"
xsetwacom --set 'GAOMON Gaomon Tablet Pad pad' Button 11 "key +ctrl e -ctrl"
xsetwacom --set 'GAOMON Gaomon Tablet Pad pad' Button 12 "key +ctrl +shift e -shift -ctrl"
# GAOMON M106K PEN KEY SETTINGS
xsetwacom --set 'GAOMON Gaomon Tablet Pen stylus' Button 2 "key del"
```
- file that reassigns the lower pen key to Del (for easy erasing of selected fragment), four tablet keys for copy, paste, reduce scaling, enlarge scaling, undo, redo, merging layers functions (for Krita).

A list of supported keys can be seen with the command **`xsetwacom --list modifiers`**

**More examples**

```
# press a, press shift, press and release b, release shift, release a, then press and release enter
xsetwacom --set 'GAOMON Gaomon Tablet Pad pad' Button 1 "key +a +shift b -shift -a 0xff0d"

# right click of mouse (read more: https://askubuntu.com/questions/729564/wacom-button-and-mouse-button-setting)
xsetwacom --set 'GAOMON Gaomon Tablet Pen stylus' Button 2 3
```
See also https://github.com/bigbigmdm/GAOMON-S620-in-LINUX


## Fixing the USB ID database

If "lsusb" doesn't show the device name, you will need to add it to the device identifier file. Become root, make a backup copy, and edit the file, then diff the result to make sure it's not screwed up accidentally:
```
$ sudo -i
# cp -p /usr/share/hwdata/usb.ids /tmp
# vi /usr/share/hwdata/usb.ids
```

In the editor, find the lines starting 256b and then add this afterwards:
```
256c  GAOMON Technology Co ltd
	006d  M106k Pro digitising tablet
	006e  M106k Pro digitising tablet
```

