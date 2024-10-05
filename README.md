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

<img alt="Schematic drawing" src="https://github.com/jwnimmer/upiez/raw/main/images/schematic.png" width="800px" />

* Power regulation for the internal components:
  * Input power comes in on the J3 and J4 pads.
  * U2 regulates the power down to 3.3V for use by the internal components, with C1 and C2 for filtering.
  * D1 illuminates to indicate suitable power input (at ~4mA per the 2V LED drop and 330 ohm R1).
* Piezoelectric sensor input:
  * An external piezo sensor harness is connected to J2.
  * For the purposes of this exposition, treat the sensor as a current source.
  * When current is input via J2, it develops a voltage across VR1 pins 1,3.
  * The user's setting of the VR1 trimmer divides that voltage onto VR1 pin 2, which feeds the N-type MOSFET gate at Q1 pin 1.
  * Q1 provides ESD protection on its gate, so if (when) the voltage developed across the trimmer goes too positive or too negative it shunts the current to ground.
    * I've seen other projects use a dedicated protection diode for this, but relying on the ESD protection saves us on part count and seems to work OK so far.
  * When the gate voltage goes high enough, Q1 turns on (connecting pins 2 and 3), thus pulling U1's pin 2 ("trigger") low.  (The 555 trigger input is active-low.)
    * With no piezo input, U1 pin 2 is pulled high by VR2.
      * This is a little trick to save on part count. By putting the 3.3V on the middle pin of the VR2 trimmer, we can use half of the part (pins 1 and 2) for the RC timer (described below) and the other half (pins 2 and 3) as a pull-up resistor for the trigger. The pull up will have an arbitrary resistance (>= 1k ohm) but that doesn't matter.
* Timer:
  * U2 is a CMOS 555 timer configured in the classic [Monostable mode](https://en.wikipedia.org/wiki/555_timer_IC#Monostable).
    * The decoupling capacitor C4 is normally supposed to be a 100x smaller value (higher frequency response), but I decided it didn't matter enough to increase the unique part count.
  * In the RC circuit we have a fixed 1uF capacitor with a variable VR2 trimmer (1k..99k ohm).
    * This allows for a range of timer output between 1-108 milliseconds.
    * For reference, at a tempo of 144 a sixteenth note is 104 milliseconds.
* Output stage:
  * When the timer output (pin 3) goes high:
    * D2 illuminates; this is helpful for debugging.
    * Q2 turns on (connecting pins 2 and 2), thus providing a ground for the barrel jack J1's negative terminal.
      * J1 positive terminal is always connected to the power input (VDC coming in from J3).

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

## Mechanical drawing

TBD

## Credits

TBD
