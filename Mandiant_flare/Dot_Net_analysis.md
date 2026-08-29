# .NET, WMI, and PowerShell — Notes

## What Is .NET?

.NET is a software framework consisting primarily of two major components:

* **An execution engine** — the **Common Language Runtime (CLR)**
* **A large class library** — a massive collection of reusable code

### Common Language Infrastructure (CLI)

Microsoft's **Common Language Infrastructure (CLI)** specification:

* Describes executable code and the runtime environment.
* Provides a **platform-agnostic** system.
* Supports language and operating-system independence.

### Supported Languages

Examples include:

* C#
* VB.NET
* F#
* PowerShell
* IronPython
* And others

---

# .NET Analysis Tools

## Static Analysis Tools

Common tools include:

* **CFF Explorer**
* **dnSpy**
* **de4dot**

### Static Analysis Concepts

When analyzing .NET malware, some important concepts include:

* **P/Invoke**
* **Reflection**
* Metadata and entry points
* Managed and unmanaged code interaction

### Indicators of a .NET Executable

Useful indicators include:

* File type
* .NET directory/header information
* CLR metadata

---

# Tool Focus — de4dot

**de4dot** is an automated .NET deobfuscation tool.

It supports many different obfuscators and also provides manual options for unsupported or partially supported obfuscators.

### de4dot Can Perform

* Member renaming
* String decryption
* Control-flow deobfuscation
* Dead-code removal

### EntrypointToken

The **EntrypointToken** identifies the original entry point of a .NET executable through its metadata token.

---

# Platform Invoke (P/Invoke)

Platform Invoke (**P/Invoke**) is built into .NET to provide interoperability with native code.

P/Invoke allows managed .NET code to access:

* Structures
* Callbacks
* Functions
* APIs in unmanaged/native libraries

A Win32 function can be declared in .NET using the **`DllImport`** attribute on a method declaration.

.NET automatically handles the **marshalling** of arguments and return values between managed and unmanaged code.

---

# de4dot Examples

```text
de4dot.exe evil.exe

de4dot.exe evil.exe -o evil_deob.exe --strtyp delegate --strtok 0600003D
```

### Important Options

* **`-o`** — specifies the output filename. The default is generally `<filename>_cleaned`.
* **`--strtyp`** — specifies the string-decryptor type.
* **`delegate`** — indicates that the string decryption routine can be executed dynamically.
* **`--strtok`** — specifies the metadata token of the string-decryption routine.

> **Safety note:** When analyzing malware, perform dynamic execution only in an isolated analysis environment such as a properly configured VM or sandbox.

### Identifying a String-Decryption Routine

Look for a routine that is used throughout the program and has characteristics such as:

* A return type of `String`
* An argument that is often a byte array (`byte[]`)
* Calls occurring where you would normally expect to see strings

### Typical de4dot Workflow

If a sample is obfuscated, first try:

```text
de4dot.exe <sample_name>
```

This attempts to automatically detect the obfuscator and produces a cleaned file, typically named:

```text
<sample_name>_cleaned
```

Examine the cleaned file. If encoded strings are still present:

1. Locate the string-decoding/decryption routine.
2. Identify its metadata token.
3. Run de4dot again with the appropriate string-decryption options.

For example:

```text
de4dot.exe <cleaned_sample_name> -o <new_sample_name> --strtyp delegate --strtok <metadata_token>
```

The **`--strtyp`** and **`--strtok`** parameters tell de4dot which string-decryption mechanism to use and where the relevant routine is located.

---

# Windows Management Instrumentation (WMI)

**Windows Management Instrumentation (WMI)** is used for local and remote system administration.

WMI can be used to:

* Survey system information
* Detect antivirus/security software
* Detect virtual machines
* Enumerate and manipulate processes
* Query hardware and software information

Malware can also abuse these capabilities for reconnaissance, environment detection, and process manipulation.

## Ways Malware Can Connect to WMI

### 1. Using a SWbemServices COM Object

VBScript:

```vbscript
Set oWmi = CreateObject("WbemScripting.SWbemLocator")
```

### 2. Using a Moniker String

VBScript:

```vbscript
GetObject("winmgmts://./root/cimv2")
```

### 3. Using PowerShell

PowerShell can interact with WMI/CIM through cmdlets such as:

```powershell
Get-CimInstance
Get-WmiObject
```

> **Note:** `Get-WmiObject` is an older cmdlet. Modern PowerShell generally uses the CIM cmdlets such as `Get-CimInstance`.

---

# Investigating WMI Activity

If malware is doing something suspicious with WMI, **`wbemtest.exe`** can be used to inspect WMI namespaces, classes, properties, and methods.

Historically, **`wmic.exe`** was also used to interact directly with WMI, although WMIC is deprecated on modern Windows versions.

Example:

```cmd
wmic bios get serialnumber
```

This retrieves the BIOS serial number, which can sometimes be used for system identification or environment checks.

---

# WMI Classes Commonly Queried by Malware

Examples include:

* `Win32_OperatingSystem`
* `Win32_ComputerSystem`
* `Win32_BIOS`
* `Win32_TimeZone`
* `Win32_PageFileUsage`
* `Win32_Processor`
* `Win32_Keyboard`
* `Win32_QuickFixEngineering`
* `Win32_NetworkAdapter`
* `Win32_NetworkAdapterConfiguration`

---

# Understanding WMI Structure

```text
Namespace
    root\cimv2

Class
    Win32_Process

Properties
    Caption       -> Process name
    CommandLine   -> Process name and arguments

Methods
    Create()      -> Creates a process
    Terminate()   -> Terminates a process
```

### WMI Namespaces

WMI classes belong to a particular **namespace**.

The most commonly used namespace is:

```text
root\cimv2
```

WMI classes provide an object-oriented interface to hardware and software through:

* **Properties** — represent data
* **Methods** — perform actions

### Example: Win32_Process

Malware can use `Win32_Process` to:

* Enumerate running processes
* Compare process names against analysis/debugging tools
* Identify processes of interest
* Potentially terminate selected processes

For example, malware might look for tools such as **Process Explorer** or **Process Monitor** as part of an analysis-evasion technique.

---

# WMI Classes and Documentation

Example:

```text
Name:
    Win32_Group

Derives from:
    Win32_Account

Properties:
    Caption
    Description
    SID
    ...

Methods:
    Rename
```

The `Rename` method can rename the Windows group associated with a particular class instance.

---

# WMI Query Language (WQL)

**WMI Query Language (WQL)** is similar to SQL but has some limitations.

General structure:

```sql
SELECT <fields>
FROM <class>
WHERE <property> <operator> <constant>
```

Example:

```sql
SELECT * FROM Win32_LogicalDisk WHERE FileSystem = "NTFS"
```

WMI queries commonly return a **collection of objects**.

Applications, administrators, and malware can iterate over these objects to read information or, where permitted, invoke methods that modify system state.

---

# WMI Capabilities and Classes

| Capability                                   | WMI Class or Namespace                                        |
| -------------------------------------------- | ------------------------------------------------------------- |
| VM detection / CPU information               | `Win32_ComputerSystem`, `Win32_BIOS`, `Win32_PnPEntity`, etc. |
| Process enumeration / termination / creation | `Win32_Process`                                               |
| Shadow-copy management                       | `Win32_ShadowCopy`                                            |
| Antivirus/security-product checks            | `AntiVirusProduct` in `root\SecurityCenter2`                  |
| Software inventory                           | `Win32_Product`                                               |
| OS information                               | `Win32_OperatingSystem`                                       |

---

# Example: WMI-Based Environment Check

The following VBScript queries the BIOS serial number:

```vbscript
Set wmi = GetObject("winmgmts:")
Set col_bios = wmi.ExecQuery("SELECT * FROM Win32_BIOS")

For Each bios In col_bios
    Echo(bios.SerialNumber)
Next
```

This type of query can be useful for legitimate system administration and can also appear in malware when gathering system-identification information.

---

# PowerShell

**PowerShell** is Microsoft's object-oriented command-line shell and scripting environment.

Key characteristics:

* Object-oriented
* Built on .NET
* Provides access to COM and WMI/CIM
* Can interact with native Windows APIs through .NET and other mechanisms
* Supports powerful scripting and automation

PowerShell has also been abused as a runtime or execution mechanism by malware, including:

* Backdoors
* Downloaders
* Credential-theft tools
* Shellcode loaders

---

# PowerShell Execution Policy

PowerShell's **execution policy** controls how PowerShell scripts are permitted to run.

Common policies include:

* `Restricted`
* `AllSigned`
* `RemoteSigned`
* `Unrestricted`

Execution policy is **not a complete security boundary**. It primarily controls script execution behavior and can be affected by configuration, scope, and invocation method.

### Common Ways Attackers Attempt to Avoid Script Restrictions

Examples include:

* Typing or pasting commands directly into an interactive PowerShell session
* Supplying commands through command-line parameters
* Using encoded commands
* Using download-and-execute techniques
* Modifying applicable execution-policy settings

> **Note:** These techniques should be studied in controlled malware-analysis or lab environments.

---

# PowerShell Command-Line Options

| Full Option                 | Common Short Form | Purpose                                          |
| --------------------------- | ----------------- | ------------------------------------------------ |
| `-ExecutionPolicy <policy>` | `-ep <policy>`    | Specify execution policy for the session/process |
| `-NoProfile`                | `-nop`            | Do not load the PowerShell profile               |
| `-NonInteractive`           | `-noni`           | Run without an interactive prompt                |
| `-WindowStyle Hidden`       | `-w hidden`       | Specify the PowerShell window style              |
| `-Command <script>`         | `-c <script>`     | Execute a command or script                      |
| `-EncodedCommand <Base64>`  | `-enc <Base64>`   | Execute a Base64-encoded command                 |

Reference:

[NetSPI — 15 Ways to Bypass the PowerShell Execution Policy](https://www.netspi.com/blog/technical-blog/network-penetration-testing/15-ways-to-bypass-the-powershell-execution-policy/?utm_source=chatgpt.com)

---

# Cmdlets

**Cmdlet** is pronounced *command-let*.

Cmdlets are:

* Lightweight commands designed specifically for PowerShell
* .NET-based components
* Able to receive and return .NET objects
* Designed to work together through the PowerShell pipeline

Cmdlets are fundamental building blocks of PowerShell functionality.

Many cmdlets are frequently encountered during malware analysis.

---

# Cmdlets and Aliases

Example:

```powershell
Write-Output
```

`Write-Output` sends objects or strings to the PowerShell pipeline.

PowerShell also supports **aliases**, which provide alternative names for commands.

Examples:

```text
dir
echo
cat
cd
cls
copy
cp
del
```

Some common aliases include:

```text
Alias    %     -> ForEach-Object
Alias    ?     -> Where-Object
Alias    ac    -> Add-Content
Alias    cat   -> Get-Content
Alias    cd    -> Set-Location
Alias    cls   -> Clear-Host
Alias    del   -> Remove-Item
```

You can inspect available aliases with:

```powershell
Get-Alias
```

---

# PowerShell Pipelines

PowerShell pipelines are conceptually similar to pipelines in Bash or `cmd.exe`, but there is an important difference:

> PowerShell passes **objects**, rather than simply passing text.

This allows commands to access object properties, filter objects, and pass them to subsequent cmdlets.

Example:

```powershell
Get-Process |
    Where-Object { $_.ProcessName -like "procmon*" } |
    Stop-Process
```

The pipeline:

1. Retrieves running processes with `Get-Process`.
2. Filters processes whose names match `procmon*`.
3. Passes the matching process objects to `Stop-Process`.

---

# Get-Member

`Get-Member` can be used to inspect the properties and methods available on PowerShell objects.

Example:

```powershell
Get-Date | Get-Member -MemberType Properties
```

Example output:

```text
TypeName: System.DateTime

Name        MemberType     Definition
----        ----------     ----------
DisplayHint  NoteProperty  DisplayHintType DisplayHint=DateTime
Date         Property      datetime Date {get;}
Day          Property      int Day {get;}
DayOfWeek    Property      System.DayOfWeek DayOfWeek {get;}
DayOfYear    Property      int DayOfYear {get;}
Hour         Property      int Hour {get;}
```

---

# Example: Creating a PowerShell Function

A PowerShell function can accept parameters and execute commands.

Example:

```powershell
PS C:\Users\cyanm> function hi_to_youu($me, $you) {
    Add-Type -AssemblyName PresentationFramework
    [System.Windows.MessageBox]::Show(
        "Hi from $env:USERNAME! $me $you",
        "Message"
    )
}

PS C:\Users\cyanm> hi_to_youu "hello" "what's up?"
```

The function displays a Windows message box containing the current username and the supplied arguments.

For example:

```text
Hi from cyanm! hello what's up?
```





```text
Cmdlet: Get-ChildItem (dir)
In a directory, child items include:
• Files
• Sub-directories
Some objects have properties as well
• Get-ItemProperty to retrieve

Cmdlet: New-Object -ComObject
Shown here: WinHttp.WinHttpRequest.5.1
• Can be used to download and later execute further script code
• Other options include: Msxml2.XMLHTTP, InternetExplorer.Application
• Can shorten the argument to -com

Cmdlet: New-Object (for .NET objects)
Shown here: System.Net.WebClient
• Often used to download and later execute further script code

Cmdlet: Start-BitsTransfer
BITS (Background Intelligent Transfer Service) client downloads via HTTP
• BITS protocol provides more robust transfer capabilities than HTTP
• Attackers are mainly interested for evasion purposes

Cmdlets: Get-CimInstance / Get-WmiObject
Get-CimInstance is the up-to-date cmdlet to use for WMI objects
Get-WmiObject (gwmi) is all but deprecated
• You will still see malware using it (backward compatible for now)

Use Get-CimInstance or Get-WmiObject to access WMI classes from PowerShell.
gwmi is an Alias for Get-WmiObject.

WQL via Get-CimInstance / Get-WimiObject
Get-CimInstance and Get-WmiObject (gwmi) support WQL
• The -query argument accepts WMI Query Language text
• Output may be iterated


PS C:\Users\flare\Desktop > $p = Get-CimInstance -Query " select * from Win32_Process"
FLARE-VM 08/28/2026 18:00:48
PS C:\Users\flare\Desktop > $p[110]

ProcessId Name        HandleCount WorkingSetSize VirtualSize
--------- ----        ----------- -------------- -----------
4964      dllhost.exe 321         14163968       2203474829312


Cmdlet: Invoke-Expression (iex)
What it does: Executes a string as PowerShell script code
Common use: Running decrypted or Base64-decoded script code

PS C:\Users\flare\Desktop > $q = "write-host hello"
Example:
PS C:\Users\flare\Desktop > iex $q
hello


Cmdlet: Add-Type
What it does: Defines a new .NET class in this PowerShell session
Common use: .NET access to use P/Invoke and directly call Windows API functions

$code= '[Dlllmport( "kernel32.dll" )] public static extern IntPtr UirtualAlloc([snip ... ] ' :
$winFunc = Add-Type -member-Definition $code -Name "Win32" -namespace Win32Functions -passthru;
[Byte[]]$sc = Ox83 , OxEC , Ox28 , [snip ... ]. OxC3 ;
$mem = $winFunc: :Uirtua1Alloc(O.$sc.Length. Ox3000 . Ox40 ):
for ($i = O; $i -le ($sc.Length- 1); $i++) $winFunc::memset(($mem.Tolnt64()+$i), $sc[$i]. 1)};
$h = $winFunc: :CreateThread(O.O.$mem, O,O.O):
$winFunc: :WaitForSingleObject($h. 4294967295);

This is a shellcode launcher for BLUESTEAL POS malware written in PowerShell that uses:
1. VirtualAlloc to creates a read/write/execute buffer in memory (0x40 = PAGE_EXECUTE_READWRITE)
2. memset to copy the shellcode into the buffer
3. CreateThread to create a thread that executes the shellcode
4. WaitForSingleObject to wait indefinitely on that thread to terminate
.NET P/Invoke is used to import the Windows API functions.

Notable Observables from Add-Type
• Add-Type invokes a compiler (usually C# is used)
• Produces file and process observables (csc.exe, cvtres.exe)


```

Add Windows Defender exclusion
Delete volume shadow copies
Disable script block logging*
Add-MpPreference -ExclusionPath "<path>"
Get-WmiObject Win32_ShadowCopy I
ForEach-Object {$_.Delete();}
Write to HKLM\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging

Cmdlet: Add-Type
What it does: Defines a new .NET class in this PowerShell session
Common use: .NET access to use P/Invoke and directly call Windows API functions

https://learn.microsoft.com/en-us/windows/win32/apiindex/windows-api-list

api list for windows
