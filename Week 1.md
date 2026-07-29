# DNS ENUMERATION

```bash
whois
whois megacorpone.com
whois megacorpone.com -h 192.168.239.251
whois offensive-security.com -h 192.168.239.251
whois kali.org -h 192.168.239.251
```
```bash
dnsenum megacorpone.com
dnsenum VERSION:1.3.1

-----   megacorpone.com   -----                                  
Host's addresses:
megacorpone.com.                         300      IN    A        149.56.244.87                                                                                                                                                                     

Name Servers:
ns3.megacorpone.com.                     300      IN    A        66.70.207.180                                                                                                                                                                     
ns1.megacorpone.com.                     300      IN    A        51.79.37.18
ns2.megacorpone.com.                     300      IN    A        51.222.39.63

Mail (MX) Servers:                                                 
fb.mail.gandi.net.                       60       IN    A        217.70.178.217                                                                                                                                                                    
fb.mail.gandi.net.                       60       IN    A        217.70.178.215
fb.mail.gandi.net.                       60       IN    A        217.70.178.216
mail2.megacorpone.com.                   300      IN    A        167.114.21.69
mail.megacorpone.com.                    300      IN    A        167.114.21.68
spool.mail.gandi.net.                    445      IN    A        217.70.178.1

Trying Zone Transfers and getting Bind Versions:
Trying Zone Transfer for megacorpone.com on ns3.megacorpone.com ... 
AXFR record query failed: REFUSED
Trying Zone Transfer for megacorpone.com on ns1.megacorpone.com ... 
AXFR record query failed: REFUSED
Trying Zone Transfer for megacorpone.com on ns2.megacorpone.com ... 
megacorpone.com.                         300      IN    SOA               (
megacorpone.com.                         300      IN    A        149.56.244.87
megacorpone.com.                         300      IN    TXT            "Try
megacorpone.com.                         300      IN    TXT               (
megacorpone.com.                         300      IN    MX               10
megacorpone.com.                         300      IN    MX               20
megacorpone.com.                         300      IN    MX               50
megacorpone.com.                         300      IN    MX               60
megacorpone.com.                         300      IN    NS       ns1.megacorpone.com.
megacorpone.com.                         300      IN    NS       ns2.megacorpone.com.
megacorpone.com.                         300      IN    NS       ns3.megacorpone.com.
admin.megacorpone.com.                   300      IN    A        167.114.21.64
ai.megacorpone.com.                      300      IN    A        149.56.244.87
beta.megacorpone.com.                    300      IN    A        167.114.21.65
fs1.megacorpone.com.                     300      IN    A        167.114.21.66
intranet.megacorpone.com.                300      IN    A        167.114.21.67
mail.megacorpone.com.                    300      IN    A        167.114.21.68
mail2.megacorpone.com.                   300      IN    A        167.114.21.69
ns1.megacorpone.com.                     300      IN    A        51.79.37.18
ns2.megacorpone.com.                     300      IN    A        51.222.39.63
ns3.megacorpone.com.                     300      IN    A        66.70.207.180
router.megacorpone.com.                  300      IN    A        167.114.21.70
siem.megacorpone.com.                    300      IN    A        167.114.21.71
snmp.megacorpone.com.                    300      IN    A        167.114.21.72
support.megacorpone.com.                 300      IN    A        167.114.21.74
syslog.megacorpone.com.                  300      IN    A        167.114.21.73
test.megacorpone.com.                    300      IN    A        167.114.21.75
vpn.megacorpone.com.                     300      IN    A        167.114.21.76
vpn2.megacorpone.com.                    300      IN    A        167.114.21.77
vpndev.megacorpone.com.                  300      IN    A        167.114.21.78
vpnprod.megacorpone.com.                 300      IN    A        167.114.21.79
www.megacorpone.com.                     300      IN    A        149.56.244.87
www2.megacorpone.com.                    300      IN    A        149.56.244.87

Brute forcing with /usr/share/dnsenum/dns.txt:
megacorpone.com class C netranges:                                                                                                                                                                             
 51.79.37.0/24
 51.222.39.0/24
 66.70.207.0/24
 149.56.244.0/24
 167.114.21.0/24

Performing reverse lookup on 1280 ip addresses:
0 results out of 1280 IP addresses.

megacorpone.com ip blocks:                                  
done.
```

# Connect via freerdp

```bash
xfreerdp /u:student /p:lab /v:192.168.225.152
```

# portscan met Netcat

## tcp
```bash
nc -nvv -w 1 -z 192.168.225.152 3388-3390
nc -nvv -w 1 -z 192.168.225.151 3388-3390
nc -nvv -w 1 -z 192.168.225.151 1-1000
nc -nvv -w 1 -z 192.168.225.151 1-10000
nc -nvv -w 1 -z 192.168.225.151 150-200
nc -nvv -w 1 -z 192.168.225.151 149-200
nc -nvv -w 1 -z 192.168.225.151 1-220
```

## udp
```bash
nc -nv -u -z -w 1 192.168.50.149 150-200
nc -nv -u -z -w 1 192.168.255.151 150-200
nc -nv -u -z -w 1 192.168.225.151 150-200
```

# portscan met nmap

SYN scan (TCP scan without completing the TCP handshake, so it is stealthier than a full connect scan)
However modern firewalls can detect SYN scans, so it is not always effective.
```bash
sudo nmap -sS 192.168.225.0/24
```

# portscan via Windows

```powershell
Test-NetConnection -Port 445 192.168.225.151
```

of meerdere:

```powershell
1..1000 | ForEach-Object {Test-NetConnection -Port $_ 192.168.225.151}
```

```powershell
1..1024 | % {echo ((New-Object Net.Sockets.TcpClient).Connect("192.168.225.151", $_)) "TCP port $_ is open"} 2>$null
```

# finding flag port
```bash
┌──(kali㉿kali)-[~/git/megacorpone.com]
└─$ sudo nmap -p - -nP -sS 192.168.225.52
Starting Nmap 7.95 ( https://nmap.org ) at 2026-05-21 07:43 EDT
Nmap scan report for 192.168.225.52
Host is up (0.023s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
2222/tcp  open  EtherNetIP-1
59811/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 46.06 seconds
                                                                                                                                                                                                                                                   
┌──(kali㉿kali)-[~/git/megacorpone.com]
└─$ nc 192.168.225.52 59811                  
You found me. Great job!
Here is your flag:
OS{dff4cd8a2f95aa1096393927e65587db}
```

# smb enum
```bash
nmap -v -p 139,445 -oG smb.txt 192.168.50.1-254

                                                                                                                                                                                                                                                   
┌──(kali㉿kali)-[~/git/offsec-week-1]
└─$ grep "139/open\|445/open" smb.txt | awk '{print $2}' 

192.168.239.9
192.168.239.11
192.168.239.12
192.168.239.13
192.168.239.14
192.168.239.15
192.168.239.20
192.168.239.149
192.168.239.151
192.168.239.152
                                                                                                                                                                                                                                                   
┌──(kali㉿kali)-[~/git/offsec-week-1]
└─$ grep "139/open\|445/open" smb.txt | awk '{print $2}' > hosts-with-open-smb.txt

grep "139/open\|445/open" smb.txt | awk '{print $2}' | while read host; do 
  enum4linux -a "$host"
done
```

## linux
```bash
sudo nbtscan -r 192.168.50.0/24
smbclient -L
```

## windows
```batch
net view \\dc01 /all
```

# smtp enum
```bash
──(kali㉿kali)-[~/git/offsec-week-1]
└─$ sudo nmap -sS -p 25 192.168.239.0/24
[sudo] password for kali: 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-05-26 09:57 EDT
Stats: 0:00:19 elapsed; 0 hosts completed (0 up), 256 undergoing Ping Scan
Parallel DNS resolution of 17 hosts. Timing: About 58.82% done; ETC: 09:57 (0:00:11 remaining)
Nmap scan report for 192.168.239.8
Host is up (0.022s latency).

PORT   STATE SERVICE
25/tcp open  smtp
                                                                                                                                                                                                                 
┌──(kali㉿kali)-[~/git/offsec-week-1]
└─$ nc -nv 192.168.239.8 25             
(UNKNOWN) [192.168.239.8] 25 (smtp) open
220 mail ESMTP Postfix (Ubuntu)
vrfy root
252 2.0.0 root
vrfy nobody
252 2.0.0 nobody
sdaf
exit
^C
```

# nmap
Scripts 
NSE scripts contain a wide range of functionalities, including vulnerability detection, service enumeration, and more. The `vuln` category includes scripts that check for known vulnerabilities in services.

```bash
sudo nmap -sV -p 443 --script "vuln" 192.168.50.13
```

To find a suitable NSE script, we can use a search engine to find the CVE number plus NSE
```bash
sudo cp /home/kali/Downloads/http-vuln-cve-2021-41773.nse /usr/share/nmap/scripts/http-vuln-cve2021-41773.nse
#then
sudo nmap -sV -p 443 --script "http-vuln-cve2021-41773" 192.168.50.124
```
