## Basic Static Analysis 
Lab - shadyrabbit.exe 


Static Analysis

File:     Shadyrabbit.exe
Size:     441899
MD5:      FBBDC39AF1139AEBBA4DA004475E8839
Compiled: Sun, Oct 22 2017, 2:33:58 - 32 Bit EXE
Version:  27,0,0,170

Resources: 10 - 28808 bytes
Signature Corrupt
Subject:  Symantec Corporation
Issuer:   VeriSign Class 3 Code Signing 2010 CA

---

```
Is the sample packed? How can you tell?

upx: C:\Users\flare\Desktop\P_labs\Shadyrabbit.exe: NotPackedException: not packed by UPX

The sample does not appear to be packed with UPX, as upx -d returns NotPackedException. However, DIE reports high entropy in the .text and .rdata sections and especially the large overlay entropy of 7.99 suggesting that portions of the executable may be compressed, encrypted, or packed using a non-UPX technique.

Offset	Size	Entropy	Status	Name
00000000	00000400	2.47377	not packed	PE Header
00000400	00003000	6.58422	packed	Section(0)['.text']
00003400	00003200	7.17737	packed	Section(1)['.rdata']
00006600	00000200	0.18616	not packed	Section(2)['.data']
00006800	00007200	4.20414	not packed	Section(3)['.rsrc']
0000da00	00000400	3.29455	not packed	Section(4)['.reloc']
0000de00	0005e02b	7.99750	packed	Overlay

PS C:\Tools > .\upx\upx-4.2.1-win64\upx.exe -d C:\Users\flare\Desktop\P_labs\Shadyrabbit.exe > C:\Users\flare\Desktop\P_labs\unpacked.exe
upx: C:\Users\flare\Desktop\P_labs\Shadyrabbit.exe: NotPackedException: not packed by UPX
Probably packed / encoded by other tools


Is there anything interesting or unique about the structure of
this PE?

Offset	Size	Entropy	Status	Name
00000000	00000400	2.47377	not packed	PE Header
00000400	00003000	6.58422	packed	Section(0)['.text']
00003400	00003200	7.17737	packed	Section(1)['.rdata'] 
00006600	00000200	0.18616	not packed	Section(2)['.data']
00006800	00007200	4.20414	not packed	Section(3)['.rsrc']
0000da00	00000400	3.29455	not packed	Section(4)['.reloc']
0000de00	0005e02b	7.99750	packed	Overlay

-> this .rdata section is healily packed . 


Can you identify any potential host-based indicators of this
sample?
Running strings on this exe provides us with crucial information on this virus
Host based indicator would be use of this this library & functions.
It can be used to read/write & create persistence in device.

CreateFileW → capability to access/create files
WriteFile → capability to write files
CreateProcessW → capability to execute another process

FlashUtil.exe —> Use of Adobe Flash here.
Adobe Flash Player Installer/Uninstaller 27.0 —> claimed file/product identity.
27.0.0.170 — >claimed version.
requestedExecutionLevel="highestAvailable" -> executable requests highest available privileges.


Can you identify any potential network-based indicators from
this sample?

No clear host based indicator were identified. The url has some linkes but they are associated with CA authority.
NO suspicious IP/ network resources were identified. 


What might this program (shadyrabbit) do?

Probably a dropper. Dropping adobe flash to our system & execute c2.
```