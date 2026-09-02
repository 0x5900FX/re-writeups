# Crackme1 by Broken

**Platform:** Windows
**Difficulty:** 1.0
**Quality:** 4.5
**Arch:** x86

---

## Stage 1 — Finding the Password

I started by looking through the binary in IDA and following the code responsible for validating the input.

The important part was a loop that reads each byte of the input and increments it by `1`.

```asm
lea     eax, [ebp+var_8]
add     eax, [ebp+counter]
lea     edx, [eax-20h]

lea     eax, [ebp+var_8]
add     eax, [ebp+counter]
sub     eax, 20h

movzx   eax, byte ptr [eax]
inc     al
mov     [edx], al

lea     eax, [ebp+counter]
inc     dword ptr [eax]
jmp     short loc_4013D0
```

The important instruction here is:

```asm
inc al
```

So, effectively, every character is transformed like:

```text
a -> b
b -> c
1 -> 2
```

The program then compares the processed string against:

```text
QbTTx1sE
```

The comparison happens here:

```asm
lea     eax, [ebp+var_38]
lea     edx, [ebp+Str]

mov     [esp+78h+Str2], eax
mov     [esp+78h+Format], edx
call    _strcmp

test    eax, eax
jnz     loc_4014F5
```

So the program is essentially doing:

```text
process(user_input) == "QbTTx1sE"
```

Since every character is incremented by `1`, we simply reverse the operation by decrementing every character:

```text
Q -> P
b -> a
T -> S
T -> S
x -> w
1 -> 0
s -> r
E -> D
```

Therefore the original password is:

```text
PaSSw0rD
```

Entering:

```text
PaSSw0rD
```

gets us past Stage 1.

---

# Stage 2 — Understanding the Serial Check

After passing the password check, the program asks for a name and a serial number.

The next interesting section starts with a length check:

```asm
loc_401480:
lea     eax, [ebp+var_48]
mov     [esp+78h+Format], eax
call    _strlen

cmp     [ebp+counter], eax
ja      short loc_4014AA
```

`var_48` contains the user-provided name.

The loop uses `counter` to process each character of the name.

The relevant transformation is:

```asm
lea     eax, [ebp+var_8]
add     eax, [ebp+counter]
sub     eax, 40h

movsx   eax, byte ptr [eax]
add     eax, [ebp+var_4C]
dec     eax
mov     [ebp+var_4C], eax

lea     eax, [ebp+counter]
inc     dword ptr [eax]

jmp     short loc_401480
```

The important part is:

```asm
movsx   eax, byte ptr [eax]
add     eax, [ebp+var_4C]
dec     eax
mov     [ebp+var_4C], eax
```

So the program takes a character from the name, adds the current value of `var_4C`, subtracts `1`, and stores the result back into `var_4C`.

In pseudocode, the logic is roughly:

```c
counter = 0;
var_4C = 0;

while (counter <= strlen(name)) {
    var_4C = name[counter] + var_4C - 1;
    counter++;
}
```

The exact surrounding memory layout makes the IDA variables look somewhat confusing, but the important observation is that the value is accumulated across the characters rather than treating every character independently.

For example, with a name such as:

```text
cyan
```

the calculation evolves character by character:

```text
initial = 0

c -> c + 0 - 1
y -> y + previous - 1
a -> a + previous - 1
n -> n + previous - 1
```

This gives us the value used by the later serial-number validation.

The key takeaway from Stage 2 is that the serial isn't simply another hardcoded string. It is derived from the supplied name using the accumulator stored in `var_4C`.

---

# Stage 3 — Removing the Console Nag

After completing the serial check, the program displays the Stage 3 message and a small console nag:

```text
Stage 2 Completed!

STAGE 3

Stage 3 Completed if you don't see nag...

Console nag... lol ...Remove Me
```

At this point there wasn't really another meaningful reverse-engineering challenge. The goal was simply to patch the binary so that the nag was no longer printed.

I located the `printf` call responsible for displaying the message and patched the call.

Using IDA's assembler/patching functionality, I replaced the relevant instruction with `NOP`s.

After applying the patch, the program no longer prints the console nag.

This completes the challenge.

---

# Summary

The challenge turned out to be a straightforward x86 reversing exercise with three parts:

### Stage 1

The program increments every password character by `1` and compares the result against:

```text
QbTTx1sE
```

Reversing the transformation gives:

```text
PaSSw0rD
```

### Stage 2

The serial calculation uses the characters of the supplied name and an accumulator:

```c
var_4C = name[counter] + var_4C - 1;
```

The resulting value is then used by the serial validation.

### Stage 3

The final "Console nag" was simply a `printf` call. Patching the call with `NOP`s removes the message.

Overall, this was a nice beginner-friendly Windows x86 reversing challenge: the first stage teaches basic string transformation, the second introduces a simple accumulator-based calculation, and the final stage demonstrates basic binary patching.



Focusing on Exam now. WIll be updating soon. RE is fun.
Learning Asm & customizing my terminal using bash for now.