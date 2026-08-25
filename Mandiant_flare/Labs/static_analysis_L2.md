## Basic Static analysis

```
level32.exe

File:     level32.exe
Size:     970752
MD5:      2F08DC24803DD87A2C4567CE79FE4954
Compiled: Tue, Nov 20 2018, 17:24:47 - 32 Bit EXE
PDB:      C:\helloworld_\FLARELABS\branches\MACC_Training\Materials\Basic Static and Dynamic\Labs\level1\source\Level32Lab\Debug\level32.pdb

Resources: 2 - 531408 bytes

---

Is the sample packed? How can you tell?

Analyzing the content using IDE & CFF exploler We can conclude that it's not packed however is heavily encrypted.


Is there anything interesting or unique about the structure of this binary?

rcdata section  lable .rsrc: has 45.52 % which can be deemed suspicious but not surely.


How can you extract the embedded binary?

While analyzing the binary with DIE, We can see that there is some offset in Resources section. Using IDE we can extract the binary pretty much easily using extractor functionality.  

We get this info
File:     level32.exe.00_000ed000.exe
Size:     970752
MD5:      2F08DC24803DD87A2C4567CE79FE4954
Compiled: Tue, Nov 20 2018, 17:24:47 - 32 Bit EXE

Resources: 2 - 531408 bytes


List any potential host-based indicators of this malware.

Imoprt of Suscpicious libraries

AdjustTokenPrivileges
OpenProcessToken
LookupPrivilegeValueA
GetCurrentProcessId
VirtualAlloc
WriteFile
FindFirstFileExW
FindNextFileW
OpenProcess
GetCurrentProcess
WinExec
K32GetModuleBaseNameA
TerminateProcess
GetCurrentThreadId
GetCurrentThread
GetEnvironmentStringsW
SetEnvironmentVariableW
RaiseException
GetModuleHandleExW
OutputDebugStringW
SetConsoleCtrlHandler

Too many EP section
0008



List any potential network-based indicators of this malware.


No there are no potential network-based indicator of this malware.

What might this program do?
This is an XOR encrypted program. Probably a self replicating dropper. It unpacks itself on loop till memory is corrupted.


---
Analyzing the content using IDE & CFF exploler We can conclude that it's not packed however is heavily encrypted.

rcdata section  lable .rsrc: has 45.52 % which can be deemed suspicious.

Analyzing the file We can extract the raw data from rcdata using CFF exploler 
Resource editor -> save raw data -> save it


Seems like it's XOR encrypted. We gonna upload the file to cyberchef site offline version so that it's not leaaked to outside network.

the XOR enc leaks it's keys as 
 key x 00 -> Key so 
 the key there would probably be 0x80




 Inputing file and using XOR decrypt using key 0x80 we can get the MZ file or next executable.

 So disable null preserver.




 Analyzing the file with flosss we get the follwing 

 Host based indicator
 
/level1.mdt
POST
%s %d core %llu MB
host=
net=
WinNT 3.51
WinNT 4.0 
Workstation
Server Standard
Server Enterprise
Windows 
2000
XP Professional x64
Home Server


Network based indicator would be
Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.0;Trident/5.0)
evil.mandiant.com

To learn the capabilities of this executable 
we can use capa exec_name 
It's probably a payload connected to C2 server to send info about system & maybe download addtional payload .
to know more we need to know more details.


C:\Tools+>capa FLOSS\non_null.exe
┍━━━━━━━━━━━━━━━━━━━━━━━━┯━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┑
│ md5                    │ f252bb2daba1e5c57ca7e54f0beb10dd                                                   │
│ sha1                   │ 98a753d8e578b84e88fa77d881d3aa4ba29f5b86                                           │
│ sha256                 │ e087a8a1363aae6a6cccb039b1b9c86085d557a45279ac42265b383f97e2d925                   │
│ os                     │ windows                                                                            │
│ format                 │ pe                                                                                 │
│ arch                   │ i386                                                                               │
│ path                   │ C:/Tools/FLOSS/non_null.exe                                                        │
┕━━━━━━━━━━━━━━━━━━━━━━━━┷━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┙

┍━━━━━━━━━━━━━━━━━━━━━━━━┯━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┑
│ ATT&CK Tactic          │ ATT&CK Technique                                                                   │
┝━━━━━━━━━━━━━━━━━━━━━━━━┿━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┥
│ DEFENSE EVASION        │ Obfuscated Files or Information T1027                                              │
├────────────────────────┼────────────────────────────────────────────────────────────────────────────────────┤
│ DISCOVERY              │ Account Discovery T1087                                                            │
│                        │ System Information Discovery T1082                                                 │
│                        │ System Location Discovery T1614                                                    │
├────────────────────────┼────────────────────────────────────────────────────────────────────────────────────┤
│ EXECUTION              │ Command and Scripting Interpreter T1059                                            │
│                        │ Shared Modules T1129                                                               │
┕━━━━━━━━━━━━━━━━━━━━━━━━┷━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┙

┍━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┯━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┑
│ MBC Objective               │ MBC Behavior                                                                  │
┝━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┿━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┥
│ COMMAND AND CONTROL         │ C2 Communication::Send Data [B0030.001]                                       │
├─────────────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
│ COMMUNICATION               │ DNS Communication::Resolve [C0011.001]                                        │
│                             │ HTTP Communication::Connect to Server [C0002.009]                             │
│                             │ HTTP Communication::Create Request [C0002.012]                                │
│                             │ HTTP Communication::Send Request [C0002.003]                                  │
│                             │ Socket Communication::Initialize Winsock Library [C0001.009]                  │
├─────────────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
│ CRYPTOGRAPHY                │ Encrypt Data::RC4 [C0027.009]                                                 │
│                             │ Generate Pseudo-random Sequence::RC4 PRGA [C0021.004]                         │
├─────────────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
│ DATA                        │ Check String [C0019]                                                          │
├─────────────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
│ DISCOVERY                   │ Code Discovery::Enumerate PE Sections [B0046.001]                             │
│                             │ System Information Discovery [E1082]                                          │
├─────────────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
│ EXECUTION                   │ Command and Scripting Interpreter [E1059]                                     │
├─────────────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
│ FILE SYSTEM                 │ Writes File [C0052]                                                           │
├─────────────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
│ OPERATING SYSTEM            │ Environment Variable::Set Variable [C0034.001]                                │
├─────────────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
│ PROCESS                     │ Allocate Thread Local Storage [C0040]                                         │
│                             │ Set Thread Local Storage Value [C0041]                                        │
│                             │ Terminate Process [C0018]                                                     │
┕━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┷━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┙

┍━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┯━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┑
│ Capability                                           │ Namespace                                            │
┝━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┿━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┥
│ get geographical location (2 matches)                │ collection                                           │
│ parse credit card information                        │ collection/credit-card                               │
│ send data                                            │ communication                                        │
│ resolve DNS                                          │ communication/dns                                    │
│ connect to HTTP server                               │ communication/http/client                            │
│ initialize Winsock library                           │ communication/socket                                 │
│ encrypt data using RC4 PRGA (2 matches)              │ data-manipulation/encryption/rc4                     │
│ list user accounts                                   │ host-interaction/accounts                            │
│ accept command line arguments                        │ host-interaction/cli                                 │
│ query environment variable (2 matches)               │ host-interaction/environment-variable                │
│ set environment variable (3 matches)                 │ host-interaction/environment-variable                │
│ write file on Windows                                │ host-interaction/file-system/write                   │
│ get memory capacity                                  │ host-interaction/hardware/memory                     │
│ print debug messages (3 matches)                     │ host-interaction/log/debug/write-event               │
│ get hostname (2 matches)                             │ host-interaction/os/hostname                         │
│ get system information on Windows (2 matches)        │ host-interaction/os/info                             │
│ allocate thread local storage (2 matches)            │ host-interaction/process                             │
│ get thread local storage value (2 matches)           │ host-interaction/process                             │
│ set thread local storage value (2 matches)           │ host-interaction/process                             │
│ terminate process (2 matches)                        │ host-interaction/process/terminate                   │
│ link function at runtime on Windows (3 matches)      │ linking/runtime-linking                              │
│ enumerate PE sections                                │ load-code/pe                                         │
│ parse PE header (4 matches)                          │ load-code/pe                                         │
┕━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┷━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┙

```