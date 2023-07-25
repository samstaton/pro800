# Pro 800 System Exclusive implementation

Attempt at reconstructing by
[Sam](https://github.com/samstaton/pro800). I did this by probing the unit with various commands. 

### System exclusive head
Every system exclusive message has the following format:

`F0 00 20 32 00 01 24 00 Data F7`

### System Settings request

Use data:

`77 7E 03`

Response will be a message 

`78 7E 03 details`

Here `details` describes the system settings. 
Bytes offset 6 and 7 describe the current patch number. 


### Version request

`08 00`

Response:

`09 00 xx yy zz` 
where version = `xx`.`yy`.`zz`



### Patch Request 

```77 xx yy ```

where `xx yy` is a number 0-399, `xx` least significant. 

Response:

```78 xx yy details```

Patch data `details` has the following format. 

* Reference for Pro 600 GliGli midi at [https://www.image-et-son.com](https://www.image-et-son.com). 
* MIDI system exclusive data is 7 bits (00-7F), but Pro 800 data uses 8-bit bytes. These bytes are encoded as follows. It is a bit different from the encoding used by the GliGli Pro 600. Each sequence of seven bytes `x1 x2 x3 x4 x5 x6 x7` is _preceeded_ by a byte `x0` that specifies the MSBs of the following seven bytes. The MSB of `x0` specifies the MSB of `x7`; the LSB of `x0` specifies the MSB of `x1`, and so on.  
* Once decoded, from offset 5, the `details` seems to appear in almost the order used by GliGli. I have checked up to Amp Envelope Shape but not beyond.
* The patch name seems to start at index 150 (0x96) and continue until the first 0. The length varies. After the patch name there is sometimes further data (arpeggiator? sequencer)

#### Patch data details

| Offset | Operation | Num bytes | Mode | Steps |
| --- | --- | --- | --- | --- |
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
|65|???|1|Stepped|Off,On|
|66|LFO Target|1|Stepped|Bit mask: VCO (1) / Filter (2) / VCA (4) / PW (8) / VCOA (16) / VCOB (32)|
|67|KeyTrk|1|Stepped|Off,Half,Full|
|68|FE Shape|1|Stepped|Lin,Exp|
|69|FE Speed|1|Stepped|Fast,Slow|
|70|AE Shape|1|Stepped|Lin,Exp|
|71|NonUnison|1|Stepped|Off,On|
|72|AE Speed|1|Stepped|Fast,Slow|
|150|Patch name|??|Text, delimited by 0||

