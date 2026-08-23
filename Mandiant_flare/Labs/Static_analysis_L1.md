## Basic Static Analysis 
Lab - shadyrabbit.exe 


Static Analysis

File:     shadyrabbit.exe
Size:     56497
MD5:      CD2CBA9E6313E8DF2C1273593E649682
Compiled: Wed, Feb 23 2011, 17:31:59 - 32 Bit EXE
Version:  1, 0, 0, 1

Resources: 2 - 956 bytes

---

```
Is the sample packed? How can you tell?


Using CFF tools we can see that it's UPX packed file 

[Results of File Scan]


Best Match: UPX 2.90 [LZMA] (Delphi stub) -> Markus Oberhumer, Laszlo Molnar & John Reise

All Matches:

Signature: UPX 2.90 [LZMA] (Delphi stub) -> Markus Oberhumer, Laszlo Molnar & John Reise
Matches: 55

Signature: UPX v0.89.6 - v1.02 / v1.05 - v1.22
Matches: 43

Signature: UPX v3.0
Matches: 7

And we can also unpack it using CFF Exlpoler


Is there anything interesting or unique about the structure of
this PE?

analyzing the Unpacked file using PEStudio we can conclude that file entropy is really suspicious here.
resources > file-ratio : The file ratio for `XIN: 82.89%` Which is highly suspicious.


Can you identify any potential host-based indicators of this sample?
Here are some:

Import of the following librarires:
ChangeServiceConfigA
CreateServiceA
RegSetValueExA
RegCreateKeyExA
RegDeleteKeyA
RegDeleteValueA
AddIPAddress
11 (inet_addr)
WriteFile

The file named is ( IP Helper API,1 )

Strings contains so much suspicious imports/data
00018298	07	A	strncat
000182a2	07	A	realloc
000182ac	08	A	wcstombs
000182b8	0e	A	_beginthreadex
000182ca	06	A	calloc
000182d2	0a	A	MSVCRT.dll
000182e0	14	A	??1type_info@@UAE@XZ
000182f8	09	A	_initterm
00018304	0c	A	_adjust_fdiv
00018312	09	A	WINMM.dll
0001831c	0a	A	WS2_32.dll
0001832a	4c	A	?_Tidy@?$basic_string@DU?$char_traits@D@std@@V?$allocator@D@2@@std@@AAEX_N@Z
0001837a	59	A	?_C@?1??_Nullstr@?$basic_string@DU?$char_traits@D@std@@V?$allocator@D@2@@std@@CAPBDXZ@4DB
000183d6	46	A	??1?$basic_string@DU?$char_traits@D@std@@V?$allocator@D@2@@std@@QAE@XZ
00018420	54	A	?assign@?$basic_string@DU?$char_traits@D@std@@V?$allocator@D@2@@std@@QAEAAV12@PBDI@Z
00018478	4e	A	?_Grow@?$basic_string@DU?$char_traits@D@std@@V?$allocator@D@2@@std@@AAE_NI_N@Z

Can you identify any potential network-based indicators from
this sample?

Yes It has clear indicator there is some network based acitivity going off in the background.

00019050	0e	A	OpenSCManagerA
00019068	07	A	connect
00019070	0b	A	getpeername
0001907c	06	A	accept
00019084	06	A	netsil
0001908c	08	A	CONNECT 
00019098	05	A	POST 
000190a0	05	A	HEAD 
000190b0	07	A	http://
000190bc	0a	A	WSAStartup
000190cc	0b	A	0oveFileExA
000190d8	27	A	%s:\Documents and Settings\Local Server
00019104	0c	A	winlogon.exe
00019114	12	A	%2d%2d%2d%2d%2d%2d
00019128	24	A	taskkill /f /t /im LiveUpdate36O.exe
00019150	0a	A	REG_BINARY
000191f8	08	A	\CMD.EXE
00019204	0f	A	GetStartupInfoA
00019214	0f	A	0erminateThread
00019224	40	A	ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/
0001926b	05	A	roup1
000192a8	0d	A	InternetOpenA
000192c0	18	A	Mozilla/4.0 (compatible)
000192dc	08	A	https://

What might this program (shadyrabbit) do?

This is probably a dropper and this malware tries to conenct to the server for C2 using RDP configuration manipulation/potential remote-access functionality.


```