# Reverse Engineering / Crackme Write-ups
 
Personal log of crackme solves and reverse engineering practice, working toward
malware analysis and Flare-On 2026.
 
Tools: IDA Pro, x64dbg, REMnux, FlareVM, Python.
 
> **Note on spoilers:** methodology and reasoning are shown in full; final
> passwords/flags/keys are redacted per crackmes.one etiquette. Process is
> shareable, answers are not.
 
## Progress log
 
| # | Name | Source | Difficulty | Technique | Write-up |
|---|------|--------|------------|-----------|----------|
| 1 | muhemed crackme | crackmes.one | — | Password hardcoded as raw hex immediates (`mov reg, <hex>`), decoded via little-endian byte reversal | [crackmes/muhemed_crackme.md](crackmes/muhemed_crackme.md) |
| 2 | [name TBD] | crackmes.one | 1.0 | Plaintext password recoverable via `strings`, confirmed via `strcmp` in disassembly | [crackmes/name_tbd.md](crackmes/name_tbd.md) |
| 3 | easyAF | crackmes.one (476f64) | 1.0 | `std::string::operator==` comparison against a literal | [crackmes/easyaf_476f64.md](crackmes/easyaf_476f64.md) |
| 4 | easy_reverse | crackmes.one (476f64) | 1.0 | `Requires check to pass then get flag. Need to ensure that it's fullfilled` | [crackmes/easyaf_476f64.md](crackmes/easyaf_476f64.md) |
 
## Repo structure
 
```
re-writeups/
├── README.md
└── crackmes/
    ├── muhemed_crackme.md
    ├── really_easy_elzooms.md
    ├── easy_reverse.md
    └── easyaf_476f64.md

```
 
## Skills exercised so far
 
- Static analysis workflow: `checksec` → `strings` → IDA (decompiler + raw disassembly)
- Recognizing hardcoded secrets embedded as immediate values vs. `.rodata` string references
- Little-endian byte-order decoding by hand
- Reading C++ name-mangled symbols and `std::string`/iostream call patterns in disassembly
- Locating win/fail branches via `cmp`/`test` + conditional jump tracing
## C fundamentals (source-to-assembly round-trip practice)
 
Parallel practice compiling small C programs at `-O0` and manually tracing the
resulting disassembly in IDA, to build fluency reading compiler-generated code
before tackling unlabeled binaries.
 
Covered: variables/arithmetic, conditionals/loops, arrays/pointers,
strings/buffers (incl. a hands-on stack buffer overflow + stack canary
demonstration), structs and memory alignment, function pointers /indirect
calls, and heap allocation.
 
## Up next
 
- More easy-tier crackmes (plain C preferred over C++ while still building speed)
- Algorithm-recognition practice (XOR, base64, checksums) ahead of Flare-On 2026
- Flare-On 2026 (starts ~Sept 24)
 