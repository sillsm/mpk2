# Sysex Description for AKAI MPK 2 series
This is a collection of notes and tools from reverse engineering the AKAI MPK261 controller.
I don't have access to other 2 series controllers, but I wouldn't be surprised if things were similar...

_Disclaimer:_ Use of this material can possibly void your warranty, brick your controller, or both!

_License:_ Do whatever you want, but include a link to https://github.com/nsmith-/mpk2

### Installation
To use python script(s), you will need `rtmidi`.  On my OS X machine, the following was sufficient:
```sh
brew install rtmidi
pip install python-rtmidi
```

### Previous work
http://practicalusage.com/akai-mpk261-mpk2-series-controlling-the-controller-with-sysex/

http://practicalusage.com/akai-mpk261-one-more-thing/

http://www.akaipro.com/files/product_downloads/MPK2_Series_Bitwig_Scripts_v1.0.8.zip


### Sysex format
All numbers below in hex
```
akai sysex header
f0 47 00

mpk261 byte
25

command byte
10 : program dump/load (presumably to non-volatile memory)
20 : query address in global memory (payload = address)
21 : query address in program memory (payload = address)

30 : load data at global address (payload = address, data)
30 00 [n+3] nn xx xx yy : load global address x with n bytes y
returns 38 00 04 nn xx xx zz, zz = 0x00 on success, 0x01 on failure it seems

31 : load data at program address (payload = address, data)
31 00 [n+3] nn xx xx {yy[0]..yy[n-1]} : load program address x with n byte array yy
return 39 00 04 nn xx xx 00 on success (01 on fail?)

31 : response from controller after command 21 [learned]
38 : response from controller after command 30
39 : response from controller after command 31

2 byte payload size (MSB of low byte packed in high byte)

payload

sysex end
f7
```

### Examples
Omitting the `f0 47 00 25` header and `f7` trailer
```
program dump yy (1-30) (Note 0c 0b = 1547 after MSB unpack)
10 0c 0b yy {1546 bytes..}

query address 0x18
20 00 03 01 00 18 f7
returns: 30 00 04 01 00 18 yy

query n bytes at address
20 00 03 nn 00 18 f7
returns 30 00 03 nn 00 18 {xx bytes} f7

load program address x with data byte y
31 00 04 01 xx xx yy

load address x with 64 bytes yy (thanks bitwig scripts! :)
31 00 43 40 xx xx {yy[0] .. yy[63]}

```

### Extras
[0x01, 0x00, nn] + [xx], where xx a message of length nn, which seems to overwrite the screen.

[i]+[0x00, 0x00], where i ranges from 0 255. This corrupted the firmware!

See https://www.akaipro.de/sites/default/files/2018-01/APC40_Communications_Protocol_rev_1.pdf_1db97c1fdba23bacf47df0f9bf64e913.pdf.
MPK249 demonstrates accepts "MMC Device Enquiry message"
[0xF0, 0x7E, 0x00, 0x06, 0x01, 0xF7]

Flip through pages on LCD
[0x30, 0, 4, 1, 0, 0x1E] [xx] 1-30 mode

### Memory Addresses
Packed (unpacked) address
```
00 01 (0x001) = current program #
00 18 (0x018) = current bank # (0-3)

Pad off colors:
0a 7c (0x57c) = bank a pad 1 (first pad)
0b 1b (0x59b) = bank b pad 16

Pad on colors:
0b 3c (0x5bc) = bank a pad 1
```

### Program dump format:
See readProgramDump.py comments for class `MPK2Sysex`, in particular, `read*Spec()` functions.

### More

Connecting to the AD2A/4A/6A/8A via a st-link v2 through the jtag pins.
[link](https://stm32-base.org/guides/connecting-your-debugger.html)

https://medium.com/techmaker/stack-buffer-overflow-in-stm32-b73fa3f0bf23

https://payatu.com/blog/asmita-jha/--stack-smashing--protection-in-hardware-attack
