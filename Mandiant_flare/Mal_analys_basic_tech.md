Notes For Mandiant

Malware Analysis Fundamentals

By FLARE


## Malware Analysis

```
Disecting malicious software to understand 
How it Works , identify & eliminate it

IOC:

```

---

```
Host-based indicators (HBIs) describe artifacts found on a host that identify malicious activity.
Unique about samples like :

o File characteristics – size, hashes, names
o Characteristics unique to the binary – strings, PDB paths
o Changes made to the system – registry keys, created files, created directories
o Other changes made to the system – named mutexes, started processes


File System:
Persistance
Drop configuration files
Store infomation from system

o %APPDATA%\updatesvc.exe
o C:\Windows\System32\kernel32.dll

HBIs – Registry Paths/Keys

• The Windows registry stores configuration data for the system and its applications

Eg:
o HKEY_CURRENT_USER\Microsoft\Windows\CurrentVersion\Run
o HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services


HBIs - Mutex

An operating system construct that is designed to synchronize access to a resource.
Used by malware to prevent multiple instances of itself executing at the same time

o Global\
o Local\

o Global\4cafb85112364d776a04862aaa4371a0


```

---

```
Network Based indicators

Communication With C2 server .
Domain & IP addrs
Protocols & ports used
Headers used 
Unique signatures, patterns, or data structures

Network Communication

To locate the server, the malware uses either:
o Domain name - example.com
o IP address - 192.168.0.1

NBIs – HTTP Headers
The HTTP User-Agent is a string that identifies various details that may include:
o Browser type – (Firefox, Chrome, Safari, etc.)
o Version
o Operating system
o Architecture

o Mozilla/5.0 (Windows NT 6.1; WOW64; rv:40.0) Gecko/20100101 Firefox/40.1


```
---


```
Basic Analysis

o Basic Static Analysis – examining an executable file without viewing the actual instructions


o Basic Dynamic Analysis – observing malware behavior in a controlled environment

```

## Lesson 2: Basic Static Analysis

```
Extracrting meaningful characterstics from an unknown binary without execution.

- Hashes
- Strings 
- OSINT
- PE File formats
- Packing 

Hashing :
Generate a unique value to identify a file.
Any small cheange will lead to chagne in hashvalue of a file.
one-way cryptographic function - >   | inp -> hash |

Most widely accepted algorithm -> SHA-256
MD5  , SHA-1   are cryptographically broken now but sometime used as checksums values
Vendors continue to track malware samples by their MD5 hash value

Hashing tools
- hashmyfiles
- sigcheck.exe (sysinternals)
- Ctf exploler & PE analysis value

Examples:
Sysinternals - www.sysinternals.com

c:\users\flare\documents\malware samples\eb0d18828cbd76d92a2577259a0946a40bc93b251f782c00e8cb59236d5f7953\FlawedAmmyy.exe:
        Verified:       Unsigned
        Link date:      8:07 AM 9/3/2004
        Publisher:      n/a
        Company:        n/a
        Description:    n/a
        Product:        n/a
        Prod version:   n/a
        File version:   n/a
        MachineType:    32-bit



Strings
Compiled binaries contain sequences of human-readable characters

We can get the followings:
o Filenames
o Registry paths/keys
o PDB strings
o Service configuration info
o HTTP User-Agent strings
o Domain names, IP addresses, URLs
o Command-line help and usage options
o Debugging messages
o Function names
o Third-party software libraries (OpenSSL, zlib)
o Keylogger-related strings (e.g., "[DELETE]", "[BS]", "[SHIFT]")


Example - Strings
• Filenames ------------malware.dll
• Registry paths/keys SOFTWARE\Microsoft\Windows\CurrentVersion\Run
• PDB strings E: \windows \dropperNew\Debug\ testShellcode . pdb
• Domain names, IP addresses, URLs evil. com, 192 . 168. 0. 2, evil. com/payload. exe
• Command-line help and usage options Usage: evil. exe host port
• Debugging messages Error: Unable to download file
• Function names----------encrypt_payload
• Third-party software libraries (OpenSSL, zlib) - MDS part of OpenSSL 1. 0. 2q 20 Nov 2018
• Keylogger-related strings------ [DELETE], [BS] , [SHIFT]

Narrow string is 0ne bytes
Wide string is 2 bytes ( used by windows and its encoding standard is UTF-16 LE )
Narrow string ends with 0x00 while wide one are terminated with double NULL (0x00, 0x00)
Microsoft’s encoding standard is UTF-16 LE


Tools
strings.exe (Sysinternals)
/usr/bin/strings (Linux)

Malware authors routinely encrypt, obfuscate, or encode strings that have forensic significance to
investigators
• Common encoding methods:
o Hexadecimal
o XOR
o Base64

```

### Encoding
```
Encoding – Hexadecimal

binary-to-text encoding where each byte is represented by two hexadecimal digits
Hexadecimal digits: 0123456789ABCDEF (not case sensitive) (HEX)
Example for GET req using HEX

GET  /chk?75 73 65 72 6E 61 6D 65
          u  s  e  r  n  a  m  e       

Can use ASCII to view the ascii table for easy review / comparision.


Encoding - Base64


Binary-to-text encoding scheme where data is represented using 64 printable characters. 
o Alphabet: ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/
o Uses the character '=' to pad the end of strings
Can easily identify Base64 enc by looking out for string ending with  "=" or "==".
Used mainly in http / smtp protocols.
JS & powershell script malicious are usually in base64 format.
R1JFQVQgRVhBTVBMRQ==



Encoding – XOR


• A binary logic operation commonly used by malware to obfuscate data
Equ  to "Either-or but not both" in a single bit
Used in cryptographic algorithms . -> Reversible
 ' ^ ' used for XOR operation.
Key can be single / multiple bytes.

unlike prev this enc can produce XOR data.

data key encoded
D   K   E
0 ^ 0 = 0 
0 ^ 1 = 1
1 ^ 0 = 1
1 ^ 1 = 0

org = key ^ enc
0 = 0 ^ 0 
0 = 1 ^ 1 
1 = 0 ^ 0 
1 = 1 ^ 1 

XOR key leakage

Any byte XORed with zero is equal to byte  ( x ^  00 = X)
Byte XORed with itself is equal to zero ( X ^ X == 0)
Most files contain blocks of null (zero) bytes that can reveal the key

CyberChef 

useful for common data transformation using drag & drop
o Supports common data encoding and encryption schemes


Can perform the followings  


CyberChef Tips
Data type conversion
• From Hex / To Hex – Convert data to/from hex and ASCII
• To Hexdump – Display hex value of data with ASCII interpretation
• Decode Text – Convert character encoding

Encoding/Decoding
• From Base64 / To Base64
• XOR / XOR Brute Force


Text manipulation
• Split – Separate data based on delimiter
• Find/Replace – Replace (or remove) repeated data values
• Remove Whitespace – Eliminate new lines, tabs, spaces

FLOSS – FLARE Obfuscated String Solver
Exposes encrypted / encoded strings
Utilizes heuristics and emulation
floss evil.exe > floss_output.txt

C:\Users\flare\Desktop>C:\Tools\FLOSS\floss.exe "C:\Users\flare\Documents\Malware Samples\eb0d18828cbd76d92a2577259a0946a40bc93b251f782c00e8cb59236d5f7953\FlawedAmmyy.exe" > flawed_text.txt
INFO: floss: extracting static strings
finding decoding function features: 100%|███████████████████| 5/5 [00:00<?, ? functions/s, skipped 0 library functions]
INFO: floss.stackstrings: extracting stackstrings from 5 functions
extracting stackstrings: 100%|███████████████████████████████████████████████████| 5/5 [00:00<00:00, 80.27 functions/s]
INFO: floss.tightstrings: extracting tightstrings from 0 functions...
extracting tightstrings: 0 functions [00:00, ? functions/s]
INFO: floss.string_decoder: decoding strings
emulating function 0x402000 (call 1/1): 100%|████████████████████████████████████| 5/5 [00:00<00:00, 63.96 functions/s]
INFO: floss: finished execution after 4.30 seconds
INFO: floss: rendering results


```



Open-Source Intelligence
```
VirusTotal
Can be a valuable source of information for investigators
But also Malware authors are known to use VT to test their malware builds

OPSEC
VT tracks where samples are uploaded from
Always start with the MD5 lookup feature
Offers a public (free) and private (paid) API

Google
o Unique strings
o Hashes
o Malware family
```