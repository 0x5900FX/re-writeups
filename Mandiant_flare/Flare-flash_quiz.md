Basic Malware Analysis  ( Page 12 ) 
```
1. What type of file might this be?
Windows application

2. Does the  malware appear to persist after reboot?
Yes with 
WriteFile -> write content to file
CreateFileA -> create a file content

3. What protocol is likely used for network communication?
Probably http:
Mozilla/ 5.0 (Windows NT 6.1; Win64; x64)
GET

4. Why type of malware might this be?
Trojan Dropper or Botnet C2.
```

--- 

Impoerts Flare- Flash quiz  ( Page 28 )

```
1. Which series of imports indicates the malware has the capability to write a file to disk and execute it?
c. CreateFileA, WriteFile, WinExec

2. True or False: A sample that imports the send function definitely sends data over a network socket.
False

3. When reviewing imports, we typically attempt to identify capabilities. Which function is not associated with
network functionality?
QueryServiceStatus
```