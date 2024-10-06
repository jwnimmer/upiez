# upiez

The upiez is a printed circuit board with a piezoelectric sensor input to detect vibration (e.g., when a drum head is hit), and when a strong enough vibration is sensed it switches on its power output (e.g., lighting up a LED strip) for a configurable, short amount of time.

<img alt="Photograph of fully-assembled PCB (including through-hole connectors)" src="https://github.com/jwnimmer/upiez/raw/main/images/upiez_profile.jpg" height="200px" /> <img alt="Rendering of fully-assembled PCB (including through-hole connectors)" src="https://github.com/jwnimmer/upiez/raw/main/images/pcb_render.png" height="200px" />

<img alt="Photo of top of assembled SMT PCB (no through-hole connectors)" src="https://github.com/jwnimmer/upiez/raw/main/images/pcba_top.jpg" height="200px" /> <img alt="Photo of bottom of PCB with wire leads attached" src="https://github.com/jwnimmer/upiez/raw/main/images/bottom.jpg" height="200px" />

The board accepts an input voltage from 4V to 20V, and can power up to a 3A load.  The input voltage should match whatever is used by the load (LED strip) -- it does not regulate the output, it only switches the power on and off.

The board offers two trimmers for adjustment -- one trimmer to adjust the piezo input sensitivity trigger, and one trimmer to adjust the time the LED stays illuminated after each trigger.

Connectors:
* LED output is on a 5.5/2.1mm barrel jack ([PJ-063AH](https://www.digikey.com/en/products/detail/cui-devices/PJ-063AH/2161208)) at J1.
* Piezo input is on a 2-pin header (0.100" pitch) at J2.
* Power input is on solder pads for hard-wiring (on the bottom).

Size:
* PCB 28.2mm x 20.3mm (not including connector overhang).
* Mounting hole spacing 23.1mm x 15.2mm for M2.5 screws.

Designed summer 2024 for the LWHS drum line.

This project requires wire stripping and soldering skills, and ideally a multi-meter.

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
    * Q2 turns on (connecting pins 2 and 3), thus providing a ground for the barrel jack J1's negative terminal.
      * The barrel jack J1's positive terminal is always connected to the VDC power input.
      * In other words, we turn the LED output on and off by connecting and disconnting its ground, not VDC.

## Single channel version

<img alt="Photo of single channel stack-up with power system" src="https://github.com/jwnimmer/upiez/raw/main/images/single.jpg" width="200px" />

This is the single channel assembly for use with a single sensor, e.g., a snare drum or bass drum.

The upiez PCB must be externally powered, using a power source voltage that matches the voltage of the LED strip. For my build, we used this [12V LED strip](https://www.amazon.com/gp/product/B09MRK3CHC) so our power source must be 12V.

When choosing a LED strip, also consider how much current it needs. According to a review comment on that product, the full 5 meter length uses 1.94 amps @ 12 volts = 23 watts. The max current of the upiez is 3 amps, so the 1.94 amps is safely within the limit.

In our case, we never put a full 5 meters of lights on a single drum anyway. That would be a lot! We only wrapped the rim of the drum head, so approx 1 meter for snares or 2 meters for a large bass drum. The LED strip can easily be cut into segments for the necessary lengths.

This brings the power requirements down into a regime where a 9V battery can handle the load, we just need to boost it from 9V to 12V. For that, we used these [DC-DC boost modules](https://www.amazon.com/gp/product/B07BNHR4HW). Checking the ballpark math: 2 meters of LED is ~9 watts, which we source as ~9V @ ~1A. Typical sustained current from a 9V cell is around 1A with burst around 2A, so the math checks out okay.

Another way to get to 12V would be with an 8-pack of AA or AAA batteries, to avoid the need for the boost converter. I never tried this, because I am somewhat skeptical it would work well -- as you use the 1.5V batteries, their voltage droops pretty quickly to the point where your 12V supply is now 11V or 10V, and that reaches the point where your LEDs will get dim and then stop working entirely -- there is a minimum voltage floor required to illuminate all of the diodes, and it's in that neighborhood. Also the 8-pack is larger and heavier than the single 9V. Using the boost converter ensures consistent brightness so long as the battery has any juice left at all.

### Single channel version: assembly details

The DC-DC boost module linked above has a trimmer to adjust its output voltage. Before wiring it to the upiez you MUST connect it to a load (e.g., 1k resistor) and supply (9V battery) and adjust its trimmer until the output is 12V on your multimeter. If its trimmer is in the wrong position, the boost module can output up to 35V which is enough to fry the upiez (max 20V).

To turn the kit on and off, instead of (un)plugging the battery we found it a lot easier to install a [rocker switch](https://www.amazon.com/dp/B07S2QJKTX) as part of the battery harness. Then the battery can stay plugged in, but still power off the kit easily.

<img alt="Photo of 1-channel channel stack-up with mechanical details" src="https://github.com/jwnimmer/upiez/raw/main/images/single_assy.jpg" width="200px" />

This photo shows how we chose to assemble the 1-channel kit:
* On the "inner" side of each of the two boards in the stack, cover metal surfaces with kapton tape to prevent a short circuit.
* Use ~3 pieces of double-sided tape as a spacer under each of the two screw holes on the side of the upiez with the small green LEDs (D1 & D2, top of photo).
* Use the following mechanical fasteners (near C4, bottom right in photo) to attach across a mounting hole in each of the two boards:
  * [91292A014](https://www.mcmaster.com/nav/enter.asp?partnum=91292A014) 18-8 Stainless Steel Socket Head Screw, M2.5 x 0.45 mm Thread, 10 mm Long, Packs of 100
  * [94669A094](https://www.mcmaster.com/nav/enter.asp?partnum=94669A094) Aluminum Unthreaded Spacer, 4.500 mm OD, 3 mm Long, for M2.5 Screw Size
  * [91828A113](https://www.mcmaster.com/nav/enter.asp?partnum=91828A113)	18-8 Stainless Steel Hex Nut, M2.5 x 0.45 mm Thread, Packs of 100
  * [90940A102](https://www.mcmaster.com/nav/enter.asp?partnum=90940A102) Polycarbonate Plastic Washer for M2.5 Screw Size, 2.7 mm ID, 5.0 mm OD, Packs of 25

## Quad channel version

<img alt="Photo of 4-channel channel stack-up with USB power" src="https://github.com/jwnimmer/upiez/raw/main/images/quads.jpg" width="300px" />

A single-channel version of upiez is great, but in a pinch it could be built on a breadboard instead (see link TBD). The raison d'être for upiez is multi-channel operation. (Building a 4- or 6-channel version on a breadboard is not very practical.)

We designed upiez to use right-angle connectors and to match the footprint of the [Adafruit 5991](https://www.adafruit.com/product/5991#tutorials) USB-C power board, so that we can stack modules for multi-channel operation. The 5991 offers DIP switches to select an output voltage (5-20V). The important point is to also choose a USB-C PD source that is compatible with the voltage you need. We used the [Anker Nano Power Bank 30W](https://www.amazon.com/gp/product/B0C9CJKCH3) which can handle [many output voltages](https://www.anker.com/products/a1259-built-in-cable-power-bank-10000mah#productSpecs). There are probably cheaper options, but the quality and usability of Anker products is top notch and now we have a good power bank to use on vacations as well.

| Adafruit 5991 | Anker Nano |
| --- | --- |
| <img src="https://cdn-shop.adafruit.com/970x728/5991-05.jpg" height="200px" /> | <img src="https://m.media-amazon.com/images/I/614OfiBkyZL._AC_SL1500_.jpg" height="200px" /> |

### Quad channel version: assembly details

TBD

* Use the following mechanical fasteners (required quantities listed are pieces, not packs):
  * 4x [93805A120](https://www.mcmaster.com/nav/enter.asp?partnum=93805A120) 18-8 Stainless Steel Threaded Rod, M2.5 x 0.45 mm Thread Size, 50 mm Long, Packs of 10
  * 8x [91828A113](https://www.mcmaster.com/nav/enter.asp?partnum=91828A113)	18-8 Stainless Steel Hex Nut, M2.5 x 0.45 mm Thread, Packs of 100
  * 12x [94669A104](https://www.mcmaster.com/nav/enter.asp?partnum=94669A104)	Aluminum Unthreaded Spacer, 4.500 mm OD, 10 mm Long, for M2.5 Screw Size
  * 4x [94669A094](https://www.mcmaster.com/nav/enter.asp?partnum=94669A094) Aluminum Unthreaded Spacer, 4.500 mm OD, 3 mm Long, for M2.5 Screw Size
  * 40x [90940A102](https://www.mcmaster.com/nav/enter.asp?partnum=90940A102) Polycarbonate Plastic Washer for M2.5 Screw Size, 2.7 mm ID, 5.0 mm OD, Packs of 25

## PCB assembly

<img alt="Bottom side of panelized PCB" src="https://github.com/jwnimmer/upiez/raw/main/images/panelized.jpg" height="300px" />

We ordered the boards from PcbWay, including SMT component sourcing and assembly. Because of the small size of the board, it is processed and shipped as panelized (2 x 2) with break-away rails (so must be ordered in multiples of 4). The first step for finishing up the assembly is to carefully break all of the board off of their rails.

After that, hand solder the connectors:
* J1 is [Cui PJ-063AH](https://www.digikey.com/en/products/detail/cui-devices/PJ-063AH/2161208)
  * Mount on the top of the board, solder from the bottom.
* J2 is [Harwin M20-9950245](https://www.digikey.com/en/products/detail/harwin-inc/M20-9950245/3724345)
  * N.B. This connector part number is a 2x2 header, but we need a 1x2 header. Carefully cut the connector in half. (Buy spares since you'll probably break some accidentally.)
  * Mount on the top of the board, solder from the bottom and then flip back to the top to touch up with some extra solder.
  * If you are building a single-channel board and don't care about the right angle header, you can use any cheap [0.100" pitch pin header](https://www.amazon.com/dp/B0CZ6VNLGZ) instead. The piezo harness will exit from the top of the board so will be less ergonomic to use, but if you already have 0.100" pin headers lying around will be easier and cheaper to solder up.
* VDC and GND are on solder pads on the bottom of the board, for wires.
  * For me, soldering in the [22 AWG stranded 2-conductor cable](https://www.amazon.com/dp/B077XBWX8V) was easier than solid wires.
  * Note that each of the two the pads is right next to a through-hole pin on the same circuit, so it's OK for the wire solder to join up with the through-hole solder.

In our first build we soldered J1 then J2 then the wires. This ended up being somewhat of a headache because J1 is very thoroughly connected to the board's ground plane and its metal shield, so the ground pin of J2 and the GND wire both struggled to flow evenly with the iron we used due to heat leaking away quickly. Next time, we should probably solder J1's shield pins last.

## Harnesses

### Piezo harness 

The upiez piezo input is on J2, via a standard 0.100" 2-pin header. There are many ways to wire up a cable to this, I'll detail the one I used:

* Use pre-made [Female Breadboard Jumper Wires 2.54mm](https://www.amazon.com/dp/B07GD1W1VL) for the cable end that connects to upiez.
  * Peel off a pair of two wires, and then cut the jumper in half (so one end is connectorized and the other end is wire leads).
* Use [22 AWG stranded 2-conductor cable](https://www.amazon.com/dp/B077XBWX8V) for the long cable run.
* Use [1" piezo sensor with with leads](https://www.amazon.com/dp/B07B8PFJCX) for the sensor at the end of the cable.
  * Add dabs of hot glue on the piezo discs to secure the tips of the wires to the metal. The connections from the piezo wire leads to the solder blobs on the piezo discs are the weakest link and *will* break off unless you reinforce it with glue.
* Prepare and stage appropriately sized [heat shrink](https://www.amazon.com/gp/product/B0BG6V4HV4) on all of the wires / cables.
  * Use a small piece on the red (positive) joints, to prevent shorting to the nearby ground joint.
  * Wrapping the black (negative) joints is also fine but probably not necessary.
  * Use a larger piece around the whole cable at the joint, for mechanical stress relief.
* Strip the ends of wires as necessary, solder the connections, slide the heat shrink into place and shrink it.

### LED harness

This is the simplest of the harnesses. Your LED strip might already come with barrel connector(s) built-in. If not, buy a pack of 5.5mm/2.1mm "male" barrel jacks with leads and solder to your LED strip(s). For example, here are [jacks with 20-inch leads](https://www.amazon.com/dp/B07SV2WY4S), or if you need longer try something like [double-ended 10' cables](https://www.amazon.com/dp/B09399KNJT) and cut them in half. For stress relief, cover the LED-wire joint with heat shrink tubing.

### Single channel power / battery

[22 AWG stranded 2-conductor cable](https://www.amazon.com/dp/B077XBWX8V)

TBD

### Dual channel power pigtails

TBD

## Mechanical drawing

TBD

## Costs

Here's the approximate part costs as of 2024. Costs shown are all per-item, based on qty 20 purchase.

| Item | Cost (USD) | 1-channel qty | 4-channel qty |
| --- | --- | --- | --- | 
| upiez board (assembled) | 6.35 | 1 | 4 |
| Piezo harness | 2.00 | 1 | 4 |
| LED harness | 1.00 | 1 | 4 |
| HiLetgo XL6009 DC-DC boost | 1.90 | 1 | |
| 9V battery harness | 1.50 | 1 | |
| 1-channel fasteners | 1.15  | 1 | |
| Adafruit 5991 USB-C power | 9.95 | | 1 |
| 4-channel fasteners | 25.65 | | 1 |

Nearly half of the fasteners cost is from the washers. In retrospect, I should have selected a cheaper part number, or omitted the washers.

For the 1-channel assembly, total cost is 6.35 + 2.00 + 1.00 + 1.90 + 1.50 + 1.15 = $13.90 (plus the cost of the LED strip and 9V battery).

For the 4-channel assembly, total cost is 4 * (6.35 + 2.00 + 1.00) + 9.95 + 25.65 = $73.00 (plus the cost of the LED strips and the USB-C battery).

### upiez cost detail

| Item | Cost (USD) |
| --- | --- | 
| Board (unpopulated) | 1.14 |
| SMT components | 2.26 |
| SMT assembly | 1.50 |
| THT components | 1.45 |
| THT assembly | 0.00 (self) |

## Credits

Reading other people's take on this kind of project was a great help in my own design. My primary sources were:
* Davide Gironi: https://davidegironi.blogspot.com/2014/06/drum-light-trigger-that-uses-leds-ne555.html
* Sam vs Sound: https://samvssound.com/2017/11/24/555-based-piezo-trigger/
