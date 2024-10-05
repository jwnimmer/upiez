# upiez

The upiez is a printed circuit board with a piezoelectric sensor input to detect vibration (e.g., when a drum head is hit), and when a strong enough vibration is sensed it switches on its power output (e.g., lighting up a LED strip) for a configurable, short amount of time.

The board accepts an input voltage from 4V to 20V, and can power up to a 3A load.  The input voltage should match whatever is used by the load (LED strip) -- it does not regulate the output, it only switches the power on and off.

The board offers two trimmers for adjustment -- one trimmer to adjust the piezo input sensitivity trigger, and one trimmer to adjust the time the LED stays illuminated after each trigger.

<img alt="Photo of top of assembled SMT PCB (no through-hole connectors)" src="https://github.com/jwnimmer/upiez/raw/main/images/pcba_top.jpg" width="200px" />

Connectors:
* LED output is on a 5.5/2.1mm barrel jack ([PJ-063AH](https://www.digikey.com/en/products/detail/cui-devices/PJ-063AH/2161208)) at J1.
* Piezo input is on 2-pin header (0.100" pitch) at J2.
* Power input is on solder pads for hard-wiring (on the bottom).

Designed summer 2024 for the LWHS drum line.

For an easier starting point, see also the (TBD link) breadboard version of this project.

## Theory of operation

TBD

## Single channel version

<img alt="Photo of single channel stack-up with power system" src="https://github.com/jwnimmer/upiez/raw/main/images/single.jpg" width="200px" />

TBD

## Quad channel version

<img alt="Photo of 4-channel channel stack-up with USB power" src="https://github.com/jwnimmer/upiez/raw/main/images/quads.jpg" width="200px" />

TBD

## PCB assembly

TBD

## Harness assembly

TBD

## Credits

TBD
