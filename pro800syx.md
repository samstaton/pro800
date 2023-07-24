# Pro 800 System Exclusive implementation

Attempt at reconstructing by
[Sam](https://github.com/samstaton/pro800).

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

* MIDI system exclusive data is 7 bits (00-7F), but Pro 800 data uses 8-bit bytes. These bytes are encoded as follows. Each sequence of seven bytes `x1 x2 x3 x4 x5 x6 x7` is preceeded by a byte `x0` that specifies the MSBs of the following seven bytes. The MSB of `x0` specifies the MSB of `x7`; the LSB of `x0` specifies the MSB of `x1`, and so on. 
* This encoding is similar to the one used by GliGli Pro 600 but seems different. 
* Once decoded, from offset 5, the `details` seems to appear in the order used by GliGli. There are a different number of fields, and these vary by patch, but mostly the patch name is at the end. 
* Reference for GliGli midi at [https://www.image-et-son.com](https://www.image-et-son.com). 

