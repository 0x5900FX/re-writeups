## Binary info


Platform:
Unix/linux etc.

Difficulty:
1.0

Quality:
4.5

Arch:
x86-64


## Approach 

Ran checksec


```
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Full RELRO      Canary found      NX enabled    PIE enabled     No RPATH   No RUNPATH   40 Symbols	  No	0		1	
```

Execute it on terminal

Ask for password
tried random pass. 
couldn't do it.

Strings into the binary
got multiple strings

----
Enter password: 
%41s
iloveicecream
I love ice cream too!
Wrong try again.
----

This caught my eyes
Tried "iloveicecream" 
Well it was it


Disassebmling using IDA


lea     rdx, s2         ; "iloveicecream"  -> loads into rdx
mov     rsi, rdx        ; s2 
mov     rdi, rax        ; s1
call    _strcmp 


Here 
rsi hold hardcoded pass
rdi hold out input pass

then based on cmp the next logic is set.



Technique: plaintext hardcoded password, no obfuscation — found via strings before disassembly was even needed

```