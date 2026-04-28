# Pro 800 System Exclusive — Defros additions

In April 2026, Defros (Eindhoven) sent me an extended version of
[pro800syx.md](../pro800syx.md) with new findings against firmware 1.4.6,
including a MIDI CC table, parsed system data dumps, worked patch examples,
and notes on system-area parameters.

Lightly cleaned to render as markdown on GitHub but otherwise as received. Use at own risk.

—Sam, 28 April 2026

---

Attempt at reconstructing by [Sam](https://github.com/samstaton/pro800). I did this by probing the unit with various commands. Thanks to Sebo Synths and sw for finding some of the offsets.

Additions: Defros, Eindhoven, 27.4.26

To access this, you can use [my very plain viewer/editor](https://raw.githack.com/samstaton/pro800/main/pro800.html). There are also a handful of more user-friendly controllers listed at [gearspace](https://gearspace.com/board/electronic-music-instruments-and-electronic-music-production/1410544-pro-800-controller.html) including [swumpf's nice browser editor](https://swumpf.com/pro800/experimental/).

### System exclusive head

Every system exclusive message has the following format:

`F0 00 20 32 00 01 24 00 Data F7`

Behringer DeviceID

### Version request

`08 00`

F0 00 20 32 00 01 24 00 **08** 00 F7

Response:

`09 00 xx yy zz`

where version = `xx`.`yy`.`zz`

F0 00 20 32 00 01 24 00 **09** 00 **01 04 06** F7

Means version 1.4.6

F0 00 20 32 00 01 24 00 **01** 00 01 F7 means this adress is unknown

### Encoding 8-bit bytes as 7-bit MIDI

* MIDI system exclusive data is 7 bits (00-7F), but Pro 800 data uses 8-bit bytes. These bytes are encoded as follows. It is slightly different from the encoding used by the GliGli Pro 600. Each sequence of seven bytes `x1 x2 x3 x4 x5 x6 x7` is _preceeded_ by a byte `x0` that specifies the MSBs of the following seven bytes. The MSB of `x0` specifies the MSB of `x7`; the LSB of `x0` specifies the MSB of `x1`, and so on.

* Reference for Pro 600 GliGli midi at [https://www.image-et-son.com](https://www.image-et-son.com).

### System Settings request
Dump Request =77

Dump System = 7E 03

**Dump Request System**
F0 00 20 32 00 01 24 00 **77 7E 03** F7

**Dump Response System** will be a message

F0 00 20 32 00 01 24 00 **78 7E** **03** 01 25 16 61 00 **6F 02** 00 44 01 02 7F 02 00 00 72 00 03 00 10 07 03 03 00 00 01 00 00 0C 04 00 01 00 00 01 00 01 01 32 32 00 00 00 00 01 01 F7

This is 8-bit data encoded as 7-bit midi.

Bytes offset 5 and 6 describe the current patch number, with the byte 5 the least significant.

### Dump Request Patch
Dump Request =77

F0 00 20 32 00 01 24 00 77 **01 00** F7

Bank A patch 01 = 00..99 bank 0

F0 00 20 32 00 01 24 00 77 **65 00** F7

Bank B patch 01 = 100..199 bank 1 = **101**
F0 00 20 32 00 01 24 00 77 **49 01** F7

Bank C patch 01 = 200..299 = 73+128 = **201**
F0 00 20 32 00 01 24 00 77 **2D 02** F7

Bank D patch 01 = 300..399 = 45+256 = **301**
F0 00 20 32 00 01 24 00 77 **0F 03** F7

Bank D patch 399 = 399 = 0F+384 = **399**
F0 00 20 32 00 01 24 00 **77 7E 03** F7 = 126+384 = adress 510 = start system

```77 xx yy ```

where `xx yy` is a number 0-399, as 7-bit data, with `xx` least significant.

This is awkward because it is slightly different from the patch numbering returned by the System Settings request, which is two 8-bit bytes.

Response:

```78 xx yy details```

**Dump Sysex bank 0 program 0 (A-00) = 78 (format 6D)**
*00 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29*

F0 00 20 32 00 01 24 00 **78 00 00** 01 25 16 61 00 **6D** 00 7A 16 00 1D 4C 54 00 7B 00 5F 02 6F 26 00 00 66 15 0F 00 04 6F 0E 00 00 00 00 00 00 3B 00 00 00 00 03 7F 7F 00 00 00 00 00 11 00 00 00 00 2B 00 00 00 00 00 00 00 00 00 00 00 00 01 00 00 01 01 00 00 00 01 01 04 02 00 00 00 00 00 01 02 02 02 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 01 00 62 61 73 69 00 63 00 6F 77 20 46 6C 00 6F 77 20 25 00 00 F7

B A S I C _ P A T C H W _ _ LFO aftertouch F7

*Parameter Number*

01 = field with MSB x7

> Parameter Continuous = 2 bytes
>
> Parameter Stepped = 1 byte
>
> Parameter Stepped = 1 byte (name 0-15, *=variable length)

F0 00 20 32 00 01 24 00 **78 00 00** 01 25 16 61 00 **6D** *05 05 16 07 07 09 09 11 11 13 5F 13 15 15 17 17 19 19 0F 21 21 23 23 25 25 27 00 27 29 29 31 31 33 33 03 35 35 37 37 39 39 41 11 41 43 43 45 45 47 47 00 49 49 51 51 53 53 55 00 56 57 58 59 60 61 62 00 63 64 65 66 67 68 69 00 70 71 72 73 74 75 76 00 76 78 78 80 80 82 82 00 84 -- 86 87 88 89 90 00 91 92 93 94 94 96 96 00 98 98 100 100 102 102 104 00 104 106 106 108 108 110 110 00 112 112 114 114 116 116 118 00 118 120 120 122 122 124 124 00 126 126 128 128 130 130 132 00 132 134 134 136 136 138 138 00 140 140 142 142 144 144 146 00 146 148 149 150 151 152 153 00 154 155 156 157 158 159 160 00 161 162 163 (164 165)* 166 166* F7


**Dump Sysex bank 1 program 0 (B-00) = 78 (format 6E)**
*00 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29*

F0 00 20 32 00 01 24 00 **78 65 00** 41 25 16 61 00 **6E** 00 00 3A 00 00 00 00 00 02 00 1C 00 00 00 00 00 00 5B 4C 00 12 3B 23 00 58 7F 21 7F 00 00 00 00 00 52 03 7F 7F 00 00 00 00 00 00 00 00 00 00 00 00 00 30 00 00 00 00 7F 7F 01 00 01 00 00 00 00 00 00 00 00 00 01 00 01 00 01 00 00 00 01 01 02 02 00 04 00 00 00 00 00 00 00 78 01 00 00 7F 7F 7F 7F 07 7F 7F 7F 00 00 00 00 6B 2E 7F 7C 3E 62 56 1D 09 3D 2B 0C 57 3D 1D 3D 06 07 3E 67 5A 03 3D 56 4C 47 5E 3E 67 5A 03 3D 4F 7C 5B 0D 3E 2B 0C 57 2D 3D 62 56 1D 3D 18 51 02 35 3E 00 00 00 00 00 00 00 01 00 4D 65 61 6E 00 74 6F 6E 65 00 F7

M E A N T O N E F7

*Parameter Number*

01 = field with MSB x7

> Parameter Continuous = 2 bytes
>
> Parameter Stepped = 1 byte
>
> Parameter Stepped = 1 byte (name 0-15, *=variable length)

F0 00 20 32 00 01 24 00 **78 65 00** 41 25 16 61 00 **6D** *05 05 3A 07 07 09 09 11 11 13 1C 13 15 15 17 17 19 19 4C 21 21 23 23 25 25 27 21 27 29 29 31 31 33 33 03 35 35 37 37 39 39 41 00 41 43 43 45 45 47 47 30 49 49 51 51 53 53 55 00 56 57 58 59 60 61 62 00 63 64 65 66 67 68 69 00 70 71 72 73 74 75 76 04 76 78 78 80 80 82 82 78 84 -- 86 87 88 89 90 07 91 92 93 94 94 96 96 6B 98 98 100 100 102 102 104 09 104 106 106 108 108 110 110 06 112 112 114 114 116 116 118 4C 118 120 120 122 122 124 124 4F 126 126 128 128 130 130 132 2D 132 134 134 136 136 138 138 02 140 140 142 142 144 144 146 00 146 148 149 150 151 152 153 00 154 155 156 157 158 159 160 00 161 162 163 (164 165)** F7**


**Dump Sysex bank 0 program 3 (A-02) = 78 (format 6F)**
*00 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29*

F0 00 20 32 00 01 24 00 **78 02 00** 01 25 16 61 00 **6F** 00 20 5B 00 7C 00 00 00 60 7F 3D 7F 00 00 00 02 00 57 78 00 2B 77 1F 00 71 00 77 38 00 11 00 64 00 75 0E 00 21 00 16 00 7F 00 01 00 00 00 00 00 00 60 00 00 00 00 00 00 00 00 00 00 01 **00** 00 01 01 00 00 00 01 01 04 01 00 01 00 01 00 01 02 01 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 3F 00 00 01 00 50 6F 6C 79 00 73 69 78 20 46 31 41 00 00 00 00 00 00 00 00 00 00 01 00 00 60 F7

P o l y s i x _ F 1 H .. .. .. .. .. LFOat + new param vers 6F

*Parameter Number*

01 = field with MSB x7

> Parameter Continuous = 2 bytes
>
> Parameter Stepped = 1 byte
>
> Parameter Stepped = 1 byte (name 0-15, *=variable length)

F0 00 20 32 00 01 24 00 **78 00 00** 01 25 16 61 00 **6F** *05 05 5B07 07 09 09 11 11 13 3D 13 15 15 17 17 19 19 78 21 21 23 23 25 25 27 77 27 29 29 31 31 33 33 0E35 35 37 37 39 39 41 01 41 43 43 45 45 47 47 00 49 49 51 51 53 53 55 00 56 57 58 59 60 61 62 00 63 64 65 66 67 68 69 00 70 71 72 73 74 75 76 00 76 78 78 80 80 82 82 00 84 -- 86 87 88 89 90 00 91 92 93 94 94 96 96 00 98 98 100 100 102 102 104 00 104 106 106 108 108 110 110 00 112 112 114 114 116 116 118 00 118 120 120 122 122 124 124 00 126 126 128 128 130 130 132 00 132 134 134 136 136 138 138 00 140 140 142 142 144 144 146 00 146 148 149 150 151 152 153 00 154 155 156 157 158 159 160 00 161 162 163 164 165 166 166 168 169 170 171 171* F7

Patch data `details` has the following format.

* Once decoded, from offset 5, the `details` seems to appear in almost the order used by GliGli. I have checked up to Amp Envelope Shape but not beyond.


## Patch Data Details

With bank 0 program 2 (A-02) as example analysed (firmware 6F = v1.3.6 – v1.4.6)

| Offset | Operation | Num bytes | Mode | Steps |
| --- | --- | --- | --- | --- |

F0 00 20 32 00 01 24 00 **78 02 00**
01 field with MSB x7

25 16 61 00 **6F** 0x6E for firmware to 1.2.7; 0x6F for firmware 1.3.6 and upwards|

00 20 |5|Freq A|2|Continuous||

5B field with MSB x7

00 7C |7|Vol A|2|Continuous||

00 00 |9|PWA|2|Continuous||

00 60 |11|Freq B|2|Continuous||

7F |13|Vol B|2|Continuous||

3D field with MSB x7

7F |13|Vol B|2|Continuous||

00 00 |15|PWB|2|Continuous||

00 02 |17|Fine B|2|Continuous||

00 57 |19|Cutoff|2|Continuous||

78 field with MSB x7

00 2B |21|Res|2|Continuous||

77 1F |23|Filt Env|2|Continuous||

00 71 |25|FE R|2|Continuous||

00 |27|FE S|2|Continuous||

77 field with MSB x7

38 |27|FE S|2|Continuous||

00 11 |29|FE D|2|Continuous||

00 64 |31|FE A|2|Continuous||

00 75 |33|AE R|2|Continuous||

0E field with MSB x7

00 21 |35|AE S|2|Continuous||

00 16 |37|AE D|2|Continuous||

00 7F |39|AE A|2|Continuous||

00 |41|PM Env|2|Continuous||

01 field with MSB x7

00 |41|PM Env|2|Continuous||

00 00 |43|PM OscB|2|Continuous||

00 00 |45|LFO Freq|2|Continuous||

00 60 |47|LFO Amt|2|Continuous||

00 field with MSB x7

00 00 |49|Glide|2|Continuous||

00 00 |51|Amp Vel|2|Continuous||

00 00 |53|Filt Vel|2|Continuous||

00 |55|Saw A|1|Stepped|Off,On|

00 field with MSB x7

00 |56|Tri A|1|Stepped|Off,On|

01 |57|Sqr A|1|Stepped|Off,On|

**00** |58|Saw B|1|Stepped|Off,On|

00 |59|Tri B|1|Stepped|Off,On|

01 |60|Sqr B|1|Stepped|Off,On|

01 |61|Sync|1|Stepped|Off,On|

00 |62|PM Freq|1|Stepped|Off,On|

00 field with MSB x7

00 |63|PM Filt|1|Stepped|Off,On|

01 |64|LFO Shape|1|Stepped|Pulse,Tri,Rand,Sin,Noise,Saw|

01 |65|LFO Range|1|Stepped|Slow,Fast|

> 04 |66|LFO Target|1|Stepped|Bit mask: Freq* (&H1) / Filter* (&H2) / PW* (&H4) / Osc A (&H8) / Osc B (&H10) / VCA (&H20) / -- (&H40) / |(* = Hardware-knob) ###

01 |67|KeyTrk|1|Stepped|Off,Half,Full|

00 |68|FE Shape|1|Stepped|Lin,Exp|

01 |69|FE Speed|1|Stepped|Fast,Slow|

00 field with MSB x7

01 |70|AE Shape|1|Stepped|Lin,Exp|

00 |71|Unison|1|Stepped|Off,On|

01 |72|Pitchbend target|1|Off,VCO,VCF,Vol|

02 |73|Modwheel amt|1|Stepped|Min,Low,High,Full|

01 |74|OscA freq mode|1|Free,Semi,Oct,FiHD|

00 |75|OscB freq mode|1|Free,Semi,Oct,FiHD|

00 |76|Mod delay|2|Continuous||

00 field with MSB x7

00 |76|Mod delay|2|Continuous||

00 00 |78|Vibrato speed|2|Continuous||

00 00 |80|Vibrato amount|2|Continuous||

00 00 |82|Detune|2|Continuous||

00 field with MSB x7

00 |84|Modwheel target|1|Stepped|LFO,Vib|

00 |85|**reserved**

00 |86|chord note 1 (Bytes encoding notes, as semitone intervals above the root)

00 |87|chord note 2 (0=root, 255 no note = mono unison)

00 |88|chord note 3

00 |89|chord note 4

00 |90|chord note 5

00 field with MSB x7

00 |91|chord note 6

00 |92|chord note 7

00 |93|chord note 8

00 00 |94|Tuning|48| Tuning Table; C

00 00 |96|Tuning|2| Tuning Table; C

00 field with MSB x7

00 00 |098|Tuning|2| Tuning Table; C#

00 00 |100|Tuning|2| Tuning Table; C#

00 00 |102|Tuning|2| Tuning Table; D

00 |104|Tuning|2| Tuning Table; D

00 field with MSB x7

00 |104|Tuning|2| Tuning Table; D

00 00 |106|Tuning|2| Tuning Table; D#

00 00 |108|Tuning|2| Tuning Table; D#

00 00 |110|Tuning|2| Tuning Table; E

00 field with MSB x7

00 00 |112|Tuning|2| Tuning Table; E

00 00 |114|Tuning|2| Tuning Table; F

00 00 |116|Tuning|2| Tuning Table; F

00 |118|Tuning|2| Tuning Table; F#

00 field with MSB x7

00 |118|Tuning|2| Tuning Table; F#

00 00 |120|Tuning|2| Tuning Table; F#

00 00 |122|Tuning|2| Tuning Table; G

00 00 |124|Tuning|2| Tuning Table; G

00 field with MSB x7

00 00 |126|Tuning|2| Tuning Table; G#

00 00 |128|Tuning|2| Tuning Table; G#

00 00 |130|Tuning|2| Tuning Table; A

00 |132|Tuning|2| Tuning Table; A

00 field with MSB x7

00 |132|Tuning|2| Tuning Table; A

00 00 |134|Tuning|2| Tuning Table; A#

00 00 |136|Tuning|2| Tuning Table; A#

00 00 |138|Tuning|2| Tuning Table; B

00 field with MSB x7

00 00 |140|Tuning|2| Tuning Table; B

00 00 |142|Noise|2|Continuous||

00 00 |144|VCA Aftertouch|2|Continuous||

3F |146|VCF Aftertouch|2|Continuous||

00 field with MSB x7

00 |146|VCF Aftertouch|2|Continuous||

01 |148|AE Speed|1|Stepped|Fast,Slow|

00 |149|Arp Mode|1|Stepped|Off, Up, Down, UpDown, Up+Down, Rnd, Assign

|150|Patch name|16|Text, seems to be delimited by 0. In version 0x6E, the LFO AT amount is immediately after the patch name. There are reports that if there is no LFO AT amount then the 0 delimiter may be omitted. In version 0x6F, patch name is always 16 bytes, and unused bytes are 0.||

50 |150|Name0

6F |151|Name1

6C |152|Name2

79 |153|Name3

00 field with MSB x7

73 |154|Name4

69 |155|Name5

78 |156|Name6

20 |157|Name7

46 |158|Name8

31 |159|Name9

41 |160|Name10

00 field with MSB x7

00 |161|Name11

00 |162|Name12

00 |163|Name13

00 |164|Name14

00 |165|Name15

00 00 |166|LFO AT amount|2|Continuous||

00 |168|Voice spread|1|Stepped|On,Off (from version 0x6F)|

00 field with MSB x7

01 |169|Keyb Tracking Ref note|1|Stepped| (from version 0x6F)|

00 |170|Glide mode|1|Stepped|Time,Speed (from version 0x6F)|

00 60 |171|PB range|2|Continuous|Default = 0x00 0x60, 0-31 i.e. 12 semitones. Presumably 0x0800 is one semitone. (From version 0x6F)|

F7

#### Firmware update
```0A ... ```

```0B ... ```

#### Sequence 1 and 2
*I could not find any info about saving the Sequence 1 and 2 by Sys-ex. You should expect this somewhere in the system-area. But no respons found on adresses from (77) **10 03 \~ 7F 07.***
*Defros 210426  
*

|66|LFO Target|1|Stepped|Bit mask: Freq* (&H1) / Filter* (&H2) / PW* (&H4) / Osc A (&H8) / Osc B (&H10) / VCA (&H20) / -- (&H40) / |(* = Hardware-knob)

| 6    | 5        | 4        | 3        | 2        | 1        | 0        |     |     |     |
|------|----------|----------|----------|----------|----------|----------|-----|-----|-----|
| &H40 | &H20     | &H10     | &H08     | &H04     | &H02     | &H01     |     |     |     |
| --   | Software | Software | Software | Hardware | Hardware | Hardware |     |     |     |
| --   | VCA      | B        | A        | PW AB    | Filter   | Freq AB  |     | DEC | &H  |
|      | 0        | 0        | 0        | 0        | 0        | 1        |     |     |     |
|      | 0        | 0        | 1        | 0        | 0        | 1        |     | 9   | 09  |
|      | 0        | 1        | 0        | 0        | 0        | 1        |     | 11  | 0B  |
|      | 1        | 0        | 0        | 0        | 0        | 1        |     | 21  | 15  |
|      |          |          |          |          |          |          |     |     |     |
|      | 0        | 0        | 0        | 0        | 1        | 0        |     | 2   | 02  |
|      | 0        | 0        | 1        | 0        | 1        | 0        |     | 10  | 0A  |
|      | 0        | 1        | 0        | 0        | 1        | 0        |     | 18  | 12  |
|      | 1        | 0        | 0        | 0        | 1        | 0        |     | 34  | 22  |
|      |          |          |          |          |          |          |     |     |     |
|      | 0        | 0        | 0        | 1        | 0        | 0        |     | 4   | 04  |
|      | 0        | 0        | 1        | 1        | 0        | 0        |     | 12  | 0C  |
|      | 0        | 1        | 0        | 1        | 0        | 0        |     | 20  | 14  |
|      | 1        | 0        | 0        | 1        | 0        | 0        |     | 36  | 24  |
|      |          |          |          |          |          |          |     |     |     |
|      | 0        | 0        | 0        | 0        | 1        | 1        |     | 3   | 03  |
|      | 0        | 0        | 1        | 0        | 1        | 1        |     | 11  | 0B  |
|      | 0        | 1        | 0        | 0        | 1        | 1        |     | 19  | 13  |
|      | 1        | 0        | 0        | 0        | 1        | 1        |     | 35  | 23  |
|      |          |          |          |          |          |          |     |     |     |
|      | 0        | 0        | 0        | 1        | 0        | 1        |     | 5   | 05  |
|      | 0        | 0        | 1        | 1        | 0        | 1        |     | 13  | 0D  |
|      | 0        | 1        | 0        | 1        | 0        | 1        |     | 21  | 15  |
|      | 1        | 0        | 0        | 1        | 0        | 1        |     | 37  | 25  |
|      |          |          |          |          |          |          |     |     |     |
|      | 0        | 0        | 0        | 1        | 1        | 0        |     | 6   | 06  |
|      | 0        | 0        | 1        | 1        | 1        | 0        |     | 14  | 0E  |
|      | 0        | 1        | 0        | 1        | 1        | 0        |     | 22  | 16  |
|      | 1        | 0        | 0        | 1        | 1        | 0        |     | 38  | 26  |
|      |          |          |          |          |          |          |     |     |     |
|      | 0        | 0        | 0        | 1        | 1        | 1        |     | 7   | 07  |
|      | 0        | 0        | 1        | 1        | 1        | 1        |     | 15  | 0F  |
|      | 0        | 1        | 0        | 1        | 1        | 1        |     | 23  | 17  |
|      | 1        | 0        | 0        | 1        | 1        | 1        |     | 39  | 27  |

##### Per-note tuning table examples

*From Byte 94 and on:*

12TET:

`00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00`

Quarter-comma meantone:

`00 00 00 00 ae ff 7c be 62 d6 9d bd 2b 0c d7 3d 1d 3d 07 be e7 5a 03 3d 56 47 5e be e7 5a 03 bd fc db 8d be 2b 0c d7 bd 62 d6 9d 3d 98 51 35 be`

Pythagorean:

`00 00 00 00 2b 0c d7 bd aa 50 2c 3d 88 5b 89 bd d3 13 a8 3d 4a ca b4 bc 65 1e 02 3e 4a ca b4 3c d3 13 a8 bd 88 5b 89 3d aa 50 2c bd 2b 0c d7 3d`

Werckmeister III:

`00 00 00 00 2b 0c d7 bd d3 13 a8 bd 88 5b 89 bd 2b 0c d7 bd 4a ca b4 bc 65 1e 02 be aa 50 2c bd d3 13 a8 bd 65 1e 02 be aa 50 2c bd d3 13 a8 bd`

Note that the mapping from cents to values is not linear, possibly logarithmic.

#### Tuning ?
```38 ... ```

During tuning long were sent out (I guess corrections):

F0 00 20 32 00 01 24 00 **38** 00 11 00 F7

F0 00 20 32 00 01 24 00 38 00 12 00 F7

F0 00 20 32 00 01 24 00 38 00 15 00 F7

F0 00 20 32 00 01 24 00 38 00 16 00 F7

F0 00 20 32 00 01 24 00 38 00 17 00 F7


## System Data Details

F0 00 20 32 00 01 24 00 **78 7E 03**
01 25 16 61 00 6F 7C 00 **4**4 02 02 7F 02 00 00 **20** 00 **0F** 00 10 07 03 03 01

patch + bank Midi Inp Ch Tempo------- Screen Brightness

Midi Outp Ch Screen param Time

CC PC

00 01 00 00 0C 02 00 01 00 02 01 00 00 01 01 32 00 00 00 00 01 01

FilMA Div Preset Name Display Pp NoteLeng Midi Soft Thru

> Swing Tran Midi Local Enable

Midi Sync In Forward AutoTunePrecision Aftertouch VCA Polarity

Voice Prio Midi Sync In Polarity Aftertouch VCF Polarity

Midi Sync In Start/Stop

01 25 16 61 00 6F 7A 00 04 00

01 field with MSB x7

25 ..

16 ..

61 00 ..

6F 0x6E for firmware to 1.2.7; **0x6F for firmware 1.3.6** and upwards

7C 00 current patch + bank

**4**4 field with MSB x7

02 Mode (0 manual 1 loaded 2 edited)

02 Midi Input Channel

7F Kill Voices (voice 6543210) (voice 7 is in the MSB **4**4)

02 Midi Output Channel

00 Sync Source

00 ..

**20** BPM LSB

00 field with MSB x7

**0F** BPM MSB

00 ..

10 Screen Brightness

07 Screen Param Time

03 Midi CC

03 Midi PC

01 ..

00 field with MSB x7

01 Midi Sync In Forward

00 ..

00 Ext. Filter Mod Amount

0C .. (Range Pitchbend? Moved from de system-area in later revs to the patch-area)

02 Divider

00 Voice Priority

01 Preset Name Display

00 field with MSB x7

02 Midi Sync In Polarity

01 ..

00 AutoTunePrecision

00 Midi Sync In Start/Stop

01 Ppqm

01 NoteLength

32 Swing

00 field with MSB x7

00 Aftertouch VCA Polarity

00 Aftertouch VCF Polarity

00 Midi Transpose

01 Midi Soft Thru

01 Midi Local Enable

F7

**Mode** (found in the Synthtribe app 3.02)

0 = manual

1 = loaded

2 = edited

**1 MidiInputCh**
0 = all

1 = dipswitch

2 = ch1

3 = ch2

..

17= ch16

**1 MidiOutpCh**
0 = Thru

1 = dipswitch

2 = ch1

3 = ch2

..

17= ch16

**1 Midi CC**
0 = Off

1 = Tx

2 = Rx

3 = TxRx

**1 Midi PC**
0 = Off

1 = Tx

2 = Rx

3 = TxRx

**1 Midi Sync In Forward**
0 = Off

1 = On

**1 Midi Sync In Polarity**
0 = Rise

1 = Fall

2 = Both

**1 Midi Sync Start/stop**
0 = Off

1 = On

**1 Midi Sync in PQQM**
0 = 1PPS

1 = 1PPQ

2 = Korg

3 = 4PPQ

4 = 24 (ppq)

5 = 48 (ppq)

**1 Midi Local Enable**
0 = Off

1 = On

**1 Midi Soft Thru**
0 = Off

1 = On

**2 Midi Transpose**
00 .. 35 = + 00 -35

7D = -3

5D = -35

**7 Screen Brightness**
1-16 (16)

**7 Screen param Time**
0-100 (7)

**7 Show Preset Name Display**
0 = Off

1 = On

**8 AutoTunePrecision**
0 = 0.5c

1 = 1c

2 = 1.5c

3 = 2c

**9 Ext Filter Mod Amount**
0-999

**9 Voice Priority**
0 = Last

1 = Low

2 = High

**9 Pedal Polarity**
0 = Low

1 = High

**[Sync clock]: BPM**
50-400

**[Sync Source]**
0 = Internal Sync

1 = Midi Sync

2 = USB Sync

3 = External sync

**[Sync clock]: Divide**
00 = 14 = ¼

01 = 14 t = 1/4t

02 = 18 = 1/8

03 = 18 t = 1/8t

04 = 16 = 1/16

05 = 16 t = 1/16t

06 = 32 = 1/32

07 = 32 t = 1/32t

**[Sync clock]: Swing**
50-95

**[Sync clock]: NoteLength**
1-100

**Aftertouch VCA Polarity** (found in the Synthtribe app 3.02)

0 = High

1 = Low

**Aftertouch VCF Polarity** (found in the Synthtribe app 3.02)

0 = High

1 = Low

**[Sync clock]: BPM analysed:**
50-400

F0 00 20 32 00 01 24 00 **78 7E 03 =** introstring

01 25 16 61 00 6F 7C 00

**0**4 02 02 7F 02 00 00 **74** 00 **01** 00 10 07 03 03 01 00 01 00 00 0C

BPM--------

74 = 116 + **3**x128 = 500 =50.0

**0**4 02 02 7F 02 00 00 **7E** 00 **01** 00 10 07 03 03 01 00 01 00 00 0C

7E = 126 + **3**x128 = 510 =51.0

**0**4 02 02 7F 02 00 00 **08** 00 **02** 00 10 07 03 03 01 00 01 00 00 0C

08 = 8 + **4**x128 = 520 =52.0

..

**0**4 02 02 7F 02 00 00 **58** 00 **02** 00 10 07 03 03 01 00 01 00 00 0C

58 = 88 + **4**x128 = 600 =60.0

**0**4 02 02 7F 02 00 00 **76** 00 **02** 00 10 07 03 03 01 00 01 00 00 0C

76 = 118 + **4**x128 = 630 =63.0

**4**4 02 02 7F 02 00 00 **00** 00 **02** 00 10 07 03 03 01 00 01 00 00 0C

00 = 0 + **5**x128 = 640 =64.0

..

**4**4 02 02 7F 02 00 00 **04** 00 **03** 00 10 07 03 03 01 00 01 00 00 0C

04 = 4 + **7**x128 = 896 +4 = 900 =90.0

**4**4 02 02 7F 02 00 00 **0E** 00 **03** 00 10 07 03 03 01 00 01 00 00 0C

0E = 14 + **7**x128 = 896+14= 910 =91.0

**4**4 02 02 7F 02 00 00 **20** 00 **0F** 00 10 07 03 03 01 00 01 00 00 0C

20 = 32 + **1F**x128 = 3968+32= 4000 =400.0


## MIDI Continuous Controllers

The Pro 800 responds to a number of MIDI CC Controls:

<table>
<colgroup>
<col style="width: 6%" />
<col style="width: 5%" />
<col style="width: 26%" />
<col style="width: 7%" />
<col style="width: 13%" />
<col style="width: 11%" />
<col style="width: 29%" />
</colgroup>
<thead>
<tr class="header">
<th>CC #</th>
<th>Hex</th>
<th>Parameter</th>
<th><p>MSB Coarse</p>
<p>LSB Fine</p></th>
<th>Value Display</th>
<th>DEC</th>
<th>HEX</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>0</td>
<td>00</td>
<td>Bank Select</td>
<td>-</td>
<td>A,B,C,D</td>
<td>0-3</td>
<td>00-03</td>
</tr>
<tr class="even">
<td>1</td>
<td>01</td>
<td>Mod Wheel</td>
<td>MSB</td>
<td>-</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>2</td>
<td>02</td>
<td>*Breath</td>
<td>-</td>
<td>-</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>3</td>
<td>03</td>
<td>Master Tune</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>6</td>
<td>06</td>
<td>*NRPN Data MSB</td>
<td>-</td>
<td>-</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>7</td>
<td>07</td>
<td>Main volume</td>
<td>MSB -</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>8</td>
<td>08</td>
<td>OSC A Freq</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>9</td>
<td>09</td>
<td>OSC A Vol</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>10</td>
<td>0A</td>
<td>OSC A PW</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>11</td>
<td>0B</td>
<td>OSC B Freq</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>12</td>
<td>0C</td>
<td>OSC B Vol</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>13</td>
<td>0D</td>
<td>OSC B PW</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>14</td>
<td>0E</td>
<td>OSC B Fine</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>15</td>
<td>0F</td>
<td>VCF Freq</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>16</td>
<td>10</td>
<td>VCF Reso</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>17</td>
<td>11</td>
<td>VCF ENV amount</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>18</td>
<td>12</td>
<td>VCF Rel</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>19</td>
<td>13</td>
<td>VCF Sus</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>20</td>
<td>14</td>
<td>VCF Dec</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>21</td>
<td>15</td>
<td>VCF Atck</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>22</td>
<td>16</td>
<td>VCA Rel</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>23</td>
<td>17</td>
<td>VCA Sus</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>24</td>
<td>18</td>
<td>VCA Dec</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>25</td>
<td>19</td>
<td>VCA Atck</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>26</td>
<td>1A</td>
<td>Pmod Filter Env amount</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>27</td>
<td>1B</td>
<td>Pmod OSC B amount</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>28</td>
<td>1C</td>
<td>LFO Freq</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>29</td>
<td>1D</td>
<td>LFO Amount</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>30</td>
<td>1E</td>
<td>Glide</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>31</td>
<td>1F</td>
<td>VCA Vel</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>32</td>
<td>20</td>
<td>VCF Vel</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>33</td>
<td>21</td>
<td>Mod delay LFO2</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>34</td>
<td>22</td>
<td>Vib Freq LFO2</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>35</td>
<td>23</td>
<td>Vib Amt LFO2</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>36</td>
<td>24</td>
<td>Unison Spread Detune</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>37</td>
<td>25</td>
<td>Noise</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>38</td>
<td>26</td>
<td>*NRPN Data LSB</td>
<td>-</td>
<td>-</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>39</td>
<td>27</td>
<td>VCA Aftertouch</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>40</td>
<td>28</td>
<td>VCF Aftertouch</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>41</td>
<td>29</td>
<td>LFO Aftertouch</td>
<td>MSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>42</td>
<td>2A</td>
<td>Pitchbend Wheel Amt</td>
<td>MSB</td>
<td>0-31</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>43</td>
<td>2B</td>
<td>-empty-</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>44</td>
<td>2C</td>
<td>-empty-</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>45</td>
<td>2D</td>
<td>-empty-</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>46</td>
<td>2E</td>
<td>-empty-</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>47</td>
<td>2F</td>
<td>-empty-</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>48</td>
<td>30</td>
<td>OSC A Saw</td>
<td>-</td>
<td>Off-On</td>
<td>Off-On</td>
<td>00-40</td>
</tr>
<tr class="even">
<td>49</td>
<td>31</td>
<td>OSC A Tri</td>
<td>-</td>
<td>Off-On</td>
<td>Off-On</td>
<td>00-40</td>
</tr>
<tr class="odd">
<td>50</td>
<td>32</td>
<td>OSC A Squ</td>
<td>-</td>
<td>Off-On</td>
<td>Off-On</td>
<td>00-40</td>
</tr>
<tr class="even">
<td>51</td>
<td>33</td>
<td>OSC B Saw</td>
<td>-</td>
<td>Off-On</td>
<td>Off-On</td>
<td>00-40</td>
</tr>
<tr class="odd">
<td>52</td>
<td>34</td>
<td>OSB B Tri</td>
<td>-</td>
<td>Off-On</td>
<td>Off-On</td>
<td>00-40</td>
</tr>
<tr class="even">
<td>53</td>
<td>35</td>
<td>OSB B Squ</td>
<td>-</td>
<td>Off-On</td>
<td>Off-On</td>
<td>00-40</td>
</tr>
<tr class="odd">
<td>54</td>
<td>36</td>
<td>OSC B Sync</td>
<td>-</td>
<td>Off-On</td>
<td>Off-On</td>
<td>00-40</td>
</tr>
<tr class="even">
<td>55</td>
<td>37</td>
<td>Pmod Freq A</td>
<td>-</td>
<td>Off-On</td>
<td>Off-On</td>
<td>00-40</td>
</tr>
<tr class="odd">
<td>56</td>
<td>38</td>
<td>Pmod VCF</td>
<td>-</td>
<td>Off-On</td>
<td>Off-On</td>
<td>00-40</td>
</tr>
<tr class="even">
<td>57</td>
<td>39</td>
<td>LFO Shape</td>
<td>-</td>
<td><p>Pulse</p>
<p>Triangle</p>
<p>Random</p>
<p>Sine</p>
<p>Noise</p>
<p>SawUp</p></td>
<td><p>Pulse</p>
<p>Triangle</p>
<p>Random</p>
<p>Sine</p>
<p>Noise</p>
<p>SawUp</p></td>
<td><p>00</p>
<p>16</p>
<p>2C</p>
<p>42</p>
<p>58</p>
<p>6E</p></td>
</tr>
<tr class="odd">
<td>58</td>
<td>3A</td>
<td>LFO Speed</td>
<td>-</td>
<td>Fast-Slow</td>
<td>Fast-Slow</td>
<td>00-40</td>
</tr>
<tr class="even">
<td>59</td>
<td>3B</td>
<td>LFO Targets</td>
<td>-</td>
<td><p>AB</p>
<p>A</p>
<p>B</p>
<p>VCA</p></td>
<td><p>AB</p>
<p>A</p>
<p>B</p>
<p>VCA</p></td>
<td><p>00</p>
<p>21</p>
<p>42<br />
63</p></td>
</tr>
<tr class="odd">
<td>60</td>
<td>3C</td>
<td>VCF Keyboard</td>
<td>-</td>
<td>Off<br />
Half<br />
Full</td>
<td>Off<br />
Half<br />
Full</td>
<td>00<br />
2B<br />
56</td>
</tr>
<tr class="even">
<td>61</td>
<td>3D</td>
<td>VCF ENV Exp</td>
<td>-</td>
<td>Lin -Exp</td>
<td>Lin -Exp</td>
<td>00-40</td>
</tr>
<tr class="odd">
<td>62</td>
<td>3E</td>
<td>VCF ENV Speed</td>
<td>-</td>
<td>Fast-Slow</td>
<td>Fast-Slow</td>
<td>00-40</td>
</tr>
<tr class="even">
<td>63</td>
<td>3F</td>
<td>VCA ENV Exp</td>
<td>-</td>
<td>Lin -Exp</td>
<td>Lin -Exp</td>
<td>00-40</td>
</tr>
<tr class="odd">
<td>64</td>
<td>40</td>
<td>Sustain pedal</td>
<td>-</td>
<td>Off-On</td>
<td>Off-On</td>
<td>00-40</td>
</tr>
<tr class="even">
<td>65</td>
<td>41</td>
<td>Unison</td>
<td>-</td>
<td>Off-On</td>
<td>Off-On</td>
<td>00-40</td>
</tr>
<tr class="odd">
<td>66</td>
<td>42</td>
<td>Bender target</td>
<td>-</td>
<td>Off<br />
VCO<br />
VCF<br />
Vol</td>
<td>Off<br />
VCO<br />
VCF<br />
Vol</td>
<td>00<br />
20<br />
40<br />
60</td>
</tr>
<tr class="even">
<td>67</td>
<td>43</td>
<td>Modwheel Amount</td>
<td>-</td>
<td><p>Min 25%</p>
<p>Low 50%</p>
<p>High 75%</p>
<p>Full 100%</p></td>
<td><p>Min 25%</p>
<p>Low 50%</p>
<p>High 75%</p>
<p>Full 100%</p></td>
<td>00<br />
20<br />
40<br />
60</td>
</tr>
<tr class="odd">
<td>68</td>
<td>44</td>
<td>OscA Freq Pot Mode</td>
<td>-</td>
<td>Free<br />
SEMi<br />
Oct<br />
FiXd</td>
<td>Free<br />
SEMi<br />
Oct<br />
FiXd</td>
<td>00<br />
20<br />
40<br />
60</td>
</tr>
<tr class="even">
<td>69</td>
<td>45</td>
<td>OscB Freq Pot Mode</td>
<td>-</td>
<td>Free<br />
SEMi<br />
Oct<br />
FiXd</td>
<td>Free<br />
SEMi<br />
Oct<br />
FiXd</td>
<td>00<br />
20<br />
40<br />
60</td>
</tr>
<tr class="odd">
<td>70</td>
<td>46</td>
<td>Modwheel Target</td>
<td>-</td>
<td>LFO<br />
VIB-LFO</td>
<td>LFO<br />
VIB-LFO</td>
<td>00<br />
40</td>
</tr>
<tr class="even">
<td>71</td>
<td>47</td>
<td>-empty-</td>
<td>-</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>72</td>
<td>48</td>
<td>VCA ENV Speed</td>
<td>-</td>
<td>Fast-Slow</td>
<td>Fast-Slow</td>
<td>00-40</td>
</tr>
<tr class="even">
<td>73</td>
<td>49</td>
<td>Arp Mode</td>
<td>-</td>
<td><p>Off</p>
<p>Up</p>
<p>Down</p>
<p>UpDown</p>
<p>Up-Down</p>
<p>Assign</p>
<p>Random</p></td>
<td><p>Off</p>
<p>Up</p>
<p>Down</p>
<p>UpDown</p>
<p>Up-Down</p>
<p>Assign</p>
<p>Random</p></td>
<td><p>00</p>
<p>13</p>
<p>25</p>
<p>37</p>
<p>4A</p>
<p>5C</p>
<p>6E</p></td>
</tr>
<tr class="odd">
<td>74</td>
<td>4A</td>
<td>LFO Dest Freq</td>
<td>-</td>
<td>Off-On</td>
<td>Off-On</td>
<td>00-40</td>
</tr>
<tr class="even">
<td>75</td>
<td>4B</td>
<td>LFO Dest Filter</td>
<td>-</td>
<td>Off-On</td>
<td>Off-On</td>
<td>00-40</td>
</tr>
<tr class="odd">
<td>76</td>
<td>4C</td>
<td>LFO Dest PW</td>
<td>-</td>
<td>Off-On</td>
<td>Off-On</td>
<td>00-40</td>
</tr>
<tr class="even">
<td>77</td>
<td>4D</td>
<td>Voice Spread</td>
<td>-</td>
<td>Off-On</td>
<td>Off-On</td>
<td>00-40</td>
</tr>
<tr class="odd">
<td>78</td>
<td>4E</td>
<td>Keyboard Filter Tracking Reference Note</td>
<td>-</td>
<td>C1<br />
C2<br />
C3<br />
C4</td>
<td>C1<br />
C2<br />
C3<br />
C4</td>
<td>00<br />
20<br />
40<br />
60</td>
</tr>
<tr class="even">
<td>79</td>
<td>4F</td>
<td>Glide Mode</td>
<td>-</td>
<td><p>Time</p>
<p>Speed</p></td>
<td><p>Time</p>
<p>Speed</p></td>
<td>00<br />
40</td>
</tr>
<tr class="odd">
<td>80</td>
<td>50</td>
<td>SubDivision<br />
<em>(PC800 sends CC50 for subdivision)</em></td>
<td>-</td>
<td><p>14 = ¼</p>
<p>14t= 1/4t</p>
<p>18 = 1/8</p>
<p>18t= 1/8t</p>
<p>16 = 1/16</p>
<p>16t=1/16t</p>
<p>32 = 1/32 32t=1/32t</p></td>
<td><p>14 = ¼</p>
<p>14t= 1/4t</p>
<p>18 = 1/8</p>
<p>18t= 1/8t</p>
<p>16 = 1/16</p>
<p>16t=1/16t</p>
<p>32 = 1/32 32t=1/32t</p></td>
<td><p>00</p>
<p>02</p>
<p>04</p>
<p>06</p>
<p>08</p>
<p>0A</p>
<p>0C<br />
0E</p></td>
</tr>
<tr class="even">
<td>80</td>
<td>50</td>
<td>OSC A Freq<br />
<em>(PC800 receives CC50 as Osc A freq LSB)</em></td>
<td>LSB</td>
<td>0-999</td>
<td>00-7F</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>81</td>
<td>51</td>
<td>OSC A Vol</td>
<td>LSB</td>
<td>0-999</td>
<td>00-7F</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>82</td>
<td>52</td>
<td>OSC A PW</td>
<td>LSB</td>
<td>0-999</td>
<td>00-7F</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>83</td>
<td>53</td>
<td>OSC B Freq</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>84</td>
<td>54</td>
<td>OSC B Vol</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>85</td>
<td>55</td>
<td>OSC B PW</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>86</td>
<td>56</td>
<td>OSC B Fine</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>87</td>
<td>57</td>
<td>VCF Freq</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>88</td>
<td>58</td>
<td>VCF Reso</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>89</td>
<td>59</td>
<td>VCF ENV amount</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>90</td>
<td>5A</td>
<td>VCF Rel</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>91</td>
<td>5B</td>
<td>VCF Sus</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>92</td>
<td>5C</td>
<td>VCF Dec</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>93</td>
<td>5D</td>
<td>VCF Atck</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>94</td>
<td>5E</td>
<td>VCA Rel</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>95</td>
<td>5F</td>
<td>VCA Sus</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>96</td>
<td>60</td>
<td>*NRPN Data Inc</td>
<td>-</td>
<td>-</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>97</td>
<td>61</td>
<td>*NRPN Data Dec</td>
<td>-</td>
<td>-</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>98</td>
<td>62</td>
<td>*NRPN Par LSB</td>
<td>-</td>
<td>-</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>99</td>
<td>63</td>
<td>*NRPN Par MSB</td>
<td>-</td>
<td>-</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>100</td>
<td>64</td>
<td>VCA Dec</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>101</td>
<td>65</td>
<td>VCA Atck</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>102</td>
<td>66</td>
<td>Pmod Filter Env amount</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>103</td>
<td>67</td>
<td>Pmod OSC B amount</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>104</td>
<td>68</td>
<td>LFO Freq</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>105</td>
<td>69</td>
<td>LFO Amount</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>106</td>
<td>6A</td>
<td>Glide</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>107</td>
<td>6B</td>
<td>VCA Vel</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>108</td>
<td>6C</td>
<td>VCF Vel</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>109</td>
<td>6D</td>
<td>Mod delay LFO2</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>110</td>
<td>6E</td>
<td>Vib Freq LFO2</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>111</td>
<td>6F</td>
<td>Vib Amt LFO2</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>112</td>
<td>70</td>
<td>Unison Spread Detune</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>113</td>
<td>71</td>
<td>Noise</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>114</td>
<td>72</td>
<td>VCA Aftertouch</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>115</td>
<td>73</td>
<td>VCF Aftertouch</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>116</td>
<td>74</td>
<td>LFO Aftertouch</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>117</td>
<td>75</td>
<td>Pitchbend Wheel Amt</td>
<td>LSB</td>
<td>0-999</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td>118</td>
<td>76</td>
<td>(bug?)</td>
<td>..</td>
<td>0-31</td>
<td>0-31</td>
<td>00-1F</td>
</tr>
<tr class="odd">
<td>119</td>
<td>77</td>
<td>-empty-</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>120</td>
<td>78</td>
<td>*All Sounds Off</td>
<td>-</td>
<td>-</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="odd">
<td>123</td>
<td>7B</td>
<td>*All NotesOff</td>
<td>-</td>
<td>-</td>
<td>0-127</td>
<td>00-7F</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</tbody>
</table>
