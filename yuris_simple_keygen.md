



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

rax = str[0] + 1 

movzx   eax, byte ptr [rax]
movsx   edx, al

edx = str[0]

mov     eax, [rbp+counter]
cdqe
eax =  0


lea     rcx, [rax+1]
rcx = [str+1]

mov     rax, [rbp+our_string]
add     rax, rcx
rax = [str0] + str[1]

movzx   eax, byte ptr [rax]
movsx   eax, al
eax = [str_computed]


sub     edx, eax
edx = [str0] - [str1]
mov     eax, edx
eax = [str0 - str1]

cmp     eax, 0FFFFFFFFh
cmp  [str0 - st1] , -1

if (str0 - str1)  == -1
jump

jz      short loc_11FF

counter = 2