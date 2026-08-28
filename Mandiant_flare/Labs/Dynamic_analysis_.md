## Dynamic Analysis
```
TMPprovider038.dll

My setup ideal for this
Create a restore point. 

Flare-vm -> network -> (Dns server ) Remnux (Inetsim + wireshark)

For capturing details in Flare.
I used
Procmon  -> Monitor the process status 
procexp -> Look at ongoing process.
regshot  -> check registry changes
Wireshark for communication interception.




Basic Analysis
File:     TMPprovider038.dll
Size:     1750016
MD5:      713111BF1249A567B9928C75901A5FB8
Compiled: Fri, Jan 19 2018, 1:36:54 - 32 Bit DLL

Exports:  1
Resources: 1 - 469 bytes

Analyzing dll we can see that it has only 1 export
RunDllEntry

So we can run it with rundll32.exe TMPprovider038.dll , RunDllEntry


Any interesting observations from basic static analysis?
Nothing unusual everything is encrypted. So no unusual sightings.



What do you observe this program doing through dynamic analysis? HBI also included here


Changes in Registry and Queries using regitry.
Such as

Desired Access: Read Data/List Directory, Synchronize, Disposition: Create, Options: Directory, Synchronous IO Non-Alert, Open Reparse Point, Attributes: N, ShareMode: Read, Write, AllocationSize: 0, Impersonating: DESKTOP-B1UK6HF\flare
Enumerate the values using registers.
Query for HKLM\SOFTWARE\Microsoft\SecurityManager\AdminCapabilities\userSigninSupport.

Change of Reg values
HKU\S-1-5-21-643046334-913388524-3226014734-1000\SOFTWARE\Classes\Local Settings\Software\Microsoft\Windows\Shell\BagMRU\4\1\7\4
HKU\S-1-5-21-643046334-913388524-3226014734-1000\SOFTWARE\Classes\Local Settings\Software\Microsoft\Windows\Shell\Bags\52\ComDlg


Keys Delted

HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Group Policy\ServiceInstances
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Group Policy\ServiceInstances\63d61e49-8530-4603-ba9c-c98a163c2b2e
HKLM\SOFTWARE\Microsoft\Windows\Windows Error Reporting\TermReason\2580
HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Group Policy\ServiceInstances
HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Group Policy\ServiceInstances\63d61e49-8530-4603-ba9c-c98a163c2b2e
HKU\S-1-5-21-643046334-913388524-3226014734-1000\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\SessionInfo\1\ApplicationViewManagement\W32:00000000001D05BC
HKU\S-1-5-21-643046334-913388524-3226014734-1000\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\SessionInfo\1\ApplicationViewManagement\W32:000000000038032A
HKU\S-1-5-21-643046334-913388524-3226014734-1000\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\SessionInfo\1\ApplicationViewManagement\W32:00000000008C0224



List any potential network-based indicators of this malware.
Analyzing the network communication from the malware to my configured remnux set up . We can see the followings     

GET /msdownload/update/v3/static/trustedr/en/authrootstl.cab?810fd5a747a20315 HTTP/1.1
Connection: Keep-Alive
Accept: */*
User-Agent: Microsoft-CryptoAPI/10.0
Host: ctldl.windowsupdate.com

GET /MFEwTzBNMEswSTAJBgUrDgMCGgUABBT3xL4LQLXDRDM9P665TW442vrsUQQUReuir%2FSSy4IxLVGLp6chnfNtyA8CEA6bGI750C3n79tQ4ghAGFo%3D HTTP/1.1

---
Edit on file ps:
Mthodologies that we're supposed to do were kinda out of way . SO fixing it now.

D.I.E has 2 method
One is DIE & NFD (Detect it easy) & Nauz file detector
Chossing NFD we sucessfully find it being loaded it packed with VmProtectPacker.



using floss
We can get this


VMProtect Software1
VMProtect Software CA0
160716000000Z
260714235959Z0@1!0
VMProtect Client ipn56211
VMProtect Software0
O=M_x
l[a9Fd>(
)>I?
|>+I
V0T0R
Lhttp://pki-crl.symauth.com/ca_219679623e6b4fa507d638cbeba72ecb/LatestCRL.crl07
+0)0'
http://pki-ocsp.symauth.com0

This dll has one export in it. Named RunDllEntry

Lets save out VMstate first then 
run that dll into our machine.




After running & dynamically assessing Procexp and Wireshark request


These are the one of the suspect here. 
7:45:24.4520523 AM	rundll32.exe	3268	WriteFile	C:\Users\flare\AppData\Local\Temp\qln.dbx	SUCCESS	Offset: 0, Length: 3, Priority: Normal
7:45:24.4627858 AM	rundll32.exe	3268	RegSetValue	HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\TmProvider	SUCCESS	Type: REG_SZ, Length: 150, Data: rundll32 C:\Users\flare\AppData\Local\Temp\TMPprovider038.dll, RunDllEntry
7:45:24.4660037 AM	rundll32.exe	3268	RegSetValue	HKCU\SOFTWARE\Microsoft\Internet Explorer\InternetRegistry\fertger	SUCCESS	Type: REG_SZ, Length: 66, Data: 793D7DBD667E4A61ABE88FC7B33FE964
7:45:24.6009397 AM	rundll32.exe	3268	RegSetValue	HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\ProxyBypass	SUCCESS	Type: REG_DWORD, Length: 4, Data: 1
7:45:24.6009827 AM	rundll32.exe	3268	RegSetValue	HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\IntranetName	SUCCESS	Type: REG_DWORD, Length: 4, Data: 1
7:45:24.6010265 AM	rundll32.exe	3268	RegSetValue	HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\UNCAsIntranet	SUCCESS	Type: REG_DWORD, Length: 4, Data: 1
7:45:24.6010544 AM	rundll32.exe	3268	RegSetValue	HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\AutoDetect	SUCCESS	Type: REG_DWORD, Length: 4, Data: 0
7:45:24.6057667 AM	rundll32.exe	3268	RegSetValue	HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\ProxyBypass	SUCCESS	Type: REG_DWORD, Length: 4, Data: 1
7:45:24.6057953 AM	rundll32.exe	3268	RegSetValue	HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\IntranetName	SUCCESS	Type: REG_DWORD, Length: 4, Data: 1
7:45:24.6058228 AM	rundll32.exe	3268	RegSetValue	HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\UNCAsIntranet	SUCCESS	Type: REG_DWORD, Length: 4, Data: 1
7:45:24.6058499 AM	rundll32.exe	3268	RegSetValue	HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\AutoDetect	SUCCESS	Type: REG_DWORD, Length: 4, Data: 0

We can see a writefile here on qln.dbx -> it contain 044 as a value only nothing more. 


The dll is copied to the temp location. 

there is RegSetValue value changed in here
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\TmProvider

In this case rundll32 is used to launch the DLL export RunDllEntry, establishing persistence on the
host.  ( host-based indicator )

Another HBI would be 
RegSetValue	HKCU\SOFTWARE\Microsoft\Internet Explorer\InternetRegistry\fertger -> the value is set  of 793D7DBD667E4A61ABE88FC7B33FE964 .
We don't know details yet we'll connect with it.


So our Host Based Indicators would be : 

1. WriteFile	C:\Users\flare\AppData\Local\Temp\qln.dbx  ( With data 044 )
2. RegSetValue HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\TmProvider
with the Data: rundll32 C:\Users\flare\AppData\Local\Temp\TMPprovider038.dll, RunDllEntry
3. RegSetValue	HKCU\SOFTWARE\Microsoft\Internet Explorer\InternetRegistry\fertger 
with the value 793D7DBD667E4A61ABE88FC7B33FE964.
 

```