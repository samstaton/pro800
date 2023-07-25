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
* Once decoded, from offset 5, the `details` seems to appear in exactly the order used by GliGli up to Amp Envelope Shape. Then Unison comes before Amp Envelope Speed. I have not looked beyond there yet. 
* The patch name seems to start at index 150 (0x96) and continue until the first 0. The length varies. After the patch name there is sometimes further data (arpeggiator? sequencer)


