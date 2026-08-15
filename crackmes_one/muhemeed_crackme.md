# Crackeme : muhemed crackme

## Binary info
Language:
C/C++
Platform:
Unix/linux etc.
Arch:
x86-64


## Approach

Ran this program into terminal. 
See if we can find any hints

Ran checksec
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Partial RELRO   No canary found   NX enabled    PIE enabled     No RPATH   No RUNPATH   39 Symbols	  No	0	1crackme

Opened the binary into IDA - Pro

Reading the binary
.text:0000000000001174                 mov     rax, 58384E58686F7677h
.text:000000000000117E                 mov     rdx, 3171726A34314337h
.text:0000000000001188                 mov     qword ptr [rbp+key], rax
.text:000000000000118F                 mov     qword ptr [rbp+key+8], rdx
.text:0000000000001196                 mov     qword ptr [rbp+key+10h], 6A212A46h

We were able to get the values copyinh into key 

58 38 4E 58 68 6F 76 77h
X8NXhovw

31 71 72 6A 34 31 43 37h
1qrj41C7

6A 21 2A 46h
j!*F

little endianess by revesing the order 
We get -> wvohXN8X7C14jrq1F*!j


using this passcode to binary
Completed the challenge