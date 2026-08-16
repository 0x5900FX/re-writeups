Platform: Unix/linux etc.

Difficulty: 1.5

Quality: 5.0

Arch: x86-64

Author: Yuri

Approach

Ran checksec & strings into binary. Not much useful info - no hardcoded serial, no obvious password string. This one's a real keygen, not a flat comparison.

```
Ran it once blind to see the prompt/format:

./SimpleKeyGen 1212121212121212
Good Serial

Opened in IDA, found checkSerial.

Disassembly
asm
; __int64 __fastcall checkSerial(const char *)
checkSerial proc near

our_string= qword ptr -28h
counter= dword ptr -14h

mov     [rbp+our_string], rdi
mov     rax, [rbp+our_string]
mov     rdi, rax
call    _strlen
cmp     rax, 10h
jz      short loc_11BD
mov     eax, 0FFFFFFFFh      ; fail - wrong length
jmp     short loc_121F

loc_11BD:
mov     [rbp+counter], 0
jmp     short loc_1203

loc_11C6:
mov     eax, [rbp+counter]
movsxd  rdx, eax
mov     rax, [rbp+our_string]
add     rax, rdx
movzx   eax, byte ptr [rax]
movsx   edx, al                  ; edx = str[counter]

mov     eax, [rbp+counter]
cdqe
lea     rcx, [rax+1]
mov     rax, [rbp+our_string]
add     rax, rcx
movzx   eax, byte ptr [rax]
movsx   eax, al                  ; eax = str[counter+1]

sub     edx, eax                 ; edx = str[counter] - str[counter+1]
mov     eax, edx
cmp     eax, 0FFFFFFFFh          ; -1 in two's complement
jz      short loc_11FF           ; pair passes
mov     eax, 0FFFFFFFFh          ; fail
jmp     short loc_121F

loc_11FF:
add     [rbp+counter], 2         ; advance by 2 - checks pairs, not single chars

loc_1203:
mov     eax, [rbp+counter]
movsxd  rbx, eax
mov     rax, [rbp+our_string]
mov     rdi, rax
call    _strlen
cmp     rbx, rax
jb      short loc_11C6           ; loop while counter < strlen
mov     eax, 0                   ; all pairs passed - success
Logic, in plain terms
strlen(serial) must == 16, else fail immediately

for counter = 0, 2, 4, ... while counter < 16:
    if str[counter] - str[counter+1] != -1:
        fail
return success

str[i] - str[i+1] == -1 rearranges to:

str[i+1] == str[i] + 1

So every pair of characters (positions 0-1, 2-3, 4-5, ...) must be two consecutive characters, e.g. '1','2' or 'a','b'. The pairs are independent of each other (stride 2, non-overlapping), so any pair can be reused across the whole 16-character string.

Tracing one iteration with real values

Test string: 1212121212121212, counter = 0

str[0] = '1' = 49
str[1] = '2' = 50

edx = str[0] = 49
eax = str[1] = 50
edx - eax = 49 - 50 = -1   -> pass

The key thing that wasn't obvious at first: rax holds an address, and add rax, rcx is just pointer arithmetic (address + 1). It's only movzx eax, byte ptr [rax] - the dereference - that actually reads the character sitting at that address. Easy to misread [rax+1] as "rax plus one" instead of "go to address rax+1 and read what's there."

Verified
$ ./SimpleKeyGen 1212121212121212
Good Serial
$ ./SimpleKeyGen 2323232323232323
Good Serial

Both confirm: any string of 8 repeated (X, X+1) pairs, 16 characters total, is valid.


import random
import string

def make_serial():
    chars = string.digits + string.ascii_letters
    c1 = random.choice(chars[:-1])
    c2 = chr(ord(c1) + 1)
    return (c1 + c2) * 8

print(make_serial())

```


This is my mess 
```
mov     [rbp+s], rdi
mov     rax, [rbp+s]
mov     rdi, rax        ; s
call    _strlen
cmp     rax, 10h

Check for string len 
if rax -> inp == 10h / 16 


Call _strlen
-> return currentlen -> rax 
rvx -> contain the 


s= qword ptr -28h
var_14= dword ptr -14h

s -> word reference 
rbp+s
rdi

counter = 0 


loop:

loc_1203:
mov     eax, [rbp+counter]
movsxd  rbx, eax
mov     rax, [rbp+our_string]
mov     rdi, rax        ; s
call    _strlen
cmp     rbx, rax  -> 0 < 16 ( true ) 
    iter2 ->          2 < 16
L1
jb      short loc_11C6


loc_11C6:
mov     eax, [rbp+counter]
movsxd  rdx, eax
rdx = 2

mov     rax, [rbp+our_string]
add     rax, rdx

rax = str[0] + 2
rax = str[2] 

movzx   eax, byte ptr [rax]
movsx   edx, al

edx = str[2]


mov     eax, [rbp+counter]
cdqe
eax  = 2


lea     rcx, [rax+1]
rcx = [str+1]

mov     rax, [rbp+our_string]
add     rax, rcx
rax = [str0] + str[1]
rax = [str1] + str[2]

movzx   eax, byte ptr [rax]
movsx   eax, al
eax = [str_computed]


sub     edx, eax
edx = [str0] - [str1]
mov     eax, edx
eax = [str0 - str1]
eax = [str2 - str3]

cmp     eax, 0FFFFFFFFh
cmp  [str0 - st1] , -1
cmp [str2 - str3] , -1 

if (str0 - str1)  == -1
if (str2 - str3)  == -1

jump

jz      short loc_11FF

counter = 2



---
; Attributes: bp-based frame

; __int64 __fastcall checkSerial(const char *)
public checkSerial
checkSerial proc near

our_string= qword ptr -28h
counter= dword ptr -14h

; __unwind {
push    rbp
mov     rbp, rsp
push    rbx
sub     rsp, 28h
mov     [rbp+our_string], rdi
mov     rax, [rbp-28h]
mov     rdi, rax        ; s
call    _strlen
cmp     rax, 10h
jz      short loc_11BD
mov     eax, 0FFFFFFFFh
jmp     short loc_121F

loc_11BD:
mov     [rbp+counter], 0
jmp     short loc_1203

loc_11C6:
mov     eax, [rbp+counter]
movsxd  rdx, eax
mov     rax, [rbp+our_string]
add     rax, rdx
movzx   eax, byte ptr [rax]
movsx   edx, al
mov     eax, [rbp+counter]
cdqe
lea     rcx, [rax+1]
mov     rax, [rbp+our_string]
add     rax, rcx
movzx   eax, byte ptr [rax]
movsx   eax, al
sub     edx, eax
mov     eax, edx
cmp     eax, 0FFFFFFFFh
jz      short loc_11FF
mov     eax, 0FFFFFFFFh
jmp     short loc_121F

loc_11FF:
add     [rbp+counter], 2

loc_1203:
mov     eax, [rbp+counter]
movsxd  rbx, eax
mov     rax, [rbp+our_string]
mov     rdi, rax        ; s
call    _strlen
cmp     rbx, rax
jb      short loc_11C6
mov     eax, 0

