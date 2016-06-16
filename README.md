Logitech G710+ Keyboard Driver
==========================

The Logitech G710+ mechanical keyboard is a great piece of hardware. Unfortunately, there is no support in the kernel for the additional features of the keyboard.

This kernel driver allows the keys M1-MR and G1-G6 to be used.


Installation Instructions
-------------------------

First you have to compile the driver:
<pre>
make
</pre>

if the compilation was successfull, you will now have a new kernel module in the directory.

The next step is to install the kernel module:

<pre>
sudo make install
sudo depmod -a
</pre>

At this point the generic driver will still take control. The simple fix for that issue is to copy the 90-logitech-g710-plus.rules file from the misc folder to /etc/udev/rules.d/:

<pre>
sudo cp misc/90-logitech-g710-plus.rules /etc/udev/rules.d/
</pre>

Finally, if you do not receive any events in your environment, it might be necessary to use the modmap provided in the misc folder:

<pre>
xmodmap misc/.Xmodmap
</pre>


Usage
--------------------------
Use the key shortcut utilities provided by your DE to make use of the additional buttons.

Invoke the xmodmap to map the M1, M2, M3, and MR keys to modifier keys (ctrl, shift, etc). This can be done globally or in a bashrc file, etc. The driver will emit a press event for the active M* key when a G# key is pressed. For example:

- M1 pressed
  - emit M1 down
  - emit M1 up
- G1 pressed
  - emit M1 down
  - emit G1 down
  - emit G2 up
  - emit M1 up

The end result is it looks like each G# key is pressed together with a modifier. Eqivalent to ctrl+G1, etc. Key mapping applications should pick the combination which allows for the original G1-6 * 3 = 18 keys instead of just G1-6 + 3 = 9.

```
./xmodmap
```

End result should be similar to https://www.youtube.com/watch?v=uzDpQJYsC1E.

Example of utilizing exposed leds https://gist.github.com/boombatower/ccdab16926c919eee6b2.

API
--------------------------
The driver also exposes a way to set the keyboard backlight intensity. That is done by writing either:

<pre>
/sys/bus/hid/devices/0003:046D:C24D.XXXX/logitech-g710/led_macro
/sys/bus/hid/devices/0003:046D:C24D.XXXX/logitech-g710/led_keys
</pre>

where XXXX is varies (e.g. 0008)

The led_macro file expects a single number which is a bitmask of the first 4 bits. Each bit corresponds to a button. E.g. if you want to light up M1 and M3, you must write the bit pattern 0101, which corresponds to 5 in decimal:

<pre>
echo -n "5" > /sys/bus/hid/devices/0003:046D:C24D.XXXX/logitech-g710/led_macro
</pre>

Writing the led_keys is a bit more involved. The file expects a single digit which is constructed in the following way:
<pre>
value= wasd &lt;&lt; 4 | key

where: 
   wasd - WASD light intensity
   key - Other key light intensity

Only values from 0-4 accepted
</pre>


This version is 100% a WIP and reflects my own noobish attempt at combining certain elements of Mikefaille's version and Bombatower's version so that it (hopefully) works on my Ubuntu based install with Plasma 4 as a DE.

I was able to get Mikefaille's version to work quite well with the macro keys as normal modifiers, but Bombatower's version was having a bit of trouble sending outputs that qt could understand in the slightest. If this works for you, honestly just thank Wattos, Mikefaille, and Bombatower for their work--I repeat: THIS IS JUST A BODGE THAT I JURY-RIGGED TOGETHER IN ABOUT 15 MINUTES
