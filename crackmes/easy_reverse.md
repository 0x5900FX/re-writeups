Platform:
Unix/linux etc.

Difficulty:
1.3

Quality:
4.7

Arch:
x86-64


Author:
cbm-hackers

```
Ran chcksec & Strings into Binary .
Didn't get quite information 

So identifying binary into IDA.
we get the following.


mov     [rbp+var_10], rsi
cmp     [rbp+var_4], 2
jnz     short loc_1257

-> compare the argument -> check if there is 2 args


mov     rax, [rax]
mov     rdi, rax        ; s
call    _strlen
cmp     rax, 0Ah
jnz     short loc_1246

check if string length is 0Ah -> 10

add     rax, 8
mov     rax, [rax] -> 0 
add     rax, 4  -> it'll be 0+4 -> 5th char
movzx   eax, byte ptr [rax]
cmp     al, 40h ; '@'
jnz     short loc_1235

check for rax[0]+4 -> arg2 [4] -> index[4] like abcde -> e is index[4]  -> 40h ? or @

mov     rax, [rax]
mov     rsi, rax
lea     rdi, aFlagS     ; "flag{%s}\n"
mov     eax, 0

prints the flag.

So the password to it would be 

abcb@ababa
Nice Job!!
flag{abcb@ababa}



Hence problem solved..

```