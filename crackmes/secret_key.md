Platform:
Windows

Difficulty:
1.0

Quality:
4.2

Arch:
x86

Author:
RR7


## Approach

Ran the binary blind - prompts for a number, then a secret key.

Opened in IDA. Straightforward flat comparisons, no obfuscation.


## Disassembly

```asm
lea     rax, _Format         ; "Enter Your Number : "
mov     rcx, rax
call    printf
lea     rax, [rbp+var_4]
lea     rcx, aI              ; "%i"
mov     rdx, rax
call    scanf
mov     eax, [rbp+var_4]
cmp     eax, 21h             ; IDA shows '!' but format is %i - this is decimal 33
jnz     short loc_13F581520

lea     rax, aEnterTheSecret ; "Enter The Secret Key : "
mov     rcx, rax
call    printf
lea     rax, [rbp+var_8]
lea     rcx, aI              ; "%i"
mov     rdx, rax
call    scanf
mov     eax, [rbp+var_8]
cmp     eax, 66h             ; IDA shows 'f' but format is %i - this is decimal 102
jnz     short loc_13F581520

lea     rax, aCongratulation ; "Congratulations, you have completed the..."
mov     rcx, rax
call    printf
```


## Logic

```
number == 33   (0x21)
secret == 102  (0x66)
```

The one thing worth catching here: both `scanf` calls use format `"%i"`,
meaning input is parsed as an **integer**, not a character. IDA
auto-annotates any hex immediate that matches a printable ASCII code with
that character in a comment (`21h ; '!'`, `66h ; 'f'`) regardless of how
the value is actually used. Since these are compared against `%i`-parsed
input, the correct answers are the plain decimal numbers (33, 102), not
the letters `!`/`f`.


## Verified

Ran in FlareVM. Console window closed before the success message was
visible (no pause/`getch()` at the end of the program), so confirmed via
IDA's remote debugger with breakpoints on the comparison checks and the
final `printf` call - hit "Congratulations, you have completed..." with
inputs `33` then `102`.


## What I learned

- IDA's auto-comment on hex immediates (showing the ASCII character) is
  just a convenience annotation, not a claim about how the value is
  actually used - always check the format string / data type context
  before trusting it.
- First console app without a pause at the end - worth having a debugger
  breakpoint ready as a fallback when output disappears before you can
  read it, rather than assuming the program failed.