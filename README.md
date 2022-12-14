## What this is

Hack a PR3000 trail camera (camera trap) to add external trigger
functionality.  There are 2 options, _export_ the trigger (as
a master unit), or _accept_ it (as a slave unit).  This allows
multiple cameras to be connected together and be triggered
simultaneously, if one of them is a master, or by an external
switch.

This project is in beta, use at your own risk.

[![Modifying as master](https://img.youtube.com/vi/B8RjDWIedTY/0.jpg)](https://www.youtube.com/watch?v=B8RjDWIedTY)

Not all trail cameras can be modified in the same way.  There needs
to be enough space to add the transmitter or receiver board, and
there must be space to add a connector in a weatherproof manner.
Here I repurpose the external battery jack for this purpose.

## Triggering in a trail camera

It is assumed that only cameras of the same make and model can share
triggers, since only then can it be guaranteed that they can
understand each other.  In practice, I have only ever seen triggers
signals that are active high, and they stay high for about 2 or 3
seconds.

The transmitter and receiver boards translate this signal between
active high/pushpull internally and active low/open drain externally.

### Tracing the signal path

In general, the PIR sensor outputs a signal that ultimately becomes
the trigger signal that will cause the camera to capture an image.
The trigger signal is active high (usually 3.3V), and stays high
for a few seconds (typically 2).

We want to be able to either insert our own trigger signal into
the existing signal path, or read this signal.  For this, we need
to trace the signal path starting at the PIR sensor’s output pin.

PIR sensors are easy to identify.  They are a through-hole component,
in a TO-5 metal can package, with a window.  They can have 3, 4,
or more pins.

### 4 or more pins
The
[BS412](https://cdn-shop.adafruit.com/product-files/4666/Datasheet.pdf)
is representative of this type.  Here,
the output pin is usually called REL (short for relay) or VOUT.
The sensor contains the relevant conditioning functionality,
so the REL pin outputs the TRIGGER signal to the MCU, no further
conditioning necessary.

The [PIR612](https://www.sunrom.com/get/751778) works similarly,
except it has 6 pins.  TRIGGER signal is on pin 5, VOUT.

First identify the REL pin on the PIR.  Trace this pin (using a
continuity tester) to a diode which will be close by.  There may
be an intervening low value resistor before the diode.

If the unit is to be a master, we want to connect the output of
this diode to the INT\_TRIG point on the transmitter circuit.
If the unit is to be a slave, connect the output of this diode to
the INT\_TRIG point on the receiver circuit instead.

### 3 pins
The [RE200](http://www.ladyada.net/media/sensors/RE200B.pdf)
and [D203S](https://www.allelectronics.com/mas_assets/media/allelectronics2018/spec/D203S.pdf)
are representative of this type.
The output pin is usually called Source or Output, and the output
signal is analog, not digital.  If the unit is to be a slave,
connect the Source pin to the INT\_TRIG point on the receiver
circuit (the transmitter board is not used).

The signal must therefore pass on to another chip such as the
[SGM8139](http://cn.sg-micro.com/uploads/soft/20200227/1582798839.pdf)
or [BISS0001](http://www.sc-tech.cn/en/BISS0001.pdf),
which conditions the signal to obtain the
digital trigger signal, which is routed to the MCU.

If the unit is to be a master, identify the Output pin of the
SGM8139/BISS0001 (pin 2).  This will be connected to 
INT\_TRIG of the transmitter board.

In all cases, the GND point on the transmitter and receiver boards
should be connected to any convenient ground pin on the CT board,
and the VCC point on the receiver board should be connected to a
3.3V pin on the CT board, such as the VDD pin on the PIR.

