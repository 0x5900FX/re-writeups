Platform:
Unix/linux etc.

Difficulty:
1.0

Quality:
3.6

Arch:
x86-64

Author: 476f64's easyAF


```
Skipped checksec for easy programs. 

Ran strings but didn't quite get much info . Too noise


.text:000000000000121A                 call    __ZNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEC1Ev ; std::string::basic_string(void)
.text:000000000000121F                 lea     rax, [rbp+var_60]
.text:0000000000001223                 lea     rsi, aPass      ; "pass"
.text:000000000000122A                 mov     rdi, rax
.text:000000000000122D ;   try {

Here rsi is passed the variable storing "pass" as password.

call    _ZSteqIcEN9__gnu_cxx11__enable_ifIXsrSt9__is_charIT_E7__valueEbE6__typeERKNSt7__cxx1112basic_stringIS3_St11char_traitsIS3_ESaIS3_EEESE_ ; std::operator==<char>(std::string const&,std::string const&)
test    al, al
jz      short loc_129B

here it compare the two string

and based on input the code executes.
If we enter "pass" we can get over it.


Here : Skipped strings after initial noise, went straight to IDA and found the comparison directly
```