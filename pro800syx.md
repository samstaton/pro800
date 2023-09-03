# Pro 800 System Exclusive implementation

Attempt at reconstructing by
[Sam](https://github.com/samstaton/pro800). I did this by probing the unit with various commands. 

Thanks to Sebo Synths and sw for finding some of the offsets. 

To access this, you can use [my very plain viewer/editor](https://raw.githack.com/samstaton/pro800/main/pro800.html). There are also a handful of more user-friendly controllers listed at  [gearspace](https://gearspace.com/board/electronic-music-instruments-and-electronic-music-production/1410544-pro-800-controller.html) including [swumpf's nice browser editor](https://swumpf.com/pro800/experimental/).

### System exclusive head
Every system exclusive message has the following format:

`F0 00 20 32 00 01 24 00 Data F7`

### Version request

`08 00`

Response:

`09 00 xx yy zz` 
where version = `xx`.`yy`.`zz`

### Encoding 8-bit bytes as 7-bit MIDI

* MIDI system exclusive data is 7 bits (00-7F), but Pro 800 data uses 8-bit bytes. These bytes are encoded as follows. It is slightly different from the encoding used by the GliGli Pro 600. Each sequence of seven bytes `x1 x2 x3 x4 x5 x6 x7` is _preceeded_ by a byte `x0` that specifies the MSBs of the following seven bytes. The MSB of `x0` specifies the MSB of `x7`; the LSB of `x0` specifies the MSB of `x1`, and so on.  

* Reference for Pro 600 GliGli midi at [https://www.image-et-son.com](https://www.image-et-son.com). 


### System Settings request

Use data:

`77 7E 03`

Response will be a message 

`78 7E 03 details`

Here `details` describes the system settings. 
This is 8-bit data encoded as 7-bit midi. 

Bytes offset 5 and 6 describe the current patch number, with the byte 5 the least significant. 


### Patch Request 

```77 xx yy ```

where `xx yy` is a number 0-399, as 7-bit data, with `xx` least significant. 

This is awkward because it is slightly different from the patch numbering returned by the System Settings request, which is two 8-bit bytes. 

Response:

```78 xx yy details```

Patch data `details` has the following format. 

* Once decoded, from offset 5, the `details` seems to appear in almost the order used by GliGli. I have checked up to Amp Envelope Shape but not beyond.
* The patch name seems to start at index 150 (0x96) and continue until the first 0. The length varies. After the patch name there is sometimes further data (arpeggiator? sequencer)

#### Patch data details

| Offset | Operation | Num bytes | Mode | Steps |
| --- | --- | --- | --- | --- |
|4|Version|1|Stepped|0x6E for firmware to 1.2.7; 0x6F for firmware 1.3.6 and upwards|
|5|Freq A|2|Continuous||
|7|Vol A|2|Continuous||
|9|PWA|2|Continuous||
|11|Freq B|2|Continuous||
|13|Vol B|2|Continuous||
|15|PWB|2|Continuous||
|17|Fine B|2|Continuous||
|19|Cutoff|2|Continuous||
|21|Res|2|Continuous||
|23|Filt Env|2|Continuous||
|25|FE R|2|Continuous||
|27|FE S|2|Continuous||
|29|FE D|2|Continuous||
|31|FE A|2|Continuous||
|33|AE R|2|Continuous||
|35|AE S|2|Continuous||
|37|AE D|2|Continuous||
|39|AE A|2|Continuous||
|41|PM Env|2|Continuous||
|43|PM OscB|2|Continuous||
|45|LFO Freq|2|Continuous||
|47|LFO Amt|2|Continuous||
|49|Glide|2|Continuous||
|51|Amp Vel|2|Continuous||
|53|Filt Vel|2|Continuous||
|55|Saw A|1|Stepped|Off,On|
|56|Tri A|1|Stepped|Off,On|
|57|Sqr A|1|Stepped|Off,On|
|58|Saw B|1|Stepped|Off,On|
|59|Tri B|1|Stepped|Off,On|
|60|Sqr B|1|Stepped|Off,On|
|61|Sync|1|Stepped|Off,On|
|62|PM Freq|1|Stepped|Off,On|
|63|PM Filt|1|Stepped|Off,On|
|64|LFO Shape|1|Stepped|Pulse,Tri,Rand,Sin,Noise,Saw|
|65|LFO Range|1|Stepped|Slow,Fast|
|66|LFO Target|1|Stepped|Bit mask: Freq (1) / Filter (2) / PW (4) / Osc A (8) / Osc B (16) / ?? (32) / VCA (64) |
|67|KeyTrk|1|Stepped|Off,Half,Full|
|68|FE Shape|1|Stepped|Lin,Exp|
|69|FE Speed|1|Stepped|Fast,Slow|
|70|AE Shape|1|Stepped|Lin,Exp|
|71|Unison|1|Stepped|Off,On|
|72|Pitchbend target|1|Off,VCO,VCF,Vol|
|73|Modwheel amt|1|Stepped|Min,Low,High,Full|
|74|OscA freq mode|1|Free,Semi,Oct,FiHD|
|75|OscB freq mode|1|Free,Semi,Oct,FiHD|
|76|Mod delay|2|Continuous||
|82|Detune|2|Continuous||
|78|Vibrato speed|2|Continuous||
|80|Vibrato amount|2|Continuous||
|84|Modwheel target|1|Stepped|LFO,Vib|
|87|Unison voice pattern|7|Bytes encoding notes, as semitone intervals above the root, or `FF` for mono unison|
|94|Per-note tuning|48|4 bytes per note (C-B). Least significant byte first. Most significant bit of fourth bit is the sign (tuning up or down). Scaling seems strange. See [tuning.csv](tuning.csv) for rough values, or use examples below.
|142|Noise|2|Continuous||
|144|VCA Aftertouch|2|Continuous||
|146|VCA Aftertouch|2|Continuous||
|148|AE Speed|1|Stepped|Fast,Slow|
|150|Patch name|??|Text, seems to be delimited by 0. In version 0x6E, the LFO AT amount is immediately after the patch name. There are reports that if there is no LFO AT amount then the 0 delimiter may be omitted. In version 0x6F, patch name is always 16 bytes, and unused bytes are 0.||
|166|LFO AT amount|2|Continuous||
|168|Voice spread|1|Stepped|On,Off (from version 0x6F)|
|169|Keyboard Tracking reference note|1|Stepped|0x03=C4,0x02=C3 etc (from version 0x6F)|
|170|Glide mode|1|Stepped|Time,Speed (from version 0x6F)|
|171|Pitch range|2|Continuous|Default = 0x00 0x60, i.e. 12 semitones. Presumably 0x0800 is one semitone. (From version 0x6F)|

##### Per-note tuning table examples: 

12TET:
`00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00`

Quarter-comma meantone:
`00 00 00 00 ae ff 7c be 62 d6 9d bd 2b 0c d7 3d 1d 3d 07 be e7 5a 03 3d 56 47 5e be e7 5a 03 bd fc db 8d be 2b 0c d7 bd 62 d6 9d 3d 98 51 35 be`

Pythagorean: 
`00 00 00 00 2b 0c d7 bd aa 50 2c 3d 88 5b 89 bd d3 13 a8 3d 4a ca b4 bc 65 1e 02 3e 4a ca b4 3c d3 13 a8 bd 88 5b 89 3d aa 50 2c bd 2b 0c d7 3d`

Werckmeister III:
`00 00 00 00 2b 0c d7 bd d3 13 a8 bd 88 5b 89 bd 2b 0c d7 bd 4a ca b4 bc 65 1e 02 be aa 50 2c bd d3 13 a8 bd 65 1e 02 be aa 50 2c bd d3 13 a8 bd`

Note that the mapping from cents to values is not linear, possibly logarithmic.

#### Firmware update

```0A ... ```

```0B ... ```
