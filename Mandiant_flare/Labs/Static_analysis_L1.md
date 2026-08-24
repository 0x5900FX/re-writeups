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



---

More detailed analysis . How it should be done.

Cff exploler. 
File info -> UPX 2.90 [LZMA] (Delphi stub) -> Markus Oberhumer, Laszlo Molnar & John Reise

Examining header -> raw size is 0 while virtual size is 0x11000

Import directory => There is only 1 import with 6,7 function import

Unpack the upx ->  or use Cff to unpack the file.


Analyzing the unpacked file.

.rsrc	000181DC	00004000	00019000  `XIN: 82.89%`  
virtual size -> 000181DC
Raw size -> 00019000 

Seems like we can extract more file from here.
Using 
Resouce editor and XIn. save raw we can get the file. Seems like it was a dll. 
Only exports the function called Solo.

Using capa to analyze this dll.

┍━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┯━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┑
│ Capability                                            │ Namespace                                            │
┝━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┿━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┥
│ check for time delay via GetTickCount (2 matches)     │ anti-analysis/anti-debugging/debugger-detection      │
│ reference anti-VM strings                             │ anti-analysis/anti-vm/vm-detection                   │
│ contain obfuscated stackstrings                       │ anti-analysis/obfuscation/string/stackstring         │
│ log keystrokes                                        │ collection/keylog                                    │
│ send data (3 matches)                                 │ communication                                        │
│ receive and write data from server to client          │ communication/c2/file-transfer                       │
│ resolve DNS (4 matches)                               │ communication/dns                                    │
│ create pipe                                           │ communication/named-pipe/create                      │
│ create two anonymous pipes                            │ communication/named-pipe/create                      │
│ read pipe                                             │ communication/named-pipe/read                        │
│ get socket information (2 matches)                    │ communication/socket                                 │
│ get socket status (4 matches)                         │ communication/socket                                 │
│ initialize Winsock library (2 matches)                │ communication/socket                                 │
│ set socket configuration (2 matches)                  │ communication/socket                                 │
│ create UDP socket                                     │ communication/socket/udp/send                        │
│ act as TCP client                                     │ communication/tcp/client                             │
│ reference Base64 string                               │ data-manipulation/encoding/base64                    │
│ encode data using XOR (3 matches)                     │ data-manipulation/encoding/xor                       │
│ read clipboard data                                   │ host-interaction/clipboard                           │
│ write clipboard data (2 matches)                      │ host-interaction/clipboard                           │
│ interact with driver via control codes (2 matches)    │ host-interaction/driver                              │
│ get common file path (2 matches)                      │ host-interaction/file-system                         │
│ get file system object information                    │ host-interaction/file-system                         │
│ create directory                                      │ host-interaction/file-system/create                  │
│ delete directory                                      │ host-interaction/file-system/delete                  │
│ delete file (4 matches)                               │ host-interaction/file-system/delete                  │
│ enumerate files recursively (2 matches)               │ host-interaction/file-system/files/list              │
│ get file size                                         │ host-interaction/file-system/meta                    │
│ read file on Windows (3 matches)                      │ host-interaction/file-system/read                    │
│ write file on Windows (4 matches)                     │ host-interaction/file-system/write                   │
│ enumerate gui resources                               │ host-interaction/gui                                 │
│ get graphical window text                             │ host-interaction/gui/window/get-text                 │
│ get CPU information                                   │ host-interaction/hardware/cpu                        │
│ get number of processors                              │ host-interaction/hardware/cpu                        │
│ simulate CTRL ALT DEL                                 │ host-interaction/hardware/keyboard                   │
│ get memory capacity                                   │ host-interaction/hardware/memory                     │
│ power down monitor                                    │ host-interaction/hardware/monitor                    │
│ get disk information                                  │ host-interaction/hardware/storage                    │
│ access the Windows event log                          │ host-interaction/log/winevt/access                   │
│ check mutex and exit                                  │ host-interaction/mutex                               │
│ get local IPv4 addresses (2 matches)                  │ host-interaction/network/address                     │
│ get hostname                                          │ host-interaction/os/hostname                         │
│ get system information on Windows                     │ host-interaction/os/info                             │
│ check OS version                                      │ host-interaction/os/version                          │
│ create a process with modified I/O handles and window │ host-interaction/process/create                      │
│ create process on Windows (2 matches)                 │ host-interaction/process/create                      │
│ enumerate processes (2 matches)                       │ host-interaction/process/list                        │
│ acquire debug privileges (5 matches)                  │ host-interaction/process/modify                      │
│ modify access privileges (3 matches)                  │ host-interaction/process/modify                      │
│ enumerate process modules                             │ host-interaction/process/modules/list                │
│ terminate process (5 matches)                         │ host-interaction/process/terminate                   │
│ query or enumerate registry key (2 matches)           │ host-interaction/registry                            │
│ delete registry key (3 matches)                       │ host-interaction/registry/delete                     │
│ delete registry value (2 matches)                     │ host-interaction/registry/delete                     │
│ query service status                                  │ host-interaction/service                             │
│ run as service                                        │ host-interaction/service                             │
│ delete service                                        │ host-interaction/service/delete                      │
│ stop service                                          │ host-interaction/service/stop                        │
│ create thread (6 matches)                             │ host-interaction/thread/create                       │
│ terminate thread (2 matches)                          │ host-interaction/thread/terminate                    │
│ overwrite Master Boot Record (MBR)                    │ impact/wipe-disk/wipe-mbr                            │
│ get kernel32 base address                             │ linking/runtime-linking                              │
│ link many functions at runtime                        │ linking/runtime-linking                              │
│ linked against ZLIB                                   │ linking/static/zlib                                  │
│ resolve function by parsing PE exports (3 matches)    │ load-code/pe                                         │
│ execute shellcode via indirect call                   │ load-code/shellcode                                  │
│ persist via Windows service (2 matches)               │ persistence/service                                  │
┕━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┷━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┙

Ultimately this malware is a dropper that writes a dll to a disk. Then it is installed as a service then it'll act as a C2 server for communication .

```