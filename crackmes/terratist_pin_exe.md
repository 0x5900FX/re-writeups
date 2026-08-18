Platform:
Unix/linux etc.

Difficulty:
Easy / beginner

Arch:
x86-64

Author:
offlinemark


## Approach

Ran the binary blind to check usage format:

```
$ ./crackme
usage: ./crackme <secret>
$ ./crackme test
```

Ran `strings` - no obvious password sitting in plaintext, so the secret is
likely encoded/encrypted rather than hardcoded as-is.

Opened in IDA. Found `main` setting up a local buffer and calling a
`decrypt` function - not a flat `strcmp`, this one actually transforms data
before comparing.


## Disassembly - setup in main

```asm
mov     [rbp+var_27], 43594B4Fh
mov     [rbp+var_27+3], 19181B43h
lea     rdx, [rbp+var_27]
lea     rax, [rbp+Str2]
mov     r9d, 7          ; length
mov     r8d, 2Ah        ; '*'  - XOR key
mov     rcx, rax
call    decrypt
```

Two `mov` immediates write 8 bytes total into a 7-byte region, so the second
write partially overlaps and overwrites part of the first. Working out the
actual bytes sitting in memory afterward (little-endian, accounting for the
overlap):

```
43594B4Fh -> bytes 4F 4B 59 43   (occupies offset 0-3)
19181B43h -> bytes 43 1B 18 19   (occupies offset 3-6, overwrites the 43 at offset 3)

final 7 bytes: 4F 4B 59 43 1B 18 19
```


## Disassembly - inside decrypt

```asm
; args: rcx = dest (Str2), rdx = src (encrypted bytes), r8 = key byte, r9 = length

mov     [rbp+arg_0], rcx     ; arg_0 = dest address
mov     [rbp+arg_8], rdx     ; arg_8 = src address
mov     eax, r8d
mov     [rbp+arg_18], r9d    ; arg_18 = length (7)
mov     [rbp+arg_10], al     ; arg_10 = key ('*' / 0x2A)
mov     [rbp+counter], 0
jmp     short loc_check

loc_body:
mov     eax, [rbp+counter]
movsxd  rdx, eax
mov     rax, [rbp+arg_8]     ; rax = src address (fixed, doesn't move)
add     rax, rdx             ; rax = src + counter
movzx   eax, byte ptr [rax]  ; eax = src[counter]
mov     ecx, eax

mov     eax, [rbp+counter]
movsxd  rdx, eax
mov     rax, [rbp+arg_0]     ; rax = dest address (fixed)
add     rdx, rax             ; rdx = dest + counter

mov     eax, ecx
xor     al, [rbp+arg_10]     ; al = src[counter] XOR key
mov     [rdx], al            ; dest[counter] = al

add     [rbp+counter], 1

loc_check:
cmp     [rbp+counter], 7
jl      short loc_body

; after loop:
mov     rax, [rbp+arg_0]
add     rax, 7
mov     byte ptr [rax], 0    ; null-terminate the decrypted string
```


## Logic, in plain terms

```
src  = [0x4F, 0x4B, 0x59, 0x43, 0x1B, 0x18, 0x19]   (encrypted, 7 bytes)
key  = 0x2A   ('*')

for i in 0..7:
    dest[i] = src[i] XOR key
dest[7] = 0   (null terminator)
```

A straightforward per-byte XOR decryption loop - same shape as the earlier
array/loop exercises, with `xor` in place of `add`/`sub`.

The main gotcha while tracing: `arg_0` and `arg_8` are **addresses**
(pointers passed in as parameters), not the actual data. Easy to misread
`mov rax, [rbp+arg_8]` as loading the encrypted bytes directly - it only
loads the address of them. The dereference (`movzx eax, byte ptr [rax]`)
is the actual read.


## Recovering the key

```python
src = bytes([0x4F, 0x4B, 0x59, 0x43, 0x1B, 0x18, 0x19])
key = 0x2A

result = bytes(b ^ key for b in src)
print(result)
```

Output: `b'easi123'`


## Verified

```
$ ./crackme easi123
access granted!
```


## What I learned

- First encrypted-string crackme (previous ones were plaintext, hex-immediate,
  or arithmetic checks on the raw input) - had to trace an actual
  encrypt/decrypt routine, not just a comparison.
- Reinforced pointer-vs-value one level deeper than before: not just
  "`rax` is an address, `[rax]` is the value" inside a loop, but a
  *function parameter itself* can hold an address rather than the data,
  which isn't obvious until you trace where that parameter's value
  originally came from (`lea rdx, [rbp+var_27]` in `main`, not the raw bytes).
- Two `mov` immediates writing overlapping stack ranges will partially
  overwrite each other - worth manually reconstructing the final byte
  layout rather than assuming both writes land untouched.
- Single-byte XOR loops look almost identical in assembly to a plain sum/
  accumulate loop (Day 3, Yuri's keygen) - the operator changes, the loop
  skeleton doesn't.