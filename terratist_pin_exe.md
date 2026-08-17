Solving this

```call    __main     
mov     [rbp+var_27], 43594B4Fh     →  CYKO
mov     [rbp+var_27+3], 19181B43h → C
lea     rdx, [rbp+var_27]  
lea     rax, [rbp+Str2] 
mov     r9d, 7
mov     r8d, 2Ah ; '*'
mov     rcx, rax
call    decrypt

Decrypt

var_4= dword ptr -4  → 0 ( counter prob )
arg_0= qword ptr  10h  → 19181B43h
arg_8= qword ptr  18h → 43594B4Fh
arg_10= byte ptr  20h  → ‘*’
arg_18= dword ptr  28h → 7

mov     [rbp+arg_0], rcx 
mov     [rbp+arg_8], rdx
mov     eax, r8d   → ‘*’
mov     [rbp+arg_18], r9d  
mov     [rbp+arg_10], al
mov     [rbp+var_4], 0
jmp     short loc_1400014AD

eax  → 0

cmp  0  , 7 ( jl ) so cnt

0 1 2 3 4 5 6 7 8

eax → 0

rdx → 0

rax → 43594B4Fh

eax → 43h

ecx → 43h

mov     eax, [rbp+var_4] → [addr 0]

rdx → 0

eax → 43h

xor 43h , ‘*’

loop into the 

loc_140001483:
mov     eax, [rbp+counter]   → eax  → 0
movsxd  rdx, eax   → rdx → 0
mov     rax, [rbp+arg_8] → rax → 1st_str
add     rax, rdx → rax → 1st_str
movzx   eax, byte ptr [rax] - > eax → [str1[0]] 
mov     ecx, eax  → ecx → [str1[0]]
mov     eax, [rbp+counter]  → eax → 0 
movsxd  rdx, eax  → rdx - > 0 
mov     rax, [rbp+arg_0] → [str2]
add     rdx, rax → rdx →  [str2 + 0] 
mov     eax, ecx → eax → [str2]
xor     al, [rbp+arg_10] → [str2[0]]^’*’
mov     [rdx], al → [rdx] →  [str2+counter] → al
add     [rbp+counter], 1  → counter  = 1

loc_140001483:
mov     eax, [rbp+counter]   → eax  → 1
movsxd  rdx, eax   → rdx → 1
mov     rax, [rbp+arg_8] → rax → 1st_str
add     rax, rdx → rax → [1st_str + 1]
movzx   eax, byte ptr [rax] - > eax → [str1[1]] 
mov     ecx, eax  → ecx → [str1[1]]
mov     eax, [rbp+counter]  → eax → 1 
movsxd  rdx, eax  → rdx - > 1
mov     rax, [rbp+arg_0] → [str2]
add     rdx, rax → rdx →  [str2 + 1] 
mov     eax, ecx → eax → [str2]
xor     al, [rbp+arg_10] → [str2[1]]^’*’
mov     [rdx], al → [rdx] → [str2 + counter]
add     [rbp+counter], 1  → counter  = 2

. . . .  → counter till 6

counter = 7 

mov     eax, [rbp+_7] → eax → 7
movsxd  rdx, eax  →  7 
mov     rax, [rbp+arg_0]  → rax → [str1]
add     rax, rdx → [str1 [7]]
mov     byte ptr [rax], 0 → str1[7] → 0
nop 
add     rsp, 10h  
pop     rbp
retn
decrypt endp

xxxxxxxx0

43 59 4B 4Fh 

C Y K O 
```

```python
str1 = bytes([0x4F , 0x4B , 0x59, 0x43 , 0x1b , 0x18 , 0x19])
key = 0x2A

str2 = bytes(b ^ key for b in str1)

print(str2)
```

Here we get the key now