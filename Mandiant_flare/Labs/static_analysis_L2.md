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

```