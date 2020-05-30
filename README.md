# Game Boy Color IR3R53 chip

This repository has reverse-engineered schematics for the AMP-MGB (IR3R53) audio amplifier chip used in the Game Boy Color.

My [blog post](http://www.righto.com/2020/05/reverse-engineering-audio-chip-in.html) has more details and discussion of the chip.

I generated these schematics from John McMaster's [die photos](https://siliconpr0n.org/archive/doku.php?id=mcmaster:nintendo:ir3r53n-amp-mgb).
The schematics are hand-generated, so there may be errors; please let me know if you find any.

## Explanation of the schematics

The first page has the speaker amplifier and the control logic. The second page has the headphone amps.

The component numbers on the schematics are non-consecutive. Numbers 0xx are for a headphone channel.
Numbers 1xx are for the speaker amp.
Numbers 2xx are shared between both headphone channels.

## Speaker amplifier

The amplifier consists of a power op amp with feedback. The differential input box shows the two inputs.
On the positive side, the two input channels are summed by R5 and R5' and buffered by Q138 before being fed into the op amp at Q132.
The negative input is Q131. It takes feedback from the output through R138, an external filter capacitor at pin 17, and a current sink through R144.
This sets the amplification level and (I think) implements a high-pass filter.

The op amp has a fairly standard design with a differential input, a gain stage, and an output stage.
Power transistors Q126 and Q125 implement a class B push-pull output.

### Current mirrors

The amplifier uses several current mirrors. The current source is Q134/135. This is mirrored by Q136/Q137.
That current is mirrored by Q113, Q133, Q127, Q114, which provides current to the differential input.
Another mirror Q115, Q116 reflects the current to the final mirror Q117-Q120, which feeds the output stage.
Note the parallel transistors used in many places to scale the currents. Also note the emitter resistors on Q127 and Q133.

The current mirror is partially shut down to disable the amplifier. When Q115 is pulled low, this shuts off the last two
mirrors, but leaves the first three still powered.
One subtlety is Q112, which provides additional current into the current mirror when Q115 is active.
Thus, the input stage is partially powered down, but not entirely. I'm not sure what the benefit of this is.

### Output drive

The drive and bias circuitry (Q121-Q124) makes sense to me in a hand-waving way:
Q128 takes the output from the differential input stage and amplifies it.
Q117-Q120 are a current mirror, with the current from each transistor scaled differently.
Q118 (and Q119) provide base current for Q126, turning it on and pulling the output high.

If Q128 conducts, the current from Q118 and Q119 flows through Q124 and Q128, turning off Q126.
Meanwhile, Q121 will mirror the current through Q122, turning on Q125 and pulling the output low.
Otherwise, Q123 will "dump" the current from Q117 to ground.

Thus, the input current results in an inverted, amplified output.
The transistors are carefully sized (e.g. x4 indicates 4 transistors in parallel for four times the current), so there's something clever going on with the currents that I'm missing.

### Logic

At the bottom of the schematic is the logic that enables and disables the amplifiers.
The idea is that if SD (pin 5) is pulled low, everything is disabled.
(This pin is pulled high in the Game Boy, so this feature isn't used.)
The headphone jack switch (pin 6) switches between the speaker amplifier and the headphone amplifier.
I assume that powering down the unused amplifier saves battery life.

This logic is essentially resistor-transistor logic (RTL).
One peculiar circuit is Q109, which seems to replace a pull-up resistor. If SD is low, the pull-up through
R112 and R111 is disabled. This seems to add complexity compared to using a pull-up resistor directly.
Maybe it saves a bit of power?

## Headphone amplifier

The second sheet shows the headphone amplifier. This circuit is almost identical to the speaker amplifier.
The most important difference is there are two amplifiers (one for the left channel and one for the right channel).
Almost all the circuitry is duplicated for the two channels, except for the current mirror circuitry on the left, which
is shared by the two channels.

The other difference is the headphone amplifier is lower-power than the speaker amplifier and uses smaller transistors
in many places. This is most visible in the output transistors, but the transistors in the current mirror and output stage
are also sized differently.

At the bottom of the schematic, the input signals are split between the headphone amplifier and the speaker amplifier.
